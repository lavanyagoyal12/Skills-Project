# HEP/CLHEP unit conventions

> Values below verified against `src/hepunits` with Pint 0.25 (see final section for how).

## Base units

Defined in `src/hepunits/units/units.py`, each set to `1.0`:

| Quantity | Name | Short alias |
|---|---|---|
| Length | `millimeter` | `mm` |
| Time | `nanosecond` | `ns` |
| Energy | `megaelectronvolt` | `MeV` |
| Charge | `eplus` | — |
| Temperature | `kelvin` | `K` |
| Amount of substance | `mole` | `mol` |
| Luminous intensity | `candela` | `cd` |
| Plane / solid angle | `radian` / `steradian` | `rad` / `sr` |

Everything else is a plain `float` derived arithmetically from these. **This is not SI**: a raw number is always in mm, ns, MeV, eplus.

Key scale factors (verified):

- `meter = 1000.0`, `second = 1e9`, `electronvolt = 1e-6`
- `GeV = 1000.0`, `keV = 1e-3`, `eV = 1e-6`
- `joule = electronvolt / e_SI` ≈ `6.24151e12` (exactly `6241509074460.763`)
- `kilogram = joule * second**2 / meter2` ≈ `6.24151e24`
- `barn = 1e-28 * meter2` ≈ `100 * fm2`
- `c_light = 299792458 * m / s` = `299.792458` (mm/ns)

## Importing units

All of these work and return the same object (verified with `is`):

```python
from hepunits import GeV                       # top level (everything flat)
from hepunits.units import GeV                 # subpackage
from hepunits.units.units import GeV           # defining module
```

Every name in `hepunits.units.units.__all__` is re-exported all the way up to `hepunits` (enforced by `tests/test_missing_all.py`).

## Prefixes

Defined in `src/hepunits/units/prefixes.py`, re-exported everywhere (`from hepunits import mega, milli, kibi`):

- SI (powers of 10): `quetta` 1e30, `zetta` 1e21, `giga` 1e9, `mega` 1e6, `kilo` 1e3, `milli` 1e-3, `micro` 1e-6, `nano` 1e-9, `pico` 1e-12, … `quecto` 1e-30.
- Binary (powers of 2): `kibi` 2**10, `mebi` 2**20, `gibi` 2**30, `tebi` 2**40, `pebi`, `exbi`, `zebi`, `yobi`.
- Misc: `googol` = 1e100.

Don't mix the families: `mega = 1e6` but `mebi = 2**20`. Example from the tests: `1 * mega * joule / second == 1 * MW`.

## Conversion rules

A "value with units" is just a `float` in HEP base units.

- **Attach units by multiplying:** `mass_window = 500 * keV` (stored value `0.5`, i.e. MeV).
- **Convert by dividing:** `mass_window / GeV` → `0.0005`. Never multiply to convert.

```python
from hepunits.units import MeV, GeV, keV, mm, micrometer, meter, cm

1.0 * MeV / GeV                     # 0.001
500.0 * keV / GeV                   # 0.0005
3 * mm / micrometer                 # 3000.0
(1 * meter + 5 * cm) / meter        # 1.05
1 * meter + 5 * cm                  # 1050.0  (mm)
```

## Common aliases

- Length: `mm`/`cm`/`m`/`km`; `micrometer` also `micron` (there is **no `um`**); `femtometer` also `fermi`/`fm`; `angstrom`.
- Time: `ns`/`us`/`ms`/`s`; `ps`/`fs`/`zs`/`ys` — there is **no `as`** (reserved word; use `attosecond`); `minute`, `hour`/`h`, `day`/`d`, `year`/`y` (`year = 365.25 * day`).
- Energy: `eV`, `keV`, `MeV`, `GeV`, `TeV`, `PeV`, `EeV`, `ZeV`; `eV2` = `electronvolt2`.
- Cross sections: `barn`; `mb`/`ub`/`nb`/`pb`/`fb`/`ab` plus inverses `invmb` … `invab` (`invpb / invfb == 1e-3`).
- Sub-SI symbols use `u`, not `µ`: `us`, `uA`, `uF`, `uGy`, `uCi`, `uWb`.
- Ambiguous symbols (verified in `units.py`): `A` = **ampere** (not area); `G` and `Gs` = **gauss**, `kG`/`kGs` = kilogauss (tesla is `T`); `C` = coulomb; `deg` = degree, `mrad` = milliradian.

## Dimensional-safety pitfalls

There is **no runtime dimension checking** in the core — nothing catches nonsense:

```python
1 * meter + 1 * second     # 1000001000.0  (silently meaningless)
5 * MeV - 3 * meter        # -2995.0
gray == sievert            # True  (both joule/kilogram, different concepts)
```

Also: `eplus = 1.0` is the base unit of charge, while `e_SI = 1.602176634e-19` is the SI coulomb (exact, PDG 2022) — don't confuse them.
