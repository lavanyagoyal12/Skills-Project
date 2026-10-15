# hepunits prefixes reference

Purpose: commonly used SI and binary prefixes in `hepunits.units.prefixes`, with values in CLHEP base units.
Validated against: hepunits 2.4.7, numpy 2.5.4, hist 2.12.0, particle 1.0.1, pint 0.26.1 (Python 3.14.8). Values below were produced by `repr(getattr(hepunits, name))` from the installed package.

Prefixes are plain multiplicative floats, re-exported at top level. Combine with a unit: `1 * kilo * MeV` is `1000.0 * 1.0` = `1000.0` (== `GeV`).

## Common SI prefixes

| Name | Value | Name | Value |
| --- | --- | --- | --- |
| `zepto` | 1e-21 | `kilo` | 1000.0 |
| `atto` | 1e-18 | `mega` | 1000000.0 |
| `femto` | 1e-15 | `giga` | 1000000000.0 |
| `pico` | 1e-12 | `tera` | 1000000000000.0 |
| `nano` | 1e-09 | `peta` | 1000000000000000.0 |
| `micro` | 1e-06 | `exa` | 1e+18 |
| `milli` | 0.001 | `zetta` | 1e+21 |
| `centi` | 0.01 | `yotta` | 1e+24 |

The full SI range is defined down to `quecto` (1e-30) and up to `quetta` (1e+30), including `deci`, `deca`, `hecto`, `ronna`. Binary prefixes (`kibi` 1024.0, `mebi`, `gibi`, `tebi`, `pebi`, `exbi`, `zebi`, `yobi`) are also defined; `googol` = 1e+100.

## Verified identities

```python
from hepunits import kilo, mega, micro, zepto, GeV, MeV
from math import isclose
print(1 * kilo * MeV)                                  # 1000.0
print(4 * mega == 1.0 / 0.25 / micro)                  # True
# 1*zepto*GeV is 9.999999999999999e-19, not exactly 1e-18
print(isclose(1 * zepto * GeV, 1e-18, rel_tol=1e-12))  # True
```

Prefixes are floats with no dimension checking (`kilo * ns` is `1000.0`); prefer the predefined named units (`GeV`, `keV`, …) where they exist.
