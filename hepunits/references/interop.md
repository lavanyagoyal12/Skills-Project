# hepunits interoperability

Purpose: how hepunits plain floats relate to particle, Pint, NumPy, hist and
vector/awkward.
Validated against: hepunits 2.4.7, numpy 2.5.4, hist 2.12.0, particle 1.0.1,
pint 0.26.1, vector 1.9.0 (Python 3.14.8); Pint behavior cross-checked against
the upstream `tests/test_pint.py` (v2.4.7).

## particle

`particle` declares `hepunits>=2.4.0` as a dependency, so their unit
conventions line up (verified from the `particle` source): `mass` in **MeV**,
`width`/`lifetime` in **ns**, `ctau` in **mm**. In fact `ctau` is implemented
as `c_light * self.lifetime`.

```python
from particle import Particle
from hepunits import c_light
b0 = Particle.from_name("B0")
print(float(c_light * b0.lifetime))                 # 0.45310443274237083
print(float(b0.ctau))                               # 0.45310443274237083
print(c_light * b0.lifetime == b0.ctau)             # True
```

Use this equality as a built-in cross-check for hand-computed decay lengths
(see [recipes.md](recipes.md) recipe 4). `mass`/`lifetime`/`ctau` can be `None`
when the width is unknown.

## hepunits.pint (optional)

`hepunits.pint` converts between Pint quantities and CLHEP base units. Pint is
an **optional** dependency: importing `hepunits.pint` without Pint installed
raises `ImportError: Pint is required to use hepunits.pint.`

```python
import pint
import hepunits
from hepunits.pint import to_clhep, from_clhep
ureg = pint.UnitRegistry()
print(to_clhep(9.8 * ureg.meter / ureg.second**2))   # 9.800000000000001e-15
print(to_clhep(3 * ureg.cm))                         # 30.0
print(to_clhep(ureg.coulomb) == hepunits.coulomb)    # True
print(from_clhep(hepunits.c_light, ureg.meter / ureg.second))  # 299792458.0 meter / second
print(from_clhep(hepunits.tesla, ureg.tesla))        # 1.0 tesla
```

API:

- `to_clhep(val)` — accepts a Pint `Quantity` **or a bare `Unit`**
  (`to_clhep(ureg.meter)` → `1000.0`); returns the magnitude in CLHEP base
  units as a plain `float`.
- `from_clhep(val, unit)` — takes a CLHEP magnitude (`float`) and a Pint
  `Unit`; returns a Pint `Quantity`.

Only four dimensions map to CLHEP (`_clhep_base_units` in `hepunits/pint.py`):

| Pint dimension | CLHEP unit |
| --- | --- |
| `[length]` | `millimeter` |
| `[time]` | `nanosecond` |
| `[mass]` | `MeV * millimeter**-2 * nanosecond**2` |
| `[current]` | `elementary_charge / nanosecond` |

Anything else raises `ValueError` — for both functions (the target unit is
checked too):

```python
import pint
from hepunits.pint import to_clhep, from_clhep
ureg = pint.UnitRegistry()
to_clhep(1 * ureg.kelvin)      # ValueError: Unsupported dimension [temperature] in 1 kelvin
to_clhep(1 * ureg.mole)        # ValueError: Unsupported dimension [substance] in 1 mole
from_clhep(1.0, ureg.kelvin)   # ValueError: Unsupported dimension [temperature] in kelvin
```

Caveats:

- With the default Pint registry, angles are dimensionless, so
  `to_clhep(1 * ureg.radian)` returns `1` instead of raising.
- Combine hepunits floats with Pint only through `to_clhep`/`from_clhep`:
  `1 * hepunits.mm * to_clhep(3 * ureg.nanosecond) == 3 * hepunits.mm * hepunits.nanosecond`
  (`True`), and `from_clhep(1 * hepunits.mm, ureg.mm) * (3 * ureg.nanosecond) == 3 * ureg.mm * ureg.nanosecond`
  (`True`).
- Verified against pint 0.26.1; the upstream test extras pin `pint<0.25.1`
  (`tests/test_pint.py` checks speed of light / Planck / elementary charge to
  ~1e-15 relative).

## hist

`hist` axes and `fill()` receive plain floats — nothing converts units. Keep
axis limits and fill values in the **same** unit system
([recipes.md](recipes.md) recipe 7):

```python
import numpy as np
import hist
from hepunits import GeV
h = hist.Hist(hist.axis.Regular(50, 0 * GeV, 1000 * GeV, name="pt"))
h.fill(pt=np.array([25000.0, 500000.0]))               # MeV, matching the axis
```

## NumPy

No special support needed: units are floats, so they compose with NumPy
scalars, masks and array arithmetic natively. Write thresholds as
`200.0 * GeV` and convert output arrays by dividing (`pt / GeV` for GeV).

## vector / awkward

No built-in integration — convert to a consistent unit system before
constructing four-vectors. `vector` never converts units (see the `vector`
skill). One vector-specific note: in momentum mode supply either `E` or
`t`/`tau`, not both (`TypeError: specify t= or tau=, but not more than one`).

## Scope note

hepunits is **not** a units library: no automatic conversion, no dimension
checking, no unit objects. For dimension checking use Pint via
`hepunits.pint` (four dimensions only) or keep unit discipline in your own
code.
