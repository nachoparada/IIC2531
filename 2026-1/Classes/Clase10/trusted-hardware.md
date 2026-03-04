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

# Hardware Confiable (Trusted Hardware)

Aislamiento en un nuevo modelo de amenaza:
**El adversario tiene algún control físico de la computadora.**

---

# Ataques físicos potenciales

* El adversario roba un laptop, intenta extraer datos.
* Laptop descomisionado desechado, el adversario intenta obtener datos del disco.
* El adversario manipula un servidor en un centro de datos.
* El adversario modifica el disco para contener software malicioso.
* El adversario intenta extraer datos secretos de un laptop o servidor.

---

# Dos contextos generales para esta amenaza

**1. El dueño del dispositivo es confiable**
  * Historia de Bitlocker, principalmente sobre robo.

**2. El dueño del dispositivo no es confiable**
  * DRM, computación en la nube, tarjetas inteligentes.

---

# Seguridad contra atacantes físicos

En general, la seguridad contra un atacante físico tiende a ser un **área gris**.

* Un adversario capaz puede manipular físicamente el hardware, extraer secretos, etc.
* Es posible aumentar el esfuerzo del ataque, pero pocas ideas proporcionan un gran salto.
* Una de estas ideas es el foco de este paper: **usar criptografía**.

---

# Modelo general para hardware confiable

* **Chip confiable** que asumimos no ha sido manipulado.
* Generalmente, se asume que el chip confiable es simple, para hacerlo resistente a manipulación.
  * Dispositivos más grandes podrían ser muy costosos de proteger contra manipulación física.
* **Todo lo externo** al chip confiable podría ser controlado por el adversario.
  * En particular, incluye I/O, memoria y almacenamiento.
* Técnica poderosa: usar criptografía (encriptación, autenticación).

---

# Paper de hoy: Bitlocker

Paper interesante: trade-offs de ingeniería del mundo real para un sistema de seguridad.

* Los autores reconocen plenamente que su sistema no es perfecto.
* Sin embargo, el diseño tiene sentido para el modelo de amenaza objetivo.
* **Sistema real:** usado en Windows Vista en adelante.
* Historia limpia sobre usar TPM como hardware confiable para ataques físicos.

---

# ¿Qué problema intenta resolver Bitlocker?

**Prevenir que datos sean robados si un atacante roba físicamente un laptop.**

¿Cómo obtendría un atacante datos de un laptop robado, sin Bitlocker?
* Fácil: sacar el disco; bootear desde CD y cambiar contraseñas; etc.

**Foco de Bitlocker:** encriptación de disco, con soporte de hardware confiable.

---

# ¿Por qué Bitlocker necesita hardware confiable?

**Problema:** ¿de dónde vienen las llaves de encriptación (o desencriptación) del disco?

---

# Enfoque simple: el usuario provee la llave

* El usuario podría ingresar contraseña (hasheada para producir llave, como en Kerberos).
  * Problema: usuario necesita ingresar contraseña temprano en el proceso de boot (BIOS).
  * Problema: las contraseñas son débiles, el adversario puede intentar adivinar.
* El usuario podría conectar una unidad USB conteniendo la llave.
  * Problema: los usuarios no quieren cargar llaves USB extra.
  * Problema: los usuarios podrían perder la llave USB junto con el laptop.

---

# Plan de Bitlocker

**Usar hardware confiable para obtener la llave.**

Trade-off de diseño entre garantías de seguridad fuertes y usabilidad.

---

# Punto de partida: ¿cómo saber que tu sistema está ejecutando el software correcto?

A menudo aparece en dispositivos de función fija:
* Consolas de juegos
* Chromebooks
* etc.

---

# Plan básico: Secure Boot

1. El código de boot inicial viene de **ROM**, cableado en tiempo de manufactura.
2. El código ROM carga el boot loader, **verifica la firma** del boot loader.
   * ROM viene con llave pública usada para verificación.
3. El boot loader carga el kernel del OS, similarmente verifica la firma.
4. El kernel del OS verifica la integridad de todos los demás datos que carga.

---

# Complicación técnica

