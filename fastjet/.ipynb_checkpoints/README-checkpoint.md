# fastjet skill

A practical, task-oriented skill for using the Scikit-HEP `fastjet` Python
package (the official Python and Awkward Array bindings to the C++ FastJet
jet-finding library).

## Files

| Path | Purpose |
|---|---|
| `SKILL.md` | Concise entry point: interface decision, minimal skeleton, hard rules, troubleshooting path. Loaded automatically by the `skill` tool when relevant. |
| `references/api-reference.md` | Full query surface: algorithms, `JetDefinition`, `ClusterSequence` methods, classic-interface classes, helpers. |
| `references/input-formats.md` | Which coordinate systems, Awkward layouts, and backends (Dask/graphed) are supported; output shapes. |
| `references/gotchas.md` | Error catalog, known bugs and sharp edges (verified against this repository's source and tests). |
| `references/examples.md` | Copy-paste recipes for the main workflows, each tied to the test that asserts it. |

## Design principles

- `SKILL.md` is a checklist that changes what the agent does — it is not a
  tutorial. Depth lives in `references/`.
- Facts are taken from this repository (`src/fastjet/*`, `src/_ext.cpp`,
  `docs/*.rst`, `tests/*.py`) and are current as of FastJet 3.5.2 /
  fjcontrib 1.105.
- The test suite is ground truth: `tests/test_001-basic_clustering.py` pins the
  canonical values, `tests/test_002*` covers exclusive jets, `tests/test_006*`
  covers subjets/parents, `tests/test_007-general.py` covers exotic layouts,
  `tests/test_008-dask.py` covers dask-awkward, and `tests/test_010/011` cover
  graphed.
- C++ FastJet behavior is intentionally not duplicated; the external references
  are http://fastjet.fr and `docs/Awkward.rst`.

## Maintenance

When the API changes, update `api-reference.md` first, then re-check `gotchas.md`
for stale error messages. Add new recipes only when they appear in the test suite;
the recipes here should never diverge from tested behavior.
