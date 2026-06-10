# belief-collections

Belief collections for the [Composable Beliefs](https://github.com/amieval/composable-beliefs)
framework. Each collection is a namespaced belief graph with a `manifest.json`
declaring its namespace and cross-namespace `depends_on`. Collections may depend
on the framework's own `cb:` graph (schema + mechanism); `cb:` depends on nothing.

## Collections

| Dir | Namespace | Depends on | What it is |
|---|---|---|---|
| `agent-behavior/` | `agent-behavior` | `cb`, `paradigm` | Reasoning errors, interaction patterns, and corrective rules observed in operation. |
| `paradigm/` | `paradigm` | `cb`, `agent-behavior` | Meta-observations on how the human-agent system works and should work. |
| `library/` | `lib` | — | A self-contained worked example (a lending library) with its own enum contracts. The gentle on-ramp before the self-referential `cb:` graph. |
| `method/` | `method` | — | **The base collection every published eval depends on**: the shared eval vocabulary (artifact-scheme/kind/domain enums; the six-subject, staleness-pivot, tag, judge-naming, and fixture-provenance conventions) plus methodology contracts v1 — corroboration, provenance, subjects, minimum runs, judge validation, correction protocol — each routed to a machine-checked predicate. Self-contained. |
| `eval-provenance/` | `sdl` | `method` | A worked example of the eval evidence-provenance capability: one published eval (`silent-data-loss-v1`) in miniature, from atomic scored observations up to downstream routing guidance. Borrows its vocabulary from `method:` (its original local enum is superseded by the shared one — the worked demonstration of borrowing) and **deliberately fails two methodology contracts** as the teaching/failing fixture; see its README. |
| `toy-eval/` | `toy` | `method` | The compliant counterpart to `sdl`: three runs, two rulers (one an LLM judge *with* a validation record), a cross-ruler-agreement compound, and a corroborated verdict. Verifies green — including all six methodology checks — with zero local schema contracts. Entirely synthetic, so every belief carries the `fixture` tag. |

`agent-behavior:` and `paradigm:` are **compositionally coupled** — compounds in
one compose beliefs in the other (a real cross-namespace chain), so they
co-resolve as a pair rather than separating cleanly. `lib:` and `method:` are
fully standalone; `sdl:` and `toy:` resolve against `method:`.

`collections.json` is the local resolution map (namespace → path to that
collection's `beliefs.json`); the durable dependency declarations live in each
collection's `manifest.json`.

## The eval family: method, sdl, toy

Three of the collections form one lesson about publishing eval findings. The idea:
house methodology usually lives in prose nobody can mechanically check. Here it is
graph structure — `method:` carries six contract-grade beliefs (minimum three runs
per verdict, cross-ruler corroboration, raw-log provenance, the six-subject
convention, LLM-judge validation, corrections-as-supersessions), and
`mix cb.verify.collection` executes them as named predicates over any collection
that borrows the vocabulary. "Methodology v2" is not a doc edit; it is a batch of
adjudicated supersessions of these contracts, dated and diffable with `bs history`.

`sdl` and `toy` are the two sides of that coin, on purpose:

- **`sdl` fails visibly.** Its verdict cites one run (minimum: three) and its LLM
  judge has no validation record, so two methodology checks fail with the offending
  belief ids named. A teaching collection that visibly fails the house methodology
  teaches both the mechanism and the culture.
- **`toy` passes everything** with zero local schema contracts — proof that a new
  eval collection authored against `method:` inherits its entire vocabulary and
  discipline by declaring one `depends_on`.

One convention worth knowing even outside evals: beliefs derived from synthetic,
mock, or test data carry the `fixture` tag (`method:a5`), so test scaffolding can
never be mistaken for a real finding.

Observation beliefs in eval collections are not hand-authored: a harness run is
converted to a neutral *run-manifest* and imported deterministically by
`mix cb.import.eval` (see `composable-beliefs/docs/run-manifest.md`). The
execution side — Inspect tasks, judges, the adapter — lives in the sibling
`bench` repo; executed-eval records live in the sibling `evals` repo.

## Using

The query/verification tooling lives in the framework. With
[`composable-beliefs`](https://github.com/amieval/composable-beliefs) checked out
as a sibling (`../composable-beliefs`), from the framework root:

```sh
mix bs --beliefs ../belief-collections/agent-behavior/beliefs.json list   # query a collection
mix cb.verify.collection agent-behavior                                   # verify against declared deps
mix cb.verify.collection toy   # the compliant eval fixture: all six methodology checks pass
mix cb.verify.collection sdl   # the failing fixture: m-runs and m-judge-validation fail, ids named
```

See [`quickstart.md`](quickstart.md) for the guided tour, and `library/` for the
worked `lib:` example.

## License

Apache-2.0 (see [LICENSE](LICENSE)).
