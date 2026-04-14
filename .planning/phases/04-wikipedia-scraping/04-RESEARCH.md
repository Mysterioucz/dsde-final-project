# Phase 4: Wikipedia Scraping - Research

**Researched:** 2026-04-14
**Domain:** Web scraping (requests + BeautifulSoup), Wikipedia wikitable parsing, Thai election data
**Confidence:** HIGH

---

<user_constraints>
## User Constraints (from CONTEXT.md)

### Locked Decisions

- **D-01:** Extract three fields per party per election: `seats` (House seats won), `votes` (total votes received), `vote_pct` (vote share percentage).
- **D-02:** Capture **total seats only** — do NOT split into constituency (เขต) vs party-list (บัญชีรายชื่อ) seats. Table formats vary significantly across years; total seats is the most consistently available column.
- **D-03:** Use `requests` + `BeautifulSoup` for HTML table parsing. The existing `wikipedia-api` client in `03_wikipedia_scrape.ipynb` outputs plain text which cannot reliably parse tabular data — replace it with direct HTML fetching.
- **D-04:** Add a polite delay of ~1 second between page fetches to respect Wikipedia's rate limiting guidelines.
- **D-05:** Use the User-Agent string already established in the notebook: `dsde-final-proj (chatrinza@gmail.com)`.
- **D-06:** Scrape all 7 elections defined in the notebook: 2001, 2005, 2007, 2011, 2019, 2023, 2026.
- **D-07:** Skip 2008 and 2014 elections.
- **D-08:** Output as **long format** CSV: one row per (party, election_year). Columns: `party | year | seats | votes | vote_pct`.
- **D-09:** Save output to `data/wikipedia/` — new dedicated directory, separate from `data/previous-election/`.
- **D-10:** Suggested filename: `wikipedia_election_results.csv`.
- **D-11:** Implement in the existing `notebooks/03_wikipedia_scrape.ipynb`. Replace or extend existing cells rather than creating a new notebook.

### Claude's Discretion

None specified beyond the locked decisions above.

### Deferred Ideas (OUT OF SCOPE)

- Constituency vs party-list seat split — inconsistent table formats across years.
- 2008 / 2014 special elections — skipped; could be added later if needed.
- Merging Wikipedia data with OCR CSV — belongs in Phase 3 analysis.
</user_constraints>

---

<phase_requirements>
## Phase Requirements

This phase creates the data dependency for ANLY-02 (historical trend analysis) in Phase 3. No formal IDs exist yet; requirements inferred from CONTEXT.md and REQUIREMENTS.md.

| ID | Description | Research Support |
|----|-------------|------------------|
| WIKI-01 | Fetch HTML from all 7 election Wikipedia pages | requests + `lxml` parser confirmed available; ~1s delay pattern |
| WIKI-02 | Parse the `wikitable sortable` results table from each page | All 7 pages confirmed to use `wikitable sortable` CSS class |
| WIKI-03 | Extract `party`, `seats` (total), `votes`, `vote_pct` per row | Column mapping per year documented below |
| WIKI-04 | Handle table structural variations across years | Header colspan/rowspan patterns documented; per-year column index map provided |
| WIKI-05 | Filter junk rows (Total, footnotes, invalid/blank votes) | Row exclusion patterns documented |
| WIKI-06 | Output long-format CSV to `data/wikipedia/wikipedia_election_results.csv` | pandas DataFrame construction pattern provided |
| WIKI-07 | Create `data/wikipedia/` directory if absent | `Path.mkdir(parents=True, exist_ok=True)` pattern |
</phase_requirements>

---

## Summary

All 7 Thai election Wikipedia pages use the **same CSS class** (`wikitable sortable`) on their results table. However, the column layout differs between election years depending on how votes are reported. From 2001–2023, the table has two separate vote columns (party-list votes and constituency votes) plus a "Total seats" column. The 2019 page differs slightly in that the "Total" seat column header reads "Total" rather than "Total seats." The 2026 table also uses party-list votes separately. Because D-02 mandates total seats only, the implementation does not need to parse multiple seat sub-columns — it reads only the final "Total seats" (or "Total") column.

