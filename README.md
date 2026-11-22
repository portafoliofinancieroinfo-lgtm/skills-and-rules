# Skills and Rules

Colección sencilla y portable de **skills** y **reglas** para agentes de IA (**Antigravity / Gemini**, **Claude Code**, **Codex**, **Cursor**).

## Skills

Flujos reutilizables que el agente activa cuando los necesita.

| Skill | Qué hace | Archivo |
|---|---|---|
| `skill_map` | Crea un prototipo HTML de solo lectura del proyecto actual para navegarlo, dejar comentarios con pines (clic derecho / `Shift` / `Ctrl + arrastrar`) y exportarlos en Markdown. Pregunta dónde guardar la primera vez y sobrescribe al actualizar. | [`skills/skill_map.md`](skills/skill_map.md) |

## Rules

Reglas permanentes (`trigger: always_on`) que guían el comportamiento del agente en cada sesión.

| Regla | Qué hace | Archivo |
|---|---|---|
| `model-routing` | Revisa las suscripciones y cuotas reales del entorno, balancea tareas entre pools/niveles, protege los modelos escasos (reserva del 35%), decide cuándo ir directo o usar subagentes/Swarm y aplica fallbacks automáticos ante errores o límites (`429`). | [`rules/model-routing.md`](rules/model-routing.md) |

## Instalación rápida (nota para agentes de IA)

> **Nota para la IA que descargue o instale este repositorio:**  
> En este repo, cada skill está guardada directamente como `skills/<nombre>.md` (sin subcarpeta). Si el entorno del agente requiere el formato estándar de carpeta (`<nombre>/SKILL.md`), al copiarla debes ubicarla como `<ruta_skills>/<nombre>/SKILL.md` (por ejemplo, `skills/skill_map.md` → `.../skills/skill_map/SKILL.md`). Las reglas en `rules/<archivo>.md` se copian directo sin renombrar.

- **Por proyecto:** copia `skills/<nombre>.md` como `.agents/skills/<nombre>/SKILL.md` (o `.claude/skills/<nombre>/SKILL.md`) y `rules/<archivo>.md` en `.agents/rules/` (o `.claude/rules/`).
- **Global en Antigravity / Gemini:** copia `skills/<nombre>.md` como `~/.gemini/config/skills/<nombre>/SKILL.md` y `rules/<archivo>.md` en `~/.gemini/config/rules/`.
- **Global en Claude Code:** copia `skills/<nombre>.md` como `~/.claude/skills/<nombre>/SKILL.md` y `rules/<archivo>.md` en `~/.claude/rules/`.
