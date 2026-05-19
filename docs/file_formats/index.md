# AmpliconSuite File Formats

This directory documents the text formats that originated in AmpliconArchitect and are now produced or consumed across AmpliconSuite tools. The two first-class exchange formats are the graph file (`*_graph.txt`) and the cycles file (`*_cycles.txt`). The AA sashimi/SV plot is also a first-class user-facing output, but it is an image/report artifact rather than a machine-readable interchange format and is not specified here beyond its dependency on the graph and copy-number data.

History

AmpliconArchitect defined a breakpoint graph representation for focal amplifications and wrote one graph file plus one cycles file per amplicon. The current AA README describes these formats in its file formats section and remains the starting point for this specification. Long-read and downstream tools now use the same core records but add fields or sections for path constraints, paths, annotations, optical-map contigs, and BFB-specific scores.

Data flow

```
          WGS / long-read / OM inputs
                    |
        +-----------+-----------+
        |                       |
 AmpliconArchitect          CoRAL / BFBArchitect / ecSimulator
        |                       |
        +----- *_graph.txt -----+
                    |
          Cycle decomposition / extraction
                    |
        +----- *_cycles.txt ----+
        |
        +--> AmpliconClassifier annotations and BEDs
        +--> CycleViz figures
        +--> AmpliconReconstructorOM paths and CMAPs
        +--> ec3D-oriented interval inputs
        +--> AmpliconSuiteAggregator archival packaging
```

Tool matrix

| Tool | Graph | Cycles | Related outputs |
| --- | --- | --- | --- |
| AmpliconArchitect | writes and reads AA graph | writes and reads AA cycles | `_cnseg.txt`, SV/sashimi plot images |
| AmpliconClassifier | reads AA graph | reads AA cycles, writes annotated cycles variant | classification interval BEDs |
| CoRAL | writes and reads long-read AA-like graph | writes AA-like cycles with paths and path constraints | cycle-to-BED conversion |
| CycleExtractor | reads AA/CoRAL graph | writes extracted cycles/paths variant | metrics |
| CycleViz | reads cycles and graph-derived tracks | reads cycles; converts cycles for visualization | bedGraph/wig-like tracks |
| BFBArchitect | writes CoRAL-like graph | writes paths with BFB fields | reads/foldback visualization |
| ecSimulator | writes AA-like graph | writes AA-like cycles with `Length` | simulated reads |
| ec3D | not a graph consumer | consumes an oriented interval/structure file, not standard AA cycles directly | 3D/Hi-C matrix inputs |
| AmpliconReconstructorOM | reads AA graph, rewrites graph/CMAP | reads AA cycles and writes path cycles with `Contigs` | graph BED/bedGraph |
| AmpliconSuiteAggregator | preserves graph/cycles files | preserves graph/cycles files | packages CNV BEDs and classification BEDs |

Divergence

The shared core is keyword-oriented, not header-strict. Many parsers skip headers and accept any line starting with `sequence`, `concordant`, `discordant`, `source`, `Segment`, `Interval`, or `Cycle=`. CoRAL and BFBArchitect use `PredictedCN` and `NumberOfLongReads` headers instead of AA's `PredictedCopyCount` and `NumberOfReadPairs`. Several tools append semicolon fields to cycle records. These extensions are documented on the format pages and should not be assumed to be present in baseline AA output.

Audit method

The parser/writer survey for this document was done from scratch checkouts of the AmpliconSuite GitHub organization under `/tmp/as-format-audit`, leaving upstream repositories unmodified. To repeat it:

```
mkdir -p /tmp/as-format-audit
for repo in AmpliconArchitect AmpliconClassifier CoRAL ec3D ecSimulator BFBArchitect CycleExtractor CycleViz AmpliconReconstructorOM AmpliconSuiteAggregator
do
  git clone https://github.com/AmpliconSuite/${repo}.git /tmp/as-format-audit/${repo}
done
rg -n "_graph\.txt|_cycles\.txt|SequenceEdge|BreakpointEdge|Cycle=|path_constraint|bedGraph|_intervals\.bed" /tmp/as-format-audit -g '!**/.git/**'
rg -n "def .*cycle|def .*graph|write.*cycle|read.*cycle|write.*graph|read.*graph|class .*Graph" /tmp/as-format-audit -g '!**/.git/**'
```

Canonical pages

- [Cycles file](cycles.md)
- [Graph file](graph.md)
- [Copy-number bedGraph](bedgraph.md)
- [Amplicon intervals BED](intervals_bed.md)
- [Changelog](changelog.md)
