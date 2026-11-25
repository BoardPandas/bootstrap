# Project Rules

Claude Code starter template providing a ready-to-use `.claude/` config (with generated Codex and Antigravity mirrors).

## Commands

```bash
npm run check:claude   # wiring guard: rule scoping, hook matchers, frontmatter, budgets, mirror drift
npm run sync:codex     # regenerate AGENTS.md, .agents/, .codex/ after any CLAUDE.md or .claude/ change
npm test               # asserts every guard check still fires, plus commit-gate and hook-adapter paths
```

## Verifying your work

A change to this repo is **not done** until both are green:

```bash
npm run check:claude
npm test
```

Healthy output ends with `OK -- .claude wiring verified.` and `# fail 0`. Never skip, delete, or narrow a failing check — checks exist because these defects are otherwise silent. If a check is wrong, change it deliberately and document why in the changelog.

## Workflow: Plan First, Then Init

1. **`/plan-repo`** — choose stack, generate README, create design guardrails.
2. **"initialize repo"** — configure `.claude/` using the plan.
3. **"update practices"** — stay current periodically.

Skills marked `/command` set `disable-model-invocation: true`, so plain-English phrases will not start them.

## Artifact Chain

One artifact, one owner, one approval. Never skip straight to a diff for work larger than one file.

| Artifact | Location | Produced by | Approved by |
|---|---|---|---|
| `intent.md` | `intent/<slug>/intent.md` | `/capture-intent` | Product owner |
| `spec.md` | `intent/<slug>/spec.md` | `/spec-developer` | Product owner |
| `plan.md` | `tasks/<slug>-plan.md` | plan mode / `/spec-developer` | Engineer |
| diff | git branch | `builder` / session | Code owner, per `REVIEW.md` |
| lesson | LL-G / BP | `/add-lesson`, `/add-practice` | Authorized GitHub commit; CI |

## Review Policy

`REVIEW.md` defines passes, what "Important" means, the nit cap, and exclusions. Read it before reviewing or acting on review feedback; it is version-controlled policy.

## Coding Standards

- Handle errors explicitly -- never swallow exceptions silently.
- Validate inputs at system boundaries (user input, API responses, file I/O).
- Avoid premature abstraction. Three similar lines beat a forced helper.
- Comment only non-obvious "why", not self-explanatory code.
- Split files over 500 lines.

## Things Claude Gets Wrong

Corrections needed twice; move generalisable entries to LL-G via `/add-lesson`.

- **Folding changelog edits into the commit command.** `PreToolUse` runs *before* the command (`edit && git commit` cannot satisfy it). Stage first, commit second.
- **Writing `Tool(pattern)` in hook `matcher:`.** That is permissions syntax and silently matches nothing. Use `matcher: "Bash"` + `if: "Bash(git *)"`.
- **Using `$CLAUDE_FILE_PATH` in hooks.** Nonexistent; expands to `""` and makes formatters walk the repo. Parse `tool_input.file_path` from stdin and hard-guard non-empty.
- **Picking a JSON parser via `command -v python3`.** Finds the WindowsApps stub on Windows; probe by running a candidate against a known payload (`_json-parser.sh`).
- **Dead `paths:` globs.** Match nothing silently; `npm run check:claude` fails unless recorded in `wiring-exemptions.json`.
- **Cursor `globs:` / `alwaysApply:` in `.claude/rules/`.** Claude Code ignores both, loading the rule in every session.
- **Assuming worktree agents see uncommitted work.** Branching from a commit hides uncommitted edits; orient with `git status --short` first.
- **Line-count context budgets.** Gamed by long lines; budget by bytes (`wc -c`).
- **Echoing advice from PreToolUse/PostToolUse hooks.** Plain stdout goes to the debug log; emit `additionalContext` JSON via `emit_context` (`.claude/scripts/_hook-context.sh`), or exit 2 + stderr to block.
- **Cwd-relative hook paths.** Hooks follow `cd`; write `bash "$CLAUDE_PROJECT_DIR"/.claude/scripts/x.sh` (or `bash ../.claude/scripts/_antigravity-hook.sh` in `.agents/hooks.json` for Unix + Windows `cmd /c`).

## Hierarchical CLAUDE.md Architecture

Loads top-down: user -> project root -> subfolder.

- Root `CLAUDE.md` — project-wide rules, stack, global conventions.
- `.claude/rules/*.md` — conditional rules with `paths:` frontmatter (omitting `paths:` loads **every** session). Prefer `.claude/rules/*.md` over subfolder `CLAUDE.md` when targeting Antigravity, because Antigravity does not load subfolder rules below the workspace root.
- Nested `.claude/` directories (`subdir/.claude/skills|agents|workflows/`) load automatically; closest wins (`<dir>:<name>`).
- Prune after model updates; do not bloat with generic advice.

## Subagent Usage

Offload online research, doc fetching, log analysis, and codebase exploration to subagents to keep main context narrow.

- **Include a "why"** in every subagent prompt.
- **Parallel exploration:** Spin up parallel custom `explorer` subagents (not built-in `Explore`, which loads every MCP schema).
- Subagents are resumable, run in the background by default, and nest up to 5 levels deep.

## Frontmatter

Full catalog in `instructions.md`. Three enforced rules (`npm run check:claude`):

