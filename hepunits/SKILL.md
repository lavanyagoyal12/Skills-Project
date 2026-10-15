---
name: hepunits
description: >-
  Use when writing unit-safe HEP code in Python: converting between MeV and GeV,
  expressing cross-sections or luminosities with explicit units, computing ctau
  or decay lengths from particle lifetimes, checking that cut thresholds or
  histogram ranges carry correct units, or bridging hepunits values with Pint
  quantities.
metadata:
  validated-against: hepunits 2.4.7, numpy 2.5.4, hist 2.12.0, particle 1.0.1, pint 0.26.1 (Python 3.14.8)
  derived-from: usatlas/marketplace hepunits skill
---

# hepunits
## What it is
hepunits provides unit and physical constants as **plain Python floats** in the CLHEP base system (**MeV**, **mm**, **ns**, **eplus**). No unit objects, no dimension checks: `1 * mm + 1 * ns == 2.0`.

## Core rules
1. `value * unit` converts **into** base units (`25 * GeV` → `25000.0`).
2. `value / unit` converts **out of** base units (`25000.0 / GeV` → `25.0`).
3. `1 * GeV == 1000.0` — the base energy unit is MeV.
4. Wrong `*` or `/`, and mixed dimensions, fail silently (`1 * mm + 1 * ns == 2.0`).
5. Prefer named units (`25 * GeV`) over magic numbers (`25000.0`).

## Names
If a name is not listed here or in `references/units.md`, verify it with `hasattr(hepunits, name)` before using it — never guess. Single-letter `m`, `s`, `h`, `d`, `y` collide with ordinary variables (use `meter`, `second`, `hour`, `day`, `year`); no `um` (use `micrometer`); no bare `b` for barn (use `barn`).

## Quick reference
| Name | Value (base) | Name | Value (base) |
| --- | --- | --- | --- |
| `GeV` | 1000.0 | `ns` | 1.0 |
| `TeV` | 1000000.0 | `ps` | 0.001 |
| `mm` | 1.0 | `pb` | 1e-34 |
| `cm` | 10.0 | `fb` | 1e-37 |
| `m` | 1000.0 | `invfb` | 1e+37 |
| `c_light` | 299.792458 (mm/ns) | `hbarc` | 1.973269804593025e-10 (MeV·mm) |

## Snippets
```python
import numpy as np
from hepunits import GeV
pt = np.array([25000.0, 500000.0, 1000000.0])        # MeV
print((pt[pt > 200.0 * GeV] / GeV).tolist())         # [500.0, 1000.0]
```
```python
from hepunits import GeV, TeV
print(25000.0 / GeV)                                 # 25.0
print(1.8 * TeV)                                     # 1800000.0
```
```python
from hepunits import c_light, picosecond, micrometer
tau = 1.5 * picosecond                               # 0.0015 ns (base unit)
print(c_light * tau)                                 # 0.44968868700000003
print(c_light * tau / micrometer)                    # 449.688687
```
```python
import numpy as np
from hepunits import MeV
from particle import Particle
m_z = Particle.from_name("Z0").mass                  # 91187.9 (MeV)
m = np.array([91000.0, 91187.9, 91300.0])            # candidates (MeV)
print(m[(m > m_z - 10 * MeV) & (m < m_z + 10 * MeV)].tolist())  # [91187.9]
```
```python
from hepunits import pb, picobarn, invfb, invpb
print(invpb * pb)                                    # 1.0
print(40.0 * picobarn)                               # 3.9999999999999995e-33
print((10.0 * invfb) * (1.0 * pb))                   # 9999.999999999998
```
```python
import numpy as np
import hist
from hepunits import GeV
h = hist.Hist(hist.axis.Regular(50, 0 * GeV, 1000 * GeV, name="pt"))
h.fill(pt=np.array([25000.0, 500000.0]))
print(float(h.axes[0].edges[-1]))                    # 1000000.0
```
```python
from hepunits import c_light
from particle import Particle
b0 = Particle.from_name("B0")
print(c_light * b0.lifetime)                         # 0.45310443274237083
print(b0.ctau)                                       # 0.45310443274237083
print(c_light * b0.lifetime == b0.ctau)              # True
```

## Gotchas
- **Data units**: hepunits cannot tell you the units of your data, so check your sample before converting.

## References
| File | Contains | Open when |
| --- | --- | --- |
| [references/units.md](references/units.md) | commonly used unit values by category, aliases | looking up a unit name or value |
| [references/constants.md](references/constants.md) | physical & math constants | computing ctau, energies, Boltzmann factors |
| [references/prefixes.md](references/prefixes.md) | SI & binary prefixes | combining prefixes with units |
| [references/conventions.md](references/conventions.md) | base system, ×/÷, silent mistakes, where names live | explaining or sanity-checking unit arithmetic |
| [references/recipes.md](references/recipes.md) | 7 runnable examples | writing analysis code |
| [references/interop.md](references/interop.md) | particle, Pint, numpy, hist, vector/awkward | bridging libraries |
