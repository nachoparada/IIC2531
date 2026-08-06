# Laboratorio 1: Análisis de Seguridad

## Introducción

En este laboratorio aplicarás el marco conceptual presentado en la primera clase para analizar la seguridad de diferentes sistemas. No escribirás código — el objetivo es desarrollar tu capacidad de pensar sistemáticamente sobre seguridad antes de implementar cualquier solución.

Recordemos el plan de alto nivel para pensar sobre seguridad:

* **Objetivo:** Lo que el sistema está tratando de lograr. Las categorías principales son:
  - *Confidencialidad:* No hay forma de que el adversario aprenda información secreta.
  - *Integridad:* No hay forma de que el adversario corrompa el estado del sistema.
  - *Disponibilidad:* El sistema sigue funcionando a pesar del adversario.

* **Modelo de Amenaza:** Suposiciones sobre lo que el atacante puede y no puede hacer.
  - ¿Qué recursos tiene el atacante?
  - ¿Qué acceso tiene al sistema?
  - ¿Qué está fuera de alcance?

* **Política:** El plan o conjunto de reglas que hará que el sistema logre el objetivo.
  - ¿Quién puede hacer qué?
  - ¿Bajo qué condiciones?
  - ¿Qué está prohibido?

* **Mecanismo:** El software/hardware que el sistema usa para hacer cumplir la política.
  - ¿Cómo se implementa la política técnicamente?
  - ¿Qué tecnologías específicas se usan?

---

## Instrucciones Generales

Para cada escenario presentado, debes escribir un análisis que incluya:

1. **Objetivo**: debes indicar el orden de importancia de confidencialidad, integridad y disponibilidad.
2. **Modelo de Amenaza**: mínimo 3 tipos de atacantes o vectores de ataque.
3. **Política**: mínimo 5 reglas concretas.
4. **Mecanismo**: mínimo 3 mecanismos técnicos que implementen las políticas.

Sé específico. No escribas generalidades como "el sistema debe ser seguro". En cambio, escribe cosas como "solo el dueño de una cuenta puede ver su historial de compras" (política) o "las contraseñas deben hashearse con bcrypt antes de almacenarse" (mecanismo).

**Formato de entrega:** Un archivo .md con tu análisis para cada escenario. Usa el template proporcionado al final de este documento.

---

## Ejemplo Resuelto: Sistema de Notas de IIC2531

**Contexto:** El curso IIC2531 necesita almacenar las notas de los estudiantes en un servidor. Los ayudantes deben poder leer y escribir el archivo con las notas. Los estudiantes solo deberían poder ver sus propias notas.

### Objetivo

**Orden de prioridad: Integridad > Confidencialidad > Disponibilidad**

- **Integridad:** Las notas no pueden ser modificadas por personas no autorizadas. Un estudiante no puede cambiar su propia nota ni la de otros.
- **Confidencialidad:** Un estudiante solo puede ver sus propias notas, no las de sus compañeros.
- **Disponibilidad:** El sistema debe estar disponible para consultas durante el semestre, especialmente cerca de fechas de entrega.

*Justificación del orden:* La integridad es lo más crítico porque notas incorrectas tienen consecuencias académicas graves. La confidencialidad importa por privacidad (FERPA/regulaciones locales). La disponibilidad es importante pero una caída temporal es tolerable — las notas pueden consultarse después.

### Modelo de Amenaza

#### Atacantes Considerados

1. **Estudiante curioso:** Tiene cuenta en el sistema, motivado por ver notas de compañeros o mejorar las propias. Puede intentar manipular URLs, explotar bugs en la aplicación web, o hacer ingeniería social a los ayudantes.

2. **Atacante externo:** Sin cuenta en el sistema. Puede intentar explotar vulnerabilidades del servidor, interceptar tráfico de red, o realizar ataques de fuerza bruta contra contraseñas.

