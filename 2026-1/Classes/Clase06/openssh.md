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

# Separación de Privilegios en OpenSSH

Comenzamos un nuevo módulo: **casos de estudio de diseño de sistemas para seguridad**.

* Gran tema en muchos de estos casos de estudio: **separación de privilegios**.
* La separación de privilegios también es el foco del laboratorio 2.

---

# El Problema: Bugs Explotables en Software

* El software es complejo → bugs → exploits.
* ¿Qué hacer al respecto?

**Plan A:** Encontrarlos, arreglarlos, evitar crear nuevos.
  * Hablaremos de varias técnicas así en el próximo módulo.
  * Mucho progreso aquí, pero para sistemas grandes, no es suficiente.

---

# Ejemplo: OpenSSH (antes de separación de privilegios)

* Proceso que escucha corre como **root**, acepta conexiones en puerto 22.
  * Necesita privilegio root para bind al puerto 22.
  * Necesita privilegio root para operaciones posteriores.
* Crea un nuevo proceso (fork) para cada conexión entrante.
  * Procesa mensajes de red arbitrarios.
  * Pero sigue corriendo como root (necesitará verificar contraseña, iniciar shell, etc.).

---

# Mucho código potencialmente con bugs

* zlib para compresión sobre la red.
* Parsing de paquetes de red.
* Encriptación, intercambio de llaves.
* Autenticación: verificar contraseña, challenge-response, etc.
* Iniciar un shell.
* Re-negociación de llaves después de cierto tiempo.

---

# Los bugs pueden ser muy dañinos

* Buffer overflows y similares.
* Pero también filtrar contenidos sensibles de memoria (como la llave privada).
* O acceder a archivos incorrectos como root.

**Este diseño de "cáscara dura, interior blando" hace que los bugs sean devastadores.**

---

# Plan B: Construir sistemas seguros incluso con bugs

¿Podemos hacer algo así?

**Meta: Principio de Mínimo Privilegio**
  * Cada componente debería tener los mínimos privilegios necesarios para hacer su trabajo.

---

# Gran Idea: Separación de Privilegios

Dividir el software y los datos para limitar el daño de los bugs.

**Dos beneficios relacionados:**
  * Limitar daño de exploit exitoso → "mínimo privilegio"
  * Limitar acceso del atacante a código con bugs → "superficie de ataque"

---

# La Separación de Privilegios es Difícil

* Necesitas diseñar un plan de separación fructífero.
* Necesitas aislar (cliente/servidor, VMs, contenedores, procesos, SFI, etc.).
* Necesitas permitir interacción controlada (API estrecha, verificaciones de seguridad significativas).
* Necesitas mantener buen rendimiento (pocos cruces de dominio en ruta crítica).
* Necesitas refactorizar el software para trabajar con el plan de separación.

---

# El diseñador debe elegir el plan de separación

Posibles criterios:
  * Por servicio / tipo de datos (listas de amigos vs contraseñas)
  * Por usuario (mi email vs tu email)
  * Por propensión a bugs (redimensionar imágenes vs todo lo demás)
  * Por exposición a ataque directo (parsing de mensajes de red vs todo lo demás)
  * Por privilegio inherente (ocultar procesos de superusuario; ocultar llaves o BD)

**El plan de separación depende mucho de la aplicación.**

---

# ¿Cómo hace OpenSSH la separación de privilegios?

* **Proceso listener privilegiado** (corre como root) acepta conexiones entrantes.
* **Proceso monitor privilegiado** (corre como root) por conexión.
* **Proceso worker no privilegiado** para hacer la mayor parte del trabajo de conexión.
  * Parsing de mensajes de red, protocolo crypto, intercambio de llaves, compresión...
  * La conexión TCP se pasa al proceso worker no privilegiado.

---

# Arquitectura de OpenSSH (cont.)

* El proceso worker se **re-crea** después de la autenticación.
  * Mayormente un detalle debido a cómo se puede establecer el user ID de un proceso.
* Eventualmente crea un **proceso shell del usuario**.
  * El proceso worker permanece, manejando la sesión de red encriptada.

