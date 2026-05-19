# File Format Spec Notes

Decisions to make

1. Canonize a source vertex spelling for graph files. AA documents `POS=-1`; CoRAL writes `0-->chr:pos`.
2. Decide whether long-read graph headers (`PredictedCN`, `NumberOfLongReads`) are a named subtype or accepted aliases in the core graph spec.
3. Decide whether CoRAL/CycleExtractor `PathConstraint` sections belong in the canonical graph and cycles formats or remain tool-specific extensions.
4. Decide whether `Path=` records are first-class structures in cycles files or an extension parallel to AA's `Cycle=` records.
5. Decide the canonical signs for source segment `0` in linear paths. CoRAL uses `0+ ... 0-`; ecSimulator currently writes `0+ ... 0+`.
6. Decide whether cycle extension fields such as `Length`, `IsCyclicPath`, `CycleClass`, `Contigs`, `Score`, and `Multiplicity` should be registered names with stable semantics.
7. Decide whether sequence-edge `Size` is inclusive (`end - start + 1`) or AA-style (`end - start`) and document any historical compatibility rule.
8. Decide whether an AA copy-number bedGraph is a canonical emitted format. The audited AA README documents `_cnseg.txt`; bedGraph output appears in utility and visualization code.
9. Decide whether `_amplicon_intervals.bed` is a canonical AmpliconSuite format or whether the cycles `Interval` section plus AC classification `_intervals.bed` files are the only documented interval formats.

Repeatable audit note

Keep upstream source checkouts temporary. A repeat audit can clone the AmpliconSuite repositories under `/tmp/as-format-audit`, then run:

```
rg -n "_graph\.txt|_cycles\.txt|SequenceEdge|BreakpointEdge|Cycle=|path_constraint|bedGraph|_intervals\.bed" /tmp/as-format-audit -g '!**/.git/**'
rg -n "def .*cycle|def .*graph|write.*cycle|read.*cycle|write.*graph|read.*graph|class .*Graph" /tmp/as-format-audit -g '!**/.git/**'
```

Do not modify upstream repositories during a spec audit. Record the exact upstream commit SHAs in future updates if the spec starts tracking versioned behavior.