3. **Ayudante malicioso/comprometido:** Tiene acceso legítimo al archivo de notas. Podría modificar notas indebidamente o filtrar el archivo completo. Su laptop podría ser robada con una copia local.

#### Fuera de Alcance

- Ataques físicos al datacenter del DCC (asumimos seguridad física adecuada)
- Compromiso del sistema operativo del servidor (responsabilidad de TI)
- Coerción física a los ayudantes

### Política

1. Solo los ayudantes autenticados pueden leer el archivo completo de notas.
2. Solo los ayudantes autenticados pueden modificar notas.
3. Los estudiantes autenticados pueden ver únicamente su propia nota.
4. Toda modificación de notas debe quedar registrada (quién, cuándo, qué cambió).
5. Las contraseñas de ayudantes deben tener mínimo 12 caracteres y no estar en listas de contraseñas filtradas.
6. Las sesiones expiran después de 30 minutos de inactividad.
7. El acceso al sistema requiere conexión desde la red universitaria o VPN.

### Mecanismo

| Política | Mecanismo |
|----------|-----------|
| Solo ayudantes pueden leer/escribir | Control de acceso basado en roles (RBAC) en la aplicación; archivo de notas sin acceso directo por web |
| Estudiantes ven solo su nota | Consulta filtrada por RUT del usuario autenticado; nunca exponer lista completa |
| Registro de modificaciones | Log de auditoría append-only con timestamp, usuario, y diff del cambio |
| Contraseñas seguras | Validación contra HaveIBeenPwned API al crear contraseña; hash con Argon2 |
| Sesiones expiran | Token JWT con expiración de 30 min; renovación requiere re-autenticación |
| Acceso desde red universitaria | Firewall que solo permite conexiones desde rangos IP de la universidad + VPN |
| Protección en tránsito | HTTPS obligatorio (TLS 1.3); HSTS habilitado |

### Análisis de Brechas

- **Laptop de ayudante robada:** La política actual no cubre copias locales. Posible mejora: prohibir descargas locales del archivo completo, o requerir cifrado de disco en laptops de ayudantes.
- **Ayudante malicioso:** El log de auditoría detecta cambios pero no los previene. Posible mejora: requerir aprobación de dos ayudantes para cambios de notas.
- **Ingeniería social:** No hay mecanismo técnico contra un ayudante que voluntariamente comparte su contraseña. Mitigación: capacitación + consecuencias disciplinarias (componente humano de la política).

---

## Escenario 1: Clínica Dental Pequeña

**Contexto:** Una clínica dental de barrio quiere digitalizar su operación. Los pacientes pueden reservar horas, recibir recordatorios por correo o WhatsApp, subir formularios médicos previos a la atención, y revisar presupuestos de tratamientos. La recepcionista administra la agenda y los pagos, mientras que los dentistas registran diagnósticos, notas clínicas y tratamientos realizados.

**Consideraciones adicionales:**
- Los pacientes deben poder ver y modificar sus propias reservas
- La recepcionista necesita mover horas, registrar pagos y contactar pacientes
- Los dentistas necesitan acceder al historial clínico de sus pacientes
- Algunos pacientes son menores de edad y sus apoderados gestionan las reservas
- El sistema es administrado por personas sin formación técnica

**Preguntas guía:**
- ¿Qué información es sensible en este sistema?
- ¿Qué debería poder hacer un paciente, una recepcionista y un dentista?
- ¿Qué pasa si un paciente puede ver fichas clínicas de otros pacientes?
- ¿Qué pasa si alguien modifica o elimina una reserva?
- ¿Cómo cambia el modelo de amenaza cuando hay datos médicos y menores de edad?
- ¿Cómo afecta que el sistema sea usado por personas no expertas en tecnología?

---

## Escenario 2: Plataforma de Payroll para Pymes

