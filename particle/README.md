# particle library-usage skill

This folder is a practical, agent-oriented guide to **using** the Scikit-HEP
`particle` library from Python code. It is not a guide to developing,
maintaining, or contributing to the library.

## Purpose

The skill answers questions like "how do I look up a particle?", "what is the
PDG ID of the B+?", "how do I convert a Pythia ID to a PDG ID?", or "what does
`particle` return for an unknown mass?" — with verified APIs, behavior notes,
and runnable examples.

## Folder structure

```
particle/
├── references/
│   ├── pdgid.md                  # PDGID queries, classification, validity, antiparticles
│   ├── particle-properties.md    # Particle lookups, properties, searches, literals, custom particles
│   ├── quantum-numbers.md        # charge, spin, parity, isospin, flavour numbers, enums
│   └── interoperability.md       # MC IDs, EvtGen/LHCb names, hepunits, pandas/tabulate
├── SKILL.md                      # Entry point: overview, when to use, core concepts, gotchas, links
└── README.md                     # This file
```

## Role of each file

- **`SKILL.md`** — concise entry point: library overview, when to reach for the
  library, core concepts, essential usage patterns, major gotchas, and links to
  the detailed references. Read this first.
- **`references/pdgid.md`** — everything about `PDGID` and `particle.pdgid`:
  what a PDG ID encodes, classification predicates, validity, antiparticles,
  and the free-standing functions.
- **`references/particle-properties.md`** — everything about the `Particle`
  record: lookups, properties, searches, literals, name conventions,
  mass/width/lifetime, custom data tables, and user-defined particles.
- **`references/quantum-numbers.md`** — quantum numbers and classification:
  charge, spin (`J`, `L`, `S`, `spin_type`), parity (`P`, `C`, `G`), isospin
  (`I`), flavour numbers, baryon/lepton numbers, and the `Charge`, `Parity`,
  `SpinType`, `Inv`, `Status` enums.
- **`references/interoperability.md`** — integrations: `PythiaID`, `Geant3ID`,
  `Corsika7ID`, custom MC ID classes, EvtGen/LHCb names, `hepunits` quantities,
  pandas/tabulate.

Each reference file is self-contained: read only the ones relevant to your task.
All technical claims and examples were verified against the library's source
code and bundled data.
