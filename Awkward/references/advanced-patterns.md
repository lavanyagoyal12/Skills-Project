# Awkward — Advanced patterns, gotchas & version notes

Read this when your analysis moves past the quick start: cutting/masking, broadcasting per-event scalars, reducer semantics, missing values, flattening/grouping, combinatorics, sorting, NumPy interop, file I/O, performance, or migrations from awkward 1.x.

Verified against `awkward` 2.14.0. Function signatures: [`api.md`](api.md). Creating arrays: [`high-level.md`](high-level.md). Types/layouts: [`types-layouts.md`](types-layouts.md). Backends/Numba/JAX/vector: [`backends-behaviors.md`](backends-behaviors.md).

## Contents

- [Slicing & cut vs mask](#slicing--cut-vs-mask)
- [Broadcasting](#broadcasting)
- [Reducer semantics](#reducer-semantics)
- [argmin / argmax](#argmin--argmax)
- [Missing data & options](#missing-data--options)
- [Flattening, lengths & grouping](#flattening-lengths--grouping)
- [Filtering & aggregation](#filtering--aggregation)
- [Records & fields](#records--fields)
- [Combinatorics](#combinatorics)
- [Sorting & indexing](#sorting--indexing)
- [NumPy interop](#numpy-interop)
- [File I/O (awkward-files)](#file-io-awkward-files)
- [Best practices & performance](#best-practices--performance)
- [1.x vs 2.x differences](#1x-vs-2x-differences)
- [Pitfalls](#pitfalls)

## Slicing & cut vs mask

```python
import awkward as ak
arr = ak.Array([[1, 2, 3], [], [4, 5]])
print(ak.to_list(arr[arr > 2]))                 # cut: elements removed -> [[3], [], [4, 5]]
print(ak.to_list(ak.mask(arr, arr > 2)))        # mask: shape kept, failures become None
#                                                              -> [[None, None, 3], [], [4, 5]]
print(ak.to_list(arr[ak.Array([2, 0])]))        # integer-list indexing -> [[4, 5], [1, 2, 3]]
```

`arr[sel]` supports `...`, slices, negatives, booleans, integer lists, and `ak.Array` index arrays (missing values in the index → missing in output). Boolean getitem **removes** (cut); `ak.mask` **preserves shape**. Rule: if a later step depends on list lengths, use `ak.mask` — independently cut arrays (`x[x > 2]`, `y[y > 30]`) lose per-item alignment: `x` keeps items 3,4,5 while `y` keeps 40,50.

## Broadcasting

Rules follow NumPy by nested dimension: scalars and length-1 lists broadcast against lists. Use `ak.broadcast_arrays` to force alignment; `keepdims=True` keeps the reduced axis so a scalar becomes a length-1 list that broadcasts back.

```python
import awkward as ak
jets_pt = ak.Array([[10.0, 20.0, 30.0], [5.0]])
best = ak.broadcast_arrays(ak.max(jets_pt, axis=1, keepdims=True), jets_pt)[0]
print(ak.to_list(best))        # [[30.0, 30.0, 30.0], [5.0]]   (per-event max at jet level)
added = ak.Array([1, 2])[:, None]           # np.newaxis / None: new trailing axis
print(ak.to_list(added))       # [[1], [2]]
```

Options broadcast: `?T` with `T` → `?T`. Records combine field-by-field; differing record shapes need `ak.merge_union_of_records` / `ak.merge_option_of_records` before operations.

## Reducer semantics

Reducers take `axis=None` (all levels, returns a 0-d `ak.Array`), `axis=0` outermost … `axis=-1` innermost, and `keepdims=True` keeps the reduced axis (length 1) so results broadcast.

```python
import awkward as ak
x = ak.Array([[1.0, 2.0], [], [None, None]])
print(ak.to_list(ak.max(x, axis=1, mask_identity=True)))    # [2.0, None, None]  (default for min/max/argmax/argmin)
print(ak.to_list(ak.max(x, axis=1, mask_identity=False)))   # [2.0, -inf, -inf]  identity instead of None
print(ak.to_list(ak.sum(x, axis=1, mask_identity=False)))   # [3.0, 0.0, 0.0]    (default False for sum/prod/...)
print(ak.to_list(ak.sum(x, axis=1, mask_identity=True)))    # [3.0, None, None]
```

- `keepdims` defaults to **`False`** for all reducers (including `argmax`/`argmin`).
- `mask_identity=True` → **`None`** for empty/all-`None` input; `False` → the identity: `sum`→0, `prod`→1, `max`→-inf, `min`→+inf. Defaults: `True` for min/max/argmax/argmin, `False` for sum/prod/count/mean.
- `nan*` variants (`ak.nansum`, `ak.nanmax`, …) skip NaN instead of making the result NaN; `ak.nan_to_none` converts NaN → None, `ak.nan_to_num` → numbers.
- `ak.count` (non-None count), `ak.count_nonzero`, `ak.any`/`ak.all`, `ak.mean`/`var`/`std` (`weight=`), `ak.moment`, `ak.corr`/`ak.covar`, `ak.linear_fit`, `ak.ptp`, `ak.softmax` (index: `api.md`).

## argmin / argmax

```python
import awkward as ak
score = ak.Array([[0.1, 0.9, 0.5], [], [0.7, 0.3]])
data = ak.Array([[10, 20, 30], [], [40, 50]])
best = data[ak.argmax(score, axis=1, keepdims=True)][:, 0]   # index, then drop the length-1 axis
print(ak.to_list(best))                     # [20, None, 40]   (empty list -> None)
top2 = data[ak.argsort(score)][:, :2]       # top-n via argsort + slice
print(ak.to_list(top2))                     # [[10, 30], [], [50, 40]]
```

`argmax`/`argmin` return an array with **one fewer dimension** — that's why `keepdims=True` plus `[:, 0]` is the pattern. Broadcasting `x[ak.argmax(x, axis=1, keepdims=True)]` directly against `x` raises `ValueError: cannot broadcast nested list` — use `ak.max(..., keepdims=True)` for values (see Broadcasting) or `[:, 0]` for selections. `ak.argsort` keeps dimensions, so no `keepdims` needed.

## Missing data & options

Options are `?T`. `ak.fill_none` defaults to `axis=-1` (fills only the innermost level — an outer `None` stays); `axis=None` fills all levels. `ak.pad_none(a, n, axis=1)` pads events to `n`; `clip=True` makes the result a fixed-size `RegularType` (needed for ML batching).

```python
import awkward as ak
a = ak.Array([[1.0, 2.0], None, [3.0, None, 4.0]])
print(ak.to_list(ak.is_none(a, axis=0)))            # [False, True, False]
print(ak.to_list(ak.fill_none(a, -1)))              # [[1.0, 2.0], None, [3.0, -1.0, 4.0]]  outer None stays
print(ak.to_list(ak.fill_none(a, -1, axis=None)))   # [[1.0, 2.0], -1, [3.0, -1.0, 4.0]]   all levels
print(ak.to_list(ak.drop_none(a)))                  # [[1.0, 2.0], [3.0, 4.0]]
print(ak.to_list(ak.pad_none(a, 3, axis=1)))        # [[1.0, 2.0, None], None, [3.0, None, 4.0]]
p = ak.pad_none(a, 3, axis=1, clip=True)
print(str(ak.type(p)))                              # 3 * option[3 * ?float64]  (RegularArray inside Option)
print(ak.to_list(ak.firsts(a, axis=1)))             # [1.0, None, 3.0]
```

`ak.drop_none` before NumPy/plotting; `ak.fill_none` before NumPy-only code (NumPy functions don't accept awkward's options). `ak.mask(a, cond, valid_when=False)` inverts which side becomes None.

## Flattening, lengths & grouping

```python
import awkward as ak
x = ak.Array([[1, 2, 3], [4, 5]])
print(ak.to_list(ak.num(x, axis=1)))                    # [3, 2]
print(ak.to_list(ak.flatten(x, axis=1)))                # [1, 2, 3, 4, 5]
print(ak.to_list(ak.unflatten(ak.flatten(x, axis=1), [2, 3], axis=0)))  # [[1, 2], [3, 4, 5]]
print(ak.to_list(ak.ravel(x)))                          # [1, 2, 3, 4, 5]
print(ak.to_list(ak.flatten(ak.Array([None, [1, 2], None]))))  # [1, 2]  (default axis=0 drops top None)
```

- `ak.num`/`ak.flatten`/`ak.pad_none` default to `axis=1` (event level); reducers default to `axis=None`.
- `ak.flatten(axis=0)` only removes **top-level `None`s** (lists stay nested — use `ak.flatten(axis=1)` first for full flattening).
- Grouping: `ak.unflatten(values, counts, axis=0)` restores raggedness from counts (`ak.num` gives counts); `ak.run_lengths` finds consecutive-run lengths; `ak.local_index` gives within-list positions.

## Filtering & aggregation

```python
import awkward as ak
x = ak.Array([[1, 2, 3], [4, 5, 6]])
print(ak.to_list(ak.sum(x > 3, axis=1)))         # [0, 3]        count of passing items
print(ak.to_list(ak.any(x > 3, axis=1)), ak.to_list(ak.all(x > 3, axis=1)))   # [False, True] [False, True]
print(ak.to_list(ak.count_nonzero(x > 3, axis=1)))   # [0, 3]
print(ak.to_list(ak.num(x[x > 3], axis=1)))          # [0, 2]       sizes after a cut
```

Per-event selection + reduction: combine a boolean `ak.mask` (keeps shape) with reducers, or cut then reduce. For "one representative item per event", use argmin/argmax patterns; for groupwise statistics, `ak.broadcast_arrays` + reducers with `keepdims=True`.

## Records & fields

```python
import awkward as ak
r = ak.Array([{"pt": 1.0, "eta": 0.5}, {"pt": 2.0, "eta": 1.5}])
print(ak.fields(r))                                       # ['pt', 'eta']
print(ak.to_list(ak.with_field(r, ak.Array([1, 2]), "phi")))   # adds field 'phi'
print(ak.to_list(ak.without_field(r, "eta")))             # [{'pt': 1.0}, {'pt': 2.0}]
z = ak.zip({"a": ak.Array([1, 2]), "b": ak.Array([3, 4])})
print(ak.to_list(z), ak.is_tuple(z))                      # [{'a': 1, 'b': 3}, {'a': 2, 'b': 4}] False
```

`ak.zip` renames nothing (use `with_name=` for behaviors — `backends-behaviors.md`); `ak.merge_union_of_records`/`ak.merge_option_of_records` join differently-typed records. `ak.parameters`/`ak.with_parameter`/`ak.without_parameters` attach metadata; `arr["new_field"] = ...` is the in-place variant (fields only — see `high-level.md`).

## Combinatorics

```python
import awkward as ak
jets = ak.Array([[{"pt": 10.0, "eta": 1.0}, {"pt": 20.0, "eta": 2.0}, {"pt": 30.0, "eta": 3.0}],
                 [{"pt": 5.0, "eta": 1.0}]])
pairs = ak.combinations(jets, 2, axis=1)                  # all pairs within each list (n choose k)
print(ak.to_list(ak.num(pairs, axis=1)))                  # [3, 0]
j1, j2 = ak.unzip(pairs)
print(ak.to_list(j1["pt"]))                               # [[10.0, 10.0, 20.0], []]
print(ak.to_list(ak.argcombinations(jets, 2, axis=1)))    # [[(0, 1), (0, 2), (1, 2)], []]
cart = ak.cartesian([ak.Array([[1.0, 2.0], [3.0]]), ak.Array([[1.0], [2.0]])], axis=1)
print(ak.to_list(ak.num(cart, axis=1)))                   # [2, 1]   list -> tuples
named = ak.cartesian({"e": ak.Array([[1.0, 2.0], [3.0]]), "j": ak.Array([[1.0], [2.0]])}, axis=1)
print(ak.fields(named))                                   # ['e', 'j']  dict -> record
```

`ak.combinations(..., replacement=True)` allows self-pairs (for `ak.cartesian`, `replacement=` repeats same elements). `ak.argcartesian` = indices only. For invariant masses over pairs, name records `Momentum4D` and use vector (`backends-behaviors.md`).

## Sorting & indexing

```python
import awkward as ak
x = ak.Array([[3, 1, 2], [9, 5]])
print(ak.to_list(ak.sort(x, axis=1)))                  # [[1, 2, 3], [5, 9]]
i = ak.argsort(x, axis=1)
print(ak.to_list(i))                                   # [[1, 2, 0], [1, 0]]
print(ak.to_list(x[i]))                                # [[1, 2, 3], [5, 9]]  argsort indexes another array
print(ak.to_list(ak.local_index(x, axis=1)))           # [[0, 1, 2], [0, 1]]
print(ak.to_list(ak.run_lengths(ak.Array([[1, 1, 2], [2]]))))  # [[2, 1], [1]]
```

`ak.argsort` always keeps dimensions, so `data[ak.argsort(score)]` aligns rows; combine with `[:, :n]` for top-n (see argmin/argmax). `ak.sort`/`ak.argsort` default `axis=-1`, support `ascending=False`, `stable=True`.

## NumPy interop

```python
import awkward as ak
import numpy as np
x = ak.Array([[1.0, 4.0], [9.0]])
print(ak.to_list(np.sqrt(x)))                          # ufuncs work directly (no ak.sqrt) -> [[1.0, 2.0], [3.0]]
print(ak.to_list(ak.Array(np.array([1.0, 2.0, 3.0])))) # NumPy -> awkward: [1.0, 2.0, 3.0]
print(type(ak.to_numpy(ak.Array([[1.0, None], [2.0, 3.0]]))).__name__)  # MaskedArray (options)
```

- NumPy ufuncs (`np.sqrt`, `np.square`, `abs()`, …) work on `ak.Array` (no `ak.sqrt`, no `ak.abs`); NumPy functions that don't dispatch (e.g. `np.max`) don't — use `ak.max` or `.to_numpy()`.
- `ak.to_numpy` **raises on ragged arrays** and returns a `np.ma.MaskedArray` for option data — flatten/`pad_none`/`fill_none` or use `ak.to_list` first.
- `ak.Array(numpy_array)` wraps a NumPy array; `x[:, np.newaxis]` / `x[:, None]` add axes (there is no `ak.expand_dims`).

## File I/O (awkward-files)

```python
import awkward as ak, pathlib
print(ak.to_list(ak.from_json('[[1, 2], [3]]')))       # parses the string CONTENTS -> [[1, 2], [3]]
try:
    ak.from_json("some/file.json")                     # NOT a path!
except ValueError as e:
    print(type(e).__name__)                            # ValueError
print(ak.to_json(ak.Array([["a", "b"], []])))          # [["a","b"],[]]
pq = pathlib.Path("/tmp/opencode/demo.parquet")
ak.to_parquet(ak.Array([{"x": 1, "y": [1, 2]}, {"x": 2, "y": [3]}]), pq)
print(ak.to_list(ak.from_parquet(pq)))                 # [{'x': 1, 'y': [1, 2]}, {'x': 2, 'y': [3]}]
```

- `ak.from_json(str)` parses the string's contents — to read a file use `pathlib.Path("f.json").read_text()` (or `ak.from_json(pathlib.Path(...))`, supported in 2.14).
- Parquet preserves raggedness/options (PyArrow-backed); Arrow (`ak.to_arrow`/`from_arrow`) and buffers (`ak.to_buffers`/`from_buffers`) round-trip types exactly. Complete conversion list: `api.md`.
- `ak.Array(json_str)` also parses JSON — wrap plain text in a list or you get a parse error.

## Best practices & performance

1. Build once with `ArrayBuilder` (or `ak.Array(python)`) then `ak.to_packed`; avoid repeatedly appending to Python lists.
2. Vectorize with `ak.*` reducers/combinatorics; use `ak.to_packed` after slicing chains that produce `IndexedArray`.
3. Prefer single boolean conditions shared across fields (mask) over repeated cuts, and `keepdims`+`broadcast_arrays` over per-event Python loops.
4. `ak.pad_none(..., clip=True)` / `ak.to_regular` for ML batching; `ak.to_numpy` only for regular output.
5. Avoid `ak.to_list` on large arrays (Python objects = slow, memory-heavy); larger-than-memory patterns → typetracer/lazy backends and Numba loops (`backends-behaviors.md`).

## 1.x vs 2.x differences

- Type notation: `var * T` / `N * T` / `?T` (not `"O"` dtypes); `ak.type` replaces `ak.typename` (removed).
- `ak.combinations(..., replacement=...)` — the keyword was `replace` in some 1.x versions.
- `ArrayBuilder` → `snapshot()` (not `build()`); `ak.ArrayBuilder` is not a context manager.
- `ak.Array(json_str)` / `ak.from_json(str)` parse *content*; in-place `__setitem__` is fields-only (see `high-level.md`).
- No `ak.abs` / `ak.take` / `ak.expand_dims` / `ak.to_dlpack` / `ak.sqrt` — use `abs()`, `arr[ak.Array([...])]`, `arr[:, np.newaxis]`, and `np.sqrt`.
- `ak.str` namespace: real names are `find_substring`, `count_substring`, `split_pattern`, `split_whitespace`, `join`, `length` (no `find`/`split`/`multiplicity`).

## Pitfalls

- `arr[:, 0]` **raises `IndexError` on empty events** — use `ak.firsts(arr)` (`[[1, 2], None]` for `[[[1,2],[3,4]], []]`).
- `ak.to_numpy` on option data returns a `MaskedArray`; on ragged it raises — flatten or `pad_none` first.
- `ak.flatten(axis=0)` only removes top-level `None`; reduce nested lists with `axis=1`.
- Independent boolean cuts lose alignment; `ak.mask` keeps shape.
- `ak.mask` keyword is `valid_when` (not `when`); `ak.combinations` is `replacement` (not `replace`).
- Axes: `0` outermost, `1` event level, `-1` innermost; `ak.num`/`flatten`/`pad_none` default `axis=1`, reducers `axis=None`.
