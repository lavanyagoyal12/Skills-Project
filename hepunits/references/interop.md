# hepunits interoperability

Purpose: how hepunits plain floats relate to particle, Pint, NumPy, hist and vector/awkward.
Validated against: hepunits 2.4.7, numpy 2.5.4, hist 2.12.0, particle 1.0.1, pint 0.26.1, vector 1.9.0 (Python 3.14.8).

## particle

`particle` depends on `hepunits`, so their unit conventions line up (verified from the `particle` source docstrings): `mass` in MeV, `lifetime` in ns, `ctau` in mm.

```python
from particle import Particle
from hepunits import c_light
b0 = Particle.from_name("B0")
print(float(c_light * b0.lifetime))                 # 0.45310443274237083
print(float(b0.ctau))                               # 0.45310443274237083
print(c_light * b0.lifetime == b0.ctau)             # True
```

Use this equality as a built-in cross-check for hand-computed decay lengths.

## hepunits.pint (optional)

```python
import pint
import hepunits
from hepunits.pint import to_clhep, from_clhep
ureg = pint.UnitRegistry()
print(to_clhep(9.8 * ureg.meter / ureg.second**2))   # 9.800000000000001e-15
print(from_clhep(hepunits.c_light, ureg.meter / ureg.second))  # "299792458.0 meter / second"
```

- Supported dimensions (from `_clhep_base_units` in `hepunits/pint.py`): `[length]`, `[time]`, `[mass]`, `[current]`.
- Anything else raises `ValueError`, e.g. `to_clhep(300 * ureg.kelvin)` → `Unsupported dimension [temperature] in 300 kelvin`.
- Without pint installed, `import hepunits.pint` raises `ImportError: Pint is required to use hepunits.pint.` — guard the import or declare the dependency.

## hist

`hist` axes and `fill()` receive plain floats — nothing converts units. Keep axis limits and fill values in the **same** unit system; see `recipes.md` recipe 7.

## vector / awkward

No built-in integration; convert to a consistent unit system before constructing four-vectors. `vector.Array` in momentum mode takes either `E` or `t`/`tau` (not both) — verified: `{"pt": …, "eta": …, "phi": …, "E": …}` runs.

## Scope note

hepunits is **not** a units library: no automatic conversion, no dimension checking, no unit objects. For dimension checking use Pint via `hepunits.pint` (limited dimensions) or keep unit discipline in your own code.