---

# ¿Qué operaciones privilegiadas necesitan ocurrir por conexión?

* El protocolo requiere **firmar un mensaje** con la llave privada del servidor.
* Necesita **verificar la contraseña** del usuario.
* Necesita **autenticar** al usuario con public-key auth (challenge-response).
* Necesita **asignar un pseudo-terminal** para la sesión de login del usuario.
* Necesita **iniciar shell** con el UID del usuario.

---

# ¿Por qué estas operaciones requieren root?

La mayoría de estas operaciones privilegiadas requieren privilegio root en Unix:
  * El archivo de llave privada del host solo es legible por root.
  * Por eso OpenSSH solía correr completamente como root.

---

# ¿Cómo hace el worker no privilegiado estas operaciones?

La separación de privilegios define una **nueva interfaz** entre worker y monitor.

* Lista enumerada de operaciones que pueden ser solicitadas.
  * [ Ref: https://github.com/openssh/openssh-portable/blob/master/monitor.h ]
* El proceso monitor realizará **solo estas operaciones** correspondientes.
  * [ Ref: https://github.com/openssh/openssh-portable/blob/master/monitor.c ]

---

# Control estricto sobre operaciones permitidas

* Control estricto sobre qué operaciones están permitidas en qué momento.
  * Ej., flags `MON_ONCE` y `MON_AUTH`.
  * Solo puede hacer ciertas operaciones **una vez** (firmar con llave privada del servidor).
  * Solo puede hacer ciertas operaciones **después de proveer username válido** (verificar pw).

---

# Frontera de seguridad significativa

Frontera de seguridad significativa entre proceso hijo worker y proceso padre monitor.

* **Asumir que el proceso hijo worker está comprometido.**
  * El adversario puede emitir solicitudes arbitrarias al monitor.
* El proceso monitor tiene **privilegios completos de root**.
* Pero las operaciones que exporta son **mucho menos dañinas**.

---

# Operaciones limitadas del monitor

El worker comprometido:
  * No puede obtener la llave privada, solo puede **firmar** con ella.
  * No puede obtener la lista de usuarios, solo puede **verificar** un nombre de usuario particular.
  * No puede obtener el archivo de contraseñas, solo puede **verificar** la contraseña de un usuario.
  * Etc.

---

# Superficie de Ataque: Proceso Worker

* Mensajes de red arbitrarios.
* Parsing, compresión.
* Implementaciones de encriptación e intercambio de llaves.

**Alta exposición a ataques externos.**

---

# Superficie de Ataque: Proceso Monitor

* Aceptar una conexión de red.
* Solicitudes del monitor (monitor.h).

**Exposición limitada y controlada.**

---

# Superficie de Ataque: Proceso Listener

* Casi nada: nueva conexión TCP entrando.
* Sin datos, solo crea un nuevo proceso monitor por cada conexión aceptada.

**Mínima exposición.**

---

# ¿Qué daño si el worker se compromete?

* Podría intentar iniciar sesión como usuario.
  * Pero podría haber hecho eso intentando iniciar sesión via ssh normalmente.
* Podría firmar mensajes usando la llave privada del servidor.
  * Ligeramente preocupante: podría suplantar al servidor para otra conexión.
  * Pero no para conexiones futuras: necesitaría firmar futuros mensajes aleatorios.

---

# ¿Qué daño si el worker se compromete? (cont.)

* Post-autenticación: podría acceder al estado de ese usuario.
  * Pero podría haber hecho eso solo iniciando sesión normalmente.
* Post-autenticación: podría asignar un pseudo-terminal.
  * No mucho daño.
* Podría enviar spam o atacar otras cosas desde la máquina del servidor.
  * El acceso a red no está limitado para el worker no privilegiado.
* Podría agotar memoria, procesos, tiempo de CPU de la máquina.
  * Quizás se podrían aplicar límites de memoria/fork al proceso worker.

---

# ¿Por qué challenge-response requiere el monitor?

**Pregunta:** ¿Por qué no hacer que el proceso hijo genere un challenge aleatorio y verifique la firma?

**Respuesta:** El monitor tiene que verificar el resultado de autenticación, que depende del challenge fresco.
  * Si el worker genera el challenge, un worker comprometido podría falsificarlo.

---

# Mecanismos de Aislamiento y Control

El paper usa:
  * Procesos Unix
  * User IDs (UIDs)
  * Permisos de archivos
  * Paso de file descriptors (fd passing)

---

# setuid(uid)

* Un proceso puede **abandonar sus privilegios** de root a un uid ordinario.
* Operación **irreversible** — una vez que abandonas root, no puedes volver.
* Usado por el worker para reducir sus privilegios.

---

# chroot(dirname)

* Causa que `/` se refiera a `dirname` para este proceso y descendientes.
* El proceso **no puede nombrar archivos fuera** de `dirname`.
* Crea una "jaula" del sistema de archivos.

---

# Prevenir interferencia entre workers

* `P_SUGID` previene que un proceso haga debug de otro proceso usando ptrace.
* Incluso aunque los dos procesos corran con el mismo user ID.
* Importante porque múltiples workers corren como el mismo usuario "sshd".

---

# FD Passing (Paso de File Descriptors)

* Un proceso abre un file descriptor, lo pasa a otro proceso.
* Ej., el monitor asigna un pseudo-terminal, pasa el fd al proceso worker.
* Permite al worker acceder a recursos sin tener el privilegio de crearlos.

---

# Desafío: ¿Cómo establecer User ID después de autenticación exitosa?

* No se puede pasar user ID como file descriptor.
* **Plan de OpenSSH:** matar el viejo proceso hijo worker, iniciar uno nuevo.
* Necesita pasar todo el estado relevante del proceso viejo al nuevo.

---

# Estado que debe transferirse (Sección 4.1)

* Algoritmos y llaves de encriptación/autenticación.
* Contadores de secuencia de mensajes de red.
* Datos de red en buffer.
* Estado de compresión.

---

# Demo: sshd privsep en acción

```bash
es# cp /usr/sbin/sshd /usr/sbin/sshd-demo
es# /usr/sbin/sshd-demo -d -p 2022

es% ps aux | grep sshd-demo
# Un proceso, corriendo como root, esperando conexiones

% ssh es -p 2022
 
es% ps aux | grep sshd-demo
# Un proceso monitor como root, un proceso slave como usuario sshd
```

---

# Demo: sshd privsep en acción (cont.)

```bash
es# cd /proc/monitor-pid; ls -ld root
es# cd /proc/slave-pid; ls -ld root
es# ls -la /run/sshd

# Decir "yes" para aceptar llave en cliente ssh, continuar login

es# ps aux | grep sshd-demo
# Un proceso monitor como root, un proceso slave con nuevo PID como nickolai
```

---

# Herramientas de aislamiento a nivel de proceso Unix son difíciles de usar

* Muchos espacios de nombres globales: archivos, UIDs, PIDs, puertos.
  * Cada uno puede permitir que procesos vean lo que otros están haciendo.
  * Cada uno es una invitación para bugs o configuración descuidada.
* No hay idea de "por defecto sin acceso".
  * Difícil para el diseñador razonar sobre lo que un proceso puede hacer.

---

# Limitaciones de herramientas Unix (cont.)

* No hay concesiones de privilegio de grano fino.
  * No se puede decir "el proceso solo puede leer estos tres archivos".
* No hay forma de limitar acceso a red.
* `chroot()` y `setuid()` solo pueden ser usados por superusuario.
  * Usuarios no-root no pueden reducir/limitar sus propios privilegios.
  * Incómodo ya que la seguridad sugiere *no* correr como superusuario.

---

# Lab 2 usa Linux Containers (LXC)

* No existían cuando los autores diseñaron la separación de privilegios de OpenSSH.
* Los contenedores proveen la ilusión de máquinas virtuales sin usar VMs.
  * Los contenedores son más eficientes que las máquinas virtuales.

---

# ¿Qué es un contenedor?

Un contenedor es un proceso Linux, pero fuertemente aislado:
  * Acceso limitado a los espacios de nombres del kernel.
  * Acceso limitado a system calls.
  * Sin acceso al sistema de archivos (del host).

---

# Los contenedores se comportan como una VM

* Se inician desde una imagen de VM.
* Tienen su propia dirección IP.
* Tienen su propio sistema de archivos.

---

# Contenedores no privilegiados

* Lab 2 usa contenedores **no privilegiados**.
* Estos contenedores corren como procesos de usuario no-root.
* Si el proceso dentro del contenedor corre como root, aún tiene privilegios limitados.
* **Más difícil escapar de un contenedor que de un proceso con chroot.**

---

# ¿Cómo agregar separación de privilegios a código existente?

**Paso 1:** Diseñar el plan de separación.
  * Requirió algo de refactorización del código para exponer esta frontera.

---

# Agregando separación de privilegios (cont.)

**Paso 2:** Wrappers RPC para funciones en la frontera de interfaz del monitor.

Ejemplo del paper:
```c
PRIVSEP(auth_password(authctxt, pwd))
```

* Cuando PRIVSEP está deshabilitado: simplemente `auth_password(authctxt, pwd)`.
* Cuando PRIVSEP está habilitado: `mm_auth_password(authctxt, pwd)`.
* `mm_auth_password()` es un stub de cliente RPC.
  * [ Ref: https://github.com/openssh/openssh-portable/blob/master/monitor_wrap.c ]

---

# Agregando separación de privilegios (cont.)

**Paso 3:** Enviar estado actual al monitor cuando la autenticación tiene éxito.
  * [ Ref: https://github.com/openssh/openssh-portable/blob/master/sshd-auth.c llamada a `mm_send_keystate()` ]
  * Y correspondientemente, desempaquetar este estado cuando el monitor inicia el nuevo proceso worker.

---

# Desafío: separación de privilegios para bibliotecas existentes

**Ejemplo:** worker pre-autenticación usaba zlib para compresión.
  * zlib asignaba sus propios buffers.
  * ¿Cómo transferir esos buffers al nuevo worker post-autenticación?

---

# Solución de OpenSSH para zlib

* Dar a zlib una implementación especial de malloc/free.
* Asigna memoria en una **región de memoria compartida**.
* Esta región de memoria compartida se pasará al nuevo worker tal cual.

---

# Pros y contras de memoria compartida

**Bueno:** Transparente al código existente (como zlib).
  * Muchas bibliotecas/toolkits de separación de privilegios usan trucos similares.

**Malo:** Interfaz complicada con el proceso monitor.
  * Pero al menos el monitor no mira esta región de memoria compartida.
  * Solo pasa la memoria compartida al nuevo proceso worker post-autenticación.

**Malo:** Podría tener punteros corruptos, causará errores arbitrarios en worker.
  * Probablemente no muy malo, porque requiere poder iniciar sesión como usuario.

---

# La asignación de memoria compartida fue removida

* Por razones criptográficas, la compresión pre-autenticación era indeseable.
* El código de asignación de memoria compartida era complejo.
  * Bugs en él debido a comportamiento indefinido, incluso.
  * [ Ref: https://github.com/openssh/openssh-portable/commit/0082fba4efdd492f765ed4c53f0d0fbd3bdbdf7f ]
* Removido en 2016 al no hacer compresión en el proceso slave pre-auth.

---

# ¿Dónde debería buscar un atacante debilidades?

* Podría haber bugs en el proceso worker — buen punto de partida.
* Bugs en el kernel del OS.
  * Explotar bug del kernel, volverse root, escapar del aislamiento.
* Bugs en el código de autenticación del monitor.
  * Buffer overflow, error de lógica, errores criptográficos.
  * Autenticarse incorrectamente como usuario víctima.

---

# ¿Qué tan seguro es el OpenSSH con separación de privilegios?

**Sección 5 del paper.**

**Una medida de vulnerabilidades potenciales: líneas de código.**
  * Worker no privilegiado es aproximadamente **2/3 del código**.
  * Monitor privilegiado es aproximadamente **1/3 del código**.
  * Menos líneas de código → menos bugs.

---

# Medidas de seguridad (cont.)

**Otra medida: superficie de ataque.**
  * Worker no privilegiado: mensajes de red arbitrarios.
  * Monitor privilegiado: interfaz bien definida, pocas operaciones, estructura fija.
  * ¿Relativamente menos probable tener corrupción de memoria, etc.?

---

# Estudio empírico: vulnerabilidades prevenidas

Muchas vulnerabilidades anteriores habrían sido prevenidas:
  * **Pre-autenticación:** integer overflow en código de procesamiento de paquetes de red.
  * **Pre-autenticación:** bug de zlib.
  * **Post-autenticación:** error off-by-one en código de channel.
  * **Post-autenticación:** paso de tickets Kerberos.

---

# Separación ayuda incluso con bugs post-autenticación

* Solía seguir corriendo como root, debido a re-negociación de llaves (necesita firmar).
* Con separación de privilegios, el worker ya no tiene esos privilegios.

---

# ¿Cuál es el overhead de rendimiento?

**Diseño descrito/desplegado: ¡prácticamente sin overhead de rendimiento!**
  * Sección 6 del paper.

Rápido porque la separación de privilegios **no está en la ruta crítica** para transferencia de datos.
  * Después del login, todo funciona básicamente igual que sin privsep.

---

# Rendimiento (cont.)

* Overhead menor para establecer nueva conexión / login, pero no significativo.
* **Resultado directo de elegir cuidadosamente la interfaz de separación de privilegios correcta.**

---

# Diseño alternativo ("3 procesos") de sección 4.3

Sería más lento:
  * Evitaría la necesidad de transferencia de estado compleja.
  * Worker existente sigue manejando encriptación/compresión en conexión de red.
  * Nuevo worker maneja la sesión del usuario.
  * Introduce algo de overhead en estado estable: más cambio de contexto.

---

# OpenSSH hoy

* OpenSSH todavía usa este diseño básico de separación de privilegios hoy.
* Cambios relativamente menores (como eliminar la memoria compartida para zlib).

---

# Aspectos únicos de OpenSSH

OpenSSH tiene algunos aspectos únicos que influyen en su plan de separación de privilegios:
  * Cada conexión es mayormente **independiente**.
    * El estado compartido está en los archivos que el usuario puede acceder después de iniciar sesión.
    * No es realmente problema de OpenSSH.
  * Un proceso monitor privilegiado, relativamente pocos recursos privilegiados.
    * Llave privada del servidor, base de datos de contraseñas, capacidad de setuid().

---

# Otros sistemas son bastante diferentes

Otros sistemas que veremos son bastante diferentes:
  * **Aplicaciones web:** lab 2 y paper de Google del jueves.
  * Muchos recursos diferentes (autenticación de usuario, bases de datos, servicios, etc.).
  * Servicios con estado (ej., BD) en lugar de iniciar un worker fresco cada vez.
  * Permisos dinámicos (ej., tickets de permiso de usuario de Google).

---

# Resumen

* **Separación de privilegios** es una técnica poderosa para limitar daño de bugs.
* OpenSSH demuestra un diseño exitoso:
  * Worker no privilegiado maneja datos de red no confiables.
  * Monitor privilegiado expone operaciones limitadas y controladas.
* Rendimiento puede ser excelente si la separación está fuera de la ruta crítica.
* Los contenedores modernos ofrecen mejor aislamiento que las herramientas Unix tradicionales.

---

# Referencias

* Paper: "Privilege Separated OpenSSH" - Niels Provos, Markus Friedl, Peter Honeyman
* Código fuente:
  * https://github.com/openssh/openssh-portable/blob/master/monitor.h
  * https://github.com/openssh/openssh-portable/blob/master/monitor.c
  * https://github.com/openssh/openssh-portable/blob/master/monitor_wrap.c
