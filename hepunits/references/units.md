# hepunits units reference

Purpose: commonly used `hepunits` unit and prefix names with their values in
CLHEP base units (mm, ns, MeV, eplus).
Validated against: hepunits 2.4.7, numpy 2.5.4, hist 2.12.0, particle 1.0.1,
pint 0.26.1 (Python 3.14.8). Every value was produced by
`repr(getattr(hepunits, name))` from the installed package — these are the
truth, not hand-typed.

Everything can be imported from `hepunits` (top level) or `hepunits.units`;
both give the same float object. Only the commonly used names are listed: the
package also defines frequency, volume, force/pressure, electromagnetic and
radiation units — verify any other name with `hasattr(hepunits, name)` first
(prefixes and constants live in the last two sections and in
[constants.md](constants.md)).

## Base units (value 1.0)

| Quantity | Names |
| --- | --- |
| Energy | `MeV`, `megaelectronvolt` |
| Length | `mm`, `millimeter` |
| Time | `ns`, `nanosecond` |
| Charge | `eplus` (+1.0), `eminus` (−1.0) |
| Temperature | `kelvin`, `K` |
| Amount | `mole`, `mol` |
| Luminous intensity | `candela`, `cd` |
| Plane / solid angle | `radian`, `rad`, `steradian`, `sr` |

## Energy

| Name | Value | Name | Value |
| --- | --- | --- | --- |
| `eV` | 1e-06 | `keV` | 0.001 |
| `MeV` | 1.0 | `GeV` | 1000.0 |
| `TeV` | 1000000.0 | `PeV` | 1000000000.0 |
| `EeV` | 1000000000000.0 | `ZeV` | 1000000000000000.0 |

Long names (`electronvolt`, `megaelectronvolt`, `gigaelectronvolt`, …) are
aliases with identical values. `eV2` (`electronvolt2`) is 1e-12 (MeV²).

## Length

| Name | Value | Name | Value |
| --- | --- | --- | --- |
| `fermi`/`fm` | 1e-12 | `femtometer` | 1e-12 |
| `angstrom` | 1.0000000000000001e-07 | `nanometer` | 1.0000000000000002e-06 |
| `micron` | 0.001 | `micrometer` | 0.001 |
| `mm` | 1.0 | `cm` | 10.0 |
| `m` | 1000.0 | `km` | 1000000.0 |

No `nm` (use `nanometer`) and no `um` (use `micrometer`/`micron`).

## Time

| Name | Value | Name | Value |
| --- | --- | --- | --- |
| `fs`/`femtosecond` | 1.0000000000000002e-06 | `ps`/`picosecond` | 0.001 |
| `ns`/`nanosecond` | 1.0 | `us`/`microsecond` | 1000.0 |
| `ms`/`millisecond` | 1000000.0 | `s`/`second` | 1000000000.0 |
| `minute` | 60000000000.0 | `hour`/`h` | 3600000000000.0 |
| `day`/`d` | 86400000000000.0 | `year`/`y` | 3.15576e+16 |

`year = 365.25 * day` (not a calendar year). `zs` (`zeptosecond`) and `ys`
(`yoctosecond`) exist; **no `as`** (reserved word — use `attosecond`). Sub-SI
symbols use `u`, not `µ`: `us`, `uA`, `uF`, `uGy`, `uCi`, `uWb`.

## Angle

| Name | Value | Name | Value |
| --- | --- | --- | --- |
| `rad`/`radian` | 1.0 | `sr`/`steradian` | 1.0 |
| `mrad`/`milliradian` | 0.001 | `deg`/`degree` | 0.017453292519943295 |

Verified: `360 * degree == two_pi * radian` → `True`.

## Area & volume

| Name | Value | Name | Value |
| --- | --- | --- | --- |
| `mm2`/`millimeter2` | 1.0 | `cm2`/`centimeter2` | 100.0 |
| `m2`/`meter2` | 1000000.0 | `km2`/`kilometer2` | 1000000000000.0 |
| `fm2` | 1e-24 | `m3`/`meter3` | 1000000000.0 |
| `cm3` | 1000.0 | `fm3` | 1e-36 |

## Cross section

| Name | Value | Name | Value |
| --- | --- | --- | --- |
| `barn` | 9.999999999999999e-23 | `mb`/`millibarn` | 9.999999999999999e-26 |
| `ub`/`microbarn` | 9.999999999999999e-29 | `nb`/`nanobarn` | 1e-31 |
| `pb`/`picobarn` | 1e-34 | `fb`/`femtobarn` | 1e-37 |
| `ab`/`attobarn` | 1e-40 | | |

`barn = 1e-28 * meter2` (so `barn == 100 * fm2` → `True`). `40.0 * picobarn`
prints `3.9999999999999995e-33` because of float rounding (exact value 4e-33).

## Luminosity (inverse cross-section)

| Name | Value | Name | Value |
| --- | --- | --- | --- |
| `invmb` | 1e+25 | `invub` | 1.0000000000000002e+28 |
| `invnb` | 1e+31 | `invpb` | 1.0000000000000001e+34 |
| `invfb` | 1e+37 | `invab` | 1e+40 |

