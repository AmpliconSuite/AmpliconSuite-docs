---
name: amplicon-repository
description: Query AmpliconRepository (ampliconrepository.org), a public database of ecDNA, BFB and other focal amplification calls in cancer genomes and cell lines, through its REST API. Use when asked whether a gene (MYC, EGFR, ERBB2, ...) is amplified or carried on ecDNA, in which cancer types, samples, cell lines or projects; how often ecDNA or a given amplification class occurs; which genes are co-amplified; or for AmpliconArchitect / AmpliconClassifier results from published cohorts. Not for running AmpliconSuite on the user's own sequencing data.
---

# AmpliconRepository

AmpliconRepository holds AmpliconSuite results (AmpliconArchitect reconstructions,
AmpliconClassifier calls) for thousands of tumour samples and cell lines. Answer
from it with the REST API at `https://ampliconrepository.org/api/v1/`. Never
scrape the HTML pages: one API call replaces hundreds of page renders, and
crawling the pages is what the site's rate limits exist to stop.

This skill holds the rules that most often produce wrong answers. For the full
parameter list and field definitions, fetch
`https://ampliconrepository.org/llms.txt` (prose) or
`https://ampliconrepository.org/api/v1/openapi.json` (spec). Both are current;
this file may be an older copy.

## Workflow

1. **Start with `GET /api/v1/features/`.** It searches every amplicon in every
   public project at once; you do not need to find a project first.
   ```
   /api/v1/features/?gene_any=MYC&classification=ecDNA
   /api/v1/features/?gene_all=MYC,PVT1&same_amp=true
   /api/v1/features/?gene_any=EGFR&count_only=true
   ```
   `project_id=` or `project_name=` scopes any query to one project.
2. **For questions about samples rather than amplicons**, use
   `GET /api/v1/features/samples/` with the same filters. It returns one entry per
   sample with its `project_id` and the `classifications` it carries. For a
   rate, read `count` from two calls per project (with and without
   `classification=ecDNA`, `limit=1`); it has no `count_only`.
3. **Before filtering on cancer type, sample type or tissue**, read
   `GET /api/v1/features/facets/` and use the spellings listed there.
4. **Read `GET /api/v1/projects/` once** before any count or fraction. Each
   project's `description` says what the cohort is, how it was selected and
   whether it overlaps another project; `metadata_coverage` says how much of its
   metadata was recorded; `AC_version` and `reconstruction_tools` say how it was
   classified. It returns every project as a plain list and takes no
   parameters. Search its descriptions too: some samples are named only by
   accession, and the project is the only place their identity is recorded.
5. **For rankings** ("which genes are most often on ecDNA"), page through
   `/features/` with the classification filter and `fields=` and aggregate
   yourself. There is no aggregation endpoint.
6. **Cite** each project by `id` and `version`, and link `sample_page_url` for
   a human-readable page.

## What the classes mean

AmpliconClassifier resolves each amplicon into one or more features:

- `ecDNA`: a cyclic, extrachromosomal structure.
- `FAN`: focal amplification in neochromosome. A broad (often tens of Mb),
  SV-dense amplification drawn from two or more chromosomes and most likely
  integrated in a chromosome. FAN amplicons may have passed through an ecDNA
  intermediate and may hold rare ecDNA subclones, but the call means the
  dominant structure is chromosomal. Introduced in AmpliconClassifier 2.0; see
  the [FAN primer](https://github.com/AmpliconSuite/AmpliconClassifier/blob/main/docs/fan_primer.md).
- `BFB`: breakage-fusion-bridge cycles.
- `Complex-non-cyclic`: a complex rearranged amplification with no cyclic
  structure.
- `Linear`: a simple, non-rearranged focal amplification.
- `Virus`: an amplification involving viral sequence.

Two caveats change how to read an `ecDNA` call:

- **The method cannot see simple reintegration.** From bulk short-read WGS,
  an ecDNA that reintegrated into a chromosome as a homogeneously staining
  region (HSR) still looks cyclic, so it is called `ecDNA`. In a line known to
  carry the amplification as an HSR (`COLO320HSR`, for example), an `ecDNA`
  call describes the structure, not proof that it is extrachromosomal now.
  Neochromosome formation is the exception, and that is what `FAN` captures.
- **Classifications depend on how the project was classified.** Each project
  in `/api/v1/projects/` carries `AC_version`. Projects classified before 2.0
  have no `FAN` calls, and many amplicons that 2.0 calls `FAN` were called
  `ecDNA` before it, so do not combine ecDNA counts from before and after 2.0.
  An empty or `NA` `AC_version` means the version is unknown; if the
  project's `classifications` list has no `FAN`, treat it as pre-2.0. Calls can
  also differ by `reconstruction_tools` on the same AC version: the same cell
  line can be `ecDNA` from AmpliconArchitect (short reads) and `FAN` from CoRAL
  (long reads).

## Rules that change the answer

- **A row is one feature, not one sample.** A sample can have many features.
  On `/features/`, `count` counts
  features and `sample_count` counts samples; on `/features/samples/`, `count`
  is already samples. A sample is a (project, sample name) pair. Do not
  deduplicate by `sample_name` alone: unrelated samples in different projects
  share names.
- **The same biological sample can be in several projects.** Some projects are
  filtered and unfiltered releases of one cohort, cohorts overlap each other
  under different naming schemes, cell lines recur across collections under
  different names (`HCC1569`, `HCC1569_BREAST`), and some small projects were
  chosen *because* they carry ecDNA. So a sample count is not a count of
  distinct tumours. Before reporting a count or fraction, check the contributing
  projects' descriptions, decide which to exclude or scope with `project_id`,
  and say what you did.
- **A pooled fraction depends on which cohorts were deposited.** The rate of
  ecDNA varies several-fold between cohorts. Report it per project alongside any
  pooled figure, and do not present the pooled figure as a population
  prevalence. When one project supplies a large share of a result, say so, or
  give the result with and without it. There is no age field: a pediatric or
  adult split has to come from the cohorts, and adult cohorts can include
  pediatric tumour types.
- **`classification: "None"` rows are samples with nothing focal found.** They
  carry no genes, so gene filters already exclude them. Exclude them from any
  other amplicon count. They do belong in the denominator of "what fraction of
  samples carry ecDNA".
- **The corpus includes mouse (`mm10`) samples.** The `reference_builds`
  breakdown in a response counts features, not samples, so it cannot correct a
  sample count. For human-only counts, query `reference_build=hg19` and
  `reference_build=hg38` separately and add them.
- **`same_amp=true` means both genes are on one feature.** Without it,
  `gene_all=A,B` matches samples carrying both genes anywhere.
- **`oncogenes` includes neighbours of the driver.** Genes such as PVT1 (next to
  MYC), FGF3/FGF4 (CCND1) or MIEN1 (ERBB2) appear because they sit on the same
  amplicon. `FAN` features are large and carry many oncogenes each, so a
  gene-by-gene ranking fills with one locus; rank loci (name the driver) and
  count samples, not features. Gene names follow each sample's reference build (`C17orf37` in hg19
  is `MIEN1` in hg38), so merge aliases before counting across builds.
