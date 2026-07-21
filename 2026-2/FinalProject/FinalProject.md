# Proyecto Final

Al hacer el proyecto, puedes elegir qué construir, sujeto a nuestra aprobación. El proyecto puede realizarse individualmente, o puedes formar un grupo de 2 a 3 estudiantes para colaborar ya que esto a menudo permite proyectos más interesantes. Entregarás tu código y un breve informe describiendo el diseño e implementación de tu proyecto, y harás una breve presentación en clase sobre tu trabajo.

El requisito principal para las ideas de proyecto es que sean **interesantes**. También deben estar al menos tangencialmente relacionadas con seguridad, y deben ser de tamaño y dificultad razonable para un proyecto de curso.

Fomentamos proyectos finales que aprovechen múltiples cursos que puedas estar tomando, o que involucren otra investigación o proyectos en los que ya estés trabajando. Por ejemplo, si también estás tomando otro curso de ingeniería o computación, estaría bien tener un proyecto que se beneficie de lo que están haciendo en la otra clase.

## Entregables

| Entrega | Fecha |
|---------|-------|
| Propuesta del proyecto | 29 de Abril, 2026 |
| Presentación final + Informe | 22 de Junio, 2026 |

Hay tres pasos para realizar un proyecto final:

1. **Propuesta del proyecto — 29 de Abril**
   Discute tu idea propuesta con el equipo docente para definir el problema exacto que abordarás, cómo lo harás, y qué herramientas podrías necesitar en el proceso. Entrega una propuesta de una a dos páginas describiendo: la lista de miembros de tu grupo, el problema que quieres abordar, cómo planeas abordarlo, y qué propones específicamente diseñar e implementar.
   
   *Esta entrega no es calificada, pero es obligatoria para recibir feedback y asegurar que tu proyecto va por buen camino.*

2. **Presentación del proyecto — 22 de Junio**
   Prepara una breve presentación en clase sobre el trabajo que has realizado para tu proyecto final. Dependiendo del número de grupos y los tipos de proyectos que cada grupo elija, podríamos decidir limitar el número total de presentaciones.

3. **Informe y código — 22 de Junio**
   Escribe un documento describiendo el diseño e implementación de tu proyecto, y entrégalo junto con el código de tu proyecto (en caso de que aplique). El documento debe ser de aproximadamente 5-10 páginas de texto que nos ayude a entender qué problema resolviste, y qué hace tu código.

## Nota sobre proyectos orientados a ataques

Si estás interesado en un proyecto final más orientado a ataques, tu objetivo para este proyecto es elegir un servicio interesante y tratar de encontrar vulnerabilidades en él. Juzgaremos tu proyecto basándonos en qué tipos de vulnerabilidades encuentres. Ten en cuenta que no hay garantía de éxito con este (o cualquier otro proyecto orientado a ataques), porque podrías accidentalmente elegir un servicio muy seguro, y podrías terminar sin encontrar vulnerabilidades.

Sin embargo, si tienes un enfoque interesante para encontrar vulnerabilidades (por ejemplo, has diseñado una nueva herramienta para encontrar bugs), podrías recibir una buena calificación incluso sin encontrar vulnerabilidades reales.

**IMPORTANTE:**
En cualquier proyecto orientado a ataques, debes ser muy cuidadoso de evitar:
- Interrumpir servicios existentes
- Incomodar a los usuarios de esos servicios
- Comprometer la seguridad de ese servicio
- Aprovechar cualquier vulnerabilidad que encuentres

Si alguna vez tienes dudas, por favor contáctanos. Si descubres vulnerabilidades reales en un servicio, por favor contacta tanto a los operadores de ese servicio como a nosotros. Por favor no explotes la vulnerabilidad para obtener privilegios adicionales en un servicio, y no la anuncies ampliamente antes de que los operadores del servicio tengan la oportunidad de entenderla.

## Ideas de proyectos

Te animamos a que propongas tus propias ideas para lo que te gustaría trabajar, pero si buscas inspiración, a continuación hay una lista de ideas que podrían convertirse en buenos proyectos finales:

### 1. Análisis de seguridad de sistemas UC

Realizar un análisis de seguridad de algún sistema utilizado en la Pontificia Universidad Católica de Chile. La universidad cuenta con múltiples plataformas web y sistemas internos que podrían beneficiarse de una revisión de seguridad, como por ejemplo SIDING, Portal UC, Canvas, sistemas de bibliotecas, o aplicaciones móviles institucionales.

