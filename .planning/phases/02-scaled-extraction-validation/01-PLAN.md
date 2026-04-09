---
wave: 1
depends_on: []
files_modified: ["notebooks/02_scaled_extraction.ipynb"]
autonomous: false
requirements: ["EXTR-05", "DATA-01"]
---

# Plan 02-01: Scaled Extraction Mechanism & Rate Limiting

## Objective
Create the main loop that iterates over all 250+ PDFs, invokes Typhoon OCR via API with strict rate limiting, handles hard API errors gracefully by logging them to a reusable structure, and parses the numerical votes.

## Tasks

```xml
<task>
  <description>Create Jupyter Notebook with batch queueing system</description>
  <action>Create `notebooks/02_scaled_extraction.ipynb`. Set up a processing cell that uses python's `time.sleep()` to ensure maximum throughput is heavily throttled to respect Typhoon OCR's limits of 20 requests per minute AND 2 requests per second (e.g. 1 request every 3 seconds) when iterating over `source/เขตเลือกตั้งที่ 2/`. Exclusively use Typhoon OCR. Discard use of Tesseract or PaddleOCR.</action>
  <read_first>[".planning/phases/02-scaled-extraction-validation/02-CONTEXT.md", "notebooks/01_prototype_ocr_extraction.ipynb"]</read_first>
  <acceptance_criteria>
    - `notebooks/02_scaled_extraction.ipynb` exists.
    - Code contains time delay or token bucket logic limiting Typhoon OCR calls to 20/min.
    - Uses full-page extraction rather than bounding box crops.
  </acceptance_criteria>
</task>

<task>
  <description>Implement Error Logging (D-02)</description>
  <action>In the main extraction loop, wrap the Typhoon API call and text extraction logic in a try-except block. If an extraction fails, do NOT crash. Append the failed `pdf_path` and `error_reason` to a Python list/dict `failed_extractions`. At the end of the cell, write this list to `logs/failed_extractions.json`.</action>
  <read_first>["notebooks/02_scaled_extraction.ipynb"]</read_first>
  <acceptance_criteria>
    - Python cell has a try-except block capturing errors without exiting the loop.
    - Code outputs a JSON log file storing failed paths.
  </acceptance_criteria>
</task>
```

## Verification
- Run a subset of 5 PDFs. Ensure output rate is throttled visibly in the notebook output (e.g., tqdm progress bar shows ~3s per iteration).
- Manually trigger an error (e.g. bad API key) and verify it skips the file but continues the loop and writes to `logs/failed_extractions.json`.

## must_haves
- [ ] Throttles Typhoon OCR API calls to 20/min.
- [ ] Catches errors and logs them to a structured file without crashing.
