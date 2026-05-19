# Graph File

The graph file is a tab-separated text file named like `{out}_amplicon{id}_graph.txt`. It represents an amplicon breakpoint graph with genomic sequence edges and breakpoint edges. AmpliconArchitect's AA graph is the baseline format.

Spec version: draft 0.1.

## File Structure

```
<optional comment/header line>
SequenceEdge: StartPosition, EndPosition, PredictedCopyCount, AverageCoverage, Size, NumberReadsMapped
sequence    <start_vertex>    <end_vertex>    <copy_number>    <average_coverage>    <size>    <read_count>
...
BreakpointEdge: StartPosition->EndPosition, PredictedCopyCount, NumberOfReadPairs, HomologySizeIfAvailable(<0ForInsertions), Homology/InsertionSequence
concordant  <vertex1>-><vertex2>    <copy_number>    <support>    <homology_size_or_None>    <homology_or_insertion_sequence_or_None>
discordant  <vertex1>-><vertex2>    <copy_number>    <support>    <homology_size_or_None>    <homology_or_insertion_sequence_or_None>
source      <source_vertex>-><vertex>    <copy_number>    <support>    <homology_size_or_None>    <homology_or_insertion_sequence_or_None>
```

A file may begin with one or more comment or metadata lines, commonly beginning with `#`; ecSimulator uses this pattern. The `SequenceEdge:` and `BreakpointEdge:` header lines are expected in normal graph files, but several consumers parse by record keyword rather than by validating header text.

## Vertex Syntax

| Element | Syntax | Meaning |
| --- | --- | --- |
| Genomic vertex | `{CHROM}:{POS}{STRAND}` | A breakpoint vertex at a genomic coordinate. |
| `CHROM` | string | Chromosome or contig name. |
| `POS` | integer | AA documents this as a 0-based coordinate where the base at the coordinate is included in the participating segment. |
| `STRAND` | `+` or `-` | `+` indicates the participating segment ends on the right of the basepair; `-` indicates it begins on the left. |
| Source vertex | `{CHROM}:-1{STRAND}` in AA docs; `0-` appears in CoRAL output | Represents unknown sequence or sequence outside the amplicon interval set. |

## Divergence

The AA README describes unknown/outside positions as `POS=-1` on a breakpoint vertex. CoRAL writes source edges as `source 0--><chr>:<pos><strand>`.

## Sequence Edge Fields

| Field | Type | Meaning |
| --- | --- | --- |
| `sequence` | literal | Record keyword. |
| `StartPosition` | vertex | First base of the segment. Commonly written with `-` strand. |
| `EndPosition` | vertex | Last base of the segment. Commonly written with `+` strand. |
| `PredictedCopyCount` | float | Predicted segment copy number. |
| `AverageCoverage` | float | Average depth over the segment. |
| `Size` | integer | Segment length in basepairs. AA writes `end - start`; several other tools write inclusive size `end - start + 1`. |
| `NumberReadsMapped` | integer or float-like integer | Number of reads aligned to the segment. |

## Breakpoint Edge Fields

| Field | Type | Meaning |
| --- | --- | --- |
| `concordant`, `discordant`, `source` | literal | Edge category. |
| `StartPosition->EndPosition` | vertex pair | Two endpoint vertices joined by literal `->`. |
| `PredictedCopyCount` | float | Predicted copy number of the breakpoint edge. |
| `NumberOfReadPairs` | integer or float-like integer | Short-read discordant pair support in AA. For long-read tools, support is long-read count. |
| `HomologySizeIfAvailable(<0ForInsertions)` | integer, `None`, or empty | Homology length at the breakpoint. Negative values indicate insertion length. `None` means split-read homology/insertion was not detected or not available. `0` means the breakpoint was checked and no homology/insertion was found. |
| `Homology/InsertionSequence` | string, `None`, or empty | Homology or inserted sequence. `None` means sequence was not available. Empty string with size `0` means a blunt junction. |

## Edge Categories

`sequence` edges are non-overlapping genomic segments that cover the intervals in the amplicon. `concordant` edges connect adjacent positions already connected in the reference genome, typically `chr:x+->chr:x+1-`. `discordant` edges connect non-reference breakpoint vertices. `source` edges connect known sequence to unknown or out-of-interval sequence.

