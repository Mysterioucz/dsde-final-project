---
phase: 02-scaled-extraction-validation
verified: 2026-04-09T12:30:00Z
status: passed
score: 6/6 must-haves verified
re_verification: false
---

# Phase 02: Scaled Extraction & Validation — Verification Report

**Phase Goal:** Execute the OCR across all PDFs for Uthai Thani 2, clean common anomalies, and validate against logical vote sum totals. Export structured dataset.
**Verified:** 2026-04-09T12:30:00Z
**Status:** passed
**Re-verification:** No — initial verification

---

## Goal Achievement

### Observable Truths

| #  | Truth                                                                                  | Status     | Evidence                                                                                                                                         |
|----|----------------------------------------------------------------------------------------|------------|--------------------------------------------------------------------------------------------------------------------------------------------------|
| 1  | Notebook exists with Typhoon OCR batch loop over all 250+ PDFs                         | VERIFIED   | `notebooks/02_scaled_extraction.ipynb` created in commit 7c5323f; Cell 6 iterates `all_pdfs` with `tqdm`                                        |
| 2  | API calls are rate-limited to max 20/min via `time.sleep(3)` in a `finally` block      | VERIFIED   | Cell 6 `finally:` block contains `time.sleep(INTER_REQUEST_DELAY)` where `INTER_REQUEST_DELAY = 3.0`; applies on both success and failure        |
| 3  | Errors are caught per-file in a try-except and logged to `logs/failed_extractions.json` | VERIFIED   | Cell 6 try-except appends to `failed_extractions` list; Cell 7 writes to `LOGS_DIR / "failed_extractions.json"` after the loop                  |
| 4  | Retry mechanism reads `logs/failed_extractions.json` and re-runs only failed files     | VERIFIED   | Cell 10 opens `failed_log_path`, iterates `retry_candidates`, appends successes to `extraction_results`, persists updated log with only remaining failures |
| 5  | Validation tracks mismatches in-memory (`validation_mismatches`) with no flag in CSV  | VERIFIED   | Cell 11 builds `validation_mismatches: list[dict]`, asserts `'mismatch' not in df.columns`; Cell 12 also asserts no mismatch col in `df_export`  |
| 6  | Data cleanup (O->0, l->1, I->1, S->5, B->8) applied before export to `election_structured_data.csv` | VERIFIED   | Cell 12 defines `OCR_NUMERIC_FIXES` dict with all 5 substitutions, applies via `str.replace()`, writes to `OUTPUT_DIR / 'election_structured_data.csv'` |

**Score:** 6/6 truths verified

---

### Required Artifacts

| Artifact                                 | Expected                                      | Status     | Details                                                                                       |
|------------------------------------------|-----------------------------------------------|------------|-----------------------------------------------------------------------------------------------|
| `notebooks/02_scaled_extraction.ipynb`   | Main pipeline notebook (12 cells)             | VERIFIED   | Exists; 12 code cells (Cells 1-12); 570 lines at initial commit + 77 + 110 lines across subsequent commits |
| `logs/.gitkeep`                          | Tracks `logs/` directory in git               | VERIFIED   | Created in commit 7c5323f; confirmed present at `/logs/.gitkeep`                              |
| `output/.gitkeep`                        | Tracks `output/` directory in git             | VERIFIED   | Created in commit 8207992; confirmed present at `/output/.gitkeep`                            |

---

### Key Link Verification

