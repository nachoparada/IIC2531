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
    position: absolute;
    margin-top:0px
  }

  img[alt~="align-center"] {
    position: absolute;
    left: 50%;
    transform: translateX(-50%);
  }

  .terminal-commands {
    text-align: right;
    margin-left: 400px;
  }

  ul ul li, ol ol li {
    color: #666666;
    font-size: 0.9em;
  }

  ul ul ul li, ol ol ol li {
    color: #666666;
    font-style: italic;
    font-size: 0.8em;
  }

  ul ul, ol ol {
    opacity: 0.8;
  }

---

# Seguridad de la cadena de suministro

## Open source, dependencias y ataques modernos

<!--
Notas:
Esta clase usa la transcripción como guía para entender qué es una cadena de suministro, por qué el problema se volvió urgente, qué defensas funcionan parcialmente y qué nos enseñan ataques recientes como XZ, GitHub Actions y npm.
-->

---

# Objetivo de hoy

- Definir **supply chain security**
- Entender por qué open source cambia la escala
- Revisar defensas prácticas
- Estudiar ataques reales
- Discutir qué debería cambiar

<!--
Notas:
Abrir con la idea central: casi ningún software moderno es completamente propio. Compilamos, instalamos y ejecutamos código que pasó por muchas manos, máquinas, repositorios, paquetes, scripts de build y sistemas de CI. La seguridad de cadena de suministro pregunta qué pasa si el atacante no ataca nuestra aplicación directamente, sino algo que usamos para construirla o distribuirla.
-->

---

# ¿Qué es un ataque de cadena de suministro?

- Alteración maliciosa de software confiable
  - Un ataque de cadena de suministro modifica software confiable antes de entregarlo
- Ocurre **antes de la entrega**
- El usuario recibe algo que parece legítimo
- La confianza se usa como vector

<!--
Notas:
La distinción importante en la transcripción es "antes de la entrega". Si alguien entra a mi máquina y cambia un binario, eso es un ataque directo. Si alguien compromete al proveedor, el repositorio, el sistema de build o el paquete que yo descargo antes de que llegue a mí, eso es cadena de suministro. La víctima instala o ejecuta el software porque viene por un canal aparentemente confiable.
-->

---

# Ejemplos

