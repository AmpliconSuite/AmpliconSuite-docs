# Copy-Number bedGraph

This page documents the simple bedGraph-style copy-number track used by AmpliconSuite tools when graph segment copy numbers are exported for visualization or downstream overlap operations.

Spec version: draft 0.1.

## Baseline Structure

```
chrom    start    end    value
```

| Field | Type | Meaning |
| --- | --- | --- |
| `chrom` | string | Chromosome or contig name. |
| `start` | integer | Start coordinate copied from the graph sequence edge start vertex. |
| `end` | integer | End coordinate copied from the graph sequence edge end vertex. |
| `value` | float | Copy number or coverage-like value, depending on producer. |

The clearest in-repo writer is AmpliconReconstructorOM's `graph_to_bed.py --bedGraph`, which reads `sequence` records from an AA graph and writes `chrom start end copy_number` to `<graph_basename>.bedGraph`.

## Divergence

AA's README documents `_cnseg.txt`, not a final bedGraph file, as the AA copy-number segmentation cache. CycleViz also accepts bed/bedGraph/wig-like feature tracks where column 4 is the plotted value. The project should decide whether "AA emits bedGraph" refers to a pipeline wrapper, CycleViz-derived coverage bedgraphs, or graph-to-bedGraph conversion, because the AA core README section does not name a final `_bedgraph` output.

## AA `_cnseg.txt` Cache

AmpliconArchitect documents an intermediate copy-number segmentation file named like `{out}_{CHROM}_{START}_{END}_..._cnseg.txt`. It is tab-separated and has a header. The README fields are:

| Field | Meaning |
| --- | --- |
| `CHROM` | Chromosome name. |
| `START` | Coordinate of first basepair in the segment. |
| `END` | Coordinate of last basepair in the segment. |
| `CN` | Predicted copy number. |
| `start_refined` | Whether the start position could be determined at fine resolution. |
| `end_refined` | Whether the end position could be determined at fine resolution. |

## Worked Example

```
chr1	100	199	12.0
chr1	300	399	10.0
```

## Tools Using bedGraph-Like Tracks

| Tool | Role |
| --- | --- |
| AmpliconArchitect | Documents `_cnseg.txt`; AA core does not document a final bedGraph output in the audited README. |
| AmpliconReconstructorOM | Can export graph sequence copy numbers as `.bedGraph`. |
| CycleViz | Consumes feature bedGraph/wig-like tracks and can generate coverage bedgraphs. |
| AmpliconSuiteAggregator | Preserves CNV BED outputs, not graph-derived bedGraph tracks. |

Implementation-level locations are tracked in [notes.md](notes.md) for maintainers.
