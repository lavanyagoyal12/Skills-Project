# Physical and mathematical constants

All constants are **plain floats in HEP base units**, computed in `src/hepunits/constants/constants.py` from `hepunits.units`. No unit is attached to a value — divide by a unit to convert (see [conventions.md](conventions.md)).

## Importing

All of these work and return the same object (verified with `is`):

```python
from hepunits import c_light, hbar, pi             # top level (everything flat)
from hepunits.constants import c_light             # subpackage
from hepunits.constants.constants import hbar      # defining module
```

`hepunits.constants` also re-exports the unit names `e_SI`, `electronvolt`, `eplus`, `joule`, `kelvin`, `m`, `mole`, `s`.

## Mathematical constants

`pi` (from `math`), plus derived `two_pi`, `half_pi`, `pi_sq` — e.g. `pi_sq == 9.869604401089358`.

## Physical constants (HEP base units)

| Name | Value in HEP base units | Notes |
|---|---|---|
| `c_light` | `299.792458` | mm/ns = `299792458 * m / s`; exact |
| `c_light_sq` | `89875.51787368178` | |
| `Avogadro` | `6.02214076e23` | per mole (`/ mole`); exact |
| `h_Planck` | `4.135667696923859e-12` | MeV·ns; exact |
| `hbar` / `hbar_Planck` | `6.582119569509066e-13` | MeV·ns; `h_Planck / two_pi` |
| `hbarc` | `1.973269804593025e-10` | MeV·mm |
| `hbarc_sq` | `3.893793721718595e-20` | MeV²·mm² |
| `k_Boltzmann` | `8.617333262145178e-11` | MeV/K; exact |
| `eminus` | `-1.0` | electron charge = `-eplus` |
| `e_sq` | `1.0` | |
| `e_SI` | `1.602176634e-19` | elementary charge in coulomb; exact (also in `units.py`) |

**Source:** exact values are commented in the code as `# exact value, taken from PDG 2022` (PDG 2020/2022 review; see `AGENTS.md`).

## Verified examples

```python
from hepunits import c_light, hbar
from hepunits.units import picosecond, micrometer, eV, s, nanometer, m

c_light * (1.5 * picosecond)                # 0.44968868700000003 (mm)
(c_light * (1.5 * picosecond)) / micrometer # 449.688687
c_light / (m / s)                           # 299792458.0
hbar / (eV * s)                             # 6.582119569509066e-16
c_light / (555.17121851852 * nanometer)     # 539999.9999999985  (~540 THz)
```

The last line is the green-light check from `tests/constants/test_constants.py` (asserted `approx(540 * THz)`).