- Hardware manipulado
  - [Crypto AG](https://www.washingtonpost.com/graphics/2020/world/national-security/cia-crypto-encryption-machines-espionage/)
- Firmware con puertas traseras
  - [XcodeGhost](https://unit42.paloaltonetworks.com/novel-malware-xcodeghost-modifies-xcode-infects-apple-ios-apps-and-hits-app-store/)
- Algoritmos o estándares débiles
  - [Juniper / Dual EC](https://www.wired.com/2015/12/researchers-solve-the-juniper-mystery-and-they-say-its-partially-the-nsas-fault/)
- Software modificado antes de distribuirse
  - [SolarWinds Orion](https://www.axios.com/2020/12/14/russian-hack-cyber-emergency)

<!--
Notas:
Estos ejemplos físicos e históricos muestran que la idea no es nueva. Crypto AG muestra una cadena de suministro física: gobiernos compraban equipos de cifrado supuestamente confiables, pero la empresa estaba controlada por inteligencia estadounidense y alemana, que podía leer muchas comunicaciones. XcodeGhost es un ejemplo de herramienta de desarrollo comprometida: desarrolladores descargaron copias falsas de Xcode y las apps compiladas con ellas heredaron el malware. El caso Juniper / Dual EC muestra cómo un estándar o generador criptográfico débil puede abrir una puerta trasera en productos de red, incluso antes de hablar de código de aplicación. SolarWinds Orion muestra el caso moderno clásico: una actualización firmada y legítima de software empresarial distribuyó malware a clientes que confiaban en el proveedor. El punto pedagógico es que la cadena de suministro existe siempre que recibimos un artefacto de alguien más; en software, el problema se intensifica porque la distribución es masiva, rápida y automatizada.
-->


---

# Software supply chain

- Código fuente
- Dependencias
- Herramientas de build
- Repositorios y registries
- CI/CD
- Binarios y contenedores

<!--
Notas:
Conviene presentar la cadena como todos los lugares donde se puede introducir un ataque o una vulnerabilidad. No es una línea simple. Incluye quién escribe código, quién revisa, qué dependencia se descarga, qué compilador se usa, qué runner ejecuta el build, qué registry aloja el paquete y cómo se instala finalmente.
-->
---

# Vulnerabilidad vs. ataque

- Vulnerabilidad:
  - condición explotable en una dependencia
- Ataque:
  - alguien la usa o introduce código malicioso
- Supply chain security cubre ambos

<!--
Notas:
No todo problema de supply chain es un ataque. Log4Shell, por ejemplo, se puede pensar como una vulnerabilidad de cadena de suministro porque una biblioteca open source muy usada expuso a muchas aplicaciones. Un ataque, en cambio, implica alteración o explotación maliciosa. La clase debe separar estas ideas para no mezclar bugs accidentales con campañas adversariales.
-->
---

# Ataques en Open source

- Un ataque de cadena de suministro de software open-source es la
alteración maliciosa de un componente open-source confiable que luego se usa en un programa confiable.

---

# Ejemplos

- [event-stream](https://snyk.io/blog/malicious-code-found-in-npm-package-event-stream/)
  - 2 meses no detectado
  - librería open source ampliamente usada
  - el developer estaba cansado

<!--
Notas:
event-stream era una librería npm open source muy usada. En 2018, el maintainer cedió el paquete a otra persona porque ya no tenía tiempo para mantenerlo; esa persona agregó la dependencia maliciosa flatmap-stream, que incluía código diseñado para robar criptomonedas desde aplicaciones específicas como Copay. El ataque pasó inadvertido por cerca de dos meses, lo que lo convierte en un ejemplo claro de cómo la confianza en maintainers y dependencias transitivas puede convertirse en vector de ataque.
-->

---

# Vulnerabilidades en Open source

- Una vulnerabilidad de cadena de suministro open source es
una debilidad explotable en un paquete de software confiable
causada por uno de sus componentes open source.

---

# Ejemplos

- [NSO zero-click iMessage exploit](https://projectzero.google/2021/12/a-deep-dive-into-nso-zero-click.html)
- [log4j Minecraft attack](https://www.minecraft.net/en-us/article/important-message--security-vulnerability-java-edition)

<!--
Notas:
NSO/FORCEDENTRY muestra una vulnerabilidad de cadena de suministro muy indirecta: en septiembre de 2021, Apple corrigió un bug que permitía tomar control de un iPhone sin clicks mediante un iMessage con una imagen especialmente construida. La cadena descrita en la transcripción es larga: el archivo decía ser un GIF, pero el pipeline de iMessage lo pasaba a un procesador general de imágenes; en realidad era un PDF; el PDF contenía una imagen JBIG2; Apple usaba el decodificador JBIG2 de Xpdf, open source y escrito en C; el decoder no validaba correctamente tablas Huffman/datos codificados; eso permitía operaciones bit a bit sobre memoria en offsets controlados por el atacante; con esas operaciones, NSO implementó una CPU virtual completa, escaneó memoria del proceso, escapó del sandbox de iMessage y terminó tomando control del teléfono. La lección no es que Xpdf fuera malicioso, sino que una vulnerabilidad en un componente open source usado dentro de software confiable puede habilitar una explotación extremadamente sofisticada.

Log4j/Minecraft muestra el otro tipo de vulnerabilidad: jugadores descubrieron que mensajes de chat especialmente construidos podían hacer que Minecraft descargara y ejecutara código Java desde servidores remotos. El problema real estaba en Log4j, una librería open source muy usada, y en una funcionalidad de logging que podía resolver referencias externas. Minecraft no era open source, pero dependía de Log4j; por eso el incidente es un buen ejemplo de vulnerabilidad de cadena de suministro en software cerrado causada por un componente open source.
-->

---

# Tres tareas

1. Entender la cadena
2. Hacerla más fuerte contra ataques
3. Monitorearla para encontrar vulnerabilidades

<!--
Notas:
La estructura de la charla se puede resumir en estas tres tareas. Primero hay que mapear la cadena: qué dependencias, qué build, qué credenciales, qué máquinas. Luego hay que endurecer los puntos de ataque. Finalmente, hay que buscar vulnerabilidades antes de que alguien las explote. Sin inventario, las otras dos tareas son casi imposibles.
-->

---

# Entender la cadena

---


# Open source supply chain

- Dependencias públicas
- Maintainers voluntarios
- Repositorios distribuidos
- Versiones descargadas automáticamente
- Ecosistemas enormes: npm, Go, Rust, Python

<!--
Notas:
El open source aumenta muchísimo la superficie. Un paquete pequeño puede terminar siendo usado por miles o millones de sistemas a través de dependencias transitivas. Además, muchos proyectos críticos son mantenidos por pocas personas, a veces en su tiempo libre. Eso crea un desbalance: muchísimo impacto potencial con pocos recursos de mantención.
-->

---

# La cadena no es una cadena

```text
Proyecto
  ├─ dependencia A
  │   ├─ dependencia C
  │   └─ herramienta D
  └─ dependencia B
      └─ workflow E
```

- Es un grafo
- Cambia con cada versión
- Cada nodo es una posible entrada
- k8s.io/kubernetes usa 3000 paquetes aprox

<!--
Notas:
La palabra "cadena" engaña: se parece más a un fractal o grafo de dependencias. En un proyecto real, hay dependencias de runtime, dependencias de build, herramientas que generan código, acciones de CI, imágenes base y servicios externos. Cada borde del grafo implica confianza.
-->

---

# Problema base

- No controlamos todo lo que ejecutamos
- Tampoco sabemos todo lo que ejecutamos
- Muchas decisiones son automáticas
- Los atacantes buscan el eslabón barato

<!--
Notas:
El atacante no necesita comprometer el objetivo más protegido. Puede comprometer una dependencia pequeña, un maintainer cansado, una cuenta npm, una GitHub Action, un token o un script de release. En seguridad de sistemas esto es clásico: se ataca el camino de menor resistencia. La diferencia es que en supply chain ese camino puede estar muy lejos del código que creemos estar auditando.
-->

---

# Fortalecer la cadena


---

# C/C++: unsafe at any speed

> Cruzar una autopista de 8 pistas con los ojos vendados

- Undefined behavior
- Use-after-free
- Out-of-bounds
- ¿Por qué? Performance por sobre seguridad

<!--
Notas:
Una buena forma de introducir este punto es una cita de John Regehr: escribir código crítico en C o C++ es como cruzar una autopista de ocho pistas con los ojos vendados. Es una buena pausa retórica para discutir por qué tantas vulnerabilidades de supply chain son vulnerabilidades de memoria; la conclusión seria es que ya no podemos aceptar performance como excusa universal.
-->
---

# Fuzzing

- Probar entradas generadas automáticamente
- Buscar crashes y comportamientos raros
- Especialmente útil con parsers y formatos
- Funciona porque corre millones de pruebas

<!--
Notas:
Fuzzing empuja el espacio de entradas para encontrar caminos inesperados. Se puede describir casi como golpear el programa con una piedra muy rápido: no es elegante, pero funciona. OSS-Fuzz ha encontrado miles de vulnerabilidades y decenas de miles de bugs. Esto muestra una defensa realista: no garantiza ausencia de bugs, pero sube el costo para atacantes.
-->

---

# Reducir vulnerabilidades

- Lenguajes seguros en memoria
  - Go
  - Rust
- Fuzzing continuo
- Revisión de dependencias
- Monitoreo de CVEs
- Actualizaciones con criterio

<!--
Notas:
Conviene criticar especialmente C y C++ por undefined behavior, uso de memoria inseguro y falta de bounds checks. El punto no es demonizar lenguajes, sino mostrar que muchas vulnerabilidades explotables nacen de decisiones técnicas antiguas. Fuzzing, como OSS-Fuzz y syzkaller, ayuda a encontrar bugs, aunque su éxito también muestra lo frágil que sigue siendo mucho software.
-->


---

# Reducir vectores de ataque

- Firmas criptográficas
- Checksums inmutables
- Builds reproducibles
- Máquinas de build dedicadas
- Revisión por más de una persona

<!--
Notas:
Aquí cambiamos desde vulnerabilidades a alteración maliciosa. Si el problema es que alguien cambia el software antes de que llegue al usuario, necesitamos mecanismos que hagan detectable o imposible esa modificación. Firmas, checksums, logs, builds reproducibles y separación de permisos son formas de reducir dónde puede intervenir el atacante.
-->

---

# Firmas y checksums

- Verifican que el artefacto no cambió
- Eliminan proxies y mirrors como puntos de ataque
- Crean un problema: ¿qué llave confiamos?
- Requieren distribución de confianza

<!--
Notas:
Una firma o checksum no dice que el software sea bueno. Dice que el artefacto que recibí corresponde a cierto valor esperado o fue firmado por cierta llave. Eso ya es enorme: evita que un proxy, mirror o servidor intermedio cambie el paquete sin ser detectado. Pero queda el problema de cómo sabemos qué llave o checksum corresponde.
-->

---

# Caso Go: checksum database

- SHA-256 de versiones públicas
- Entradas firmadas por el servidor
- Llave pública embebida en `go`
- Modelo trust-on-first-use
- Versiones quedan inmutables

<!--
Notas:
El Go checksum database registra el checksum de cada versión pública de paquetes Go. La primera vez que observa una versión, fija qué significa esa versión. Eso no prueba que el código sea seguro, pero evita que el significado de `v1.2.3` cambie mañana. Esta inmutabilidad es base para análisis, auditoría y bases de vulnerabilidades.
-->

---

# Builds reproducibles

- Mismo código + mismas herramientas
- Mismo output bit a bit
- Permite verificar binarios publicados
- Reduce confianza en una sola máquina

<!--
Notas:
Los builds reproducibles permiten que terceros reconstruyan un artefacto y comparen el resultado. Si dos máquinas distintas producen el mismo binario, es menos probable que una máquina de build comprometida haya introducido algo. Go consiguió builds reproducibles para sus distribuciones, incluso entre sistemas operativos y arquitecturas.
-->

---

# Control de cambios

- Nadie debería tener poder unilateral
  - Autor + reviewer
- Acceso privilegiado desde dispositivos confiables
- Builds y producción con autorización fuerte

<!--
Notas:
Google usa reglas de dos personas para código de producción y sistemas productivos. La idea es que una cuenta o laptop comprometida no baste para cambiar el sistema. Para open source, esto puede traducirse en revisión obligatoria, maintainer quorum, protección de ramas y credenciales resistentes a phishing.
-->

---

# Monitorear la cadena

---

# Monitorear y responder

- Inventario de dependencias
- Saber quién usa qué versión
- Alertas ante CVEs o ataques
- Capacidad de rotar y reconstruir

<!--
Notas:
No basta con prevenir. Cuando aparece un ataque o vulnerabilidad, necesitamos saber si estamos afectados. Eso requiere inventario: qué paquetes, versiones, imágenes, runners, tokens y builds existen. Sin inventario, la respuesta se vuelve artesanal y lenta.
-->

---

# Ataques básicos

- Cuenta de maintainer comprometida
  - Roban credenciales y publican como alguien confiable
- Publicación de versión maliciosa
  - Una release legítima incluye código introducido por el atacante
- Typosquatting
  - Paquetes con nombres casi iguales capturan instalaciones por error
- Dependencias transitivas
  - El ataque entra por una librería indirecta que nadie revisa
- Tags reescritos
  - Un nombre de versión apunta después a otro código

<!--
Notas:
Antes del caso XZ, conviene recordar ataques más comunes. No todos requieren años de infiltración ni una puerta trasera sofisticada. Una cuenta de maintainer comprometida permite actuar con identidad legítima; una versión maliciosa aprovecha la confianza normal en el proceso de release; el typosquatting espera errores humanos como `request` en vez de `requests`; una dependencia transitiva afecta a proyectos que ni siquiera saben que la usan; y un tag reescrito rompe la suposición de que un nombre de versión apunta siempre al mismo código. La idea central es que la cadena de suministro amplifica acciones pequeñas porque los ecosistemas instalan, actualizan y confían automáticamente.
-->

---

# Compiler backdoor

- El código fuente puede estar limpio
- El binario del compilador inserta la trampa
- Compilar `login` o el kernel agrega la puerta trasera
- Recompilar el compilador vuelve a infectarlo
- Builds reproducibles permiten detectarlo

<!--
Notas:
Este es el ataque de "Reflections on Trusting Trust" de Ken Thompson, anticipado por el informe de la Fuerza Aérea sobre Multics. La idea es inquietante: el atacante no necesita dejar la puerta trasera en el código fuente visible. Puede modificar el compilador binario para que, cuando compile un programa sensible como `login` o un system call del kernel, inserte la trampa automáticamente. Luego el compilador también se modifica a sí mismo al recompilarse, de modo que el código fuente vuelve a verse limpio y la puerta trasera persiste. La defensa moderna es comparar builds reproducibles hechos con compiladores independientes: si ambos llegan al mismo binario, o todo está bien, o ambos compiladores estaban comprometidos exactamente de la misma forma.
-->

---

# Domain name resurrection

- Un dominio expira
- El atacante lo compra
- Reaparece como proveedor "confiable"
- Viejas referencias vuelven a resolver
- Github también lo permite!!
  - Aunque ha agregado limitaciones

<!--
Notas:
La diapositiva del PDF tiene una formulación muy buena: en vez de falsificar papelería, compra dominios expirados. Es una forma simple de explicar que la confianza puede quedar pegada a nombres, URLs, emails o dominios que ya no controla la persona original. Para supply chain, esto afecta dependencias, documentación, instaladores, correos de maintainer y endpoints de actualización.
-->

---

# XZ: el ataque paciente

- Objetivo final en 2024: OpenSSH
- Camino indirecto: `sshd → systemd → liblzma → xz`
- Proyecto mantenido por Lasse Collin desde 2005
- Primera señal: patch inocuo de Jia Tan en oct. 2021
- Backdoor activa: releases 5.6.0/5.6.1 en feb.-mar. 2024

<!--
Notas:
El ataque XZ es el caso central de la charla. Atacar OpenSSH directamente es difícil porque es un proyecto muy observado. Pero en muchas distribuciones Linux, OpenSSH termina cargando libsystemd, y libsystemd dependía de liblzma, que viene de xz-utils. Esto conecta con el chiste de xkcd 2347: infraestructura crítica sostenida por una persona; en este caso, Lasse Collin llevaba desde 2005 manteniendo xz como proyecto de hobby. La primera acción conocida de Jia Tan fue un patch inocuo el 29 de octubre de 2021. La backdoor recién aparece en releases de febrero-marzo de 2024, casi dos años y medio después.
-->

---

# XZ: toma de confianza

- Oct. 2021: Jia Tan envía un patch inocuo
- Nov. 2021 / abr. 2022: más parches útiles
- Abr.-jun. 2022: identidades falsas presionan a Lasse
- Jun. 2022: Jia Tan ya parece casi co-maintainer
- Nov. 2022: README y email lo reconocen oficialmente

<!--
Notas:
El ataque no empezó con malware. Empezó el 29 de octubre de 2021 con una contribución inocua que fue ignorada, siguió con otro patch en noviembre de 2021 y con trabajo más técnico en abril de 2022. Desde abril de 2022 aparecen cuentas falsas como Gigar Kumar y Dennis Ens presionando al maintainer por la lentitud del proyecto y empujando la narrativa de que "se necesita un nuevo maintainer". En junio de 2022, Lasse Collin ya decía que Jia Tan era prácticamente co-maintainer. En noviembre de 2022, el README y el email del proyecto lo reflejan como co-maintainer. La parte técnica vino después; primero se comprometió el proceso humano.
-->

---

# XZ: la puerta trasera

- Feb. 2024: nuevos test files "corruptos"
- 5.6.0: el payload aparece en tarballs de release
- Mar. 4 2024: Red Hat ve fallas bajo Valgrind
- Mar. 9 2024: 5.6.1 actualiza la backdoor
- Objetivo técnico: interceptar RSA en `sshd`

<!--
Notas:
En febrero de 2024, Jia Tan agrega archivos de prueba nuevos: uno válido y otro supuestamente corrupto de una forma que conviene testear en una librería de compresión. En la release 5.6.0, el tarball contiene un cambio en scripts generados que no aparece igual en el repo: durante `configure` y `make`, esos test files reconstruyen scripts y un objeto malicioso que reemplaza código legítimo. El 4 de marzo de 2024, Red Hat observa una falla de `get_cpuid` bajo Valgrind. Cuatro días después llega un supuesto fix, y el 9 de marzo se etiqueta 5.6.1 con test files actualizados. El objetivo final era interceptar `RSA_public_decrypt` dentro de `sshd`, aprovechando que liblzma estaba cargada en el proceso.
-->

---

# XZ: por qué casi funcionó

- 2021-2024: dos años y medio ganando confianza
- Proyecto real y ampliamente usado
- Maintainer legítimo agotado
- Payload solo aparece en tarballs de release
- Mar. 2024: ventana corta antes de que systemd removiera la dependencia

<!--
Notas:
El caso es inquietante porque muchos controles normales habrían fallado. El código malicioso no estaba simplemente como una función llamada `backdoor`: estaba repartido entre test files, scripts generados y artefactos de release. Además, para 2024 el atacante ya tenía reputación dentro del proyecto. Había otra presión temporal: los desarrolladores de systemd estaban avanzando en remover dependencias de compresión como liblzma, lo que cerraba la ventana para que la backdoor llegara a `sshd` por ese camino. Esto muestra que "revisar el diff" no siempre basta.
-->

---

# XZ: cómo se descubrió

- Mar. 4 2024: Red Hat ve fallas bajo Valgrind
- Mar. 27 2024: Debian acepta 5.6.1
- Mar. 28 2024: Ubuntu ya no lo toma desde Debian
- Mar. 29 2024: Andres Freund publica el hallazgo
- Señal clave: `ssh` consumía más CPU de lo esperado

<!--
Notas:
La detección tuvo varias señales con fechas muy comprimidas. El 4 de marzo de 2024 aparece una falla bajo Valgrind que obliga al atacante a ajustar los archivos maliciosos. El 27 de marzo, Debian acepta 5.6.1 después de presión de sock puppets. El 28 de marzo, Ubuntu intenta evaluar la actualización, pero ya ve que fue retirada desde Debian. El 29 de marzo, Andres Freund publica que el repositorio upstream y los tarballs de xz estaban backdooreados. Freund estaba haciendo benchmarks de PostgreSQL y notó que `ssh` consumía más CPU de lo esperado. La lección no es "confiemos en la suerte"; la lección es que observabilidad, reproducibilidad y gente curiosa importan muchísimo.
-->

---

# XZ: lecciones

- Lo humano es parte del sistema
- Maintainers críticos necesitan apoyo
- Releases deben ser reproducibles
- Tarballs no deben diferir del repo
- La economía favorece al atacante

<!--
Notas:
El caso enfatiza el desbalance económico: un ataque de años puede costar mucho, pero una backdoor global en OpenSSH valdría muchísimo más. También hay una lección comunitaria: proyectos críticos mantenidos por una persona agotada son infraestructura, aunque no se financien como tal.
-->
---

# Cierre

- El atacante no necesita entrar por la puerta principal
- La confianza transitiva es superficie de ataque
- Buenas defensas existen, pero no son universales
- La industria ya mejoró antes: HTTPS, passkeys
- Supply chain será un tema central

<!--
Notas:
Cerrar con una nota relativamente optimista: cuando la industria se coordina, puede mejorar. HTTPS pasó de ser minoritario a ser estándar. Passkeys hacen posible reducir phishing. La pregunta es si los ataques recientes serán suficiente presión para mejorar registries, CI/CD, credenciales y defaults de instalación. La clase debería cerrar con esa tensión: el problema es grave, pero no insoluble.
-->

---

# Ataque común: phishing npm

- Email parecido al legítimo
- Dominio casi correcto
- Maintainer entrega credenciales
- Publicación maliciosa inmediata

<!--
Notas:
Después de XZ, volver a ataques más comunes. Un maintainer puede recibir un correo falso de npm y perder la cuenta. No hace falta ser una operación sofisticada. El punto crítico es que registries como npm son objetivos enormes y deberían exigir credenciales resistentes a phishing, como passkeys, especialmente para cuentas con paquetes populares.
-->

---

# GitHub Actions

- CI como parte de la cadena
- Ejecuta código en runners
- Tiene tokens y secretos
- Puede publicar paquetes
- Puede comentar, firmar, desplegar

<!--
Notas:
GitHub Actions no es solo automatización conveniente. Es una pieza privilegiada de la supply chain. Si un workflow puede acceder a secretos o publicar releases, entonces comprometer ese workflow equivale a comprometer el proyecto. El diseño de permisos y triggers es central.
-->

---

# `pull_request` vs `pull_request_target`

- `pull_request`
  - código no confiable
  - sin secretos
- `pull_request_target`
  - permisos del repo
  - peligroso si ejecuta código del PR

<!--
Notas:
La diferencia es clave. `pull_request` corre en un entorno más limitado porque el código viene de cualquiera. `pull_request_target` fue creado para permitir acciones con más permisos, por ejemplo comentar en el PR, pero se vuelve peligroso cuando el workflow termina ejecutando código controlado por el PR. El nombre no ayuda: no comunica lo peligroso que es.
-->

---

# Patrón de falla

1. PR externo cambia un script
2. Workflow privilegiado hace checkout del PR
3. Script se ejecuta con secretos
4. Token se exfiltra
5. El atacante encadena accesos

<!--
Notas:
Un caso típico muestra cómo un workflow termina ejecutando `mvnw` del PR con un token personal en el ambiente. Eso permite robar el token y moverse a otros repositorios donde el usuario tiene acceso. La cadena no termina en el primer repo: los secretos robados abren nuevas puertas.
-->

---

# Inyección en workflows

```yaml
run: echo "${{ github.event.pull_request.title }}"
```

- Sustitución de texto
- Shell interpreta caracteres especiales
- El título del PR puede ejecutar código
- Documentación no es una defensa suficiente

<!--
Notas:
Otro patrón es inyección por interpolación. Si el título o cuerpo de un PR se inserta en un comando shell sin escape correcto, el atacante puede poner sintaxis de shell en el texto. GitHub advierte sobre esto en documentación, pero el punto crítico es que la plataforma conoce el contexto y podría ofrecer mecanismos más seguros por diseño.
-->

---

# Caso NX

- Workflow inseguro para validar títulos
- Vulnerabilidad queda en ramas viejas
- Atacante roba tokens GitHub y npm
- Publica versiones maliciosas
- Malware busca secretos locales

<!--
Notas:
El caso NX muestra cómo una mala configuración puede sobrevivir incluso después de revertirse en main. El workflow vulnerable quedó en otra rama, y un atacante envió un PR contra esa rama. Luego robó tokens y publicó versiones maliciosas del build tool. El malware incluso usaba agentes de IA locales para buscar archivos sensibles.
-->

---

# Supply chain worm

- Paquete malicioso roba credenciales
- Usa credenciales para publicar más paquetes
- Se propaga por permisos de maintainers
- Intenta persistencia en GitHub runners
- Exfiltra secretos y repos privados

<!--
Notas:
La evolución natural de estos ataques es un worm: código que al instalarse busca qué paquetes puede modificar la víctima y los backdoorea también. Existen variantes que se propagaron por npm usando credenciales robadas, GitHub runners persistentes y repos públicos creados en cuentas de víctimas para exfiltrar datos.
-->

---

# Seguridad que instala malware

- Ataques contra scanners y herramientas de seguridad
- Trivy, Checkmarx, paquetes populares
- Mayor impacto porque corren con privilegios
- Ironía: la defensa se vuelve vector

<!--
Notas:
Una parte especialmente grave es el ataque contra herramientas de seguridad. Si un scanner de vulnerabilidades o un paquete muy común se compromete, puede ejecutarse en muchos entornos privilegiados. El atacante busca herramientas que naturalmente tienen acceso a código, imágenes, secretos o infraestructura.
-->

---

# Decisiones de producto importan

- `pull_request_target` expone una trampa
- Ataques tan comunes que tienen nombre: **pwn requests**
- npm instala "lo último" por defecto
- Tags o versiones mutables amplifican daño
- Tokens de un factor contradicen el 2FA

<!--
Notas:
Estos ataques no se deben tratar como accidentes inevitables. Muchas causas son decisiones de plataforma. Si npm instala siempre la última versión, el malware se distribuye inmediatamente. Si un tag puede reescribirse, una versión deja de tener significado estable. Si los tokens actúan como contraseñas sin segundo factor, el 2FA es en parte teatro. El PDF enfatiza que los exploits de `pull_request_target` son tan comunes que la comunidad ya los llama pwn requests.
-->

---

# Qué deberíamos exigir

- Versiones inmutables
- Passkeys para maintainers
- Tokens limitados y de corta duración
- Workflows seguros por defecto
- Secrets inaccesibles a código no confiable
- Builds reproducibles

<!--
Notas:
Esta diapositiva debe convertirse en discusión. Algunas medidas son técnicas: inmutabilidad, reproducibilidad, firmas, permisos mínimos. Otras son de plataforma: defaults seguros, mejores APIs para workflows, bloqueo de patrones peligrosos. Otras son humanas: apoyo a maintainers y procesos de revisión.
-->

---

# ¿Qué puede hacer un equipo?

- Inventariar dependencias y workflows
- Pinnear versiones y revisar upgrades
- Usar proxies con retraso para npm
- Auditar GitHub Actions
- Rotar secretos y reducir permisos
- Separar build, release y deploy

<!--
Notas:
Como prácticas concretas, conviene mencionar passkeys, proxies que retrasan nuevas versiones para evitar ser los primeros en recibir malware, y apoyo de empresas de seguridad. Para una organización normal, lo importante es reducir exposición automática y asegurarse de poder responder cuando algo se descubre.
-->



