# ShipNET

Network-based models for early-stage ship distributed-system design: generating
spatial arrangements of components, routing the systems that connect them, and
scoring the results with graph and current-flow metrics.

The repository is a research codebase — a set of library modules driven by
Jupyter notebooks that produce the analyses and figures. It is not a packaged
library and has no installer or test suite.

## Requirements

**This code targets Python 2.7 and networkx 1.x. It does not run on Python 3.**

Every notebook records `kernelspec: python2` / `language_info.version: 2.7.11`,
and the modules use Python 2 syntax and APIs that were removed in Python 3
(`xrange`, `dict.iteritems()`, tuple-unpacking parameters) alongside networkx 1.x
graph attribute access (`G.node[...]`, `G.edge[...]`, removed in networkx 2.4).

Dependencies: `networkx` (1.x), `numpy` and `matplotlib` throughout; `pandas`
additionally for `shipnet*.py`, `scipy` additionally for `ANCR*.py`.

See [Modernization](#modernization) below before attempting to run any of this on
a current interpreter.

## Layout

### `shipnet*.py` — arrangement and affordance routing

Defines `ShipNET` and `MultiPlex` classes covering disjoint-set generation,
affordance routing, k-shortest-paths, and node/affordance complexity metrics.

| Module | Used by |
| --- | --- |
| `shipnetv1.py` | `complexity_sweep.ipynb` |
| `shipnet_randv1.py` | — (intermediate revision) |
| `shipnet_randv1_1.py` | `NEJ Analysis.ipynb`, `Thesis analysis.ipynb`, `complexity_sweep_rand.ipynb`, `perm sweep analysis.ipynb` |

### `ANCR*.py` — arrangement and connection routing

A functional module for probabilistic arrangement/routing (`i_arrange`,
`i_route`, `i_ANCR`, `i_mapping`), current-flow computation (`current_st`,
`project_current_distribution_bus`), and the 3-D plotting routines behind the
figures.

| Module | Used by |
| --- | --- |
| `ANCR.py` | `ANCR_Routing.ipynb`, `ANCR_Systems.ipynb` |
| `ANCR_v2.py` | `ANCR Compiling.ipynb` |
| `ANCR_v3.py` | — (intermediate revision) |
| `ANCR_v4.py` | `ANCR Paper.ipynb` |

`ANCR_v4.py` is the most recent revision and the one the paper notebook imports.
The earlier versions are kept because the notebooks above still import them by
name — they are not redundant copies, and deleting one breaks the notebook that
pins it.

### Notebooks

The remaining notebooks do not import the library modules — they define whatever
they need inline. They vary widely in weight, so check one before assuming it is
a finished analysis:

- **Standalone explorations**, the substantial ones, several defining their own
  helper functions inline: `ANCR Testing`, `ANCR Arrange`, `K path`,
  `Concept_SBD`, `Current Flow`, `affordance routing`, `disjoint routing`,
  `disjoint single locations`, `incidence simulation`, `multiplex affordance`,
  `ANCR_Resistance`, `Mapping Testing`.
- **Scratch notebooks**, a handful of cells with no definitions:
  `network structure` (2 cells), `information dual` (6 cells).

22 of the 23 notebooks have their outputs stored in the repository. Treat those
outputs as the record of the results rather than as noise to be stripped — they
are the only reference available for checking a re-run, and most of the
repository's size is the embedded figures.

`Figs.pptx` and the `.PNG`/`.png` files are figure sources and diagrams.

## Modernization

Porting this to Python 3 and networkx 2/3 is a migration, not a cleanup: the
numerical results back a paper, so any port needs the original outputs as a
reference to diff against. The mechanical parts (`xrange` → `range`,
`.iteritems()` → `.items()`, `G.node[n]` → `G.nodes[n]`, `G.edge[u][v]` →
`G.edges[u, v]`) are unambiguous, but the library upgrade spans behavioral
changes in networkx's centrality and matrix routines that the mechanical
rewrites will not surface. Validate against the committed notebook outputs
before trusting a ported run.