- **Model resolution:** declare `model:` directly, or bind `agent:` alongside `context: fork` (without the fork, `agent:` is ignored).
- **Hyphenated keys only:** underscored forms (`disable_model_invocation`, `allowed_tools`) are silently ignored.
- **Forked skill tools:** `allowed-tools` only pre-approves; every entry must be in the bound agent's `tools:`.

## Infrastructure Profiles

Server-side projects deploy whole onto **Cloudflare** or **Railway** (fixed on both: Cloudflare R2, Resend, Better Auth); local-only desktop apps get neither. Full specs in `.claude/references/infrastructure.md` (read-only).

## File Organization

- Keep `.claude/` self-contained (no external refs except `CLAUDE.md`, `AGENTS.md`, `README.md`, `REVIEW.md`, `instructions.md`).
- Skills: `.claude/skills/<name>/SKILL.md`; agents: `.claude/agents/<name>.md`; rules: `.claude/rules/*.md`; hook scripts: `.claude/scripts/*.sh`; memory: `.claude/agent-memory/`.
- Intents/specs: `intent/<slug>/`; plans: `tasks/`.
- References: `source-urls.md`, `infrastructure.md`, `tools.md`, `design-guardrails.md`, `template-sync-state.json`, `template-sync-ignore.md`, `wiring-exemptions.json`, `managed-settings.example.json`.
- Mirrors: `AGENTS.md`, `.agents/`, `.codex/` are generated by `scripts/sync-codex.mjs` (+ `sync-antigravity.mjs` / `antigravity-format.mjs`). Never hand-edit; see `.agents/README.md` and `.codex/README.md`.
- Wiring guard: `scripts/check-claude-wiring.mjs` (every check is an error; stale exemptions fail; tested by `npm test`).
- Settings: `.claude/settings.json` (project), `.claude/settings.local.json` (git-ignored personal overrides).

## Hooks and Settings

**Read `.claude/references/hooks-and-settings.md` before editing hooks or settings.** Canonical catalog of all 30 hook events, optional settings keys, and gotchas. Hook types: `command`, `http`, `prompt`, `agent`, `mcp_tool`.

## Planning

- Phase-based only (Foundation, Core, Polish, Ship) — never timelines.
- Plan in one session, execute in another; save plans to `tasks/`; use `/spec-developer` for multi-file features.
- Every plan ends with **Lessons Learned / Gotchas**. After implementation, route discoveries to LL-G via `/add-lesson` and add a check to `scripts/check-claude-wiring.mjs` when statically detectable.

## Context Management

- Keep `CLAUDE.md` under 16 KB and always-on context under 20 KB (`wc -c`); `npm run check:claude` enforces both, and `sync:codex` caps `AGENTS.md` at 24,000 B for Antigravity.
- Keep tasks under 50% context; use `/compact` proactively and fresh sessions for unrelated topics.
- **Preserve prompt cache:** lock MCP/tools and model at session start.
- **Code bias fix:** if stuck, prototype isolated in a fresh folder then port in.
- Write failed attempts to `.claude/agent-memory/debugging.md`; use `/handoff` before ending a session.

## Date Awareness

Check today's date before fetching best practices or recommending versions.

## Available Skills and Agents

Skills auto-discover from `.claude/skills/`; full tables in `README.md`, `instructions.md`, and `.claude/references/agent-registry.md`.

- Slash-only (`disable-model-invocation: true`): `/capture-intent`, `/plan-repo`, `/spec-developer`, `/mermaid-diagram`, `/merge-worktrees`, `/triage-issues`.
- Pipeline: `/capture-intent` -> `/plan-repo` -> "initialize repo" -> `/spec-developer` -> build -> "update practices".

## Workflow

1. Read URLs from `.claude/references/source-urls.md` -- never hardcode URLs in skills.
2. Check `.claude/references/tools.md` before running CLI tools; offer to install missing ones.

## RULE 0: Read-Only First (MANDATORY)

**Gather information before taking action. Read-only commands first. Modifications only with user approval.**

- Always safe: diagnostics and read-only operations (`Get-*`, `Test-*`, queries, list/read API calls).
- Requires approval: state-changing commands (`Set-*`, `Remove-*`, `Stop-*`, writes to shared systems).
- Never without explicit request: destructive operations (recursive deletes, credential resets, formatting). (BP `safety/read-only-first-rule`.)

## RULE 1 -- Check LL-G Before Scripting (MANDATORY)

**Before writing scripts, API calls, or automation, fetch `https://raw.githubusercontent.com/BoardPandas/LL-G/main/llms.txt`, then `kb/<tech>/llms.txt` for each technology used. Read ALL HIGH-severity entries plus matching MEDIUM entries.**

Includes `kb/claude-code/` and `kb/antigravity-cli/` for config/hook work. Full procedure in `.claude/rules/llg-check.md`. Route new discoveries back via `/add-lesson` (atomic GitHub commit with `gh`).

## RULE 3 -- Check BP Before Starting New Work

**When onboarding a repo, starting a feature, or setting up tooling, fetch `https://raw.githubusercontent.com/BoardPandas/BP/main/llms.txt`, then `practices/<concern>/llms.txt`. Load ALL FOUNDATIONAL entries plus matching RECOMMENDED entries.** Full procedure in `.claude/rules/bp-check.md`.
