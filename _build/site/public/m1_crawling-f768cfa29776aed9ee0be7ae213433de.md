---
jupytext:
  text_representation:
    extension: .md
    format_name: myst
kernelspec:
  display_name: Python 3
  language: python
  name: python3
---

# Milestone 1 — Crawling

## 1. Objectives

Milestone 1 builds the initial scientific-paper corpus from arXiv. Its main goals are to:

- Resolve the assigned arXiv IDs and identify every version of each paper.
- Download the source archive for each version and determine whether it contains usable LaTeX.
- Collect publication metadata and reference information using Semantic Scholar.
- Record statuses and failures so that the resulting corpus can be inspected and later milestones can consume it.
- Prepare source material for Milestone 2 (parsing and standardization) and Milestone 3 (reference matching).

This report uses the supplied `M1_benchmark/summary.json` and `M1_benchmark/papers.csv`. The figures describe the saved export; they do not claim that a new crawl was run while generating this report.

```{warning}
The supplied summary marks this run as `report_only: true` and `measured: false`. The recorded 12.19 seconds is not a cold-start, end-to-end crawl benchmark: discovery is listed as 0 seconds and no single-paper trial is available. Peak/average RAM and peak temporary disk usage were not measured. Those values are therefore reported as unavailable rather than zero.
```

## 2. Scope and discovery coverage

