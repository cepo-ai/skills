# Cepo skills

Reusable AI skills for Claude Code and other coding agents.

## Skills

| Skill | Description |
| --- | --- |
| [codebase-trivia](plugins/codebase-health/skills/codebase-trivia/SKILL.md) | Interactive quizzes about your codebase, with explanations and code references. |

## Claude Code

Start Claude Code in the project you want to use the skill with, then install:

```text
/plugin marketplace add cepo-ai/skills
/plugin install codebase-health@cepo-plugins
```

Follow any activation instructions shown after installation, then run:

```text
/codebase-health:codebase-trivia
```

## npx skills

With Node.js and npm installed, run this from your project directory:

```sh
npx skills add cepo-ai/skills --skill codebase-trivia
```

Choose your agent during installation, then ask it:

> Use codebase-trivia to quiz me on this project.
