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

# Defendiendo desbordamientos de búfer
  * Tema: defensas contra desbordamientos de búfer.
  * Los desbordamientos son una ruta de ataque popular, vale la pena entenderlos. Incluso ahora.
    * 5 desbordamientos de búfer de alto perfil en 2019 (ej., whatsapp):
    * https://securityboulevard.com/2019/11/5-buffer-overflow-vulnerabilities-in-popular-apps/
    * https://blog.zimperium.com/whatsapp-buffer-overflow-vulnerability-under-the-scope/
  * Ejemplo de evolución de defensa/ataque a lo largo del tiempo.
    * Ha sido muy valioso elevar la barra,
    * aunque las defensas aún no son perfectas.

---

# Resumen de la situación
  * Problema básico: código C con errores que escribe más allá del final del búfer/array.
  * La solución ideal es usar un lenguaje que haga cumplir los límites, ej.
    Python o Java. Es un gran esfuerzo re-entrenar programadores y
    re-escribir software, pero no imposible (ej. Microsoft y C#).
  * Pero C se usa para muchas aplicaciones y bibliotecas valiosas, así que no
    podemos abandonarlo, y a menudo no podemos evitar escribir nuevo código C. La definición
    de C hace difícil o imposible verificar límites automáticamente con precisión.
  * El programador perfecto verificaría límites 100% del tiempo, pero
    resulta que ningún programador es perfecto.
  * Así que necesitamos defensas que hagan los desbordamientos de búfer más difíciles
    de explotar, para programas C grandes y con errores que no entendemos.

---

# Empecemos con desbordamientos de búfer de stack "clásicos"
  * Los ataques tienen dos partes:
    * 1) escribir algunas instrucciones en el búfer del stack.
    * 2) sobrescribir el PC de retorno para apuntar a las instrucciones del atacante.
  * Las instrucciones del atacante pueden hacer cualquier cosa que la aplicación pueda hacer.
    * Por lo tanto es un ataque poderoso.

---

# Idea de defensa: O/S le dice al hardware que no ejecute en el stack
  * Ej. O/S establece el bit NX (No eXecute) de Intel en cada página del stack.
  * Esto previene la ejecución de instrucciones inyectadas en el stack.
  * P: ¿NX significa que no tenemos que preocuparnos por desbordamientos de búfer?
  * P: ¿NX es una pérdida de tiempo si no es perfecto?

---

# Idea de defensa: canarios de stack (ej., StackGuard, Stack Smashing Protector de gcc)
  * Detecta modificación del PC de retorno en el stack *antes* de que sea usado por RET.
  * El compilador genera código que empuja un valor "canario" en el stack al
    entrar a la función, hace pop y verifica el valor antes del retorno.
  * El canario se sienta entre variables y dirección de retorno, ej.:
                         |                  |
                         +------------------+
        entry %esp ----> |  dirección de retorno  |    ^
                         +------------------+    |
        new %ebp ------> |    %rbp guardado    |    |
                         +------------------+    |
                         |     CANARIO       |    | El desbordamiento va
                         +------------------+    | en esta dirección.
                         |     buf[127]     |    |
                         |       ...        |    |
                         |      buf[0]      |    |
                         +------------------+
                         |                  |
  * P: ¿qué valor deberíamos usar para el canario?
  * R: tal vez un número aleatorio, elegido al inicio del programa,
     almacenado en algún lugar.

---

# Idea de defensa: aleatorización del layout del espacio de direcciones (ASLR)
  * Colocar memoria del proceso en direcciones aleatorias.
  * El adversario no conoce la dirección precisa del stack, código, heap de la aplicación, ...
  * Requiere soporte del compilador para hacer todas las secciones reubicables.

---

