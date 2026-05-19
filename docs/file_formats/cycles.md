# Cycles File

The cycles file is a text file named like `{out}_amplicon{id}_cycles.txt`. It lists amplicon intervals, graph segments, and decomposed structures as segment walks. AmpliconArchitect's output is the baseline format.

Spec version: draft 0.1.

## File Structure

```
<optional comment/header line>
Interval    <interval_id>    <chrom>    <start>    <end>
...
List of cycle segments
Segment     <segment_id>     <chrom>    <start>    <end>
...
Cycle=<cycle_id>;Copy_count=<copy_count>;Segments=<segment_id><orientation>,...;<optional_key>=<optional_value>
```

The section label `List of cycle segments` is conventional. Most parsers identify records by line prefixes and do not require the label. A file may begin with one or more comment or metadata lines, commonly beginning with `#`; ecSimulator uses this pattern.

## Interval Records

| Field | Type | Meaning |
| --- | --- | --- |
| `Interval` | literal | Record keyword. |
| `interval_id` | integer | 1-based interval identifier. |
| `chrom` | string | Chromosome or contig name. |
| `start` | integer | Coordinate of the first basepair. |
| `end` | integer | Coordinate of the last basepair. |

## Segment Records

| Field | Type | Meaning |
| --- | --- | --- |
| `Segment` | literal | Record keyword. |
| `segment_id` | integer | 1-based segment identifier. Segment `0` is reserved and is not listed as a normal segment. |
| `chrom` | string | Chromosome or contig name. |
| `start` | integer | Coordinate of the first basepair. |
| `end` | integer | Coordinate of the last basepair. |

## Cycle Records

Cycle lines are semicolon-separated key/value fields. Baseline AA fields are:

| Field | Type | Meaning |
| --- | --- | --- |
| `Cycle` | integer or string identifier | Cycle ID. AA writes cycles sorted by decreasing copy count. |
| `Copy_count` | float | Copy count of the cycle. |
| `Segments` | comma-separated oriented segment IDs | The segment walk. Each item is `<segment_id>+` or `<segment_id>-`. |

The last listed segment connects back to the first listed segment for circular cycles.

Cycle and path records may contain additional semicolon-separated `key=value` fields. Readers should parse the record into a key/value map, require the fields they need, and preserve unknown fields when rewriting. Known extension fields include `Length`, `Path_constraints_satisfied`, `IsCyclicPath`, `CycleClass`, `Contigs`, `Score`, and `Multiplicity`.

## Segment Orientation

`+` means the structure traverses the segment from its `start` coordinate to its `end` coordinate. `-` means the structure traverses the segment from `end` to `start`.

## Segment 0 Convention

Segment ID `0` is reserved for the source/connection vertex. AA documents that a cycle may contain segment `0` either zero times or two times. A line containing `0` represents a linear path whose endpoints connect to undetermined or out-of-interval positions. AA examples and CoRAL path output commonly use `0+` at the beginning and `0-` at the end.

## Divergence

ecSimulator currently writes non-circular structures as `0+, ..., 0+`, while CoRAL writes paths as `0+, ..., 0-`. AA's README says segment `0` may occur two times but does not canonize the sign pair. Consumers should treat any `0` item as a source/path sentinel unless they need stricter path orientation semantics.

## Informal Grammar

```
cycles_file     ::= header_line* interval_line* segment_section segment_line+ extension_section* structure_line+
header_line     ::= "#" text | other_metadata_line
interval_line   ::= "Interval" TAB id TAB chrom TAB integer TAB integer
segment_section ::= text line, conventionally "List of cycle segments"
segment_line    ::= "Segment" TAB id TAB chrom TAB integer TAB integer
structure_line  ::= cycle_line | extension_path_line
cycle_line      ::= "Cycle=" id ";" "Copy_count=" number ";" "Segments=" segment_walk extension_field*
extension_path_line ::= "Path=" id ";" "Copy_count=" number ";" "Segments=" segment_walk extension_field*
segment_walk    ::= oriented_segment ("," oriented_segment)*
oriented_segment::= integer ("+" | "-")
extension_field ::= ";" key "=" value
```

## Worked Example

```
Interval	1	chr1	100	399
List of cycle segments
Segment	1	chr1	100	199
Segment	2	chr1	300	399
Cycle=1;Copy_count=8.5;Segments=1+,2-
Cycle=2;Copy_count=1.0;Segments=0+,1+,0-
```

## Real Data Excerpt

