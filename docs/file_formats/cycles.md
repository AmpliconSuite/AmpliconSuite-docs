# Cycles File

The cycles file is a text file named like `{out}_amplicon{id}_cycles.txt`. It lists amplicon intervals, graph segments, and decomposed structures as segment walks. AmpliconArchitect's output is the baseline format.

File structure

```
Interval    <interval_id>    <chrom>    <start>    <end>
...
List of cycle segments
Segment     <segment_id>     <chrom>    <start>    <end>
...
Cycle=<cycle_id>;Copy_count=<copy_count>;Segments=<segment_id><orientation>,...
```

The section label `List of cycle segments` is conventional. Most parsers identify records by line prefixes and do not require the label.

Interval records

| Field | Type | Meaning |
| --- | --- | --- |
| `Interval` | literal | Record keyword. |
| `interval_id` | integer | 1-based interval identifier. |
| `chrom` | string | Chromosome or contig name. |
| `start` | integer | Coordinate of the first basepair. |
| `end` | integer | Coordinate of the last basepair. |

Segment records

| Field | Type | Meaning |
| --- | --- | --- |
| `Segment` | literal | Record keyword. |
| `segment_id` | integer | 1-based segment identifier. Segment `0` is reserved and is not listed as a normal segment. |
| `chrom` | string | Chromosome or contig name. |
| `start` | integer | Coordinate of the first basepair. |
| `end` | integer | Coordinate of the last basepair. |

Cycle records

Cycle lines are semicolon-separated key/value fields. Baseline AA fields are:

| Field | Type | Meaning |
| --- | --- | --- |
| `Cycle` | integer or string identifier | Cycle ID. AA writes cycles sorted by decreasing copy count. |
| `Copy_count` | float | Copy count of the cycle. |
| `Segments` | comma-separated oriented segment IDs | The segment walk. Each item is `<segment_id>+` or `<segment_id>-`. |

The last listed segment connects back to the first listed segment for circular cycles.

Segment orientation

`+` means the structure traverses the segment from its `start` coordinate to its `end` coordinate. `-` means the structure traverses the segment from `end` to `start`.

Segment 0 convention

Segment ID `0` is reserved for the source/connection vertex. AA documents that a cycle may contain segment `0` either zero times or two times. A line containing `0` represents a linear path whose endpoints connect to undetermined or out-of-interval positions. AA examples and CoRAL path output commonly use `0+` at the beginning and `0-` at the end.

Divergence

ecSimulator currently writes non-circular structures as `0+, ..., 0+`, while CoRAL writes paths as `0+, ..., 0-`. AA's README says segment `0` may occur two times but does not canonize the sign pair. Consumers should treat any `0` item as a source/path sentinel unless they need stricter path orientation semantics.

Informal grammar

```
cycles_file     ::= interval_line* segment_section segment_line+ extension_section* structure_line+
interval_line   ::= "Interval" TAB id TAB chrom TAB integer TAB integer
segment_section ::= text line, conventionally "List of cycle segments"
segment_line    ::= "Segment" TAB id TAB chrom TAB integer TAB integer
structure_line  ::= cycle_line | extension_path_line
cycle_line      ::= "Cycle=" id ";" "Copy_count=" number ";" "Segments=" segment_walk extension_field*
segment_walk    ::= oriented_segment ("," oriented_segment)*
oriented_segment::= integer ("+" | "-")
extension_field ::= ";" key "=" value
```

Worked example

```
Interval	1	chr1	100	399
List of cycle segments
Segment	1	chr1	100	199
Segment	2	chr1	300	399
Cycle=1;Copy_count=8.5;Segments=1+,2-
Cycle=2;Copy_count=1.0;Segments=0+,1+,0-
```

Tool-specific extensions

CoRAL cycles

CoRAL writes AA-like intervals and segments, optional subpath-constraint sections, then `Cycle=` and `Path=` structures. `Path=` records have the same semicolon key/value style as cycle records and use `Segments=0+,<walk>,0-`. CoRAL can append `Path_constraints_satisfied=<comma-separated ids>`.

```
List of longest subpath constraints
Path constraint	1	e1+:1,d1+:1,e2+:1	Support=12	Satisfied
List of extracted cycles/paths
Cycle=1;Copy_count=10.0;Segments=1+,2+;Path_constraints_satisfied=1
Path=2;Copy_count=2.0;Segments=0+,1-,0-;Path_constraints_satisfied=
```

CycleExtractor cycles

CycleExtractor writes extracted cycles and paths in a CoRAL-like form. Some writer paths omit `Interval` lines and begin at `List of cycle segments`; this means consumers that require intervals must derive them from segments or the original graph. CycleExtractor's constraint records may use `Path constraint` with a space instead of the graph file keyword `path_constraint`.

