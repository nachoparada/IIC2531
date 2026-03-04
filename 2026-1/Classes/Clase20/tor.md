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

# Tor: The Onion Router

---

# ¿Cuál es el objetivo de Tor?

* **Anonimato para clientes** que quieren conectarse a servidores en internet.
* **Anonimato para servidores** que quieren servir solicitudes de usuarios.

---

# ¿Qué es anonimato?

* El adversario **no puede determinar** qué usuarios se están comunicando con qué servidores.
* El adversario (muy probablemente) **sabe** que usuarios y servidores se están comunicando vía Tor.
* Es decir, Tor **no está diseñado** para prevenir que el adversario encuentre usuarios de Tor.

---

# ¿Cómo lograr anonimato?

* Debe **encriptar tráfico** hacia/desde la persona que quiere ser anónima.
  * De otra forma, el adversario mira los paquetes y descubre qué está pasando.
* Pero la encriptación **no es suficiente**: aún se podría rastrear a dónde fueron los paquetes encriptados.

---

# Mezclar tráfico

* **Mezclar** el tráfico de un usuario con tráfico de otros usuarios (o "tráfico de cobertura").
* El anonimato para un usuario requiere tener **muchos otros usuarios similares** al primero.
  * Si todos los otros usuarios solo usan BitTorrent, el usuario de Wikipedia es fácil de detectar.
  * Si todos los otros usuarios usan Firefox, un usuario de Chrome sería fácil de detectar.

---

# Enfoque básico

* El adversario no podría determinar qué usuario inició qué conexiones.
* El componente de mezcla debe **cambiar los paquetes** (ej., encriptar/desencriptar).
  * De otra forma, se puede buscar dónde aparece el mismo paquete exacto después.
* **Enfoque:** retransmitir tráfico vía intermediario que encripta/desencripta.

---

# ¿Por qué necesitamos más de un nodo?

**Escalabilidad:** manejar más tráfico que un solo nodo.

**Compromiso:** el atacante aprende info sobre clientes directos del nodo comprometido.
  * Con muchos nodos independientes, esto afecta solo una pequeña fracción del tráfico.
  * Con onion routing, el atacante debe comprometer **todos los nodos** en la cadena.

---

# Análisis de tráfico

**Análisis de tráfico:** el atacante puede correlacionar tráfico entrante/saliente.
  * Puede mirar timing entre paquetes, o volumen de paquetes.
  * El encadenamiento hace que el ataque de análisis de timing/volumen sea más difícil.

---

# ¿Puede el atacante aún tener éxito?

**Sí**, si observa o compromete suficientes nodos.
  * Por ejemplo, puede ser suficiente observar el primer y último nodo.
  * El atacante también puede inyectar información de timing (retrasando paquetes) para analizar.

---

# Idea principal: Onion Routing

* Malla de **onion routers (ORs)** en la red.
* **Supuesto:** el cliente conoce las llaves públicas de todos los ORs.
* El cliente **escoge alguna ruta** a través de esta red.

---

# Strawman de Onion Routing (no exactamente Tor)

1. El cliente encripta el mensaje con la llave pública de cada OR en la ruta, en orden.
2. Envía el mensaje al primer OR en la ruta, que desencripta y retransmite, y así sucesivamente.
3. El **"exit node"** (último OR en la ruta) envía los datos a la red real.

---

# ¿Por qué es un buen diseño?

* Cada OR conoce el **hop anterior y siguiente**, no el origen o destino final.
* Con dos ORs, comprometer un solo OR **no rompe el anonimato**.

---

# ¿A qué nivel debemos retransmitir?

Podríamos hacer cualquier nivel:
* Paquetes IP
* Conexiones TCP
* Nivel de aplicación (HTTP)

---

# Ventajas y desventajas de cada nivel

**Nivel más bajo (IP):**
* Más general, menos cambios a apps, funciona con más apps.

