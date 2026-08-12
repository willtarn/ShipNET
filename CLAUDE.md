# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

A research codebase for early-stage ship distributed-system design: library
modules driven by Jupyter notebooks that produce the analyses and figures behind
a paper. It is not a package — there is no installer, no test suite, no linter
config, and no CI.

## The code does not run as committed

Everything targets **Python 2.7 and networkx 1.x**. Every notebook records
`kernelspec: python2` / `language_info.version: 2.7.11`. Do not assume a
snippet works before checking:

- The `shipnet*.py` files are not valid Python 3 at all (print statements,
  tuple-unpacking parameters) — `python3 -m py_compile` fails on them.
- The `ANCR*.py` files parse under Python 3 but fail at runtime: `xrange` and
  `dict.iteritems()` are undefined, and `G.node` / `G.edge` were removed in
  networkx 2.4.

So "it imports" proves nothing here. `ANCR*.py` will import cleanly on a modern
stack and then raise `NameError` the moment a function body executes.

## Do not delete the version-suffixed modules

`ANCR.py`, `ANCR_v2/3/4.py`, `shipnetv1.py`, `shipnet_randv1.py`, and
`shipnet_randv1_1.py` look like abandoned copies. They are not. Each is still
imported **by name** from a different notebook, so removing or consolidating one
breaks the notebook that pins it:

| Module | Imported by |
| --- | --- |
| `ANCR.py` | `ANCR_Routing.ipynb`, `ANCR_Systems.ipynb` |
| `ANCR_v2.py` | `ANCR Compiling.ipynb` |
| `ANCR_v3.py` | — (intermediate revision) |
| `ANCR_v4.py` | `ANCR Paper.ipynb` |
| `shipnetv1.py` | `complexity_sweep.ipynb` |
| `shipnet_randv1.py` | — (intermediate revision) |
| `shipnet_randv1_1.py` | `NEJ Analysis.ipynb`, `Thesis analysis.ipynb`, `complexity_sweep_rand.ipynb`, `perm sweep analysis.ipynb` |

`ANCR_v4.py` is the current revision and backs the paper notebook. The remaining
notebooks import nothing and define whatever they need inline.

The files are near-duplicates — 97–99% line similarity within each family, with
21 of 24 `shipnet` methods and 9 of 23 `ANCR` functions byte-identical across
the files defining them. That redundancy is real, but it is load-bearing.

## Textual identity does not imply shared behaviour

If you ever consolidate these, the trap is that a function can be byte-identical
in every version and still behave differently in each, because it calls
something that varies. `basic_arch` is identical across v2/v3/v4 but calls
`i_ANCR`, `plot_current`, `plot_setups`, and `project_current_distribution_bus`
— all of which differ per version. Hoisting it into a shared module silently
binds every version to one version's callees, and nothing errors.

Safe sharing requires a definition to be identical **and** to have all its
transitive local callees identical.

## Notebook outputs are the validation reference

22 of the 23 notebooks have their outputs stored, and they are the only record
of the published results. Do not strip them to shrink the diff, and treat a
re-run that changes numbers as a meaningful change rather than noise. Most of
the repository's size is embedded figures.

Notebooks run up to 2.3 MB; reading one wholesale will flood context. Extract
what you need instead:

```sh
python3 -c "
import json; d=json.load(open('ANCR Paper.ipynb'))
for i,c in enumerate(d['cells']):
    if c['cell_type']=='code': print(i, ''.join(c['source'])[:200])
"
```

## Dependencies

`networkx` (1.x), `numpy`, and `matplotlib` throughout; `pandas` additionally
for `shipnet*.py`, `scipy` additionally for `ANCR*.py`. There is no
`requirements.txt` — the versions are not pinned anywhere in the repo.

## If asked to port to Python 3

This is a migration, not a cleanup, and the numbers back a paper. Validate any
port against the committed notebook outputs before trusting it. Beyond the
mechanical `2to3` changes, the networkx upgrade is where the risk concentrates:

- `G.node[n]` → `G.nodes[n]`, `G.edge[u][v]` → `G.edges[u, v]`
- `G.nodes() + G.edges()` — these were lists and are views now, so `+` fails
- `add_node(n, attr_dict)` → `add_node(n, **attr)` — positional dicts removed
- `set_node_attributes(G, name, values)` → `(G, values, name)` — **the arguments
  were swapped in 2.0.** The old order raises nothing; it quietly writes the
  wrong thing and surfaces much later as a `KeyError` on an attribute that was
  never set. 12 call sites, all in `ANCR.py`/`ANCR_v2.py`/`ANCR_v3.py`.
- `flow_matrix_row` may orient `(s, t)` opposite to how `H.edges()` stored the
  edge, which `KeyError`s in `current_st`. The callers already tolerate either
  orientation (`i_dist` tests both `ele` and `ele[::-1]`), so follow that
  convention rather than inventing one.

Dict and set iteration order also differs between 2.7 and 3.x, which alone can
change output wherever the algorithms iterate over graph elements to make greedy
or randomised choices — no bug required.
