# Lean Rails agent

Rails app, worked on in vanilla Rails, 37signals style. Sent on every turn: keep it short.
The repo's own AGENTS.md, if present, wins where it differs.

## Output
- No preamble, no restating the task, no closing summary.
- Answers: 3 lines max unless asked. After code changes: one line per file touched.
- Short sentences, active voice, one idea each. Same word for the same thing.
- Edit with str_replace; never rewrite a whole file. Never re-read a file to check an edit.
- Blocked on a choice? Pick the obvious option, say so in one line. Ask only if truly stuck.
- Prose (docs, commits, UI copy): never use em-dashes.

## Commands
```
~/.kiro/lean-rails/bin/test spec/x_spec.rb:12   # rspec or minitest, compact; no args = full suite, once per session
~/.kiro/lean-rails/bin/lint [-a] [files]        # rubocop on changed files, one line per offense
rails g migration Name                          # never hand-write migrations
rails routes -g pattern
git status -sb | git diff --stat | git diff -U1 -- path | git log --oneline -n 10
```
A guard blocks verbose forms and names the compact one. `RAW=1 cmd` bypasses when full output is needed.

## Search, then read
- `rg -l pat` → `rg -n pat path` → read only the line range you need.
- Files: `rg --files | rg pat`. Logs: `tail -n 50 log/test.log`.
- Schema: `rg -A25 'create_table "users"' db/schema.rb`.

## Code style
- No service objects. Logic for one model: a method on it, or `Model::Feature` concern in
  `app/models/<model>/`. Bulky doer for one model: `app/models/<model>/x.rb`, agent noun
  (`Advert::Tagger`). Standalone concept, form object or external gateway: `app/models/x.rb`.
  Shared by unrelated models: `app/models/concerns/`, `-able` name. `lib/` only for gem-like code.
- Namespace by what a class knows, not who calls it. Never gerund doers (`GoogleIndexing`),
  never `Manager`/`Helper`/`Service`/`Util`.
- Jobs and mailers are thin: the model exposes `x_later`, which enqueues `XJob`, which calls `x_now`.
- Controllers thin and RESTful: a new resource (`resource :closure`) over a custom action.
  `create!`/`update!`, no `rescue_from`. `includes` to avoid N+1.
- Deep modules: split when the piece has a name, never for line count. No pass-through layers.
- Comments only for why, constraints, measured numbers, cross-system coupling.
- Migrations are DDL only; data changes are rake tasks run by hand. Never `db:seed` outside dev.
- Designing a new model, controller, job or migration? Read the rails-style skill first.

## Git & safety
- Lint passes before commit. Subject: plain imperative ("Add user avatar upload").
- Never push; pushes are manual. Never commit to main: branch first.
- No `rm -rf`: use `trash` or one named file. `rg` the repo before removing a method or constant.
