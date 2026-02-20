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
9. **Executive Summary** — markdown report with findings and recommendations

## Supported AEO Tools

| Tool | Status |
|---|---|
| [Profound](https://www.profound.so/) | Fully supported (auto-detected) |
| [Scrunch](https://scrunch.ai/) | Column mapping supported |
| [Peec](https://peec.ai/) | Column mapping supported |
| Other tools | Manual column mapping via interactive prompts |

The skill auto-detects known CSV formats by matching column headers. For unrecognized
formats, it walks you through mapping your columns to the required fields.

## Installation

### Option 1: Copy into your project (recommended)

```bash
# Clone or download this repo
git clone https://github.com/YOUR_ORG/aeo-analysis-skill.git

# Copy the skill directory into your project's Claude commands
cp -r aeo-analysis-skill/aeo-analysis/ /path/to/your/project/.claude/commands/aeo-analysis/
```

### Option 2: Copy into global Claude config

```bash
# For global availability across all projects
cp -r aeo-analysis-skill/aeo-analysis/ ~/.claude/commands/aeo-analysis/
```

After copying, the `/aeo-analysis` command will be available in Claude Code.

## Usage

```
/aeo-analysis path/to/your-export.csv
```

If you omit the file path, the skill will search the current directory for CSV files.

### What it asks you

1. **Brand name** — your brand + any alternate names / former names
2. **Owned domains** (for citation analysis) — your website domains
3. **Column mapping** (only if the CSV format isn't auto-detected)

### Example

```
/aeo-analysis Feb_12_4_weeks_profound_raw_data_with_citations.csv
```

Output:
- Full analysis printed to console (7 sections + citation analysis)
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

### Console output (abbreviated)

```
================================================================================
  1. DATASET OVERVIEW
================================================================================
Total rows:              9,323
Date range:              2026-01-15 to 2026-02-11
Platforms tracked:       3
Unique prompts:          85
Topics tracked:          12

--- Rows per Platform ---
  Google AI Overviews                 3,340 rows  (35.8%)
  ChatGPT                            3,154 rows  (33.8%)
  Perplexity                         2,829 rows  (30.3%)

================================================================================
  2. BRAND VISIBILITY
================================================================================
Overall mention rate:    4,661 / 9,323 = 50.0%

--- Mention Rate by Platform ---
  Google AI Overviews                 1,804 / 3,340  = 54.0%
  ChatGPT                            1,621 / 3,154  = 51.4%
  Perplexity                         1,236 / 2,829  = 43.7%
  ...
```

### Executive summary (`aeo_exec_summary.md`)

A complete markdown report including TL;DR, key findings across all analysis areas,
competitive context, citation analysis (when available), and strategic recommendations.

## Contributing

1. Fork the repo
2. Add support for new AEO tools in `references/csv-formats.md`
3. Submit a PR

## License

Apache-2.0 — see [LICENSE](LICENSE).
