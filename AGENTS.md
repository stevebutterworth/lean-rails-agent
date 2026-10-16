# Project: YourApp

Rails X.Y, Ruby X.Y, PostgreSQL. Replace with your project's rules.
Sent on every turn: keep it short.

## Output
- No preamble, no restating the task, no closing summary.
- Answers: 3 lines max unless asked. After code changes: one line per file touched.
- Edit with str_replace; never rewrite a whole file. Never re-read a file to check an edit.
- Blocked on a choice? Pick the obvious option, say so in one line. Ask only if truly stuck.

## Commands
```
bin/t spec/x_spec.rb:12          # tests, compact output; bin/t alone = full suite, once per session
bin/lint [-a] [files]            # rubocop on changed files, one line per offense
rails g migration Name           # never hand-write migrations
rails routes -g pattern
git status -sb | git diff --stat | git diff -U1 -- path | git log --oneline -n 10
```
Guard blocks verbose forms and names the compact one. `RAW=1 cmd` bypasses when full output is needed.

## Search, then read
- `rg -l pat` → `rg -n pat path` → read only the line range you need.
- Files: `rg --files | rg pat`. Logs: `tail -n 50 log/test.log`.
- Schema: `rg -A25 'create_table "users"' db/schema.rb`.

## Code
- No service objects. One model's logic: `app/models/<model>.rb`. Shared: `app/models/concerns/`.
  Background: `app/jobs/`. App-agnostic only: `lib/`.
- Thin RESTful controllers. `includes` to avoid N+1.
- Migrations are DDL only; data changes are rake tasks run by hand. Never `db:seed` outside dev.

## Git & safety
- RuboCop passes before commit. Subject: plain imperative ("Add user avatar upload").
- Never push to main; pushes are manual.
- No `rm -rf`: use `trash` or one named file. `rg` the repo before removing a method or constant.