# ¿Ya terminamos?
  * ¿Qué tipos de ataques podrían funcionar a pesar de ASLR?
  * ¿Qué tipos de ataques podrían funcionar a pesar de los canarios de stack?
  * Tal vez el atacante puede escribir o leer el valor canario secreto aleatorio de alguna manera.
  * Sobrescribir puntero de función en el stack antes del canario.
  * Sobrescribir alguna otra variable crucial en el stack, ej.
    bool ok = ...;
    char buf[128];
    gets(buf);
    if(ok){
      ...
    }
  * Desbordamiento de una variable global a la siguiente (muy parecido al stack).
  * Desbordamiento de búfer asignado en heap.

---

# ¿Son explotables los desbordamientos de búferes asignados en heap?
  * Importante porque el código moderno tiende a usar heap mucho.
  * foo(){
    char *p = malloc(16);
    gets(p);
  }
  * ¿Puede el atacante predecir qué está después de p en memoria?
    [diagrama de bloques malloc libres/usados]

---

# Resulta que ¡hay algunos ataques de heap bastante poderosos!
  * Aquí hay una versión simplificada de un ataque real.
  * Algunos malloc()s organizan bloques de memoria libre/usada así en
  lista doblemente enlazada:
  
            ^
	    |   data
	    |-- next
		prev --|
	    |-> ----   |
	    |   data   |
	    |-- next   |
		prev --|-|
	    |-> ---- <-| |
	    |   data     |
	    |-- next     |
		prev <---     

---

# Ataques de heap (cont.)
  * Malloc mantiene bloques libres en una lista doblemente enlazada.
    * En orden de dirección, para poder fusionar bloques pequeños adyacentes en uno grande.
  * Cuando se asigna un bloque libre, aquí está parte de lo que hace malloc():
    * b = elegir un bloque libre
    * b->next->prev = b->prev;
    * b->prev->next = b->next;
  * Si el atacante desborda un bloque malloc()ed, el atacante puede
    * modificar los punteros next y prev en el siguiente bloque.
  * Así: supongamos que el atacante escribe x y al inicio del siguiente bloque.
    * Llamar siguiente bloque b, entonces b->prev = x, b->next = y.
    * Supongamos que b es libre y resulta ser elegido por el siguiente malloc():
    * malloc() efectivamente ejecutará
      * *y = x
    * ¡Así escribiendo un valor elegido por el atacante a cualquier ubicación de memoria!

---

# Ataques de heap (cont.)
  * Si el atacante puede adivinar la dirección del PC de retorno guardado,
    * y puede adivinar la dirección del búfer siendo desbordado,
    * puede cargar instrucciones en el búfer y causar
    * que PC apunte a instrucciones inyectadas.
  * Similarmente para *cualquier* puntero de función con dirección predecible.
  * P: ¿cómo podría el atacante predecir tales direcciones?
  * Los ataques reales tienen que ser más complejos; busque en la web
    * Smashing the Heap for Fun and Profit, o Vudo Malloc Tricks.

---

# Las piezas que el atacante debe ensamblar
  * Encontrar un error de desbordamiento de búfer en la aplicación o biblioteca.
  * Encontrar una forma de hacer que el programa ejecute el código con errores
    * de una manera que cause que los bytes del atacante desborden el búfer.
  * Entender la implementación de malloc().
  * Encontrar un puntero de código y adivinar su dirección.
  * Adivinar la dirección del búfer, es decir, las instrucciones inyectadas del atacante.

---

# Estos ataques requieren esfuerzo y habilidad del atacante
  * El atacante debe entender casos extremos en la lógica de la aplicación, malloc(), salida del compilador.
  * Malas noticias para el defensor: pocos programadores de aplicaciones piensan en casos extremos.
  * Buenas noticias para el defensor: múltiples piezas frágiles en el rompecabezas del atacante.

---

# Un punto de alto nivel: si hay un error de desbordamiento de búfer, un atacante
  * lo suficientemente inteligente probablemente puede explotarlo. Más generalmente, muchos errores que
  * parecen inofensivos pueden ser convertidos en ventaja del atacante, tal vez en
  * combinación con otras fallas.

---