El objetivo es identificar potenciales vulnerabilidades utilizando técnicas aprendidas en el curso: análisis de la superficie de ataque, revisión de configuraciones, pruebas de inyección, análisis de autenticación y manejo de sesiones, entre otras.

**Importante:** Este proyecto es estrictamente de **detección**, no de explotación. Está prohibido:
- Explotar vulnerabilidades encontradas para acceder a datos reales
- Interrumpir el funcionamiento normal de los servicios
- Acceder a información de otros usuarios

Si encuentras una vulnerabilidad, debes:
1. Documentarla de forma responsable (sin incluir datos sensibles)
2. Notificar a los administradores del sistema a través de los canales oficiales de la universidad
3. Informar al equipo docente del curso

Este tipo de "responsible disclosure" es la práctica estándar en la industria de seguridad y es una habilidad valiosa para tu carrera profesional.

### 2. Análisis de vulnerabilidades en sistemas de IA

Los modelos de lenguaje (LLMs) y sistemas de inteligencia artificial presentan una nueva superficie de ataque que la industria aún está aprendiendo a proteger. Este potencial proyecto consiste en realizar un análisis práctico de vulnerabilidades en sistemas basados en IA, con énfasis en pruebas reales y documentación de resultados.

Algunas áreas de investigación posibles:

- **Prompt Injection:** Técnicas para hacer que un modelo ignore sus instrucciones originales y ejecute comandos del atacante. ¿Cómo se comportan diferentes modelos (GPT-4, Claude, Gemini, Llama) ante los mismos ataques? ¿Qué defensas son más efectivas?

- **Jailbreaking:** Métodos para evadir las restricciones de seguridad de los modelos. Análisis comparativo de la robustez de diferentes proveedores.

- **Extracción de información:** ¿Es posible extraer el system prompt o información sensible del contexto de un modelo? ¿Qué técnicas funcionan y cuáles no?

- **Ataques indirectos:** Cuando un modelo tiene acceso a herramientas o puede navegar la web, ¿cómo puede un atacante explotar esto colocando instrucciones maliciosas en contenido que el modelo procesará?

