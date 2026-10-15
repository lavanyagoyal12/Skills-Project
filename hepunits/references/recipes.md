# hepunits recipes

Purpose: 7 runnable, self-contained examples with their actual printed output.
Validated against: hepunits 2.4.7, numpy 2.5.4, hist 2.12.0, particle 1.0.1, pint 0.26.1 (Python 3.14.8).

## 1. Readable physics cuts (NumPy)

```python
import numpy as np
from hepunits import GeV
pt = np.array([25000.0, 500000.0, 1000000.0])              # MeV
print(pt[pt > 200.0 * GeV].tolist())                       # [500000.0, 1000000.0]
print((pt / GeV).tolist())                                 # [25.0, 500.0, 1000.0]
```

## 2. Converting between energy units

```python
from hepunits import GeV, TeV, MeV
print(25000.0 / GeV)                                       # 25.0
print(1.8 * TeV)                                           # 1800000.0
print((1.8 * TeV) / GeV)                                   # 1800.0
print((1.8 * TeV) / MeV)                                   # 1800000.0
```

## 3. ctau (decay length) from a lifetime

```python
from hepunits import c_light, picosecond, micrometer
tau = 1.5 * picosecond                                     # 0.0015 ns (base)
ctau = c_light * tau                                       # in mm
print(ctau)                                                # 0.44968868700000003
print(ctau / micrometer)                                   # 449.688687
print(ctau / 1e-3)                                         # 449.688687
```

## 4. Cross-check with `particle`

```python
from particle import Particle
from hepunits import c_light
b0 = Particle.from_name("B0")
print(b0.mass)                                             # 5279.72
print(b0.lifetime)                                         # 0.0015113937013798086
print(b0.ctau)                                             # 0.45310443274237083
print(c_light * b0.lifetime)                               # 0.45310443274237083
print(c_light * b0.lifetime == b0.ctau)                    # True
```

## 5. Mass window over a real array

```python
import numpy as np
from hepunits import MeV
from particle import Particle
m_z = Particle.from_name("Z0").mass                        # 91187.9 (MeV)
m = np.array([91000.0, 91187.9, 91300.0])                  # candidates (MeV)
selected = m[(m > m_z - 10 * MeV) & (m < m_z + 10 * MeV)]
print(m_z)                                                 # 91187.9
print(selected.tolist())                                   # [91187.9]
```

## 6. Cross-section and luminosity

```python
from hepunits import picobarn, femtobarn, pb, invfb, invpb, fb
print(40.0 * picobarn)                                     # 3.9999999999999995e-33
print((40.0 * picobarn) / femtobarn)                       # 39999.99999999999
print((40.0 * picobarn) / pb)                              # 40.0
print((10.0 * invfb) * (1.0 * pb))                         # 9999.999999999998
print(invpb * pb)                                          # 1.0
print(invfb * fb)                                          # 1.0
```

## 7. Histogram range (axis and fill in the same units)

```python
import numpy as np
import hist
from hepunits import GeV
h = hist.Hist(hist.axis.Regular(50, 0 * GeV, 1000 * GeV, name="pt"))
h.fill(pt=np.array([25000.0, 500000.0]))                   # MeV, matching the axis
print(float(h.axes[0].edges[0]))                           # 0.0
print(float(h.axes[0].edges[-1]))                          # 1000000.0
print(float(np.sum(h.values())))                           # 2.0
```

Always sanity-check with the `particle` cross-check (recipe 4) when computing decay lengths, and keep axis/fill units consistent (recipe 7).
