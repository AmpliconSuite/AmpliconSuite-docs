# Copy-Number bedGraph

This page documents the simple bedGraph-style copy-number track used by AmpliconSuite tools when graph segment copy numbers are exported for visualization or downstream overlap operations.

Baseline structure

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

Divergence

AA's README documents `_cnseg.txt`, not a final bedGraph file, as the AA copy-number segmentation cache. CycleViz also accepts bed/bedGraph/wig-like feature tracks where column 4 is the plotted value. The project should decide whether "AA emits bedGraph" refers to a pipeline wrapper, CycleViz-derived coverage bedgraphs, or graph-to-bedGraph conversion, because the AA core README section does not name a final `_bedgraph` output.

AA `_cnseg.txt` cache

AmpliconArchitect documents an intermediate copy-number segmentation file named like `{out}_{CHROM}_{START}_{END}_..._cnseg.txt`. It is tab-separated and has a header. The README fields are:

| Field | Meaning |
| --- | --- |
| `CHROM` | Chromosome name. |
| `START` | Coordinate of first basepair in the segment. |
| `END` | Coordinate of last basepair in the segment. |
| `CN` | Predicted copy number. |
| `start_refined` | Whether the start position could be determined at fine resolution. |
| `end_refined` | TODO: confirm exact field name and semantics in `AmpliconArchitect/README.md:257` and `src/bam_to_breakpoint.py:801`. |

Worked example

```
chr1	100	199	12.0
chr1	300	399	10.0
```

Parsers and writers in the wild

| Repo | Path | Function or code |
| --- | --- | --- |
| AmpliconArchitect | `README.md:257` | documents `_cnseg.txt` |
| AmpliconArchitect | `src/bam_to_breakpoint.py:801` | constructs `_cnseg.txt` filenames |
| AmpliconReconstructorOM | `graph_to_bed.py:57` | writes `.bedGraph` from graph sequence copy numbers |
| CycleViz | `extract_bedgraph.py:58` | writes coverage bedgraph tracks |
| CycleViz | `VizUtil.py:1663` | parses `primary_feature_bedgraph` and `secondary_feature_bedgraph` via BED parser |
| AmpliconSuiteAggregator | `src/asa_aggregator.py:74` | recognizes `_CNV_CALLS_unfiltered_gains.bed`, not graph bedGraph |
