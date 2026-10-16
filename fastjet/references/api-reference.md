# API reference

All names below are available from `import fastjet` (the classic C++ surface is
re-exported from the SWIG module `fastjet._swig`). The array-oriented surface is
on `fastjet.ClusterSequence` and its dispatcher subclasses.

## Jet definitions

```python
fastjet.JetDefinition(algorithm, R, recomb_scheme=..., strategy=..., nparameters=...)
fastjet.JetDefinition(fastjet.antikt_algorithm, 0.6)          # most common
fastjet.JetDefinition(fastjet.genkt_algorithm, 0.6, fastjet.E_scheme, fastjet.Best, -1)  # p = -1 = anti-kT
```

- First argument is **always** the algorithm constant; remaining args depend on it.
- `R_in` may be passed as a keyword; the class is a picklable subclass of
  `_swig.JetDefinition` (pickling matters for Dask/graphed workers). If `R_in` is
  omitted entirely, construction fails — always pass a radius.
- `jetdef.description()` gives a stable, hashable description string (used by the
  graphed backend for payload identity).

### Algorithm constants

| Constant | Notes |
|---|---|
| `kt_algorithm` | longitudinally invariant kT |
| `cambridge_algorithm` | C/A (Aachen) |
| `antikt_algorithm` | anti-kT |
| `genkt_algorithm` | generalized kT; `p = extra_param()`: 1=kT, 0=C/A, −1=anti-kT |
| `ee_kt_algorithm` | e+e− kT (Durham) |
| `ee_genkt_algorithm` | e+e− genkt |
| `cambridge_for_passive_algorithm` | internal; auto-selected for passive Cambridge areas |
| `plugin_algorithm` | plugin-based definitions |

Recombination schemes: `E_scheme` (default), `pt_scheme`, `pt2_scheme`,
`Et_scheme`, `Et2_scheme`, `BIpt_scheme`, `BIpt2_scheme`, `WTA_pt_scheme`,
`WTA_modp_scheme`, `external_scheme`. Strategies include `Best`, `N2Plain`,
`N2Tiled`, `N2MinHeapTiled`, `N3Dumb`, `NlnN*`, `N2MHTLazy*`.

## `fastjet.ClusterSequence(data, jetdef)`

Dispatches on `data` type (see `input-formats.md`). Every query below is
available on the Awkward, Dask, and graphed arms. Return types: Awkward arm
returns `ak.Array` (scalar-like methods return a plain value for single-event
input, an `ak.Array` for multi-event); Dask returns lazy dask-awkward arrays;
graphed returns recorded queries.

### Inclusive / core

| Method | Returns |
|---|---|
| `jet_def()` | the `JetDefinition` used |
| `inclusive_jets(min_pt=0)` | jets with pt ≥ `min_pt`, one list per event |
| `unclustered_particles()` | particles not in any inclusive jet |
| `constituents(min_pt=0)` | particles per jet (preserves extra record fields) |
| `constituent_index(min_pt=0)` | indices of constituents per jet (into the input particle list) |
| `jet_scale_for_algorithm(data)` | kt²-style scale for a jet under this algorithm |
| `n_particles()` | number of input particles per event |
| `childless_pseudojets()` | pseudojets with no children not among inclusive jets |
| `jets()` | full internal pseudojet list (inputs + intermediate + final) |
| `unique_history_order()` | order that gives identical histories for degeneracies |

### Exclusive jets

| Method | Notes / errors |
|---|---|
| `exclusive_jets(n_jets=-1, dcut=-1)` | exactly one of `n_jets`/`dcut` must be set; `0` raises `ValueError`; `n_jets` > constituents raises `RuntimeError` |
| `exclusive_jets_up_to(n_jets=-1)` | safe variant that returns what exists (never raises for too many) |
| `exclusive_jets_ycut(ycut=-1)` | e+e− style ycut variant |
| `exclusive_jets_constituent_index(njets=10)` | per-jet constituent indices (njets must be > 0) |
| `exclusive_jets_constituents(njets=10)` | per-jet constituent particles |
| `exclusive_dmerge(njets=10)` | `d_min` of the merge n+1 → n |
| `exclusive_dmerge_max(njets=10)` | max `d_min` up to the n-jet state |
| `exclusive_ymerge(njets=10)` / `exclusive_ymerge_max(njets=10)` | normalised by Q² |
| `Q()` / `Q2()` | total energy / its square (e+e−) |
| `n_exclusive_jets(dcut=0)` | count of jets at `dcut` |

### Subjets / clustering history (need jets from this sequence as `data`)

