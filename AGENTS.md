# Project: YourApp

Rails X.Y, Ruby X.Y, PostgreSQL (or MySQL). Replace this file with your project's rules.
The agent reads this file at every session start. Keep it short and factual.

## Commands

```
bundle exec rspec <file:line>   # run one spec
bundle exec rspec               # full suite
bin/dev                         # start dev server (:3000)
bundle exec rubocop             # lint
```

- Generate migrations with `rails g migration`; never hand-write them.
- After a view or controller change, check the page in the browser.

## Before committing

- RuboCop must pass.
- Never push directly to main.
- Commit subject: plain imperative, e.g. "Add user avatar upload".

## Where code lives

No service objects. Logic goes on the model it belongs to.

| Kind of thing              | Where                   |
|----------------------------|-------------------------|
| Logic for one model        | `app/models/<model>.rb` |
| Shared across models       | `app/models/concerns/`  |
| Knows nothing about the app | `lib/`                 |
| Background work            | `app/jobs/`             |

- Controllers are thin and RESTful.
- Use `includes` to avoid N+1 queries.

## Data

- Migrations are DDL only. Data changes are rake tasks, run by hand.
- Never `db:seed` in staging or production.

## Safety

- No `rm -rf`. Use `trash` or name the file explicitly.
- Grep the whole repo before removing a method or constant.
