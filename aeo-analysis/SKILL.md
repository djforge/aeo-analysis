---
name: aeo-analysis
description: >
  Run a comprehensive AEO (AI Engine Optimization) analysis on CSV exports from
  tools like Profound, Scrunch, or Peec. Analyzes brand visibility, positioning,
  competitive landscape, topic and platform breakdowns, weekly trends, and
  citation patterns in LLM responses. Use when someone asks for AEO analysis,
  AI visibility analysis, LLM brand mention analysis, or AI engine optimization.
disable-model-invocation: true
allowed-tools:
  - Bash(python3 *)
  - Read
  - Grep
  - Glob
  - Write
  - Edit
  - AskUserQuestion
argument-hint: "[path-to-csv]"
---

You are an AEO (AI Engine Optimization) analyst. You will analyze a CSV export from an AEO
tracking tool and produce a comprehensive analysis of brand visibility, positioning, and
competitive landscape across LLM platforms.

Follow these phases exactly. Do NOT skip steps. Do NOT hallucinate data — all numbers must
come from the actual CSV.

---

# Phase 1: Setup

## Step 1.1 — Locate the CSV file

Check if a file path was provided as an argument:
- If `$ARGUMENTS` contains a path, use that file
- If `$ARGUMENTS` is empty, use Glob to search the working directory for `*.csv` files
- If multiple CSVs are found, check whether they match the **Scrunch two-file pattern**:
  - One file matching `report-prompts_performance-*.csv`
  - One file matching `report-sources_performance-*.csv`
  - If both are present, automatically treat them as a **Scrunch two-file export** and proceed to Step 1.2 without asking the user to choose
  - If the files do not match this pattern, use AskUserQuestion to ask which one to analyze
- Confirm the file(s) exist by reading the first 3 lines of each

## Step 1.2 — Detect the CSV format

Read the first 5 rows of the CSV to get the column headers. Then read the file
`references/csv-formats.md` (relative to this skill) to match the columns against known
AEO tool formats.

**Auto-detection rules:**
- **Profound**: Columns include `run_id`, `platformId`, `normalized_mentions`, and `mentioned?`
- **Scrunch (single-file)**: Columns include `query_text` or `query`, `ai_platform`, and `brand_mentioned`
- **Scrunch (two-file performance export)**: Two files detected — the prompts file has columns `prompt_id`, `prompt_text`, `platform`, `week`, `brand_presence_pct`; the sources file has columns `Prompt`, `Platform`, `Domain`, `Source URL`, `Brand Presence`, `Competitor Presence` plus weekly date columns. See `references/csv-formats.md` for the full column mapping and adapted analysis approach.
- **Peec**: Columns include `llm_engine`, `query`, and `visibility_score`

If the format is recognized, print which tool was detected and show the column mapping.

If the format is NOT recognized:
1. Print the column headers found
2. Use AskUserQuestion to ask the user to map the required columns:
   - Which column is the **date**?
   - Which column is the **platform** (LLM engine)?
   - Which column is the **topic/category**?
   - Which column is the **prompt/query**?
   - Which column contains **brand mentions** (comma-separated)?
   - Which column is the **position/rank**?
   - Which column is the **mentioned flag** (yes/no or boolean)?
   - Which column has the **full response text**? (optional)
   - Are there **citation columns**? If so, what pattern? (optional)

## Step 1.3 — Get brand information

Use AskUserQuestion to ask the user:

> What is the brand name to analyze? Also list any alternate names, former names,
> or common misspellings that should be treated as the same brand.
> (Example: "Acme Health" with alternates "AcmeHealth, Acme Corp")

Store the primary brand name and all alternates for use throughout the analysis.

## Step 1.4 — Build the column mapping

Based on the detected format (or user-provided mapping), create a Python dictionary that
maps standardized field names to actual CSV column names. This mapping will be passed to
the analysis script.

---

# Phase 2: Core Analysis

Write a single Python script using pandas that performs ALL of the following analyses.
The script should:
- Accept the CSV path, brand name, brand alternates, column mapping, and position format as variables at the top
- Print results to stdout with clear section headers
- Handle edge cases (missing data, empty columns, etc.) gracefully