**Contexto:** SueldoSimple es una plataforma SaaS que ayuda a pequeñas y medianas empresas a calcular sueldos, cotizaciones, impuestos, liquidaciones y archivos de transferencia bancaria. La plataforma atiende a 800 empresas, cada una con entre 10 y 300 empleados. En cada empresa hay administradores de RR.HH., contadores externos y empleados que consultan sus liquidaciones.

**Consideraciones adicionales:**
- Cada empresa solo debe acceder a sus propios empleados y liquidaciones
- Los administradores pueden modificar sueldos, bonos, descuentos y cuentas bancarias
- Los empleados pueden descargar sus propias liquidaciones históricas
- El equipo de soporte de SueldoSimple puede ayudar a clientes con problemas de configuración
- La plataforma genera archivos para pagos bancarios mensuales

**Preguntas guía:**
- ¿Qué información es sensible en este sistema?
- ¿Qué pasa si una empresa puede ver la información de otra empresa?
- ¿Qué pasa si un administrador malicioso cambia la cuenta bancaria de un empleado?
- ¿Qué información debería poder ver soporte y qué debería estar fuera de su alcance?
- ¿Cómo se auditan cambios en sueldos, bonos y cuentas bancarias?
- ¿Cómo afecta que el sistema maneje dinero y datos laborales?

---

## Escenario 3: Portal de Resultados Médicos

**Contexto:** RedLab es una red de laboratorios clínicos que permite a pacientes descargar resultados de exámenes, a médicos revisar resultados de sus pacientes, y a técnicos de laboratorio subir informes validados. El sistema procesa exámenes de sangre, imágenes y otros resultados clínicos para varias clínicas asociadas. Aproximadamente 60,000 pacientes usan el portal mensualmente.

**Consideraciones adicionales:**
- Los resultados pueden contener diagnósticos o información de salud altamente sensible
- Algunos resultados deben ser revisados por un médico antes de liberarse al paciente
- Los médicos solo deberían ver resultados de pacientes bajo su atención
- Los técnicos de laboratorio pueden cargar resultados, pero no deberían modificarlos después de validados
- El sistema debe mantener un registro de quién accedió a cada resultado

**Preguntas guía:**
- ¿Qué consecuencias tiene filtrar resultados médicos?
- ¿Qué consecuencias tiene publicar un resultado incorrecto o incompleto?
- ¿Cómo se verifica que un médico tiene derecho a ver los resultados de un paciente?
- ¿Qué acciones deberían requerir registro de auditoría?
- ¿Qué pasa si una cuenta de médico es comprometida?
- ¿Cómo se equilibra confidencialidad con disponibilidad en casos urgentes?

---

## Escenario 4: Sistema de Tickets QR para Conciertos

**Contexto:** TicketSur vende entradas para conciertos y eventos masivos. Los usuarios compran entradas en línea, reciben un código QR, pueden transferir entradas a otros usuarios y, en algunos eventos, revenderlas dentro de la misma plataforma. En la entrada del recinto, guardias usan teléfonos con una app de validación para escanear los QR. Algunos recintos tienen conexión inestable durante eventos grandes.

**Consideraciones adicionales:**
- Cada ticket debe permitir el ingreso de una sola persona
- Los usuarios pueden transferir tickets antes del evento
- La plataforma debe resistir alta demanda cuando se abre la venta de eventos populares
- Los guardias necesitan validar entradas rápidamente en la puerta
- Puede haber reembolsos, contracargos y disputas por tickets revendidos

**Preguntas guía:**
- ¿Qué pasa si un ticket QR se copia y se comparte muchas veces?
- ¿Qué pasa si un atacante compra miles de entradas usando bots?
- ¿Cómo se asegura que una transferencia de ticket sea válida?
- ¿Cómo debería funcionar la validación si el recinto pierde conexión?
- ¿Qué información necesita ver un guardia al escanear un ticket?
- ¿Cómo se protegen compradores legítimos frente a fraudes de reventa?

---

## Escenario 5: Bicicletas Compartidas

