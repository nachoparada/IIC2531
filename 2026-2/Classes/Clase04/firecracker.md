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

# Aislamiento de SO y VM

---

# Caso de estudio: Paper de Firecracker de Amazon
  * Servicio Lambda: ejecutar aplicación Linux suministrada por el cliente, escalando según la carga
  * Desafío de seguridad: código arbitrario, necesita aislarlo de otros clientes
  * Desafío de rendimiento: la carga puede variar ampliamente
    * Podría ser mucho menos que una máquina (muchos clientes por máquina)
    * Podría crecer rápidamente y necesitar ejecutar código de cliente en nueva máquina
  * Objetivo: dominios de aislamiento de bajo overhead pero fuertes

---

# Enfoques de aislamiento
  * Procesos Linux (como en OKWS)
  * Contenedores, usando namespaces de Linux + cgroups
  * VMs
  * Runtimes de lenguaje (esto no lo veremos hoy)

---

# Procesos Linux
  * IDs de usuario
  * Permisos por archivo
  * Diseñado para compartir granular entre usuarios, no aislamiento grueso
    * Básicamente tenemos demasiados recursos compartidos
  * Usamos permisos, chroot, etc. pero no es suficiente

---

# Los contenedores Linux sirven dos propósitos
  * Empaquetar software junto con todas las dependencias (bibliotecas, paquetes, archivos, etc)
  * Aislamiento de seguridad y rendimiento para ejecutar ese software
  * Ambos dependen de namespaces de Linux: abstracción de ejecutar en una máquina Linux separada
  * El aislamiento de rendimiento usa cgroups de Linux para controlar el uso de recursos

---

# ¿Por qué es desafiante el aislamiento en Linux?
  * Mucho estado compartido ("recursos") en el kernel
  * Las llamadas al sistema acceden al estado compartido nombrándolo
    * PIDs
    * Nombres de archivo
    * Direcciones IP / puertos
    * (Incluso IDs de usuario, en alguna forma)
  * El control de acceso típico gira en torno a IDs de usuario (ej., permisos de archivo)
    * Difícil usar eso para hacer cumplir aislamiento entre dos aplicaciones
    * Muchos archivos con permisos
    * Las aplicaciones crean archivos compartidos por accidente o a propósito (ej., world-writable)

---

# Mecanismo Linux: chroot
  * Vimos esto en OKWS
  * Beneficio: limita los archivos que una aplicación puede nombrar
    * No importa si la aplicación crea accidentalmente archivos world-writable
  * Algunas limitaciones técnicas, pero un buen punto de partida para mejor aislamiento

---

