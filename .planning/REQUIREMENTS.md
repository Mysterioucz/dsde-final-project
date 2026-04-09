# Requirements: Thai Election 2026 Data Pipeline (Uthai Thani 2)

**Defined:** 2026-04-09
**Core Value:** Accurately transforming unstructured, high-volume official PDF election forms into a reliable, structured dataset that drives trustworthy data science visual analysis.

## v1 Requirements

### Data Prep Pipeline (Component 1)

- [x] **EXTR-01**: Pipeline iterates recursively through target constituency directories to locate raw PDF files.
- [x] **EXTR-02**: Pipeline implemented completely within a Jupyter Notebook (`.ipynb`).
- [x] **EXTR-03**: Pipeline converts PDF pages into readable images (via tools like `pdf2image`).
- [x] **EXTR-04**: OCR engine selection prioritizes (1) Typhoon OCR (Thai specialized), (2) PaddleOCR, then (3) Tesseract.
- [x] **EXTR-05**: OCR precisely captures numerical party/candidate vote counts from the formatted grid.
- [x] **DATA-01**: Data cleaning logic handles common OCR misinterpretations.
- [x] **DATA-02**: Validation logic asserts `sum(candidate votes) == total valid votes`.
- [x] **DATA-03**: Final output is structured as `election_structured_data.csv`.

### Data Analysis Pipeline (Components 2 & 3 & 4)

- [ ] **ANLY-01**: Analysis pipeline implemented completely within a Jupyter Notebook.
- [ ] **ANLY-02**: Trend Analysis — Compare current vote data against historical distributions dating back to the Pheu Thai era.
- [ ] **ANLY-03**: PAO (อบจ) Correlation — Analyze the impact of recent local Administrative Organization elections on these results.
- [ ] **ANLY-04**: Vote Buying Detection — Identify extreme outliers compared to previous elections or neighboring districts.
- [ ] **ANLY-05**: "No Name" Party Identification — Filter out localized vs. national parties by comparing the percentage of constituency candidates fielded globally.
- [ ] **ANLY-06**: Vote Banking Analysis — Compute ratio of Party-List votes to Constituency votes (e.g., highlighting 10-20x gaps).
- [ ] **ANLY-07**: Media & Social Impact — Integrate Google Trends, Facebook/TikTok metric indicators, or News API to correlate engagement with success.
- [ ] **ANLY-08**: Geo-Spatial Vote Anomalies — Detect localized vote concentrations ("vote fogs" / ဖุ้งในเขต).
- [ ] **ANLY-09**: Ballot Number Correlation — Evaluate if candidate/party "number" affects votes (e.g. bias toward lower integer identifiers).
- [ ] **ANLY-10**: Political Dynasty Tracking (บ้านใหญ่) — Examine last names to see if voting power follows the individual shifting parties vs. party loyalty, and check connections with local offices (เทศบาล/อบต).
- [ ] **ANLY-11**: Geographic & Economic Profiling — Correlate local profiles (urban vs agricultural vs industrial) with party/candidate performance within the districts.
- [ ] **DASH-01**: Final visualization acts as an analytical "dashboard" view within the notebook highlighting these insights.

## v2 Requirements

### Output Enhancements
- **ENHC-01**: Export the visual dashboard as a standalone HTML page using Plotly or similar.
- **ENHC-02**: Establish continuous data pipelines fetching from live News APIs for real-time sentiment analysis.

## Out of Scope

| Feature | Reason |
|---------|--------|
| Parallel Processing Framework | Avoid complex distributed environment (e.g., Spark); keep straightforward using Python local multiprocessing if necessary. |
| Country-wide OCR Extraction | Will take too long locally; must stick to representative subset (Uthai Thani 2). |
| Web App Dashboard | Request specifies keeping the pipeline inside an `ipynb` file. |

## Traceability

| Requirement | Phase | Status |
|-------------|-------|--------|
| EXTR-01 | Phase 1 | Complete |
| EXTR-02 | Phase 1 | Complete |
| EXTR-03 | Phase 1 | Complete |
| EXTR-04 | Phase 1 | Complete |
| EXTR-05 | Phase 2 | Complete |
| DATA-01 | Phase 2 | Complete |
| DATA-02 | Phase 2 | Complete |
| DATA-03 | Phase 2 | Complete |
| ANLY-01 | Phase 3 | Pending |
| ANLY-02 | Phase 3 | Pending |
| ANLY-03 | Phase 3 | Pending |
| ANLY-04 | Phase 3 | Pending |
| ANLY-05 | Phase 3 | Pending |
| ANLY-06 | Phase 3 | Pending |
| ANLY-07 | Phase 3 | Pending |
| ANLY-08 | Phase 3 | Pending |
| ANLY-09 | Phase 3 | Pending |
| ANLY-10 | Phase 3 | Pending |
| ANLY-11 | Phase 3 | Pending |
| DASH-01 | Phase 3 | Pending |

**Coverage:**
- v1 requirements: 20 total
- Mapped to phases: 20
- Unmapped: 0 ✓

---
*Requirements defined: 2026-04-09*
*Last updated: 2026-04-09 after user refinement*