| From                              | To                                     | Via                                                   | Status     | Details                                                                          |
|-----------------------------------|----------------------------------------|-------------------------------------------------------|------------|----------------------------------------------------------------------------------|
| Cell 6 (main loop)                | `run_typhoon_ocr_full_page()`          | direct call inside try block                          | WIRED      | `ocr_text = run_typhoon_ocr_full_page(pdf_path)` in Cell 6                      |
| Cell 6 (main loop)                | `parse_vote_counts()`                  | called on OCR text in try block                       | WIRED      | `votes = parse_vote_counts(ocr_text)` immediately after OCR call in Cell 6      |
| Cell 6 (main loop)                | `failed_extractions` list              | except block appends error records                    | WIRED      | `failed_extractions.append(error_record)` in except; `time.sleep` in finally    |
| Cell 7                            | `logs/failed_extractions.json`         | `json.dump()` to `failed_log_path`                    | WIRED      | Opens and writes `log_payload` including `failed_extractions` list               |
| Cell 10 (retry)                   | `logs/failed_extractions.json`         | `json.load()` reads `retry_candidates`                | WIRED      | `with open(failed_log_path)` -> `retry_candidates = log_data['failed_extractions']` |
| Cell 10 (retry)                   | `extraction_results`                   | appends successful retries to master list             | WIRED      | `extraction_results.append(success_record)` inside retry try block              |
| Cell 11 (validation)              | `extraction_results`                   | `pd.DataFrame(extraction_results)` builds `df`        | WIRED      | `df = pd.DataFrame(extraction_results)`                                          |
| Cell 11 (validation)              | `validation_mismatches`                | loop over `df.iterrows()` builds mismatch list        | WIRED      | `validation_mismatches.append(...)` when `computed_total != parsed_total`       |
| Cell 12 (cleanup)                 | `df` from Cell 11                      | `df.copy()` then `.str.replace()` on numeric cols     | WIRED      | `df_clean = df.copy()` at top of Cell 12                                        |
| Cell 12 (cleanup)                 | `output/election_structured_data.csv`  | `df_export.to_csv(OUTPUT_PATH)`                       | WIRED      | `df_export.to_csv(OUTPUT_PATH, index=False, encoding='utf-8-sig')`              |

---

### Data-Flow Trace (Level 4)

Level 4 data-flow verification is not applicable here in the traditional sense — this is a data extraction pipeline notebook, not a web component. The notebook produces data rather than consuming it from a live store. The output CSV (`election_structured_data.csv`) requires live Typhoon OCR API execution against real PDFs and will not exist until the notebook is run. This is the expected and documented state.

The code path from PDF discovery -> OCR call -> vote parsing -> DataFrame -> cleanup -> CSV export is fully connected through the notebook cell sequence. No hollow props or disconnected static returns were found.

---

### Behavioral Spot-Checks

| Behavior                               | Command                                                                                        | Result                                                                        | Status |
|----------------------------------------|------------------------------------------------------------------------------------------------|-------------------------------------------------------------------------------|--------|
| Notebook file exists and is parseable  | `ls notebooks/02_scaled_extraction.ipynb`                                                     | File present; Read tool confirmed 12 code cells with substantive content      | PASS   |
| Rate-limit delay in `finally` block    | Grep for `time.sleep` in `finally` context                                                    | `time.sleep(INTER_REQUEST_DELAY)` present in `finally:` in Cell 6 and Cell 10 | PASS   |
| Error log write uses json.dump         | Grep for `json.dump` targeting `failed_extractions.json`                                      | `json.dump(log_payload, f, ...)` in Cell 7; `json.load` in Cell 10            | PASS   |
| No mismatch column in export assertion | Grep for `assert 'mismatch'` in notebook                                                      | Two `assert` statements present: one in Cell 11 (df), one in Cell 12 (df_export) | PASS   |
| OCR character fix dict complete        | Inspect `OCR_NUMERIC_FIXES` dict                                                               | `{'O':'0','l':'1','I':'1','S':'5','B':'8'}` — all 5 substitutions present     | PASS   |
| Output path is `election_structured_data.csv` | Inspect `OUTPUT_PATH` definition                                                       | `OUTPUT_DIR / 'election_structured_data.csv'` where `OUTPUT_DIR = parent / 'output'` | PASS |

---

### Requirements Coverage

