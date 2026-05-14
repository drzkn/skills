# skills

Personal collection of agent skills for Cursor / Claude Code.

**Repo**: `git@github.com:drzkn/skills.git`

## Structure

```
skills/
└── <skill-name>/
    └── SKILL.md   ← skill definition (frontmatter + instructions)
```

Each skill lives in its own folder under `skills/`. The `SKILL.md` file contains a YAML frontmatter block (`name`, `description`) followed by the workflow the agent must follow when the skill is triggered.

## Available skills

| Skill | Trigger |
|---|---|
| `wethod-log-hours` | "registrar horas", "imputar horas", "log hours", "fichar" |

## Installation
Para instalar la skill de Wethod y utilizarla tanto en Cursor como en Claude Code, ejecuta:

```bash
npx skills@latest add drzkn/<SKILL NAME>
```

Este comando instalará la skill `wethod-log-hours` y te permitirá usar los triggers como "registrar horas", "imputar horas", "log hours" o "fichar" en ambos asistentes, siempre que tengas soporte para skills personalizadas.


