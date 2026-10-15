# awkward backend

The common HEP case: jagged/variable-length collections of particles.
`vector.zip(...)` and behavior-registered `ak.Array` records gain `.deltaR()`,
`.mass`, `.boost()`, `.px`, etc.

## Registration

Call **before** any vector method use on `ak.Array`, or `.deltaR()` etc. raises
`AttributeError`:

```python
import vector
vector.register_awkward()  # once, at module level
```

Alternative — install behaviors on a single array without mutating global
`ak.behavior`:

```python
import awkward as ak
arr = ak.Array([...], with_name="Momentum4D",
               behavior=vector.backends.awkward.behavior)
```

## Core patterns

```python
import awkward as ak
import vector

vector.register_awkward()

# Momentum4D records from NTuple columns; vector.zip infers the record type
# (ak.zip would need with_name="Momentum4D", otherwise records are plain dicts)
jets = vector.zip({
    "pt": events["jet_pt"], "phi": events["jet_phi"],
    "eta": events["jet_eta"], "mass": events["jet_m"],
})

# Invariant mass of all jet pairs (MeV → GeV)
j1, j2 = ak.unzip(ak.combinations(jets, 2, axis=1))
mjj = (j1 + j2).mass / 1000

# deltaR for overlap removal
pairs = ak.cartesian({"e": electrons, "j": jets}, axis=1)
jets = jets[~ak.any(pairs.e.deltaR(pairs.j) < 0.4, axis=1)]

# Boost (parent must be a vector object)
boosted = daughter.boost(-parent.to_beta3())
```

## Numba

`@nb.njit` compiles `vector.obj` and awkward `vector.Array` (Momentum4D
records) — best for custom loops over large ragged arrays where awkward
vectorization cannot express the logic:

```python
import numba as nb
import numpy as np

@nb.njit
def sum_mass(array):  # array: awkward vector.Array of Momentum4D records
    out = np.empty(len(array), np.float64)
    for i, event in enumerate(array):
        total = vector.obj(px=0.0, py=0.0, pz=0.0, E=0.0)
        for vec in event:
            total = total + vec
        out[i] = total.mass
    return out
```

JIT has a cold-start cost but can be much faster on large arrays; speedups
depend on workload, hardware, and versions.

## Interop

- **uproot**: fields read from ROOT files usually need renaming to match vector
  conventions.
- **hist**: compute quantities with vector, then flatten and fill a `Hist`.
