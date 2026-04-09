# Phase 2: Scaled Extraction & Validation - Research

## Objective
"What do I need to know to PLAN this phase well?"

## Current State
- The Phase 1 prototype (`notebooks/01_prototype_ocr_extraction.ipynb`) has successfully extracted text using Typhoon OCR (API) and proved the logic.
- We have the PDF data located in `source/เขตเลือกตั้งที่ 2/`.
- We need to scale this extraction sequentially across all PDFs.

## Architecture / Batching Pattern
1. **Concurrency and Rate Limits**: Typhoon has a rate limit of 2 req/s and 20 req/min. This means we can process at most 20 PDFs per minute.
2. **Parallelization inside Jupyter**: We can use `concurrent.futures.ThreadPoolExecutor` combined with `time.sleep` or a custom rate-limiter, or we can use `asyncio` with a semaphore and token bucket logic, but a simple synchronous batch approach with delays might be more stable in an `ipynb`. Given the hard 20 req/min limit, a simple loop that processes up to 20 PDFs, then sleeps for the remainder of the minute (or delays 3 seconds between each call, ensuring max 20 per minute) is highly reliable.
3. **Regex Extraction**: We continue using the Strict Template Matching built in phase 1.
4. **Validation Logic (DATA-02)**: For each file, the parser extracts candidate votes and sum total. We will compare sum(candidate_votes) with parsed_total.
5. **Error Tracking Mechanism**:
   - If a file completely fails (OCR failure, network timeout, or regex mismatch), log the file path and error reason to a structured log file (e.g., `logs/failed_extractions.json` or a python list dumped to disk).
   - If the file succeeds, append to `election_structured_data.csv`.
   - If sum doesn't match total, keep a separate internal tracker (e.g., list `validation_mismatches`) for the user to manually review, but STILL output the extracted row to the dataframe. Do NOT flag it in the CSV directly (D-04).
6. **Retry Mechanism**: The notebook should have a specific cell or function that can read the `logs/failed_extractions.json` and attempt the pipeline again only on those paths.

## Validation Architecture (Nyquist Framework)
- Dimension 1 (State & Data): `election_structured_data.csv` must be generated with numerical vote counts.
- Dimension 8 (Requirements): Fulfills EXTR-05, DATA-01, DATA-02, DATA-03. Validation is checked by running the notebook and manually ensuring the mismatches are surfaced in the dataframe summary without stopping the execution.

## Summary
The pipeline involves reading ~250+ PDFs, submitting them to Typhoon OCR at 3s intervals (or using a rate-limiting semaphore), running the regex templates, and cleanly tracking both hard failures (for retry) and logic mismatches (for manual user review).

## RESEARCH COMPLETE
