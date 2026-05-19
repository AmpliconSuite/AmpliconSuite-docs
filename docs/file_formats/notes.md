# File Format Spec Notes

## Decisions To Make

1. Canonize a source vertex spelling for graph files. AA documents `POS=-1`; CoRAL writes `0-->chr:pos`.
2. Decide whether long-read graph headers (`PredictedCN`, `NumberOfLongReads`) are a named subtype or accepted aliases in the core graph spec.
3. Decide whether CoRAL/CycleExtractor `PathConstraint` sections belong in the canonical graph and cycles formats or remain tool-specific extensions.
4. Decide whether `Path=` records are first-class structures in cycles files or an extension parallel to AA's `Cycle=` records.
5. Decide the canonical signs for source segment `0` in linear paths. CoRAL uses `0+ ... 0-`; ecSimulator currently writes `0+ ... 0+`.
6. Decide whether cycle extension fields such as `Length`, `IsCyclicPath`, `CycleClass`, `Contigs`, `Score`, and `Multiplicity` should be registered names with stable semantics.
7. Decide whether sequence-edge `Size` is inclusive (`end - start + 1`) or AA-style (`end - start`) and document any historical compatibility rule.
8. Decide whether an AA copy-number bedGraph is a canonical emitted format. The audited AA README documents `_cnseg.txt`; bedGraph output appears in utility and visualization code.
9. Decide whether `_amplicon_intervals.bed` is a canonical AmpliconSuite format or whether the cycles `Interval` section plus AC classification `_intervals.bed` files are the only documented interval formats.
10. Revisit versioning once the first downstream tool links here. The current draft uses repo-level documentation versioning plus per-page spec notes only when needed.

## Repeatable Audit Note

Keep upstream source checkouts temporary. A repeat audit can clone the AmpliconSuite repositories under `/tmp/as-format-audit`, then run:

```
rg -n "_graph\.txt|_cycles\.txt|SequenceEdge|BreakpointEdge|Cycle=|path_constraint|bedGraph|_intervals\.bed" /tmp/as-format-audit -g '!**/.git/**'
rg -n "def .*cycle|def .*graph|write.*cycle|read.*cycle|write.*graph|read.*graph|class .*Graph" /tmp/as-format-audit -g '!**/.git/**'
```

Do not modify upstream repositories during a spec audit. Record the exact upstream commit SHAs in future updates if the spec starts tracking versioned behavior.

## Implementation Audit Index

This section is intentionally internal. Public spec pages list tools by role, not function names.

Graph readers and writers checked in the initial audit:

| Repo | Path | Function or code |
| --- | --- | --- |
| AmpliconArchitect | `src/bam_to_breakpoint.py:2416` | graph writer in breakpoint graph optimization output |
| AmpliconArchitect | `src/breakpoint_graph.py:276` | `breakpoint_graph.load_graphfile()` |
| CoRAL | `coral/breakpoint/breakpoint_utilities.py:682` | `output_breakpoint_graph_lr()` |
| CoRAL | `coral/breakpoint/parse_graph.py:141` | `parse_breakpoint_graph()` |
| AmpliconClassifier | `ampclasslib/ac_util.py:431` | `get_graph_cns()` |
| BFBArchitect | `src/BFBArchitect.py:260` | `generate_graph_file()` |
| CycleExtractor | `CE.py:18`, `CE.py:257` | graph parsing and graph construction |
| AmpliconReconstructorOM | `breakpoint_graph.py:280`, `OMPathFinder.py:431`, `generate_cmap.py:43` | graph parsing and copy-count lookup |
| ecSimulator | `src/ecSim_IO.py:118` | `write_bpg_file()` |
| AmpliconSuiteAggregator | `src/asa_aggregator.py:50` | preserves `_graph.txt` |

Cycles readers and writers checked in the initial audit:

| Repo | Path | Function or code |
| --- | --- | --- |
| AmpliconArchitect | `src/breakpoint_graph.py:590`, `src/breakpoint_graph.py:815` | cycle parser and string writer |
| AmpliconClassifier | `ampclasslib/ac_util.py:372`, `ampclasslib/ac_io.py:323` | cycle parser and annotated writer |
| CoRAL | `coral/output/cycle_output.py:24`, `coral/output/utils.py:36`, `coral/output/utils.py:66` | cycle/path writers |
| BFBArchitect | `src/BFBArchitect.py:288` | BFB path-style cycles writer |
| CycleExtractor | `CE.py:1567`, `CE.py:2531`, `CE.py:2644` | cycles/path writers |
| CycleViz | `VizUtil.py:325`, `convert_cycles_file.py:51` | cycles parser and converter |
| ecSimulator | `src/ecSim_IO.py:98` | simulated cycles writer |
| AmpliconReconstructorOM | `breakpoint_graph.py:678`, `OMPathFinder.py:910` | cycle parser and path cycles writer |
