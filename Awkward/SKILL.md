---
name: awkward
description: >-
  Use when working with nested, variable-length (jagged/ragged) data in
  Python: per-event particle collections, records, tuples, missing (None)
  values; vectorized math, combinatorics, broadcasting; conversion
  (Arrow/Parquet/JSON/NumPy/ML), Numba/JAX/CUDA acceleration.
---
# Awkward Array
NumPy-like arrays for **nested, variable-sized data** (jagged lists, records,
mixed types, missing data); compiled, fast. Verified against `awkward` **2.14.0**.
```python
import awkward as ak, numpy as np
a = ak.Array([{"x": 1.1, "y": [1, 2]}, {"x": 2.2, "y": []}, {"x": None, "y": [3, 4, 5]}])
a["y"], a["y", ..., 1:]        # -> [[1,2],[] ,[3,4,5]] / [[2],[],[4,5]]   ... = innermost
np.sqrt(a["x"]); ak.sum(a["y"], axis=1)  # ufuncs work, None propagates; -> [3, 0, 12]
```
## When to use
- Per-event variable-length lists (jets, tracks) with records/`None` — data NumPy can't hold rectangularly.
- Vectorized cuts, invariant masses, pair combinations, reductions without loops. Full `ak.*` index: `references/api.md`.
## ArrayBuilder
```python
# events = [{"jets": [{"pt": 10.0, "eta": 1.0}, ...]}, ...]   one dict per event
b = ak.ArrayBuilder()                        # NOT a context manager
for event in events:
    with b.list():                           # one event
        for jet in event["jets"]:
            with b.record():                 # one jet
                b.field("pt").real(jet["pt"]); b.field("eta").real(jet["eta"])
array = b.snapshot()                         # ak.Array (not .build())
```
Methods: `.append(x)`, `.real()/.integer()/.boolean()/.null()/.string()` (there is
**no `.none()`**), `.list()/.record()/.tuple()`, `.field(name)`, `.extend(iterable)`;
call `ak.to_packed(array)` after building.
## Indexing
```python
# arr = ak.Array([[1, 2, 3], [], [4, 5]]); rec = ak.Array([{"x": 1, "y": 2}, {"x": 3, "y": 4}])
arr[0]; arr[1:]; arr[:, 1:]                 # -> [1,2,3] / [[],[4,5]] / [[2,3],[],[5]]
arr[arr > 2]; arr[[0, 2]]; arr[ak.Array([2, 0])]   # boolean cut; int lists; ak.Array indices
rec["x"]; rec[..., "x"]; rec[["x", "y"]]           # field; at any depth; multi-field record
```
`...`/`:`/ranges/negatives/booleans/integer lists work; boolean getitem
**removes** (cut) — use `ak.mask` to keep shape.
## Reducers & math
```python
ak.sum / ak.prod / ak.mean / ak.var / ak.std / ak.min / ak.max / ak.argmax / ak.argmin  # axis, keepdims=False, mask_identity
```
- NumPy ufuncs work directly (no `ak.sqrt`); reducers drop one axis unless `keepdims=True` (default **`False`**).
- **`mask_identity=True`** (default min/max/argmax/argmin) → **`None`** for empty/all-`None` inputs; `False` (default sum/prod) → identity (`sum`→0, `prod`→1, `max`→-inf, `min`→+inf).
- Per-event scalar (`# jets = per-event records with .pt`): `ak.broadcast_arrays(ak.max(jets.pt, axis=1, keepdims=True), jets.pt)[0]`
## Combinatorics
```python
# jets = per-event list of {pt, eta, phi, mass} records; electrons = per-event scalars
ak.combinations(jets, 2, axis=1, *, replacement=False)  # all pairs (n choose k)
j1, j2 = ak.unzip(ak.combinations(jets, 2, axis=1))     # (j1+j2).mass needs vector
ak.cartesian([electrons, jets], axis=1)                 # tuples (cross product)
ak.cartesian({"e": electrons, "j": jets}, axis=1)       # records with fields e, j
```
`ak.cartesian` dict → named fields, list → tuples; `replacement=True` allows self-pairs; `ak.argcombinations`/`ak.argcartesian` = indices only.
## Missing data
```python
a = ak.Array([[1.0, 2.0], None, [3.0, None, 4.0]])
ak.is_none(a, axis=0)          # -> [False, True, False]
ak.mask(a, a > 1)              # -> [[None, 2.0], None, [3.0, None, 4.0]]  (keeps shape)
ak.fill_none(a, -1)            # default axis=-1: outer None stays -> [[1., 2.], None, [3., -1., 4.]]
ak.fill_none(a, -1, axis=None) # all levels        -> [[1., 2.], -1, [3., -1., 4.]]
ak.firsts(a, axis=1)           # -> [1.0, None, 3.0]
ak.pad_none(a, 3, axis=1)      # -> [[1., 2., None], None, [3., None, 4.]]
ak.drop_none(a)                # -> [[1., 2.], [3., 4.]]
```
`ak.mask` keeps shape with `None`; independent cuts (`x[x>3]`, `y[y>3]`) **lose alignment** — derive both from one boolean; `None` propagates (`ak.fill_none` before NumPy, `ak.drop_none` before plotting).
## Common patterns
```python
import awkward as ak, numpy as np
events = ak.Array([
    {"jets": [{"pt": 30., "eta": 0.1}, {"pt": 10., "eta": 2.0}, {"pt": 45., "eta": -0.5}]},
    {"jets": []},
])
jets = events["jets"]
sel = (jets.pt > 15) & (abs(jets.eta) < 1.5)         # object cut: & | ~, never and/or
jets[sel]; events[ak.num(jets, axis=1) > 1]          # cut objects / cut events
ak.sum(sel, axis=1)                                  # passing per event -> [2, 0]
srt = jets[ak.argsort(jets.pt, axis=1, ascending=False)]   # sort by pt, leading first
lead = ak.firsts(srt, axis=1)                        # leading jet; None if empty -> pt [45.0, None]
jets = ak.with_field(jets, ak.fill_none(ak.max(jets.pt, axis=1), 0.0), "max_pt")   # per-event scalar on each jet; keeps [] for empty events
x = ak.Array([[1, 2, 3], [4, 5]])
ak.unflatten(np.sqrt(ak.flatten(x, axis=1)), ak.num(x), axis=0)   # apply f flat, restore shape
```
## Fields & metadata
- `ak.zip` gives **unnamed** records; `with_name="Momentum4D"` (or `vector.zip`) + `vector.register_awkward()` → `.mass`/`.deltaR()`; also `ak.with_name`, `ak.with_field/without_field`, `ak.fields`.
- Strings: `ak.str.find_substring/split_pattern/split_whitespace/count_substring/...` (full list in `references/api.md`). `ak.to_numpy(a)`: regular only, options → `np.ma.MaskedArray`.
## Gotchas
- `len(arr)` is the number of events; `ak.num(arr, axis=1)` gives per-event counts.
- Use `&`/`|`/`~` for boolean arrays — `and`/`or` don't work.
- Arrays are **immutable**: item assignment (`arr[0] = x`, `arr[mask] = x`) raises `TypeError`; only field assignment (`arr["name"] = ...`) works.
- `ak.mean` of an empty list is `nan`; `ak.sum` is `0` (identity).
- Do not loop over events in Python — use reductions/combinatorics (Numba for irreducible loops).
- Read `ak.type` left-to-right: `events * var * {pt: float64, ...}` (length * variable list * fields).
- `ak.to_numpy` raises on ragged arrays — flatten or `ak.to_list`; axes: `0` outermost, `1` event, `-1` innermost (`num/flatten/pad_none` default `axis=1`, reducers `axis=None`).
- **No `ak.abs`/`ak.take`/`ak.expand_dims`** — use `abs()`, `arr[ak.Array([...])]`, or `arr[:, np.newaxis]`.
- `ak.from_json(str)` parses contents, not a path: `ak.from_json(pathlib.Path("f").read_text())`.
- `arr[:, 0]` **raises on empty events** — use `ak.firsts(arr)`.
- A per-event `None` (from `ak.max`/`ak.firsts` on an empty event) **turns the whole event into `None`** when attached to its objects via `ak.with_field`/`ak.zip` — `ak.fill_none` it first.
- `ak.ArrayBuilder` → `snapshot()` (not `build()`), **not** a context manager; `ak.combinations` uses `replacement` (not `replace`); `ak.mask` keyword is `valid_when`.
- `ak.flatten(axis=0)` removes only top-level `None`; `ak.Array(json_str)` parses JSON (wrap text in a list); no `ak.typename`; `ak.to_backend`/`ak.materialize`/`@numba.njit`.
## Interop
- **uproot** — ROOT → ak.Array · **vector** — `register_awkward()` + `vector.zip` · **coffea** — columnar HEP analysis on awkward.
- **hist** (weights): broadcast the weight first, then fill:
```python
import awkward as ak, hist
h = hist.Hist(hist.axis.Regular(10, 0, 5), storage=hist.storage.Weight())
x = ak.Array([[1.1, 2.2], [0.5]]); w = ak.Array([1.0, 3.0])   # per-event weight
wb = ak.broadcast_arrays(w, x)[0]       # must broadcast per-event weight -> [[1.0, 1.0], [3.0]]
h.fill(ak.flatten(x).to_numpy(), weight=ak.flatten(wb).to_numpy())   # flattened weights -> [1.0, 1.0, 3.0]
```
## Reference files
| Task | Reference |
|---|---|
| Creating, `ak.Array`/`ak.Record`, `ArrayBuilder` | `references/high-level.md` |
| Any `ak.*`/`ak.str.*` function (full index, strings) | `references/api.md` |
| `ak.type`, types, layouts, kernels | `references/types-layouts.md` |
| Backends, behaviors, Numba/JAX | `references/backends-behaviors.md` |
| Slicing/broadcasting/reducers/missing/perf/version notes | `references/advanced-patterns.md` |
Docs: https://awkward-array.org/ · https://github.com/scikit-hep/awkward
