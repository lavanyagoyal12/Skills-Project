# hepunits conventions

Purpose: how the CLHEP base system works, how units are derived, the two
arithmetic directions, silent mistakes, where names live, and how to patch
the package.
Validated against: hepunits 2.4.7 (source + values), numpy 2.5.4, hist 2.12.0,
particle 1.0.1, pint 0.26.1 (Python 3.14.8); upstream structure verified
against scikit-hep/hepunits v2.4.7.

## The base system

hepunits stores every quantity as a **plain float in the base units** (no unit
objects, no dimensions): length in mm, time in ns, energy in MeV, charge in
eplus. A unit constant is the factor from that unit into the base unit
(`GeV` = `1000.0` MeV). **This is not SI** — read a raw number by dividing.

Base units are defined in `hepunits/units/units.py`, each set to `1.0`:

| Quantity | Name | Short alias |
| --- | --- | --- |
| Length | `millimeter` | `mm` |
| Time | `nanosecond` | `ns` |
| Energy | `megaelectronvolt` | `MeV` |
| Charge | `eplus` | — |
| Temperature | `kelvin` | `K` |
| Amount of substance | `mole` | `mol` |
| Luminous intensity | `candela` | `cd` |
| Plane / solid angle | `radian` / `steradian` | `rad` / `sr` |

## How other units are derived (source-verified)

| Unit | Definition | Value (base) |
| --- | --- | --- |
| `meter` | `_pre.kilo * millimeter` | 1000.0 |
| `second` | `_pre.giga * nanosecond` | 1e9 |
| `electronvolt` | `_pre.micro * megaelectronvolt` | 1e-6 |
| `joule` | `electronvolt / e_SI` | 6241509074460.763 (MeV) |
| `kilogram` | `joule * second**2 / meter2` | 6.241509074460762e+24 (MeV) |
| `barn` | `1.0e-28 * meter2` | 9.999999999999999e-23 (mm²) |
| `c_light` | `299792458 * m / s` | 299.792458 (mm/ns) |
| `coulomb` | `eplus / e_SI` | 6.241509074460763e+18 |
| `ampere` | `coulomb / second` | 6241509074.460763 |
| `hertz` | `1.0 / second` | 1e-9 |
| `year` | `365.25 * day` | 3.15576e+16 |

`e_SI = 1.602176634e-19` (exact, PDG 2022), so 1 J ≈ 6.24151e12 MeV.

## Where names live

| Import | What you get |
| --- | --- |
| `from hepunits import GeV, c_light, mega` | everything (flat) |
| `from hepunits.units import GeV, picosecond, kibi` | all units + prefixes |
| `from hepunits.units.units import GeV` | defining module |
| `from hepunits.constants import c_light` | constants (+ re-exported units) |
| `from hepunits.constants.constants import c_light` | defining module |
| `from hepunits.pint import to_clhep, from_clhep` | optional Pint bridge |

All paths return the **same object** (verified with `is`). Every name in a
submodule's `__all__` is re-exported up to `hepunits.__all__` (enforced by the
upstream `tests/test_missing_all.py`).

## The two arithmetic directions

```python
import hepunits.units as u
print(1 * u.meter + 5 * u.cm)                # 1050.0  (both converted to mm)
print((1 * u.meter + 5 * u.cm) / u.meter)    # 1.05    (read back out in m)
```

- `value * unit` → **into** base units.
- `value / unit` → **out of** base units.
- Never multiply to convert: `1.0 * MeV / GeV` → `0.001`, and
  `500.0 * keV / GeV` → `0.0005`.

## Common silent mistakes

| Mistake | Code | Silently gives | Correct |
| --- | --- | --- | --- |
| Multiply instead of divide to read GeV | `25000.0 * GeV` | 25000000.0 | `25000.0 / GeV` → 25.0 |
| GeV/MeV confusion | `1 * GeV` | 1000.0 (MeV!) | remember the base energy unit is MeV |
| Mixing dimensions | `1 * mm + 1 * ns` | 2.0, no error | never mix; convert first |
| Mixing dimensions | `1 * meter + 1 * second` | 1000001000.0, no error | convert first |
| Mixing dimensions | `5 * MeV - 3 * meter` | -2995.0, no error | convert first |
| Treating values as SI | `c_light` | 299.792458 (mm/ns) | ×1e6 → m/s, or use Pint |
| Single letters shadow constants | `m = 5` | `m` is now 5, not 1000.0 mm | use `meter`, `second`, `hour`, `day`, `year` |

`m`, `s`, `h`, `d`, `y` are valid hepunits constants but collide with ordinary
variable names — reassigning `m` silently breaks every later `m` use.
There is **no runtime dimension checking** in the core; only the optional
`hepunits.pint` layer checks dimensions ([interop.md](interop.md)).

## Alias subtleties

- `year = 365.25 * day`; `hour = 60 * minute`, `day = 24 * hour`.
- `A` = **ampere** (not area); `G` and `Gs` = **gauss** (`kG`/`kGs` = kilogauss;
  tesla is `T`); `C` = coulomb.
- `gray == sievert` numerically (both `joule / kilogram`): equal in value,
  different physical concepts.
- `eplus = 1.0` is the base charge unit; `e_SI = 1.602176634e-19` is the SI
  coulomb — don't confuse them.
- Sub-SI symbols use `u`, not `µ`: `us`, `uA`, `uF`, `uGy`, `uCi`, `uWb`.
- `deg` = degrees (π/180 rad), `mrad` = milliradian.
- Short and long names are interchangeable (`eV`/`electronvolt`,
  `fm`/`fermi`, `m2`/`meter2`); see [units.md#aliases](units.md#aliases).

## Prefixes

Defined in `hepunits/units/prefixes.py`, re-exported everywhere
(`from hepunits import mega, milli, kibi`). SI prefixes are powers of 10
(`mega = 1e6`), binary prefixes are powers of 2 (`mebi = 2**20`) — don't mix
the families. Values and identities: [units.md#prefixes](units.md#prefixes).

## If you patch hepunits (upstream repo)

These points are for working in the scikit-hep/hepunits source, not for using
the package.

- **Keep `__all__` in sync at every level** when adding/renaming a unit or
  constant: defining module → package `__init__.py` → top-level
  `hepunits/__init__.py`. `tests/test_missing_all.py::test_missing_all` and
  `::test_exported` enforce exactly this.
- **Cite values**: numeric constants follow the PDG review (currently
  2020/2022); add a comment such as `# exact value, taken from PDG 2022`.
- **Run the tests**: `uv run pytest` (pytest is configured with
  `filterwarnings = ["error"]` and `xfail_strict`; the test extras pin
  `pint<0.25.1`).
- **Never edit `src/hepunits/_version.py`** — it is generated by hatch-vcs.
