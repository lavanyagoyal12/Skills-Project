---
name: hepunits
description: >-
  Use when writing unit-safe HEP code in Python: converting between MeV and GeV
  or other CLHEP units, expressing cross-sections or luminosities with explicit
  units, computing ctau or decay lengths from particle lifetimes, checking that
  cut thresholds or histogram ranges carry correct units, looking up a hepunits
  unit, prefix or constant name/value, or bridging hepunits floats with Pint
  quantities.
metadata:
  validated-against: hepunits 2.4.7, numpy 2.5.4, hist 2.12.0, particle 1.0.1, pint 0.26.1, vector 1.9.0 (Python 3.14.8)
  derived-from: usatlas/marketplace hepunits skill
---

# hepunits

hepunits provides units and physical constants as **plain Python floats** in the
CLHEP base system: length in **mm**, time in **ns**, energy in **MeV**, charge in
**eplus**. There are no unit objects and no dimension checks
(`1 * mm + 1 * ns` is `2.0`). A unit constant is just the conversion factor into
the base system (`GeV` is `1000.0` MeV).

## Two rules

1. `value * unit` converts **into** base units: `25 * GeV` → `25000.0`.
2. `value / unit` converts **out of** base units: `25000.0 / GeV` → `25.0`.

Slice out a cut as a scalar and read results back by dividing:

```python
import numpy as np
from hepunits import GeV, MeV
pt = np.array([25000.0, 500000.0, 1000000.0])        # MeV
print((pt[pt > 200.0 * GeV] / GeV).tolist())         # [500.0, 1000.0]
print(500.0 * MeV / GeV)                             # 0.5
```

## Base units (each == 1.0)

| Quantity | Names |
| --- | --- |
| Energy | `MeV`, `megaelectronvolt` |
| Length | `mm`, `millimeter` |
| Time | `ns`, `nanosecond` |
| Charge | `eplus` (+1.0), `eminus` (−1.0) |
| Temperature | `kelvin`, `K` |
| Amount | `mole`, `mol` |
| Luminous intensity | `candela`, `cd` |
| Angle | `radian`, `rad`, `steradian`, `sr` |

## Quick reference

| Name | Value (base) | Name | Value (base) |
| --- | --- | --- | --- |
| `GeV` | 1000.0 | `ns` | 1.0 |
| `TeV` | 1000000.0 | `ps` | 0.001 |
| `MeV` | 1.0 | `mm` | 1.0 |
| `keV` | 0.001 | `cm` | 10.0 |
| `eV` | 1e-06 | `m` | 1000.0 |
| `pb` | 1e-34 | `fb` | 1e-37 |
| `invfb` | 1e+37 | `barn` | 9.999999999999999e-23 |
| `c_light` | 299.792458 (mm/ns) | `hbarc` | 1.973269804593025e-10 (MeV·mm) |

## Names & gotchas

- Everything is importable from `hepunits` (top level); `from hepunits.units
  import picosecond` and `from hepunits.constants import c_light` return the
  same objects (see [references/conventions.md](references/conventions.md)).
- Single-letter names `m`, `s`, `h`, `d`, `y` exist but collide with ordinary
  variables — use `meter`, `second`, `hour`, `day`, `year`.
- **Not defined:** `um` (use `micrometer`/`micron`), `nm` (use `nanometer`),
  bare `b` (use `barn`), `as` (use `attosecond`), `pd`.
- If a name is not in the tables below, verify it with
  `hasattr(hepunits, name)` before using it — never guess.
- hepunits cannot tell you the units of your data; check your sample before
  converting.

## References

| File | Contains | Open when |
| --- | --- | --- |
| [references/units.md](references/units.md) | unit & prefix names/values by category, aliases, missing names | looking up a unit, prefix or alias |
| [references/constants.md](references/constants.md) | physical & math constants | computing ctau, Boltzmann factors, energies |
| [references/conventions.md](references/conventions.md) | base system, derivations, imports, silent mistakes | explaining or sanity-checking unit arithmetic |
| [references/interop.md](references/interop.md) | particle, Pint, numpy, hist, vector | bridging libraries |
| [references/recipes.md](references/recipes.md) | 7 runnable examples with output | writing analysis code |

`particle` and `vector` of Scikit-HEP have their own skills; only the
hepunits-specific parts are covered in [references/interop.md](references/interop.md).