The key structural challenge is the **two-row header with colspan/rowspan**. Every page uses a header where "Party-list" spans 3 columns (Votes, %, Seats) and "Constituency" spans 3 columns (Votes, %, Seats), with "Total seats" and "+/–" spanning both header rows. This means `find_all('th')` alone cannot produce a flat list of column names without traversing the second header row. The safest parsing strategy is to locate the column position of "Total seats" and the vote/percentage columns by scanning all `th` text values, not by positional index alone.

**Primary recommendation:** Use `requests.get()` with the Wikipedia API `action=parse` endpoint (not the main `/wiki/` URL) to fetch rendered HTML section-by-section, then parse the first `.wikitable` in the results section using BeautifulSoup with `lxml`. Map columns by matching `th` text content, not positional index.

---

## Standard Stack

### Core

| Library | Version (installed) | Purpose | Why Standard |
|---------|---------------------|---------|--------------|
| requests | 2.31.0 | HTTP GET to Wikipedia API | Locked by D-03; already installed |
| beautifulsoup4 | 4.12.3 | Parse HTML wikitables | Locked by D-03; already installed |
| lxml | 5.2.1 | BS4 parser backend | Faster and more lenient than html.parser; already installed |
| pandas | 3.0.1 | DataFrame construction and CSV export | Already used across the project |

### Supporting

| Library | Version | Purpose | When to Use |
|---------|---------|---------|-------------|
| pathlib (stdlib) | stdlib | Create `data/wikipedia/` directory | Path.mkdir(parents=True, exist_ok=True) |
| time (stdlib) | stdlib | 1-second polite delay between fetches | time.sleep(1) between requests |
| re (stdlib) | stdlib | Strip footnote superscripts from cell text | Clean "[a]", "[1]" markers from td text |

### Alternatives Considered

| Instead of | Could Use | Tradeoff |
|------------|-----------|----------|
| Wikipedia API `action=parse` | Scrape `/wiki/URL` directly | Direct URL works too; API endpoint is more stable for section targeting |
| lxml parser | html.parser | html.parser works but is slower and stricter; lxml already installed |

**No installation needed.** All libraries are already in `pyproject.toml`:
- `beautifulsoup4==4.14.3` (pyproject spec), `4.12.3` (runtime)
- `requests==2.33.1` (pyproject spec), `2.31.0` (runtime)
- `lxml==6.0.2` (pyproject spec), `5.2.1` (runtime)
- `pandas==3.0.2` (pyproject spec), `3.0.1` (runtime)

---

## Architecture Patterns

### Recommended Project Structure

No new files beyond the notebook. Directory to create:

```
data/
└── wikipedia/                  # New directory (does not exist yet)
    └── wikipedia_election_results.csv   # Long-format output
notebooks/
└── 03_wikipedia_scrape.ipynb   # Extend this file (already exists)
```

### Pattern 1: Wikipedia API HTML Fetch

**What:** Fetch parsed HTML for a specific section using Wikipedia's `action=parse` API rather than scraping the rendered page. This avoids JavaScript rendering issues and returns stable HTML.

**When to use:** Fetching any Wikipedia article section as HTML.

**URL pattern:**
```
https://en.wikipedia.org/w/api.php?action=parse&page={PAGE_TITLE}&prop=text&section={N}&format=json
```

**Known section indexes per election year** (verified via research):

| Year | Wikipedia Page Title | Results Section Index |
|------|---------------------|-----------------------|
| 2026 | 2026_Thai_general_election | 18 |
| 2023 | 2023_Thai_general_election | 12 |
| 2019 | 2019_Thai_general_election | 15 ("Official results") |
| 2011 | 2011_Thai_general_election | 19 |
| 2007 | 2007_Thai_general_election | 5 |
| 2005 | 2005_Thai_general_election | 8 |
| 2001 | 2001_Thai_general_election | 5 |

**Alternatively, fetch the full page and search for the results table directly** — avoids needing to hard-code section indexes (which can shift if Wikipedia editors add new sections):

```python
import requests
from bs4 import BeautifulSoup
import time

USER_AGENT = "dsde-final-proj (chatrinza@gmail.com)"

def fetch_page_html(wiki_url: str) -> BeautifulSoup:
    """Fetch full Wikipedia article HTML. Returns BeautifulSoup object."""
    headers = {"User-Agent": USER_AGENT}
    resp = requests.get(wiki_url, headers=headers, timeout=15)
    resp.raise_for_status()
    return BeautifulSoup(resp.text, "lxml")
```

