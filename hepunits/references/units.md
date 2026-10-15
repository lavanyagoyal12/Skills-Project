# hepunits units reference

Purpose: commonly used `hepunits` unit names with their values in CLHEP base units.
Validated against: hepunits 2.4.7, numpy 2.5.4, hist 2.12.0, particle 1.0.1, pint 0.26.1 (Python 3.14.8). Every value below was produced by `repr(getattr(hepunits, name))` from the installed package — these are the truth, not hand-typed.

Everything can be imported from `hepunits` (top level) or `hepunits.units`; both give the same float. Only the commonly used names are listed here: the package also defines frequency, volume, force/pressure, electromagnetic and radiation units — verify any name with `hasattr(hepunits, name)` before using it (check `references/prefixes.md`/`references/constants.md` for prefixes and constants too).

## Base units (value 1.0)

| Quantity | Unit names |
| --- | --- |
| Energy | `MeV`, `megaelectronvolt` |
| Length | `mm`, `millimeter` |
| Time | `ns`, `nanosecond` |
| Charge | `eplus` (+1.0), `eminus` (−1.0) |
| Temperature | `kelvin`, `K` |

(Radian, steradian, mole and candela are also base units; see `references/constants.md`.)

## Energy

| Name | Value | Name | Value |
| --- | --- | --- | --- |
| `eV` | 1e-06 | `keV` | 0.001 |
| `MeV` | 1.0 | `GeV` | 1000.0 |
| `TeV` | 1000000.0 | `PeV` | 1000000000.0 |

Long names (`electronvolt`, `megaelectronvolt`, `gigaelectronvolt`, …) are aliases with identical values; `eV2` is 1e-12 (MeV²).

## Length

| Name | Value | Name | Value |
| --- | --- | --- | --- |
| `fermi` | 1e-12 | `fm` | 1e-12 |
| `nanometer` | 1.0000000000000002e-06 | `micrometer` | 0.001 |
| `micron` | 0.001 | `mm` | 1.0 |
| `cm` | 10.0 | `m` | 1000.0 |
| `km` | 1000000.0 | | |

## Time

| Name | Value | Name | Value |
| --- | --- | --- | --- |
| `fs` | 1.0000000000000002e-06 | `femtosecond` | 1.0000000000000002e-06 |
| `ps` | 0.001 | `picosecond` | 0.001 |
| `ns` | 1.0 | `nanosecond` | 1.0 |
| `us` | 1000.0 | `microsecond` | 1000.0 |
| `ms` | 1000000.0 | `millisecond` | 1000000.0 |
| `s` | 1000000000.0 | `second` | 1000000000.0 |

## Angle

| Name | Value | Name | Value |
| --- | --- | --- | --- |
| `rad` | 1.0 | `radian` | 1.0 |
| `mrad` | 0.001 | `milliradian` | 0.001 |
| `deg` | 0.017453292519943295 | `degree` | 0.017453292519943295 |
| `sr` | 1.0 | `steradian` | 1.0 |

Check (verified): `360 * degree == two_pi * radian` → `True`.

## Area

| Name | Value | Name | Value |
| --- | --- | --- | --- |
| `mm2` | 1.0 | `centimeter2` | 100.0 |
| `m2` | 1000000.0 | `km2` | 1000000000000.0 |
| `fm2` | 1e-24 | | |

## Cross section

| Name | Value | Name | Value |
| --- | --- | --- | --- |
| `barn` | 9.999999999999999e-23 | `mb` | 9.999999999999999e-26 |
| `millibarn` | 9.999999999999999e-26 | `ub` | 9.999999999999999e-29 |
| `microbarn` | 9.999999999999999e-29 | `nb` | 1e-31 |
| `nanobarn` | 1e-31 | `pb` | 1e-34 |
| `picobarn` | 1e-34 | `fb` | 1e-37 |
| `femtobarn` | 1e-37 | `ab` | 1e-40 |
| `attobarn` | 1e-40 | | |

`40.0 * picobarn` prints `3.9999999999999995e-33` because of float rounding (the exact value is 4e-33).

## Luminosity (inverse cross-section)

| Name | Value | Name | Value |
| --- | --- | --- | --- |
| `invub` | 1.0000000000000002e+28 | `invmb` | 1e+25 |
| `invnb` | 1e+31 | `invpb` | 1.0000000000000001e+34 |
| `invfb` | 1e+37 | `invab` | 1e+40 |

Verified identities: `invpb * pb == 1.0`, `invfb * fb == 1.0`.

## Mass

| Name | Value | Name | Value |
| --- | --- | --- | --- |
| `g` | 6.241509074460762e+21 | `gram` | 6.241509074460762e+21 |
| `kg` | 6.241509074460762e+24 | `kilogram` | 6.241509074460762e+24 |

(Mass is stored as energy: 1 kg = 6.241509074460762e+24 MeV.)

## Charge & SI base

| Name | Value | Name | Value |
| --- | --- | --- | --- |
| `eplus` | 1.0 | `eminus` | -1.0 |
| `mol` | 1.0 | `mole` | 1.0 |
| `K` | 1.0 | `kelvin` | 1.0 |

## Aliases

Short and long names are interchangeable: `eV`/`electronvolt`, `MeV`/`megaelectronvolt`, `GeV`/`gigaelectronvolt`, `fm`/`fermi`, `mm`/`millimeter`, `cm`/`centimeter`, `m`/`meter`, `km`/`kilometer`, `ps`/`picosecond`, `ns`/`nanosecond`, `us`/`microsecond`, `ms`/`millisecond`, `s`/`second`, `h`/`hour`, `d`/`day`, `y`/`year`, `rad`/`radian`, `deg`/`degree`, `m2`/`meter2`, `m3`/`meter3`, `pb`/`picobarn`, `fb`/`femtobarn`, `mol`/`mole`, `K`/`kelvin`.

Not defined: `um` (use `micrometer`/`micron`), bare `b` for barn (use `barn`), `pd`.
