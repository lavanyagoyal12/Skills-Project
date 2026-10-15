# Optional Pint interoperability

`hepunits.pint` converts between Pint quantities and CLHEP/HEP base units. Pint is an **optional** dependency — the core package works without it.

- Importing `hepunits.pint` without Pint installed raises `ImportError: Pint is required to use hepunits.pint.` (`src/hepunits/pint.py`).
- The repo's test extras pin `pint<0.25.1` (`pyproject.toml`), and `tests/test_pint.py` covers the behavior below.
- Numbers below were verified with Pint 0.25 against this repo's `src/`.

## API

```python
from hepunits.pint import to_clhep, from_clhep
```

- `to_clhep(val)` — accepts a Pint `Quantity` (or a bare `Unit`, e.g. `to_clhep(ureg.meter)` → `1000.0`); returns the magnitude in CLHEP base units (typed `float`).
- `from_clhep(val, unit)` — takes a CLHEP magnitude (`float`) and a Pint `Unit`; returns a Pint `Quantity`.

## Supported dimensions

Only four dimensions map to CLHEP (`_clhep_base_units` in `src/hepunits/pint.py`):

| Pint dimension | CLHEP unit |
|---|---|
| `[length]` | `millimeter` |
| `[time]` | `nanosecond` |
| `[mass]` | `MeV * millimeter**-2 * nanosecond**2` |
| `[current]` | `elementary_charge / nanosecond` |

Any other dimension raises `ValueError` with `Unsupported dimension [...] in <value>` — for both functions (the target unit is checked too):

```python
to_clhep(1 * ureg.kelvin)      # ValueError: Unsupported dimension [temperature] in 1 kelvin
to_clhep(1 * ureg.mole)        # ValueError: Unsupported dimension [substance] in 1 mole
from_clhep(1.0, ureg.kelvin)   # ValueError: Unsupported dimension [temperature] in kelvin
```

Note: with the default Pint registry, angles are dimensionless, so `to_clhep(1 * ureg.radian)` returns `1` instead of raising.

## Verified examples

```python
import pint
import hepunits
from hepunits.pint import to_clhep, from_clhep

ureg = pint.UnitRegistry()

to_clhep(9.8 * ureg.meter / ureg.second**2)               # 9.800000000000001e-15  (mm/ns**2)
to_clhep(3 * ureg.cm)                                     # 30.0  (mm)
to_clhep(ureg.coulomb) == hepunits.coulomb                # True
to_clhep(1 * ureg.speed_of_light)                         # 299.79245800000007  (≈ c_light)

from_clhep(hepunits.c_light, ureg.meter / ureg.second)    # 299792458.0 meter / second
from_clhep(hepunits.c_light, ureg.fathom / ureg.fortnight)  # 198287528400528.03 fathom / fortnight
from_clhep(hepunits.tesla, ureg.tesla)                    # 1.0 tesla
```

## Round-tripping with plain hepunits values

hepunits values are plain floats, so combine them with Pint only through `to_clhep` / `from_clhep`:

```python
a = 1 * hepunits.mm
b = 3 * ureg.nanosecond
a * to_clhep(b) == 3 * hepunits.mm * hepunits.nanosecond     # True
from_clhep(a, ureg.mm) * b == 3 * ureg.mm * ureg.nanosecond  # True
```

(The consistency checks — including `speed_of_light`, `planck_constant`, `elementary_charge` matching `c_light`, `h_Planck`, `eplus` to ~1e-15 relative — live in `tests/test_pint.py`.)
