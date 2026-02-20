# Known AEO Tool CSV Formats

This file maps column names from popular AEO (AI Engine Optimization) tools to the
standardized field names used by the analysis skill.

---

## Standardized Fields

The analysis requires these logical fields. Each tool maps its own column names to these:

| Standardized Field | Description | Required? |
|---|---|---|
| `date` | Date of the LLM response | Yes |
| `platform` | LLM platform (ChatGPT, Perplexity, Google AI Overviews, etc.) | Yes |
| `topic` | Topic/category the prompt belongs to | Yes |
| `prompt` | The actual prompt/query sent to the LLM | Yes |
| `mentions` | Raw comma-separated list of brands mentioned in the response | Yes |
| `normalized_mentions` | Cleaned/normalized brand list | Preferred (falls back to `mentions`) |
| `position` | Brand's ranking position in the response (e.g., "#2") | Yes |
| `mentioned_flag` | Whether the target brand was mentioned ("Yes"/"No") | Yes |
| `response` | Full text of the LLM response | Optional (needed for rebrand analysis) |
| `citation_*` | Citation URL columns (citation_1, citation_2, ...) | Optional (enables citation analysis) |
| `run_id` | Unique identifier for the tracking run | Optional |
| `tags` | Additional tags/labels | Optional |
| `region` | Geographic region | Optional |
| `persona` | User persona used for the prompt | Optional |
| `type` | Prompt type (open-ended, comparative, etc.) | Optional |
| `search_queries` | Related search queries | Optional |

---

## Profound

**Auto-detect columns:** `run_id`, `platformId`, `normalized_mentions`, `mentioned?`

| Profound Column | Standardized Field | Notes |
|---|---|---|
| `run_id` | `run_id` | UUID format |
| `date` | `date` | ISO format (YYYY-MM-DD) |
| `platformId` | _(internal)_ | Profound's internal platform UUID |
| `platform` | `platform` | Human-readable: "Google AI Overviews", "ChatGPT", "Perplexity" |
| `topic` | `topic` | Free text topic name |
| `tags` | `tags` | Comma-separated tags |
| `region` | `region` | e.g., "United States" |
| `persona` | `persona` | User persona |
| `type` | `type` | e.g., "open-ended" |
| `prompt` | `prompt` | Full prompt text |
| `mentions` | `mentions` | Raw brand list |
| `normalized_mentions` | `normalized_mentions` | Cleaned brand list, comma-separated |
| `position` | `position` | Format: "#1", "#2", etc. |
| `response` | `response` | Full LLM response text (may contain markdown) |
| `search_queries` | `search_queries` | Related search queries |
| `mentioned?` | `mentioned_flag` | "Yes" or "No" |
| `citation_1` through `citation_48` | `citation_*` | URL columns, up to 48 per row |

**Position format:** `#N` (e.g., `#1`, `#2`, `#3`)
**Mentioned flag values:** `Yes` / `No` (case-insensitive)
**Date format:** `YYYY-MM-DD`

---

## Scrunch

Scrunch has two distinct export formats depending on which report you download.

---

### Scrunch — Single-File Query Export

**Auto-detect columns:** `query_text`, `ai_platform`, `brand_mentioned`

_Note: Scrunch column names may vary by export version. The mappings below are based on known exports._

| Scrunch Column | Standardized Field | Notes |
|---|---|---|
| `date` or `response_date` | `date` | May use various date formats |
| `ai_platform` | `platform` | Platform name |
| `category` or `topic` | `topic` | Topic/category |
| `query_text` or `query` | `prompt` | The prompt sent |
| `brands_mentioned` | `mentions` | Comma-separated brand list |
| `brand_rank` or `position` | `position` | May be numeric (1, 2, 3) without "#" prefix |
| `brand_mentioned` | `mentioned_flag` | Boolean or Yes/No |
| `response_text` | `response` | Full response |
| `source_url_*` | `citation_*` | Citation URLs if available |

**Position format:** Numeric without `#` (e.g., `1`, `2`, `3`) — skill will normalize
**Mentioned flag values:** `true`/`false`, `1`/`0`, or `Yes`/`No`

---

### Scrunch — Two-File Performance Export

**Auto-detect:** Two CSV files present, one matching `report-prompts_performance-*.csv` and one matching `report-sources_performance-*.csv`.

This export format provides **pre-aggregated weekly metrics** rather than individual response rows. Each row in the prompts file represents a unique combination of prompt × platform × persona × geography × week.

#### Prompts Performance File (`report-prompts_performance-*.csv`)

| Column | Standardized Field | Notes |
|---|---|---|
| `prompt_id` | `run_id` | Unique prompt identifier |
| `prompt_text` | `prompt` | The prompt text (also used as topic proxy — no separate topic column) |
| `platform` | `platform` | Platform name (e.g., `chatgpt`, `perplexity`, `google_ai_overviews`, `google_ai_mode`) |
| `persona` | `persona` | User persona |
| `geo_country` | `region` | Geographic country |
| `week` | `date` | Week start date (YYYY-MM-DD) |
| `brand_presence_pct` | `mentioned_flag` | **Pre-aggregated**: % of that week's responses where brand was present (0–100%). Use as mention rate, not a per-row boolean. |
| `top_position_pct` | `position` (top bucket) | % of brand-present responses where brand appeared in top position |
| `middle_position_pct` | `position` (mid bucket) | % of brand-present responses where brand appeared in middle position |
| `bottom_position_pct` | `position` (bot bucket) | % of brand-present responses where brand appeared in bottom position |
| `positive_sentiment_pct` | _(extra)_ | % of brand-present responses with positive sentiment |
| `mixed_sentiment_pct` | _(extra)_ | % of brand-present responses with mixed sentiment |
| `negative_sentiment_pct` | _(extra)_ | % of brand-present responses with negative sentiment |
| `citation_pct` | _(extra)_ | % of responses that included citations |
| `brand_citation_pct` | _(extra)_ | % of responses where the brand's own domain was cited |

