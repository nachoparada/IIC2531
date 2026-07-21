# Lab Setup: Instalando el Ambiente de Desarrollo

## Introducción

Realizarás una secuencia de laboratorios en IIC2531. Estos laboratorios te darán experiencia práctica con ataques comunes y contramedidas. Para hacer los temas concretos, explorarás los ataques y contramedidas en el contexto de la aplicación web zoobar de las siguientes maneras:

- **Lab 2:** mejorarás la aplicación web zoobar usando separación de privilegios, de modo que si un componente es comprometido, el adversario no obtenga control sobre toda la aplicación web.
- **Lab 3:** explorarás la aplicación web zoobar, y usarás ataques de desbordamiento de buffer para romper sus propiedades de seguridad.
- **Lab 4:** mejorarás la aplicación zoobar contra ataques de navegador.
- **Lab 5:** agregarás soporte HTTPS y autenticación con llave de seguridad (WebAuthn).

Cada laboratorio requiere que aprendas un nuevo lenguaje de programación u otra pieza de infraestructura. Por ejemplo, en el lab 3 debes familiarizarte íntimamente con ciertos aspectos del lenguaje C, lenguaje ensamblador x86, `gdb`, etc. Se necesita familiaridad detallada con muchas piezas diferentes de infraestructura para entender ataques y defensas en situaciones realistas: las debilidades de seguridad a menudo aparecen en casos límite, por lo que necesitas entender los detalles para crear exploits y diseñar defensas para esos casos límite. Estos dos factores (nueva infraestructura y detalles) pueden hacer que los laboratorios consuman mucho tiempo. Deberías comenzar temprano con los laboratorios y trabajar en ellos diariamente por un tiempo limitado (cada laboratorio tiene varios ejercicios), en lugar de intentar hacer todos los ejercicios de una sola vez justo antes de la fecha límite. Tómate el tiempo para entender los detalles relevantes.

Varios laboratorios te piden diseñar exploits. Estos exploits son lo suficientemente realistas como para que podrías usarlos en un ataque real, pero *no* deberías hacerlo. El punto de diseñar exploits es enseñarte cómo defenderte contra ellos, no cómo usarlos---atacar sistemas computacionales es ilegal y puede meterte en serios problemas. No lo hagas.

**NOTA:** Dado que reutilizamos las mismas asignaciones de laboratorio a través de los años, te pedimos que por favor no hagas tu código de laboratorio públicamente accesible (por ejemplo, subiendo tus soluciones a un repositorio público en GitHub). Esto ayuda a mantener los laboratorios justos e interesantes para estudiantes en años futuros.

## Infraestructura del laboratorio

