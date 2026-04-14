# Phase 4: Wikipedia Scraping - Context

**Gathered:** 2026-04-14
**Status:** Ready for planning

<domain>
## Phase Boundary

Scrape historical party-level election results (seats won, total votes, vote share %) from English Wikipedia pages for Thai general elections (2001–2026). Build a structured long-format CSV dataset that supports the comparative trend analyses planned in Phase 3 (ANLY-02 and related).

This phase delivers **reference data acquisition only** — cleaning, merging, and actual analysis are Phase 3's responsibility.

</domain>

<decisions>
## Implementation Decisions

### Data Fields
- **D-01:** Extract three fields per party per election: `seats` (House seats won), `votes` (total votes received), `vote_pct` (vote share percentage).
- **D-02:** Capture **total seats only** — do NOT split into constituency (เขต) vs party-list (บัญชีรายชื่อ) seats. Table formats vary significantly across years; total seats is the most consistently available column.

### Scraping Method
- **D-03:** Use `requests` + `BeautifulSoup` for HTML table parsing. The existing `wikipedia-api` client in `03_wikipedia_scrape.ipynb` outputs plain text which cannot reliably parse tabular data — replace it with direct HTML fetching.
- **D-04:** Add a polite delay of ~1 second between page fetches to respect Wikipedia's rate limiting guidelines.
- **D-05:** Use the User-Agent string already established in the notebook: `dsde-final-proj (chatrinza@gmail.com)`.

### Elections Scope
- **D-06:** Scrape all 7 elections defined in the notebook:
  - 2001, 2005, 2007, 2011, 2019, 2023, 2026
- **D-07:** Skip 2008 and 2014 — those elections occurred under special circumstances (military/post-coup coalition) and their Wikipedia pages have inconsistent table structures.

### Output Format & Storage
- **D-08:** Output as **long format** CSV: one row per (party, election_year). Columns: `party | year | seats | votes | vote_pct`.
- **D-09:** Save output to `data/wikipedia/` — a new dedicated directory for Wikipedia-sourced data, separate from OCR-extracted data in `data/previous-election/`.
- **D-10:** Suggested filename: `wikipedia_election_results.csv`.

### Notebook
- **D-11:** Implement in the existing `notebooks/03_wikipedia_scrape.ipynb`. Replace or extend existing cells rather than creating a new notebook.

</decisions>

<canonical_refs>
## Canonical References

**Downstream agents MUST read these before planning or implementing.**

### Project Scope
- `.planning/PROJECT.md` — Core vision and project context.
- `.planning/ROADMAP.md` — Phase 4 goal and dependency on Phase 3.
- `.planning/REQUIREMENTS.md` — ANLY-02 (historical trend analysis) is the primary requirement this phase feeds.

### Existing Code to Extend
- `notebooks/03_wikipedia_scrape.ipynb` — Already has the 7 election URLs and a wikipedia-api client. Extend this notebook using requests + BeautifulSoup instead.

### Wikipedia Target Pages
- English Wikipedia pages for Thai general elections (2001, 2005, 2007, 2011, 2019, 2023, 2026) — URLs already in the notebook's `URLS` list.

### Existing Data Reference
- `data/previous-election/election_scores_2566.csv` — Station-level 2023 data already available. Wikipedia data goes in a separate directory (`data/wikipedia/`).

</canonical_refs>

<deferred>
## Deferred Ideas

- Constituency vs party-list seat split — decided against due to inconsistent table formats across election years. Could be revisited as a separate enrichment step.
- 2008 / 2014 special elections — skipped for now; could be added later if needed.
- Merging Wikipedia data with OCR CSV — out of scope for this phase; belongs in Phase 3 analysis.

</deferred>
