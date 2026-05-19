# Amplicon Intervals BED

There is no single AA-core `_amplicon_intervals.bed` format in the audited source equivalent to `*_graph.txt` or `*_cycles.txt`. The interoperable interval files in current use are BED-like files produced by classification and utility scripts, plus the `Interval` section embedded in cycles files.

Spec version: draft 0.1.

## Recommended Minimal BED Structure

```
chrom    start    end
```

| Field | Type | Meaning |
| --- | --- | --- |
| `chrom` | string | Chromosome or contig name. |
| `start` | integer | Start coordinate. |
| `end` | integer | End coordinate. |

Intervals BED files should be sorted by reference position: chromosome/contig order, then increasing start coordinate. This ordering is not an amplicon structure, traversal, or adjacency statement. It is only a reference-coordinate ordering for interval lookup and display.

Optional columns may be present in BED-like files from downstream tools, but this pass did not find one project-wide canonical schema for an `_amplicon_intervals.bed` filename.

## Divergence

AmpliconClassifier writes classification feature interval files named like `<sample>_<amplicon>_<feature>_intervals.bed`. These are feature/classification intervals, not necessarily the raw AA amplicon interval list. AmpliconReconstructorOM's `graph_to_bed.py` writes `<graph_basename>.bed` from graph sequence edges and can also write padded/merged BEDs. CoRAL stores amplicon intervals in graph objects and writes them into cycles `Interval` records, but the graph-file interval section is commented out in the audited writer.

## Worked Example

```
chr1	100	399
chr8	1000	1500
```

## Tools Using Interval BEDs

| Tool | Role |
| --- | --- |
| AmpliconClassifier | Produces and consumes feature/classification interval BED files. |
| CoRAL | Tracks amplicon intervals internally and writes them into cycles and summaries; the graph-file interval section is an extension. |
| AmpliconReconstructorOM | Produces BED intervals derived from graph sequence edges. |
| AmpliconSuiteAggregator | Preserves classification interval BEDs when packaging results. |

Implementation-level locations are tracked in [notes.md](notes.md) for maintainers.
