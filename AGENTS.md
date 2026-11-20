# Proyecto `skills-and-rules`

Repositorio público y agnóstico de plataforma con skills reutilizables (`skills/`) y reglas de comportamiento (`rules/`) para agentes de programación con IA (Antigravity, Gemini, Claude Code, Codex, Cursor).

## Convenciones de este repositorio

- `skills/<nombre>/SKILL.md`: instrucciones autocontenidas bajo el estándar `SKILL.md` (frontmatter YAML con `name` y `description`).
- `rules/<nombre>.md`: reglas globales o reutilizables (con frontmatter `trigger: always_on` cuando aplique).
- **Portabilidad obligatoria:** no incluir rutas locales privadas, credenciales, nombres de usuario, proyectos privados ni acoplar reglas a versiones caducables o a una única suscripción.
