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

The archived `pyjet` PyPI package (fjcore over NumPy structured arrays) is a
separate, deprecated project — it is **not** an older name for `fastjet`; use
this package for new work. (The `fastjet._pyjet` module in this package is an
internal implementation detail, unrelated to `pyjet`.)

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
jets = cluster.inclusive_jets()                # or inclusive_jets(min_pt=5.0)
```

Exclusive selection — set exactly one of `n_jets` / `dcut`:

```python
cluster.exclusive_jets(n_jets=3)        # exactly 3 jets, or exclusive_jets(dcut=0.0001)
cluster.exclusive_jets_up_to(n_jets=4)  # never raises for too many jets
```

Classic interface, one event, one line:

```python
fastjet.ClusterSequence([fastjet.PseudoJet(px, py, pz, E), ...], jetdef).inclusive_jets()
```

**Performance:** pass the `ak.Array` directly to `ClusterSequence`. Building
`PseudoJet` objects one-by-one in a Python loop crosses the interpreter on every
particle and event; the columnar path pushes the conversion into C++.

**Before writing clustering code, read `references/examples.md`** — copy-paste
recipes for substructure observables (SoftDrop, N-jettiness, energy correlators,
Lund), the Dask-awkward (out-of-core/ROOT) backend, graphed (deferred)
clustering, and the classic interface, all pinned to the test suite.

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
10. **Units**: FastJet has no unit system — keep all four-vectors in the same
    units (GeV by HEP convention). Don't mix GeV and MeV inputs.
11. **`phi()` range**: FastJet returns `phi` in `[0, 2π)`. Code expecting
    `(−π, π]` should use `PseudoJet.phi_std()` (classic interface) or normalize
    the array via `vector`/`awkward` when using the columnar interface.
12. **Cutoff kwarg name differs by interface**: the columnar/Dask/graphed arms
    take `inclusive_jets(min_pt=...)`; the classic SWIG bindings follow the C++
    name `inclusive_jets(ptmin=...)`. A positional argument works in both.

## Interop

- **pyhepmc / pylhe**: read HepMC3 or LHE events, extract final-state particles
  into an `ak.Array` with `px, py, pz, E` fields in a single pass, and feed it
  directly to `ClusterSequence` (columnar interface). There's no per-event
  `PseudoJet` loop needed.
- **vector / awkward**: `vector.register_awkward()` + `with_name="Momentum4D"`
  converts `(pt, eta, phi, M)` records for the columnar interface. The classic
  interface always needs `PseudoJet(px, py, pz, E)` objects.

## Reference files

- `references/api-reference.md` — algorithms, `JetDefinition`, every
  `ClusterSequence` query method, classic-interface classes, helpers.
- `references/input-formats.md` — coordinate systems, Awkward layout dispatch,
  Dask/graphed input rules, output shapes.
- `references/gotchas.md` — full error catalog and sharp edges.
- `references/examples.md` — copy-paste recipes with test-pinned values.