# Afortunadamente, como muestra el paper, ¡uno puede adaptar verificación automática
  * de límites de array a programas C existentes!
  * Si estamos dispuestos a modificar el compilador, recompilar aplicaciones,
  * y tal vez modificar las aplicaciones.

---

# Enfoque de verificación de límites #1: Punteros gordos
  * Directo aunque no muy práctico.
  * Cada puntero contiene inicio y fin del objeto de memoria original,
    * así como el valor actual del puntero:
    * +--------------+------------+-------------+
    * | inicio 32-bit | fin 32-bit | actual 32-bit |
    * +--------------+------------+-------------+
  * Modificar compilador para generar código que:
    * Establece p.start y p.end en malloc() y p = &x.
    * Durante la desreferencia, verifica start <= curr < end,
      * pánico si está fuera.
    * Durante la asignación de puntero, copia todo el puntero gordo.

---

# Punteros gordos (cont.)
  * Así:
    * p = malloc(4);  // malloc llena start y end.
    * *p = 1;         // start <= curr < end, así que la verificación tiene éxito.
    * q = p;          // copia puntero gordo a q.
    * q = q + 8;      // suma 8 a q.curr.
    * *q = 1;         // verificación falla, ya que curr >= end.
    * Pero:
      * q = q - 6;    
      * *q = 1;       // verificación tiene éxito.
    * La disposición start/end recuerda cuál era el objeto original.
    * Mucho código C requiere esto, ej. puntero a uno más allá del final.
  * Problema #1: Las verificaciones pueden ser costosas.
  * Problema #2: No muy compatible.
    * No puedes pasar un puntero gordo a una biblioteca no modificada.
    * Cambia el tamaño de las estructuras de datos.
    * Las actualizaciones a punteros gordos no son atómicas, porque abarcan
      * múltiples palabras. Las aplicaciones con hilos y interrupciones/señales
      * asíncronas pueden fallar debido a estas actualizaciones no atómicas.

---

# Enfoque de verificación de límites #2: Mantener información de límites en una tabla separada
  * Para que no tengamos que cambiar la representación del puntero.
  * Baggy Bounds (y otros) hacen esto.

  * Idea básica: Para cada objeto asignado, almacenar inicio y tamaño del objeto.
    * malloc() o código generado por el compilador mantiene la tabla.
    * ej. tabla[p] = ???
  * Establecer un bit OOB (Out Of Bounds) en el puntero si sale de su objeto original.
  * Para cada puntero, el compilador genera código para dos tipos de operaciones
    * (El compilador "interpone" en ellas)
    * Aritmética de punteros: char *q = p + 256;
      * Debe detectar si el puntero ya no apunta al objeto original, y establecer el bit OOB.
      * comparar q con tabla[p]
    * Desreferencia de punteros: char ch = *q;
      * Debe hacer pánico si el bit OOB de q está establecido.

---

# Desafíos de Baggy Bounds
  * Desafío: necesitamos indexar tabla[] con tanto p como ej. p + 10,
    * es decir, si p apunta al medio del objeto en lugar del inicio.

  * Desafío: si la aritmética modifica p para apuntar a un objeto diferente,
    * tenemos que saber NO mirar tabla[p] en usos futuros de p.
    * podemos usar el bit OOB en el puntero para esto.

  * Desafío: si la aritmética modifica p para volver a su objeto original,
    * debemos detectar eso y limpiar OOB, ya que el puntero ahora es válido de nuevo.
    * en particular, ¿cómo saber cuál era el objeto original?

  * Desafío: prevenir que la tabla sea enorme, o costosa de acceder.

---

# ¿Por qué rastrear OOB? Porque puede pasar en programas C legales; por ejemplo:
  * size_t array_size = ...;
  * int *arr = malloc(array_size * sizeof(*arr));
  * int *end = arr + array_size; // end es un puntero fuera de límites. si se desreferencia, baggy lanzará un error

  * for (int *i = end - 1; i >= arr; i--) {
    * // bucle hacia atrás sobre el array
  * }

---

