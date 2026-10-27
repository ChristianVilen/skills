# skills

Agent skills and subagent definitions for Claude Code.

## Install

Both directories are consumed as symlinks, so edits here are live immediately — no copy
step, no sync.

```sh
ln -s "$PWD/skills" ~/.claude/skills
ln -s "$PWD/agents" ~/.claude/agents
```

The links are absolute. Moving or renaming this repo breaks both silently — skills and
agents just stop appearing, with no error. Re-point them afterwards:

```sh
ln -sfn "$PWD/skills" ~/.claude/skills
ln -sfn "$PWD/agents" ~/.claude/agents
```

## Layout

- `skills/` — one directory per skill, each with a `SKILL.md` carrying YAML frontmatter
  (`name`, `description`).
- `agents/` — subagent definitions as markdown with frontmatter (`name`, `description`,
  optional `tools`, `model`, `effort`).

Skills marked `disable-model-invocation: true` are user-invoked only, via `/<name>`. They
won't be offered automatically. Skills marked `user-invocable: false` (the `principle-*`
skills) are hidden from the `/` menu, but the agent can still load them.

## Adding a skill

```
skills/<name>/
├── SKILL.md
└── references/        # optional
```

`SKILL.md` needs frontmatter with `name` matching the directory, and a `description`.

The `description` is the only thing the agent sees when deciding whether to invoke the
skill, so write it for matching, not for humans: name the symptoms and the phrasings that
should trigger it. A vague description means the skill never fires.

Keep `SKILL.md` short. Anything long belongs in `references/`, with `SKILL.md` acting as a
router that says which file to read for which situation — the agent then loads one file
instead of the whole body. `react-best-practices/` is the worked example: an 80-line router
over eight reference files, where the old single-file version pulled 2116 lines into context
for any React question.

## Dependencies

Some skills shell out to tools you need to install separately:

- `git-surgeon` — https://github.com/raine/git-surgeon
- `workmux` — used by the `workmux`, `worktree`, and `coordinator` skills
- `playwright-cli` — used by the `playwright-cli` skill
- `gh` — the GitHub CLI, used by the pstack PR skills and playbooks
- Bun — used by the `watch-pr` and `orch` scripts in `poteto-mode/scripts/`, which install
  their own dependencies on first run
- Node.js — used by the `.mjs` scripts in `poteto-mode` and `reflect`

## Sources

- https://github.com/mattpocock/skills (synced to `4588b32`, 2026-10-05)
- https://github.com/michael-denyer/pstack-claude (synced to `b0bffae`, 2026-10-11). The
  Claude Code port of [pstack](https://github.com/cursor/plugins/tree/main/pstack).
- https://github.com/anthropics/claude-plugins-official, `skill-development` from
  `plugin-dev` (synced to `f359d43`, 2026-10-10)

### pstack local changes

pstack is a plugin, but this repo has it as plain skills. Do these steps again after each
sync:

- Copy `plugins/pstack/skills/*` into `skills/`, but not
  `thermo-nuclear-code-quality-review` (`judo-code-review` replaces it). Copy
  `plugins/pstack/agents/*` and `plugins/pstack/effort-agents/*` into `agents/`.
- Remove the parts for Copilot, Pi, opencode, Gemini CLI and Prime Agent:
  `copilot-tools.md`, `pi-tools.md`, `setup-pstack/copilot.md`, `setup-pstack/scripts/`,
  and the lines that point to them. Keep the Codex parts.
- Remove the plugin prefix: `pstack:<agent>` becomes `<agent>`, and
  `plugin-dev:skill-development` becomes `skill-development`.
- Change "under the installed plugin" script paths to `~/.claude/skills/...`. The
  `check-plan.mjs` marker `installed plugin` becomes `~/.claude/`.
- There is no `SessionStart` routing hook. `setup-pstack`, `poteto-help` and
  `codex-tools.md` say so, and the model sheet has no `session hook` line.

After the steps, this command must print nothing:

```sh
grep -rnE 'installed plugin|<plugin>/|pstack:[a-z]|plugin-dev:[a-z]' skills agents
```
