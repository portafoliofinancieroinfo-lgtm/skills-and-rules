# Skills and Rules

Repositorio público con **skills** y **reglas globales** portables para agentes de desarrollo con IA (**Antigravity / Gemini**, **Claude Code**, **OpenAI Codex**, **Cursor** y otros agentes compatibles).

Todo lo publicado aquí es general e independiente de rutas locales o suscripciones fijas.

## Contenido

| Tipo | Nombre | Descripción | Archivo |
|---|---|---|---|
| Skill | `skill_map` | Genera un prototipo HTML comentable de solo lectura (estilo Figma) para el proyecto abierto en ese momento, con pines, agrupación (`Shift + clic derecho`), reubicación (`Ctrl + arrastrar`) y exportación en Markdown. Pregunta dónde guardar la primera vez y sobrescribe al actualizar. | [`skills/skill_map/SKILL.md`](skills/skill_map/SKILL.md) |
| Regla | `model-routing` | Regla de orquestación multi-modelo y subagentes: inspecciona suscripciones/cuotas activas, balancea trabajo diario entre pools, protege reservas en modelos premium, elige entre modo directo/subagente/workflow/enjambre y aplica fallbacks reales ante `429` o caídas. | [`rules/model-routing.md`](rules/model-routing.md) |

## Instalación rápida

### Por proyecto (cualquier agente compatible)
- **Skills:** copia `skills/<nombre>/` dentro de `.agents/skills/` (Antigravity / Codex) o `.claude/skills/` (Claude Code) en tu repositorio.
- **Reglas:** copia `rules/model-routing.md` en `.agents/rules/` (Antigravity) o agrega su contenido/referencia en `.claude/rules/` / `AGENTS.md`.

### Global (en tu equipo)
- **Antigravity / Gemini:**
  - Skills → `~/.gemini/config/skills/`
  - Reglas → `~/.gemini/config/rules/`
- **Claude Code:**
  - Skills → `~/.claude/skills/`
  - Reglas → `~/.claude/rules/` (o referenciado desde `~/.claude/CLAUDE.md`)
