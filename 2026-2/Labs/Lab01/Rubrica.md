# Rúbrica de Evaluación - Laboratorio 1: Análisis de Seguridad

## Sistema de Calificación

Cada escenario vale **1.0 punto**:
- 6 escenarios correctos → 6.0 puntos → Nota 7.0

**Nota importante:** Este es un laboratorio introductorio. Se espera análisis conceptual, no expertise técnico. Las respuestas pueden ser generales, especialmente en la sección de mecanismos.

---

## Evaluación por Escenario (1.0 punto cada uno)

### 1. Objetivo (0.25 puntos)

**Criterio de aprobación:** ¿El estudiante identifica un orden de prioridad (C/I/D) y explica por qué?

| Puntaje | Descripción |
|---------|-------------|
| **0.25** | Orden claro (ej: I > C > D) con razón que tenga sentido para el sistema. No necesita ser profundo, pero debe mostrar que pensó en el contexto. |
| **0.15** | Orden presente pero justificación muy breve o genérica. |
| **0.05** | Orden presente sin justificación real. |
| **0** | No hay orden o es claramente incorrecto para el sistema. |

**Ejemplos aceptables:**
- ✅ "Integridad > Confidencialidad > Disponibilidad porque las notas incorrectas tienen consecuencias académicas graves."
- ✅ "Disponibilidad > Integridad > Confidencialidad porque si el cajero no funciona, los clientes no pueden sacar dinero."
- ❌ "Seguridad es importante." (demasiado vago)

---

### 2. Modelo de Amenaza (0.25 puntos)

**Criterio de aprobación:** ¿El estudiante identifica al menos 3 tipos diferentes de atacantes relevantes para el sistema?

| Puntaje | Descripción |
|---------|-------------|
| **0.25** | 3+ atacantes diferentes y relevantes. Cada uno con descripción básica de quién es y qué quiere. Incluye lista de "Fuera de alcance". |
| **0.15** | 3 atacantes pero muy similares entre sí, o descripciones muy breves. Puede faltar "Fuera de alcance". |
| **0.05** | Solo 2 atacantes, o 3 atacantes irrelevantes para el sistema. |
| **0** | Menos de 2 atacantes o descripciones sin sentido. |

**Ejemplos aceptables:**
- ✅ "1) Estudiante curioso: quiere ver notas de compañeros. 2) Atacante externo: intenta acceder sin cuenta. 3) Ayudante malicioso: tiene acceso legítimo pero puede abusar."
- ✅ "1) Cliente estafador. 2) Hacker externo. 3) Empleado deshonesto." (breve pero cubre tipos diferentes)
- ❌ "1) Hacker. 2) Otro hacker. 3) Más hackers." (no hay diversidad)

---

### 3. Política (0.25 puntos)

**Criterio de aprobación:** ¿El estudiante define al menos 5 reglas que tengan sentido para el sistema?

| Puntaje | Descripción |
|---------|-------------|
| **0.25** | 5+ reglas que aborden los riesgos del sistema. Pueden ser generales (ej: "solo X puede hacer Y"). |
| **0.15** | 5 reglas pero muy genéricas o redundantes entre sí. |
| **0.05** | 3-4 reglas, o 5 reglas pero varias no tienen sentido para el sistema. |
| **0** | Menos de 3 reglas o completamente irrelevantes. |

**Nota importante:** En este laboratorio introductorio, las reglas pueden ser generales. No es necesario especificar tiempos exactos o parámetros técnicos.

**Ejemplos aceptables:**
- ✅ "Solo los ayudantes pueden modificar notas." (regla simple pero clara)
- ✅ "Las contraseñas deben ser seguras." (aunque vago, es aceptable en este nivel)
- ✅ "Los estudiantes solo ven sus propias notas."
- ❌ "El sistema debe funcionar bien." (demasiado vago, no es una regla de seguridad)

---

### 4. Mecanismo (0.25 puntos)

**Criterio de aprobación:** ¿El estudiante menciona al menos 3 mecanismos técnicos que implementen las políticas?

| Puntaje | Descripción |
|---------|-------------|
| **0.25** | 3+ mecanismos mencionados y mapeados a políticas. Pueden ser vagos (ej: "encriptación", "autenticación", "firewall"). |
| **0.15** | 3 mecanismos pero muy poco detalle o mapeo débil con las políticas. |
| **0.05** | Solo 2 mecanismos, o 3 mecanismos irrelevantes. |
| **0** | Menos de 2 mecanismos o sin mapeo a políticas. |

**Nota importante:** Se espera que los mecanismos sean vagos a este nivel del curso. "Usar contraseñas" o "encriptar la conexión" son respuestas aceptables. No se requieren tecnologías específicas (Argon2, TLS 1.3, etc.).

**Ejemplos aceptables:**
- ✅ "Autenticación con usuario y contraseña" → implementa política de acceso restringido
- ✅ "Encriptación" → protege la confidencialidad de datos
- ✅ "Firewall" → bloquea accesos no autorizados
- ✅ "HTTPS" → protege datos en tránsito
- ❌ "Seguridad" (demasiado vago, no es un mecanismo)