Verified: `invpb * pb == 1.0`, `invfb * fb == 1.0`.

## Mass

| Name | Value | Name | Value |
| --- | --- | --- | --- |
| `g`/`gram` | 6.241509074460762e+21 | `kg`/`kilogram` | 6.241509074460762e+24 |

Mass is stored as energy: 1 kg = 6.241509074460762e+24 MeV.

## Other commonly seen units

| Name | Value (base) | Meaning |
| --- | --- | --- |
| `Hz`/`hertz` | 1e-09 | 1/ns; `GHz` = 1.0, `MHz` = 0.001, `THz` = 1000.0000000000001 |
| `J`/`joule` | 6241509074460.763 | MeV; `kJ` = 6241509074460763.0 |
| `W`/`watt` | 6241.509074460762 | joule/second |
| `Pa`/`pascal` | 6241.509074460763 | newton/m²; `bar` = 624150907.4460763, `atm` = 632420906.9697368 |
| `T`/`tesla` | 0.001 | `G`/`Gs`/`gauss` = 1.0000000000000001e-07, `kG` = 0.0001 |
| `C`/`coulomb` | 6.241509074460763e+18 | `A`/`ampere` = 6241509074.460763, `V`/`volt` = 1e-06 |
| `Gy`/`gray`, `Sv`/`sievert` | 1.0000000000000002e-12 | both joule/kilogram — numerically equal, different concepts |
| `Bq`/`becquerel` | 1e-09 | `Ci`/`curie` = 37.0 |

Ambiguity warnings: `A` is **ampere** (not area), `G`/`Gs` is **gauss** (tesla
is `T`), `C` is coulomb. Many more names exist (`F`, `H`, `Wb`, `Mx`, `ohm`,
`S`, `N`, `dyne`, `erg`, `lm`, `lx`, `MGy`, …) — verify with `hasattr` before use.

## Prefixes

Defined in `hepunits.units.prefixes` and re-exported everywhere
(`from hepunits import kilo, mega, kibi`). Prefixes are plain multiplicative
floats: `1 * kilo * MeV` is `1000.0 * 1.0` = `1000.0` (== `GeV`).

### SI prefixes (powers of 10)

| Name | Value | Name | Value | Name | Value |
| --- | --- | --- | --- | --- | --- |
| `quecto` | 1e-30 | `ronto` | 1e-27 | `zepto` | 1e-21 |
| `atto` | 1e-18 | `femto` | 1e-15 | `pico` | 1e-12 |
| `nano` | 1e-09 | `micro` | 1e-06 | `milli` | 0.001 |
| `centi` | 0.01 | `deci` | 0.1 | `deca` | 10.0 |
| `hecto` | 100.0 | `kilo` | 1000.0 | `mega` | 1000000.0 |
| `giga` | 1000000000.0 | `tera` | 1000000000000.0 | `peta` | 1000000000000000.0 |
| `exa` | 1e+18 | `zetta` | 1e+21 | `yotta` | 1e+24 |
| `ronna` | 1e+27 | `quetta` | 1e+30 | | |

### Binary prefixes (powers of 2) and misc

`kibi` 1024.0, `mebi` 1048576.0, `gibi` 1073741824.0, `tebi` 1099511627776.0,
`pebi` 1125899906842624.0, `exbi` 1.152921504606847e+18, `zebi`
1.1805916207174113e+21, `yobi` 1.2089258196146292e+24; `googol` = 1e+100.

**Don't mix the families:** `mega = 1e6` but `mebi = 2**20`. Prefer the
predefined named units (`GeV`, `keV`, `GHz`, …) where they exist.

## Verified identities

```python
from hepunits import kilo, mega, giga, eV, MeV, GeV, Hz, GHz, invpb, pb, degree, two_pi, radian
print(1 * kilo * MeV)                                  # 1000.0
print(1 * kilo * MeV == GeV)                           # True
print(1 * mega * eV == MeV)                            # True
print(1 * giga * Hz == GHz)                            # True
print(invpb * pb)                                      # 1.0
print(360 * degree == two_pi * radian)                 # True
```

Float caveat: `1 * zepto * GeV` is `9.999999999999999e-19`, not exactly 1e-18 —
use `math.isclose(v, 1e-18, rel_tol=1e-12)` for such checks.

## Aliases

Short and long names are interchangeable: `eV`/`electronvolt`,
`MeV`/`megaelectronvolt`, `GeV`/`gigaelectronvolt`, `fm`/`fermi`,
`mm`/`millimeter`, `cm`/`centimeter`, `m`/`meter`, `km`/`kilometer`,
`ps`/`picosecond`, `ns`/`nanosecond`, `us`/`microsecond`, `ms`/`millisecond`,
`s`/`second`, `h`/`hour`, `d`/`day`, `y`/`year`, `rad`/`radian`,
`deg`/`degree`, `m2`/`meter2`, `m3`/`meter3`, `pb`/`picobarn`, `fb`/`femtobarn`,
`mol`/`mole`, `K`/`kelvin`, `micron`/`micrometer`.
