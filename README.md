# IIC2531 - Seguridad Computacional

Materiales del curso IIC2531 Seguridad Computacional.

El repositorio esta organizado por semestre. Cada carpeta semestral contiene el programa, clases, laboratorios y proyecto final correspondientes.

## Semestres

| Semestre | Estado | Notas |
|---|---|---|
| `2026-2` | En preparacion | Copia inicial de `2026-1`; fechas y ajustes pendientes |
| `2026-1` | Cerrado | Material completo del semestre 2026-1 |
| `2025-2` | Archivo | Material historico |

## Estructura

```text
2026-2/
  README.md              # Informacion del semestre
  Classes/               # Clases y recursos
  Labs/                  # Laboratorios
  FinalProject/          # Enunciado y criterios del proyecto final
```

## Flujo de preparacion de un nuevo semestre

1. Copiar el semestre anterior a una nueva carpeta.
2. Limpiar entregas de estudiantes y materiales invitados que no se reutilizaran.
3. Actualizar el `README.md` del semestre con fechas, sala, horario y evaluaciones.
4. Revisar el orden de clases y laboratorios contra el calendario academico.
5. Registrar cambios importantes en el `CHANGELOG.md` del semestre.

## Presentaciones

Las clases estan escritas principalmente en Markdown compatible con Marp. Para previsualizar o exportar una clase:

```bash
npx marp --server --watch 2026-2/Classes
npx marp --pdf 2026-2/Classes/Clase01/introduction.md
```

Tambien puede usarse la extension "Marp for VS Code" para previsualizar archivos individuales.

## Licencia y fuentes

Este curso esta basado parcialmente en materiales de MIT 6.858 / 6.566 Computer Systems Security, utilizados bajo licencia Creative Commons Attribution 3.0, junto con materiales propios y lecturas academicas o de industria citadas en cada clase.