BFBArchitect cycles

BFBArchitect writes intervals and segments, then a `List of longest subpath constraints` label followed by `Path=` lines, not `Cycle=` lines. Path lines may append `Score=<value>` and `Multiplicity=<value>`. These fields are BFBArchitect-specific and are not baseline cycle fields.

AmpliconClassifier annotated cycles

AmpliconClassifier writes corrected/annotated cycle files with additional semicolon fields before `Segments`: `Length`, `IsCyclicPath`, and `CycleClass`. Consumers should preserve unknown key/value fields when rewriting.

AmpliconReconstructorOM path cycles

AmpliconReconstructorOM writes AA-like cycles for optical-map paths with an additional `Contigs=<comma-separated contig ids>` field.

ecSimulator cycles

ecSimulator writes a comment header, baseline intervals/segments, and `Cycle=1;Copy_count=1.0;Length=<bp>;Segments=<walk>`. The `Length` field is an extension.

Fully resolved cyclic structures

CycleExtractor's output is intended to represent extracted closed walks and paths from an input amplicon graph. Its closed structures are still serialized as `Cycle=` with segment orientations; its linear structures are serialized as `Path=` or as segment walks containing source segment `0`, depending on the writer path used. The file format alone does not prove biological completeness; it records the structure the tool emitted.

Edge cases

Cycle lines may include extra semicolon fields. The most compatible parser reads the fields into a key/value map, requires `Copy_count` and `Segments` for baseline structures, and preserves unrecognized fields. Segment IDs should be parsed by removing the final orientation character, not by assuming a fixed number of digits. Several consumers ignore `Interval` records entirely.

Parsers and writers in the wild

| Repo | Path | Function or code |
| --- | --- | --- |
| AmpliconArchitect | `src/breakpoint_graph.py:590` | `cycle_decomposition.__init__()` parses cycles |
| AmpliconArchitect | `src/breakpoint_graph.py:815` | `cycle_decomposition.__str__()` writes cycles |
| AmpliconArchitect | `src/breakpoint_graph.py:578` | cycle logger writes `Cycle=` lines |
| AmpliconArchitect | `src/cycle_visualization/draw_episome.py:79` | visualization parser for `Cycle=` |
| AmpliconClassifier | `ampclasslib/ac_util.py:372` | `parseCycle()` |
| AmpliconClassifier | `ampclasslib/ac_io.py:323` | `write_annotated_corrected_cycles_file()` |
| AmpliconClassifier | `ampclasslib/convert_cycles_file.py:65` | `make_new_cycle()` |
| CoRAL | `coral/output/cycle_output.py:24` | `output_cycles()` |
| CoRAL | `coral/output/utils.py:36` | `get_single_cycle_str()` |
| CoRAL | `coral/output/utils.py:66` | `get_single_path_str()` |
| CoRAL | `coral/scoring/io_utils.py:17` | `read_cycles_intervals_to_bed()` |
| CoRAL | `coral/scoring/io_utils.py:47` | `read_cycles_file_to_bed()` |
| CoRAL | `coral/cycle2bed.py:13` | `convert_cycles_to_bed()` |
| CoRAL | `coral/summary/parsing.py:350` | `parse_cycle_file()` |
| BFBArchitect | `src/BFBArchitect.py:288` | `generate_cycle_file()` |
| CycleExtractor | `CE.py:1423` | cycle string formatter |
| CycleExtractor | `CE.py:1567` | `write_ILP_cycle_file()` |
| CycleExtractor | `CE.py:2531` | `write_paths_in_ILP_cycle_file()` |
| CycleExtractor | `CE.py:2644` | `write_all_cycles_and_paths()` |
| CycleViz | `VizUtil.py:325` | `parse_cycles_file()` |
| CycleViz | `convert_cycles_file.py:51` | `make_new_cycle()` |
| ecSimulator | `src/ecSim_IO.py:98` | `write_cycles_file()` |
| ecSimulator | `src/cycles_file_to_fasta.py:36` | `writeCycleFasta()` consumes cycles |
| AmpliconReconstructorOM | `breakpoint_graph.py:678` | cycle parser |
| AmpliconReconstructorOM | `breakpoint_graph.py:909` | cycle string writer |
| AmpliconReconstructorOM | `OMPathFinder.py:910` | `write_path_cycles()` |
| AmpliconReconstructorOM | `align_to_candidate_path.py:99` | `get_cycles_from_cnums()` |
| AmpliconReconstructorOM | `AmpliconReconstructorOM.py:57` | `parse_cycles_file()` |
| AmpliconSuiteAggregator | `src/asa_aggregator.py:50` | preserves `_cycles.txt` in AA result packaging |
