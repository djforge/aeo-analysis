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

**Auto-detect columns:** `llm_engine`, `query`, `visibility_score`

_Note: Peec exports may vary. These mappings are based on known export structures._

| Peec Column | Standardized Field | Notes |
|---|---|---|
| `timestamp` or `date` | `date` | May include time component |
| `llm_engine` | `platform` | LLM engine name |
| `category` | `topic` | Category/topic |
| `query` | `prompt` | Prompt text |
| `mentioned_brands` | `mentions` | Comma-separated |
| `rank` or `brand_position` | `position` | Numeric position |
| `is_mentioned` or `mentioned` | `mentioned_flag` | Boolean |
| `full_response` or `answer` | `response` | Response text |
| `visibility_score` | _(extra)_ | Peec-specific visibility metric |
| `references` or `sources` | `citation_*` | May be a single column with delimited URLs |

**Position format:** Numeric (e.g., `1`, `2`, `3`)
**Mentioned flag values:** `true`/`false`, `1`/`0`, `Yes`/`No`

---

## Adding New Formats

To add support for a new AEO tool:

1. Add a new section to this file with the tool name
2. List the **auto-detect columns** — 2-3 column names unique to that tool
3. Map each tool column to the standardized field
4. Document the position format and mentioned flag values
5. Note any special handling (date formats, delimited fields, etc.)
