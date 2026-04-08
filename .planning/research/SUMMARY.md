# Project Research Summary

**Project:** Thai Election 2026 Data Pipeline (Uthai Thani 2)
**Domain:** Data Engineering, OCR Extraction, and Data Science Analysis
**Researched:** 2026-04-08
**Confidence:** HIGH

## Executive Summary

The project requires extracting voting tallies from unstructured PDF forms (ส.ส.5/16, 5/17, 5/18) and transforming them into a structured dataset for analysis. Since the PDFs are image-based, OCR is essentially the main bottleneck and risk factor. The recommended approach is to process PDFs sequentially, apply robust heuristics during text extraction using coordinates or keyword matching, followed by exploratory data analysis (EDA). The architecture naturally separates into a distinct extraction notebook and an analysis notebook to avoid re-running expensive OCR tasks on every analysis iteration.

## Key Findings

### Recommended Stack

**Core technologies:**
- **`pandas`**: Data manipulation — Essential for DataFrame generation and data validation logic.
- **`pdf2image`**: Preprocessing — Necessary for transforming PDF pages into images suitable for OCR.
- **`pytesseract` / `easyocr`**: OCR engine — EasyOCR often provides better Thai language support out of the box than Tesseract, making it ideal for Thai election forms.
- **`matplotlib` / `seaborn`**: Visualization — Standard stack for statistical plotting and identifying data anomalies in EDA.

### Expected Features

**Must have (table stakes):**
- Automated traversal of nested directories to locate all relevant PDFs.
- Image to string translation using OCR with specific bounding constraints or regex filtering.
- Vote counting logic that accurately identifies party list votes vs constituency votes.
- Validation logic that checks whether sum of individual tallies matches total votes cast.

**Should have (competitive):**
- Confidence scores associated with extracted text to manually review low-confidence extractions.
- Visual dashboards aggregating station-level data into district-level trends.

### Architecture Approach

**Major components:**
1. **Raw File Ingestion**: Standard Python `os` and `glob` usage to load the polling station directory structures.
2. **Component 1 (Extraction & Cleaning Pipeline)**: Runs OCR, uses regex to format text, verifies sums, and exports `election_data.csv`.
3. **Component 2 (Analysis Pipeline)**: Uses `pandas` to read the CSV, perform EDA, and generate dashboard visualizations.

### Critical Pitfalls

1. **OCR Noise on Checkboxes & Noisy Fonts** — Form shapes or signatures can be misread. *Mitigation:* Apply strict regex for typical numeric shapes and cross-validate with the row/column names. Include manual verification sets.
2. **High Execution Time per PDF** — Running full-page OCR on 250+ polling stations (with multiple PDF pages per station) can take hours. *Mitigation:* Extract only specific cropped regions where tables reside, or use multithreading/multiprocessing. Run locally rather than in immediate memory streams.

## Implications for Roadmap

Based on research, suggested phase structure:

### Phase 1: Prototype Extraction on Sample
**Rationale:** We need to confirm whether Tesseract or EasyOCR handles Thai election forms better before running it 250+ times.
**Delivers:** Small `ipynb` testing basic OCR on just 3-5 PDFs.
**Addresses:** Image to string translation, bounding box strategies.

### Phase 2: Full Data Preparation Pipeline (Component 1)
**Rationale:** Once OCR logic is vetted, scale it up to the entire constituency.
**Delivers:** `Data_Preparation.ipynb` and `election_structured_data.csv`.
**Uses:** `easyocr` or `pytesseract`, `pandas`, `pdf2image`.

### Phase 3: Data Analysis & Visualization (Component 2 & 3)
**Rationale:** Cannot begin analysis until the CSV is reliable.
**Delivers:** `Data_Analysis.ipynb`, insights, and visual charts.
**Uses:** `matplotlib`, `seaborn`.

### Phase Ordering Rationale

- We must prototype the OCR strategy before automating the entire directory because PDF format variations or layout shifts might instantly break the script, causing hours of lost compute time.

## Confidence Assessment

| Area | Confidence | Notes |
|------|------------|-------|
| Stack | HIGH | `easyocr` and `pandas` are the standard open-source choices for this tasks. |
| Features | HIGH | Project requirements map perfectly to feature expectations. |
| Architecture | HIGH | Two-notebook approach (Prep then Analysis) is the golden standard. |
| Pitfalls | HIGH | OCR inaccuracy is the universal problem in this domain. |

---
*Research completed: 2026-04-08*
*Ready for roadmap: yes*
