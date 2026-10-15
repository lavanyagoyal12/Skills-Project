# hepunits conventions

Purpose: how the CLHEP base system works, the two arithmetic directions, silent mistakes, and where names live.
Validated against: hepunits 2.4.7, numpy 2.5.4, hist 2.12.0, particle 1.0.1, pint 0.26.1 (Python 3.14.8).

## The base system

hepunits stores every quantity as a **plain float in the base units** (no unit objects): length in mm, time in ns, energy in MeV, charge in eplus. A unit constant is the factor from that unit into the base unit (`GeV` = `1000.0` MeV). All values are in [units.md](units.md).

## The two arithmetic directions

```python
import hepunits.units as u
# 1*m + 5*cm: both converted to mm, then added
print(1 * u.meter + 5 * u.cm)                # 1050.0
# divide by u.meter to read the result out in m
print((1 * u.meter + 5 * u.cm) / u.meter)    # 1.05
```

- `value * unit` → **into** base units.
- `value / unit` → **out of** base units.

## Common silent mistakes

| Mistake | Code | Silently gives | Correct |
| --- | --- | --- | --- |
| Multiply instead of divide to read GeV | `25000.0 * GeV` | 25000000.0 | `25000.0 / GeV` → 25.0 |
| GeV/MeV confusion | `1 * GeV` | 1000.0 (MeV!) | remember base energy is MeV |
| Mixing dimensions | `1 * mm + 1 * ns` | 2.0, no error | never mix; convert first |
| Treating values as SI | `c_light` | 299.792458 (mm/ns) | ×1e6 → m/s, or use Pint |
| Single letters shadow constants | `m = 5` | `m` is now 5, not 1000.0 mm | use `meter`, `second`, `hour`, `day`, `year` |

The last row is the real hazard behind `m`, `s`, `h`, `d`, `y`: they are valid hepunits constants (`m`=1000.0 mm, `s`=1e9 ns), but they collide with ordinary variable names — reassigning `m` silently breaks every later `m` use. Prefer long names in code.

## Aliases

Short and long names are interchangeable (e.g. `eV`/`electronvolt`, `fm`/`fermi`, `m2`/`meter2`); see [units.md#aliases](units.md#aliases). `um` and a bare `b` do **not** exist — use `micrometer` and `barn`.

## Where names live

- `hepunits` top level: `from hepunits import GeV, MeV, c_light, picosecond, mega`; `hepunits.units`: units/prefixes; `hepunits.constants`: constants; `hepunits.pint`: optional Pint bridge (`to_clhep`, `from_clhep`).
