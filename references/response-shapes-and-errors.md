# Response shapes and error taxonomy (§4–§5)

Part of the **sistrix-mcp** skill (CC BY 4.0 — Eoghan Henn / [rebelytics.com](https://rebelytics.com)). Section numbers are global across `SKILL.md` and `references/` — the section map in `SKILL.md` says where each § lives.

**Load trigger:** Read before scripting or building any artifact against a SISTRIX response, and whenever a call returns an error, a timeout or an empty result — before deciding what the symptom means.

## Contents

- 4. Response shape — assume nothing
- 5. Error taxonomy & resilience

---

## 4. Response shape — assume nothing

SISTRIX responses are not uniformly enveloped. Inspect the actual structure
before parsing. Observed top-level keys include: `result`, `prompts` (for
`ai_top`), `source` (for `ai_tracker` prompts *and* sources views), `competitor`,
`counts`, `environment`, German keys (`sichtbarkeitsindex`, `kwcount.seo`,
`kwcount.seo.top10`), nested keys (`optimizer.rankings` → `optimizer.ranking`),
and dotted keys (`url.from`, `url.to`). Some tools return a bare array of
strings (`domain view=competitors`, `domain view=ideas`, `links view=linktexts`). And
**single-row results can unwrap**: when `keyword_domain_seo` matches exactly
one row, it returns a **bare object instead of a one-element array** — scripts
that iterate over the response break on single-row results, so handle
object-or-array. When building an artifact or script around a tool, call it
once and read the real shape first.

---

## 5. Error taxonomy & resilience

| Symptom | Meaning | What to do |
|---|---|---|
| `Timeout: The request took too long to complete.` | Transient slowness (seen on first-hit visindex for some domains) | **Retry once** before treating as failure. Split big parallel batches into smaller groups. |
| `SISTRIX API Error (1000): no result` | Valid request, no data for that view (e.g. onpage with no crawl, or a sub-dataset like `project_onpage view=cookies` that a *completed* crawl didn't capture) | Not a bug — verify the object actually has that data; report "no data" to the user. |
| `SISTRIX API Error (4000): project not found` | Bad/unknown project hash | Re-list with `project view=overview` / `ai_tracker_overview` and confirm the hash. |
| `SISTRIX API Error (1007): invalid SERP-Feature` | Bad `serp_feature` value (server-side; does NOT list valid values) | Use the uppercase tokens from `keyword view=serpfeatures` (ORGANIC, PRODUCTS, …); look them up first. |
| `Error (1000): no result` on `ai_prompt_answers` for a tracker prompt | That tool only serves SISTRIX's global prompt pool, not custom tracker prompts | Expected — use `ai_tracker` views instead; per-prompt answer text isn't retrievable for tracker prompts. |
| `Error (1000): no result` on `keyword view=seo` with a `domain` filter | Ambiguous: either the domain isn't in the SERP, OR the keyword has no SERP data at all | Run an undated, unfiltered control call first to prove the data source has content before concluding "doesn't rank". |
| `Error (1000): no result` on a `keyword` volume/traffic call for a term with evident demand | The keyword isn't in that country's database yet — emerging vocabulary, not absent demand | Control with `show_all_countries=true` and with a sibling term of known demand in the same country. Report "not covered", never "no volume"; take the demand read from a market-observing source. (See the `keyword` family note.) |
| `ai_topicresearch overview` returns **0 topics** for an emerging entity or category | Same coverage lag in the topic dataset — the category isn't clustered yet, not that the market has no topics | Check a sibling or parent term the database does cover, and the term in another country. Report "not covered yet", never "no topics exist". (See the `keyword` family note.) |
| `SISTRIX API Error (3001): no keyword history` on `keyword view=seo` (`date` + `domain`) | Per-keyword SERP history isn't stored for low/zero-volume keywords outside a higher-volume set | Use the domain-centric route instead: `keyword_domain_seo(date, search="<exact keyword>")` returns the historical positions (single-row bare object). |
| `SISTRIX API Error (1755): link data is currently being updated` | **Transient** — link index is mid-refresh | Retry later; the same call succeeds once the update finishes. Distinct from 1733 below. |
| `SISTRIX API Error (1733): linkplus needs to be activated for this domain` | Account/module state — LinkPlus isn't active for that domain | **User action required** (activate LinkPlus); not a retry. The same domain can surface 1755 first and 1733 on retry — read the code, don't blind-retry. |
| Empty output (no error) | Some tools (e.g. `project_onpage view=overview`) return nothing rather than a 1000 error when there's no data | Treat as "no data", same as 1000. |
| Pydantic validation error listing valid keys | Bad enum value (e.g. `issue`, `view`) | Read the returned list and retry with a valid key. |
| Silent wrong data | Invalid `country` accepted without error | Prevent it: validate `country` client-side up front. |

**Credits.** SISTRIX API calls consume account credits and there's no MCP tool
that reports remaining quota. For broad/exploratory work, batch deliberately and
ask the user about a credit budget before running dozens of calls.

---

