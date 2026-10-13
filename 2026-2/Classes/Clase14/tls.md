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

# Canal Seguro

---

# Canal Seguro
  * TCP/IP no proporciona autenticidad y confidencialidad
  * Usando criptografía, superponemos un canal seguro sobre TCP/IP
    * Caso de estudio: TLS/SSL
  * Los canales seguros son una base sólida para la seguridad
    * Ejecutar HTTP sobre SSL y la Web es "segura"
      * Convirtiéndose en el predeterminado
    * Ejecutar SMTP sobre SSL y la entrega de correo es "segura"
    * Ejecutar IMAP/POP sobre SSL y la recogida de correo es "segura"
    * Etc.
  * Área de seguridad bien entendida/desarrollada

---

# La criptografía es un mecanismo de seguridad poderoso

  * Cifrado: confidencialidad
  * Firmas: integridad
  * Dos tipos de llaves
    * Llave Simétrica
    * Clave pública vs clave privada
    

---

# Operaciones de clave simétrica:
  * KeyGen() -> clave privada K
    * Hay que hacerla llegar a ambos participantes
  * Encriptación
    * Encrypt(K, m) -> c
    * Decrypt(K, c) -> m
  * Firma
    * MAC(K, m) -> etiqueta t
  * Las claves para MAC y cifrado deben ser diferentes. 
    * Si necesitas tanto cifrado como MAC, necesitas dos claves.

---

# Operaciones de clave asimétrica:
  * KeyGen() -> clave pública PK, clave privada SK
    * PK se comparte; SK debe permanecer secreta
  * Encriptación
    * Encrypt(PK, m) -> c
    * Decrypt(SK, c) -> m
  * Firma
    * Sign(SK, m) -> firma sig
    * Verify(PK, m, sig) -> ¿válida?
  * La clave pública y la privada cumplen funciones complementarias
    * Una operación realizada con una se verifica o revierte con la otra

---

# Criptografía asimétrica: dos claves
  * Cada participante crea un par de claves relacionadas
    * **Clave pública:** se puede compartir con cualquiera
    * **Clave privada:** debe permanecer secreta
  * Conocer la clave pública no permite obtener la privada en un tiempo práctico
  * Es más lenta que la criptografía simétrica
    * Se usa para proteger datos pequeños o iniciar una conexión segura

---

# Cifrado con claves asimétricas
  * El receptor comparte su clave pública
  * El emisor cifra usando esa clave pública
  * Solo la clave privada del receptor puede descifrar
  * Objetivo: **confidencialidad**
    * Un tercero puede observar el tráfico, pero no leer el contenido
  * En SSL/TLS, ayuda a iniciar un canal seguro

---

# Firmas digitales
  * El emisor firma usando su clave privada
  * Cualquiera puede verificarla con la clave pública correspondiente
    * Primero debe confiar en que esa clave pertenece al emisor
  * Una firma válida demuestra que:
    * Fue creada por quien controla la clave privada correspondiente
    * El mensaje no fue modificado
  * Firmar no oculta el mensaje
  * En SSL/TLS, las firmas autentican al servidor y protegen el handshake

---

# SSL: de la criptografía a un canal seguro
  * Netscape creó SSL para proteger aplicaciones que usan TCP
    * Por ejemplo: HTTP, correo y otros protocolos
  * SSL se ubica entre la aplicación y TCP
  * Objetivos principales:
    * Autenticar al servidor
    * Mantener los datos confidenciales
    * Detectar modificaciones y repeticiones
  * Combina los mecanismos que acabamos de estudiar
    * Criptografía asimétrica al iniciar la conexión
    * Criptografía simétrica para proteger el resto del tráfico
  * Primero construiremos una versión simple y descubriremos qué le falta

---

# Establecer un canal seguro
  * Objetivo: establecer una clave compartida para criptografía simétrica
  * Strawman:
    * C --> S: conectar
    * C <-- S: mi clave pública es PK_S
    * C --> S: Encrypt(PK_S, freshKey)
    * C <-> S: Encrypt(freshKey, m...)

