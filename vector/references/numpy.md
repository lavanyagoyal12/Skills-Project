# numpy backend

`vector.obj` builds one scalar vector; `vector.array` builds a NumPy
structured-array collection for fixed-shape data (vectorized, but not suitable
for jagged/variable-length structure).

## Scalars (`vector.obj`)

```python
v = vector.obj(pt=30.0, phi=0.5, eta=1.2, mass=0.105)  # single muon
print(v.px, v.py, v.pz, v.energy)
```

Lists of `vector.obj` are slow in plain Python — use `vector.array` or Numba.

## Fixed-shape arrays (`vector.array`)

```python
import numpy as np

muons = vector.array({
    "pt": np.array([30.0, 45.0]),
    "phi": np.array([0.5, -1.2]),
    "eta": np.array([1.2, -0.8]),
    "mass": np.full(2, 0.105),
})
print(muons.px, muons.energy)  # vectorized over the whole array
```

## Coordinate conversion (numerical precision)

Any coordinate is readable regardless of construction; `.to_*()` switches
storage to avoid repeated trig:

```python
v = vector.obj(px=3.0, py=4.0, pz=0.0, energy=5.0)
v.pt                    # computed on the fly, no copy
v.to_rhophithetatau()   # switch storage to rho/phi/theta/tau
```

## Numba

`@nb.njit` compiles `vector.obj` fine, but NumPy-backed `vector.array` support
inside Numba is incomplete (upstream issue #43). There is no performance
advantage (likely a disadvantage) compiling a calculation on just a few
vectors.
