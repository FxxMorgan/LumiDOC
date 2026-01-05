# 📚 LumiDOC - Documentación del Sistema Luminova

Repositorio de documentación del **Sistema Luminova**, un dashboard de monitoreo energético multi-tenant construido con Node.js, React, MongoDB y Socket.io.

## 🌐 Ver la Documentación Online

La documentación está publicada en GitHub Pages y se actualiza automáticamente:

**👉 [https://fxxmorgan.github.io/LumiDOC](https://fxxmorgan.github.io/LumiDOC)**

## 🚀 Ejecutar la Documentación Localmente

### Prerrequisitos

- Python 3.x instalado

### Instalación

```bash
# Clonar el repositorio
git clone https://github.com/FxxMorgan/LumiDOC.git
cd LumiDOC

# Instalar dependencias
pip install mkdocs-material
```

### Servidor de Desarrollo

```bash
# Iniciar servidor local
mkdocs serve

# La documentación estará disponible en:
# http://127.0.0.1:8000
```

El servidor se recargará automáticamente cuando modifiques los archivos.

### Construir la Documentación

```bash
# Generar sitio estático en /site
mkdocs build

# El resultado estará en el directorio site/
```

## 📝 Agregar Nueva Documentación

### 1. Crear el Archivo Markdown

Tienes dos opciones:

**Opción A: Archivo en la raíz (para documentos generales)**
```bash
# Crear archivo en la raíz del proyecto
touch MI_NUEVO_DOCUMENTO.md
```

**Opción B: Archivo en docs/ (para documentos específicos)**
```bash
# Crear en la estructura docs/
touch docs/mi-nuevo-documento.md
# o en una subcarpeta
touch docs/dias/dia-6.md
```

### 2. Editar el Archivo

Escribe tu contenido en formato Markdown con sintaxis estándar y extensiones de Material for MkDocs.

### 3. Actualizar la Navegación

Edita el archivo `mkdocs.yml` y agrega tu documento en la sección `nav:`:

```yaml
nav:
  - Inicio: index.md
  - Roadmap: roadmap.md
  - Progreso por Días:
    - Día 1 - Setup Inicial: dias/dia-1.md
    - Día 6 - Mi Nuevo Día: dias/dia-6.md  # ← Agregar aquí
  - Mi Nueva Sección:
    - Mi Documento: mi-nuevo-documento.md  # ← O crear nueva sección
```

### 4. Hacer Commit y Push

```bash
# Agregar archivos
git add .

# Commit
git commit -m "docs: agregar nueva documentación"

# Push a main
git push origin main
```

El **GitHub Actions workflow** se ejecutará automáticamente y actualizará el sitio en unos minutos.

## 📁 Estructura del Proyecto

```
LumiDOC/
├── docs/                      # Directorio principal de documentación
│   ├── index.md              # Página de inicio
│   ├── roadmap.md            # Plan de desarrollo
│   ├── dias/                 # Documentación diaria
│   │   ├── dia-1.md
│   │   ├── dia-2.md
│   │   └── ...
│   ├── tecnica/              # Documentación técnica
│   │   └── importador.md
│   └── images/               # Imágenes de la documentación
│       ├── image.png
│       ├── image-1.png
│       └── image-2.png
├── .github/
│   └── workflows/
│       └── deploy-docs.yml   # Workflow de despliegue automático
├── mkdocs.yml                # Configuración de MkDocs
├── .gitignore                # Archivos ignorados por Git
└── README.md                 # Este archivo
```

## 🎨 Características del Tema Material

La documentación utiliza **Material for MkDocs** con las siguientes características:

- ✨ **Modo Oscuro/Claro**: Cambio automático según preferencias del sistema
- 🔍 **Búsqueda Integrada**: Con sugerencias y resaltado
- 📱 **Diseño Responsive**: Optimizado para móvil, tablet y desktop
- 🎨 **Syntax Highlighting**: Código con colores para múltiples lenguajes
- 📑 **Navegación con Pestañas**: Organización clara y accesible
- 🔗 **Tabla de Contenidos**: Navegación interna en cada página
- 📋 **Código Copiable**: Botón para copiar bloques de código
- 😀 **Emojis y Iconos**: Soporte completo para emojis

## ⚙️ Configuración

### Personalizar el Tema

Edita `mkdocs.yml` para cambiar:

- **Colores**: Sección `theme.palette`
- **Características**: Sección `theme.features`
- **Navegación**: Sección `nav`
- **Extensiones de Markdown**: Sección `markdown_extensions`

### Variables de Sitio

```yaml
site_name: LumiDOC - Sistema Luminova
site_description: Documentación del Sistema Luminova
site_author: FxxMorgan
site_url: https://fxxmorgan.github.io/LumiDOC
```

## 🔄 Workflow de Deploy Automático

El archivo `.github/workflows/deploy-docs.yml` configura el despliegue automático:

- **Trigger**: Push a rama `main` cuando cambian archivos `.md` o `mkdocs.yml`
- **Acciones**:
  1. Checkout del código
  2. Instalación de Python 3.x
  3. Instalación de MkDocs y Material
  4. Deploy a rama `gh-pages`

## 📖 Recursos

- [Documentación de MkDocs](https://www.mkdocs.org/)
- [Material for MkDocs](https://squidfunk.github.io/mkdocs-material/)
- [Sintaxis Markdown](https://www.markdownguide.org/)
- [GitHub Pages](https://pages.github.com/)

## 🤝 Contribuir

1. Fork el repositorio
2. Crea una rama para tu feature (`git checkout -b feature/nueva-documentacion`)
3. Haz commit de tus cambios (`git commit -m 'docs: agregar nueva documentación'`)
4. Push a la rama (`git push origin feature/nueva-documentacion`)
5. Abre un Pull Request

## 📄 Licencia

Este proyecto de documentación es parte del Sistema Luminova.

---

**Desarrollado con ❤️ por FxxMorgan**  
**Stack**: MkDocs + Material Theme + GitHub Pages
