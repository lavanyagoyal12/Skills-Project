---
name: hepunits-usage
description: Use this skill when writing or debugging Python code that uses scikit-hep/hepunits for HEP/CLHEP units, physical constants, unit conversions, or optional Pint interoperability through to_clhep and from_clhep.
---

# Using hepunits (HEP System of Units)

> hepunits values are **plain `float` numbers in HEP base units — not dimension-aware quantities.** There is no quantity class, no unit attached to a value, and no runtime dimension checking in the package core (only the optional Pint layer checks dimensions).

## When to use this skill

- Your task involves units or constants in the HEP/CLHEP convention (GEANT4, Gaudi), or converting values between HEP units and SI-like units.
- Your task imports from `hepunits`, `hepunits.units`, `hepunits.constants`, or `hepunits.pint`, or a numeric factor looks wrong.
- You are modifying the `hepunits` package itself (adding/renaming units or constants).

## Core HEP/CLHEP unit conventions

The base units are defined in `src/hepunits/units/units.py` and set to `1.0`:

| Quantity | Base unit | Value |
|---|---|---|
| Length | `millimeter` (`mm`) | 1.0 |
| Time | `nanosecond` (`ns`) | 1.0 |
| Energy | `megaelectronvolt` (`MeV`) | 1.0 |
| Charge | `eplus` (positron charge) | 1.0 |
| Temperature | `kelvin` (`K`) | 1.0 |
| Amount | `mole` (`mol`) | 1.0 |
| Luminous intensity | `candela` (`cd`) | 1.0 |
| Angles | `radian` / `steradian` | 1.0 |

Every other unit is a plain `float` derived arithmetically from these: `meter = _pre.kilo * millimeter` (1000.0), `second = _pre.giga * nanosecond` (1e9), `electronvolt = _pre.micro * megaelectronvolt` (1e-6), `gigaelectronvolt` (`GeV`) = 1000.0, `joule = electronvolt / e_SI`. `e_SI = 1.602176634e-19` (exact, PDG 2022). So 1 J ≈ 6.24151e12 MeV.

**This is not SI**: all numbers are expressed in mm, ns, MeV, eplus. Read them back by dividing.

## Importing units and constants

All of these work and return the same object:

```python
from hepunits import GeV, c_light                 # top level (everything flat)
from hepunits.units import GeV, picosecond        # subpackage
from hepunits.units.units import GeV              # defining module
from hepunits.constants import c_light            # constants subpackage
from hepunits.constants.constants import c_light  # defining module
```

Frequently used constants (all in `src/hepunits/constants/constants.py`): `c_light` (299.792458 mm/ns), `Avogadro`, `h_Planck`, `hbar`, `hbar_Planck`, `hbarc`, `hbarc_sq`, `k_Boltzmann`, `e_SI`, `eplus`, `eminus`, `e_sq`, plus `pi`, `two_pi`, `half_pi`, `pi_sq`.

## Unit conversions

A "value with units" is just `number * unit`, stored in HEP base units. To express a value in a target unit, divide the quantity by the target unit. For example, 500 * keV / GeV gives 0.0005. This works because hepunits represents units as numerical scale factors in the HEP/CLHEP system; it does not perform dimensional analysis automatically.
```python
from hepunits.units import MeV, GeV, keV, mm, micrometer
from hepunits.constants import c_light
from hepunits.units import picosecond

1.0 * MeV / GeV          # -> 0.001
500.0 * keV / GeV        # -> 0.0005
3 * mm / micrometer      # -> 3000.0

tau_Bs = 1.5 * picosecond                 # 0.0015 (ns)
ctau_Bs = c_light * tau_Bs                # 0.44968868700000003 (mm)
ctau_Bs / micrometer                      # -> 449.688687
```

## Pint integration and its limitations

`hepunits.pint` is optional; importing it without Pint raises `ImportError: Pint is required to use hepunits.pint.` (repo pins `pint<0.25.1`).

```python
import pint
import hepunits
from hepunits.pint import to_clhep, from_clhep

ureg = pint.UnitRegistry()
to_clhep(9.8 * ureg.meter / ureg.second**2)              # -> 9.800000000000001e-15 (mm/ns**2)
to_clhep(3 * ureg.cm)                                    # -> 30.0 (mm)
from_clhep(hepunits.c_light, ureg.meter / ureg.second)   # -> 299792458.0 meter / second
```

