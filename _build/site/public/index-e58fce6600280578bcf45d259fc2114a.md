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

# arXiv Corpus: Crawling and Parsing

**Introduction to Data Science · University of Science, VNU-HCM (HCMUS)**

This report documents the available progress through **Milestone 1 — Crawling**, using the supplied `my_data` benchmark exports. The report follows the structure of the provided example, but all project-specific figures below are based on the supplied data rather than the example's corpus.

- **[Milestone 1 — Crawling](m1_crawling.md)**: discovery coverage, version downloads, LaTeX availability, metadata and reference collection, data-size changes, and measurement limitations.

## Headline numbers

| Measure | Result |
|---|---:|
| Assigned IDs represented in the export | 5,000 |
| Papers found | 5,000 |
| IDs not found / discovery failures | 0 / 0 |
| Total paper versions | 6,101 |
| Versions with status `ok` | 5,767 |
| PDF-only versions | 311 |
| Version errors | 23 |
| Papers scraped successfully | 4,978 / 5,000 (99.56%) |
| Papers with TeX available | 4,698 / 5,000 (93.96%) |
| Papers with metadata | 5,000 / 5,000 (100%) |
| Papers without a Semantic Scholar record | 2 |
| Mean references returned per paper | 32.38 |
| Mean arXiv-linked references per paper | 15.41 |
| Mean paper size before / after cleanup | 4.61 / 0.40 MB (decimal) |
| Final corpus size reported in summary | 1.91 GB (decimal) |

## How to read these results

The supplied `summary.json` identifies itself as a **report-only** export (`report_only: true`, `measured: false`). Therefore, its saved corpus statistics are useful for describing the existing output, but its 12.19-second wall-time field is **not a fresh end-to-end crawling benchmark**. Discovery is recorded as 0 seconds, no single-paper trial was recorded, and memory/disk peak measurements are absent. These missing measurements are reported as unavailable, not as zero.

The data indicates strong coverage for discovery and publication metadata. It also shows why download success and usable LaTeX coverage are different: some versions are PDF-only or have errors, so not every discovered paper can provide TeX for later parsing. Reference metadata completeness is reported for the saved arXiv-linked references; it should not be interpreted as proof that every reference in every bibliography was retained.

## Reproduction and evidence

The numerical evidence in this report comes from `my_data/summary.json` and `my_data/papers.csv` supplied for this task. The CSV contains 5,000 publication rows and records per-paper version counts, download/TeX status, metadata and reference counts, and file sizes. The companion chapter gives definitions and caveats for the headline figures.

For the course requirements and audit baseline, consult the [official Milestone 1 specification](https://hcmus-cs-ta.github.io/lab-book/intro-ds/statement/crawling/).