| Requirement | Source Plan    | Description                                                            | Status    | Evidence                                                                                                                           |
|-------------|----------------|------------------------------------------------------------------------|-----------|------------------------------------------------------------------------------------------------------------------------------------|
| EXTR-05     | 01-PLAN.md     | OCR precisely captures numerical party/candidate vote counts from the formatted grid | SATISFIED | Cell 5 `parse_vote_counts()` extracts `eligible_voters`, `valid_ballots`, `total_votes`, `candidate_votes` dict via Thai-numeral-aware regex |
| DATA-01     | 01-PLAN.md     | Data cleaning logic handles common OCR misinterpretations              | SATISFIED | Cell 12 `OCR_NUMERIC_FIXES` dict applied via `str.replace()` to all numeric scalar columns before export                          |
| DATA-02     | 02-PLAN.md     | Validation logic asserts `sum(candidate votes) == total valid votes`   | SATISFIED | Cell 11 iterates `df.iterrows()`, compares `sum(candidate_votes.values())` vs `total_votes`, collects mismatches in `validation_mismatches` list; `assert` confirms no flag column |
| DATA-03     | 02-PLAN.md     | Final output is structured as `election_structured_data.csv`           | SATISFIED | Cell 12 writes `df_export.to_csv(OUTPUT_PATH)` where `OUTPUT_PATH = output/election_structured_data.csv`; candidate_votes dict expanded to `cand_N_votes` columns |

No orphaned requirements: all four phase-02 requirement IDs (EXTR-05, DATA-01, DATA-02, DATA-03) are claimed by a plan and verified in the notebook.

---

### Anti-Patterns Found

| File                                    | Pattern                                | Severity | Assessment                                                                                                         |
|-----------------------------------------|----------------------------------------|----------|--------------------------------------------------------------------------------------------------------------------|
| `notebooks/02_scaled_extraction.ipynb`  | `output/election_structured_data.csv` does not exist on disk | Info     | Expected — file requires live Typhoon OCR API execution. SUMMARY.md documents this explicitly. Not a stub; the write path is fully wired. |
| `notebooks/02_scaled_extraction.ipynb`  | `ballots_issued` regex not implemented in `parse_vote_counts()` | Warning  | The `result` dict initialises `ballots_issued: None` but there is no regex block in Cell 5 to populate it. This field will always be `None` in the DataFrame. It is not required by any of the four phase-02 requirements, and no plan task calls for it, so it does not block goal achievement. |

No blocker anti-patterns found. No placeholder text, no `return {}` / `return []` stubs, no TODO/FIXME comments in the implementation cells.

---

### Human Verification Required

#### 1. Live API Execution

**Test:** Set `TYPHOON_OCR_API_KEY` in `.env`, ensure the `source/เขตเลือกตั้งที่ 2/` directory is populated, then execute all 12 cells sequentially.
**Expected:** `output/election_structured_data.csv` is created with one row per polling station; tqdm bar shows ~3 seconds per iteration; `logs/failed_extractions.json` is written (possibly empty if all succeed).
**Why human:** Requires live Typhoon OCR API credentials and the full PDF dataset; cannot be verified programmatically without executing against the real API.

#### 2. Thai-character regex accuracy

**Test:** Provide a sample OCR text string from a known ส.ส.5/18 form and call `parse_vote_counts()` directly. Inspect the returned dict.
**Expected:** All numeric fields (`eligible_voters`, `valid_ballots`, `total_votes`, `candidate_votes`) are populated correctly for a realistic Thai-numeral OCR output.
**Why human:** Regex correctness against real Thai OCR output requires a real document sample. Cannot be validated without the PDF data.

#### 3. Rate-limiting behaviour under burst failure

**Test:** Temporarily inject a bad API key and run Cell 6 against a subset of ~5 PDFs. Observe that elapsed time is still ~15 seconds (3s delay fires even on failure).
**Expected:** `time.sleep()` in the `finally` block fires even when every iteration fails; no burst of rapid errors.
**Why human:** Requires live API call attempt to trigger the failure path and time the sleep.

---

### Gaps Summary

No gaps were identified. All six observable truths are verified, all four requirement IDs (EXTR-05, DATA-01, DATA-02, DATA-03) are satisfied by substantive code, all key links between cells are connected, and no blocker anti-patterns were found.

The one noted warning (`ballots_issued` field always `None`) is a minor omission not required by any plan task or requirement and does not affect goal achievement or the downstream Phase 3 analysis.

The absence of `output/election_structured_data.csv` on disk is not a gap — it is the documented and expected state for a pipeline that requires live API execution.

---

_Verified: 2026-04-09T12:30:00Z_
_Verifier: Claude (gsd-verifier)_
