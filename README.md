# Pro Patent Search Skill — v3.1

A Claude Code skill that turns an AI agent into a **Virtual Patent Engineer**: multi-round keyword search, automated deduplication and scoring, FTO analysis, claim charting, and citation snowballing.

## Features

- **Multi-round search** — `patent_search_runner.py` auto-generates up to 8 diversified queries via synonym expansion, deduplicates across rounds, and scores by country weight + year + family size
- **Audit trail** — every run produces a `*_Search_Process_Report.md` with full query strings, per-round counts, dedup details, and per-patent scores
- **Tor auto-rotation** — requests a new exit node via NEWNYM after 2 consecutive 503 responses; no manual intervention needed
- **Abstract enrichment** — batch-fetches full abstracts and detailed IPC codes from Google Patents (no count limit)
- **IPC-first classification** — tech/effect dimensions derived from IPC/CPC codes before falling back to keywords
- **langdetect** — non-English abstracts (JP/KR/CN/RU) excluded from keyword matching to avoid garbled-text misclassification
- **Blue Ocean detection** — zero-count cells in the tech-effect matrix highlighted as technology white spaces

## Installation

```powershell
# Windows (PowerShell) — run once
.\install.ps1
```

The script installs all Python dependencies, sets up Playwright Chromium (browser fallback), and configures Tor with `CookieAuthentication 1`.

**Manual dependency install:**
```bash
pip install -r requirements.txt
playwright install chromium
```

Set `ANTHROPIC_API_KEY` for LLM-powered synonym expansion and claim analysis.

## Quick Start

```powershell
# Start Tor proxy
python scripts/proxy_manager.py --start
python scripts/proxy_manager.py --check   # verify exit IP

# One-command search + audit report
python patent_search_runner.py --topic "nebulizer" --outdir "D:\patent\run1"
```

**Advanced options:**
```powershell
# Custom round count and final list size
python patent_search_runner.py --topic "blood pressure monitor" --max 50 --final 60 --outdir "D:\patent\bp"

# Disable Tor (testing)
python patent_search_runner.py --topic "nebulizer" --no-tor --outdir "D:\patent\test"
```

## Scripts

| Script | Purpose |
|--------|---------|
| `scripts/patent_search_runner.py` | Main orchestrator: synonym expansion → multi-round search → dedup → score → report |
| `scripts/google_patents_collector.py` | Google Patents XHR API search (keyword / IPC / assignee) + Tor auto-rotation |
| `scripts/advanced/abstract_enricher.py` | Batch-enrich patents with abstracts and IPC codes |
| `scripts/advanced/ipc_classifier.py` | IPC-prefix-first tech/effect classifier with langdetect |
| `scripts/advanced/lang_utils.py` | Language detection utility (`is_english`, `build_classification_text`) |
| `scripts/advanced/visualizer.py` | Generate 4 charts: assignee, trend, country, tech-effect matrix |
| `scripts/advanced/citation_crawler.py` | Citation snowball crawling |
| `scripts/advanced/claim_chart_gen.py` | Element-by-element claim chart vs. product description |
| `scripts/advanced/legal_status_calculator.py` | Patent expiry date and legal status calculation |
| `scripts/advanced/browser_renderer.py` | Playwright fallback renderer (when API is blocked) |

## Tor Setup

Add to your `torrc`:
```
SocksPort 9050
ControlPort 9051
CookieAuthentication 1
```

`install.ps1` writes this automatically.

## Downstream: Patent Mapping

The output CSV from this skill feeds directly into the [patent-mapping](https://github.com/jack-lee2022/patent-mapping) skill for tech-effect matrix generation and Blue Ocean analysis.

## Skill SOP

See `SKILL.md` for the full professional workflow: Tor setup → one-command search → FTO analysis → invalidity search → patent landscape.
