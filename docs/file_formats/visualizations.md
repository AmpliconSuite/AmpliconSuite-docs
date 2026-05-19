# Sashimi and Cycle Visualizations

This page documents visualization conventions that users and developers should interpret consistently. These are not interchange file formats, but they are first-class outputs because they are the main way many users inspect graph structure.

Spec version: draft 0.1.

## AA Sashimi-Style Graph Plot

AmpliconArchitect's SV/sashimi-style graph plot summarizes the amplicon graph in reference coordinates. It shows coverage/copy-number segmentation, discordant read-pair clusters, and source connections. The plotted image depends on graph and copy-number data, but the image itself is not meant to be parsed as a data file.

Canonical SV colors follow the AmpliconArchitect convention:

| Color | Read-pair orientation | Interpretation |
| --- | --- | --- |
| Red | forward-reverse | Length-discordant in expected orientation; deletion-like. |
| Brown | reverse-forward | Everted orientation; duplication-like. |
| Teal | reverse-reverse | Left inversion-like. |
| Magenta | forward-forward | Right inversion-like. |

Vertical colored lines extending toward the top of the plot indicate connections to the source vertex, meaning sequence outside the amplicon interval set or an undetermined endpoint.

## Divergence

CoRAL also produces sashimi-style breakpoint graph plots, but its current color mapping is known to have drifted from the AA convention. Treat the AA color table above as canonical until CoRAL's color bug is fixed.

## CoRAL Breakpoint Graph Plots

CoRAL graph plots visualize long-read breakpoint graphs. They should be interpreted using the graph-file semantics in [graph.md](graph.md): sequence edges are genomic segments, concordant edges connect consecutive reference segments, discordant edges connect nonconsecutive segments, and path constraints represent long-read-supported subpaths. The CoRAL wiki documents the same graph concepts in its [Breakpoint Graphs section](https://github.com/AmpliconSuite/CoRAL/wiki#breakpoint-graphs).

When path constraints are shown or summarized alongside a plot, the edge prefixes follow the CoRAL convention:

| Prefix | Edge type |
| --- | --- |
| `e` | sequence edge |
| `c` | concordant edge |
| `d` | discordant edge |

## CycleViz and CoRAL Cycle Visualization

CycleViz and CoRAL cycle plotting display segment walks from cycles files. The important interpretation rules come from [cycles.md](cycles.md):

| Visual element | Format concept |
| --- | --- |
| Ordered segment blocks | The `Segments=` walk. |
| Segment direction | The `+` or `-` orientation suffix. |
| Linear endpoint/source marker | Segment `0` in the walk, commonly `0+ ... 0-`. |
| Cycle/path weight | `Copy_count`. |
| Long-read support annotations | CoRAL `Path_constraints_satisfied` and path-constraint sections. |

Cycle visualizations are renderings of the emitted cycle/path records. They should not be interpreted as proof that a displayed walk is a complete biological ecDNA unless the producing tool and downstream classification support that conclusion.

## Tools Producing or Consuming These Visualizations

| Tool | Role |
| --- | --- |
| AmpliconArchitect | Produces canonical SV/sashimi-style graph plots and cycle-view outputs. |
| CoRAL | Produces breakpoint graph plots and cycle/path plots for long-read reconstructions. |
| CycleViz | Consumes cycles files, graph-derived tracks, and optional feature tracks to render cycle structures. |
| BFBArchitect | Produces foldback-oriented visualization for BFB reconstructions. |
| AmpliconSuiteAggregator | Preserves plot images when packaging AA results. |
