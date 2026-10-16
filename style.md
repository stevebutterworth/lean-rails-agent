---
name: rails-style
description: Read before designing a new model, concern, PORO, controller, job or migration, or when naming a new class. Not needed for bug fixes, renames, small edits or git work.
---

# Rails style (vanilla Rails, 37signals)

Extends the Code style section of rules.md. Where to put code is there; this covers shape.

## Naming
- Records, values, proxies: plain nouns. Doers: agent nouns (`Notifier`, `Indexer`). Concerns: adjectives (`Closeable`).
- A class proxying a remote thing is named for the thing: `GoogleIndex`, so `google_index.delete(url)` reads as English.
- Predicates read as English (`delinquent?`). Domain actions are verbs (`close`, `publish`, `move_to`).
- `!` only when a non-bang twin exists. Constants carry units: `TIMEOUT = 7.seconds`.

## Models
- The model file holds identity: associations, core scopes, a few core operations.
- Each facet is a concern: `included do` for associations, scopes and callbacks, then one to six public methods. Never a concern of only private methods.
- State as records, not booleans: `has_one :closure` over `closed: boolean`, so you get who and when for free.
- Constant per-subclass facts: `class_attribute :flag, default: false, instance_writer: false`.
- Callbacks are fine for indexing, broadcasts and denormalisation. Validations sparse.
- `Current.user` as the default actor: `belongs_to :creator, default: -> { Current.user }`.
- `delegate` one hop to an association, no `prefix:`. `transaction do` inside model methods for multi-write invariants.
- Errors are one-line classes beside their owner. Rescue specific classes only.

## Controllers and jobs
- Only the seven REST actions. A state change is a new singular resource: `resource :closure` gives `create`/`destroy` for close/reopen.
- Scope lookups through the current user or account. `before_action :set_x`.
- Jobs hold no logic: `def perform(card) = card.deliver_now`.

## Migrations
- Index every foreign key and every column you filter or sort on. Unique index backs any uniqueness validation.
- `null: false` and defaults in the database, not only in the model.

## Ruby
- Expanded `if/else` over guard clauses, except one `return if` at the top of a longer method.
- Order: class methods, then public (`initialize` first), then private, each in call order so the file reads top-down.
- No blank line under `private`; private methods indented one extra level.
- `%i[ show edit ]`, `[ :name ]` spacing. Keyword shorthand at call sites. `it` block param.

## Tests
- Fixtures over factories where the app has them. Test behaviour through the model's public methods.
- Stub only at the boundary with external systems; never stub the class under test.
