# hepunits constants reference

Purpose: key physical and mathematical constants in `hepunits.constants`, with
meaning and CLHEP base units.
Validated against: hepunits 2.4.7, numpy 2.5.4, hist 2.12.0, particle 1.0.1,
pint 0.26.1 (Python 3.14.8). Values were produced by
`repr(getattr(hepunits, name))` from the installed package.

All constants are **plain floats in HEP base units** (MeV, mm, ns); `c_light`
is in mm/ns, not m/s. No unit is attached — divide by a unit to convert (see
[conventions.md](conventions.md)).

## Importing

All of these work and return the same object (verified with `is`):

```python
from hepunits import c_light, hbar, pi             # top level (everything flat)
from hepunits.constants import c_light             # subpackage
from hepunits.constants.constants import hbar      # defining module
```

`hepunits.constants` also re-exports the unit names `e_SI`, `electronvolt`,
`eplus`, `joule`, `kelvin`, `m`, `mole`, `s`.

## Mathematical constants

`pi` (from `math`), plus derived `two_pi` (6.283185307179586), `half_pi`
(1.5707963267948966), `pi_sq` (9.869604401089358).

## Physical constants

| Name | Value | Meaning (base units) |
| --- | --- | --- |
| `c_light` | 299.792458 | Speed of light, mm/ns (`299792458 * m / s`); exact |
| `c_light_sq` | 89875.51787368178 | c², mm²/ns² |
| `h_Planck` | 4.135667696923859e-12 | Planck constant, MeV·ns; exact |
| `hbar` / `hbar_Planck` | 6.582119569509066e-13 | Reduced Planck constant, MeV·ns (`h_Planck / two_pi`) |
| `hbarc` | 1.973269804593025e-10 | ħc, MeV·mm |
| `hbarc_sq` | 3.893793721718595e-20 | (ħc)², MeV²·mm² |
| `k_Boltzmann` | 8.617333262145178e-11 | Boltzmann constant, MeV/K; exact |
| `Avogadro` | 6.02214076e23 | per mole; exact |
| `eplus` | 1.0 | CLHEP base charge unit (positron charge) |
| `eminus` | -1.0 | Electron charge = `-eplus` |
| `e_sq` | 1.0 | `eplus * eplus` |
| `e_SI` | 1.602176634e-19 | Elementary charge in coulombs (**SI, not CLHEP**); exact, PDG 2022 |

Exact values are commented in the source as `# exact value, taken from PDG
2022` (PDG 2020/2022 review).

## Typical use

```python
from hepunits.constants import hbarc, k_Boltzmann
from hepunits import c_light, hbar, eV, s, picosecond, micrometer, nanometer
print(k_Boltzmann * 300.0)                       # 2.5851999786435534e-08
print(hbarc)                                     # 1.973269804593025e-10
print(c_light * (1.5 * picosecond))              # 0.44968868700000003 (mm)
print(c_light * (1.5 * picosecond) / micrometer) # 449.688687
print(hbar / (eV * s))                           # 6.582119569509066e-16
print(c_light / (555.17121851852 * nanometer))   # 539999.9999999985 (≈ 540 THz)
```

The last line is the green-light check from the upstream
`tests/constants/test_constants.py` (asserted `approx(540 * THz)`).

## Notes

- `e_SI` is the one constant **not** in CLHEP units; use `eplus` (or
  `coulomb`) for CLHEP charge values.
- Every constant is a plain float: no unit metadata, no dimensional safety.
