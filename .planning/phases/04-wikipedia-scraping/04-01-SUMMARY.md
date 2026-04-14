---
phase: 04-wikipedia-scraping
plan: 01
subsystem: data-acquisition
tags: [requests, beautifulsoup, wikipedia, scraping, pandas, csv, thai-elections]

requires:
  - phase: 02-scaled-extraction-validation
    provides: Project notebook infrastructure and data pipeline patterns

provides:
  - notebooks/03_wikipedia_scrape.ipynb — fully executable scraper using requests+BeautifulSoup
  - data/wikipedia/wikipedia_election_results.csv — 318 rows long-format election data (2001-2026)
  - Helper functions: fetch_page_html, find_results_table, get_cell_text, parse_int, parse_float, extract_row_data, parse_results_table

affects:
  - Phase 3: ANLY-02 historical trend analysis now has its reference dataset

tech-stack:
  added: [nbconvert (for notebook execution)]
  patterns:
    - BeautifulSoup heading-sibling walk with div.mw-heading parent fallback for Wikipedia HTML
    - Per-year column index map for wikitable parsing (standard 10-col vs 2019 7-col layout)
    - sup.decompose() footnote stripping before numeric cell extraction
    - Long-format DataFrame assembly with per-year sanity count assertions

key-files:
  created:
    - notebooks/03_wikipedia_scrape.ipynb
    - data/wikipedia/wikipedia_election_results.csv (gitignored, produced by running notebook)
  modified: []

key-decisions:
  - "Wikipedia HTML structure: h2/h3 headings are now wrapped in div.mw-heading — must walk parent.find_next_siblings() not heading.find_next_siblings()"
  - "2019 election table has 7 tds per data row (swatch+party+votes+%+FPTP+List+Total); summary rows have 6 tds and are skipped by len check"
  - "Party-list votes used as the votes field for standard layout years (2001, 2005, 2007, 2011, 2023, 2026); single Votes column for 2019"
  - "data/wikipedia/ is gitignored — CSV is runtime output; notebook is the committed deliverable"

patterns-established:
  - "Wikipedia scraping: always walk div.mw-heading parent siblings, not heading siblings directly"
  - "2019 wikitable exception: 7-column layout vs standard 10-column layout across all other years"
  - "Universal sup.decompose() on every cell before get_text() to handle footnote markers ([a], [ag], [1])"

requirements-completed: [WIKI-01, WIKI-02, WIKI-03, WIKI-04, WIKI-05, WIKI-06, WIKI-07]

duration: 7min
completed: 2026-04-14
---

# Phase 4 Plan 1: Wikipedia Scraping Summary

**requests+BeautifulSoup HTML table scraper for 7 Thai elections, exporting 318 rows (2001-2026) to wikipedia_election_results.csv**

## Performance

- **Duration:** 7 min
- **Started:** 2026-04-14T08:05:33Z
- **Completed:** 2026-04-14T08:12:49Z
- **Tasks:** 2
- **Files modified:** 1 (notebook), 1 (CSV, gitignored)

## Accomplishments

- Replaced `wikipedia-api` plain-text client with `requests` + `BeautifulSoup` HTML scraper
- Implemented 8 helper functions (fetch_page_html, find_results_table, get_cell_text, parse_int, parse_float, extract_row_data, parse_results_table) with all anti-patterns from RESEARCH.md avoided
- Scraped and exported 318 party-year rows across all 7 Thai elections (2001, 2005, 2007, 2011, 2019, 2023, 2026) with per-year counts: 2001:31, 2005:21, 2007:27, 2011:34, 2019:77, 2023:69, 2026:59
- Notebook executes end-to-end producing valid CSV with correct header, all 7 years, no junk rows

## Task Commits

1. **Task 1: Rewrite notebook with scraping helper functions** - `fda2e3d` (feat)
2. **Task 2: Implement main scraping loop and export CSV** - `1197ba8` (feat)

