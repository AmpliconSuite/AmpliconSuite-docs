# Graph File

The graph file is a tab-separated text file named like `{out}_amplicon{id}_graph.txt`. It represents an amplicon breakpoint graph with genomic sequence edges and breakpoint edges. AmpliconArchitect's AA graph is the baseline format.

File structure

```
SequenceEdge: StartPosition, EndPosition, PredictedCopyCount, AverageCoverage, Size, NumberReadsMapped
sequence    <start_vertex>    <end_vertex>    <copy_number>    <average_coverage>    <size>    <read_count>
...
BreakpointEdge: StartPosition->EndPosition, PredictedCopyCount, NumberOfReadPairs, HomologySizeIfAvailable(<0ForInsertions), Homology/InsertionSequence
concordant  <vertex1>-><vertex2>    <copy_number>    <support>    <homology_size_or_None>    <homology_or_insertion_sequence_or_None>
discordant  <vertex1>-><vertex2>    <copy_number>    <support>    <homology_size_or_None>    <homology_or_insertion_sequence_or_None>
source      <source_vertex>-><vertex>    <copy_number>    <support>    <homology_size_or_None>    <homology_or_insertion_sequence_or_None>
```

Vertex syntax

| Element | Syntax | Meaning |
| --- | --- | --- |
| Genomic vertex | `{CHROM}:{POS}{STRAND}` | A breakpoint vertex at a genomic coordinate. |
| `CHROM` | string | Chromosome or contig name. |
| `POS` | integer | AA documents this as a 0-based coordinate where the base at the coordinate is included in the participating segment. |
| `STRAND` | `+` or `-` | `+` indicates the participating segment ends on the right of the basepair; `-` indicates it begins on the left. |
| Source vertex | `*-1*` in AA docs; `0-` appears in CoRAL output | Represents unknown sequence or sequence outside the amplicon interval set. |

Divergence

The AA README describes unknown/outside positions as `POS=-1` on a breakpoint vertex. CoRAL writes source edges as `source 0--><chr>:<pos><strand>`. AA's `breakpoint_graph.load_graphfile()` treats `source` like a discordant edge internally. Consumers should not assume a single serialized source spelling until the project canonizes one.

Sequence edge fields

| Field | Type | Meaning |
| --- | --- | --- |
| `sequence` | literal | Record keyword. |
| `StartPosition` | vertex | First base of the segment. Commonly written with `-` strand. |
| `EndPosition` | vertex | Last base of the segment. Commonly written with `+` strand. |
| `PredictedCopyCount` | float | Predicted segment copy number. |
| `AverageCoverage` | float | Average depth over the segment. |
| `Size` | integer | Segment length in basepairs. AA writes `end - start`; several other tools write inclusive size `end - start + 1`. |
| `NumberReadsMapped` | integer or float-like integer | Number of reads aligned to the segment. |

Breakpoint edge fields

| Field | Type | Meaning |
| --- | --- | --- |
| `concordant`, `discordant`, `source` | literal | Edge category. |
| `StartPosition->EndPosition` | vertex pair | Two endpoint vertices joined by literal `->`. |
| `PredictedCopyCount` | float | Predicted copy number of the breakpoint edge. |
| `NumberOfReadPairs` | integer or float-like integer | Short-read discordant pair support in AA. For long-read tools, support is long-read count. |
| `HomologySizeIfAvailable(<0ForInsertions)` | integer, `None`, or empty | Homology length at the breakpoint. Negative values indicate insertion length. |
| `Homology/InsertionSequence` | string, `None`, or empty | Homology or inserted sequence. |

Edge categories

`sequence` edges are non-overlapping genomic segments that cover the intervals in the amplicon. `concordant` edges connect adjacent positions already connected in the reference genome, typically `chr:x+->chr:x+1-`. `discordant` edges connect non-reference breakpoint vertices. `source` edges connect known sequence to unknown or out-of-interval sequence.

Informal grammar

```
graph_file       ::= sequence_header sequence_line+ breakpoint_header breakpoint_line+ extension_section*
sequence_header  ::= "SequenceEdge:" text
breakpoint_header::= "BreakpointEdge:" text
sequence_line    ::= "sequence" TAB vertex TAB vertex TAB number TAB number TAB integer TAB number
breakpoint_line  ::= edge_type TAB vertex_pair TAB number TAB number (TAB homology_size TAB homology_seq)?
edge_type        ::= "concordant" | "discordant" | "source"
vertex_pair      ::= vertex "->" vertex
vertex           ::= chrom ":" integer strand | source_token
strand           ::= "+" | "-"
```

