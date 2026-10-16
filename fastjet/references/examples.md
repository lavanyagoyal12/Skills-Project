# Examples

Recipes below mirror the repository's own tests; the tests are ground truth for
the expected values. The canonical input used throughout the suite:

```python
import awkward as ak
import fastjet

jetdef = fastjet.JetDefinition(fastjet.antikt_algorithm, 0.6)

single = ak.Array([
    {"px": 1.2, "py": 3.2, "pz": 5.4, "E": 2.5, "ex": 0.78},
    {"px": 32.2, "py": 64.21, "pz": 543.34, "E": 24.12, "ex": 0.35},
    {"px": 32.45, "py": 63.21, "pz": 543.14, "E": 24.56, "ex": 0.0},
])  # one event -> flat output

multi = ak.Array([single, single])  # two events -> output has one list per event
```

## 1. Basic clustering (single + multi event)

```python
cluster = fastjet.ClusterSequence(single, jetdef)   # AwkwardClusterSequence
cluster.inclusive_jets().to_list()
# [{"px": 1.2, "py": 3.2, "pz": 5.4, "E": 2.5},
#  {"px": 64.65, "py": 127.41999999999999, "pz": 1086.48, "E": 48.68}]

cluster.constituents().to_list()
# [[{"px": 1.2, ...}], [{"px": 32.2, ...}, {"px": 32.45, ...}]]   # extra field `ex` kept

cluster.constituent_index().to_list()   # [[0], [1, 2]]
```

Pinned by `tests/test_001-basic_clustering.py` (`test_multi`, `test_single`).
With a cut: `cluster.inclusive_jets(min_pt=5.0)`.

## 2. Exclusive jets

```python
cluster.exclusive_jets(n_jets=3).to_list()   # exactly 3 jets (all particles)
# [{"px": 32.2, ...}, {"px": 32.45, ...}, {"px": 1.2, ...}]

cluster.exclusive_jets(dcut=0.0001).to_list()
# [{"px": 1.2, "py": 3.2, "pz": 5.4, "E": 2.5}]

cluster.exclusive_jets_up_to(n_jets=4).to_list()   # safe: only 3 exist
```

Pinned by `tests/test_002-exclusive_jets.py`: `exclusive_jets(n_jets=4)` raises
`RuntimeError`; `exclusive_jets_up_to(n_jets=4)` returns the 3 constituents.
Other exclusive quantities: `exclusive_dmerge(njets=1)` ≈ 4.898e-5
(`tests/test_004-exclusive_single-out.py`).

## 3. Subjets and the clustering tree

Run only on jets produced by the same cluster sequence:

```python
jets = cluster.inclusive_jets()
cluster.exclusive_subjets(jets, nsub=2).to_list()      # or dcut=
cluster.exclusive_subjets_up_to(jets, nsub=2).to_list()
cluster.n_exclusive_subjets(jets, dcut=0.0001)
cluster.get_parents(jets).to_list()                    # [] for jet 0, particles for jet 1
cluster.has_parents(jets).to_list()                    # [False, True]
```

Pinned by `tests/test_006-jet_input.py` (note: for the multi-event arm these
methods require the jet array to have the same event axis as the clustered data).

## 4. Substructure observables

```python
# SoftDrop-groomed exclusive jets -> record with constituents + softdrop metadata
cluster.exclusive_jets_softdrop_grooming(njets=1).to_list()

# N-jettiness: default taus for N = 1,2,3,4 -> shape (events, 4)
cluster.njettiness(njets=2, beta=1.0, R0=0.8).to_list()

# Energy correlators / ECFG
cluster.exclusive_jets_energy_correlator(njets=1, beta=1, npoint=2).to_list()

# Lund declustering: records {Delta, kt}
cluster.exclusive_jets_lund_declusterings(njets=10).to_list()
```

Covered by `tests/test_006-misc.py` and `src/_ext.cpp`; `docs/Awkward.rst` has no
worked numbers, so verify against a known physics value before trusting a bulk run.

## 5. Sorting and helpers

