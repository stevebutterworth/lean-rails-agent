# lean-rails-agent

A lean [Kiro](https://kiro.dev) agent for Rails projects, built for tight token budgets. Hooks stop
the agent wasting context on discovery, large reads, verbose command output and whole-file
rewrites. Kiro hooks can block a command but not rewrite it, so every block names the compact
alternative.

## What it does

- **Three tools only** (`fs_read`, `fs_write`, `execute_bash`), no MCP: every tool schema is sent
  on every request, so unused ones cost tokens each turn
- **Spawn hook** (`bin/kiro-context`): prints branch, Ruby version, recent commits, pending
  migrations and open ports so the agent skips discovery turns
- **Bash guard** (`bin/kiro-guard`): blocks dangerous commands (destructive git, db:drop, rm -rf,
  git push) and verbose ones, pointing at the compact form:

  | Blocked | Use instead |
  |---|---|
  | `rspec` in any form | `bin/t [files]` (full suite once per session) |
  | `rubocop` | `bin/lint [-a] [files]` |
  | `rails routes` | `rails routes -g pattern` |
  | `git log` without a limit | `git log --oneline -n 10` |
  | `git diff` / `git show` bare | `--stat`, then `-U1 -- path` |
  | `git status` | `git status -sb` |
  | `grep -r`, `tree`, `ls -R`, bare `find` | `rg -n`, `rg --files \| rg pattern` |
  | `cat` on a log, or on any file over 200 lines | `tail -n 50`, `rg -n` then `sed -n 'A,Bp'` |
  | `bundle install` | `bundle install --quiet` |

  Prefix `RAW=1` to bypass the token rules (not the safety rules).
- **Read guard** (`bin/kiro-guard-read`): blocks whole-file reads over 200 lines without a line
  range, and directory listings deeper than 1
- **Write guard** (`bin/kiro-guard-write`): blocks rewriting an existing file over 40 lines with
  `create`; the agent must use `str_replace`. Output tokens cost the most, and a rewrite
  regenerates the whole file
- **`bin/t`**: rspec with failure details, summary and rerun list only (no dots, no pending
  section), capped at 80 lines. Overrides any `--format documentation` in `.rspec`
- **`bin/lint`**: rubocop in emacs format (one line per offense) on files changed against the
  default branch plus uncommitted ones
- **`.ignore`**: keeps `coverage/`, `log/`, `tmp/`, `vendor/`, assets and `node_modules/` out of
  `rg` results
- **`AGENTS.md`**: terse-output rules and the compact commands, kept short because it is sent on
  every turn

## Files

```
your-rails-repo/
  .ignore                 ← paths rg skips
  .kiro/
    agents/rails.json     ← agent wiring (tools, hooks, resources)
    settings/cli.json     ← sets this as the default agent
  AGENTS.md               ← your project rules (sent every turn)
  bin/
    kiro-context          ← agentSpawn hook
    kiro-guard            ← preToolUse hook for bash
    kiro-guard-read       ← preToolUse hook for file reads
    kiro-guard-write      ← preToolUse hook for file writes
    t                     ← compact rspec
    lint                  ← compact rubocop
```

## Setup

From your Rails repo root:

```bash
src=/path/to/lean-rails-agent
mkdir -p .kiro/agents .kiro/settings
cp $src/.kiro/agents/rails.json .kiro/agents/
cp $src/.kiro/settings/cli.json .kiro/settings/
cp $src/.ignore .
cp $src/bin/{kiro-context,kiro-guard,kiro-guard-read,kiro-guard-write,t,lint} bin/
chmod +x bin/{kiro-context,kiro-guard,kiro-guard-read,kiro-guard-write,t,lint}
```

Then edit `AGENTS.md` for your project. Needs `rg` and `python3` on the path.

## Habits that save more than any config

- One session per task; `/compact` long ones instead of carrying stale context.
- Keep `model: auto`: its credit multiplier is lower than pinning a model (check Kiro's current rates).
- Keep `AGENTS.md` to what the agent needs on most turns.

## Customising

- **`bin/kiro-context`**: add or remove spawn facts (e.g. a Redis/Sidekiq check)
- **`bin/t`**: swap `bundle exec rspec` for your runner; the guard's rspec rule needs the same change
- **`bin/kiro-guard`**: thresholds and patterns are one line each

## Based on

The ej agent from [Working Planet](https://www.workingplanet.com), stripped to the generic core.
