# hepunits constants reference

Purpose: key physical and mathematical constants in `hepunits.constants`, with meaning and CLHEP base units.
Validated against: hepunits 2.4.7, numpy 2.5.4, hist 2.12.0, particle 1.0.1, pint 0.26.1 (Python 3.14.8). Values below were produced by `repr(getattr(hepunits, name))` from the installed package.

All values are in the CLHEP base system (MeV, mm, ns): `c_light` is in mm/ns, not m/s.

## Physical constants

| Name | Value | Meaning (base units) |
| --- | --- | --- |
| `c_light` | 299.792458 | Speed of light, mm/ns |
| `h_Planck` | 4.135667696923859e-12 | Planck constant, MeV·ns |
| `hbar` | 6.582119569509066e-13 | Reduced Planck constant, MeV·ns |
| `hbarc` | 1.973269804593025e-10 | ħc, MeV·mm |
| `k_Boltzmann` | 8.617333262145178e-11 | Boltzmann constant, MeV/K |
| `e_SI` | 1.602176634e-19 | Elementary charge in coulombs (SI, not CLHEP) |
| `eplus` | 1.0 | CLHEP base charge unit |
| `eminus` | -1.0 | Negative unit charge |

Math: `pi` = 3.141592653589793, `two_pi` = 6.283185307179586 (used by the `units.md` angle check).

## Typical use

```python
from hepunits.constants import hbarc, k_Boltzmann
print(hbarc)                                 # 1.973269804593025e-10
print(k_Boltzmann * 300.0)                   # 2.5851999786435534e-08
```

## Notes

- `e_SI` is the one constant **not** in CLHEP units; use `coulomb`/`eplus` for CLHEP charge values.
- Every constant is a plain float: no unit metadata, no dimensional safety.
