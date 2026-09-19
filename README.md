# Cepo plugins and skills

Author plugins once in `plugins/`. Install them as Claude Code plugins or install
their individual skills with `npx skills`. Both use the same source files.

There is no build step, generated skill directory, or commit hook.

## Available skills

| Plugin | Skill | Purpose |
| --- | --- | --- |
| `codebase-health` | `codebase-trivia` | Multiple-choice questions grounded in the current codebase, with explanations and code references after each answer. |

To use the trivia skill locally, start Claude Code in the project you want to quiz
yourself on and load this repository's plugin by its absolute path:

```sh
claude --plugin-dir /absolute/path/to/this-repo/plugins/codebase-health
```

Then run `/codebase-health:codebase-trivia`. You can include a topic or difficulty, such as
`/codebase-health:codebase-trivia authentication, beginner`. The quiz uses the project
where your session is running. By default it asks five questions, one at a time,
mixing architecture, behavior, and recent changes. After each answer it explains
the result with code references, and it ends with a score and suggested review
topics. You can ask for a hint, skip a question, or stop early.

The question count is configurable in plain language, including mid-session. The
skill uses the active harness's native question tool when available and permitted,
and falls back to chat if necessary.

```text
/codebase-health:codebase-trivia 10 questions about authentication, beginner
/codebase-health:codebase-trivia 3 questions, save as Markdown
/codebase-health:codebase-trivia 8 questions, save to docs/codebase-quiz.md
```

Markdown export is optional and is also offered at the end. It includes the
questions, choices, your answers, explanations, source references, score, and
suggested review topics. Without a specified path, an export uses a timestamped
`codebase-trivia-YYYYMMDD-HHMMSS.md` file in the project root. You can also ask to
save current progress during a quiz; unanswered solutions stay hidden.

## Layout

```text
.claude-plugin/
  marketplace.json              Plugin catalog
plugins/
  <plugin-name>/
    .claude-plugin/
      plugin.json               Plugin metadata
    skills/
      <skill-name>/
        SKILL.md                Skill instructions and metadata
        references/             Optional supporting documentation
        scripts/                Optional helpers
        assets/                 Optional assets
```

The marketplace lists each plugin's name and relative source path. The plugin
manifest owns its description and version. Skill content lives only inside its
plugin directory.

## Add a plugin

For example, create `plugins/writing/.claude-plugin/plugin.json`:

```json
{
  "name": "writing",
  "description": "Writing and editing skills.",
  "version": "0.1.0",
  "author": { "name": "Cepo AI" }
}
```

Append an entry to the `plugins` array in `.claude-plugin/marketplace.json`:

```json
{
  "name": "writing",
  "source": "./plugins/writing"
}
```

Paths start with `./` and resolve from the repository root. Keep each plugin's
folder name and manifest name identical. Add another directory and catalog entry
for each additional plugin.

## Add a skill

Create `plugins/writing/skills/draft-summary/SKILL.md`:

```markdown
---
name: draft-summary
description: Summarize supplied text into a concise brief with decisions and open questions.
---

Summarize the supplied text. Preserve decisions, owners, and deadlines when
present. Separate confirmed facts from open questions and do not invent details.
```

Use lowercase letters, digits, and hyphens for names. Match the skill's `name` to
its folder and keep skill names unique across all plugins so standalone installs
are unambiguous. Include a meaningful `description` in every skill's frontmatter.

Keep supporting files inside the skill directory and reference them with relative
paths, such as `references/style-guide.md`. The skills installer includes these
files. Plugin-level hooks, agents, MCP configuration, and files outside the skill
directory are not part of a standalone skill installation. Skills intended for
both installation methods should avoid depending on `${CLAUDE_PLUGIN_ROOT}` or
plugin-only capabilities.

## Validate and preview locally

With the Claude Code CLI installed, validate the marketplace and each new plugin:

```sh
claude plugin validate .
claude plugin validate ./plugins/codebase-health
```

After adding skills, preview what the skills installer discovers (requires Node.js
and npm):

```sh
npx skills add . --list
```

This lists available skills without installing them. To validate another plugin,
replace `./plugins/codebase-health` with its directory. The `writing` paths in this guide
are authoring examples; use those only after creating that plugin.

To try a plugin in a Claude Code session before publishing:

```sh
claude --plugin-dir ./plugins/writing
```

The example skill is available as `/writing:draft-summary`.

## Install from a Git repository

Push this repository to your chosen Git host. In the following examples,
`<owner>/<repo>` is the GitHub repository you publish; no remote is configured by
this scaffold.

In Claude Code:

```text
/plugin marketplace add <owner>/<repo>
/plugin install codebase-health@cepo-plugins
```

For standalone skills, run these from the project where you want to use them:

```sh
npx skills add <owner>/<repo> --list
npx skills add <owner>/<repo> --skill codebase-trivia
```

The skills CLI reads `.claude-plugin/marketplace.json`, follows local plugin source
paths, and discovers each plugin's `skills/<skill-name>/SKILL.md`. No top-level
`skills/` directory is required.

When releasing plugin changes, bump the version in that plugin's `plugin.json`.

## Compatibility check

Verified on September 17, 2026 with `skills@1.7.0`: a temporary marketplace with two
plugins and no top-level `skills/` directory exposed both skills. Installing one
skill into a temporary Claude Code project preserved its `SKILL.md` and supporting
reference file byte for byte. Claude's marketplace validator also accepted that
layout. This checked local discovery and installation; remote installation needs
a published repository.

References:

- [Claude Code plugin marketplaces](https://code.claude.com/docs/en/plugin-marketplaces)
- [Skills CLI plugin manifest discovery](https://github.com/vercel-labs/skills#plugin-manifest-discovery)
