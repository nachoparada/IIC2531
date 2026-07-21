# IIC2531 - Seguridad Computacional

## Informacion General

| | |
|---|---|
| **Codigo** | IIC2531 |
| **Semestre** | 2026-2 |
| **Creditos** | 10 |
| **Horario** | Por definir |
| **Sala** | Por definir |
| **Profesor** | Ignacio Parada (yo@ignacioparada.com) |

## Descripcion

Este curso ofrece una vision general de los principales temas en seguridad computacional. A traves de lecturas de papers academicos y de la industria, los estudiantes exploran tecnicas de ataque y defensa en sistemas reales. El enfoque es practico: los laboratorios permiten experimentar con vulnerabilidades y contramedidas en el contexto de aplicaciones y sistemas concretos.

El curso prioriza **amplitud sobre profundidad**, cubriendo una variedad de temas que van desde aislamiento de procesos hasta seguridad de redes, pasando por arquitecturas de sistemas seguros, seguridad de software, seguridad web, sistemas distribuidos y seguridad en sistemas de IA.

## Contenidos

### Modulo 1: Aislamiento
- **Modelos de amenaza:** identificacion de adversarios, superficies de ataque y objetivos de seguridad
- **Procesos y separacion de privilegios:** principio de minimo privilegio, diseno de OKWS
- **Contenedores Linux:** namespaces, cgroups, chroot, seccomp-bpf
- **Maquinas virtuales:** KVM, QEMU, virtualizacion de CPU, memoria y dispositivos
- **Sandboxing moderno:** Firecracker, microVMs, trade-offs entre aislamiento y rendimiento

### Modulo 2: Arquitecturas y Sistemas Seguros
- **Arquitecturas de aplicaciones web:** separacion de componentes y servicios de autenticacion
- **Aislamiento en navegadores:** modelo de mismo origen y sandboxing de procesos
- **Sistemas operativos seguros:** capacidades y control de acceso mandatorio
- **Sistemas distribuidos:** coordinacion segura y manejo de fallas

### Modulo 3: Seguridad de Software
- **Vulnerabilidades de memoria:** buffer overflow, use-after-free, format strings
- **Tecnicas de explotacion:** inyeccion de codigo y ROP
- **Defensas en tiempo de ejecucion:** stack canaries, ASLR, DEP/NX, CFI
- **Analisis de codigo:** fuzzing, analisis estatico y ejecucion simbolica
- **Cadena de suministro:** dependencias, reproducibilidad, firmas de codigo y ataques a paquetes

### Modulo 4: Seguridad Web, Redes e IA
- **Criptografia aplicada:** cifrado, hashing y firmas digitales
- **Protocolos seguros:** TLS/HTTPS, certificados X.509 y Certificate Transparency
- **Autenticacion:** passwords, hashing con salt, tokens, WebAuthn/FIDO2
- **Seguridad web:** mismo origen, CORS, cookies, CSRF, XSS e inyeccion SQL
- **Seguridad en IA:** prompt injection, herramientas, agentes y patrones de aislamiento

## Calendario de Clases

El calendario detallado de clases esta disponible en [CALENDARIO.md](CALENDARIO.md).

## Material del Curso

- [Calendario de clases](CALENDARIO.md)
- [Proyecto final](FinalProject/FinalProject.md)
- [Entrega final del proyecto](FinalProject/EntregaFinal.md)

## Evaluacion

La nota final se calcula de la siguiente manera:

```text
NotaFinal = 0.4 * Evaluaciones + 0.4 * Laboratorios + 0.2 * Proyecto
```

Donde:
- **Evaluaciones** = 0.3 * I1 + 0.3 * I2 + 0.4 * Examen
- **Laboratorios** = 0.1 * L1 + 0.2 * L2 + 0.3 * L3 + 0.3 * L4 + 0.1 * L5

### Requisitos de Aprobacion
- Evaluaciones >= 4.0
- Examen >= 4.0
- Proyecto >= 4.0

Si alguno de estos requisitos no se cumple, la nota final corresponde a la nota del componente reprobado.

### Fechas de Evaluaciones

| Evaluacion | Fecha |
|------------|-------|
| I1 | 2026-09-30 |
| I2 | 2026-11-19 |
| Examen | 2026-12-10 |

### Fechas de Laboratorios

| Lab | Tema | Fecha de Entrega |
|-----|------|------------------|
| Lab 1 | Modelo de Amenaza | 2026-08-25 |
| Lab 2 | Separacion de Privilegios | 2026-09-10 |
| Lab 3 | Buffer Overflow | 2026-10-01 |
| Lab 4 | Seguridad en Navegadores | 2026-10-22 |
| Lab 5 | Seguridad en Sistemas de IA | 2026-11-05 |

### Proyecto Final
- **Fecha de entrega:** 2026-11-19
- Puede ser orientado a ataque o defensa
- Presentaciones al final del semestre
- Ver [Proyecto final](FinalProject/FinalProject.md) y [Entrega final del proyecto](FinalProject/EntregaFinal.md)

## Bibliografia

### Textos de Referencia
- Anderson, R. *Security Engineering: A Guide to Building Dependable Distributed Systems* (3rd ed.). Wiley, 2020. [Disponible online](https://www.cl.cam.ac.uk/~rja14/book.html)
- Viega, J. & McGraw, G. *Building Secure Software*. Addison-Wesley, 2001.

### Papers y Lecturas
El curso se basa en papers academicos y de la industria que se asignan clase a clase. Los estudiantes deben leer el material asignado antes de cada sesion.

---

*Este curso esta basado parcialmente en materiales de MIT 6.858 / 6.566 Computer Systems Security, utilizados bajo licencia [Creative Commons Attribution 3.0](http://creativecommons.org/licenses/by/3.0/us/).*