This is a shortened CoRAL example from the [public CoRAL wiki](https://github.com/AmpliconSuite/CoRAL/wiki#cycles), included only to show realistic field shapes and path constraints:

```
Interval	1	chr2	11450087	12860095
List of cycle segments
Segment	1	chr2	11450087	11551950
Segment	2	chr2	11551951	11569465
List of longest subpath constraints
Path constraint	1	e14+:1,d1+:1,e18-:1,d4-:1,e20+:1	Support=181	Satisfied
Extracted cycles
Cycle=1;Copy_count=101.72274978772717;Segments=2+,3+,14+,18-,20+,21+,22+,11+,12+,5+,6+,7+,8+,9+;Path_constraints_satisfied=1,3,4,9,14
```

## Tool-Specific Extensions

### CoRAL Cycles

CoRAL writes AA-like intervals and segments, optional subpath-constraint sections, then `Cycle=` and `Path=` structures. The public CoRAL wiki describes these as the heaviest graph walks found by cycle extraction; if no cycle is found, CoRAL may output a heaviest path. `Path=` records have the same semicolon key/value style as cycle records and use `Segments=0+,<walk>,0-`. CoRAL can append `Path_constraints_satisfied=<comma-separated ids>`.

```
List of longest subpath constraints
Path constraint	1	e1+:1,d1+:1,e2+:1	Support=12	Satisfied
List of extracted cycles/paths
Cycle=1;Copy_count=10.0;Segments=1+,2+;Path_constraints_satisfied=1
Path=2;Copy_count=2.0;Segments=0+,1-,0-;Path_constraints_satisfied=
```

### CycleExtractor Cycles

CycleExtractor writes extracted cycles and paths in a CoRAL-like form. Some writer paths omit `Interval` lines and begin at `List of cycle segments`; this means consumers that require intervals must derive them from segments or the original graph. CycleExtractor's constraint records may use `Path constraint` with a space instead of the graph file keyword `path_constraint`.

### BFBArchitect Cycles

BFBArchitect writes intervals and segments, then a `List of longest subpath constraints` label followed by `Path=` lines, not `Cycle=` lines. Path lines may append `Score=<value>` and `Multiplicity=<value>`. These fields are BFBArchitect-specific and are not baseline cycle fields.

### AmpliconClassifier Annotated Cycles

AmpliconClassifier writes corrected/annotated cycle files with additional semicolon fields before `Segments`: `Length`, `IsCyclicPath`, and `CycleClass`. Consumers should preserve unknown key/value fields when rewriting.

### AmpliconReconstructorOM Path Cycles

AmpliconReconstructorOM writes AA-like cycles for optical-map paths with an additional `Contigs=<comma-separated contig ids>` field.

### ecSimulator Cycles

ecSimulator writes a comment header, baseline intervals/segments, and `Cycle=1;Copy_count=1.0;Length=<bp>;Segments=<walk>`. The `Length` field is an extension.

## Fully Resolved Cyclic Structures

CycleExtractor's output is intended to represent extracted closed walks and paths from an input amplicon graph. Its closed structures are still serialized as `Cycle=` with segment orientations; its linear structures are serialized as `Path=` or as segment walks containing source segment `0`, depending on the writer path used. The file format alone does not prove biological completeness; it records the structure the tool emitted.

## Edge Cases

Cycle lines may include extra semicolon fields. The most compatible parser reads the fields into a key/value map, requires `Copy_count` and `Segments` for baseline structures, and preserves unrecognized fields. Segment IDs should be parsed by removing the final orientation character, not by assuming a fixed number of digits. Several consumers ignore `Interval` records entirely.

## Tools Using Cycles Files

| Tool | Role |
| --- | --- |
| AmpliconArchitect | Produces baseline cycles files and consumes them for visualization and sequence export. |
| AmpliconClassifier | Consumes AA cycles and writes annotated/corrected cycles variants. |
| CoRAL | Produces long-read cycles/path files with path-constraint annotations; consumes cycles for conversion/scoring utilities. |
| CycleExtractor | Consumes AA/CoRAL graph files and writes cycles/path files. |
| CycleViz | Consumes cycles files for visualization and cycle conversion. |
| BFBArchitect | Writes path-style cycles files with BFB-specific score fields. |
| ecSimulator | Writes simulated AA-like cycles files. |
| AmpliconReconstructorOM | Consumes AA cycles and writes optical-map path cycles variants. |
| AmpliconSuiteAggregator | Preserves cycles files when packaging results. |

Implementation-level locations are tracked in [notes.md](notes.md) for maintainers.