`exclusive_subjets(data, dcut=-1, nsub=-1)`, `exclusive_subjets_up_to(data, nsub=0)`,
`exclusive_subdmerge(data, nsub=0)`, `exclusive_subdmerge_max(data, nsub=0)`,
`n_exclusive_subjets(data, dcut=0)`, `has_parents(data)`, `has_child(data)`,
`get_parents(data)`, `get_child(data)`.

Caveat: `has_*`/`get_*` identify the input jet among the event's inclusive jets by
rapidity (`_ext.cpp` builds a `rap → index` map), so a jet not in this sequence
throws `RuntimeError: Jet Not in this ClusterSequence`. In the multi-event arms,
`data` must be an array whose first axis matches events (Dask additionally
requires partition-wise compatibility with the clustered input).

### Jet substructure / observables (from fastjet-contrib)

- `exclusive_jets_softdrop_grooming(njets=1, beta=0.0, symmetry_cut=0.1,
  symmetry_measure="scalar_z", R0=0.8, recursion_choice="larger_pt", mu_cut=inf)`
  → record with `constituents`, `msoftdrop`, `ptsoftdrop`, `etasoftdrop`,
  `phisoftdrop`, `Esoftdrop`, `pzsoftdrop`, `deltaRsoftdrop`, `symmetrysoftdrop`.
- `njettiness(measure_definition="NormalizedMeasure",
  axes_definition="OnePass_KT_Axes", njets=[1,2,3,4], beta=1.0, R0=0.8,
  Rcutoff=None, nPass=None, akAxesR0=None)` → tau values, shape (events, len(njets)).
  Measure names: `NormalizedMeasure`, `UnnormalizedMeasure`,
  `OriginalGeometricMeasure`, `NormalizedCutoffMeasure`,
  `UnnormalizedCutoffMeasure`, `GeometricCutoffMeasure`. Axes names: `KT_Axes`,
  `CA_Axes`, `AntiKT_Axes`, `WTA_KT_Axes`, `WTA_CA_Axes`, `Manual_Axes`,
  `OnePass_*`, `MultiPass_Axes`. Unsupported names silently fall back to the
  defaults in `_ext.cpp` — pass exact strings.
- `exclusive_jets_energy_correlator(njets=1, beta=1, npoint=0, angles=-1,
  alpha=0, func="generalized", normalized=True)` → ECF/ECFG values.
- `exclusive_jets_lund_declusterings(njets=10)` → records `{Delta, kt}`.

## Classic interface (list of PseudoJet)

```python
particles = [fastjet.PseudoJet(px, py, pz, E), ...]          # order: px, py, pz, E
cluster = fastjet.ClusterSequence(particles, jetdef)          # list => SWIG ClusterSequence
inc = cluster.inclusive_jets()                                # list[PseudoJet]
# cutoff key follows the C++ name: inclusive_jets(ptmin=25.0)  (columnar uses min_pt=)
```

- `PseudoJet` accessors: `.px() .py() .pz() .E() .pt() .eta() .phi() .m() .rap()`,
  `.area() .has_area()`, `constituents()`, `has_parents()`, `has_child()`,
  `parent1()`, `parent2()`.
- Classic surface includes `ClusterSequenceArea`, `AreaDefinition`,
  `GhostedAreaSpec`, `VoronoiAreaSpec`, `Selector*` (e.g. `SelectorPtMin`,
  `SelectorAbsRapMax`), taggers/filters (`Filter`, `Pruner`, `Recluster`,
  `MassDropTagger`, `JHTopTagger`, `CASubJetTagger`,
  `RestFrameNSubjettinessTagger`), `Subtractor`, `BackgroundEstimator*`.
  The docs point at the C++ Doxygen for semantics: http://fastjet.fr.
- **Areas/selectors/taggers are classic-only** (SWIG): there is no
  columnar/Dask/graphed path for `ClusterSequenceArea`/`AreaDefinition` — see
  `examples.md` §9.

## Utility functions (`fastjet._utils`, re-exported)

`sorted_by_E`, `sorted_by_pt`, `sorted_by_pz`, `sorted_by_rapidity` (Awkward or
SWIG input), `join(pj1[, pj2[, pj3[, pj4]]])` (max 4 args), `dot_product`,
`theta`, `cos_theta`, `have_same_momentum`, `PtYPhiM`. Passing Awkward arrays to
the SWIG-only math helpers raises `TypeError` ("Use inbuilt methods for Awkward
Array") — use vector's methods instead.

## Internal modules (do not use directly)

`fastjet._pyjet.AwkwardClusterSequence` / `DaskAwkwardClusterSequence`,
`fastjet._graphed.GraphedClusterSequence`, `fastjet._singleevent/_multievent/
_generalevent._class*` are implementation details; tests use them, but public code
should only use `fastjet.ClusterSequence`. (The `_pyjet` module name here is
unrelated to the archived `pyjet` PyPI package.)
