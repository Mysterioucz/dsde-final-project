---
wave: 2
depends_on: ["01-PLAN.md"]
files_modified: ["notebooks/02_scaled_extraction.ipynb"]
autonomous: false
requirements: ["DATA-02", "DATA-03"]
---

# Plan 02-02: Retry Mechanism, Validation, & Export

## Objective
Implement logic to re-run failed operations specifically from the structured error log, ensure data validation criteria are met without flagging the CSV output directly, and export the valid dataset.

## Tasks

```xml
<task>
  <description>Implement Rerun/Retry Mechanism</description>
  <action>Add a cell in `notebooks/02_scaled_extraction.ipynb` that reads `logs/failed_extractions.json`. For each failed `pdf_path`, re-invoke the rate-limited Typhoon OCR pipeline. Successful extractions on pass 2 should be appended to the master dataframe and removed from the failed log.</action>
  <read_first>["notebooks/02_scaled_extraction.ipynb", ".planning/phases/02-scaled-extraction-validation/02-CONTEXT.md"]</read_first>
  <acceptance_criteria>
    - Cell exists specifically for processing `logs/failed_extractions.json`.
    - Functionality only runs on previously failed files.
  </acceptance_criteria>
</task>

<task>
  <description>Implement Validation Logic (DATA-02)</description>
  <action>Create a validation cell that iterates over the parsed dataframe. Calculate `sum(candidate_votes)` and compare against `parsed_total`. If they do NOT match, add the row index to a Python list `validation_mismatches`. Do NOT write an error flag out to the CSV file.</action>
  <read_first>["notebooks/02_scaled_extraction.ipynb"]</read_first>
  <acceptance_criteria>
    - Mismatches are tracked in a `validation_mismatches` list in the notebook environment.
    - Final output dataframe exported does not contain a "Mismatch" flag column.
  </acceptance_criteria>
</task>

<task>
  <description>Data Cleanup & Export (DATA-03)</description>
  <action>Clean common OCR numerical misinterpretations (e.g., "O" -> "0") in the pandas dataframe before export. Save the final dataframe as `output/election_structured_data.csv`.</action>
  <read_first>["notebooks/02_scaled_extraction.ipynb"]</read_first>
  <acceptance_criteria>
    - Cleaning logic is applied via specific pandas `.replace()` or string replacements.
    - Output `election_structured_data.csv` is created.
  </acceptance_criteria>
</task>
```

## Verification
- Review the exported `election_structured_data.csv` and ensure all rows (even mismatches) are present and lack a validation flag.
- Display `validation_mismatches` visually in the notebook for user review.

## must_haves
- [ ] Rerun logic processes only from the error log.
- [ ] No mismatch flags are stored in the final `.csv`.
- [ ] Final output is named `election_structured_data.csv`.
