# Marp Presentations - Seguridad Computacional

Este proyecto contiene presentaciones creadas con Marp para el curso de Seguridad Computacional.

## ¿Qué es Marp?

Marp es una herramienta que permite crear presentaciones hermosas usando sintaxis Markdown. Puedes escribir tus diapositivas en Markdown y convertirlas a HTML, PDF, PowerPoint y otros formatos.

## Instalación

### Opción 1: Usando Node.js (Recomendado)

1. Instala Node.js desde [nodejs.org](https://nodejs.org/)
2. Instala Marp CLI globalmente:
   ```bash
   npm install -g @marp-team/marp-cli
   ```

### Opción 2: Usando VS Code

1. Instala VS Code desde [code.visualstudio.com](https://code.visualstudio.com/)
2. Instala la extensión "Marp for VS Code" desde el marketplace
3. Abre cualquier archivo `.md` y usa `Ctrl+Shift+P` → "Marp: Open Preview"

### Opción 3: Usando el Navegador

1. Ve a [marp.app](https://marp.app/)
2. Sube tu archivo `.md` o escribe directamente en el editor

## Uso

### Ver la Presentación

```bash
# Servidor local con vista previa en tiempo real
marp --server --watch .

# Generar HTML
marp --html sample-presentation.md

# Generar PDF
marp --pdf sample-presentation.md

# Generar PowerPoint
marp --pptx sample-presentation.md
```

### Estructura de Archivos

- `sample-presentation.md` - Presentación de ejemplo sobre Seguridad Computacional
- `package.json` - Configuración del proyecto
- `README.md` - Este archivo

## Sintaxis de Marp

### Encabezados de Diapositivas

```markdown
---
marp: true
theme: default
paginate: true
---

# Título de la Diapositiva

Contenido aquí...

---

# Nueva Diapositiva

Más contenido...
```

### Temas Disponibles

- `default` - Tema por defecto
- `gaia` - Tema Gaia
- `uncover` - Tema Uncover

### Características Especiales

- **Imágenes**: `![alt](image.jpg)`
- **Código**: ```javascript`código aquí```
- **Matemáticas**: `$E = mc^2$`
- **Notas del orador**: `<!-- speaker: notas aquí -->`

## Personalización

### CSS Personalizado

Crea un archivo `custom-theme.css`:

```css
/* @theme custom */

section {
  background-color: #f0f0f0;
  color: #333;
  font-size: 30px;
  padding: 40px;
}

h1 {
  color: #2c3e50;
}
```

### Usar Tema Personalizado

```markdown
---
marp: true
theme: custom
---
```

## Comandos Útiles

```bash
# Instalar dependencias
npm install

# Iniciar servidor de desarrollo
npm start

# Generar presentación HTML
npm run build-html

# Generar presentación PDF
npm run build
```

## Recursos Adicionales

- [Documentación oficial de Marp](https://marpit.marp.app/)
- [Marp CLI](https://github.com/marp-team/marp-cli)
- [Marp for VS Code](https://marketplace.visualstudio.com/items?itemName=marp-team.marp-vscode)
- [Editor web de Marp](https://marp.app/)

## Próximos Pasos

1. Personaliza `sample-presentation.md` con tu contenido
2. Crea nuevas presentaciones siguiendo el mismo formato
3. Experimenta con diferentes temas y estilos
4. Comparte tus presentaciones con la clase

¡Disfruta creando presentaciones hermosas con Marp! 