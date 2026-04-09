# Thai Election 2026 Data Pipeline (Uthai Thani 2)

## What This Is

An end-to-end data pipeline implemented in Jupyter Notebooks to extract, clean, and analyze official election results from raw PDF documents published by the Election Commission of Thailand (ECT). The analysis focuses on Uthai Thani Constituency 2 (a constituency with at least 250 polling stations) to provide structured texts and uncover actionable insights.

## Core Value

Accurately transforming unstructured, high-volume official PDF election forms into a reliable, structured dataset that drives trustworthy data science visual analysis.

## Requirements

### Validated

<!-- Shipped and confirmed valuable. -->

- ✓ Source election data is locally available in nested directories as PDF files.

### Validated

<!-- Shipped and confirmed valuable. -->

- ✓ Source election data is locally available in nested directories as PDF files.
- ✓ Build Data Extraction Pipeline (Component 1) — Validated in Phase 1: Prototyping Data Prep Pipeline (Typhoon OCR confirmed for Thai text)
- ✓ Implement Data Cleaning and Validation logic — Validated in Phase 2: Scaled Extraction & Validation (retry, in-memory mismatch tracking, CSV export to election_structured_data.csv)

### Active

<!-- Current scope. Building toward these. -->

- [ ] Develop Data Science Analysis Notebook (Component 2) to explore patterns and trends from the parsed dataset.
- [ ] Produce analytical visual dashboard components (Component 3) and summarize insights (Component 4).

### Out of Scope

<!-- Explicit boundaries. Includes reasoning to prevent re-adding. -->

- Processing the entire country's election data — High volume makes OCR processing too slow; we restrict processing to representative case studies.
- Building a full-stack interactive web application — The current goal focuses on creating pipeline logic within `ipynb` files.

## Context

The raw election data is published in several official document forms representing different voting methods:
- ส.ส.5/16 (Constituency/Party-list) : Advance voting (in-district)
- ส.ส.5/17 (Constituency/Party-list) : Advance voting (out-of-district)
- ส.ส.5/18 (Constituency/Party-list) : Election day voting

Forms 5/16 and 5/17 are at the constituency level, while Form 5/18 is at the individual polling station level. Uthai Thani Constituency 2 has been selected as the key target for OCR.

## Constraints

- **Data Source**: Must heavily rely on unstructured, sometimes noisy PDF documents.
- **Scope Limit**: Must select at least 250 polling stations (fulfilled by picking Uthai Thani Constituency 2).
- **Environment**: Python-based data science stack focusing on OCR libraries and pandas.

## Key Decisions

<!-- Decisions that constrain future work. Add throughout project lifecycle. -->

| Decision | Rationale | Outcome |
|----------|-----------|---------|
| Implementing pipelines in Jupyter Notebooks | Easiest format for exploratory data analysis (EDA) and sequential data extraction steps | — Pending |

## Evolution

This document evolves at phase transitions and milestone boundaries.

**After each phase transition** (via `/gsd-transition`):
1. Requirements invalidated? → Move to Out of Scope with reason
2. Requirements validated? → Move to Validated with phase reference
3. New requirements emerged? → Add to Active
4. Decisions to log? → Add to Key Decisions
5. "What This Is" still accurate? → Update if drifted

**After each milestone** (via `/gsd-complete-milestone`):
1. Full review of all sections
2. Core Value check — still the right priority?
3. Audit Out of Scope — reasons still valid?
4. Update Context with current state

---
*Last updated: 2026-04-09 after Phase 2 completion — scaled extraction pipeline and CSV export ready*
