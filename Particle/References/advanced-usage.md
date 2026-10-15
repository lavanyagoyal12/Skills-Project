# Particle Advanced Usage Patterns — Deep Reference

Read this reference when filtering the PDG table by quantum numbers or name,
working with antiparticles, using standalone `PDGID` queries, converting
generator-specific IDs (Geant3, Pythia, Corsika7) to PDG IDs, looking up decay
modes, classifying MC truth particles, or printing a full property summary.
Verified against `particle` 1.0.1.

## Table of Contents

- [Search / filter](#search--filter)
- [Antiparticles](#antiparticles)
- [PDGID standalone](#pdgid-standalone-fast-no-full-table-load)
- [Particle and PDGID literals](#particle-and-pdgid-literals)
- [MC generator ID converters](#mc-generator-id-converters)
- [Decay modes](#decay-modes)
- [Use with generator truth](#use-with-generator-truth-eg-from-pyhepmc)
- [Describe a particle](#describe-a-particle)

## Search / filter

```python
from particle import Particle

# All neutral beauty hadrons (particle=True excludes antiparticles)
Particle.findall(lambda p: p.pdgid.has_bottom and p.charge == 0, particle=True)

# By exact PDG name
Particle.findall(pdg_name="pi")           # pi0, pi+, pi-

# By quantum numbers
Particle.findall(J=1, P=-1)               # spin-1, negative parity (vectors)

# A plain string is a FUZZY SUBSTRING match on the name, not a glob:
Particle.findall("K*")                    # names containing the literal "K*"
Particle.findall("K[+-]")                 # [] -- brackets are not a pattern

# For real patterns, use fnmatch inside a callable
import fnmatch
Particle.findall(lambda p: fnmatch.fnmatch(p.name, "K[+-]"))   # K+, K-

# Strange mesons with c*tau > 1 m -- use hepunits for unit safety
from hepunits import meter
Particle.findall(
    lambda p: p.pdgid.is_meson and p.pdgid.has_strange and p.ctau > 1 * meter,
    particle=True,
)
# -> [K(L)0, K+]

# Stable particles: there is no is_stable, test the lifetime
Particle.findall(lambda p: p.lifetime == float("inf"), particle=True)

# finditer is lazy -- prefer it for large scans
for p in Particle.finditer(lambda p: p.pdgid.has_charm):
    print(p.name, p.mass)
```

`hepunits` is a separate package: `pip install hepunits` if it is missing.

## Antiparticles

```python
from particle import Particle

pi_plus  = Particle.from_name("pi+")
pi_minus = pi_plus.invert()
print(pi_minus.pdgid)                                # -211
print(pi_plus.is_self_conjugate)                     # False
print(Particle.from_name("pi0").is_self_conjugate)   # True
```

## PDGID standalone (fast, no full table load)

```python
from particle import PDGID

pid = PDGID(211)
print(pid.is_meson, pid.has_strange)  # True False
print(pid.is_valid)                   # True

bad = PDGID(99999999)
print(bad.is_valid)                   # False (malformed ID)
# NOTE: PDGID(99).is_valid and PDGID(9999999).is_valid are True, but
# Particle.from_pdgid() raises ParticleNotFound for them.

# Standalone functions mirror PDGID properties
from particle.pdgid import is_meson, has_bottom
print(is_meson(211))                  # True
print(has_bottom(5122))               # True (Lambda_b)
```

## Particle and PDGID literals

```python
from particle import literals as lp
print(lp.pi_plus)                     # pi+
print(lp.Lambda_b_0.J)                # 1/2  (a Fraction, not 0.5)

from particle.pdgid import literals as lid
print(lid.pi_plus)                    # <PDGID: 211>
print(lid.Lambda_b_0.has_bottom)      # True
```

## MC generator ID converters

```python
from particle import Particle, PDGID, Geant3ID, PythiaID, Corsika7ID

# Geant3 -> PDG
print(Particle.from_pdgid(Geant3ID(8).to_pdgid()).name)      # pi+

# Pythia -> PDG
print(PythiaID(10221).to_pdgid())                            # <PDGID: 9010221>

# Corsika7 -> PDG
print(Particle.from_pdgid(Corsika7ID(5).to_pdgid()).name)    # mu+

# PDG -> generator ID
print(PythiaID.from_pdgid(PDGID(9010221)))                   # <PythiaID: 10221>
print(Geant3ID.from_pdgid(PDGID(211)))                       # <Geant3ID: 8>
print(Corsika7ID.from_pdgid(PDGID(-13)))                     # <Corsika7ID: 5>
```

`Pythia2PDGIDBiMap` still works but is deprecated; use `PythiaID.from_pdgid()`
and `.to_pdgid()`.

## Decay modes

`Particle` objects carry PDG identity/mass/width data, not branching-fraction
decay tables — there is no `decay_modes` attribute. For decay-chain data (modes,
branching fractions, `.dec` files), use the separate `decaylanguage` package
(`pip install decaylanguage`). Illustrative, not run here:

```python
from decaylanguage import DecFileParser

parser = DecFileParser.from_file("DECAY.DEC")
parser.parse()
print(parser.list_decay_modes("B0"))
```

## Use with generator truth (e.g. from pyhepmc)

```python
from particle import PDGID

def classify_truth_particle(pdgid: int) -> str:
    pid = PDGID(pdgid)
    if not pid.is_valid:
        return "invalid"
    if pid.is_lepton:
        return "lepton"
    if pid.has_bottom:
        return "b_hadron"
    if pid.has_charm:
        return "c_hadron"
    return "other"

print(classify_truth_particle(11), classify_truth_particle(5122),
      classify_truth_particle(421), classify_truth_particle(99999999))
# lepton b_hadron c_hadron invalid
```

## Describe a particle

```python
from particle import Particle

print(Particle.from_pdgid(321).describe())
# Name: K+   ID: 321   Latex: $K^{+}$
# Mass  = 493.677 ± 0.015 MeV
# Lifetime = 12.379 ± 0.021 ns
# Q (charge) = +   J (total angular) = 0   P (space parity) = -
# C (charge parity) = None   I (isospin) = 1/2   G (G-parity) = None
#     SpinType: -1
#     Quarks: uS
#     Antiparticle name: K- (antiparticle status: ChargeInv)
```