# Baggy Bounds usa el truco de potencia de dos para optimizar la tabla de límites
  * La entrada de la tabla mantiene solo el tamaño, como log base 2, para caber en un byte: tabla más pequeña.
    * 2^tabla[i] produce tamaño, así que ej. 20 significa 1 megabyte.
  * Asignar solo tamaños de potencia de dos en límites de potencia de dos.
    * El código compilado puede calcular el inicio de un objeto desde el puntero y la entrada tabla[].
    * Solo limpiar los bits bajos log2(tamaño).
  * Optimización: cada entrada de tabla cubre slot_size=16 bytes, así que la tabla no es enorme.
    * Objeto de 64 bytes en dirección x usa 4 entradas de tabla empezando en tabla[x/16].

---

# Baggy Bounds usa truco de memoria virtual para manejar punteros fuera de límites
  * Establecer el bit más significativo en un puntero OOB.
    * Marcar páginas en la mitad superior del espacio de direcciones como inaccesibles.
    * ¡El compilador no tiene que insertar instrucciones de verificación para desreferencias!
  * Marca OOB significa "ptr está dentro de slot_size/2 del objeto original".
    * El código de aritmética puede reconstruir cuál era el objeto original.
    * Limpiar OOB si ptr se mueve de vuelta al objeto original.
    * Pánico si la aritmética trata de mover OOB más de slot_size/2 lejos.

---

# Dado objeto p, aquí está cómo encontrar tamaño e inicio del objeto
  * p apunta, para slot_size=16:
  * tamaño = 1 << tabla[p >> 4]; // 4 es el número de bits en 16.
  * inicio = p & ~(tamaño - 1);   // limpia los bits bajos log2(tamaño).

---

# Ejemplo:
  * p = malloc(25); // redondea hacia arriba a 32, establece dos entradas tabla[].
  * q = p + 10;     // aritmética; lee tabla[], ve OK.
  * r = p + 35;     // aritmética; no OK; establece OOB.
  * *r = 1;         // sistema VM fuerza crash.
  * s = p + 35;     // establece OOB.
  * s = s - 20;     // aritmética limpia OOB.
  * *s = 1;         // OK
  * t = p + 100;    // aritmética fuerza crash: no dentro de slot_size/2.

---

# ¿Cuál es la respuesta al problema de tarea?
  * char *p = malloc(256);
  * char *q = p + 256;
  * char ch = *q;  //¿Esto lanza una excepción?

---

# Así que:
  * Baggy Bounds puede ser aplicado a programas C grandes y con errores existentes,
  * y automáticamente detiene el programa si un atacante trata de
  * explotar un desbordamiento de búfer. ¡Esto es una gran victoria!

---

# ¿Puede Baggy Bounds alguna vez hacer pánico cuando el código es realmente legal?
  * Considerar código C que convierte punteros a enteros, y luego los compara.
  * Puede romperse debido al bit OOB.

---

# ¿Qué errores de desbordamiento podría Baggy Bounds no detectar (Sección 7)?
  * Desbordamiento de array dentro de una struct malloc()'d más grande.
  * Convertir puntero a int, modificar, convertir de vuelta a puntero.
  * La aplicación podría implementar su propio asignador.
  * Punteros colgantes a memoria re-asignada.

---

# Baggy Bounds, y desbordamientos en general, ilustran un patrón general:
  * Los defensores habitualmente cometen un tipo específico de error.
  * Los atacantes encuentran una forma de explotar ese error.
  * Los defensores construyen una herramienta para detectar/arreglar el error.

---

# Referencias
  * Atacar ciegamente un servidor de red
    * https://css.csail.mit.edu/6.858/2014/readings/brop.pdf
  * Vudo Malloc Tricks
    * http://phrack.org/issues/57/8.html
  * Smashing the heap for fun and profit
    * http://doc.bughunter.net/buffer-overflow/heap-corruption.html
  * Mucha investigación en esquemas de seguimiento: ej.,
    * https://www.comp.nus.edu.sg/~gregory/papers/cc16lowfatptrs.pdf 