Limitations:

- Only four dimensions map to CLHEP, in `_clhep_base_units` (`src/hepunits/pint.py`): `[length]` → `millimeter`, `[time]` → `nanosecond`, `[mass]` → `MeV * millimeter**-2 * nanosecond**2`, `[current]` → `elementary_charge / nanosecond`.
- Anything else raises `ValueError` with message `Unsupported dimension [...] in ...`, e.g. `to_clhep(1 * ureg.kelvin)` (temperature) or `to_clhep(1 * ureg.mole)` (amount of substance).
- `to_clhep(val)` returns a plain `float`; `from_clhep(val, unit)` returns a Pint `Quantity`.

## Common mistakes and pitfalls

- **Multiply/divide mix-up**: convert with `value / unit`, not `value * unit`.
- **Expecting dimension checks in the core**: `1 * meter + 1 * second` silently returns `1000001000.0`, and `5 * MeV - 3 * meter` returns `-2995.0`. Only `hepunits.pint` checks dimensions.
- **Assuming SI bases**: 1 m = 1000 mm, 1 s = 1e9 ns, 1 J ≈ 6.24e12 MeV, `kilogram = joule * second**2 / meter2`.
- **`eplus` vs `e_SI`**: `eplus = 1.0` is the base charge; `e_SI = 1.602176634e-19` is the SI coulomb.
- **`gray == sievert` numerically** (both `joule / kilogram`): equal in value, different physical concepts.
- **`as` is not available** (reserved word): attosecond has no short alias; `zs` (`zeptosecond`) and `ys` (`yoctosecond`) do.
- **Alias subtleties**: `year = 365.25 * day`; `deg` = degrees (π/180 rad), `mrad` = milliradian; `G` and `Gs` both mean gauss (`kG`/`kGs` = kilogauss), and `A` = ampere not area.
- **SI vs binary prefixes**: `mega = 1e6`, `kibi = 2**10` — don't mix powers of 1000 and 1024.

## Verified examples

Values confirmed by executing against this repo's `src/` with Pint 0.25:

```pycon
>>> from hepunits.units import MeV, GeV, keV, mm, micrometer
>>> 1.0 * MeV / GeV
0.001
>>> 500.0 * keV / GeV
0.0005
>>> 3 * mm / micrometer
3000.0
>>> from hepunits import c_light; from hepunits.units import picosecond
>>> (c_light * (1.5 * picosecond)) / micrometer
449.688687
>>> import pint
>>> from hepunits.pint import to_clhep
>>> to_clhep(1 * pint.UnitRegistry().kelvin)
ValueError: Unsupported dimension [temperature] in 1 kelvin
```

## Guidance for modifying hepunits itself (contributor)
Everything above is **usage** instructions. This section is for contributing to the `scikit-hep/hepunits` source.

- **Keep `__all__` in sync at every level.** When adding/renaming a unit or constant, update the matching `__all__` and import lists in: the defining module (`src/hepunits/units/units.py` or `src/hepunits/constants/constants.py`) → its package `__init__.py` (`src/hepunits/units/__init__.py` or `src/hepunits/constants/__init__.py`) → the top-level `src/hepunits/__init__.py`. Lists are alphabetically sorted (ruff `RUF022`). Skipping a level fails:
  - `tests/test_missing_all.py::test_missing_all` — each submodule's `__all__` must exactly equal its public namespace.
  - `tests/test_missing_all.py::test_exported` — every submodule name must appear in the parent's `__all__` up to `hepunits.__all__`.
- **Cite values**: numeric constants follow the PDG review (currently 2020/2022); add a comment such as `# exact value, taken from PDG 2022` (see `e_SI` in `units.py`).
- **Run the tests**: `uv run pytest` (pytest is configured with `filterwarnings = ["error"]` and `xfail_strict`, and `pint<0.25.1` is pinned). If `uv` is unavailable: `.venv/bin/python -m pytest`.
- **Lint/type**: ruff plus mypy `strict` over `src` (`nox -s lint` runs pre-commit).
- **Never edit `src/hepunits/_version.py`** — it is generated by hatch-vcs.
- **Contribution requirements**: the in-repo guidance is `AGENTS.md`. No skill-specific format exists in this repository, and this skill was created locally; before submitting it upstream, still check the central scikit-hep/skills repository's contribution requirements.