Worked example

```
SequenceEdge: StartPosition, EndPosition, PredictedCopyCount, AverageCoverage, Size, NumberReadsMapped
sequence	chr1:100-	chr1:199+	12.0	36.5	99	220
sequence	chr1:300-	chr1:399+	10.0	31.2	99	190
BreakpointEdge: StartPosition->EndPosition, PredictedCopyCount, NumberOfReadPairs, HomologySizeIfAvailable(<0ForInsertions), Homology/InsertionSequence
concordant	chr1:199+->chr1:200-	2.0	4	None	None
discordant	chr1:399+->chr1:100-	9.5	18	0
source	chr1:100-->chr1:-1+	1.0	2	None	None
```

Long-read graph extension

CoRAL, CycleExtractor sample data, and BFBArchitect use a four-column breakpoint-edge body and headers like:

```
SequenceEdge: StartPosition, EndPosition, PredictedCN, AverageCoverage, Size, NumberOfLongReads
BreakpointEdge: StartPosition->EndPosition, PredictedCN, NumberOfLongReads
```

The records keep the same first fields, but the support column is long-read support and homology/insertion fields are absent. CoRAL may append:

```
PathConstraint: Path, Support
path_constraint	<edge_path>	<support>
```

The path string uses edge IDs with type prefixes (`e`, `c`, `d`) and orientations/counts such as `e6-:1,c5+:1`. This is a CoRAL/CycleExtractor extension, not baseline AA graph format.

Edge cases

Parsers in several tools split on arbitrary whitespace, so tabs are expected for writers but spaces often work for readers. CoRAL explicitly accepts float-like support values for older graphs and converts them to integers. Some older simulations had an extra empty field in breakpoint records; CoRAL removes it while parsing. Header spelling is not reliable enough to identify the subtype; parse record keywords and field counts.

Parsers and writers in the wild

| Repo | Path | Function or code |
| --- | --- | --- |
| AmpliconArchitect | `src/bam_to_breakpoint.py:2416` | graph writer in breakpoint graph optimization output |
| AmpliconArchitect | `src/breakpoint_graph.py:276` | `breakpoint_graph.load_graphfile()` |
| CoRAL | `coral/breakpoint/breakpoint_utilities.py:682` | `output_breakpoint_graph_lr()` |
| CoRAL | `coral/breakpoint/parse_graph.py:31` | `parse_sequence_edge()` |
| CoRAL | `coral/breakpoint/parse_graph.py:43` | `parse_concordant_edge()` |
| CoRAL | `coral/breakpoint/parse_graph.py:66` | `parse_discordant_edge()` |
| CoRAL | `coral/breakpoint/parse_graph.py:141` | `parse_breakpoint_graph()` |
| AmpliconClassifier | `amplicon_classifier.py:65` | `tot_rearr_edges()` reads breakpoint lines |
| AmpliconClassifier | `amplicon_classifier.py:188` | `compute_f_from_AA_graph()` reads sequence and foldback features |
| AmpliconClassifier | `ampclasslib/ac_util.py:431` | `get_graph_cns()` |
| AmpliconClassifier | `ampclasslib/ac_annotation.py:6` | `summarize_breakpoints()` |
| BFBArchitect | `src/BFBArchitect.py:260` | `generate_graph_file()` |
| CycleExtractor | `CE.py:18` | `parse_graph_file()` for segments and path constraints |
| CycleExtractor | `CE.py:257` | `Create_graph()` |
| AmpliconReconstructorOM | `breakpoint_graph.py:280` | `breakpoint_graph.load_graphfile()` |
| AmpliconReconstructorOM | `OMPathFinder.py:431` | `get_edge_copy_counts()` |
| AmpliconReconstructorOM | `generate_cmap.py:43` | `read_graph()` |
| AmpliconReconstructorOM | `ARAlignDetect.py:170` | `rewrite_graph_and_CMAP()` |
| AmpliconReconstructorOM | `graph_to_bed.py:22` | top-level graph-to-BED reader |
| ecSimulator | `src/ecSim_IO.py:118` | `write_bpg_file()` |
| AmpliconSuiteAggregator | `src/asa_aggregator.py:50` | preserves `_graph.txt` in AA result packaging |
