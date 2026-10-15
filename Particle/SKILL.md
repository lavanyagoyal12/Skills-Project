---
name: particle
description: >-
  Use when looking up particle properties (mass, charge, width, lifetime,
  spin, parity, PDG ID) from the PDG tables in Python: converting between
  particle names and PDG IDs, filtering particles by quantum numbers or
  stability, identifying Monte Carlo truth particles, converting generator
  codes (Geant3, Pythia, Corsika7, EvtGen), or using Particle/PDGID in
  analysis code.
---

# particle

`particle` wraps the PDG table in Python: names, PDG IDs, masses, widths,
charges, lifetimes, and quantum numbers. Standard Scikit-HEP tool for
particle identification. Verified against `particle` 1.0.1.

| Class | Use for |
|---|---|
| `PDGID` | fast integer PDG ID checks (no full table load) |
| `Particle` | full entry: mass, width, quantum numbers, names |

## When to Use

- Identifying particles in Monte Carlo truth records by PDG ID
- Looking up masses or widths for four-vector construction or selection cuts
- Converting between generator-specific IDs (Geant3, Pythia, Corsika7, EvtGen) and PDG IDs
- Checking antiparticle relations, stability, and quantum numbers

## Lookups

| Method | Example | Result |
|---|---|---|
| `from_pdgid(id)` | `Particle.from_pdgid(211)` | `Particle` object (name `pi+`, pdgid 211) |
| `from_name(name)` | `Particle.from_name("K+")` | `Particle` object (pdgid 321) |
| `from_evtgen_name(n)` | `Particle.from_evtgen_name("J/psi")` | `Particle` object (name `J/psi(1S)`, pdgid 443) |

```python
from particle import Particle
p = Particle.from_pdgid(211)   # pi+
print(p.mass)                  # 139.57039  (MeV)
print(p.mass / 1000)           # ~0.13957    (GeV)
```

## Attributes

| Attribute | Unit | Notes |
|---|---|---|
| `p.name` | — | `"pi+"` |
| `p.pdgid` | — | `PDGID` object — see PDGID properties below |
| `p.mass` | MeV | ÷1000 → GeV |
| `p.width` | MeV | `0.0` for stable; `None` if unknown (e.g. quarks) |
| `p.lifetime` | ns | `inf` for stable; `None` if unknown |
| `p.ctau` | mm | c·τ |
| `p.charge` | e | `Fraction` in particle 1.0.1 (prints `1`, not `1.0`) |
| `p.is_self_conjugate` | bool | own antiparticle (π⁰) |
| `p.J`, `p.P`, `p.C`, `p.G`, `p.I` | — | quantum numbers (`J`, `I` are `Fraction`) |
| `p.spin_type` | — | `SpinType` enum (Scalar, Vector, …) |
| `p.invert()` | — | antiparticle |
| `p.describe()` | — | full human-readable summary |
| `p.latex_name` | — | `\pi^{+}` |
| `p.programmatic_name` | — | `"pi_plus"` |

## PDGID properties

On `p.pdgid` or standalone `PDGID(id)`:

| Property | Meaning |
|---|---|
| `.is_valid` | PDG ID is well formed — **not** proof it is in the table |
| `.is_meson` / `.is_baryon` / `.is_lepton` | is a meson / baryon / lepton |
| `.is_hadron` | is a hadron (meson or baryon) |
| `.has_bottom` / `.has_charm` / `.has_strange` | contains a b / c / s quark |
| `.is_nucleus` | is a nuclear PDG ID |
| `.is_generator_specific` | generator-internal code (e.g. 99) |
| `.J` | total spin (`Fraction`) |
| `.charge` | charge in units of e |

```python
from particle import PDGID, Particle

pid = PDGID(211)
print(pid.is_valid, pid.is_meson, pid.has_strange)       # True True False
print(pid.is_lepton, pid.is_hadron, pid.J, pid.charge)   # False True 0 1

print(PDGID(99999999).is_valid)           # False (malformed)
print(PDGID(99).is_generator_specific)    # True

p = Particle.from_pdgid(5122)
print(p.name, p.pdgid.has_bottom)         # Lambda(b)0 True
```

## Stability

There is no `p.is_stable` — test the lifetime. Only truly stable particles
(e, p, γ, ν) have `lifetime == inf`; the free neutron has a finite lifetime.

```python
from particle import Particle

def is_stable(pdgid):
    return Particle.from_pdgid(pdgid).lifetime == float("inf")

print(is_stable(11), is_stable(2212), is_stable(211))   # True True False
```

## Filtering

```python
from particle import Particle

Particle.findall(lambda p: p.pdgid.has_bottom and p.charge == 0, particle=True)
Particle.findall(J=1, P=-1)                       # also pdg_name="pi", charge=1, ...
Particle.findall("K*")                            # string = fuzzy substring on name, NOT a glob
Particle.finditer(lambda p: p.pdgid.has_charm)    # lazy generator
```

`findall` returns a list; `finditer` is a lazy generator — prefer
`finditer` for large scans.

## Generator ID conversion

```python
from particle import Particle, PDGID, Geant3ID, PythiaID, Corsika7ID

print(Particle.from_pdgid(Geant3ID(8).to_pdgid()).name)      # pi+
print(Particle.from_pdgid(Corsika7ID(5).to_pdgid()).name)    # mu+
print(PythiaID(10221).to_pdgid())                            # <PDGID: 9010221>
print(PythiaID.from_pdgid(PDGID(9010221)))                   # <PythiaID: 10221>
```

Prefer `PythiaID.from_pdgid()` / `.to_pdgid()` — `Pythia2PDGIDBiMap` is deprecated.

## Gotchas

- Masses are in **MeV**, not GeV.
- **Stable particles** (p, e⁻, γ, ν) have `p.width == 0.0`, not `None`, and `p.lifetime == inf`. Width/lifetime are `None` only when unknown (e.g. quarks). **`p.is_stable` does not exist** — absent in particle 0.10.0, 0.16.3, 0.26.2 and 1.0.1; use `p.lifetime == inf`.
- `p.pdgid` is a `PDGID` object — use `int(p.pdgid)` for numpy/awkward.
- **Unknown IDs raise two different exceptions**: `InvalidParticle` for a malformed ID (`99999999`), `ParticleNotFound` for a well-formed ID absent from the table (`99`, `9999999`). Both are `RuntimeError` subclasses, not related to each other. `PDGID.is_valid` is `True` for `99` and `9999999`, so it does **not** prevent `ParticleNotFound` — catch both:

  ```python
  from particle import Particle, ParticleNotFound, InvalidParticle

  try:
      p = Particle.from_pdgid(9999999)
  except (ParticleNotFound, InvalidParticle):
      p = None
  ```
  (`from particle.exceptions import ...` does not work.)
- `findall` includes antiparticles; pass `particle=True` to exclude them.
- A string given to `findall` is a fuzzy name match, not a glob: `"K[+-]"` and `"pi*"` match nothing. Use a lambda with `fnmatch` for patterns.
- `Particle` has **no `decay_modes` attribute** — for branching fractions and decay chains use the `decaylanguage` package.


## References

- `references/advanced-usage.md` — antiparticles, literals, truth classification, `describe()`, more filtering and converter patterns.
- Docs: https://github.com/scikit-hep/particle#readme
