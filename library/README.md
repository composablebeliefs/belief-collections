# Library Circulation - a worked example

A small, self-contained belief graph in a familiar domain: a lending library. Use
it to learn the mechanics of composable beliefs - primitive, compound, implication,
composition, supersession, staleness, and materialization - before reading the main
graph, which is self-referential (the framework reasoning about its own design).

Everything here lives in its own `lib:` namespace and never touches the main `cb:`
graph.

## Point the shell at it

This collection lives in `belief-collections`; the `mix` tasks live in the
framework. Run them from the framework root (`composable-beliefs/`),
pointing at this collection:

    mix bs --beliefs ../belief-collections/library/beliefs.json stats

or, to avoid repeating the flag, export the env var the rest of this walkthrough
assumes:

    export CB_BELIEFS=../belief-collections/library/beliefs.json
    mix bs stats

You should see 14 beliefs - 6 primitives, 2 compounds, 6 implications (4 of them
contracts) - with one superseded node and one stale belief.

## 1. Primitives - what a source said

Primitives are atomic, source-grounded facts. Each cites an `artifact` and may carry
dated `evidence`. They have no `deps` - they stand on their source, not on other
beliefs.

    mix bs show lib:a001

`lib:a001` says the loan period is 21 days, grounded in the 2026 circulation policy.

## 2. Compounds - what combining sources means

A compound is an emergent conclusion that none of its inputs states on its own.

    mix bs show lib:a010
    mix bs tree lib:a010

`lib:a010` observes that you cannot predict when a held item becomes collectable from
the loan period alone - it depends jointly on the loan clock (`lib:a001`) and the
hold-expiry clock (`lib:a005`). The `tree` view shows the conclusion standing on its
two primitives.

## 3. Implications - what must happen

An implication says what to do, or what must hold.

    mix bs show lib:a020

`lib:a020` turns the hold-expiry rule into an action: when a hold expires, transition
the item and notify the next member. It depends on the rule (`lib:a005`) and on the
lifecycle contract (`lib:c001`).

## 4. Contracts - implications with teeth

A contract-grade implication carries enforceable `rules` and `invariants` and is
marked `contract: true`. By convention contracts get a `c`-prefix id; everything else
is `a`-prefix.

    mix bs show lib:c001

`lib:c001` is a state machine for an item's lifecycle (available, checked_out,
on_hold, lost, withdrawn). Its `rules` are transition edges (`from -> to`, with
per-edge requirements); its `invariants` are properties that always hold. This is the
same shape as the main graph's own `cb:c029` status-lifecycle contract - read both
side by side.

## 5. Supersession - immutability with a paper trail

Beliefs are never edited in place; a changed fact is a new belief that supersedes the
old one.

    mix bs history lib:a001

The loan period was 14 days (`lib:a002`) before it became 21 (`lib:a001`). The old
belief is still in the graph, marked `superseded` and linked forward - so the history
is auditable.

## 6. Staleness - what a change invalidates

Because a superseded belief is still referenced, anything that depended on it is now
suspect.

    mix bs stale

`lib:a011` (the overdue-fine schedule, calibrated to the old 14-day baseline) is
flagged stale: it depends on the superseded `lib:a002`. The graph tells you the fine
schedule needs revisiting - it does not pretend the change had no consequences.

## 7. Materialization - from belief to action

An implication can be materialized into concrete work items, with the link recorded
back on the belief.

    mix bs show lib:a021

`lib:a021` (recompute due dates when the loan period changes) is already materialized:
its `materialized` field records the date and the item it produced (see
`todos.json` in this directory). Contrast `lib:a020`, whose `materialized` is still
null. The `/materialize` skill turns an unmaterialized implication into items; point
its sink at this directory (a `todos_path` override) to try it without touching the
main graph.

## 8. The schema is beliefs too

The kinds, domains, and artifact schemes this collection uses are not hard-coded
anywhere - they are declared by beliefs *in* the collection:

    mix bs list contracts

`lib:c002`, `lib:c003`, and `lib:c004` are enum-registry contracts that close the
`kind`, `domain`, and `artifact-scheme` vocabularies. The verifier checks the
collection against them:

    mix cb.verify.schema --beliefs ../belief-collections/library/beliefs.json

Every check passes, and none is hard-coded to this collection: the verifier
*discovers* each enum by the field it declares. (The belief status lifecycle -
active/superseded/retracted/retired - is framework-universal, so the collection
inherits it rather than restating it; status is checked against the framework
canon.) This is the self-describing pattern in miniature: the collection carries
its own schema, and the same tool that checks the main `cb:` graph checks this
one. Because it is self-contained, it also verifies through the collection-aware
task that resolves declared dependencies - `mix cb.verify.collection lib` loads
only `lib:`, since its manifest lists no `depends_on`.

## Now read the real graph

    unset CB_BELIEFS
    mix bs stats
    mix bs tree cb:a138

Same mechanics, turned on the framework itself: the main graph is composable beliefs
reasoning about composable beliefs. This example was the gentle version; the real one
is the thesis.

## What's here

| id | type | role |
|---|---|---|
| `lib:a001` | primitive | loan period is 21 days (current) |
| `lib:a002` | primitive | loan period is 14 days (superseded by a001) |
| `lib:a003` | primitive | returned item with a hold goes on_hold |
| `lib:a004` | primitive | member hold limit |
| `lib:a005` | primitive | hold expiry passes item to next in queue |
| `lib:a006` | primitive | 30-days-overdue item is marked lost |
| `lib:a010` | compound | collectable date depends on two clocks |
| `lib:a011` | compound | fine schedule assumes 14-day baseline (stale) |
| `lib:a020` | implication | act on hold expiry (unmaterialized) |
| `lib:a021` | implication | recompute due dates on loan-period change (materialized) |
| `lib:c001` | contract | item-status state machine |
| `lib:c002` | contract | kind enum (this collection's vocabulary) |
| `lib:c003` | contract | domain enum |
| `lib:c004` | contract | artifact-scheme enum |

See `SCHEMA.md` for the conventions this collection follows.
