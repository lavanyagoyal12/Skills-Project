# pytree (optree)

Flatten/unflatten vector objects and nested structures containing them, for
integrations expecting a 1D numeric array (e.g. `scipy.integrate`, optimizers).

- Requires `optree`, which is **not** a default vector dependency;
  `register_pytree()` raises `ImportError` without it.
- Call once at module level before any flatten/unflatten use.

```python
# pip install optree
pytree = vector.register_pytree()  # returns flatten/unflatten interface

state = {
    "position": vector.obj(x=1.0, y=2.0, z=3.0, t=0.0),
    "momentum": vector.obj(x=0.0, y=10.0, z=0.0, t=14.0),
}
flat, treedef = pytree.flatten(state)   # plain list of 8 scalars
reconstructed = pytree.unflatten(treedef, flat)  # round-trips exactly

# Typical use: wrap scipy.integrate.solve_ivp so it sees a 1D array
def wrapped_solve(fun, t_span, y0, t_eval):
    flat_y0, treedef = pytree.flatten(y0)

    def flat_fun(t, flat_y):
        flat_dstate_dt, _ = pytree.flatten(fun(t, pytree.unflatten(treedef, flat_y)))
        return flat_dstate_dt

    from scipy.integrate import solve_ivp
    sol = solve_ivp(flat_fun, t_span, flat_y0, t_eval=t_eval)
    return pytree.unflatten(treedef, sol.y)
```