---

# Problema 1: autenticar el servidor
  * ¿Qué pasa si un adversario intercepta mensajes al servidor?
    * La vista del cliente es exactamente la misma
    * El cliente tendrá una clave compartida con el adversario
    * El adversario puede conectarse al servidor real y reenviar mensajes
    * Los mensajes subsecuentes parecen estar bien
    * "Ataque de hombre en el medio"
  * Necesitamos vincular PK_S a la identidad del servidor

---

# Solución: certificados
  * Servidor de autoridad confiable
  * Mantiene tabla de pares (nombre principal, clave pública)
    * El nombre principal es típicamente el nombre DNS del servidor
  * Otros participantes confían en que el servidor de autoridad tenga el mapeo correcto
    * Necesitan conocer la clave pública del servidor de autoridad para arrancar todo esto
  * Más sobre certificados en la próxima clase

---

# Solución: certificados (cont.)
  * Podría consultar el servidor, pero eso es malo para disponibilidad, rendimiento
  * En cambio, el servidor de autoridad puede firmar un mensaje:
    * Sign(SK_CA, {servidor, PK_server})
  * Este es el "certificado" del servidor
  * El servidor puede enviar este certificado al cliente
  * El cliente puede verificar que está correctamente firmado por PK_CA
  * El adversario no podría proporcionar tal certificado para PK_S del adversario

---

# Protocolo strawman revisado:
  * C --> S: conectar
  * C <-- S: PK_S, Sign(SK_CA, {servidor: PK_S})
  * C --> S: Encrypt(PK_S, freshKey)
  * C <-> S: Encrypt(freshKey, m...)

---

# Solución alternativa: confianza en el primer uso (TOFU), como en SSH
  * Asumir que la primera conexión está bien
  * Recordar el vínculo entre PK_S y el nombre principal del servidor para uso futuro
  * Más fácil de desplegar, menos propiedades de seguridad

---

# Problema 2: autenticar los mensajes
  * C <-> S: Encrypt(freshKey, m...)
  * Supongamos que el cliente envía la solicitud "Transferir $1 a la cuenta de Bob"
  * El cifrado garantiza confidencialidad pero no integridad
    * Una forma de pensarlo: Decrypt() siempre tiene éxito
    * El adversario manipula el texto cifrado -> Decrypt devolverá otra cosa
    * A veces efectos predecibles: voltear bit de texto cifrado -> voltear bit de mensaje
    * Nota que esto no rompe la confidencialidad
    * En nuestro ejemplo: voltear un bit para enviar $129 a Bob

---

# Solución: usar MAC() para calcular una etiqueta de autenticación
  * C <-> S: c=Encrypt(freshKey1, m...), t=MAC(freshKey2, c)
    * [ Nota: importante usar claves diferentes para Encrypt y MAC ]
  * Esto se llama "Cifrado autenticado"
  * La etiqueta será incorrecta: mensaje rechazado después de manipulación

---

# ¿Qué pasa si el adversario retransmite los mensajes?
  * No manipula el mensaje pero solo lo envía 129 veces
  * Desde la perspectiva del servidor: 129 mensajes válidos
  * Necesitamos defender contra ataques de replay

---

# Solución: mantener registro de mensajes previamente vistos
  * Descartar mensajes que ya han sido procesados
  * Dos enfoques comunes:
    * Número de secuencia en el mensaje
      * El receptor rastrea el último número de secuencia visto
      * Descartar mensajes con números de secuencia más antiguos
      * Bueno para protocolos orientados a flujo
    * Expiración / ID de sesión en el mensaje
      * El receptor rastrea TODOS los mensajes pasados dentro del tiempo de expiración o dentro de la sesión
      * Descartar mensajes previamente recibidos
      * Bueno para protocolos orientados a mensajes (no flujo)

---