---

---

## Resumen por Escenario

Cada escenario se evalúa en 4 secciones, cada una vale 0.25 puntos:
- **Objetivo:** 0.25 pts
- **Modelo de Amenaza:** 0.25 pts
- **Política:** 0.25 pts
- **Mecanismo:** 0.25 pts

**Total por escenario:** 1.0 punto

**Análisis de Brechas:** Opcional. Si está presente y bien hecho, puede ayudar a compensar debilidades en otras secciones, pero no suma puntos extra al total de 1.0 por escenario.

---

## Criterios Transversales (Aplican a toda la entrega)

### Descuentos por Problemas Graves

- **No usar el template o faltar secciones completas:** -0.5 pts por escenario afectado

### Coherencia Mínima

No se hacen descuentos por inconsistencias menores en este laboratorio introductorio. Se espera que los estudiantes estén aprendiendo a pensar en estos términos.

---

## Notas sobre los Escenarios

Los 6 escenarios tienen diferentes niveles de complejidad:

### Escenarios Más Simples (buenos para empezar)
- **Escenario 1 (Tienda de Don Carlos):** Sistema pequeño, pocos actores
- **Escenario 4 (Votación estudiantil):** Objetivos claros, contexto familiar

### Escenarios de Complejidad Media
- **Escenario 2 (Marketplace):** Múltiples actores, mayor escala
- **Escenario 3 (Portal gobierno):** Implicaciones legales, datos sensibles

### Escenarios Más Complejos
- **Escenario 5 (Cajero automático):** Combina seguridad física y digital
- **Escenario 6 (Tu propio sistema):** Requiere describir el contexto además del análisis

**Recomendación:** Si tienes dudas, comienza con los escenarios más simples para familiarizarte con la estructura del análisis.

---

## Ejemplos de Evaluación

### Ejemplo 1: Respuesta Completa para un Escenario (1.0 punto)

**Escenario:** Sistema de notas

**Objetivo (0.25):** ✅ "Integridad > Confidencialidad > Disponibilidad. Notas incorrectas afectan el futuro académico de estudiantes."

**Modelo de Amenaza (0.25):** ✅ "1) Estudiante curioso que quiere ver notas. 2) Atacante externo sin acceso. 3) Ayudante malicioso con acceso legítimo."

**Política (0.25):** ✅ "1) Solo ayudantes pueden modificar notas. 2) Estudiantes solo ven sus propias notas. 3) Toda modificación queda registrada. 4) Contraseñas seguras. 5) Sesiones expiran."

**Mecanismo (0.25):** ✅ "Autenticación con contraseña → controla acceso. HTTPS → protege datos en tránsito. Log de auditoría → registra cambios."

**Puntaje:** 1.0 (las 4 secciones completas)

---

### Ejemplo 2: Respuesta Incompleta (0.5 puntos)

**Objetivo (0.25):** ✅ "Integridad es lo más importante."

**Modelo de Amenaza (0.15):** ⚠️ "Hackers, estudiantes, ayudantes." (muy breve, sin descripción)

**Política (0.05):** ⚠️ "Solo personal autorizado puede acceder. Las notas deben ser correctas." (solo 2 reglas, muy vagas)

**Mecanismo (0.05):** ⚠️ "Seguridad, encriptación." (demasiado vago, no mapea a políticas)

**Puntaje:** 0.5 (solo Objetivo completo, lo demás insuficiente)

---

## Preguntas Frecuentes

**P: ¿Necesito nombrar tecnologías específicas en los mecanismos?**  
R: **No es necesario.** Este es un laboratorio introductorio. Respuestas como "autenticación con contraseña", "encriptación de datos", o "firewall" son aceptables. Si conoces tecnologías específicas (ej: TLS, bcrypt), puedes mencionarlas, pero no es requisito.

**P: ¿Puedo usar IA para ayudarme?**  
R: Puedes usar IA para investigar conceptos o entender el marco de análisis, pero **el análisis debe ser tuyo**. Si copias respuestas generadas sin entenderlas, será evidente y se penalizará.

**P: ¿Qué tan largo debe ser cada análisis?**  
R: No hay un largo mínimo o máximo. El ejemplo resuelto es extenso para mostrar posibilidades, pero puedes ser más conciso. Lo importante es cubrir las 4 secciones (Objetivo, Modelo de Amenaza, Política, Mecanismo).

**P: ¿Debo hacer el Análisis de Brechas?**  
R: Es opcional. Si lo incluyes y está bien hecho, puede ayudarte si otras secciones están débiles, pero no es obligatorio.

**P: ¿Qué pasa si no estoy seguro de mi análisis?**  
R: Está bien no tener certeza absoluta. Este laboratorio es para practicar pensar en términos de seguridad. Haz tu mejor esfuerzo basándote en lo visto en clases.