- **Copy numbers are per feature, not per gene.** `feature_max_copy_number` is
  an upper bound for any gene on the feature. If you plot or report a gene's
  copy number from the API, say it is the feature's. Both numbers come from
  the copy-number segments under the feature's genomic intervals, and features
  of one amplicon can overlap (a `FAN` can span an `ecDNA`'s intervals), so two
  features can report the same maximum. The length-weighted
  `feature_median_copy_number` tells them apart. Per-gene copy number of an
  amplified gene is in the project archive (`*_gene_list.tsv`, column
  `gene_cn`). A gene on no feature in a sample was not focally amplified there;
  the API has no copy number for it, but the archive's genome-wide
  `*_CNV_CALLS.bed` usually does.
- **A count filtered on metadata is a floor, not an answer.** `cancer_type`,
  `sample_type` and `tissue_of_origin` are free text from each submitter. A
  missing value can be empty (`""`) or the string `nan`, and `metadata_coverage`
  counts `nan` as recorded. Whole projects can lack a field, so check
  `metadata_coverage` for projects that may hold the samples you want. The
  facets endpoint covers the whole corpus and ignores filters, so to measure the
  shortfall in *your* result, run the query without the metadata filter, with
  `fields=` naming that field, and count the missing values. Say so in the
  answer. `NA` usually means "not applicable", but a project that records `NA`
  for every sample has simply not provided the field, even though
  `metadata_coverage` reports it as complete.
- **Spellings vary in granularity.** Case is folded, but `Breast` and
  `Breast Adenocarcinoma` are separate values, and some projects record only a
  family (`glioma`) or a system (`Nervous system`) instead of a diagnosis. Query
  every spelling in the facets that you mean. Where the choice is a judgement
  call, give the strict and the broad answer.
- **Sample names are exact by default.** `sample_name=` matches exactly
  (case-insensitive). For a cell line whose recorded name you are unsure of, use
  `sample_name_contains=` (on `/features/` or `/features/samples/`), try
  punctuation variants (`U87` misses `U-87-IDH1mutant`), and look at the
  candidates before aggregating: `COLO320` matches `COLO320DM` and `COLO320HSR`,
  which are different lines.

## Calling conventions

- No authentication and no special User-Agent are needed for public data.
- **Only `classification`, `cancer_type`, `sample_type` and `tissue_of_origin`
  take several values**, by repeating the parameter:
  `?cancer_type=Breast&cancer_type=Breast%20Adenocarcinoma` matches either
  (OR). Genes take a
  comma-separated list (`gene_any=MYC,EGFR`). Repeating any other parameter
  (`gene_any`, `project_id`, `project_name`, `reference_build`) silently keeps
  only the last value, so run one query per value.
- Page with `limit`, then pass the response's `next_cursor` back as `cursor`.
  `?next_cursor=` is an error. The error for a too-large `limit` gives the
  maximum.
- `fields=` works on `/features/` only.
- On the `/features/` endpoints, an unknown parameter returns `400` with the
  list of valid ones. Fix it and retry; never report results you are not sure
  were filtered.
- Errors look like `{"error": "...", "code": "..."}`. Branch on `code`.
- `429`: wait for `Retry-After` seconds. `403`: you lack access; stop.
  `503` with `index_unavailable`: retry shortly; it does not mean "no results".
- Download a project archive (`/api/v1/projects/{id}/download/`) only when the
  question needs something the API lacks: per-gene copy number, cycles and
  breakpoint graphs, BED files, or figures. Never download archives in bulk.
  `llms.txt` explains the archive layout.
