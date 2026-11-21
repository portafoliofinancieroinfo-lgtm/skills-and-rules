---
name: skill_map
description: >-
  Genera un Prototipo Comentable de Solo Lectura (estilo Figma) para el proyecto
  que esté abierto en ese momento, permitiendo navegar la UI recreada, hacer clic
  derecho sobre cualquier elemento para dejar pines con comentarios y exportar el
  pedido exacto en Markdown.
---

# Prototipo Comentable de Solo Lectura (`skill_map`)

Usa esta skill cuando el usuario pida crear o actualizar el prototipo/mapa
visual del proyecto abierto en ese momento, recrear la interfaz en solo lectura
para dejar comentarios encima, o planificar cambios visuales y funcionales sin
ambigüedades.

## Dónde guardar el archivo (primera vez y actualizaciones)

1. **Primera vez en el proyecto:** antes de escribir el archivo por primera vez,
   **pregunta al usuario dónde prefiere guardar** `prototipo-<proyecto>.html`,
   ofreciendo opciones claras según el proyecto actual (por ejemplo: en la raíz
   del proyecto `./prototipo-<proyecto>.html`, en `./docs/prototipo-<proyecto>.html`,
   o en una carpeta externa/personalizada que el usuario indique).
2. **Actualizaciones posteriores:** si el usuario ya indicó una ruta o ya existe
   un `prototipo-<proyecto>.html` previo en esa ubicación, **sobrescribe
   directamente ese mismo archivo** (así el usuario solo recarga el navegador sin
   cambiar de enlace ni acumular copias).

> Usa el formato de cajas/nodos abstractos únicamente si el usuario pide de
> forma explícita un "mapa de flujo" o "diagrama de arquitectura". Por defecto,
> genera siempre la réplica visual interactiva de la UI.

## Flujo de trabajo

1. **Inspecciona el proyecto activo** (sin asumir arquitectura de otro repo):
   - Detecta rutas, menú/navegación, pantallas, tablas/formularios, columnas,
     botones, diálogos y tema visual real (claro/oscuro).
2. **Construye el HTML autocontenido (Solo lectura + Comentarios encima):**
   - Usa Tailwind (`https://www.gstatic.com/antigravity/web/dev/tailwindcss.min.js`)
     y variables CSS con fallbacks en `:root` + `html[data-theme="dark"]` para
     que funcione bien al abrirlo directamente en cualquier navegador.
   - **Réplica visual fiel:** sidebar/riel, barra superior, tema claro/oscuro,
     pantallas navegables, modales y tablas/formularios con datos de muestra
     (100% solo lectura, sin llamar APIs reales ni modificar datos).
   - **Barra superior de revisión e interacción natural:**
     - **Clic izquierdo:** navega libremente por toda la app (sin bloquear
       botones, pestañas ni menús).
     - **Clic derecho:** coloca un pin numerado (`#1`, `#2`…) sobre esa posición
       y detecta `data-elemento="<Pantalla> › <Elemento>"` (si hay elementos
       acumulados con `Shift`, crea un único comentario agrupándolos todos).
     - **`Shift + clic derecho`:** selecciona o deselecciona múltiples elementos
       de la pantalla (mostrando un borde resaltado y contador); al hacer
       **clic derecho** sin `Shift` (o pulsar el botón de comentar selección),
       los agrupa en un solo comentario.
     - **`Ctrl + arrastrar` un pin existente:** mueve el pin por la pantalla y
       al soltar detecta automáticamente con `elementFromPoint` el nuevo
       `data-elemento` donde quedó ubicado.
     - Panel derecho con etiquetas (**Cambiar**, **Agregar**, **Quitar**,
       **No tocar**, **Idea / Duda**) y caja de texto libre.
     - Guardado automático en `localStorage` usando una clave única por
       proyecto (por ejemplo `skill_map_<proyecto>_v1`).
     - Botón **Generar / Copiar cambios** con copia al portapapeles en Markdown
       y descarga `.md`.
3. **Cuando el usuario pegue o envíe los comentarios exportados:**
   - Respeta estrictamente las marcas `NO TOCAR` y ejecuta o arma el plan de
     implementación siguiendo la ubicación exacta de cada pin.
