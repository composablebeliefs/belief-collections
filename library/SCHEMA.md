# Library example - schema conventions

A documented schema for the `lib:` library collection (in `belief-collections`). It
follows the same structural discipline as the main `cb:` graph (see the framework's
`docs/belief-graph.md` and `lib/cb/belief.ex`), scoped to what the example needs.

`mix cb.verify.schema --beliefs ../belief-collections/library/beliefs.json`
verifies this collection against its *own* schema contracts (below): the verifier
discovers each enum by the field it declares, so the same tool that checks the main
`cb:` graph checks this one. `mix cb.verify.collection lib` does the same through the
collection-aware path (it has no declared dependencies, so it loads only `lib:`).

## Namespace and ids

- All ids are namespaced `lib:`.
- `a`-prefix = non-contract (primitives, compounds, plain implications).
- `c`-prefix = contract-grade implications only (`contract: true` with non-empty
  `rules`/`invariants`). This mirrors the `cb:` convention.

## Types

`primitive` | `compound` | `implication` - the three structural types from
`CB.Belief`. A contract is an implication with `contract: true`.

## Fields used

- `kind`: `rule` | `observation` | `state-machine` | `enum-registry` (declared by `lib:c002`).
- `domain`: `library` (declared by `lib:c003`).
- `artifact` (primitives only): `document:<file>` (scheme declared by `lib:c004`).
- `evidence[]`: `{date, detail, artifact}` entries.
- `subjects[]`: `{ref, type}`, e.g. `{ref: "policy/loan-period", type: "policy"}`.
- `deps[]`: upstream belief ids; compounds and implications only (primitives have none; contracts may have none).
- `rules` / `invariants` (contracts): a `state-machine` contract's `rules` are
  transition edges `{from, to, requires}`; an `enum-registry` contract's `rules`
  are `{field, values}` entries; `invariants` are prose properties.
- `materialized`: `null`, or `{date, todos: [{action, id}]}` once acted on.
- `status`: `active` | `superseded`; a superseded node carries `superseded_by`.

## Self-describing schema

The collection declares its own vocabulary as enum-registry contracts:

- `lib:c002` - the `kind` enum (rule, observation, state-machine, enum-registry)
- `lib:c003` - the `domain` enum (library)
- `lib:c004` - the `artifact-scheme` enum (document)

The belief status lifecycle (active/superseded/retracted/retired) is
framework-universal, not collection-specific, so the collection does not restate
it; the verifier checks status against the framework canon (`CB.Belief.statuses/0`).
That split - the framework owns structure, the collection owns vocabulary - is the
general rule for any host graph.

## Shape invariants

- 14 beliefs: 6 primitive, 2 compound, 6 implication (4 of them contracts).
- Exactly one superseded node (`lib:a002`) and one stale node (`lib:a011`).
- Every `deps` entry resolves to a node in the collection.
- `mix cb.verify.collection lib` (or `mix cb.verify.schema --beliefs ../belief-collections/library/beliefs.json`) passes with no failures and no skips.