## Files Created/Modified

- `notebooks/03_wikipedia_scrape.ipynb` — Complete rewrite: 15 cells with all helpers, main loop, DataFrame builder, CSV export, and verification
- `data/wikipedia/wikipedia_election_results.csv` — 318 rows, columns: party,year,seats,votes,vote_pct (gitignored, runtime output)

## Decisions Made

- Used party-list votes as the `votes` field for standard-layout years (2001, 2005, 2007, 2011, 2023, 2026) — the party-list ballot represents national party support; constituency votes are district-level
- 2019 uses a single "Votes" column (constituency FPTP votes) due to different table structure
- CSV output to `data/wikipedia/` (gitignored) — data files are not committed; notebook is the deliverable

## 2019 Column Layout — Open Question 1 Resolved

Confirmed at runtime: 2019 data rows have **7 tds** per row:
- `td[0]` = swatch, `td[1]` = party, `td[2]` = Votes, `td[3]` = %, `td[4]` = FPTP seats, `td[5]` = List seats, `td[6]` = Total seats

Summary rows ("None of the above", "Total") have 6 tds and are excluded by the `len(cells) < 7` guard.

## Deviations from Plan

### Auto-fixed Issues

**1. [Rule 1 - Bug] Fixed find_results_table for modern Wikipedia mw-heading HTML structure**
- **Found during:** Task 2 (first notebook execution attempt)
- **Issue:** Wikipedia now wraps all `h2`/`h3` headings inside `div.mw-heading` container elements. The original implementation walked `heading.find_next_siblings()`, but the wikitable is a sibling of the parent `div`, not the heading. This raised `ValueError: No results wikitable found on page` for all pages.
- **Fix:** Added parent detection: if `heading.parent.name == "div"` and has `mw-heading` class, walk `parent.find_next_siblings()` instead. Old structure kept as fallback.
- **Files modified:** notebooks/03_wikipedia_scrape.ipynb (cell-05-find-results-table)
- **Verification:** All 7 election pages successfully parsed
- **Committed in:** 1197ba8 (Task 2 commit)

**2. [Rule 1 - Bug] Fixed extract_row_data 2019 cell count guard causing IndexError**
- **Found during:** Task 2 (second notebook execution attempt, 2019 page)
- **Issue:** The original code checked `if len(cells) < 6: return None` for the 2019 branch, then accessed `cells[6]`. 2019 summary rows (Total, None of the above) have exactly 6 tds, passing the guard but raising IndexError on `cells[6]`.
- **Fix:** Changed the 2019 branch guard to `if len(cells) < 7: return None` so only rows with the full 7-column data layout proceed to index access.
- **Files modified:** notebooks/03_wikipedia_scrape.ipynb (cell-08-extract-row-data)
- **Verification:** 2019 produces 77 party rows; no junk rows; no IndexError
- **Committed in:** 1197ba8 (Task 2 commit)

---

**Total deviations:** 2 auto-fixed bugs
**Impact on plan:** Both fixes were required for correctness — the Wikipedia HTML structure change affected all 7 pages, and the 2019 cell count guard was a logic error in the plan's column index map. No scope creep.

## Known Stubs

None. All data flows from live Wikipedia pages to CSV. The notebook is fully wired end-to-end.

## Issues Encountered

- Wikipedia changed its article HTML structure (headings now inside `div.mw-heading`). The RESEARCH.md pattern assumed direct heading siblings. Fixed by walking parent div siblings.
- 2019 table has fewer tds in summary rows (6) than party rows (7), requiring a stricter minimum cell count guard for the 2019 branch.

## Self-Check: PASSED

- FOUND: notebooks/03_wikipedia_scrape.ipynb
- FOUND: data/wikipedia/wikipedia_election_results.csv (runtime output, gitignored)
- FOUND: commit fda2e3d (Task 1)
- FOUND: commit 1197ba8 (Task 2)
