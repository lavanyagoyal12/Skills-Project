# Input formats and backends

FastJet clusters particle 4-momenta. What it accepts depends entirely on how you
build the input; the constructor `fastjet.ClusterSequence(data, jetdef)`
dispatches on the type of `data`.

## Coordinate systems

| Record fields | Requirement |
|---|---|
| `px, py, pz, E` | Works directly on a plain record array |
| `pt, eta, phi, M` (or `pt, y, phi, M`, `rho` for spatial) | Needs `vector` behavior: `import vector; vector.register_awkward()` and `with_name="Momentum4D"` (or `behavior=vector.backends.awkward.behavior`) |

Good practice (per `docs/Awkward.rst`): register vector behavior **before** using
fastjet. `px,py,pz,E` records also work when declared `Momentum4D`.

## Backend dispatch

| `data` type | Resulting class | Behavior |
|---|---|---|
| `ak.Array` | `fastjet._pyjet.AwkwardClusterSequence` | eager, vectorized over the first axis |
| `list` (of `PseudoJet`) | `fastjet._swig.ClusterSequence` | classic, one event only |
| `dask_awkward.Array` | `fastjet._pyjet.DaskAwkwardClusterSequence` | lazy; each partition clustered when computed |
| `graphed.Array` / `graphed.Varied` | `fastjet._graphed.GraphedClusterSequence` | deferred; queries recorded as graph nodes |
| anything else | `TypeError` | (a NumPy array is NOT accepted) |

The internal rep is chosen inside the Awkward arm by **layout shape**:

- `_jaggedness == 1` and layout is a record → `_singleevent` — a **flat** array is
  one event (`[{px,...}, ...]`), results drop the event axis.
- Layout is `[event][particle]` (or starts/stops index form) → `_multievent` —
  results are `2 * var * Momentum4D`.
- `_jaggedness >= 3`, or the top level is masked/indexed/union/record →
  `_generalevent` — clustering happens per "clusterable level", and output keeps
  the nesting shape you gave it (missing/masked entries stay `None`).

Supported layouts (verified by `tests/test_007-general.py`): `ListOffsetArray`,
`ListArray`, `RegularArray`, `ByteMaskedArray`, `IndexedArray`,
`IndexedOptionArray`, `UnmaskedArray`, `UnionArray`, nested records, and
`ListOffset` subtrees whose starts need not begin at 0 and may be out of order
(`tests/test_009-list_starts.py`: `array[1:]`, `array[[2,0,3,1]]` work correctly).

Not supported: `VirtualArray` and `PartitionedArray` inputs (the tests named
`*_NOT_REALLY` document this).

## Output conventions

- Array-arm outputs are named `Momentum4D` records (`px, py, pz, E`) and carry
  the input `behavior` and `attrs` through.
- Jet/constituent order follows FastJet's own ordering (roughly descending
  `pt`); do not assume input order.
- `constituents()` and `constituent_index()` round-trip through indices, so
  extra fields on the input records are preserved on constituents.
- Scalar-ish methods (`Q`, `exclusive_dmerge`, `n_particles`, ...) return a plain
  Python float/int for single-event input and an `ak.Array` (per event) for
  multi-event input.

## Dask-awkward

```python
import dask_awkward as dak
lazy = dak.from_awkward(events, npartitions=4)
cluster = fastjet.ClusterSequence(lazy, jetdef)   # DaskAwkwardClusterSequence
jets = cluster.inclusive_jets().compute()          # -> ak.Array
```

- `data` must be a `dak.Array`; the cluster sequence runs lazily per partition
  via `map_partitions`.
- Methods that take a second array (`exclusive_subjets`, `get_parents`, ...)
  check `dak.lib.core.compatible_partitions` — the arguments must be
  partition-wise compatible with the clustering input or `ValueError`.
- Metadata is computed by running the eager arm on length-zero arrays of the
  input form; the record must expose `px,py,pz,E` even in a typetracer context.
- Reading ROOT: `uproot.dask({file: "Events"})` → pass fields as an awkward
  `Momentum4D` record (see `tests/test_008-dask.py::test_inclusive_from_file`).
- `dask` / `dask-awkward` are optional (`pip install fastjet[dask]`); version
  floors are `dask>=2023.4.0`, `dask-awkward>=2023.4.2`.

## Graphed (deferred)

```python
import graphed, graphed.awkward as ga
session = graphed.Session(ga.AwkwardBackend(behavior=vector.backends.awkward.behavior))
events = ga.from_awkward(session, "particles", events_ak)
cluster = fastjet.ClusterSequence(events, jetdef)     # GraphedClusterSequence
jets = cluster.inclusive_jets()                       # just records a node
session.materialize(jets)                             # runs the graph
```

- Every query records one node; clustering runs at evaluation. `graphed>=0.0.3`
  is optional (`pip install fastjet[graphed]`).
- The **first axis is the partition axis**, so input must hold one list of
  particles per event (`ndim > 1`). A flat collection (one single event) is
  refused with `TypeError` — cluster that eagerly instead.
- For `graphed.Varied` inputs every universe is checked, because partitioning
  may tear apart a flat universe (`tests/test_011`).
- Query arguments must also be graphed arrays, from the **same session**;
  another session raises `graphed.GraphedTypeError` and a plain awkward array
  raises `TypeError`.
- A `Varied` input clusters once per universe; over a partitioned source the
  queries become outputs of `graphed.aggregate_plan`.

## Classic input

`fastjet.PseudoJet(px, py, pz, E)` — constructor order is `(px, py, pz, E)`, not
`(pt, eta, phi, m)`. One `ClusterSequence` call = one event.
