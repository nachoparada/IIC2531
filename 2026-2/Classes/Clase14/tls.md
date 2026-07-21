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
    * Clave pública vs clave privada
    * Llave Simétrica

---

# Operaciones de clave pública:
  * KeyGen() -> clave pública PK, clave secreta SK
  * Encriptación
    * Encrypt(PK, mensaje m) -> texto cifrado c
    * Decrypt(SK, c) -> m
  * Firma
    * Sign(SK, m) -> firma sig
    * Verify(PK, m, sig) -> ¿ok?

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
  * ¿Qué pasa si un malo compromete el servidor más tarde?
  * Puede usar SK_S para descifrar paquetes de red antiguos
  * Esto a su vez revela el valor freshKey para tráfico antiguo
  * ¡Puede descifrar el contenido del tráfico antiguo!

---

# Solución: secreto hacia adelante (forward secrecy)
  * No usar claves de largo plazo para cifrado
    * Susceptible al descifrado por un adversario futuro
  * En cambio, usar claves de corta duración para cifrado, y claves de largo plazo para firmar
    * Un adversario futuro puede falsificar firmas en el futuro, pero es demasiado tarde
  * Strawman de secreto hacia adelante para nuestro protocolo:
    * C --> S: conectar
    * C <-- S: PK_conn, Sign(SK_CA, {servidor: PK_S}), Sign(SK_S, PK_conn)
    * C --> S: Encrypt(PK_conn, freshKey)
    * C <-> S: Encrypt(freshKey, m...)

