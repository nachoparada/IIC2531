---
marp: true
theme: default
paginate: true
backgroundColor: #fff
backgroundImage: url('https://marp.app/assets/hero-background.svg')
style: |
  section {
    font-size: 28px;
    display: flex;
    flex-direction: column;
    justify-content: flex-start;
    align-items: flex-start;
    padding-top: 50px;
  }

  img[alt~="align-right"] {
    margin-left: 400px;
  }

  img[alt~="align-center"] {
    position: absolute;
    left: 50%;
    transform: translateX(-50%);
  }

  /* Right-align terminal commands */
  .terminal-commands {
    text-align: right;
    margin-left: 400px;
  }

  /* Make sub-bullets lighter and smaller */
  ul ul li, ol ol li {
    color: #666666;
    font-size: 0.9em;
  }

  /* Make nested sub-bullets even lighter, italic, and smaller */
  ul ul ul li, ol ol ol li {
    color: #666666;
    font-style: italic;
    font-size: 0.8em;
  }

  /* Alternative: Use opacity for a more subtle effect */
  ul ul, ol ol {
    opacity: 0.8;
  }

---

# Asegurando interfaces: sandboxing de bibliotecas

---

# Idea central de esta clase
  * Esta clase trata sobre **cómo implementar interfaces seguras entre componentes con distintos niveles de confianza**
  * Aislar un componente no basta: también debemos controlar y validar todo lo que cruza la frontera
  * Usaremos **RLBox como caso de estudio** para identificar problemas recurrentes y mecanismos de diseño
  * La meta no es aprender una API específica, sino principios aplicables a sandboxes, procesos, RPCs y otros límites de confianza

---

# Objetivo: sandboxing de bibliotecas en aplicaciones grandes (ej., Firefox)
  * Las bibliotecas podrían tener errores de seguridad
  * Queremos asegurar que los errores de biblioteca no se traduzcan en vulnerabilidades de aplicación
  * Podríamos no obtener datos significativos de la biblioteca pero queremos evitar ataques
  * Necesitamos aislamiento + compartir controlado con la biblioteca
  * Solución: RLBox

---

# Escenario conductor: un JPEG malicioso
  * Alicia abre en Firefox un correo con una imagen `foto.jpg` controlada por el atacante
  * El *decoder* JPEG tiene un bug de corrupción de memoria
  * El JPEG explota el bug: desde ese momento, asumimos que **todo el código de la biblioteca está comprometido**
  * La biblioteca intenta devolver dimensiones, offsets, punteros o callbacks maliciosos al renderer
  * **¿Cómo evitamos que el compromiso cruce la interfaz y tome control del host?**

---

# Modelo de amenaza del escenario JPEG
  * **Objetivo de seguridad:** un decoder JPEG comprometido no debe poder leer, modificar ni controlar el resto del navegador
  * **Capacidades del adversario:**
    * Controla completamente el archivo JPEG de entrada
    * Puede explotar un error y tomar control total de la ejecución del decoder
    * Puede retornar valores y punteros arbitrarios, cambiar datos entre lecturas e invocar callbacks en momentos inesperados
  * **Suposiciones:**
    * El código del navegador fuera del decoder es correcto
    * El decoder solo debería acceder a los datos que la aplicación comparte explícitamente con él
  * **Fuera de alcance:** disponibilidad ante cuelgues/DoS y errores en el resto del navegador

---

# Nuevo problema: securitizar la interfaz
  * Ya sabemos cómo aislar código bastante bien
    * Procesos, VMs, WebAssembly
    * El paper usa WebAssembly, Native Client (predecesor de WebAssembly), y procesos
  * Un gran enfoque en este paper es diseñar la interfaz entre cajas aisladas

---

# Aparte: vemos algunas de las ideas de separación de privilegios en contexto del lado del cliente
  * Hasta ahora hemos hablado de código del lado del servidor (Google, OKWS, Firecracker)
  * RLBox muestra ideas similares siendo usadas en código del lado del cliente, en un navegador web
  * Aunque las ideas centrales son aplicables en general
    * Podríamos imaginar usar RLbox en contexto del lado del servidor también
    * Ej., sandboxing codecs de video en servidores de Youtube

---