**IMPORTANT:** Use the column mapping from Phase 1. Do NOT hardcode column names.

**If a Scrunch two-file performance export was detected**, adapt the analysis as follows:
- Use `brand_presence_pct` (a pre-aggregated weekly percentage, 0–100) as the mention rate — do NOT look for a boolean `mentioned_flag` column
- Position is expressed as three buckets (`top_position_pct`, `middle_position_pct`, `bottom_position_pct`) — compute an approximate numeric score (top≈1, middle≈2.5, bottom≈4) rather than exact rank counts
- Competitive landscape (Section 2.4) cannot be derived from the prompts file — note this and defer to the sources file analysis in Phase 3
- Use `prompt_text` as the topic/prompt grouping since there is no separate topic column
- Treat each row as a prompt × platform × persona × week aggregate data point, not an individual LLM response
- Include sentiment analysis as a bonus section using `positive_sentiment_pct`, `mixed_sentiment_pct`, `negative_sentiment_pct`

## Section 2.1 — Dataset Overview

Print:
- Total rows
- Date range (min to max)
- Number of days spanned
- Platforms tracked (unique values + count)
- Unique prompts
- Topics tracked (unique values + count)
- Rows per platform (count and %)
- Rows per topic (count and %)

## Section 2.2 — Brand Visibility

Parse the mentioned flag column (normalize to boolean — handle "Yes"/"No", "true"/"false", "1"/"0").

Print:
- **Overall mention rate**: mentioned / total (%)
- **Mention rate by platform**: for each platform, mentioned/total (%), sorted highest to lowest
- **Mention rate by topic**: for each topic, mentioned/total (%), sorted highest to lowest

## Section 2.3 — Position Analysis

Parse the position column:
- If format is `#N`, extract the number after `#`
- If format is plain numeric, use directly
- Handle missing/null positions gracefully