Un pequeño cambio en el compilador, variables de entorno, o la forma en que se ejecuta el programa puede resultar en una disposición de memoria y estructura de código ligeramente diferente, requiriendo así un exploit diferente. Por esta razón, este laboratorio usa una [virtual machine](https://en.wikipedia.org/wiki/Virtual_machine) para ejecutar el código del servidor web vulnerable.

### Descargando la VM

Hemos preparado una virtual machine para ti que contiene una instalación estándar de [Ubuntu](https://ubuntu.com/) 24.04 Linux, la cual puedes descargar [aquí](https://drive.google.com/file/d/19PW3WVsMNR0y0jrRzRjg8SSi6e73fTbk/view?usp=sharing) y descomprimirla en tu computador. Para comenzar a trabajar en esta asignación de laboratorio, necesitarás alguna forma de ejecutar esta virtual machine. Hay algunas opciones; si no estás seguro de cómo ejecutar una VM, ejecutar la VM en AWS (la última opción) puede ser un buen plan por defecto:

- **Linux con CPU x86.**
  Usa [KVM](https://www.linux-kvm.org/), que está incorporado en el kernel de Linux. KVM debería estar disponible a través de tu distribución, y está preinstalado en los computadores del clúster Athena; en Debian o Ubuntu, prueba `apt-get install qemu-kvm`. KVM requiere soporte de virtualización por hardware en tu CPU, y debes habilitar este soporte en tu BIOS (lo cual es frecuentemente, pero no siempre, el valor por defecto). Si tienes otro monitor de virtual machine instalado en tu máquina (por ejemplo, VMware), ese monitor de virtual machine puede tomar el soporte de virtualización por hardware exclusivamente y prevenir que KVM funcione.

  Para iniciar la VM con KVM, descomprime el archivo zip que descargaste arriba y ejecuta `./6.566-standalone-v26.sh` desde una terminal (`Ctrl+A x` para forzar salida). Si obtienes un error de permiso denegado de este script, intenta agregarte al grupo `kvm` con `sudo gpasswd -a \`whoami\` kvm`, luego cierra sesión y vuelve a iniciar sesión.

- **Windows con CPU x86 (la mayoría de ellos).**
  Descarga [VMware Workstation](https://ist.mit.edu/vmware/workstation) de IS&T. Para iniciar la VM del curso usando VMware, importa `6.566-standalone-v26.vmdk`. Ve a File > New, selecciona "create a custom virtual machine", elige Linux > Debian 9.x 64-bit, elige Legacy BIOS, y usa un disco virtual existente (y selecciona el archivo `6.566-standalone-v26.vmdk`, eligiendo la opción "Take this disk away"). Finalmente, haz clic en Finish para completar la configuración.

- **Mac con CPU x86 (por ejemplo, no M1).**
  Descarga [UTM](https://mac.getutm.app/). Para la configuración, haz clic en "Create a New Virtual Machine", haz clic en "Emulate", haz clic en "Other", marca "Skip ISO boot", especifica el tamaño del disco como 24 GB, y continúa hasta la creación. Después de la creación, haz clic derecho en tu VM, haz clic en "edit", haz clic derecho en "IDE Drive", haz clic en "delete", haz clic en "New" Drive, haz clic en "Import", y abre el archivo "vmdk" de la descarga de la VM del curso. Para agregar reenvío de puertos, haz clic en "Network", cambia "Network Mode" a "Emulated VLAN", cambia "Network Card" a "virtio-net-pci", haz clic en el desplegable "Port Forward", haz clic en "New", llena "22" en la 2da casilla y "2222" en la 4ta casilla.

- **Para usuarios de computadores no-x86.**
  Si estás usando un computador con un procesador no-x86 (por ejemplo, laptops Mac con procesador ARM), puedes ejecutar la virtual machine usando qemu. Para hacer esto, primero instala [Homebrew](https://brew.sh/), luego instala qemu ejecutando [brew install qemu](https://formulae.brew.sh/formula/qemu), y finalmente edita el script `6.566-standalone-v26.sh` que viene con la imagen de la VM del curso. En las versiones nuevas del script, para macOS (`Darwin`) se usa la bandera `-machine accel=hvf`, por lo que si necesitas ajustar la aceleración en Mac debes modificar esa bandera (no `-enable-kvm`, que corresponde a Linux). En este punto, deberías poder iniciar la VM del curso ejecutando `./6.566-standalone-v26.sh` como se indicó arriba.

### Iniciando sesión

Usarás la cuenta `student` en la VM para tu trabajo. La contraseña para la cuenta `student` es `student`. También puedes obtener acceso a la cuenta `root` en la VM usando `sudo`; por ejemplo, puedes instalar nuevos paquetes de software usando `sudo apt-get install pkgname`.

Puedes iniciar sesión en la virtual machine usando su consola, o usar ssh para iniciar sesión en la virtual machine a través de la red (virtual). Esto último también te permite copiar fácilmente archivos hacia y desde la virtual machine con `scp` o `rsync`. Cómo accedes a la virtual machine a través de la red depende de cómo la estés ejecutando. Si estás usando VMware, primero tendrás que encontrar la dirección IP de la virtual machine. Para hacerlo, inicia sesión en la consola, ejecuta `ip addr show dev eth0`, y anota la dirección IP listada junto a `inet`. Con kvm, puedes usar `localhost` como la dirección IP para ssh y HTTP. Ahora puedes iniciar sesión con ssh ejecutando el siguiente comando desde tu máquina host: `ssh -p 2222 student@IPADDRESS`.

Por seguridad, SSH no permite iniciar sesión a través de la red usando una contraseña (y, en este caso específico, la contraseña es conocida por todos). Para iniciar sesión vía SSH, necesitarás configurar una [SSH Key](https://www.booleanworld.com/set-ssh-keys-linux-unix-server/).

También puede resultarte útil crear un alias de host para tu VM de 6.566 en tu archivo `~/.ssh/config`, de modo que puedas simplemente ejecutar, por ejemplo, `ssh 566vm` o `scp file.txt 566vm:lab/file.txt`. Para hacer esto, agrega las siguientes líneas a tu archivo `~/.ssh/config`, ajustadas según sea necesario:

```
Host 566vm
  User student
  HostName localhost
  Port 2222
```
## Primeros pasos

Los archivos que necesitarás para este y los laboratorios siguientes se distribuyen usando el sistema de control de versiones [Git](https://git-scm.com/). También puedes usar Git para llevar un registro de cualquier cambio que hagas al código fuente inicial. Aquí tienes una [introducción a Git](https://missing.csail.mit.edu/2020/version-control/) y el [manual de usuario de Git](https://www.kernel.org/pub/software/scm/git/docs/user-manual.html), que pueden resultarte útiles.

El repositorio Git del curso está disponible en [https://github.com/nachoparada/IIC2531-26-01-labs](https://github.com/nachoparada/IIC2531-26-01-labs). Para obtener el código del laboratorio, inicia sesión en la VM usando la cuenta `student` y clona el código fuente para el laboratorio 1 de la siguiente manera:

```
student@6566-v26:~$ git clone https://github.com/nachoparada/IIC2531-26-01-labs lab
Cloning into 'lab'...
student@6566-v26:~$ cd lab
student@6566-v26:~/lab$
```

Es importante que clones el repositorio del curso en el directorio `lab`, porque la longitud de las rutas de archivos será relevante en este laboratorio.

---

**Basado en**: MIT Course 6.566 Lab 2 (Spring 2026)  
**URL Original**: https://css.csail.mit.edu/6.566/2026/labs/lab2.html  
**Licencia**: [Creative Commons Attribution 3.0 Unported](http://creativecommons.org/licenses/by/3.0/us/)

Este laboratorio está adaptado de los materiales del curso Computer Systems Security del MIT.  
Todo el crédito del contenido original corresponde al equipo docente de MIT CSAIL.