# Antecedentes: aislamiento de navegador web
  * Hablaremos más sobre seguridad web después, pero podemos discutir lo básico
  * Diseño típico: proceso renderer para cada ventana / pestaña
    * El objetivo es prevenir que vulnerabilidades del navegador comprometan el SO subyacente
    * Solía ser relativamente menos importante cuando el navegador era solo una de muchas apps
    * Hoy en día la mayoría de las cosas corren en el navegador, no tantas apps no-navegador que importen
    * Los compromisos podrían quizás acceder a todas las cookies del usuario
    * Cualquier sitio web podría estar en cualquier pestaña como imagen, frame, etc

---

# Aislamiento de sitio
  * Enfoque relativamente más nuevo en Firefox, Chrome
    * [Chromium: Site Isolation](https://www.chromium.org/Home/chromium-security/site-isolation/)
  * Proceso por dominio (como google.com)
  * El atacante aún puede ser bastante dañino
    * Solo necesita inyectar una imagen que se renderice con biblioteca con errores en google.com
    * Ej., enviar imagen como adjunto de email, subir imagen a Google Maps, Google Photos, etc
    * El código de biblioteca comprometido puede acceder a cookie de google.com

---

# Primer intento: aislar multimedia en un proceso separado

  * Firefox puede ejecutar los codecs de multimedia en un proceso restringido, separado del renderer
  * **Beneficio:** si un codec es comprometido, no obtiene acceso directo a la memoria del renderer ni a todos los recursos del sistema
  * **Limitación:** todas las bibliotecas dentro de ese proceso siguen compartiendo el mismo dominio de confianza
    * Comprometer un codec permite atacar las demás bibliotecas y datos presentes en el proceso
  * Ejemplo: si una biblioteca sensible como `gzip` comparte el proceso, un codec comprometido podría controlar cómo se usa
    * Podría hacer que `gzip` produzca JavaScript elegido por el atacante
    * El renderer recibiría ese resultado como si proviniera de una biblioteca confiable
  * **Conclusión:** el aislamiento por proceso contiene el ataque, pero la frontera todavía es demasiado amplia

---

# Mejor frontera: aislar cada biblioteca
  * La frontera de aislamiento debería seguir el principio de **mínimo privilegio**
    * Cada biblioteca recibe solo la entrada que necesita y produce una salida limitada
  * Un decoder JPEG comprometido no debería poder acceder a otros codecs, bibliotecas ni datos del renderer
  * Codecs de imagen y video, bibliotecas de fuentes y descompresores son buenos candidatos
    * Procesan una entrada y producen una salida bien definida
    * Mantienen poco estado persistente entre operaciones
  * Pero crear una frontera más pequeña introduce un nuevo problema:
    * **¿Cómo diseñamos una interfaz segura entre la aplicación y una biblioteca no confiable?**
  * RLBox es el caso de estudio para responder esa pregunta

---

# ¿Cuántos sandboxes?
  * Crear sandbox aún incurre algún overhead (1-2 msec)
  * Un sandbox para todo el renderer: no es genial porque algo del contenido es importante
    * Ej., la biblioteca gzip podría descomprimir Javascript que corre en una página
  * El paper amortiza el costo agrupando por biblioteca, tipo de contenido, y de dónde vino el contenido
    * <renderer, biblioteca, origen-contenido, tipo-contenido>

---

# ¿Por qué la interfaz es un desafío?
  * Hasta cierto punto siempre va a ser complicado
    * Pero en gran medida este problema surge porque estamos reutilizando límite existente
  * La interfaz aplicación-biblioteca no fue diseñada originalmente para ser no confiable
  * La aplicación necesita preocuparse por qué datos podría estar dando a la biblioteca
  * La aplicación necesita validar datos que vienen de la biblioteca
  * En nuestro caso: aislar el decoder JPEG **no basta** si el renderer confía en sus respuestas

---

# Antes: llamada convencional a una biblioteca confiable

<style scoped>
  pre {
    width: 100%;
    font-size: 0.72em;
    line-height: 1.2;
  }
</style>

```cpp
// C++ simplificado: jpeg_decode se considera parte confiable del proceso.
JpegInfo info{};
if (jpeg_decode(bytes, size, &info) != JPEG_OK) {
  return error();
}

// El contrato de la biblioteca basta: usamos su resultado directamente.
std::vector<Pixel> pixels(info.width * info.height);
render(pixels, info.width, info.height);
```

  * Este código es razonable **solo** bajo el supuesto original: biblioteca correcta y confiable
  * Tras explotar `foto.jpg`, `width`, `height` y el status pasan a ser entradas del atacante

---

# Después (mal): sandbox sin una interfaz segura

<style scoped>
  pre {
    width: 100%;
    font-size: 0.72em;
    line-height: 1.2;
  }
</style>

```cpp
// Pseudocódigo simplificado; la API exacta depende del sandbox.
Sandbox sbx;
JpegInfo* shared = sbx.alloc<JpegInfo>();
int status = sbx.call(jpeg_decode, bytes, size, shared);

// ❌ El host copia y usa respuestas adversarias sin validarlas.
size_t count = shared->width * shared->height; // overflow posible
std::vector<Pixel> pixels(count);
copy_from_sandbox(pixels.data(), shared->data, count);
render(pixels, shared->width, shared->height);
```

  * El aislamiento limita acceso directo, pero esta interfaz vuelve a otorgar capacidad al atacante
  * Un status, tamaño, puntero u offset malicioso puede inducir corrupción **en el host**

---

# Ejemplos de errores: sección 3

---

# No sanitizar datos que vienen de la biblioteca (sandbox)
  * En `foto.jpg`, el decoder comprometido puede devolver ancho, alto u offset elegidos por el atacante
  * Ejemplo: la biblioteca pide saltar N bytes de datos de entrada
  * El código existente probablemente no verifica que N esté en límites (la biblioteca es confiable)
  * La biblioteca comprometida en sandbox puede pedir N mucho más grande que el tamaño del buffer
    * Causar lecturas o escrituras de memoria fuera de límites en código fuera del sandbox
  * Otro ejemplo: la biblioteca retorna un código de error o flag inesperado

---

# Conversiones de punteros
  * El código en la aplicación (renderer) y biblioteca (sandbox) tienen memorias diferentes
  * Los punteros en una memoria son sin sentido en otra memoria
  * Necesitamos copiar explícitamente el contenido de memoria -- no podemos solo pasar punteros existentes
  * ¿Qué sale mal?
    * El código de biblioteca o código de app se rompe (corrompe memoria): el puntero es basura
    * El código de biblioteca puede engañar al código de app para sobrescribir memoria arbitraria
    * La app puede filtrar direcciones que debilitan ASLR (*Address Space Layout Randomization*)
      * ASLR ubica código y datos en direcciones impredecibles para dificultar exploits
      * Aunque la app no use el puntero, la biblioteca comprometida puede observarlo y descubrir dónde está ubicada la memoria del host

---

# Errores de doble-fetch
  * El código de app accede a los mismos datos compartidos dos veces
  * En el JPEG: valida `width = 800`, pero el decoder lo cambia a `0xffffffff` antes de copiar píxeles
  * Ej., struct compartido; verificar si offset está en límites, luego usar ese offset
  * La biblioteca comprometida puede modificar el valor entre las dos verificaciones: condición de carrera

---

# Callbacks
  * En el JPEG: el decoder podría invocar un callback para pedir el siguiente bloque de bytes
  * La biblioteca podría necesitar hacer callback a la aplicación: ej., obtener más datos de entrada
  * Necesitamos otorgar de forma segura a la biblioteca acceso a funciones callback específicas
  * No es seguro que la biblioteca llame cualquier función en la aplicación
  * Pero la API existente solo pasa un puntero de 64 bits para especificar callback

---

# Argumentos de callback
  * A menudo las interfaces de biblioteca involucran que la biblioteca pase algo de estado de vuelta a función callback
    * Ej., puntero a alguna estructura de datos a nivel de app
  * La biblioteca comprometida puede pasar puntero arbitrario
  * Podríamos incluso necesitar restringir los argumentos a esas funciones callback

---

# Timing de callback
  * La aplicación podría no estar esperando un callback en algún punto de su ejecución
  * Ej., podría esperar que la biblioteca invoque callback de error solo después de que ocurra el error

---

# Callbacks vs hilos
  * Podríamos esperar ciertos callbacks en ciertos hilos
  * Ej., dos hilos llaman biblioteca, biblioteca corre cb de un hilo en otro hilo
    * O mismo cb de un hilo en ambos hilos
  * Podría llevar a corrupción de memoria de aplicación, condiciones de carrera, etc

---

# Mecanismos para ayudar a desarrolladores: sección 4

---

# ¿Qué fuerzan los mecanismos de RLBox?

  * **Sandbox:** bloquea el acceso directo de la biblioteca a memoria fuera de su dominio
  * **Tipos _tainted_:** impiden usar accidentalmente valores no confiables como valores normales
  * **Copia + `verify`:** exige un punto explícito donde copiar y validar un valor
  * **Callbacks registrados:** restringen qué funciones puede invocar la biblioteca y con qué tipos
  * RLBox hace visibles los cruces peligrosos y bloquea clases de errores durante la compilación

---

# ¿Qué no garantiza RLBox por sí solo?

  * No determina si una respuesta de la biblioteca **tiene sentido** para la aplicación
  * No puede saber si el desarrollador olvidó una invariante en el validador
  * No garantiza que el orden, frecuencia o estado lógico de los callbacks sea válido
  * Tampoco corrige una configuración defectuosa del aislamiento
  * La garantía final requiere:
    * **aislamiento correcto**
    * **mecanismos RLBox**
    * **validadores semánticos correctos**

---

# Aplicado a `foto.jpg`: ¿quién hace qué?
  * **RLBox/tipos:** `width`, `height`, status y punteros llegan como *tainted*; el host no puede usarlos directamente
  * **Sandbox:** la biblioteca comprometida no puede escribir directamente la memoria del renderer
  * **Desarrollador:** define qué significa un resultado aceptable
    * status dentro del conjunto esperado
    * dimensiones positivas, límites de recursos y multiplicación sin overflow
    * buffer de píxeles consistente con formato, stride y dimensiones
  * Si el desarrollador valida solo `width > 0`, RLBox no inventará los chequeos que faltan

---

# Valores "tainted" (contaminados)
  * Intuición: valores tainted representan cosas que están en el sandbox, no confiables
    * Valores untainted están fuera del sandbox, confiables
    * No podemos pasar valores untainted al sandbox, o usar valores tainted fuera del sandbox
    * Necesitamos verificaciones explícitas para deshacerse de "taint"
    * En algunos casos necesitamos también convertir explícitamente a valores "tainted"
  * Operadores aritméticos y otros funcionan en valores tainted pero mantienen taint
  * No podemos realizar operaciones en valores tainted que tengan efectos secundarios
    * Ej., if (tainted_bool || foo()) invocará foo() solo si tainted_bool es falso

---

# Valores "tainted" (cont.) - Estructuras

<style scoped>
    pre {
    width: 70%;
    margin: 0 auto;
}
</style>

```cpp
// Pseudocódigo RLBox simplificado.
tainted<int> t_width  = sandbox_invoke(jpeg_sbx, get_width);
tainted<int> t_height = sandbox_invoke(jpeg_sbx, get_height);

// No se pueden usar como int normales:
int width = t_width;   // ❌ error de compilación
int height = t_height; // ❌ error de compilación

// Hay que copiar y validar explícitamente:
int safe_width = t_width.copy_and_verify([](int w) {
  return (w > 0 && w <= 8192) ? w : 0;
});
int safe_height = t_height.copy_and_verify([](int h) {
  return (h > 0 && h <= 8192) ? h : 0;
});

// Ojo: aún falta validar overflow y consistencia del buffer.
resize_canvas(safe_width, safe_height); // ✅ valores ya fuera del sandbox
```


---

# Valores "tainted" (cont.) - Punteros

  * Punteros tainted representan punteros a memoria del sandbox
    * En contraste, punteros regulares son punteros a memoria de aplicación
    * Necesitamos asignar explícitamente memoria en sandbox, obtener tainted<T*> de vuelta
    * Necesitamos copiar explícitamente memoria; no podemos solo convertir entre T* y tainted<T*>

---

# Desenvolver con un validador
  * Específico de aplicación, el desarrollador debe pensar qué es necesario
  * En `foto.jpg`, verificar que el status del decoder sea esperado **no basta**: también dimensiones, overflow, tamaño de buffer y formato
  * RLBox exige pasar por un validador; no demuestra que su política esté completa
  *
  ```
     tainted<int>.verify(
      [](int val) {
        if (val == ...) {
          return val;
        } else {
          panic;
        }
     });
  ```

---

# Después (bien): RLBox obliga a validar

<style scoped>
  pre {
    width: 100%;
    font-size: 0.72em;
    line-height: 1.2;
  }
</style>

```cpp
// Pseudocódigo RLBox simplificado: nombres exactos pueden variar.
auto t_info = sandbox_invoke(jpeg_sbx, jpeg_decode, t_bytes, size);

ImageSpec spec = t_info.copy_and_verify([](const JpegInfo& x) {
  constexpr uint32_t MAX_DIM = 8192;
  constexpr uint64_t MAX_PIXELS = 40'000'000;
  if (x.status != JPEG_OK || x.width == 0 || x.height == 0 ||
      x.width > MAX_DIM || x.height > MAX_DIM ||
      x.width > MAX_PIXELS / x.height) {
    return ImageSpec::invalid();
  }
  return ImageSpec{x.width, x.height}; // copia validada en el host
});
if (!spec.valid()) return error();
render(copy_pixels_checked(t_info, spec), spec.width, spec.height);
```

  * El tipo *tainted* impide el uso accidental; **el validador expresa la política semántica**
  * La copia validada evita releer dimensiones que la biblioteca podría cambiar

---

# Congelamiento
  * Error de doble-fetch: consistencia entre valores a lo largo del tiempo
  * En el JPEG, ancho, alto, stride y puntero deben provenir de una instantánea coherente
  * Mecanismo: declarar que algunos datos sean una unidad "freezable". Probablemente un struct
  * No se permite leer de un tipo freezable a menos que esté congelado
  * Congelarlo copia una instantánea de todo fuera del sandbox
  * Ahora no hay posibilidad de condiciones de carrera de doble-fetch

---

# Callbacks - Implementación técnica
  * Tecnicidad: WebAssembly no permite importar funciones adicionales en tiempo de ejecución
    * Solución: función trampolín
    * Función única importada del renderer al sandbox
    * El trampolín toma como argumento el callback específico que quieres llamar, y puntero args
    * El trampolín verifica si el callback es uno válido, y si es así, lo llama

---

# Restricciones en tipos de función callback
  * Aplicadas por register_callback()
  * Deben tomar argumentos tainted
    * (también toma un argumento al objeto sandbox)
  * Deben retornar resultado tainted (o void)
  * Vida del objeto callback: con scope o unregister explícito

---

# ¿Qué tan bien aborda RLbox sus objetivos?
  * Para `foto.jpg`, el decoder puede seguir mintiendo o fallar; la meta es que no comprometa el renderer
  * Parece relativamente fácil de usar
  * 1-3 días-persona para sandboxear una biblioteca
  * Incremento de pocos cientos de LOCs: no tanto código
  * Suena como que los validadores son la parte más difícil: debe entender app, biblioteca

---

# ¿Cómo es el rendimiento después de sandboxear bibliotecas con RLbox?

  * Overheads de CPU modestos (3% para SFI/NaCl, 13% para sandboxing de procesos)
    * El overhead de sandboxing parece relativamente pequeño comparado con costos generales
    * Tienen algunas optimizaciones para mitigar overhead de cambio de contexto
  * Overheads de memoria modestos también
    * El compartir sandbox parece funcionar bien
    * Los sandboxes no son de larga duración, así que los costos de memoria no son a largo plazo

---

# Resumen
  * Caso de estudio: un JPEG compromete su decoder, pero no debería comprometer el renderer
  * Complicado interactuar de forma segura entre componentes aislados
  * Particularmente difícil adaptar interfaces existentes a límites de seguridad
  * RLBox aporta mecanismos: aislamiento, tipos *tainted*, copias verificadas y callbacks restringidos
  * **Lo que previene:** uso accidental de datos no confiables y cruces de memoria/tipos no explícitos
  * **Lo que no garantiza:** que un validador incompleto preserve las invariantes semánticas de la aplicación
  * Garantía final = sandbox confiable + mecanismos RLBox + validadores escritos correctamente