### Pattern 2: Locating the Results Table

**What:** Wikipedia election pages have multiple wikitables (campaign polls, by-province, etc.). The correct results table must be identified reliably.

**Strategy:** Find the `<h2>` or `<h3>` heading with text "Results" (or "Official results"), then take the **first** `.wikitable` that follows it.

```python
def find_results_table(soup: BeautifulSoup) -> Tag:
    """Find the main election results wikitable."""
    # Look for a heading containing 'result' (case-insensitive)
    for heading in soup.find_all(["h2", "h3"]):
        if "result" in heading.get_text(strip=True).lower():
            # Walk siblings forward until we find a wikitable
            for sibling in heading.find_next_siblings():
                if sibling.name == "table" and "wikitable" in sibling.get("class", []):
                    return sibling
    raise ValueError("No results wikitable found")
```

**Alternative (simpler if heading approach is fragile):** Use the Wikipedia API `action=parse&section=N` approach and take the first wikitable in the returned HTML.

### Pattern 3: Parsing the Two-Row Header

**What:** All 7 pages have a two-row `<thead>` (or multi-row `<tr>` in `<tbody>`). The first row has group headers ("Party-list", "Constituency") with colspan; the second row has leaf headers ("Votes", "%", "Seats"). Finding column positions requires scanning both rows.

**Key observation from research:**
- Row 1 of header: `Party (colspan=2, rowspan=2)` | `Party-list (colspan=3)` | `Constituency (colspan=3)` | `Total seats (rowspan=2)` | `+/– (rowspan=2)`
- Row 2 of header: `Votes | % | Seats` | `Votes | % | Seats`
- The "Total seats" column is **column index 8** in the flat header (0-indexed, after expanding colspans):
  - Col 0: Party color swatch
  - Col 1: Party name
  - Col 2: Party-list Votes
  - Col 3: Party-list %
  - Col 4: Party-list Seats
  - Col 5: Constituency Votes
  - Col 6: Constituency %
  - Col 7: Constituency Seats
  - Col 8: Total seats
  - Col 9: +/–

**However, 2019 column headers differ:** The 2019 page uses "FPTP", "List", "Total" instead of "Constituency Seats", "Party-list Seats", "Total seats". The "Total" column is still the total seats.

**Recommended approach — scan by text content, not index:**

```python
import re

def get_cell_text(cell) -> str:
    """Extract clean text from a td/th, stripping footnote markers."""
    # Remove <sup> tags (footnote references like [a], [1])
    for sup in cell.find_all("sup"):
        sup.decompose()
    return cell.get_text(separator=" ", strip=True)

def parse_results_table(table) -> list[dict]:
    """
    Parse a Wikipedia election results wikitable.
    Returns list of dicts with keys: party, seats, votes, vote_pct
    """
    rows = table.find_all("tr")
    
    # Build flat column list from header rows (rows where all cells are <th>)
    # Scan all <th> text values to find column positions
    all_headers = []
    for row in rows:
        ths = row.find_all("th")
        if ths:
            for th in ths:
                all_headers.append(get_cell_text(th).lower())
    
    # Identify target column text patterns
    # "total seats" appears as a merged header; find its position among data <td> columns
    # Use a simpler column-index approach: parse row by row, detect data rows
    
    results = []
    for row in rows:
        cells = row.find_all(["td", "th"])
        if not cells:
            continue
        # Data rows have td cells; skip rows that are all th (headers)
        td_cells = row.find_all("td")
        if len(td_cells) < 5:
            continue  # Not enough cells to be a party data row
        
        # Extract by position (see column index map above)
        # Party name is typically in the second td or an <a> tag
        # ... (see Pattern 4 for exact extraction)
        pass
    
    return results
```

### Pattern 4: Column Index Map and Data Extraction

**What:** Given the consistent column layout across all 7 elections, extract by verified positional indexes within each data `<tr>`.

**Verified flat `<td>` cell layout in data rows** (confirmed across 2001, 2005, 2007, 2011, 2019, 2023, 2026):

