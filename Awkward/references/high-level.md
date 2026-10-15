# Awkward — Creating arrays: `ak.Array`, `ak.Record`, `ArrayBuilder`

Read this when you are constructing awkward arrays: from Python objects, JSON, NumPy, or incrementally with `ArrayBuilder`.

Verified against `awkward` 2.14.0. Full function index: [`api.md`](api.md). Type/layout details: [`types-layouts.md`](types-layouts.md).

## `ak.Array(data, *, behavior=None, attrs=None)`

Wraps a columnar layout and accepts: Python lists/dicts, JSON strings (parsed as JSON — `ak.Array("plain text")` raises; wrap text in a list), another `ak.Array`, a NumPy array, or a low-level `Content` layout. Members: `to_list()`, `to_numpy()`, `to_layout()`, `to_packed()`, `type`, `layout`, `behavior`, `attrs`, `__getitem__`, `__setitem__`.

```python
import awkward as ak
import numpy as np
a = ak.Array([{"x": 1.1, "y": [1, 2]}, {"x": None, "y": [3]}])   # Python -> records/lists
b = ak.Array('[[1, 2], [3]]')                                     # JSON string
c = ak.Array(np.array([1.0, 2.0, 3.0]))                           # NumPy -> regular 1d
print(ak.to_list(a))     # [{'x': 1.1, 'y': [1, 2]}, {'x': None, 'y': [3]}]
print(ak.to_list(b))     # [[1, 2], [3]]
print(ak.to_list(c))     # [1.0, 2.0, 3.0]
print(str(ak.type(a)))   # 2 * {x: ?float64, y: var * int64}
```

### In-place assignment (`__setitem__`) — fields only

In 2.14.0 only **whole-field assignment by name** is allowed: `arr["name"] = value` (value broadcasts; `= None` adds a `None` field). `arr[mask] = v`, `arr[i] = v`, and `arr["field", i] = v` all raise `TypeError: only fields may be assigned in-place (by field name)`.

```python
import awkward as ak
arr = ak.Array([{"pt": 1.0}, {"pt": 2.0}])
arr["eta"] = ak.Array([0.1, 0.2])          # add/replace a field
arr["tag"] = None                          # all-None field
print(ak.to_list(arr))   # [{'pt': 1.0, 'eta': 0.1, 'tag': None}, {'pt': 2.0, 'eta': 0.2, 'tag': None}]
```

For non-mutating versions use `ak.with_field` / `ak.without_field` (see `advanced-patterns.md`), and `ak.where(cond, x, y)` to replace values by a condition.

## `ak.Record`

A single record: `ak.Record(array, at)` or `array[i]`. Same field access, `type`, `layout`, `behavior`; `to_list()` gives a dict.

```python
import awkward as ak
a = ak.Array([{"x": 1.1, "y": [1, 2]}, {"x": 2.2, "y": [3]}])
r = a[0]
print(r["x"], ak.to_list(r["y"]), r.to_list())   # 1.1 [1, 2] {'x': 1.1, 'y': [1, 2]}
```

## `ak.ArrayBuilder` — incremental construction

Builds type-inferring output from loops; `snapshot()` returns an immutable `ak.Array`. The builder itself is **not** a context manager — create it first, then use `with b.list():` / `with b.record():` inside.

```python
import awkward as ak
events = [{"jets": [{"pt": 10.0, "eta": 1.0}, {"pt": 20.0, "eta": 2.0}]},
          {"jets": [{"pt": 5.0, "eta": 0.5}]}]
b = ak.ArrayBuilder()                        # NOT a context manager
for event in events:
    with b.list():                           # one event
        for jet in event["jets"]:
            with b.record():                 # one jet
                b.field("pt").real(jet["pt"]); b.field("eta").real(jet["eta"])
array = b.snapshot()                         # ak.Array (not .build())
print(ak.to_list(array))
# [[{'pt': 10.0, 'eta': 1.0}, {'pt': 20.0, 'eta': 2.0}], [{'pt': 5.0, 'eta': 0.5}]]
```

Builder API: `.append(x)` `.extend(iterable)` `.null()` (there is **no `.none()` — only `.null()`), `.boolean(True)`, `.integer(1)`, `.real(1.5)`, `.string("s")`, `.list()/.record()/.tuple()` (contexts), `.field("name")`, plus `begin_list()/end_list()` etc. Rules:

- **Append-only**, typed on the fly — first type seen at each position wins (later mismatches become a union or raise).
- `snapshot()` can be called repeatedly; call `ak.to_packed(array)` on the final array to compact buffers.
- `b.field("x")` must appear in the same order in every record of a given `with b.record():` block.

To create records from arrays of equal shape instead, use `ak.zip({"pt": ..., "eta": ...})` (index: `api.md` → Records); to give records physics methods, see `backends-behaviors.md` (vector registration).

## Related

- [`api.md`](api.md) — everything else (`from_*` constructors, `zip`, conversion).
- [`types-layouts.md`](types-layouts.md) — what `ak.type` reports, layouts.
