# Skills and Rules

Colección sencilla y portable de **skills** y **reglas** para agentes de IA (**Antigravity / Gemini**, **Claude Code**, **Codex**, **Cursor**).

## Skills

Flujos reutilizables (`SKILL.md`) que el agente activa cuando los necesita.

| Skill | Qué hace | Archivo |
|---|---|---|
| `skill_map` | Crea un prototipo HTML de solo lectura del proyecto actual para navegarlo, dejar comentarios con pines (clic derecho / `Shift` / `Ctrl + arrastrar`) y exportarlos en Markdown. Pregunta dónde guardar la primera vez y sobrescribe al actualizar. | [`skills/skill_map/SKILL.md`](skills/skill_map/SKILL.md) |

## Rules

Reglas permanentes (`trigger: always_on`) que guían el comportamiento del agente en cada sesión.

| Regla | Qué hace | Archivo |
|---|---|---|
| `model-routing` | Revisa las suscripciones y cuotas reales del entorno, balancea tareas entre pools/niveles, protege los modelos escasos (reserva del 35%), decide cuándo ir directo o usar subagentes/Swarm y aplica fallbacks automáticos ante errores o límites (`429`). | [`rules/model-routing.md`](rules/model-routing.md) |

## Instalación rápida

- **Por proyecto:** copia `skills/<nombre>/` en `.agents/skills/` (o `.claude/skills/`) y `rules/<archivo>.md` en `.agents/rules/` (o `.claude/rules/`).
- **Global en Antigravity / Gemini:** copia o enlaza hacia `~/.gemini/config/skills/` y `~/.gemini/config/rules/`.
- **Global en Claude Code:** copia o enlaza hacia `~/.claude/skills/` y `~/.claude/rules/`.
