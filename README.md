# Skills and Rules

Repositorio público con **skills** y **reglas globales** portables para agentes de desarrollo con IA (**Antigravity / Gemini**, **Claude Code**, **OpenAI Codex**, **Cursor** y otros agentes compatibles con `SKILL.md`).

Todo el contenido está diseñado para ser **general e independiente de máquina**: sin rutas locales privadas, credenciales ni acoplamiento a una única suscripción o proveedor.

---

## Estructura del repositorio

```text
skills-and-rules/
├── README.md                  # Documentación completa de uso, instalación y convenciones
├── skills/
│   └── skill_map/
│       └── SKILL.md           # Skill de prototipo comentable de solo lectura (estilo Figma)
└── rules/
    └── model-routing.md       # Regla global de orquestación, subagentes y enrutamiento multi-modelo
```

> Nota: cada skill vive en `skills/<nombre>/SKILL.md` (exigido por el estándar `SKILL.md` para que los agentes la detecten automáticamente). En la vista web de GitHub, mientras `skills/` tenga una sola subcarpeta puede mostrarse colapsada como `skills/skill_map`; al agregar más skills aparecerán separadas dentro de `skills/`.

---

## Contenido en detalle

### 1. Skill: `skill_map` — [`skills/skill_map/SKILL.md`](skills/skill_map/SKILL.md)

Genera un **Prototipo Comentable de Solo Lectura (`prototipo-<proyecto>.html`)** para el proyecto que esté abierto en ese momento, permitiendo revisar e iterar cambios visuales o funcionales antes de tocar código:

- **Réplica visual fiel en un solo HTML autocontenido** (Tailwind + variables CSS para tema claro/oscuro, 100% solo lectura con datos de muestra, sin llamar APIs reales).
- **Interacción natural:**
  - **Clic izquierdo:** navega libremente por pantallas, pestañas, modales y tablas.
  - **Clic derecho:** coloca un pin numerado (`#1`, `#2`…) detectando automáticamente el elemento bajo el cursor (`data-elemento`).
  - **`Shift + clic derecho`:** selecciona múltiples elementos y los agrupa en un único comentario.
  - **`Ctrl + arrastrar`:** reubica un pin existente y actualiza automáticamente el elemento apuntado.
  - **Etiquetas y persistencia:** clasifica cada comentario (`Cambiar`, `Agregar`, `Quitar`, `No tocar`, `Idea / Duda`) con guardado automático en `localStorage` y exportación en Markdown (portapapeles o archivo `.md`).
- **Ubicación inteligente:** pregunta dónde guardar `prototipo-<proyecto>.html` la primera vez y **sobrescribe el mismo archivo** en actualizaciones posteriores para no duplicar copias ni cambiar de enlace en el navegador.

### 2. Regla: `model-routing` — [`rules/model-routing.md`](rules/model-routing.md)

Regla permanente (`trigger: always_on`) para decidir el **modo agéntico** y **balancear modelos y subagentes** ajustándose automáticamente a las suscripciones y límites reales del entorno:

- **Descubrimiento dinámico de suscripciones y pools:** inspecciona modelos, proveedores y cuotas disponibles agrupando modelos que comparten límite y ventana de reinicio; se adapta automáticamente a entornos multi-pool (reserva obligatoria del **35%** en el pool premium semanal y equilibrio entre ventana corta y pool secundario), un solo proveedor (balanceo por niveles `Ligero → Medio → Alto`), API pago por uso o entornos sin visibilidad de cuota (modo conservador).
- **4 ejes para elegir modo:** evalúa riesgo/reversibilidad, modo según orquestador (1 capa o 1–2 archivos → directo; ≥ 2 capas independientes → Swarm de máx. 3–4 workers; paralelo solo si son 100% independientes y en pools distintos ≥ 40%, secuencial si hay dependencia o cuota < 40%), tamaño de contexto y complejidad (*"Planifica alto, ejecuta medio"*).
- **Resiliencia ante fallos:** ante `429` o caída del proveedor **no sube el esfuerzo**, sino que rota a otro pool independiente disponible conservando el contexto; escala esfuerzo solo tras **2 intentos técnicos fallidos**; pausa y avisa antes de degradar tareas críticas (autenticación, pagos, seguridad, producción o migraciones de BD).

---

## Instalación rápida

### Por proyecto (cualquier agente compatible)
- **Antigravity / Gemini / Codex:**
  - Copia `skills/skill_map/` en `.agents/skills/skill_map/` de tu repositorio.
  - Copia `rules/model-routing.md` en `.agents/rules/model-routing.md` (para Codex/agentes que leen instrucciones en raíz, referencia su contenido desde `AGENTS.md`).
- **Claude Code:**
  - Copia `skills/skill_map/` en `.claude/skills/skill_map/` de tu repositorio.
  - Copia `rules/model-routing.md` en `.claude/rules/model-routing.md`.

### Global (en todo tu equipo)
- **Antigravity / Gemini:**
  - Skills → `~/.gemini/config/skills/` (`%USERPROFILE%\.gemini\config\skills\` en Windows)
  - Reglas → `~/.gemini/config/rules/` (`%USERPROFILE%\.gemini\config\rules\` en Windows)
- **Claude Code:**
  - Skills → `~/.claude/skills/` (`%USERPROFILE%\.claude\skills\` en Windows)
  - Reglas → `~/.claude/rules/` (`%USERPROFILE%\.claude\rules\` en Windows)

> **Tip multi-PC (sincronización en nube):** guarda tus carpetas `skills/` y `rules/` en la nube (OneDrive, Google Drive, Dropbox) y enlázalas a `~/.gemini/config/skills` y `~/.gemini/config/rules` con `mklink /J` (Windows) o `ln -s` (macOS/Linux).

---

## Convenciones para contribuir o extender este repositorio

- **Skills (`skills/<nombre>/SKILL.md`):** usar el formato estándar `SKILL.md` (frontmatter YAML con `name` y `description`).
- **Reglas (`rules/<nombre>.md`):** incluir `trigger: always_on` cuando deban aplicarse en cada sesión.
- **Portabilidad obligatoria:** no incluir rutas privadas locales, credenciales, nombres de usuario, proyectos privados ni acoplar reglas a versiones o suscripciones específicas.
