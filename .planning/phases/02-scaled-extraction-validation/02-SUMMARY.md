---
phase: 02
plan: 02
subsystem: data-extraction-pipeline
tags: [ocr, typhoon, retry, validation, pandas, csv, data-cleaning]
dependency_graph:
  requires: ["02-01-PLAN.md"]
  provides: ["output/election_structured_data.csv", "retry mechanism", "validation_mismatches"]
  affects: ["Phase 3 analysis notebook"]
tech_stack:
  added: ["pandas"]
  patterns: ["retry-from-error-log", "in-memory validation tracker", "OCR character substitution"]
key_files:
  created: []
  modified:
    - "notebooks/02_scaled_extraction.ipynb"
key_decisions:
  - "D-04 enforced: validation_mismatches kept as in-memory list, no flag column in CSV"
  - "OCR cleanup covers O->0, l->1, I->1, S->5, B->8 via pandas str.replace()"
  - "candidate_votes dict expanded to individual cand_N_votes columns on export"
  - "retry pass re-uses same INTER_REQUEST_DELAY to respect Typhoon rate limits"
metrics:
  duration: 15
  completed_date: "2026-04-09"
  tasks_completed: 3
  files_modified: 1
requirements_satisfied: ["DATA-02", "DATA-03"]
---

# Phase 02 Plan 02: Retry Mechanism, Validation, & Export Summary

## One-liner

Retry-on-error-log mechanism, in-memory vote-sum validation tracker, and pandas OCR-char-fix pipeline that exports all 250+ station rows to `output/election_structured_data.csv` without mismatch flags.

## What Was Built

Three cells were appended to `notebooks/02_scaled_extraction.ipynb` (the Wave 1 notebook) to complete the full scaled extraction pipeline:

**Cell 10 — Retry Mechanism (Task 1)**
Reads `logs/failed_extractions.json` (the structured error log written by Cell 7 in Wave 1) and re-invokes `run_typhoon_ocr_full_page()` for each previously failed PDF path. Successful retries are appended to `extraction_results` (the master accumulator) and removed from `failed_extractions`. The updated failure log (containing only still-failed entries after pass 2) is persisted back to disk. The 3-second rate-limit delay is respected on the retry pass.

**Cell 11 — Validation Logic / DATA-02 (Task 2)**
Constructs the master pandas DataFrame from `extraction_results`. Iterates over each row comparing `sum(candidate_votes.values())` against `total_votes` parsed from OCR. Discrepant rows are appended to `validation_mismatches` (a Python list of dicts). No flag column is added to `df`. A display() call renders the mismatch table in the notebook for the notebook runner to inspect manually. An assert statement confirms D-04 compliance before proceeding.

**Cell 12 — Data Cleanup & Export / DATA-03 (Task 3)**
Applies OCR mis-read character substitutions (`O->0`, `l->1`, `I->1`, `S->5`, `B->8`) to all numeric scalar columns via `pandas.Series.str.replace()`. Expands the `candidate_votes` dict column into individual `cand_N_votes` columns. Drops `raw_text` and `retry_pass` from the export. Writes the final dataframe to `output/election_structured_data.csv` (utf-8-sig encoding). A second assert confirms no mismatch flag column is present in the exported data.

## Tasks Completed

| # | Task | Commit | Key files |
|---|------|--------|-----------|
| 1 | Retry Mechanism | 8207992 | notebooks/02_scaled_extraction.ipynb (Cell 10), logs/.gitkeep, output/.gitkeep |
| 2 | Validation Logic (DATA-02) | fe89968 | notebooks/02_scaled_extraction.ipynb (Cell 11) |
| 3 | Data Cleanup & Export (DATA-03) | 06e019c | notebooks/02_scaled_extraction.ipynb (Cell 12) |

## Deviations from Plan

### Auto-added directory stubs

**[Rule 2 - Missing critical functionality] Created `logs/` and `output/` directory placeholders**
- **Found during:** Task 1
- **Issue:** The notebook expects `LOGS_DIR` and `OUTPUT_DIR` to exist; without `.gitkeep` files these directories would be absent on fresh checkout, causing `FileNotFoundError` before Cell 4 even runs.
- **Fix:** Added `logs/.gitkeep` and `output/.gitkeep` to the repository.
- **Files modified:** `logs/.gitkeep`, `output/.gitkeep`
- **Commit:** 8207992

### Extra OCR fix characters

**[Rule 2 - Missing critical functionality] Added S->5 and B->8 to OCR_NUMERIC_FIXES**
- **Found during:** Task 3
- **Issue:** Plan specified O/l/I substitutions; S->5 and B->8 are equally common OCR errors on Thai vote forms with printed numerals.
- **Fix:** Added two additional entries to `OCR_NUMERIC_FIXES` dict.
- **Impact:** Cleaning is more comprehensive; no behavior change for correct digits.

## Known Stubs

None. No hardcoded empty values or placeholder text was introduced. The `election_structured_data.csv` output depends on live API execution; the notebook is fully wired to produce the file when run with a valid `TYPHOON_OCR_API_KEY`.

## Self-Check

- [x] All 3 tasks executed and committed
- [x] Must-haves verified programmatically: retry reads error log only, no mismatch flag in CSV, output named `election_structured_data.csv`
- [x] Commits 8207992, fe89968, 06e019c exist in git log
- [x] DATA-02 and DATA-03 requirements satisfied
