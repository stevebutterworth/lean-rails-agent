# lean-rails-agent

A lean [Kiro](https://kiro.dev) agent for Rails projects. Token-efficient by design: three hooks
stop the agent wasting context on file discovery, large reads, and redundant commands.

## What it does

- **Spawn hook** — prints branch, Ruby version, recent commits, pending migrations and open ports
  at session start so the agent skips discovery turns
- **Bash guard** — blocks dangerous commands (destructive git, db:drop, rm -rf) and token-wasteful
  ones (cat on large files, redundant cd prefixes, running the full suite twice)
- **Read guard** — blocks whole-file reads over 200 lines without a line range, forcing
  grep-then-read discipline

## Files

```
your-rails-repo/
  .kiro/
    agents/
      rails.json          ← agent wiring (hooks, tools, resources)
    settings/
      cli.json            ← sets this as the default agent
  AGENTS.md               ← your project rules (loaded into context at spawn)
  bin/
    kiro-context          ← agentSpawn hook
    kiro-guard            ← preToolUse hook for bash
    kiro-guard-read       ← preToolUse hook for file reads
```

## Setup

Copy the files into your Rails repo:

```bash
# from your rails repo root
mkdir -p .kiro/agents .kiro/settings

cp /path/to/lean-rails-agent/.kiro/agents/rails.json   .kiro/agents/
cp /path/to/lean-rails-agent/.kiro/settings/cli.json   .kiro/settings/
cp /path/to/lean-rails-agent/bin/kiro-context          bin/
cp /path/to/lean-rails-agent/bin/kiro-guard            bin/
cp /path/to/lean-rails-agent/bin/kiro-guard-read       bin/

chmod +x bin/kiro-context bin/kiro-guard bin/kiro-guard-read
```

Then edit `AGENTS.md` with your project's conventions and `rails.json` resources list if needed.

## Customising

- **`bin/kiro-context`** — add or remove the facts printed at spawn (e.g. swap rspec for minitest,
  add a Redis/Sidekiq check)
- **`bin/kiro-guard`** — the full-suite pattern matches `bundle exec rspec`; swap for your runner
- **`AGENTS.md`** — the agent reads this at every session start; put commands, conventions, and
  safety rules here

## Based on

The ej agent from [Working Planet](https://www.workingplanet.com), stripped to the generic core.
