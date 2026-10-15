# Awkward — Backends, behaviors, vector, Numba, JAX

Read this when you need to know which backend an array lives on, register behaviors (e.g. vector's physics methods), or JIT-compile a loop with Numba / run on JAX.

Verified against `awkward` 2.14.0. Function signatures: [`api.md`](api.md). Performance guidance: [`advanced-patterns.md`](advanced-patterns.md).

## Backends & dispatch

Every `ak.Array` has a backend: `"cpu"` (default), `"cuda"`, `"jax"`, `"typetracer"`. All `ak.*` operations dispatch through the array's backend; `ak.to_backend(a, "cpu")` moves it.

```python
import awkward as ak
a = ak.Array([1, 2, 3])
print(ak.backend(a))        # cpu
print(ak.to_list(ak.to_backend(a, "cpu")))   # [1, 2, 3]
```

- **CuPy / CUDA**: arrays live on GPU (`ak.to_cupy` / `ak.from_cupy`, kernels run on GPU); same code, different backend.
- **Typtracer**: shape-only arrays with unknown lengths, used by lazy engines (dask-awkward) to build graphs; `ak.materialize(array)` replaces placeholder buffers with real ones. `ak.backend(a)` reports `"typetracer"`.
- **Dispatch**: every `ak.*` operation is defined with the `@high_level_function` decorator
  (`awkward._dispatch`), which yields array arguments so third-party libraries (e.g.
  dask-awkward) can overload operations for their own array types.

## Behaviors (`ak.behavior`)

Behaviors attach Python mixin methods to data by *name*: the global `ak.behavior` dict maps record names (and `"__array__"`) to classes. `ak.mixin_class(registry, name)` / `ak.mixin_class_method(ufunc, rhs=None, transpose=True)` define methods and ufunc overloads for a name. Use `ak.with_name(a, "Momentum4D")` (or `ak.Array(..., behavior=...)` to scope without global state) to tag data.

### vector registration (physics methods)

```python
import awkward as ak
import vector
vector.register_awkward()                    # add vector behaviors to the registry
jets = vector.zip({"pt": ak.Array([[10.0, 20.0, 30.0], [5.0]]),
                   "eta": ak.Array([[1.0, 2.0, 3.0], [1.0]]),
                   "phi": ak.Array([[0.1, 0.2, 0.3], [0.0]]),
                   "mass": ak.Array([[1.0, 2.0, 3.0], [1.0]])})
print(str(ak.type(jets)))   # 2 * var * Momentum4D[rho: float64, phi: float64, eta: float64, tau: float64]
print(ak.to_list((jets + jets).mass))   # [[2.0000000000000284, ...], [2.000000000000007]]  (plus .deltaR(), .pt, ...)
```

Notes: `vector.zip` adds the `Momentum4D` name automatically; records from plain `ak.Array`/`ak.zip` are **unnamed**, so `j1 + j2` raises `ValueError: cannot broadcast nested records` — use `ak.zip({...}, with_name="Momentum4D")` or `vector.zip`. `ak.merge_union_of_records` / `ak.merge_option_of_records` combine records of different names/shapes.

## Numba

`@numba.njit` functions accept `ak.Array`, records, and `None`; iterate lists/records with plain Python loops. Good for the "matching" (best-match) problem and irreducible loops; JIT has a cold-start cost but can beat multiple array passes.

```python
import awkward as ak
import numpy as np
import numba as nb
array = ak.Array([[{"x": 1.1, "y": [1]}, {"x": 2.2, "y": [1, 2]}], [], [{"x": 4.4, "y": [1, 2, 3, 4]}]])
@nb.jit
def sum_of_y(array):
    out = np.zeros(len(array), dtype=np.int64)
    for i, list_of_records in enumerate(array):
        for record in list_of_records:
            for y in record.y:
                out[i] += y
    return out
print(ak.to_list(ak.Array(sum_of_y(array))))   # [4, 0, 10]
```

`ak.numba.register_and_check(module)` validates that awkward's Numba support is available. Keep per-element work in explicit loops — scalar-ufunc expressions on whole `ak.Array`s inside `njit` may not type-check; drive it with indexing instead.

## JAX

JAX arrays are immutable; awkward can wrap them (`ak.to_jax(a)` / `ak.from_jax(ja)`) and `to_backend(a, "jax")` switches backends. Requirements: enable 64-bit (`jax.config.update("jax_enable_x64", True)`), call `ak.jax.import_jax()` once, and use `ak.jax.register_and_check()` to validate; `ak.jax.register_behavior_class(behavior)` registers a JAX-compatible behavior mixin.

```python
# illustrative — not run here (JAX backend deprecated in awkward 2.14; requires x64 + registration)
import awkward as ak, jax
jax.config.update("jax_enable_x64", True)
ak.jax.import_jax(); ak.jax.register_and_check()
ja = ak.to_jax(ak.Array([[1, 2], [3]]))       # jax.Array backed
back = ak.from_jax(ja)
print(ak.to_list(back))                        # [[1, 2], [3]]
```

The JAX backend is deprecated (awkward emits a `DeprecationWarning`) — plan to migrate.

## Related

- [`api.md`](api.md) — signatures for `ak.backend`, `ak.to_backend`, `ak.numba.*`, `ak.jax.*`, `ak.to_cupy`, `ak.materialize`, `ak.with_name`.
- [`types-layouts.md`](types-layouts.md) — layouts & kernels; [`advanced-patterns.md`](advanced-patterns.md) — when to reach for these (performance).