# Problema 3: compromiso futuro del servidor
  * ¿Qué pasa si un atacante compromete el servidor más tarde?
  * Puede usar SK_S para descifrar paquetes de red antiguos
  * Esto a su vez revela el valor freshKey para tráfico antiguo
  * ¡Puede descifrar el contenido del tráfico antiguo!

---

# Solución: secreto hacia adelante (forward secrecy)
  * Para cada conexión, cliente y servidor crean claves temporales
  * Intercambian la parte pública de esas claves
    * El servidor autentica el intercambio usando su certificado
  * Ambos calculan de forma independiente la misma clave de sesión
    * La clave de sesión no se envía por la red
  * Al terminar la conexión, eliminan las claves temporales
  * Efecto:
    * Robar la clave permanente del servidor permite ataques futuros
    * Pero no permite descifrar conexiones antiguas que fueron grabadas

---

# De nuestro protocolo a SSL
  * Nuestro protocolo reúne las piezas fundamentales:
    * Autenticación del servidor mediante certificados
    * Una clave de sesión para cifrado simétrico
    * Integridad y protección contra replay
  * SSL combinó estas ideas en un protocolo para proteger aplicaciones sobre TCP
  * No apareció perfecto: cada generación corrigió problemas de la anterior

---

# De SSL a TLS
  * **SSL 2.0 (1995):** primera versión pública; tenía fallas de diseño graves
  * **SSL 3.0 (1996):** rediseño importante y base de TLS
  * **TLS 1.0 (1999):** estandarización de SSL 3.0 por la IETF
  * **TLS 1.1 (2006):** mitigaciones para ataques contra CBC
  * **TLS 1.2 (2008):** mejores algoritmos, hashes y suites AEAD
  * **TLS 1.3 (2018):** protocolo simplificado, handshake más rápido y defaults modernos
  * SSL 2.0, SSL 3.0, TLS 1.0 y TLS 1.1 están obsoletos

---

# SSL 2.0: por qué falló el primer intento
  * Partes importantes del handshake no estaban autenticadas
    * Un MITM podía borrar las opciones fuertes y forzar un algoritmo más débil
  * Usaba la misma clave para cifrado e integridad
    * Romper el cifrado débil también podía permitir falsificar mensajes
  * Usaba MD5 para autenticar mensajes
    * MD5 es un hash de 128 bits que luego mostró debilidades criptográficas graves
  * Las sesiones podían terminarse fácilmente
    * Un MITM podía insertar un TCP FIN y hacerlo parecer un cierre legítimo
  * Lección: usar criptografía no basta
    * También hay que autenticar la negociación y separar las claves por propósito

---

# SSL 3.0: mejoras sobre SSL 2.0
  * Autenticó el handshake completo
    * Los mensajes **Finished** detectaban cambios en la negociación
  * Separó las claves por dirección y propósito
    * Cliente y servidor usaban claves distintas para cifrado e integridad
  * Mejoró la protección de cada registro
    * Incluyó números de secuencia para detectar alteraciones, replay y reordenamiento
  * Agregó alertas autenticadas
    * **close_notify** permitía distinguir un cierre legítimo de un truncamiento
  * Separó el protocolo en componentes claros
    * Handshake, Record, Alert y ChangeCipherSpec


---

# De SSL 3.0 a TLS 1.0 y 1.1
  * SSL fue desarrollado originalmente por Netscape
  * La IETF tomó SSL 3.0 como base para crear un estándar abierto
    * IETF: comunidad internacional que desarrolla estándares abiertos para Internet
  * El nombre cambió a **Transport Layer Security (TLS)**
    * Señalaba que era un estándar de la IETF, no un producto de Netscape
  * **TLS 1.0 (1999):** sucesor directo de SSL 3.0
    * Estandarizó el protocolo e introdujo HMAC; no era compatible byte a byte
  * **TLS 1.1 (2006):** corrigió una debilidad de CBC
    * CBC mezcla cada bloque con el bloque cifrado anterior; el primero usa un IV
    * TLS 1.0 usaba el último bloque cifrado como IV del siguiente registro
    * TLS 1.1 envía un IV nuevo e impredecible con cada registro

