# Amplicon Intervals BED

There is no single AA-core `_amplicon_intervals.bed` format in the audited source equivalent to `*_graph.txt` or `*_cycles.txt`. The interoperable interval files in current use are BED-like files produced by classification and utility scripts, plus the `Interval` section embedded in cycles files.

Recommended minimal BED structure

```
chrom    start    end
```

| Field | Type | Meaning |
| --- | --- | --- |
| `chrom` | string | Chromosome or contig name. |
| `start` | integer | Start coordinate. |
| `end` | integer | End coordinate. |

Optional columns may be present in BED-like files from downstream tools, but this pass did not find one project-wide canonical schema for an `_amplicon_intervals.bed` filename.

Divergence

AmpliconClassifier writes classification feature interval files named like `<sample>_<amplicon>_<feature>_intervals.bed`. These are feature/classification intervals, not necessarily the raw AA amplicon interval list. AmpliconReconstructorOM's `graph_to_bed.py` writes `<graph_basename>.bed` from graph sequence edges and can also write padded/merged BEDs. CoRAL stores amplicon intervals in graph objects and writes them into cycles `Interval` records, but the graph-file interval section is commented out in the audited writer.

Worked example

```
chr1	100	399
chr8	1000	1500
```

Parsers and writers in the wild

| Repo | Path | Function or code |
| --- | --- | --- |
| AmpliconClassifier | `ampclasslib/ac_io.py:275` | `write_interval_beds()` |
| AmpliconClassifier | `ampclasslib/profile_connections.py:761` | discovers `<sample>_amplicon*_intervals.bed` |
| AmpliconClassifier | `make_results_table.py:342` | reads feature `_intervals.bed` paths |
| CoRAL | `coral/output/cycle_output.py:31` | writes amplicon intervals as cycles `Interval` records |
| CoRAL | `coral/breakpoint/breakpoint_graph.py:512` | `infer_amplicon_intervals()` |
| CoRAL | `coral/breakpoint/breakpoint_utilities.py:734` | graph-file `AmpliconIntervals` output is present but commented out |
| AmpliconReconstructorOM | `graph_to_bed.py:22` | writes BED intervals from graph sequence edges |
| AmpliconSuiteAggregator | `src/asa_aggregator.py:88` | rescues `_intervals.bed` under classification BED directories |
