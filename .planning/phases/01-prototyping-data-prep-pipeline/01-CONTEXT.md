# Phase 01: Prototyping Data Prep Pipeline - Context

**Gathered:** 2026-04-09
**Status:** Ready for planning

<domain>
## Phase Boundary

Set up a Jupyter Notebook prototyping environment to evaluate OCR engines (Typhoon OCR vs. PaddleOCR) on Thai election PDFs and establish a robust regex-based extraction strategy.

</domain>

<decisions>
## Implementation Decisions

### OCR Engine Strategy
- **D-01:** Implement a side-by-side comparison in the notebook.
- **D-02:** Use Typhoon OCR (via API) as the primary/specialized engine for Thai text.
- **D-03:** Use PaddleOCR (local execution with CPU/GPU) as a secondary local fallback/comparison point.
- **D-04:** Evaluate engines using both visual inspection and manual accuracy scoring against a ground-truth sample.

### Data Extraction Logic
- **D-05:** Use **Strict Template Matching** for regex extraction. This involves matching exact keyword anchors and newline patterns found in the ส.ส. 5/18 forms to ensure precision over flexibility.
- **D-06:** Parse the full page text rather than localized bounding box crops to avoid misalignment issues from scanned documents.

### Environment & Secrets
- **D-07:** Perform execution locally using CPU/GPU capabilities where applicable (PaddleOCR).
- **D-08:** Store API tokens (Typhoon) and environment-specific configs in a `.env` file using `python-dotenv`.

</decisions>

<canonical_refs>
## Canonical References

**Downstream agents MUST read these before planning or implementing.**

### Project Scope
- `.planning/PROJECT.md` — Core vision and constituency details.
- `.planning/REQUIREMENTS.md` — Full list of EXTR and ANLY requirements.

### Technical Context
- `source/เขตเลือกตั้งที่ 2/` — Target directory containing the raw PDFs.

</canonical_refs>

<deferred>
## Deferred Ideas

- None at this stage.

</deferred>