Para inspiración y práctica, pueden explorar [HackMyClaw.com](https://hackmyclaw.com), una plataforma de desafíos de seguridad en IA donde pueden probar técnicas de prompt injection en un entorno controlado.

El entregable debe incluir una metodología clara, resultados de pruebas en múltiples modelos, y recomendaciones de mitigación.

### 3. Implementar un sistema de seguridad desde cero

Diseñar e implementar un sistema relacionado con seguridad utilizando herramientas y primitivas de bajo nivel. Este proyecto permite entender en profundidad cómo funcionan las tecnologías de seguridad que normalmente usamos como "cajas negras".

Algunas ideas de sistemas a implementar:

- **VPN (Virtual Private Network):** Implementar una VPN funcional usando las herramientas que Linux ofrece: túneles TUN/TAP, iptables para routing, y criptografía para proteger el tráfico. Entender cómo WireGuard u OpenVPN logran crear canales seguros.

- **Sistema de autenticación:** Implementar un servidor de autenticación con soporte para múltiples factores (contraseña + TOTP), manejo seguro de sesiones, y protección contra ataques comunes (timing attacks, brute force).

- **Firewall de aplicación web (WAF):** Construir un proxy reverso que inspeccione tráfico HTTP y detecte patrones de ataque comunes (SQL injection, XSS, path traversal).

- **Sistema de detección de intrusos (IDS):** Implementar un monitor de red que analice tráfico y detecte comportamientos anómalos o firmas de ataques conocidos.

- **Sandbox de procesos:** Usar namespaces, cgroups, seccomp-bpf y capabilities de Linux para crear un entorno aislado donde ejecutar código no confiable de forma segura.

- **Password manager:** Implementar un gestor de contraseñas con cifrado local, derivación de claves segura (Argon2/PBKDF2), y protección contra acceso no autorizado.

El proyecto debe incluir documentación del diseño, justificación de las decisiones de seguridad tomadas, y una evaluación de las limitaciones y posibles mejoras del sistema implementado.

### 4. Estudio de ingeniería social y phishing

La ingeniería social sigue siendo uno de los vectores de ataque más efectivos, ya que explota vulnerabilidades humanas en lugar de técnicas. Este proyecto consiste en estudiar y experimentar con técnicas de ingeniería social de forma ética y controlada.

Algunas áreas de investigación:

- **Phishing con códigos QR (Quishing):** ¿Qué tan efectivo es colocar códigos QR maliciosos en lugares públicos? ¿Las personas escanean sin verificar el destino? Analizar las limitaciones técnicas y psicológicas de este vector.

- **Phishing asistido por IA:** Usar modelos de lenguaje para generar correos de phishing personalizados y comparar su efectividad con templates genéricos. ¿Qué tan convincentes pueden ser? ¿Qué defensas funcionan?

- **Pretexting y suplantación:** Estudiar técnicas de construcción de pretextos creíbles y cómo las organizaciones pueden entrenar a sus empleados para detectarlos.

- **Análisis de campañas reales:** Recopilar y analizar ejemplos de phishing recibidos (correos, SMS, WhatsApp) y categorizar las técnicas utilizadas.

**⚠️ Consideraciones éticas críticas:**

Este proyecto requiere especial cuidado para no causar daño real. Las pruebas deben ser:

- **Inofensivas:** Si haces que alguien haga clic en un enlace de prueba, el destino debe ser claramente benigno (por ejemplo, una página que diga "Esto fue un experimento" o un rickroll). Nunca recolectar credenciales reales ni información sensible.

- **Con consentimiento informado:** Idealmente, trabajar con un grupo que sepa que participará en un estudio (aunque no sepan exactamente cuándo).

- **Sin daño reputacional:** No suplantar identidades reales de profesores, empresas o instituciones de forma que pueda dañar su reputación.

- **Documentadas:** Mantener registro de todas las pruebas realizadas para poder demostrar que fueron éticas.

El entregable debe incluir un análisis de efectividad y recomendaciones de defensa para organizaciones.

### 5. Análisis de seguridad de dispositivos IoT

Los dispositivos IoT (Internet of Things) son conocidos por tener prácticas de seguridad deficientes. Este proyecto consiste en analizar la seguridad de dispositivos inteligentes del hogar: cámaras, ampolletas, enchufes, asistentes de voz, etc.

Áreas de análisis:
- **Comunicación de red:** ¿El tráfico está cifrado? ¿Qué datos envía el dispositivo y a dónde?
- **Firmware:** ¿Es posible extraer y analizar el firmware? ¿Hay credenciales hardcodeadas?
- **APIs y servicios cloud:** ¿Cómo se autentica el dispositivo con su backend? ¿Hay vulnerabilidades en la API?
- **Actualizaciones:** ¿El proceso de actualización es seguro? ¿Hay verificación de firmas?

Requiere acceso a dispositivos físicos y herramientas como Wireshark, binwalk, y potencialmente hardware para debugging (UART, JTAG).

### 6. Análisis de seguridad en aplicaciones móviles

Realizar análisis de seguridad de aplicaciones móviles Android o iOS, ya sean apps populares o desarrolladas localmente en Chile. 

Técnicas a aplicar:
- **Análisis estático:** Decompilar la aplicación y revisar el código en busca de vulnerabilidades, APIs inseguras, o secretos hardcodeados.
- **Análisis dinámico:** Interceptar tráfico de red con herramientas como Frida o mitmproxy, analizar el comportamiento en runtime.
- **Almacenamiento local:** ¿La app guarda datos sensibles de forma segura? ¿Usa el Keychain/Keystore correctamente?
- **Autenticación:** ¿Cómo maneja tokens y sesiones? ¿Es vulnerable a session hijacking?

Herramientas útiles: jadx, apktool, Frida, objection, MobSF.

---

**Nota:** Estas ideas son solo sugerencias para inspirarte. Puedes proponer cualquier proyecto que esté relacionado con seguridad computacional. Mientras más creativo y original sea tu proyecto, mejor será evaluado. ¡Sorpréndenos!

---

**Basado en**: MIT Course 6.566 Final Project Guidelines (Spring 2026)  
**URL Original**: https://css.csail.mit.edu/6.858/2026/labs/project.html  
**Licencia**: [Creative Commons Attribution 3.0 Unported](http://creativecommons.org/licenses/by/3.0/us/)

Este documento está adaptado de los materiales del curso Computer Systems Security del MIT.
