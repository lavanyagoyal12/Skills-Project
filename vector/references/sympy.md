# sympy backend

`VectorSympy2D/3D/4D` and `MomentumSympy2D/3D/4D` produce fully symbolic
coordinate transforms — useful for deriving kinematic formulas and generating
Fortran/C/LaTeX code:

```python
import sympy

# Symbols must be declared real=True; complex assumptions break trig simplification
x, y, z, t = sympy.symbols("x y z t", real=True)

v = vector.VectorSympy4D(x=x, y=y, z=z, t=t)
v.rho            # sqrt(x**2 + y**2)
v.is_timelike()  # t**2 - x**2 - y**2 - z**2 > 0

expr = v.boost(v.to_beta3()).t
expr.simplify()
expr.subs({x: 3, y: 2, z: 1, t: 10})

# Fortran / C / LaTeX for downstream use
import sympy.printing.fortran
sympy.printing.fortran.fcode(expr.simplify())
```

## Gotchas

- Sign conventions differ for space-like / negative time-like vectors: the
  SymPy backend avoids piecewise if-then branches, unlike other backends (which
  follow ROOT). For physical momentum vectors (positive time-like), all backends
  agree.
- All operations return sympy expressions, not floats.
