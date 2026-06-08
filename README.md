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
| `eval-provenance/` | `sdl` | — | A self-contained worked example of the eval evidence-provenance capability: one published eval (`silent-data-loss-v1`) in miniature, from atomic scored observations up to downstream routing guidance. Declares its own artifact-scheme enum so it verifies standalone. |

`agent-behavior:` and `paradigm:` are **compositionally coupled** — compounds in
one compose beliefs in the other (a real cross-namespace chain), so they
co-resolve as a pair rather than separating cleanly. `lib:` and `sdl:` are each
fully standalone.

`collections.json` is the local resolution map (namespace → path to that
collection's `beliefs.json`); the durable dependency declarations live in each
collection's `manifest.json`.

## Using

The query/verification tooling lives in the framework. With
[`composable-beliefs`](https://github.com/amieval/composable-beliefs) checked out
as a sibling (`../composable-beliefs`), from the framework root:

```sh
mix bs --beliefs ../belief-collections/agent-behavior/beliefs.json list   # query a collection
mix cb.verify.collection agent-behavior                                   # verify against declared deps
```

See [`quickstart.md`](quickstart.md) for the guided tour, and `library/` for the
worked `lib:` example.

## License

Apache-2.0 (see [LICENSE](LICENSE)).