---

# ¿Qué salió mal en SSL y TLS antiguo?
  * **Downgrades:** forzar una versión o algoritmo débil
    * Un MITM manipulaba la negociación o explotaba compatibilidad heredada
  * **CBC y padding oracles:** respuestas observables filtraban información
    * Errores o tiempos distintos permitían recuperar el texto plano byte a byte
  * **Algoritmos heredados:** RC4, 3DES, MD5 y SHA-1 envejecieron mal
    * Nuevos ataques y mayor poder de cómputo redujeron sus márgenes de seguridad
  * **RSA key transport:** no ofrecía secreto hacia adelante
    * Robar después la clave del servidor permitía descifrar sesiones grabadas
  * **Complejidad:** demasiadas versiones, opciones y combinaciones
    * Cada alternativa agregaba configuraciones incorrectas e interacciones peligrosas


---

# TLS 1.2: transición hacia diseños modernos
  * Permitió usar algoritmos criptográficos más fuertes
    * Por ejemplo: SHA-256 para hashes y mejores firmas digitales
  * Permitió proteger sesiones pasadas si una clave era robada en el futuro
    * Ejemplo: intercambio de claves temporales mediante ECDHE
  * Permitió cifrar y verificar integridad como una sola operación
    * Ejemplo: AEAD mediante AES-GCM
  * Problema: estas mejoras eran opcionales
    * También conservaba RSA key transport, CBC y muchas configuraciones antiguas
  * Resultado: podía ser seguro, pero dependía de configurarlo correctamente

---

# TLS 1.3: seguridad moderna por defecto
  * Eliminó opciones antiguas que causaban ataques y malas configuraciones
    * Por ejemplo: RSA key transport, CBC, RC4, 3DES y compresión TLS
  * Hizo normal el secreto hacia adelante
    * Usa claves temporales; ECDHE es el ejemplo más común
  * Exigió cifrado con integridad incorporada
    * Solo permite AEAD, como AES-GCM o ChaCha20-Poly1305
  * Protegió mejor la negociación completa
    * Autentica el transcript y cifra gran parte del handshake
  * Redujo la latencia y la complejidad
    * El handshake normal necesita un solo viaje de ida y vuelta


---

# TLS no resuelve todo
  * Protege datos **en tránsito** entre los extremos de la conexión
  * No protege contra:
    * Un servidor o cliente comprometido
    * Vulnerabilidades de la aplicación
    * Phishing o un dominio engañoso con certificado válido
    * Datos robados antes de cifrar o después de descifrar
  * TLS 1.3 0-RTT permite replay y requiere cuidado a nivel de aplicación
  * Un canal seguro no convierte automáticamente a la aplicación en segura


---

# Apéndice: detalles de RSA

Fórmulas, ejemplos numéricos y costo de factorización.

---

# RSA: elegir los primos y calcular $n$
  * Elegir dos números primos secretos y distintos: $p$ y $q$
    * Ejemplo: $p=5$ y $q=11$
  * Multiplicarlos para calcular $n$:
    * $n=pq=5\cdot11=55$
  * Calcular $\lambda(n)$:
    * $\lambda(n)=\operatorname{mcm}(p-1,q-1)$
    * $\lambda(55)=\operatorname{mcm}(4,10)=20$


---

# RSA: elegir $e$ y calcular $d$
  * Como $\lambda(n)=20$, elegir $e$ entre 1 y 20
  * $e$ no puede compartir factores con 20, son coprimos
    * Elegimos $e=3$: $\operatorname{mcd}(3,20)=1$
  * Esto garantiza que $e$ tenga un inverso modular: $d$
    * Un inverso modular de un número **e** es otro número **d** que, al multiplicarse por **e**, produce resto 1 al dividir por un módulo determinado.
  * En nuestro ejemplo, buscar $d$ tal que $3d$ deje resto 1 al dividir por 20
    * $3\cdot7=21$ y $21\bmod20=1$; por lo tanto, $d=7$
  * Clave pública: $(n,e)=(55,3)$
  * Clave privada: $d=7$