Audit baseline: [official Milestone 1 specification](https://hcmus-cs-ta.github.io/lab-book/intro-ds/statement/crawling/).

| Measure | Result |
|---|---:|
| IDs represented in the saved export | 5,000 |
| First ID in the export | `2608.19007` |
| Last ID in the export | `2608.24006` |
| Papers found on arXiv | 5,000 |
| IDs not found | 0 |
| IDs that failed during discovery | 0 |
| Total versions identified | 6,101 |

The export covers 5,000 consecutive IDs from `2608.19007` through `2608.24006`. All 5,000 IDs are represented as paper rows, and the summary reports no missing IDs or discovery failures. The corpus contains more versions than papers because many papers have revised versions.

## 3. Implementation overview

The summary and per-paper CSV are consistent with a crawling workflow that separates publication discovery, source downloads, metadata, and references. The supplied files are result exports rather than the source-code repository, so the exact implementation details of each program cannot be independently verified from these files alone.

| Pipeline concern | Evidence in the supplied data | Why it matters |
|---|---|---|
| Discovery | One row per paper ID; 5,000 papers found | Establishes the corpus scope and coverage. |
| Version downloads | Per-paper version counts and version-status totals | Captures the fact that a paper can have multiple source snapshots. |
| TeX availability | `tex_ok` and version status fields | Identifies which papers can proceed to LaTeX parsing. |
| Publication metadata | `metadata_ok` | Tracks whether core paper metadata was obtained. |
| References | `references_status`, `total_refs`, `refs_with_arxiv_id`, and `refs_metadata_ok` | Records reference coverage and metadata quality. |
| Storage | Archive, pre-cleanup, and post-cleanup byte counts | Helps quantify storage requirements and the effect of cleanup. |

## 4. Results and coverage

### 4.1 Version and download status

| Version outcome | Count |
|---|---:|
| Total versions | 6,101 |
| `ok` | 5,767 |
| PDF-only | 311 |
| Error | 23 |

The version-status counts sum to 6,101. Most versions have status `ok`, while 311 are PDF-only and 23 have an error status. A PDF-only version may still represent a successfully retrieved paper version, but it does not provide the LaTeX source required for a source-parsing workflow.

At publication level, 4,978 of 5,000 papers are marked as successfully scraped, giving an overall success rate of **99.56%**. The summary reports a download success rate of 99.56% as well.

### 4.2 TeX and metadata coverage

| Measure | Count | Rate |
|---|---:|---:|
| Papers with TeX available | 4,698 / 5,000 | 93.96% |
| Papers with metadata | 5,000 / 5,000 | 100% |
| Papers with a Semantic Scholar record | 4,998 / 5,000 | 99.96% |
| Papers without a Semantic Scholar record | 2 / 5,000 | 0.04% |

The TeX coverage rate is lower than the scrape success rate. This distinction is important: successful crawling does not guarantee that a version contains parsable LaTeX. The CSV reports 302 papers without `tex_ok`, and 275 rows have missing pre- and post-cleanup size values. Those missing size values should not be treated as zero-byte papers; they are consistent with the absence of a measured extracted-source size for those rows.

### 4.3 Paper size before and after cleanup

The summary reports the following average sizes across the corpus:

| Measure | Bytes | Approx. MiB |
|---|---:|---:|
| Average paper size before cleanup | 4,609,968.76 | 4.40 |
| Average paper size after cleanup | 396,470.09 | 0.38 |
| Average version size before cleanup | 3,777,024.86 | 3.60 |
| Average version size after cleanup | 324,834.60 | 0.31 |
| Average compressed archive size per paper | 3,767,684.88 | 3.59 |

Based on the reported average paper sizes, the reduction is approximately **91.4%**. This suggests that cleanup substantially reduces the storage footprint of the extracted corpus. The supplied summary describes the size change but does not include enough source code to independently verify exactly which file types were removed, so this report does not assume a particular cleanup implementation.

The final corpus size reported in `summary.json` is **1,910,890,368 bytes (about 1.78 GiB)**. The whole data directory is reported as 1,917,996,688 bytes (about 1.79 GiB). These are saved disk-size figures, not peak temporary disk usage during crawling.

### 4.4 Reference collection

| Measure | Result |
|---|---:|
| Average references returned per paper | 32.38 |
| Average references with an arXiv ID per paper | 15.41 |
| Total references returned (sum of CSV rows) | 161,850 |
| Total arXiv-linked references in the CSV | 77,034 |
| Saved arXiv-linked references | 77,034 |
| Saved arXiv-linked references with metadata marked complete | 77,034 |
| Overall metadata success rate for the saved arXiv-linked references | 100% |

The CSV contains 161,850 references in the `total_refs` column summed across papers, of which 77,034 are marked as arXiv-linked. The saved and metadata-complete counts for that arXiv-linked subset are also 77,034. The reported 100% completeness therefore applies to the **saved arXiv-linked subset**, not necessarily to every reference returned by Semantic Scholar or every bibliography entry in the original LaTeX sources.

Two papers have the status `no_s2_record`. This is recorded as an explicit lookup outcome and should not be interpreted as proof that those papers have no references.

### 4.5 Errors and interpretation

The aggregate results suggest that discovery and metadata collection are highly complete, while source availability is the main limitation for downstream LaTeX parsing. The key distinctions are:

- **Paper-level success is not version-level success.** One paper can have several versions with different outcomes.
- **Download success is not TeX availability.** PDF-only versions and versions with errors can prevent source parsing.
- **Reference metadata completeness is scoped.** The 100% figure is calculated for saved arXiv-linked references and does not establish complete capture of all references.
- **Missing values are not zeros.** Empty source-size fields are left as missing rather than being interpreted as zero bytes.

## 5. Performance and environment

### 5.1 Timing

| Measure | Recorded value |
|---|---:|
| Summary wall time | 12.19 seconds |
| Discovery step time | 0.00 seconds |
| Average wall time per paper in summary | 0.002438 seconds |
| Separate single-paper end-to-end trial | Not measured |
| Fresh cold-start end-to-end benchmark | Not measured |

Because the summary is explicitly marked `report_only: true` and `measured: false`, the 12.19-second figure should not be presented as the time required to crawl all 5,000 papers. Discovery is recorded as skipped/zero and the separate single-paper trial is absent. The per-paper average is therefore not a valid estimate of full processing cost for a new run.

### 5.2 Memory and disk

| Measure | Result |
|---|---:|
| Peak RAM | Not measured |
| Average RAM | Not measured |
| Number of RAM samples | 0 |
| Peak extra disk usage during the run | Not measured |
| Final corpus size | 1,910,890,368 bytes |
| Whole data directory size | 1,917,996,688 bytes |

The saved environment metadata reports Windows 11, Python 3.14.0, 16 CPU cores, and approximately 15.83 GiB of total RAM. These are environment details, not measured resource consumption by the crawler. A new instrumented run is needed to report peak/average RAM, peak temporary disk usage, and a valid end-to-end runtime. The example report's required Colab benchmark context should also be checked against the course requirements before submission.

## 6. Summary and next steps

- The supplied corpus contains **5,000 papers and 6,101 versions**, with no IDs reported missing or failing discovery.
- **4,978 papers (99.56%)** are marked as successfully scraped, and **4,698 papers (93.96%)** have TeX available.
- Publication metadata is marked complete for all 5,000 papers; two papers lack a Semantic Scholar record.
- The reported average paper size falls from approximately **4.61 MB to 0.40 MB**, a reduction of about **91.4%**.
- The CSV records **77,034 saved arXiv-linked references**, all marked metadata-complete; this is not proof that every bibliography reference was retained.
- The supplied export is report-only, so fresh timing and memory benchmarks remain outstanding.

**Recommended follow-up:** run the full crawler from a clean state with instrumentation enabled, capture per-step timings and process-tree RAM samples, measure peak temporary disk usage, and preserve the resulting run metadata. Then verify source coverage and reference retention before using the corpus for Milestone 2 parsing.
