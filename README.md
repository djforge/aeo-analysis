# AEO Analysis — Claude Code Slash Command

A reusable Claude Code custom slash command (`/aeo-analysis`) that runs comprehensive
AI Engine Optimization analysis on CSV exports from AEO tracking tools.

## What it does

Analyzes your brand's visibility, positioning, and competitive landscape across LLM
platforms (ChatGPT, Perplexity, Google AI Overviews, etc.) using data exported from
AEO tracking tools.

**Analysis includes:**

1. **Dataset Overview** — rows, date range, platforms, topics, prompts
2. **Brand Visibility** — mention rates overall, by platform, by topic
3. **Position Analysis** — rank distribution, avg/median by platform and topic
4. **Competitive Landscape** — top brands, co-mentions, brands when you're absent
5. **Topic Deep Dive** — best/worst topics by mention rate and position
6. **Platform Comparison** — head-to-head platform performance
7. **Weekly Trends** — mention rate and position week-over-week
8. **Citation Analysis** (if available) — domains cited, owned content presence, competitor citations, third-party sources
9. **Executive Summary** — markdown report with findings and strategic recommendations

> Some sections are not applicable for all export formats — see tool notes below.

## Supported AEO Tools

| Tool | Export Format | Auto-detected? |
|---|---|---|
| [Profound](https://www.profound.so/) | Single CSV with per-response rows | Yes — by `run_id`, `platformId`, `normalized_mentions`, `mentioned?` columns |
| [Scrunch](https://scrunch.ai/) | Single-file query export | Yes — by `query_text`, `ai_platform`, `brand_mentioned` columns |
| [Scrunch](https://scrunch.ai/) | Two-file performance export (`report-prompts_performance-*` + `report-sources_performance-*`) | Yes — by filename pattern |
| [Peec](https://peec.ai/) | Two-file export (`visibility_export_*` + `source-domains-gap-analysis-*_export_*`) | Yes — by filename pattern |
| Other tools | Any format | Manual column mapping via interactive prompts |

### Tool notes

**Profound** — Full analysis across all sections. Supports per-response citation columns (`citation_1` through `citation_48`).

**Scrunch (single-file)** — Full analysis. Citation columns supported if present in the export.

**Scrunch (two-file performance export)** — Pre-aggregated weekly data, not individual response rows. Brand visibility uses `brand_presence_pct`; position uses top/middle/bottom buckets rather than exact ranks. Competitive landscape is derived from the sources file. Includes a bonus sentiment analysis section. Drop both files in your working directory — the skill picks them up automatically.

**Peec** — Peec does not export per-query or per-platform data. The visibility file is a daily brand comparison table (all tracked brands vs. each other); the gap analysis file shows citation opportunities ranked by gap score. Position analysis, topic deep dive, and platform comparison sections are not applicable. Drop both files in your working directory — the skill picks them up automatically.

## Installation

### Option 1: Copy into your project

```bash
git clone https://github.com/djforge/aeo-analysis.git

cp -r aeo-analysis/aeo-analysis/ /path/to/your/project/.claude/commands/aeo-analysis/
```

### Option 2: Copy into global Claude config (available across all projects)

```bash
cp -r aeo-analysis/aeo-analysis/ ~/.claude/commands/aeo-analysis/
```

After copying, the `/aeo-analysis` command will be available in Claude Code.

## Usage

```
/aeo-analysis path/to/your-export.csv
```

If you omit the file path, the skill searches the current directory for CSV files. For
**two-file exports** (Scrunch performance or Peec), just place both files in the same
directory and run `/aeo-analysis` with no argument — the skill auto-detects the pair.

### What it asks you

1. **Brand name** — your brand plus any alternate names, former names, or common misspellings to treat as the same brand
2. **Owned domains** (for citation analysis) — e.g., `yourbrand.com, blog.yourbrand.com`
3. **Column mapping** — only if the CSV format isn't auto-detected

### Examples

```
# Single file
/aeo-analysis profound_export_feb_2026.csv

# Two-file export — just run from the directory containing both files
/aeo-analysis
```

Output:
- Full analysis printed to console
- `aeo_exec_summary.md` written to your working directory

## Requirements

- **Claude Code** (CLI)
- **Python 3** with **pandas** installed

```bash
pip install pandas
```

## File Structure

```
aeo-analysis/
├── SKILL.md                       # Main skill (the slash command logic)
└── references/
    ├── csv-formats.md             # Column mappings for Profound, Scrunch, Peec
    └── exec-summary-template.md   # Template for the executive summary output
```

## Example Output

### Console output (abbreviated, Profound format)

```
========================================================================
  SECTION 2.1 — DATASET OVERVIEW
========================================================================
Total rows:              9,323
Date range:              2026-01-15 to 2026-02-11
Platforms tracked:       3
Unique prompts:          85
Topics tracked:          12

  Platforms:
    Google AI Overviews     3,340 rows  (35.8%)
    ChatGPT                 3,154 rows  (33.8%)
    Perplexity              2,829 rows  (30.3%)

========================================================================
  SECTION 2.2 — BRAND VISIBILITY
========================================================================
Overall mention rate:    4,661 / 9,323 = 50.0%

  By Platform:
    Google AI Overviews     1,804 / 3,340  = 54.0%
    ChatGPT                 1,621 / 3,154  = 51.4%
    Perplexity              1,236 / 2,829  = 43.7%
```

### Executive summary (`aeo_exec_summary.md`)

A complete markdown report including TL;DR, key findings across all analysis areas,
competitive context, citation analysis (when available), and strategic recommendations
organized into immediate, medium-term, and strategic actions.

## Contributing

1. Fork the repo
2. Add support for new AEO tools in `references/csv-formats.md`
3. Submit a PR

## License

MIT — see [LICENSE](LICENSE).
