---
name: skill_map
description: >-
  Genera un Prototipo Comentable de Solo Lectura (estilo Figma) en un solo HTML
  para el proyecto activo, con 3 vistas (App 1:1, Mapa de tablas y Ventanas de
  registro), pines comentables, actualización incremental por diferencias y
  exportación exacta en Markdown.
---

# Prototipo Comentable de Solo Lectura (`skill_map`)

Usa esta skill cuando el usuario pida crear o actualizar el mapa/prototipo
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
   - **Regla anti-cruce de proyectos:** el proyecto objetivo es **el repositorio
     de la conversación actual** (o el que el usuario indique explícitamente). Si
     el archivo abierto en el editor pertenece a un proyecto distinto al de la
     conversación actual, **no cambies de proyecto ni sobrescribas otro mapa**
     sin confirmar primero.
   - Detecta rutas, menú/navegación, pantallas, tablas/formularios, columnas,
     botones, diálogos y tema visual real (claro/oscuro).
   - Lee las hojas de estilo reales del proyecto (`estilo.css`, `index.css`,
     tokens `:root` y `[data-theme="dark"]`), los componentes raíz (`App.tsx` /
     `__root.tsx`) y los iconos vectoriales reales usados (p. ej. `lucide-react`).
2. **Construye el HTML autocontenido (Solo lectura + Comentarios encima):**
   - **Reglas obligatorias de fidelidad visual (1:1):**
     - **Prohibido usar emojis como iconos de interfaz:** extrae e incrusta los
       `<svg>` vectoriales exactos de la librería de iconos del proyecto (p. ej.
       `lucide-react`) respetando `viewBox`, `stroke="currentColor"`,
       `stroke-width` y tamaño real.
     - **Reutiliza el CSS real del proyecto:** copia e incrusta directamente los
       tokens y reglas CSS reales, aislando las reglas de `body` al contenedor
       `#lienzo-app` para que la barra superior (`#barra-mapa`) y el panel de
       comentarios (`#panel-mapa`) no deformen el layout interno. Solo incluye
       Tailwind si el proyecto real usa Tailwind.
     - **Proporciones reales desde el inicio:** cuando no haya comentarios
       guardados (`pines.length === 0`), el panel lateral derecho de comentarios
       inicia **colapsado** para que el lienzo conserve el ancho real de la app;
       se abre automáticamente al crear un pin con clic derecho o al pulsar
       **Comentarios (N)**.
     - **Estructura y micro-interacciones reales:** replica textos exactos,
       subpestañas, acordeones, modales/overlays y logo local (incrustado en
       base64 si es liviano), 100% solo lectura y sin llamar APIs reales.
   - **Barra superior de revisión e interacción natural:**
     - **Clic izquierdo:** navega libremente por toda la app (sin bloquear
       botones, pestañas ni menús).
     - **Clic derecho:** coloca un pin numerado (`#1`, `#2`…) sobre esa posición
       y detecta `data-elemento="<Pantalla> › <Elemento>"` (si hay elementos
       acumulados con `Shift`, crea un único comentario agrupándolos todos).
     - **`Shift + clic derecho`:** selecciona o deselecciona múltiples elementos
       de la pantalla (mostrando borde resaltado y contador); al hacer **clic
       derecho** sin `Shift` (o pulsar el botón de comentar selección), los
       agrupa en un solo comentario.
     - **`Ctrl + arrastrar` un pin existente:** mueve el pin por la pantalla y
       al soltar detecta automáticamente con `elementFromPoint` el nuevo
       `data-elemento` donde quedó ubicado.
     - Panel derecho con etiquetas (**Cambiar**, **Agregar**, **Quitar**,
       **No tocar**, **Idea / Duda**) y caja de texto libre.
     - Guardado automático en `localStorage` usando una clave única por
       proyecto (`skill_map_<proyecto>_v1`).
     - Botón **Generar / Copiar cambios** con copia al portapapeles en Markdown
       y descarga `.md`.
     - **Estándar de 3 vistas en `#barra-mapa`:**
       - **`App 1:1`**: réplica interactiva navegable completa.
       - **`Mapa de tablas`**: atlas desplazable con índice sticky (`Ir a tabla:`),
         todas las tablas del sistema **reutilizando los mismos datos/renderers**
         de `App 1:1` (sin duplicar HTML) y botones `Abrir en App 1:1`.
       - **`Ventanas de registro`**: galería con índice sticky mostrando todos
         los paneles laterales, modales, diálogos, ajustes y popovers con botón
         `Probar en App 1:1`.
       - Pines, `Shift + clic derecho` y `Ctrl + arrastrar` operativos en las 3
         vistas.
3. **Protocolo de actualización incremental por diferencias (ahorro de tiempo y tokens):**
   - Incluir en la línea 2 de cada `prototipo-<proyecto>.html`:
     `<!-- MAPA_META: proyecto=<nombre> | commit=<git-HEAD> | actualizado=<YYYY-MM-DD HH:MM> -->`
   - Dividir el `<script>` en bloques estables marcados con comentarios:
     `/* MAPA:DATOS */`, `/* MAPA:MODALES */`, `/* MAPA:PANTALLA:<id> */`,
     `/* MAPA:ATLAS_TABLAS */`, `/* MAPA:ATLAS_VENTANAS */`.
   - El propio `prototipo-<proyecto>.html` es la fuente de verdad editable en
     sitio (evitar regenerar el HTML completo desde cero cuando ya exista).
   - **Auto-sincronización tras cambios visibles de UI:** al terminar cambios
     visibles en pantallas, tablas, columnas, tarjetas, botones o modales,
     aplicar el diff inmediatamente con edición puntual sobre los bloques
     `/* MAPA:... */` afectados (en sesiones con Swarm, delegarlo al worker de
     UI/frontend) y actualizar `MAPA_META`.
   - **En una conversación nueva al pedir actualizar el mapa:** leer solo
     `MAPA_META` (líneas 1–5), ejecutar `git status --short` y
     `git diff <commit> --stat` para identificar únicamente los archivos
     modificados desde la última actualización, leer solo ese diff y parchear
     únicamente los bloques `/* MAPA:... */` correspondientes.
   - Al terminar cualquier parche, validar la sintaxis JS del `<script>` con
     `node --check` y confirmar `0` emojis Unicode de interfaz.
4. **Cuando el usuario pegue o envíe los comentarios exportados:**
   - Respeta estrictamente las marcas `NO TOCAR` y ejecuta o arma el plan de
     implementación siguiendo la ubicación exacta de cada pin.