Más allá del kernel del OS, es muy costoso verificar todos los datos.
* Ej., la imagen completa del OS incluyendo todas las bibliotecas.
* No quieres cargarla del disco solo para verificar la firma.
* En su lugar, necesitas algún plan más eficiente para autenticar datos.

---

# Enfoques para autenticación eficiente

**Enfoque 1:** Firma sobre la raíz Merkle de un árbol del sistema de archivos.
* Verificar pruebas Merkle cuando se cargan datos del disco después.
* Efectivamente difiriendo las verificaciones.

**Enfoque 2:** "Autenticación del pobre" de Bitlocker.
* Más eficiente pero garantía más débil.

---

# Sistemas que usan Secure Boot

* Apple iOS devices
* Consolas de juegos (Playstation, Xbox, etc.)
* Chrome OS Verified Boot
  * [ Ref: https://www.chromium.org/chromium-os/chromiumos-design-docs/verified-boot ]
* UEFI Secure Boot
  * [ Ref: https://docs.microsoft.com/en-us/windows/security/information-protection/secure-the-windows-10-boot-process ]

---

# Desafío común: Rollback

* El Boot ROM es **sin estado**, no tiene idea qué versión pudo haber visto antes.
* Un diseño ingenuo podría ser engañado para bootear un OS antiguo que tiene bugs de seguridad.
* **Una solución:** contador monotónico en hardware rastrea la última versión de OS vista.
* Trade-offs más inteligentes posibles: ver caso de estudio de Apple iOS en clase posterior.

---

# Alternativa más flexible: "Measured Boot"

* Secure boot supone que sabes qué llave debe firmar el software.
* ¿Qué pasa si el hardware no sabe qué software es bueno vs malo?

**Idea:** Medir qué software se bootea.
* Hashear el boot loader, luego hashear el kernel del OS, luego hashear la imagen del OS, etc.

---

# Measured Boot: Generación de secretos

* **No puede prevenir** que software malo se cargue, ¡pero puede generar secretos diferentes!
* El sistema tiene algún secreto almacenado durablemente en hardware (mismo entre reboots).
* Cuando el sistema bootea, **deriva una llave secreta** basada en su secreto de hardware.
  * Derivación basada en hash del boot loader, kernel del OS, etc.

---

# Measured Boot: Implicaciones

* El OS obtiene llave secreta para desencriptar sus datos, para autenticarse con servidores remotos, etc.
* Bootear un OS diferente (ej., debido a corrupción por malware) **genera una llave diferente**.

---

# Measured Boot requiere chip de medición confiable separado

* Con secure boot, estábamos seguros de que todo el código ejecutando era "confiable".
* Con measured boot, no sabemos qué es confiable o no; solo lo medimos.
* La CPU principal podría estar ejecutando código arbitrario, entonces ¿cómo hacemos mediciones?

---

# TPM: Trusted Platform Module

Enfoque de measured boot en x86.

```
               DRAM       /-- BIOS
                 |        |
    CPU --- Northbridge --+-- TPM
```

---

# Estructura del TPM

El chip TPM tiene:
* Un conjunto **efímero** de registros (PCR0, PCR1, ..)
* Una **llave** secreta

---

# Operaciones soportadas del TPM

* `TPM_extend(m)`: extender un registro PCR
  * `PCRn = SHA1(PCRn || m)`
* `TPM_quote(n, m)`: generar firma de `(n -> PCRn, m)` con llave del TPM
* `TPM_seal(n, PCR_value, plaintext)`: retorna ciphertext
* `TPM_unseal(ciphertext)`: retorna plaintext, si PCRn coincide con PCR_value

---

# ¿Quién hace la autenticación/medición para measured boot?

* Los valores PCR se resetean a cero **solo cuando toda la computadora se resetea**.
  * Importante: CPU y TPM deben resetear juntos.
  * Importante: CPU debe saltar a código BIOS, que no está manipulado.

---

# Cadena de medición

1. Código BIOS "se mide a sí mismo": extiende PCR con hash de su código.
2. Código BIOS carga boot loader (ej., Linux grub), lo mide (extiende PCR con hash del boot loader), lo ejecuta.
3. Boot loader carga kernel, lo mide (extiende PCR con H(kernel)), lo ejecuta.

---

# ¿Qué podemos inferir si algún PCRn corresponde a una cadena particular de hashes?

* **Podría ser** que la cadena de software correcta fue cargada.
* **O** algún software en el camino tenía un bug, fue explotado, y el adversario emitió sus propios extends desde ese punto.
* **O** la CPU no comenzó con el código BIOS en primer lugar.
* **O** el hardware TPM no se reseteó sincrónicamente con la CPU.
  * [ Resultó ser "fácil" en algunas motherboards: solo cortocircuitar un pin. ]

---

# Demo: TPM en Linux

```bash
systemd-analyze pcrs
tpm2_pcrread

## obtener valores PCR actuales
tpm2_pcrread -o pcr.bin sha256:3
od -t x1 pcr.bin
```

---

# Demo: Crear política y sellar secreto

```bash
## crear una política conteniendo los valores PCR que acabamos de leer
tpm2_createpolicy --policy-pcr -l sha256:3 -f pcr.bin -L pcr.policy

## crear una llave maestra, para usar en sellado
tpm2_createprimary -c primary.ctx

## sellar el secreto bajo la política especificada arriba
echo 'secret' | tpm2_create -C primary.ctx -L pcr.policy -i- -c seal.ctx

## desellar, ya que PCRs todavía tienen los mismos valores
tpm2_unseal -c seal.ctx -p pcr:sha256:3
```

---

# Demo: Extender PCR y fallar unseal

```bash
## extender el PCR en el TPM
tpm2_pcrextend 3:sha256=a948904f2f0f479b8f8197694b30184b0d2ed1c1cd2a1ec0fb85d299a192a447

## unseal falla, ya que el valor PCR cambió
tpm2_unseal -c seal.ctx -p pcr:sha256:3
```

**El secreto ya no es accesible porque el estado del sistema cambió.**

---

# ¿Qué nos permite hacer esto?

Bajo el supuesto de que el adversario no manipula CPU, TPM, o su enlace:

---

# Attestation: Probar a otros qué software estás ejecutando

* Usar `TPM_quote()` para que el TPM firme un mensaje en tu nombre.
* Supuesto: la parte remota confía en tu TPM (pero no en ti directamente).
* TPM tiene su propia llave secreta, el fabricante de HW firma la llave pública, almacena cert en TPM.
* Típicamente llamado una **"attestation"**.

---

# Buen ajuste para configuraciones de dueño no confiable

* DRM, servidor de cómputo en la nube, etc.
* Puedes comunicarte con un dispositivo remoto, y saber que está ejecutando código esperado.
* Ej., versión correcta de Windows que no permite copiar datos de películas (DRM).
* Ej., algún VMM o bootloader confiable para ejecutar una VM en un servidor.

---

# Encriptar datos accesibles solo por software específico

* Usar `TPM_seal`, `TPM_unseal`.
* Los datos sellados solo pueden ser desencriptados por el destinatario elegido (PCR).
* Cada TPM tiene su propia llave generada aleatoriamente para encriptación.
* Ej., Bitlocker: dar llave al OS legítimo, dejar que el OS verifique credenciales del usuario.

---

# Modo TPM de Bitlocker

**Idea:** Almacenar llave en el TPM (o más bien, sellarla usando el TPM).

**Ventaja:** No hay necesidad de que el usuario interactúe con el BIOS.

---

# ¿Cuál es el punto del modo solo-TPM?

* La llave solo puede obtenerse si la máquina bootea el mismo OS (Windows).
* Como resultado, la seguridad se reduce a cualquier plan que tenga Windows.

---

# Opciones de seguridad con modo TPM

**Opción 1:** Usuario tiene contraseña de Windows.

¿Por qué es mejor que el enfoque de contraseña-en-BIOS?
1. No hay necesidad de ingresar contraseña dos veces: en BIOS y en Windows.
2. Windows puede limitar intentos de login, prevenir adivinación de contraseña.

**Opción 2:** Usuario no puede acceder a datos sensibles directamente.
* Usuario podría tener que acceder a datos sensibles vía proceso privilegiado.
* El proceso privilegiado no divulgará el conjunto completo de datos.

---

# ¿Qué se mide en el boot de BitLocker?

* Dos particiones en disco.
* **Primera partición:** contiene código de bootstrapping de BitLocker.
* **Segunda partición:** contiene datos encriptados ("volumen OS").
* Primera partición medida en boot.
* Llave de BitLocker sellada con medición PCR de la primera partición.

---

# ¿Por qué no medir la segunda partición?

* Cambia frecuentemente.
* Necesita un plan de "autenticación" más eficiente.
* Expectativa: el adversario no podrá cambiarla de manera significativa.

---

# ¿Qué pasa si necesitamos actualizar?

**Actualizar la primera partición:**
* Una posibilidad: re-sellar llave con nuevo valor PCR antes de actualizar.

**Actualizar laptops o recuperar de laptop muerto:**
* La llave de encriptación del disco está almacenada encriptada con una **contraseña de recuperación**.
* (O, almacenada en Active Directory encriptada con contraseña del admin.)
* El usuario puede escribir su contraseña de recuperación para ganar acceso al disco.

---

# ¿Cómo encriptamos el disco una vez que tenemos la llave?

* Encriptar bloques de disco (sectores) **uno a la vez**.
* ¿Por qué uno a la vez? **Atomicidad, rendimiento.**

---

# Problema potencial: Integridad

El adversario puede modificar sectores en disco.

¿Por qué es esto un problema para un esquema de encriptación de disco?

¿Por qué es insuficiente hacer secure boot (verificar firmas en código)?

---

# Opciones para asegurar integridad

Idealmente, almacenar un **MAC** (~hash con llave) para el sector en algún lugar del disco.

Recordar, los discos escriben sectores a la vez: necesita un MAC por sector.

---

# Problemas con almacenar MACs

* **Almacenar MAC en sector adyacente:** efectivamente corta espacio por factor de 2, y podría romper atomicidad si el disco falla entre 2 escrituras de sector.
* **Almacenar MAC en tabla en otro lugar:** dos seeks (y rompe atomicidad).
* **Almacenar MACs para grupo de sectores cerca:** rompe atomicidad.

---

# ¿Dónde almacenar MACs?

* Comprar discos muy caros (NetApp, EMC) que tienen sectores jumbo.
* Discos "Enterprise" tienen sectores de 520 bytes, en lugar del estándar 512.
* Los 8 bytes extra usados para almacenar checksums, IDs de transacción, etc.
* Podría usarse para almacenar MAC.
* **No va a funcionar para máquinas comunes.**

---

# Enfoque de Bitlocker: "Autenticación del Pobre"

Asumir que el adversario **no puede cambiar el ciphertext de manera "útil"**.

Es decir, no puede tener un efecto predecible en el plaintext.

---

# ¿Cuándo funcionaría o no la autenticación del pobre?

**Funciona si** las aplicaciones detectan o crashean cuando datos importantes están corruptos.

Debe ser verdad a nivel de sector, que el atacante puede corromper separadamente.

**Probablemente verdad para código:** instrucciones aleatorias levantarán una excepción.

---

# Peor caso para datos

* 1 bit (ej., "¿requiere login?") solo en un sector.
* El adversario puede adivinar ciphertexts aleatorios, ver cuándo ese bit cambia.
* Si la aplicación no nota otros bits corruptos, game over.
* Esperemos que el registro no esté construido así, entonces tal vez OK...

---

# ¿Cómo logra Bitlocker la autenticación del pobre?

* Modificar el esquema de encriptación simétrica.
* **Meta:** cambios localizados al ciphertext influencian el bloque completo.
* Los esquemas existentes no tienen esta propiedad (ej., encriptar 128 bits a la vez).
* Entonces, Bitlocker introduce un **paso de shuffling**; detalles no terriblemente relevantes aquí.

---

# ¿Qué hay de la frescura (freshness)?

Más difícil de lograr: necesita tener algún estado que no pueda ser revertido.

**Strawman:** hashear todos los bloques, almacenar hash en TPM.

**Problema:** actualizaciones requieren re-hashear todo el disco, lento.

**Idea:** árbol de hashes (árbol Merkle).

Incluso eso es frecuentemente muy costoso de actualizar.

---

# Ataques potenciales a BitLocker

No pretende ser una solución de seguridad perfecta, por diseño.

* **Ataques de hardware:** DMA, ataques cold boot, ...
* **Vulnerabilidades de seguridad en Windows** (buffer overflows, acceso root).
* **Revertir bloques de disco** a versión antigua (violar frescura).
  * El adversario probablemente no tiene bloques antiguos interesantes.
  * Difícil (costoso en términos de rendimiento) defenderse contra esto.

---

# Historial de seguridad de BitLocker

No se han encontrado vulnerabilidades en el diseño hasta ahora (8+ años), módulo modelo de amenaza.

Los ataques principalmente se enfocan en **extraer llave de la memoria del kernel de Windows**.

---

# Ataques de hardware a BitLocker

* DMA vía dispositivos como Firewire.
* Atacar interconexión entre CPU y TPM.

---

# Ataques de software a BitLocker

* Instalar un módulo de kernel como administrador; obtener dump de memoria.
* Requiere primero bypasear control de acceso en Windows de alguna manera.

**Meta era aumentar costo del ataque, y BitLocker parece tener éxito en ello.**

---

# Intel SGX: Protección más fuerte

Ideas similares aparecen en Intel SGX para proteger memoria contra ataques físicos.

* Encriptar contenidos de memoria.
* También hacer autenticación y frescura "de verdad".

---

# SGX: Ataques físicos subsumen ataques de software

* SGX usa encriptación de memoria para también proteger contra OS comprometido.
* El hardware provee un nuevo modo de ejecución llamado **"enclave"**.
* La memoria del enclave se encripta, autentica.
* Si el OS no confiable manipula memoria del enclave, es solo un caso especial de ataque físico.

---

# Alternativa a encriptación de sector: Encriptación a nivel de sistema de archivos

Ej., ecryptfs en Linux, usado por encriptación de directorio home de Ubuntu.

**Ventajas:**
* FS puede resolver problemas de atomicidad/consistencia.
* FS puede encontrar espacio para MACs extra, IVs aleatorios para prevenir reuso de IV, etc.

---

# Desventajas de encriptación a nivel de FS

* FS podría requerir mucho más código para ser "medido" en el TPM.
  * FS aparece mucho más tarde en el proceso de boot.
  * Requiere re-sellado del TPM para actualizaciones de FS, actualizaciones de drivers, etc.
* FS podría no interponerse en swapping/paging.
* FS más difícil de desplegar (cambios al FS, no se puede desplegar incrementalmente).

---

# Propiedades de seguridad deseadas

¿Qué otras propiedades de seguridad podrían querer los usuarios de encriptación de memoria/disco?

---

# Secreto de datos

**El adversario que obtiene acceso al disco no puede obtener datos.**

Bitlocker mayormente logra esto, módulo ser determinístico.

---

# Integridad de datos

**El adversario no puede reemplazar datos en disco sin detección.**

Bitlocker depende de "autenticación del pobre".

---

# Frescura de datos

**El adversario no puede revertir a versión antigua sin detección.**

Sin protección contra esto en Bitlocker.

Funciona para su modelo de amenaza.

---

# Resumen

* **Hardware confiable** permite nuevas garantías de seguridad contra ataques físicos.
* **TPM** proporciona measured boot y sellado de secretos basado en estado del sistema.
* **Bitlocker** usa TPM para encriptación de disco transparente al usuario.
* Trade-offs de diseño: integridad y frescura sacrificadas por rendimiento.
* **SGX** proporciona garantías más fuertes pero con más complejidad.

---

# Referencias

* Paper: "BitLocker Drive Encryption" - Microsoft
* TPM 2.0 Library Specification: https://trustedcomputinggroup.org/
* Intel SGX: https://www.intel.com/content/www/us/en/architecture-and-technology/software-guard-extensions.html
* Chrome OS Verified Boot: https://www.chromium.org/chromium-os/chromiumos-design-docs/verified-boot