**Nivel más alto (TCP, HTTP):**
* Más eficiente, más anónimo.

---

# ¿Qué hace Tor?

* **Retransmisión a nivel TCP**, usando SOCKS (intercepta llamadas libc).

**Ejemplos de eficiencia:**
* No necesita control de flujo TCP, Tor hace retransmisión.

**Ejemplos de generalidad perdida:**
* UDP no funciona, no se puede hacer traceroute, ...

---

# ¿Cómo funciona DNS con Tor?

* SOCKS puede capturar el **hostname** del destino, no solo dirección IP.
* El exit node realiza el lookup DNS, establece conexión TCP.

---

# Anonimato perdido en capas inferiores

* Si hiciéramos IP, filtraríamos mucha info TCP (seq#, timestamp).
* Si hiciéramos TCP, filtraríamos todo tipo de headers HTTP y cookies.
* Si hiciéramos HTTP, se puede violar anonimato vía Javascript, Flash, ...
  * Muchas características identificables en el ambiente Javascript.
  * Versión del navegador, sniffing de historial, direcciones/servidores de red local...

---

# Normalización de protocolo

* **"Protocol normalization":** arreglar todos los grados de libertad en el protocolo superior.
* Difícil de hacer en práctica; proxies específicos de app son útiles (ej., Privoxy).

**Demo de "identificación" de navegador:** https://panopticlick.eff.org/

---

# Diseño de Tor

* Malla de ORs: cada OR conectado vía SSL/TLS a cada otro OR.
  * No necesita certificados SSL/TLS firmados por CA.
  * Tor tiene su propio plan de verificación de llave pública usando servidores de directorio.
* Los ORs mayormente son ejecutados por **voluntarios**: ej., MIT ejecuta varios.

---

# Componentes del cliente

* Los usuarios finales ejecutan un **onion proxy (OP)** que implementa SOCKS.
* OR tiene dos llaves públicas: **identity key** y **onion key**.
* Identity key registrada con directorio, firma estado del OR.
* Onion key usada por OPs para conectarse a ORs, construir circuitos.

---

# Construcción de circuitos

* El cliente descarga lista de ORs del directorio.
* Escoge una cadena de ORs para formar circuito, contacta cada OR en orden.

**¿Por qué el cliente construye los circuitos?**
* Cualquier servidor individual podría estar comprometido, no se puede confiar.
* Es inevitable confiar en la máquina del cliente.

---

# ¿Por qué necesitamos onion key además de identity key?

* Podría ser posible proteger la identity key de compromisos a largo plazo.
* Cada OR usa la identity key para firmar su onion key actual.

---

# ¿Por qué Tor necesita un directorio?

* Alguien necesita **aprobar ORs**.
  * De otra forma el atacante puede crear muchos ORs, monitorear tráfico.

**¿Tener un directorio compromete el anonimato?**
* No, no necesitas consultarlo en línea.

---

# Problemas con el directorio

**¿Qué pasa si un directorio está comprometido?**
* Los clientes requieren que la **mayoría de directorios** estén de acuerdo.

**¿Qué pasa si muchos directorios están comprometidos?**
* El atacante puede inyectar muchos ORs, monitorear tráfico.

**¿Qué pasa si los directorios están desincronizados?**
* El atacante podría identificar usuarios basándose en la info del directorio que vieron.

---

# Terminología: Circuitos y Streams

**Circuito:** una ruta a través de una lista de ORs que un cliente construye.
* Los circuitos existen por algún período de tiempo (quizás unos minutos).
* Nuevos circuitos se abren periódicamente para frustrar ataques.

**Stream:** efectivamente una conexión TCP.
* Muchos streams corren sobre el mismo circuito (cada uno con ID de stream separado).
* Los streams son una optimización importante: no hay necesidad de reconstruir circuito.

---

# ¿Por qué Tor necesita circuitos?

**¿Qué sale mal si tenemos circuitos de larga duración?**
* El adversario podría correlacionar múltiples streams en un solo circuito.
* Vincular conexiones de un solo usuario a diferentes sitios, romper anonimato.

---

# Circuitos de Tor

Un circuito es una secuencia de ORs, junto con llaves simétricas (AES) compartidas.
* ORs: c_1, c_2, .., c_n
* Llaves: k_1, k_2, .., k_n

---

# Formato de celda

```
+---------+---------------+-----------+
| Circuit | Control/Relay |   DATA    |
+---------+---------------+-----------+
  2 bytes      1 byte       509 bytes
```

* Los campos "Circuit" y "Control/Relay" son como headers de capa de enlace.
* Los IDs de circuito son **por-enlace** (entre pares de ORs).

---

# Multiplexación de circuitos

* Usado para multiplexar muchos circuitos en la misma conexión TLS entre ORs.
* **Mensajes de control** son "link-local": enviados solo a un vecino inmediato.
* **Mensajes relay** son "end-to-end": retransmitidos a lo largo del circuito.

**¿Por qué todo el tráfico en celdas de tamaño fijo?**
* Hace el análisis de tráfico más difícil.

---

# Comandos de control

* **padding:** keepalive o padding de enlace.
* **create/created/destroy:** crear y destruir circuitos.

---

# Comandos relay

**Si el paquete relay está destinado al nodo actual:**

```
+----------+--------+-----+-----+-----------+
| StreamID | Digest | Len | CMD | RelayData |
+----------+--------+-----+-----+-----------+
  2 bytes    6 bytes   2     1    498 bytes
```

**Si el paquete relay está destinado a otro nodo:**

```
+-------------------------------------------+
| Datos encriptados, opacos                 |
+-------------------------------------------+
                   509 bytes
```

---

# ¿Cómo envía datos el OP vía circuito?

1. Componer paquete relay como arriba (aún no encriptado).
2. Calcular un checksum válido (digest).
   * El digest se basa en el OR objetivo que debería desencriptar el paquete.
   * El hash se toma sobre alguna función de llave + todos los msgs intercambiados con ese OR.
3. Encriptar con AES(k_n), luego AES(k_{n-1}), .., AES(k_1).
4. Enviar celda encriptada al primer OR (c_1).

---

# ¿Qué hace un OR con paquetes relay?

* Si viene de la dirección del OP: **desencriptar** y reenviar lejos del OP.
* Si no viene de la dirección del OP: **encriptar** y reenviar hacia el OP.

---

# ¿Cómo sabe un OR si un paquete relay está destinado a él?

* **Verificar checksum:** si coincide, muy probablemente es para el OR actual.
* **Optimización:** los primeros 2 bytes del digest deberían ser cero.
  * Si los primeros dos bytes no son cero, se puede saltar el hashing: no es nuestro paquete.
* Si el checksum no coincide, no es para este OR, seguir retransmitiendo.

---

# Propiedades del diseño

* El tamaño del paquete es **independiente de la longitud de la ruta**.
* Solo el **último OR** conoce el destino.

---

# ¿Cómo establecer un nuevo stream?

* El OP envía un **"relay begin"** vía circuito. Contiene hostname destino, puerto.
* El OP puede escoger ID de stream arbitrario en su circuito.

---

# Topología "leaky pipe"

* El OP puede enviar mensajes relay a **cualquier OR** a lo largo de su circuito (no solo el último OR).
* Puede construir stream (es decir, conexión TCP) vía cualquier OR, para frustrar análisis de tráfico.

---

# Inicialización de circuitos

* El OP escoge la secuencia de ORs para usar en su circuito.
  * ¿Por qué el OP hace esto? Resistencia a otros ORs "desviando" el circuito.
* Conectar al primer OR, emitir operación **"create"** para crear circuito.
  * Create incluye mensaje de intercambio de llaves DH.
  * Respuesta Created incluye respuesta de intercambio DH.

---

# Protocolo de intercambio de llaves

```
[ OP, OR acuerdan primo p, generador g ]

OP escoge x aleatorio.
OP envía E_{PK_OR}(g^x).

OR escoge y aleatorio.
OR calcula K = g^xy.
OR responde con g^y, H(K || "handshake").

OP calcula K = g^xy.
```

---

# Autenticación en el protocolo

* **¿Cómo autenticamos a las partes?**
  * El primer mensaje DH está encriptado con la onion key del OR.
  * El hash de la llave en la respuesta DH prueba al cliente que el OR correcto desencriptó el msg.
  * **El servidor no autentica al cliente** — ¡anonimato!

* **Forward secrecy:** ¿qué? ¿cómo?
* **Frescura de llave:** ¿por qué? ¿cómo?

---

# IDs de circuito

* **¿Quién escoge el ID del circuito?**
  * El extremo cliente de la conexión TLS (no el OP del circuito general).
  * Cada circuito tiene un ID diferente para cada enlace que atraviesa.

---

# Extender el circuito

* Para cada OR subsiguiente, el OP envía un mensaje **"relay extend"** vía circuito.
  * Incluir el mismo mensaje de intercambio DH en la celda "relay extend".
  * Al final del circuito, "relay extend" se transforma en "create".
  * El cliente termina con llave simétrica (AES) compartida para cada OR en el circuito.

---

# ¿Por qué celdas de control separadas vs celdas relay?

* Asegura que las celdas sean **siempre de tamaño fijo**.
* El último OR en el circuito viejo necesita saber los IDs del nuevo OR y circuito.

---

# Estado que mantiene cada OR por circuito

* ID de circuito y OR vecino para **dos direcciones** en el circuito (hacia/desde OP).
* Llave compartida con OP para este circuito y este OR.
* Estado SHA-1 para cada circuito.

---

# ¿Podemos evitar almacenar todo este estado en la red?

* No sin un descriptor de ruta de longitud variable en cada celda.
* El exit node igualmente necesitaría un descriptor de ruta para saber cómo enviar de vuelta.
* Los nodos intermedios necesitarían realizar crypto de llave pública (costoso).

---

# ¿Por qué Tor necesita políticas de salida?

* Prevenir abuso (ej., enviar spam anónimamente).
* Las políticas de salida son similares a reglas de firewall (ej., no puede conectar al puerto 25).
  * Cada exit node verifica la política de salida cuando se abre una nueva conexión.

**¿Por qué publicar la política de salida en el directorio?**
* No se usa para enforcement.
* El OP necesita saber qué exit nodes probablemente funcionarán.

---

# ¿Qué pasa si Tor no hiciera verificación de integridad?

* Se necesita integridad para prevenir un **ataque de tagging**.
* El atacante compromete un nodo interno, corrompe paquetes de datos.
* Los paquetes corruptos eventualmente serán enviados, se puede ver a dónde van.

---

# ¿Cómo previene Tor los replays?

* Cada checksum es en realidad checksum de **todas las celdas previas** entre OP y OR.
* El checksum para los mismos datos enviados de nuevo sería diferente.
* Funciona bien porque el transporte subyacente es confiable (SSL/TLS sobre TCP).

---

# Servicios Anónimos (Hidden Services)

* Los hidden services se nombran por llaves públicas (pseudo-DNS: `publickey.onion`).

---

# ¿Por qué la división entre punto de introducción y rendezvous?

**Evitar colocar carga de tráfico en los puntos de introducción.**

**Evitar que el punto de introducción transfiera datos conocidamente ilegales.**

---

# La división previene ambos problemas

1. **Bob (servicio)** tiene un **punto de introducción (IP)**.
2. **Alice** escoge un **punto de rendezvous (RP)**, le dice al IP de Bob sobre el RP.
3. El punto de introducción **no retransmite datos**.
4. El punto de rendezvous **no sabe qué datos está retransmitiendo**.

---

# ¿Por qué Bob se conecta de vuelta a Alice?

* Control de admisión, distribuir carga sobre muchos puntos de rendezvous.

**¿Qué es la cookie de rendezvous?**
* Permite a Bob probar al RP de Alice que es Bob.

---

# Cookie de autorización

* Algo que podría obligar a Bob a responder, cuando de otra forma no lo haría.
* Tal vez una palabra secreta que la mayoría de la gente no conoce.
* Limita ataques DoS al servidor de Bob (pueden solo enviar muchas cookies).
* Almacenada en hostname: `cookie.pubkey.onion`.

---

# Estado final del hidden service

* Dos circuitos al RP, con un stream conectado entre ellos.
* El RP toma celdas relay del stream de un circuito y las envía en un stream del otro circuito.
* Los datos puenteados están encriptados usando llave compartida entre Alice y Bob (DH).
* Cada uno puede controlar su propio nivel de anonimato.
* Ninguno conoce la ruta completa del otro circuito.

---

# Peligros potenciales al usar Tor

* **Fugas a nivel de aplicación** (Javascript, headers HTTP, DNS, ...)
  * Usar un proxy de nivel de app (ej., Privoxy elimina muchos headers HTTP).
* **Fingerprinting** basado en comportamiento del cliente Tor (cada cuánto se abre nuevo circuito).
* **Análisis de timing/volumen** (defensa parcial es ejecutar tu propio Tor OR).

---

# Más peligros

* **Fingerprinting de sitios web:** número de requests y tamaños de archivo de sitios populares.
  * La cuantización de celdas de tamaño fijo ayuda un poco.
* **ORs maliciosos:** unirse a la red, anunciar mucho ancho de banda, política de salida abierta.

---

# Beneficios y riesgos de ejecutar un OR

**Beneficios:**
* Más anonimato

**Riesgos:**
* Uso de recursos
* Ataques en línea (DoS, break-ins, ...)
* Ataques offline (ej., máquina confiscada por autoridades)

---

# ¿Qué tan difícil es bloquear Tor?

* Encontrar lista de IPs de ORs del directorio, bloquear todo el tráfico hacia ellos.

**¿Cómo defenderse contra tal ataque?**

---

# Defensas contra bloqueo

**¿Revelar diferentes ORs a diferentes clientes?**
* Permite fingerprinting de cliente basado en ORs usados.

**¿Mantener algunos ORs no listados?**
* Quieres usar ORs no listados solo como primer hop, para evitar fingerprinting.
* Tor tiene noción de nodo **"bridge"**, que es un OR no listado.

---

# ¿Cómo encontrar nodos "bridge"?

* Quieres que el usuario legítimo los encuentre, pero no dejar que el adversario los enumere.

**Enfoque de Tor:** directorio especial de bridges.
* Revelar 3 bridges a cada IP (via HTTP) o dirección de email (via email).
* Revelar nuevos bridges a la misma dirección de cliente solo después de 24 horas.
* Puede rate-limit por IP, encontrar intentos de enumerar base de datos de bridges, etc.

---

# Bridges y email

* Para email, es más fácil para el adversario crear identidades falsas (direcciones de email).
* Tor confía en 3 proveedores de correo para rate-limit de signup (gmail, yahoo, mit).

---

# ¿Usarías Tor?

* Podría ser muy lento para usar en todo el tráfico (alta latencia).
* Pero desafortunadamente eso significa que solo el tráfico sensible iría vía Tor.
* Existen ataques plausibles, entonces no es ideal contra adversarios muy poderosos.
* Tal vez una buena forma de evitar ataques de denegación de servicio (descargar a Tor).

---

# ¿Qué tan activo es Tor?

* Uso mucho más activo ahora que lo que describe el paper.
* ~3000 ORs públicos, ~1000 exit nodes, ~1000 bridge nodes, ~2GB/s ancho de banda OR.
* 8-9 servidores de directorio, ~1600 mirrors de directorio.
* **Problemas difíciles:** distribuir IPs de puntos de entrada, aprobar ORs, ...

---

# Enfoque alternativo: DC-nets

**"Dining cryptographer networks"**

* N participantes, pero supongamos que solo hay un emisor (no se sabe quién).
* Cada par de participantes comparte un bit secreto.
* Para transmitir un bit "0", enviar XOR de todos los secretos. De otra forma, enviar lo opuesto.
* Todas las transmisiones son públicas: para recuperar el bit, XOR las transmisiones de todos.

---

# DC-nets (cont.)

* Se puede construir para enviar múltiples bits, usar protocolo de detección de colisión, etc.
* **Costoso en términos de rendimiento**, pero proporciona seguridad mucho más fuerte que Tor.
* Ver el paper de Dissent OSDI 2012 para más detalles sobre un sistema basado en DCnet.

---

# Problema: Adversario controla primer y último hop

* Si el adversario controla el primer y último hop, puede identificar circuito de 3-hops.
* Buscar endpoints de circuito que vienen hacia/desde el mismo relay intermedio.
* Monitorear timing de paquetes enviados en primer hop, ver si coinciden en último hop.
* Para un adversario, más o menos suficiente controlar primer y último hop.

---

# Probabilidad de circuito comprometido

Supongamos que hay M nodos relay maliciosos, de un total de N nodos.

* Si escogemos un circuito aleatorio cada vez, comprometido si primero+último son maliciosos.
* Probabilidad M/N de primer relay malicioso × M/N de último relay malicioso.
* A medida que construimos más circuitos C, esta probabilidad crece, aproximadamente **C×(M/N)²**.
* Sigue creciendo linealmente con el tiempo, eventualmente llegando a 1.

---

# Guard Nodes

**Insight:** mejor escoger el primer nodo una vez, y mantenerlo por mucho tiempo.

* Si escogimos un primer nodo honesto ("guard node"), estamos bien en términos de privacidad.
* O, probabilidad M/N de que escogimos mal, y entonces nos preocupamos por probabilidad de último malo.
* Entonces, a medida que construimos más circuitos, la probabilidad total de compromiso llega a **M/N pero no a 1**.

---

# Censura y bridges

* El ISP o país podría bloquear relays Tor conocidos.
* **Idea:** nodos "bridge" no tan conocidos que proveen conexión inicial a la red Tor.
* El servicio de directorio entrega unas pocas direcciones IP de bridges a la vez, no enumeración completa.
* También **"pluggable transports"** para ayudar a desarrollar rápidamente nuevas formas de bypasear censura.

---

# Demo: Tor Browser

```bash
# pacman -S torbrowser-launcher
% torbrowser-launcher
```

* https://web.mit.edu/
  * [click en el ícono de circuito junto a la barra de URL]
* http://keybase5wmilwokqirssclfnsqrjdsi7jdir5wy7y7iu3tanwmtp6oid.onion/
  * [click en el ícono de circuito junto a la barra de URL]
* https://metrics.torproject.org/

---

# Resumen

* **Tor** proporciona anonimato mediante onion routing con múltiples capas de encriptación.
* Los **circuitos** se construyen incrementalmente, cada OR solo conoce vecinos.
* **Hidden services** permiten servidores anónimos usando puntos de introducción y rendezvous.
* **Guard nodes** reducen la probabilidad de compromiso a largo plazo.
* **Bridges** ayudan a evadir censura.

---

# Referencias

* Paper: "Tor: The Second-Generation Onion Router" - Dingledine, Mathewson, Syverson
* https://metrics.torproject.org/
* https://www.torproject.org/
* https://panopticlick.eff.org/ (fingerprinting de navegador)
* DC-nets: http://dedis.cs.yale.edu/2010/anon/papers/osdi12.pdf