## Informal Grammar

```
graph_file       ::= header_line* sequence_header sequence_line+ breakpoint_header breakpoint_line+ extension_section*
header_line      ::= "#" text | other_metadata_line
sequence_header  ::= "SequenceEdge:" text
breakpoint_header::= "BreakpointEdge:" text
sequence_line    ::= "sequence" TAB vertex TAB vertex TAB number TAB number TAB integer TAB number
breakpoint_line  ::= edge_type TAB vertex_pair TAB number TAB number (TAB homology_size TAB homology_seq)?
edge_type        ::= "concordant" | "discordant" | "source"
vertex_pair      ::= vertex "->" vertex
vertex           ::= chrom ":" integer strand | source_token
strand           ::= "+" | "-"
```

## Worked Example

```
SequenceEdge: StartPosition, EndPosition, PredictedCopyCount, AverageCoverage, Size, NumberReadsMapped
sequence	chr1:100-	chr1:199+	12.0	36.5	99	220
sequence	chr1:300-	chr1:399+	10.0	31.2	99	190
BreakpointEdge: StartPosition->EndPosition, PredictedCopyCount, NumberOfReadPairs, HomologySizeIfAvailable(<0ForInsertions), Homology/InsertionSequence
concordant	chr1:199+->chr1:200-	2.0	4	None	None
discordant	chr1:399+->chr1:100-	9.5	18	0
source	chr1:100-->chr1:-1+	1.0	2	None	None
```

## Real Data Excerpt

This is a shortened CoRAL example from the [public CoRAL wiki](https://github.com/AmpliconSuite/CoRAL/wiki#breakpoint-graphs), included to show realistic long-read field names and path-constraint syntax:

```
SequenceEdge: StartPosition, EndPosition, PredictedCN, AverageCoverage, Size, NumberOfLongReads
sequence	chr7:54659673-	chr7:54763281+	4.150534	45.907363	103609	576
sequence	chr7:54763282-	chr7:55127266+	89.340352	1052.714362	363985	40637
BreakpointEdge: StartPosition->EndPosition, PredictedCN, NumberOfLongReads
concordant	chr7:54763281+->chr7:54763282-	4.150534	26
discordant	chr7:55610095-->chr7:55609190+	86.642611	869
PathConstraint: Path, Support
path_constraint	e2+:1,c2-:1,e3+:1,c3-:1,e4+:1	6
AmpliconIntervals: chr, start, end
interval	chr7	54659673	56149664
```

## Long-Read Graph Extension

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

The CoRAL wiki defines the short edge prefixes as `e` for sequence edges, `c` for concordant edges, and `d` for discordant edges. It also documents an `AmpliconIntervals` graph section. In the audited CoRAL writer, the `AmpliconIntervals` graph output was present but commented out; CoRAL still writes amplicon intervals into cycles files and summaries.

## Edge Cases

Parsers in several tools split on arbitrary whitespace, so tabs are expected for writers but spaces often work for readers. CoRAL explicitly accepts float-like support values for older graphs and converts them to integers. Some older simulations had an extra empty field in breakpoint records; CoRAL removes it while parsing. Header spelling is not reliable enough to identify the subtype; parse record keywords and field counts.

## Tools Using Graph Files

| Tool | Role |
| --- | --- |
| AmpliconArchitect | Produces baseline short-read graph files and consumes them for cycle decomposition and visualization. |
| AmpliconClassifier | Consumes AA graph files for classification features, breakpoint summaries, and copy-number lookup. |
| CoRAL | Produces and consumes long-read AA-like graph files with path constraints. |
| CycleExtractor | Consumes AA/CoRAL graph files to extract cycles and paths. |
| BFBArchitect | Produces CoRAL-like graph files with long-read support. |
| AmpliconReconstructorOM | Consumes AA graph files and can rewrite graph/CMAP-derived representations. |
| ecSimulator | Produces simulated AA-like graph files. |
| AmpliconSuiteAggregator | Preserves graph files when packaging results. |

Implementation-level locations are tracked in [notes.md](notes.md) for maintainers.