**Contexto:** BikeCity opera bicicletas compartidas en Santiago. Los usuarios instalan una app, registran un medio de pago, encuentran bicicletas cercanas, escanean un código QR para desbloquearlas y pagan según los minutos de uso. El equipo de operaciones puede ver la ubicación de las bicicletas, bloquear unidades con fallas y reasignar bicicletas entre estaciones. Las bicicletas tienen candados electrónicos y se comunican con el servidor mediante red móvil.

**Consideraciones adicionales:**
- Los usuarios deben poder iniciar y terminar viajes desde la app
- La empresa cobra automáticamente según la duración del viaje
- Las bicicletas reportan ubicación, batería y estado del candado
- Técnicos de mantenimiento pueden desbloquear bicicletas para reparación
- Algunas zonas tienen mala conectividad móvil

**Preguntas guía:**
- ¿Qué pasa si alguien logra desbloquear bicicletas sin pagar?
- ¿Qué información de ubicación de usuarios es sensible?
- ¿Qué pasa si un atacante falsifica el término de un viaje?
- ¿Qué puede hacer un técnico de mantenimiento y cómo se limita ese poder?
- ¿Cómo debería comportarse una bicicleta si pierde conexión con el servidor?
- ¿Qué ataques combinan software, hardware y acceso físico a la bicicleta?

---

## Escenario 6: Tu Propio Escenario

**Instrucciones:** Elige un sistema real que uses frecuentemente (puede ser una app, un servicio web, un sistema de tu trabajo, etc.) y realiza el mismo análisis. El sistema debe ser suficientemente complejo como para tener múltiples tipos de usuarios y datos sensibles.

**Ejemplos de sistemas válidos:**
- Sistema de gestión académica (ej: SIDING, UCampus)
- Aplicación de banco o fintech
- Red social o servicio de mensajería
- Sistema de salud (agenda médica, resultados de exámenes)
- Plataforma de streaming o gaming

**Nota:** Si eliges un sistema de tu trabajo, puedes anonimizar nombres y detalles específicos, pero mantén la estructura y complejidad real.

---

## Entrega

**Fecha de entrega:** 18 de Marzo de 2026

**Formato:** un archivo .md con el contenido


---

## Template de Respuesta

```
# Escenario N: [Nombre]

## Objetivo

**Orden de prioridad: [Integridad/Confidencialidad/Disponibilidad] > [...] > [...]**

- **[Primera prioridad]:** [Por qué es lo más importante]
- **[Segunda prioridad]:** [Por qué importa]
- **[Tercera prioridad]:** [Por qué es menos crítico]

*Justificación del orden:* [Explicar por qué ordenaste así]

## Modelo de Amenaza

### Atacantes Considerados
1. [Tipo de atacante 1]: [Capacidades y motivación]
2. [Tipo de atacante 2]: [Capacidades y motivación]
3. [Tipo de atacante 3]: [Capacidades y motivación]

### Fuera de Alcance
- [Qué ataques NO consideramos y por qué]

## Política

1. [Regla 1]
2. [Regla 2]
3. [Regla 3]
4. [Regla 4]
5. [Regla 5]

## Mecanismo

| Política | Mecanismo |
|----------|-----------|
| [Regla 1] | [Tecnología/proceso que la implementa] |
| [Regla 2] | [Tecnología/proceso que la implementa] |
| ... | ... |

## Análisis de Brechas (Opcional)

[¿Hay políticas sin mecanismo claro? ¿Hay ataques en el modelo de amenaza 
que no están cubiertos por ninguna política?]
```

---

## Recursos Adicionales

- [Lectura de clase: Introducción a Seguridad Computacional](../Classes/Clase01/introduction.md)
- OWASP Top 10: https://owasp.org/www-project-top-ten/
- STRIDE Threat Model: https://learn.microsoft.com/en-us/azure/security/develop/threat-modeling-tool-threats

---
