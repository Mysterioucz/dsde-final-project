# Phase 4: Wikipedia Scraping - Discussion Log

**Session:** 2026-04-14
**Mode:** Interactive (discuss)

---

## Gray Areas Presented

User selected all four areas for discussion:
1. Data fields to extract
2. Scraping method
3. Elections scope
4. Output format & storage

---

## Area: Data Fields

**Q: What should be extracted from each election's Results section?**
Options: Seats only / Seats + vote counts / Seats + votes + %
→ **Selected: Seats + votes + %** (full set: seats, total votes, vote share percentage)

**Q: Which seat types to capture?**
Options: Total seats only / Constituency + Party-list split
→ **Selected: Total seats only** (consistent across all election years; constituency/party-list split varies by table format)

---

## Area: Scraping Method

**Q: How should we fetch and parse the Wikipedia election results tables?**
Options: requests + BeautifulSoup / wikipedia-api + manual parsing / Hybrid
→ **Selected: requests + BeautifulSoup** (wikipedia-api only gives plain text, unsuitable for table extraction)

**Q: How should requests to Wikipedia be handled?**
Options: Simple delay between requests / No delay needed
→ **Selected: Simple delay (~1s)** (polite crawling; Wikipedia guidelines recommend it)

---

## Area: Elections Scope

**Q: Which election years should be scraped?**
Options: All 7 from notebook / Exclude 2026 / Custom range
→ **Selected: All 7** (2001, 2005, 2007, 2011, 2019, 2023, 2026)

**Q: Elections in 2008 and 2014 were held under special circumstances. How to handle?**
Options: Skip — only use notebook URLs / Include if data available
→ **Selected: Skip** (keep to the 7 established regular-election URLs in the notebook)

---

## Area: Output Format & Storage

**Q: How should the scraped data be structured in the output file?**
Options: Long format / Wide format
→ **Selected: Long format** (one row per party × year; columns: party | year | seats | votes | vote_pct)

**Q: Where should the output CSV be saved?**
Options: data/previous-election/ / data/wikipedia/
→ **Selected: data/wikipedia/** (new dedicated directory for Wikipedia-sourced data)

---

## Final Status

User indicated ready for context after all four areas were discussed.
