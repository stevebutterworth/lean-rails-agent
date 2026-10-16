# lean-rails-agent

A global [Kiro](https://kiro.dev) agent for Rails and git work on a tight token budget. Install it
once and use it in any Rails repo: nothing is copied into the project.

Hooks stop the agent wasting context on discovery, large reads, verbose command output and
whole-file rewrites. Kiro hooks can block a command but not rewrite it, so every block names the
compact alternative.

## Install

```bash
git clone https://github.com/stevebutterworth/lean-rails-agent ~/dev/lean-rails-agent
~/dev/lean-rails-agent/bin/install
```

This links the repo to `~/.kiro/lean-rails` and writes `~/.kiro/agents/rails.json`. Then, in any
Rails repo:

```bash
kiro-cli chat --agent rails            # or once: kiro-cli agent set-default rails
```

Needs `rg`, `python3` and `kiro-cli`. Rerun `bin/install` after pulling changes to `agent.json`;
everything else is live through the link.

## What it does

- **Three tools only** (`fs_read`, `fs_write`, `execute_bash`), no MCP: every tool schema is sent
  on every request, so unused ones cost tokens each turn
- **`rules.md`**, sent every turn: terse-output rules, the compact commands, and the coding style
  (vanilla Rails, 37signals, no service objects, deep modules). The project's own `AGENTS.md` is
  loaded too if it has one. The `37signals-style` skill in `~/.kiro/skills` is attached as a
  skill, so its full text loads only when needed
- **`hooks/context`** (spawn): prints branch, Ruby version, test framework, recent commits,
  pending migrations and open ports so the agent skips discovery turns
- **`hooks/guard-bash`**: blocks dangerous commands (destructive git, db:drop, rm -rf, git push)
  and verbose ones, pointing at the compact form:

  | Blocked | Use instead |
  |---|---|
  | `rspec`, `rails test` | `bin/test [files]` (full suite once per session) |
  | `rubocop` | `bin/lint [-a] [files]` |
  | `rails routes` | `rails routes -g pattern` |
  | `git log` without a limit | `git log --oneline -n 10` |
  | `git diff` / `git show` bare | `--stat`, then `-U1 -- path` |
  | `git status` | `git status -sb` |
  | `grep -r`, `tree`, `ls -R`, bare `find` | `rg -n`, `rg --files \| rg pattern` |
  | `cat` on a log, or on any file over 200 lines | `tail -n 50`, `rg -n` then `sed -n 'A,Bp'` |
  | `bundle install` | `bundle install --quiet` |

  Prefix `RAW=1` to bypass the token rules (not the safety rules).
- **`hooks/guard-read`**: blocks whole-file reads over 200 lines without a line range, and
  directory listings deeper than 1
- **`hooks/guard-write`**: blocks rewriting an existing file over 40 lines with `create`; the
  agent must use `str_replace`. Output tokens cost the most, and a rewrite regenerates the file

The agent calls these by their full path, `~/.kiro/lean-rails/bin/...`:

- **`bin/test`**: runs rspec if the app has it, otherwise `rails test`. Prints failure details,
  the summary and the rerun list only (no dots, no pending section), capped at 80 lines
- **`bin/lint`**: rubocop in emacs format (one line per offense) on files changed against the
  default branch plus uncommitted ones

## Files

```
agent.json        agent config template (~ expanded by bin/install)
rules.md          always-on rules and coding style
bin/install       installs the agent globally
bin/test          compact test runner
bin/lint          compact rubocop
hooks/context     agentSpawn hook
hooks/guard-bash  preToolUse hook for execute_bash
hooks/guard-read  preToolUse hook for fs_read
hooks/guard-write preToolUse hook for fs_write
```

## Habits that save more than any config

- One session per task; `/compact` long ones instead of carrying stale context.
- Keep `model: auto`: its credit multiplier is lower than pinning a model (check Kiro's current rates).
- Keep `rules.md` and project `AGENTS.md` files to what the agent needs on most turns.

## Based on

The ej agent from [Working Planet](https://www.workingplanet.com), stripped to the generic core.