```python
fastjet.sorted_by_pt(events)         # per-event, ascending pt (ak.argsort-based)
fastjet.sorted_by_pz(events)
fastjet.sorted_by_E(events)
fastjet.sorted_by_rapidity(events)
```

Pinned by `tests/test_005_sorting.py`. These accept awkward arrays; for the
classic interface they accept lists of `PseudoJet`.
Avoid `fastjet.sort_indices` with non-awkward input (see gotchas.md).

## 6. Dask-awkward (out-of-core, ROOT)

```python
import dask_awkward as dak
import uproot

lazy = dak.from_awkward(events_ak, npartitions=4)
cluster = fastjet.ClusterSequence(lazy, jetdef)      # DaskAwkwardClusterSequence
cluster.inclusive_jets().compute().to_list()

# from ROOT
devents = uproot.dask({"file.root": "Events"})
particles = ak.zip({"px": devents["Px"], "py": devents["Py"],
                    "pz": devents["Pz"], "E": devents["E"]},
                   with_name="Momentum4D")
jets = fastjet.ClusterSequence(particles, jetdef).inclusive_jets().compute()
```

See `tests/test_008-dask.py` (including `test_inclusive_from_file` with
`tests/samples/pfnano_skim.root`). Requires `pip install fastjet[dask]`.

## 7. Graphed (deferred)

```python
import graphed
import graphed.awkward as ga
import vector

session = graphed.Session(ga.AwkwardBackend(behavior=vector.backends.awkward.behavior))
events = ga.from_awkward(session, "particles", list_of_events)  # one list per event!
cluster = fastjet.ClusterSequence(events, jetdef)
jets = cluster.inclusive_jets()          # records a node; nothing runs yet
session.materialize(jets)                # -> ak.Array
```

See `docs/Awkward.rst` (Deferred Clustering) and `tests/test_010-graphed.py`,
`tests/test_011-graphed_queries.py` (one event per entry requirement,
session binding). Requires `pip install fastjet[graphed]`.

## 8. Classic interface (one event, C++-style)

```python
particles = [
    fastjet.PseudoJet(1.1, 1.2, 1.3, 11.4),
    fastjet.PseudoJet(2.1, 2.2, 2.3, 12.4),
    fastjet.PseudoJet(3.1, 3.2, 3.3, 13.4),
]
cluster = fastjet.ClusterSequence(particles, fastjet.JetDefinition(fastjet.antikt_algorithm, 0.6))
for jet in cluster.inclusive_jets():
    print(jet.px(), jet.py(), jet.pz(), jet.E(), jet.pt(), jet.eta(), jet.phi())
```

Cut by pT here with the C++-style keyword `ptmin=` (columnar uses `min_pt=`):
`cluster.inclusive_jets(ptmin=25.0)` — a positional `inclusive_jets(25.0)` works
in both interfaces.

From `docs/particle.rst`. Full C++ surface (areas, selectors, taggers,
subtractors) is available through this interface — see `api-reference.md`.

## 9. Jet areas (classic interface only)

```python
import fastjet

particles = [
    fastjet.PseudoJet(1.1, 1.2, 1.3, 11.4),
    fastjet.PseudoJet(2.1, 2.2, 2.3, 12.4),
    fastjet.PseudoJet(3.1, 3.2, 3.3, 13.4),
]
jetdef = fastjet.JetDefinition(fastjet.antikt_algorithm, 0.6)
ghost_area = fastjet.AreaDefinition(fastjet.active_area, fastjet.GhostedAreaSpec(5.0))
cs_area = fastjet.ClusterSequenceArea(particles, jetdef, ghost_area)
jets = cs_area.inclusive_jets(ptmin=25.0)    # classic kwarg: ptmin, not min_pt
for j in jets:
    print(j.area(), j.has_area(), j.pt())
```

Jet-area classes (`ClusterSequenceArea`, `AreaDefinition`, `GhostedAreaSpec`,
`VoronoiAreaSpec`) exist **only** through the classic (SWIG) interface — there
is no Awkward/Dask/graphed area path. Used for pileup subtraction; see the C++
Doxygen for semantics (http://fastjet.fr).
