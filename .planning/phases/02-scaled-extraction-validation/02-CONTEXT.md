# Phase 2: Scaled Extraction & Validation - Context

**Gathered:** 2026-04-09
**Status:** Ready for planning

<domain>
## Phase Boundary

Execute the OCR across all 250+ PDFs for Uthai Thani 2, clean common anomalies, and validate against logical vote sum totals. Export a structured CSV dataset.

</domain>

<decisions>
## Implementation Decisions

### OCR Engine Strategy
- **D-01:** Exclusively use Typhoon OCR (via API). We will NOT use PaddleOCR or Tesseract as fallbacks in this pipeline.

### Error Handling & Execution
- **D-02:** When extraction fails (e.g., due to network limits or regex mismatch), log the error and skip the file. Do not crash the pipeline. Importantly, this error log must be structured so the pipeline can use it later to re-run the OCR process *only* on the un-completed/failed stations.
- **D-03:** Execute parallel processing with a queue/batching system configured precisely to respect Typhoon API rate limits (2 requests/sec, 20 requests/min max) to optimize throughput without rate-limit blocking.

### Data Validation Strategy
- **D-04:** If candidate votes do not mathematically match the explicit "Total" parsed from the document, do *not* output an error flag to the final CSV. Instead, maintain an internal tracker (variable/list/log) in the notebook environment that alerts the notebook runner to manually inspect the misaligned forms.

</decisions>

<canonical_refs>
## Canonical References

**Downstream agents MUST read these before planning or implementing.**

### Project Scope
- `.planning/PROJECT.md` — Core vision and project goals.
- `.planning/ROADMAP.md` — Sequential goals for Phase 2.
- `.planning/REQUIREMENTS.md` — Specifies the validation and cleaning requirements for this extraction.

</canonical_refs>

<deferred>
## Deferred Ideas

- None.
</deferred>