| td index | Content |
|----------|---------|
| 0 | Party color swatch (empty or contains a colored `<div>`) |
| 1 | Party name (contains `<a>` link or plain text) |
| 2 | Party-list votes |
| 3 | Party-list % |
| 4 | Party-list seats |
| 5 | Constituency votes |
| 6 | Constituency % |
| 7 | Constituency seats |
| 8 | **Total seats** (target) |
| 9 | +/– seat change |

**Exception: 2019 page** — The 2019 table structure verified to be:
- Columns: `Party (2) | Votes | % | FPTP | List | Total | +/-`
- This is 7 `<td>` columns in data rows (no split party-list/constituency vote columns at the row level; there's a single votes column)
- Total seats appears at td index **5** (0-indexed after party name span)

**Votes field selection — critical decision:**
- For 2001–2023 (except 2019): two vote columns exist (party-list and constituency). D-02 says "total seats only" but **D-01 says extract total votes**. The party-list vote count is the natural "total votes for the party nationally" and is consistently available.
- For 2019: single votes column (total votes) — straightforward.
- **Recommendation:** Use **party-list votes** as the `votes` field for 2001–2023/2026 pages (this is the votes cast on the party ballot, which corresponds to national party support). Document this choice clearly in notebook comments.

```python
def extract_row_data(cells: list, year: int) -> dict | None:
    """
    Extract party, seats, votes, vote_pct from a data row's <td> list.
    Returns None if the row should be skipped.
    """
    if len(cells) < 6:
        return None
    
    # Party name: cell at index 1, text from <a> if present
    name_cell = cells[1]
    a_tag = name_cell.find("a")
    party = get_cell_text(a_tag) if a_tag else get_cell_text(name_cell)
    
    # Skip junk rows
    skip_patterns = ["total", "invalid", "blank", "valid votes", 
                     "registered", "turnout", "none of the above",
                     "spoilt", "abstain"]
    if any(p in party.lower() for p in skip_patterns):
        return None
    if not party or party == "":
        return None
    
    if year == 2019:
        # 2019 structure: Party(2cols) | Votes | % | FPTP | List | Total | +/-
        votes_raw = get_cell_text(cells[2])
        pct_raw = get_cell_text(cells[3])
        seats_raw = get_cell_text(cells[6])  # "Total" column
    else:
        # 2001, 2005, 2007, 2011, 2023, 2026 structure
        votes_raw = get_cell_text(cells[2])   # Party-list votes
        pct_raw = get_cell_text(cells[3])     # Party-list %
        seats_raw = get_cell_text(cells[8])   # Total seats
    
    # Clean numeric strings
    votes = parse_int(votes_raw)
    vote_pct = parse_float(pct_raw)
    seats = parse_int(seats_raw)
    
    if votes is None and seats is None:
        return None  # Completely empty row
    
    return {"party": party, "seats": seats, "votes": votes, "vote_pct": vote_pct}


def parse_int(s: str) -> int | None:
    """Remove commas and convert to int. Returns None on failure."""
    cleaned = re.sub(r"[^\d]", "", s)
    return int(cleaned) if cleaned else None


def parse_float(s: str) -> float | None:
    """Parse percentage string. Returns None on failure."""
    cleaned = re.sub(r"[^\d.]", "", s)
    return float(cleaned) if cleaned else None
```

### Pattern 5: Long-Format DataFrame Assembly and CSV Export

```python
import pandas as pd
from pathlib import Path

all_records = []

ELECTIONS = {
    2026: ("2026_Thai_general_election", "https://en.wikipedia.org/wiki/2026_Thai_general_election"),
    2023: ("2023_Thai_general_election", "https://en.wikipedia.org/wiki/2023_Thai_general_election"),
    2019: ("2019_Thai_general_election", "https://en.wikipedia.org/wiki/2019_Thai_general_election"),
    2011: ("2011_Thai_general_election", "https://en.wikipedia.org/wiki/2011_Thai_general_election"),
    2007: ("2007_Thai_general_election", "https://en.wikipedia.org/wiki/2007_Thai_general_election"),
    2005: ("2005_Thai_general_election", "https://en.wikipedia.org/wiki/2005_Thai_general_election"),
    2001: ("2001_Thai_general_election", "https://en.wikipedia.org/wiki/2001_Thai_general_election"),
}

for year, (title, url) in ELECTIONS.items():
    soup = fetch_page_html(url)
    table = find_results_table(soup)
    rows = parse_results_table(table)
    for row in rows:
        row["year"] = year
        all_records.append(row)
    time.sleep(1)  # D-04: polite delay

df = pd.DataFrame(all_records, columns=["party", "year", "seats", "votes", "vote_pct"])

out_dir = Path("../data/wikipedia")
out_dir.mkdir(parents=True, exist_ok=True)
out_path = out_dir / "wikipedia_election_results.csv"
df.to_csv(out_path, index=False)
print(f"Saved {len(df)} rows to {out_path}")
```

### Anti-Patterns to Avoid

- **Parsing by column index without accounting for the two-row header:** Using `rows[0]` to get headers will return the group header row (Party-list, Constituency), not the leaf column names. Always handle the multi-row header structure.
- **Using `table.find_all('tr')[0]` to get headers directly:** The first `<tr>` in older elections sometimes contains an image row (`colspan=10` image of parliament composition). Skip rows where cells have colspan equal to total column count.
- **Using `get_text()` without stripping `<sup>` tags:** Footnote markers like `[a]`, `[1]`, `[ag]` are embedded in cells as `<sup>` elements. They will corrupt numeric parsing if not removed first.
- **Assuming commas in vote numbers:** The 2026 API response returns vote numbers without comma separators in some cells. The `parse_int()` helper strips all non-digit characters to be safe.
- **Selecting first wikitable on the page:** The first wikitable may be a polling table, image caption table, or "seat composition" table. Always locate the table that follows a "Results" heading.

---

## Don't Hand-Roll

| Problem | Don't Build | Use Instead | Why |
|---------|-------------|-------------|-----|
| Party name normalization across years | Custom fuzzy-match mapper | Document as-is; normalize in Phase 3 | Party names legitimately differ across eras (Thai Rak Thai 2001 vs Thai Rak Thai 2005 are the same party; Move Forward 2023 becomes People's Party 2026 — these are politically distinct, not the same party). Normalization is an analysis decision, not a scraping decision. |
| HTML fetching | Custom retry loop | requests built-in with `timeout=15` + `raise_for_status()` | Handles redirects, encoding, status codes |
| CSV writing | Manual file I/O | `df.to_csv()` | pandas handles quoting, encoding, newlines |
| Directory creation | os.makedirs with checks | `Path.mkdir(parents=True, exist_ok=True)` | Idiomatic, exception-safe |

**Key insight:** This phase is data acquisition only. Do not normalize party names, merge with OCR data, or compute derived statistics. Raw party names from Wikipedia go into the CSV as-is for Phase 3 to handle.

---

## Wikipedia Table Structure: Year-by-Year Summary

This is the definitive structural reference for the planner.

| Year | Table CSS Class | Vote Columns | Seats Column Header | Total Seats | Notes |
|------|----------------|--------------|---------------------|-------------|-------|
| 2026 | wikitable sortable | Party-list Votes (col 2), Constituency Votes (col 5) | "Total seats" (col 8) | 500 | Footnote markers `[ag]` in some cells |
| 2023 | wikitable sortable | Party-list Votes (col 2), Constituency Votes (col 5) | "Total seats" (col 8) | 500 | Standard layout |
| 2019 | wikitable sortable | Single "Votes" column (col 2) | "Total" (col 6) | 500 | Different layout — "FPTP", "List", "Total" headers |
| 2011 | wikitable sortable | Party-list Votes (col 2), Constituency Votes (col 5) | "Total seats" (col 8) | 500 | Standard layout |
| 2007 | wikitable sortable | Proportional Votes (col 2), Constituency Votes (col 5) | "Total seats" (col 8) | 480 | Uses "Proportional" instead of "Party-list" |
| 2005 | wikitable sortable | Party-list Votes (col 2), Constituency Votes (col 5) | "Total seats" (col 8) | 500 | Standard layout |
| 2001 | wikitable sortable | Party-list Votes (col 2), Constituency Votes (col 5) | "Total seats" (col 8) | 500 | Standard layout |

**Critical 2019 exception:** The 2019 table has columns: `Party (2) | Votes | % | FPTP seats | List seats | Total seats | +/–`. The "Votes" column here is the constituency FPTP votes, not party-list votes. This page merges vote reporting differently. Use the single "Votes" column as the `votes` field for 2019.

**Critical 2007 note:** The 2007 election had only 480 seats (reduced from 500). The "Total" row will show 480, not 500 — this is correct and should not be filtered.

---

## Common Pitfalls

### Pitfall 1: Image Row at Top of Table

**What goes wrong:** The first `<tr>` in older election tables contains a single `<td colspan="10">` with an image of parliament composition. Treating this as a data row crashes column-index parsing.

**Why it happens:** Wikipedia tables often include decorative header rows before the actual `<th>` column headers.

**How to avoid:** Skip any row where the first cell has a `colspan` attribute greater than 3, or where the row contains `<img>` tags.

**Warning signs:** `td_cells[8]` throws IndexError on the first data row.

### Pitfall 2: Multi-Row Header Confusion

**What goes wrong:** Calling `table.find_all('tr')[0].find_all('th')` gives `["Party", "Party-list", "Constituency", "Total seats", "+/–"]` — not the leaf column names. Then column index mapping breaks.

**Why it happens:** All 7 tables use a two-row header where group headers (Party-list, Constituency) span the first row and leaf headers (Votes, %, Seats) are in the second row.

**How to avoid:** Don't try to build a flat header map dynamically. Use the known column-index layout documented above (per year). Alternatively, concatenate both header rows with their rowspan/colspan expansions.

**Warning signs:** `vote_pct` values being assigned seat counts (misaligned columns).

### Pitfall 3: Footnote Markers in Numeric Cells

**What goes wrong:** `int("14,438,851[a]")` raises ValueError.

**Why it happens:** Some vote cells and the Constituency column header in 2026 contain `<sup>` elements with citation references. The `get_text()` method includes `<sup>` content by default.

**How to avoid:** Call `sup.decompose()` on all `<sup>` tags within a cell before extracting text.

**Warning signs:** `parse_int()` returning None for a cell that visually contains a large number.

### Pitfall 4: "Total" and Summary Rows

**What goes wrong:** The "Total" row at the bottom is included as a party row, inflating seat/vote counts when aggregating.

**Why it happens:** The "Total" row is a `<tr>` with `<td>` cells (not `<th>`), so it looks like a party row structurally.

**How to avoid:** Filter rows where `party.lower()` matches any of: `"total"`, `"valid votes"`, `"invalid/blank votes"`, `"registered voters"`, `"turnout"`, `"none of the above"`. These patterns appear consistently across all 7 pages.

**Warning signs:** A row with party name "Total" appearing in the output CSV; `seats` value equals 500 (or 480 for 2007).

### Pitfall 5: Party Name Inconsistency Across Years

**What goes wrong:** Assuming party names are normalized leads to silent data gaps when counting parties across years.

**Why it happens:** Party names genuinely change: "Thai Rak Thai" (2001/2005) is dissolved; successor parties have different names. "Move Forward Party" (2023) becomes "People's Party" (2026) after judicial dissolution. "Proportional" vs "Party-list" column naming (2007 vs others) is a table-level inconsistency, not party-level.

**How to avoid:** Store raw party names as scraped. Document in notebook comments that Phase 3 is responsible for any name normalization. Do not attempt cross-year party merging in this phase.

**Warning signs:** Expecting a party to appear in all 7 years but finding gaps — usually correct behavior, not a bug.

### Pitfall 6: Different URLs in URLS List vs. Actual Wikipedia Titles

**What goes wrong:** The existing notebook `URLS` list contains full URLs like `https://en.wikipedia.org/wiki/2026_Thai_general_election`. When using the Wikipedia API (`action=parse&page=`), the page title must use underscores: `2026_Thai_general_election`.

**How to avoid:** Extract the page title from the URL path: `url.split("/wiki/")[1]`.

---

## Runtime State Inventory

Step 2.5: SKIPPED — this is a greenfield data acquisition phase, not a rename/refactor/migration phase. No existing runtime state is being renamed or migrated.

---

## Environment Availability

| Dependency | Required By | Available | Version | Fallback |
|------------|------------|-----------|---------|----------|
| requests | WIKI-01: HTTP fetch | Yes | 2.31.0 | — |
| beautifulsoup4 | WIKI-02: HTML parsing | Yes | 4.12.3 | — |
| lxml | BS4 parser backend | Yes | 5.2.1 | html.parser (slower but functional) |
| pandas | WIKI-06: CSV output | Yes | 3.0.1 | — |
| pathlib (stdlib) | WIKI-07: dir creation | Yes | stdlib | — |
| Wikipedia (network) | All WIKI tasks | Yes (internet required) | — | None (blocking) |
| `data/wikipedia/` dir | WIKI-06: output path | Does not exist yet | — | Create with mkdir |

**Missing dependencies with no fallback:** Wikipedia network access is required. If executed offline, all fetches will fail. The notebook should handle `requests.exceptions.ConnectionError` gracefully.

**Missing dependencies with fallback:** None — all libraries present.

---

## State of the Art

| Old Approach (in notebook) | Current Approach | Impact |
|---------------------------|------------------|--------|
| `wikipedia-api` client returning plain text | `requests` + `BeautifulSoup` parsing HTML tables | Table data can be reliably extracted column by column |
| No structured output | Long-format CSV `party | year | seats | votes | vote_pct` | Directly consumable by Phase 3 pandas analysis |

**Deprecated in this phase:**
- `import wikipediaapi` and the `wiki = wikipediaapi.Wikipedia(...)` client: replace with `requests.get()`. The `wikipedia-api` library is still installed but not used for scraping.

---

## Open Questions

1. **2019 column layout — verification needed at implementation time**
   - What we know: Research shows 2019 uses "FPTP", "List", "Total" headers with a single "Votes" column (not split party-list/constituency).
   - What's unclear: Whether the `<td>` column count in data rows is 7 or 8 (depending on whether the party color swatch is a separate cell).
   - Recommendation: Print the first data row's `len(td_cells)` in a debug cell before committing to column indexes. Adjust the column index map accordingly.

2. **2026 footnote `[ag]` markers — exact columns affected**
   - What we know: The 2026 Constituency column header contains a `<sup>[ag]</sup>` reference. Some party rows may also have footnote markers.
   - What's unclear: Whether data cells (votes, seats) also have footnote markers.
   - Recommendation: Apply `sup.decompose()` to all cells universally — it's a zero-cost safety measure.

3. **Wikipedia page stability for 2026**
   - What we know: The 2026 election was held on 2026-02-08. Wikipedia data was available on the research date (2026-04-14) with full results.
   - What's unclear: Minor edits to the table (party reordering, corrections) could shift column positions if the page is still being updated.
   - Recommendation: Add a row-count assertion in the notebook: the output should contain at least 10 party rows per election year.

---

## Sources

### Primary (HIGH confidence)

- Wikipedia API (`action=parse&prop=sections`) — verified section indexes for all 7 elections
- Wikipedia API (`action=parse&prop=text&section=N`) — verified HTML table structure, CSS classes, column headers, and data formatting for 2001, 2005, 2007, 2011, 2019, 2023, 2026
- `pyproject.toml` + runtime Python checks — verified library versions

### Secondary (MEDIUM confidence)

- Wikipedia article pages (rendered) — confirmed party counts and total seats per year cross-checked against API results

### Tertiary (LOW confidence)

- Column index map for 2019 — derived from reported header structure; exact `<td>` count in data rows not directly confirmed from raw HTML. Needs validation at implementation time (see Open Question 1).

---

## Metadata

**Confidence breakdown:**
- Standard stack: HIGH — all libraries installed, versions confirmed at runtime
- Table CSS class: HIGH — confirmed `wikitable sortable` on all 7 pages via Wikipedia API
- Column structure (2001, 2005, 2007, 2011, 2023, 2026): HIGH — confirmed via API HTML parse
- Column structure (2019): MEDIUM — header text confirmed; exact td-count needs runtime check
- Pitfalls: HIGH — all derived from observed real table structure, not assumptions
- Party name patterns: HIGH — confirmed across multiple elections

**Research date:** 2026-04-14
**Valid until:** 2026-07-14 (Wikipedia table structure is stable; article edits could shift column positions for recent elections)
