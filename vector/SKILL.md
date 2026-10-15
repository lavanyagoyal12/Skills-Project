---
name: vector
description: >-
  Use when computing 4-vector quantities in Python: invariant mass, deltaR,
  transverse momentum, boost, or any Lorentz vector arithmetic over collections
  of particles. Also use when registering scikit-hep vector behaviors on
  awkward-array records so that ak.zip objects gain Momentum4D methods, or when
  constructing vector objects from (pt, phi, eta, mass) or (px, py, pz, energy)
  field conventions.
---

# Vector

`vector` adds Lorentz-vector math (`.deltaR()`, `.mass`, `.boost()`, `.pt`, ...)
to awkward arrays, NumPy arrays, and scalars: register a backend's behaviors
once, then use the physics methods directly instead of manual
`px = pt * cos(phi)` math.

## Backends

| Backend | Read | Use for |
| --- | --- | --- |
| awkward | `references/awkward.md` | jagged/ragged record collections (`vector.zip`, `ak.Array`) — the common HEP case |
| numpy | `references/numpy.md` | fixed-shape arrays (`vector.array`) and single scalars (`vector.obj`) |
| sympy | `references/sympy.md` | symbolic derivation and code generation |
| pytree | `references/pytree.md` | flatten/unflatten vector state for `scipy`/optimizers (needs `optree`) |

## Conventions (all backends)

- Field names are auto-detected: `pt/phi/eta/mass` or `px/py/pz/energy` — never
  mix (conflicting sets raise).
- Energy synonyms: `t`/`e`/`E`/`energy`; mass: `tau`/`m`/`M`/`mass`.
- Any coordinate (`.pt`, `.px`, ...) is readable regardless of how the vector
  was built; `.to_*()` (e.g. `.to_rhophithetatau()`) changes storage to avoid
  repeated trig.
- Units are your responsibility — vector never converts units (MeV stays MeV).

## Shared gotchas

- `vector.obj` is one scalar, not an array; for collections use
  `vector.array` / `vector.zip`.
- `v1 + v2` returns a new `Momentum4D`; take `.mass` of the sum for invariant
  mass.
- Backend-specific details `register_awkward()`, Numba support, `optree`, and
  SymPy sign conventions are in each backend's reference file.

## Docs

llms.txt available and `.md` extensions supported:

https://vector.readthedocs.io/en/latest/