# Los namespaces proporcionan una forma de delimitar los recursos que se pueden nombrar
  * [Quarkslab: Digging into Linux Namespaces](https://blog.quarkslab.com/digging-into-linux-namespaces-part-1.html)
  * El proceso pertenece a un namespace particular (para cada tipo de namespace)
    * Los nuevos procesos heredan el namespace del proceso padre
  * Ej., el namespace PID limita los PIDs que un proceso puede nombrar
  * Aislamiento de grano grueso, no sujeto a lo que la aplicación podría hacer
  * Un chroot mejor diseñado para diferentes tipos de recursos (no solo sistema de archivos)

---

# Cgroups de Linux
  * Limitar / programar el uso de recursos
  * Memoria, CPU e I/O de disco, entre otros
  * El tráfico de red puede clasificarse mediante cgroups, pero normalmente se limita usando mecanismos como tc
  * Se aplica a procesos, similar a namespaces
    * Los nuevos procesos heredan el cgroup del proceso padre
  * No es un límite de seguridad, pero importante para prevenir ataques DoS
    * Ej., un proceso o VM trata de monopolizar toda la CPU o memoria

---

# Contenedores usando namespaces + cgroups
  * Desempaquetar archivos del contenedor en algún lugar del sistema de archivos
  * Crear namespaces para aislar recursos, incluyendo un mount namespace
  * Usar pivot_root/chroot para establecer el árbol de archivos del contenedor como su raíz
  * Configurar cgroup para el contenedor basado en cualquier política de programación
  * Configurar una interfaz de red virtual para el contenedor
  * Ejecutar procesos en este contenedor
    * Parece ejecutarse en un sistema Linux separado
    * Su propio sistema de archivos, su propia interfaz de red, sus propios procesos (PIDs), etc

---

# ¿Por qué los namespaces no son suficientes para Lambda?
  * Kernel Linux compartido
  * Superficie de ataque amplia: cientos de llamadas al sistema, además de muchas operaciones especializadas mediante ioctl
    * El número exacto depende de la arquitectura y versión del kernel
  * Gran cantidad de código, escrito en C
    * Los errores (buffer overflows, use-after-free, ...) continúan siendo descubiertos
    * No hay aislamiento dentro del kernel Linux mismo
  * Los errores del kernel permiten al adversario escapar del aislamiento ("escalación de privilegios local" o LPE)
    * Relativamente común: nuevos errores LPE cada año

---

# Mecanismo de seguridad adicional: seccomp-bpf
  * Idea: filtrar qué llamadas al sistema puede invocar un proceso
  * Podría ayudarnos a abordar la amplia superficie de ataque del kernel Linux
  * Patrón común señalado en el paper de Lambda:
    * Los syscalls o características raramente usados son más propensos a tener errores
  * Cada proceso está (opcionalmente) asociado con un filtro de llamadas al sistema
    * Filtro escrito como un pequeño programa en el lenguaje bytecode BPF
    * El kernel Linux ejecuta este filtro en cada invocación de syscall, antes de ejecutar el syscall
    * El programa filtro puede decidir si el syscall debe ser permitido o no
    * Puede mirar syscall#, argumentos, etc
    * Los nuevos procesos heredan el filtro de syscall del proceso padre: "pegajoso"

---

# Mecanismo de seguridad adicional: seccomp-bpf (cont.)
  * Se puede usar seccomp-bpf para prevenir acceso a syscalls sospechosos
  * Usado por algunas implementaciones de contenedores
    * Configurar filtro bpf para deshabilitar llamadas al sistema sospechosas
  * ¿Por qué esto no es suficiente para Lambda?
    * Mal trade-off
    * Comenzando a romper código de cliente que usa syscalls poco comunes
    * Pero aún podría no ser suficiente para seguridad (mucho código/errores en syscalls comunes)

---

# Enfoque más pesado: VMs
  * Ejecutar Linux en una VM guest
  * ¿Por qué esto es mejor que Linux?
    * Superficie de ataque más pequeña: no hay syscalls complejos, solo instrucciones de CPU y dispositivos virtuales
      * El paper se enfoca en x86_64; las versiones actuales de Firecracker también soportan ARM de 64 bits (aarch64)
    * Históricamente, menos vulnerabilidades de escape que en un kernel compartido
      * Según los datos citados por el paper, se descubrían errores de escape de VM menos de una vez al año
      * Es una observación histórica, no una garantía de seguridad
  * ¿Por qué estos tampoco son suficientes para Lambda?
    * Alto costo de inicio: toma mucho tiempo arrancar la VM
    * Alto overhead: gran costo de memoria para cada VM en ejecución
    * Errores potenciales en el VMM mismo (qemu): 1.4M líneas de código C
  * Plan del paper: escribir un nuevo VMM, pero seguir usando KVM

---

# ¿Qué implica implementar soporte para VMs?
  * Virtualizar la CPU y memoria
    * Soporte de hardware en procesadores modernos
    * Tablas de páginas anidadas
    * Virtualizar registros privilegiados que normalmente solo son accesibles al kernel
  * Virtualizar dispositivos
    * Controlador de disco
    * Tarjeta de red
    * PCI
    * Tarjeta gráfica
    * Teclado, puertos serie, ...
  * Virtualizar el proceso de arranque
    * BIOS, boot loader

---

# Linux KVM
  * [Documentación oficial de la API de KVM](https://www.kernel.org/doc/html/latest/virt/kvm/api.html)
  * Abstracción para usar soporte de hardware para virtualización
  * Gestiona CPUs virtuales, memoria virtual

---

# QEMU
  * Implementa dispositivos virtuales similares al hardware real
  * También implementa dispositivos puramente virtuales (virtio)
    * Interfaces estandarizadas basadas en memoria compartida
  * Puede ejecutar la CPU guest de dos maneras:
    * Emulación por software (TCG): traduce instrucciones y permite ejecutar una arquitectura distinta a la del host
    * Virtualización por hardware (KVM): la mayoría de las instrucciones se ejecutan directamente en la CPU
      * Operaciones sensibles producen un VM exit y son manejadas por KVM o QEMU
  * Proporciona mecanismos para iniciar la VM
    * Puede cargar firmware como SeaBIOS u OVMF, o iniciar un kernel directamente
      * SeaBIOS implementa el BIOS tradicional; OVMF implementa UEFI

---

# Diseño de Firecracker
  * Usar KVM para CPU virtual y memoria
  * Re-implementar QEMU
  * Soportar un conjunto mínimo de dispositivos
    * Inicialmente: virtio network, virtio block (disco), teclado y puerto serie
    * Versiones posteriores agregaron otros dispositivos, como vsock, balloon y virtio-rng, pero mantienen una superficie de dispositivos deliberadamente limitada
      * Permiten comunicación host–guest, gestión dinámica de memoria y acceso seguro a entropía, respectivamente
  * Dispositivos de bloque en lugar de sistema de archivos: límite de aislamiento más fuerte
    * El sistema de archivos tiene estado complejo
      * Directorios, archivos de longitud variable, symlinks / hardlinks
    * El sistema de archivos tiene operaciones complejas
      * Crear/eliminar/renombrar archivos, mover directorios completos, r/w range, append, ...
    * El dispositivo de bloque es mucho más simple:
      * Bloques de 4 KByte
      * Los bloques están numerados del 0 al N, que es el tamaño del disco
      * Leer y escribir un bloque completo (Y tal vez flush / barrier)

---

# Diseño de Firecracker (cont.)
  * No soportar emulación de instrucciones
    * (Excepto instrucciones necesarias como CPUID, VMCALL/VMEXIT, ..)
  * No soportar BIOS en absoluto
    * Solo cargar el kernel en la VM en la inicialización y comenzar a ejecutarlo

---

# Implementación de Firecracker: Rust
  * Lenguaje memory-safe (módulo código "unsafe")
  * Al momento del paper: aproximadamente 50K líneas de código, mucho menos que QEMU
  * Actualmente supera las 120K líneas de Rust; QEMU supera los 2.4M de líneas de C y headers
  * Hace improbable que la implementación VMM tenga errores como buffer overflows
  * [Repositorio de Firecracker en GitHub](https://github.com/firecracker-microvm/firecracker)

---

# Aislamiento del proceso VMM: el jailer
  * Firecracker incluye un programa separado llamado jailer
    * Configura el aislamiento antes de ejecutar el proceso VMM
  * Crea un mount namespace y usa pivot_root/chroot para restringir el sistema de archivos visible
  * Ejecuta el VMM con un usuario y grupo sin privilegios
  * Puede configurar cgroups y límites de recursos
  * Los namespaces de red y PID son opcionales
  * Firecracker instala filtros seccomp-bpf para limitar las llamadas al sistema permitidas
  * Defensa en profundidad: si se explota un error del VMM, estas capas dificultan escalar el ataque

---

# Arquitectura Lambda alrededor del mecanismo central de Firecracker
  * Muchos workers (máquinas físicas ejecutando MicroVMs basados en Firecracker)
  * Cada worker tiene un número fijo de "slots" de MicroVM disponibles para ejecutar cosas
  * El código del cliente se carga en un "slot" iniciando una VM, cargando código del cliente
  * El manager de workers se encarga de decidir dónde se enrutan las solicitudes
  * El frontend busca worker del manager de workers, envía solicitud directamente allí

---

# Evaluación de Firecracker en el paper (2020)
  * Overhead bajo
    * 3MB de overhead de memoria por VM inactiva
    * 125ms de tiempo de arranque
  * Rendimiento de CPU cercano a KVM
  * Limitaciones de I/O observadas en la versión evaluada
    * El disco virtual requería mayor concurrencia para mejorar su rendimiento
    * La red virtual era más lenta; el paper menciona PCI passthrough como posible optimización
  * Versiones posteriores agregaron soporte opcional para virtio-pci
    * Mejora el transporte de dispositivos virtuales, pero no entrega un dispositivo físico directamente a la VM

---

# ¿Qué tan bien logra Firecracker sus objetivos? (cont.)
  * Seguridad probablemente bastante buena
    * Implementación en Rust: menos propensa a errores de memoria
    * VMM y modelo de dispositivos mucho más pequeños que los de QEMU
    * Capas adicionales de aislamiento mediante el jailer
  * KVM y el kernel del host todavía forman parte del Trusted Computing Base (TCB)
    * Una vulnerabilidad en KVM podría romper el aislamiento de Firecracker
    * [Google Project Zero: An EPYC Escape - KVM Vulnerability Case Study](https://googleprojectzero.blogspot.com/2021/06/an-epyc-escape-case-study-of-kvm.html)

---

# Algunos errores encontrados en Firecracker
  * [Issue #1462: Memory bounds-checking vulnerability](https://github.com/firecracker-microvm/firecracker/issues/1462)
    * Problema de verificación de límites de memoria, a pesar de estar escrito en Rust
  * [Issue #2057: Network interface DoS bug](https://github.com/firecracker-microvm/firecracker/issues/2057)
    * Error DoS en interfaz de red
  * [Issue #2177: Unbounded serial console buffer](https://github.com/firecracker-microvm/firecracker/issues/2177)
    * El buffer de consola serie creció sin límite
    * Podría causar que una VM use mucha memoria a través del proceso Firecracker

---

# Firecracker usado fuera de Lambda
  * [Fly.io: Sandboxing and Workload Isolation](https://fly.io/blog/sandboxing-and-workload-isolation/)

---

# Resumen
  * El aislamiento es un bloque de construcción clave para la seguridad (una vez más)
  * Desafiante lograr aislamiento junto con otros objetivos:
    * Alto rendimiento
    * Overheads bajos (memoria, cambio de contexto, etc)
    * Compatibilidad con sistemas existentes (ej., Linux)
  * Caso de estudio del mundo real de ingeniería de un mecanismo de aislamiento