**Position format:** Three buckets (top/middle/bottom), not exact numeric ranks. Compute approximate position score: top≈1.0, middle≈2.5, bottom≈4.0.
**Mention rate:** `brand_presence_pct` is already a percentage (0–100). Average across rows for overall rate.
**No brand mention lists:** Individual competitor names are not available in this file — derive competitive landscape from the sources performance file.

#### Sources Performance File (`report-sources_performance-*.csv`)

| Column | Standardized Field | Notes |
|---|---|---|
| `Prompt` | `prompt` | The prompt text |
| `Platform` | `platform` | Platform name |
| `Domain` | _(domain)_ | Root domain of the cited URL |
| `Source URL` | `citation_*` | The specific cited URL |
| `Owner` | _(owner)_ | `Third-party` or brand name if owned |
| `Brand Presence` | _(brand flag)_ | `Yes` if target brand was mentioned in responses citing this URL; `No` if not |
| `Competitor Presence` | _(competitor)_ | Competitor brand name(s) associated with this URL — aggregate to build competitive landscape |
| `YYYY-MM-DD` columns | _(weekly citation rate)_ | One column per tracked week. Values are citation frequency rates (0.0–1.0): proportion of that week's responses that cited this URL. Sum across all weeks for total citation weight. |

**Citation weight:** Sum all weekly date columns per row to get total citation weight for that URL.
**Competitive landscape:** The `Competitor Presence` column contains competitor brand names — aggregate counts and citation weights from this column to replace Section 2.4.
**Brand affinity:** Domains with a high share of `Brand Presence = Yes` rows are disproportionately associated with brand mentions.

---

## Peec

Peec does **not** export conversation-level data (individual queries, responses, or per-platform breakdowns). Instead it provides two complementary exports focused on competitive benchmarking and citation gap analysis.

**Auto-detect:** Two CSV files present, one matching `visibility_export_*.csv` and one matching `source-domains-gap-analysis-*_export_*.csv`.

---

### Peec — Visibility Export (`visibility_export_from-YYYY-MM-DD_to-YYYY-MM-DD.csv`)

A wide-format table comparing daily visibility scores across all tracked brands. Each row is one brand; each column (after `brand`) is a date.

| Column | Notes |
|---|---|
| `brand` | Brand name (one row per tracked brand — the target brand is one of these rows) |
| `YYYY-MM-DD` columns | Daily visibility score as a percentage string (e.g., `"44.9%"`). One column per day in the export range. |

**What "visibility" means:** The percentage of tracked LLM queries in which that brand was mentioned on that day.

**Analysis approach:**
- Melt the wide format into long format (brand, date, visibility_pct) for time-series analysis
- The target brand is identified as the row where `brand` matches the brand name (case-insensitive, including alternates)
- All other rows are competitors — this file IS the competitive landscape
- Compute avg, min, max, and std dev of visibility per brand
- Rank brands by avg visibility — the target brand's rank is its competitive position
- Aggregate daily → weekly for trend analysis
- No per-platform, per-prompt, or position data is available from this file

---

### Peec — Source Domains Gap Analysis (`source-domains-gap-analysis-top-1000_export_[brand]_from-YYYY-MM-DD_to-YYYY-MM-DD.csv`)

A ranked list of domains that competitors are being cited from, prioritized by how much the target brand is missing out. The filename includes the target brand name.

| Column | Standardized Field | Notes |
|---|---|---|
| `Domain` | _(domain)_ | Root domain |
| `Type` | _(domain type)_ | `Corporate` (product/company site), `Competitor` (tracked competitor's own domain), `UGC` (user-generated content, e.g., Reddit), `Editorial` (news/media), `Reference` (review aggregators, directories), `Other` |
| `Used` | _(citation rate)_ | Percentage of responses that cited this domain (across all tracked queries) |
| `Gap Score` | _(priority score)_ | Peec's proprietary score quantifying the citation gap — higher = bigger opportunity. Not a raw count. |

**Analysis approach:**
- This is a citation **opportunity** file, not a raw citation log
- Sort by `Gap Score` descending for priority recommendations
- Group by `Type` to understand the category mix of gaps (e.g., mostly competitor domains vs. editorial vs. UGC)
- Domains with `Type = Competitor` are competitor-owned sites being heavily cited — useful for competitive intelligence
- Domains with `Type = Corporate` are third-party product sites — potential partnership or content placement targets
- Domains with `Type = Editorial` or `UGC` are earned/organic citation opportunities
- The target brand's own domain may appear in this list if Peec detects it as under-cited relative to competitors

---

## Adding New Formats

To add support for a new AEO tool:

1. Add a new section to this file with the tool name
2. List the **auto-detect columns** — 2-3 column names unique to that tool
3. Map each tool column to the standardized field
4. Document the position format and mentioned flag values
5. Note any special handling (date formats, delimited fields, etc.)