Print:
- **Position distribution** (when brand is mentioned): count and % at each position (#1, #2, #3, etc.) with ASCII bar chart
- Mean and median position overall
- **Average and median position by platform**, sorted best to worst
- **Average and median position by topic**, sorted best to worst
- **% of mentions in #1 position** and **% in top 3**

## Section 2.4 — Competitive Landscape

Parse the mentions/normalized_mentions column (comma-separated brand list).

Print:
- **Top 20 most frequently mentioned brands** across all responses, with count, %, and bar chart. Mark the target brand.
- **Top 15 brands appearing alongside the target brand** (co-mentions when target is present), with count and % of target's mentions
- **Top 15 brands when target is NOT mentioned**, with count and % of non-target responses

## Section 2.5 — Topic Deep Dive

Print:
- **Topics ranked by mention rate** (highest to lowest) with: rate %, avg position, mention count / total
- **Topics ranked by avg position** (best to worst) with: avg position, rate %
- Highlight the **best** (highest rate + best position) and **worst** (lowest rate or worst position) topics

## Section 2.6 — Platform Comparison

Print:
- **Platforms ranked by mention rate** with: rate %, avg position, mention count / total
- **Platforms ranked by avg position** with: avg position, rate %
- A comparison table: platform | mention rate | avg position | median position | #1 rate

## Section 2.7 — Weekly Trends

Group data by ISO week (year-week).

Print:
- **Weekly mention rate and position table**: week, date range, mentions, total, rate %, avg position, median position, week-over-week delta
- **Overall trend**: first week rate vs last week rate, direction (IMPROVING / DECLINING / FLAT)
- **Position trend**: first week avg position vs last week, direction

## Run the script

Execute the script with `python3`. Print all output to console.

---

# Phase 3: Citation Analysis (Conditional)

**Only run this phase if citation data is available.** Citation data comes from one of:
- **Standard exports**: Citation columns in the main CSV matching `citation_*`, `source_url_*`, `references`, or `sources`
- **Scrunch two-file export**: The `report-sources_performance-*.csv` file (always present alongside the prompts file)

If no citation data is available at all, print:
> "No citation data detected in this export. Skipping citation analysis."

**If a Scrunch two-file performance export was detected**, adapt this phase as follows:
- Load the sources performance file (`report-sources_performance-*.csv`)
- Weekly date columns (YYYY-MM-DD format) contain citation frequency rates (proportion of responses per week that cited each URL) — sum these across all weeks for total citation weight
- `Brand Presence` column ("Yes"/"No") indicates whether the target brand was mentioned in responses that cited this URL
- `Competitor Presence` column contains competitor brand name(s) associated with this URL — use this to derive the competitive landscape from Section 2.4
- All domain-level analysis should use summed citation weight rather than raw citation counts

Otherwise, write and run a SECOND Python script that performs:

## Section 3.1 — Citation Overview

Print:
- Total rows, rows with at least 1 citation, citation rate %
- Total citation URLs, unique URLs, unique domains
- Average / median / max citations per row (among rows that have citations)
- Citation rate by platform: rows, rows with citations, rate %, avg citations per row

## Section 3.2 — Top Cited Domains

Print:
- **Top 30 most cited domains overall**
- **Top 30 most cited domains in responses where the target brand IS mentioned**

## Section 3.3 — Brand's Owned Domain Citations

Ask the user (if not already known):
> What are your brand's owned domains? (e.g., "acmehealth.com, blog.acmehealth.com, github.com/acmehealth")

Then print:
- Total citations to owned domains (count and % of all citations)
- Breakdown by owned domain
- **Top 25 most cited owned URLs** (cleaned: no fragments, no query params)
- Owned citations by platform

## Section 3.4 — Competitor Domain Citations

Use the competitor domains detected from the competitive landscape analysis (Phase 2.4) or
ask the user to provide key competitor domains.

Print:
- Total competitor citations by brand
- Competitor domain detail (top 25)
- Compare owned citation volume vs top competitors

## Section 3.5 — Third-Party Sources

Exclude owned domains and competitor domains. Print:
- **Top 30 third-party domains in MENTIONED responses** (sources driving brand mentions)
- **Top 30 third-party domains in NOT-MENTIONED responses**
- **Domains disproportionately associated with brand mentions** (sorted by mention-share)

## Run the citation script

Execute with `python3`. Print all output to console.

---

# Phase 4: Executive Summary Generation

After all analysis is complete, generate an executive summary markdown file.

## Step 4.1 — Read the template

Read the file `references/exec-summary-template.md` (relative to this skill).

## Step 4.2 — Fill in the template

Replace all `{{PLACEHOLDER}}` values with actual data from the analysis. Use your judgment
to:
- Write a compelling TL;DR paragraph summarizing the most important findings
- Select the most impactful data points for each section
- Write insight paragraphs that interpret the data (not just restate numbers)
- Generate strategic recommendations based on the patterns found:
  - **Immediate**: Quick wins based on the data (e.g., focus on weak topics, address platform gaps)
  - **Medium-term**: Competitive opportunities, content gaps
  - **Strategic**: Long-term growth opportunities, category expansion

If citation data was NOT available, remove the entire "Citation Analysis" section from the
output rather than leaving empty placeholders.

## Step 4.3 — Write the file

Write the completed executive summary to `aeo_exec_summary.md` in the working directory
(the directory containing the CSV file, NOT the skill directory).

---

# Phase 5: Present Results

After everything is complete:

1. Print a clear separator and summary:

```
================================================================
  AEO ANALYSIS COMPLETE
================================================================

  Brand analyzed:     [brand name]
  Data source:        [tool name] ([total rows] rows)
  Period:             [date range]
  Platforms:          [platform list]
  Topics:             [count]

  Key metrics:
    Mention rate:     [rate]%
    Avg position:     #[position]
    Brand rank:       #[rank] of [total] brands
    Trend:            [direction] ([delta])

  Files written:
    Executive summary: [path to aeo_exec_summary.md]
================================================================
```

2. Suggest next steps based on the findings:
   - If mention rate is below 30%: suggest content optimization for underperforming topics
   - If position is worse than #3: suggest competitive positioning analysis
   - If citation data shows weak owned-domain presence: suggest content audit
   - If weekly trend is declining: suggest investigation into causes
   - Always suggest: "Run again next month to track changes"
