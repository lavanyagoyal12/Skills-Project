---
name: fastjet
description: Cluster HEP particles into jets with FastJet from Python. Use for jet finding, jet substructure, and event analysis with awkward/dask/graphed arrays; choose the right interface, build JetDefinitions, run inclusive and exclusive jet queries, and avoid the input-format and API pitfalls.
---

# FastJet (Python bindings)

## Overview

`fastjet` is the Scikit-HEP Python binding to the C++ FastJet jet-finding library
(FastJet 3.5.2 + fjcontrib 1.105). It clusters particle 4-momenta into jets with
anti-kT, kT, Cambridge/Aachen, generalized-kT, and e+e− algorithms, and exposes
FastJet-contrib tools (SoftDrop, N-jettiness, energy correlators, Lund
declustering). One package, four backends:

- **Awkward** — eager, vectorized over many events: `ak.Array` in, `ak.Array` out.
- **Classic** — one event, C++-style: `list[PseudoJet]` in, `PseudoJet` list out;
  exposes nearly the entire C++ surface (areas, selectors, taggers).
- **Dask** — lazy, out-of-core: `dask_awkward.Array` in, clustered per partition.
- **Graphed** — deferred: queries recorded as graph nodes, evaluated later.

The C++ library does the actual clustering; the Python layer handles dispatch,
array conversion, and result reconstruction.

## When to use this library

Use it when a task involves: jet finding or jet clustering (anti-kT/kT/C-A/genkt/
ee algorithms), jet substructure or grooming, HEP event analysis over Awkward
Arrays or ROOT files (e.g., `uproot` → dask-awkward), or porting C++ FastJet code
to Python.

Do **not** use it for: generic 4-vector math (use `vector`/`awkward` directly),
algorithms not in FastJet (plugins exist, but are build-configured), or when a
single scalar/array transform would suffice. Also, `fastjet` here is the Python
package — questions about C++ algorithm internals belong to http://fastjet.fr.

## Key concepts

- **`JetDefinition(algorithm, R, ...)`** — algorithm constant + radius (plus
  optional recombination scheme, strategy, extra params). Always pass the radius.
- **`ClusterSequence(data, jetdef)`** — one constructor, dispatches on `data`:
  `ak.Array` → Awkward arm, `list` → classic, `dak.Array` → Dask, graphed → deferred.
- **Algorithms**: `kt_algorithm`, `antikt_algorithm`, `cambridge_algorithm`,
  `genkt_algorithm` (`p`: 1=kT, 0=C/A, −1=anti-kT), `ee_kt_algorithm`,
  `ee_genkt_algorithm`, `plugin_algorithm`.
- **Inclusive vs exclusive jets**: inclusive = all jets above `min_pt`
  (recommended for hadron colliders); exclusive = exactly `n_jets` jets or jets
  merged up to `dcut` (recommended for e+e−).
- **Input records**: `px, py, pz, E` directly, or `Momentum4D` from `vector`
  (`vector.register_awkward()` + `with_name="Momentum4D"`).
- **Queries return arrays of the same shape semantics** as the input: flat
  input → flat output (one event); `[event][particle]` → `2 * var * Momentum4D`;
  deeper/masked inputs keep their nesting (general arm).

## Common usage patterns

```python
import awkward as ak
import fastjet
import vector; vector.register_awkward()   # only for pt/eta/phi/M input

jetdef = fastjet.JetDefinition(fastjet.antikt_algorithm, 0.6)
events = ak.Array([[{"px": 1.2, "py": 3.2, "pz": 5.4, "E": 23.5}, ...], ...])
cluster = fastjet.ClusterSequence(events, jetdef)

# Core queries
jets = cluster.inclusive_jets()                # or inclusive_jets(min_pt=5.0)
parts = cluster.constituents()                 # preserves extra record fields
idx = cluster.constituent_index()

# Exclusive selection — exactly one of n_jets / dcut
cluster.exclusive_jets(n_jets=3)
cluster.exclusive_jets(dcut=0.0001)
cluster.exclusive_jets_up_to(n_jets=4)         # never raises for too many

# Substructure / observables (fastjet-contrib)
cluster.exclusive_jets_softdrop_grooming(njets=1)
cluster.njettiness(njets=[1, 2, 3, 4], beta=1.0, R0=0.8)
cluster.exclusive_jets_energy_correlator(njets=1, beta=1, npoint=2)
cluster.exclusive_jets_lund_declusterings(njets=10)

# Classic interface (one event)
fastjet.ClusterSequence([fastjet.PseudoJet(px, py, pz, E), ...], jetdef)
```

Other backends — see `references/examples.md`:
`fastjet.ClusterSequence(dak.from_awkward(events, npartitions=4), jetdef).inclusive_jets().compute()`
for Dask; wrap a `graphed.Session` array for deferred queries.

## Important gotchas

1. **Exclusive jets**: `exclusive_jets(n_jets=-1, dcut=-1)` — set *exactly one*;
   `0` raises `ValueError`; `n_jets` > particles raises `RuntimeError` (use
   `exclusive_jets_up_to`). Non-kT/C-A algorithms get a "interpret with care"
   warning for `dcut`.
2. **Fields**: records must be `px,py,pz,E` or registered `Momentum4D`.
   Missing behavior → `AttributeError`/`TypeError`, not silent wrong results.
3. **Type dispatch**: only `ak.Array`, `list`, `dak.Array`, graphed arrays are
   accepted (NumPy arrays are not); a `list` always means the classic interface;
   only `fastjet.JetDefinition` (the public, picklable one) is valid everywhere.
4. **Silent dispatch failure**: an `ak.Array` layout matching no branch leaves
   `cluster._internalrep` unset with no error — check it when queries misbehave.
5. **Parenting queries** (`get_parents`, `get_child`, `has_parents`, `has_child`)
   match jets to inclusive jets **by rapidity**; pass only jets from the same
   `ClusterSequence` or you get `RuntimeError: Jet Not in this ClusterSequence`.
6. **Graphed**: input must hold one list of particles per event (flat = refused);
   for `Varied`, every universe is checked; operands must come from the same
   session. Optional dependency (`fastjet[graphed]`).
7. **Dask**: methods taking a second jet array require partition-wise
   compatibility with the clustered input.
8. **Known bugs**: `fastjet.sort_indices` recurses into itself; use awkward's
   `ak.argsort`; `sorted_by_rapidity` delegates to `sorted_by_pz` for SWIG input;
   `join` caps at 4 args; `from fastjet import *` only imports `__version__`.
9. **Environment**: Python ≥ 3.11; run the canonical example before bulk work —
   expected values are pinned in `tests/test_001-basic_clustering.py`; the C++
   FastJet banner printed at construction is normal output.

## Reference files

- `references/api-reference.md` — algorithms, `JetDefinition`, every
  `ClusterSequence` query method, classic-interface classes, helpers.
- `references/input-formats.md` — coordinate systems, Awkward layout dispatch,
  Dask/graphed input rules, output shapes.
- `references/gotchas.md` — full error catalog and sharp edges.
- `references/examples.md` — copy-paste recipes with test-pinned values.
