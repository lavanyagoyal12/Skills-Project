# Awkward — Types, layouts & validity

Read this when you need to interpret `ak.type` strings, inspect the low-level node tree (`contents`), or check whether an array is well-formed.

Verified against `awkward` 2.14.0. Function signatures: [`api.md`](api.md). Creating arrays: [`high-level.md`](high-level.md).

## `ak.type(a)` — the high-level type

```python
import awkward as ak
x = ak.Array([[1, 2, 3], [], [4, 5]])                 # irregular outer? no: 3 elements, lists inside
print(str(ak.type(x)))                # 3 * var * int64        (fixed outer, ragged inner)
print(str(ak.type(ak.Array([[1.0], [2.0]]))))          # 2 * var * float64
print(str(ak.type(ak.Array([{"pt": 1.0, "eta": 0.5}]))))  # 1 * {pt: float64, eta: float64}
print(str(ak.type(ak.Array([[1.0, None], [2.0]]))))    # 2 * var * ?float64    (? = option/None)
```

Notation: `var * T` variable list, `N * T` fixed-size list, `?T` option (None-able), `{name: T, ...}` record, `(T1, T2)` tuple, `T1 | T2` union, `unknown` empty/no-type.

Type classes (in `ak.types`): `ArrayType`, `ScalarType`, `ListType`, `RegularType`, `NumpyType`, `OptionType`, `RecordType`, `UnionType`, `UnknownType`. Helpers: `ak.types.from_datashape("var * int64")`, `is_primitive`, `primitive_to_dtype`, `dtype_to_primitive`.

**Type ≠ layout**: `ak.type` distinguishes regular vs variable lists, options, records, unions — but not *how* the data is stored (`ListOffsetArray` vs `ListArray`, mask vs index).

## Layouts (`ak.contents` node tree)

Every `ak.Array` wraps a tree of `contents.Content` nodes; each node is a known category (kernels operate on whole buffers, which is why operations are fast):

| Node | Meaning |
|---|---|
| `NumpyArray` | contiguous leaf data |
| `ListOffsetArray` | variable lists via `offsets` (canonical) |
| `ListArray` | variable lists via `starts`/`stops` |
| `RegularArray` | fixed-size lists |
| `RecordArray` | named fields |
| `TupleArray` | unnamed fields (tuples) |
| `IndexedArray` | permutation (argsort, slicing) |
| `IndexedOptionArray` | missing via index `-1` |
| `ByteMaskedArray` / `BitMaskedArray` | masks |
| `UnmaskedArray` | option without storage cost |
| `UnionArray` | mixed types |
| `EmptyArray` | no data |

```python
import awkward as ak
x = ak.Array([[1, 2, 3], [], [4, 5]])
lay = ak.to_layout(x)          # or x.layout
print(type(lay).__name__)      # ListOffsetArray
form, length, buffers = ak.to_buffers(x)   # (Form, length, dict of arrays)
print(form)                    # the serialized tree structure
print(sorted(buffers))         # buffer keys, e.g. ['node0-offsets', 'node1-data']
```

`ak.contents` nodes are rarely needed in analyses, but they explain why `ak.to_numpy` can only convert "regular" shapes and why an operation chain produces `IndexedArray` (fix with `ak.to_packed`). Index buffers are `ak.index.Index8/32/64`, `IndexU8/32/64`; `ak.forms.Form` (and `ak.forms.from_json`) mirror the tree for serialization; `ak.transform(fn, array)` walks nodes applying `fn` per node.

## Validity checks

```python
import awkward as ak
x = ak.Array([[1, 2, 3], [], [4, 5]])
print(ak.is_valid(x))                                   # True
print(repr(ak.validity_error(x)))                       # '' (empty string when valid)
# A malformed structure (e.g. bad buffer lengths) returns a message string instead of ''.
ak.without_parameters(x)   # strip JSON-ish parameters that can confuse validation
```

`ak.is_valid` returns a bool; `ak.validity_error` returns `''` when valid, otherwise a description of the first problem. `ak.without_parameters`/`ak.with_parameter` (index: `api.md` → Records) control metadata, which `validity_error` may flag.

## Kernels (awkward development)

`kernel-specification.yml` (repo root) is the source of truth for every low-level kernel; CPU C++ and CUDA implementations must conform to it identically so NumPy/CuPy/JAX/typetracer backends agree. Not needed for analysis code — needed when developing awkward itself or writing new backends (see `backends-behaviors.md`).

## Related

- [`api.md`](api.md) — all `ak.*` functions (including `ak.type`, `ak.is_valid`, `ak.to_buffers`).
- [`backends-behaviors.md`](backends-behaviors.md) — what `ak.backend` reports; `ak.to_backend`.