---

# RSA: cifrado y descifrado
  * Mensaje: $m$, con $0\leq m<n$
  * Cifrado con la clave pública:
    * $c=m^e\bmod n$
  * Descifrado con la clave privada:
    * $m=c^d\bmod n$
  * Como $ed\equiv1\pmod{\lambda(n)}$:
    * $(m^e)^d\equiv m\pmod n$
  * Aplicar $d$ invierte la operación definida por $e$
    * Por eso $d$ debe permanecer secreto

---

# Ejemplo RSA: generar las claves
  * Primos secretos: $p=5$ y $q=11$
  * $n=pq=55$
  * $\lambda(n)=\operatorname{mcm}(4,10)=20$
  * Elegimos $e=3$
    * $\operatorname{mcd}(3,20)=1$: 3 y 20 son coprimos
  * Calculamos $d=7$
    * $3\cdot7=21\equiv1\pmod{20}$
  * Clave pública: $(n,e)=(55,3)$
  * Clave privada: $d=7$

---

# Ejemplo RSA: cifrar y descifrar
  * Mensaje: $m=8$
  * Cifrado con la clave pública:
    * $c=8^3\bmod55=17$
  * Descifrado con la clave privada:
    * $m=17^7\bmod55=8$
  * $d=7$ permite invertir la operación definida por $e=3$
    * Quien conoce $d$ puede descifrar; por eso es secreto
  * Este ejemplo no es seguro: $55=5\times11$ es trivial de factorizar

---

# RSA con números un poco más grandes
  * $p=10.007$
  * $q=10.009$
  * $n=pq=100.160.063$
  * $\lambda(n)=50.070.024$
  * $e=65.537$
  * $d=35.910.881$
  * Mensaje: $m=12.345$
  * Cifrado: $c=85.109.818$
  * Descifrado: $c^d \bmod n=12.345$

---

# ¿Qué tan difícil es factorizarlo?
  * Algoritmo clásico más simple: **división por tentativa**
    1. Probar cada número primo $r$ desde 2 hasta $\sqrt{n}$
    2. Cuando $n \bmod r=0$, los factores son $r$ y $n/r$
  * En RSA sabemos de antemano que $n=pq$ es compuesto
    * Uno de los dos factores debe ser menor o igual que $\sqrt{n}$
  * Para nuestro ejemplo: $\sqrt{100.160.063}\approx10.008$
    * Solo hay que probar los aproximadamente 1.229 primos menores que 10.008
  * Para RSA-2048, $\sqrt{n}$ tiene aproximadamente 309 dígitos
  * Estimación de seguridad clásica: alrededor de $2^{112}$ operaciones
    * Incluso a $10^{18}$ operaciones por segundo: aproximadamente 160 millones de años

---

# Firmar no es cifrar
  * Cifrar busca **confidencialidad**
    * Solo quien tiene la clave privada puede recuperar el mensaje
  * Firmar busca **autenticidad e integridad**
    * El mensaje sigue siendo visible
  * Para firmar:
    * Calcular $h=Hash(m)$
    * Calcular $s=Sign(SK,h)$
    * Enviar el mensaje $m$ junto con la firma $s$
  * Solo quien posee la clave privada puede crear una firma válida

---

# Verificar una firma
  * El receptor recibe el mensaje $m$ y la firma $s$
  * Calcula nuevamente $h=Hash(m)$
  * Usa la clave pública para verificar $s$
  * Si la firma es válida:
    * El mensaje no fue modificado
    * La firma fue creada con la clave privada correspondiente
  * Cualquiera puede verificar; solo el dueño de la clave privada puede firmar
  * RSA real usa un esquema seguro como RSA-PSS
