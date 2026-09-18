---
name: sistrix-mcp
description: "Companion skill for the SISTRIX MCP server (SEO + AI-visibility data via the SISTRIX API). Load this whenever the user works with SISTRIX through an MCP — pulling Visibility Index, keyword/ranking data, backlinks, competitor sets, domain traffic estimates, or AI Overviews / AI Mode / ChatGPT / Perplexity mentions, sources and trackers. Also load for SISTRIX Optimizer project work (project rankings, onpage crawls) and any cross-market comparison or report built on SISTRIX data. Trigger on mentions of Sichtbarkeitsindex, ai_check/ai_tracker, the domain/keyword/links/project tools and their view parameters, or older ai_entity/ai_top/domain_/keyword_ names — even when the tools aren't named but SISTRIX data is clearly needed. Teaches the real behaviour of the MCP, including response-shape quirks and the gotchas tool descriptions omit."
version: 1.3.0
license: CC-BY-4.0
origin: https://github.com/rebelytics/sistrix-mcp
maintainer: Eoghan Henn / rebelytics (eoghan@rebelytics.com)
---

# SISTRIX MCP Companion

**Created by Eoghan Henn / [rebelytics.com](https://rebelytics.com)**

A field guide to the SISTRIX MCP server. It documents how the tools actually
behave — parameter rules, inconsistent response shapes, error patterns, and
token-cost traps — so a session using the MCP is structured and avoids the
common mistakes. It does not replace the MCP's own tool descriptions; it adds
the operational knowledge those descriptions omit.

**Licence:** Released under [CC BY 4.0](https://creativecommons.org/licenses/by/4.0/).
You may share and adapt it for any purpose with appropriate credit.

**Feedback & Support:** If the methodology here proves wrong or incomplete, or
you hit SISTRIX MCP behaviour this skill doesn't cover, log it and open an issue
on the skill's [public repository](https://github.com/rebelytics/sistrix-mcp), or contact
Eoghan Henn via [rebelytics.com](https://rebelytics.com). Real-world breakage is the best signal
for improving the guide. If a failure is the agent not following this skill,
acknowledge and correct it rather than blaming the methodology.

> **Tool names, and how to match them.** Names below are written without the
> MCP server prefix (e.g. `domain`); your environment prefixes them, so match on
> the part after the prefix. **The server exposes a small set of tools that each
> take a `view` parameter, not one tool per dataset** — so this file names the
> **dataset (tool + view)**, e.g. `domain view=visindex`, and matching a name
> like `domain_visindex` against the schema will find nothing. Earlier versions
> of this skill, and any notes or scripts written against them, use the
> one-tool-per-dataset names; resolve those through the **name-mapping table**
> at the top of the Tool-family reference. Before a first call in a new session,
> read the `view` enum the server actually announces — it is the authoritative
> list, and this file is a description of it that can drift.

**Coverage boundary.** This skill covers **the five MCP tool families** —
`domain`, `keyword`, `links`, `ai_*`, `project` — their parameters,
response shapes, error codes and the analyses you can build from them. It does
**not** cover the SISTRIX Toolbox UI and its reports, account or credit
administration, or any Toolbox module with no MCP tool behind it. Silence in
this file is not evidence that SISTRIX lacks a capability: a companion skill is
loaded precisely because the agent doesn't know the tool, so an uncovered
surface reads as a missing feature when it may only be an uncovered surface.
Check the tool schemas the server announces before concluding something isn't
there.

---

## Mental model

The MCP exposes five tool families. Three answer questions about **any public
domain or keyword** (no account setup needed); two read **your own SISTRIX
account objects** (Optimizer projects and AI trackers) and need an ID first.

| Family | Tool(s) | Needs | Answers |
|---|---|---|---|
| Domain | `domain` (10 views) | a domain + country | Visibility Index, keyword counts, URL counts, ranking distribution, traffic estimate, competitors, ideas, opportunities |
| Keyword | `keyword` (7 views), `keyword_domain_seo` | a keyword + country (or a domain) | SERP, search volume/clicks, CPC, competition, intent, SERP features; every keyword a domain ranks for |
| Links | `links` (3 views) | a domain (no country) | backlink targets, anchor texts, individual links |
| AI visibility | `ai_check`, `ai_tracker`, `ai_tracker_overview`, `ai_prompt_answers`, `ai_topicresearch` | brands/domains, a prompt, a keyword, or a tracker ID | brand/source mentions in AI answers, prompt-level visibility, competitors, AI topic research |
| Project | `project` (5 views), `project_onpage` (9 views) | an Optimizer project hash | tracked rankings, project Visibility Index, project competitors, onpage crawl |

Within a family the tool is one endpoint and the **`view` parameter selects the
dataset**. Two consequences worth holding on to: view-specific parameters are
**silently ignored** when the chosen view doesn't use them (so a call can look
filtered and not be), and adding a dataset to the server adds a `view` value
rather than a tool — which is why this file can fall behind without any call
failing. Read the announced `view` enum, not a remembered list.

**Discovery first.** Project and tracker IDs are opaque hashes. Don't guess or
hardcode them — list them:

- `project view=overview` → all Optimizer projects with `hash` + `name`.
- `ai_tracker_overview` → all AI trackers with `hash` + `name`.

Confirm the hash a user hands you against these lists before using it; the same
hash space is *not* shared (an AI-tracker hash is not an Optimizer project hash,
and vice versa — they're used by different tool families).

---

## Universal parameter rules

- **`country`** — lowercase ISO 3166-1 alpha-2, from the fixed supported set the
  server instructions already list (default `de`). The skill-relevant gotcha,
  not in the schema: **a wrong/unsupported code is silently accepted — it does
  not error, it returns data anyway**, so a typo'd country yields
  confidently-wrong results. Validate the code client-side before calling;
  never rely on an error to catch it. SISTRIX uses **`uk`, not `gb`** (a `gb`
  request is silently mis-served).
- **AI tools and country** — `ai_*` accept the same codes (upper- or lowercase),
  or omit `country` entirely to aggregate across all countries.
- **Pagination is inconsistent.** Some tools use `limit` + `offset`
  (domain view=traffic_estimation, keyword_domain_seo, domain view=opportunities,
  links view=list, project view=ranking, project_onpage view=urls/issue). Others use `limit` +
  `page` (the `ai_*` family, project view=competitors, project_onpage view=links/
  resources/cookies). Check the schema rather than assuming.
- **`api_key`** — leave unset; the server is configured with one. Only pass it
  if the user explicitly supplies a different key.

---

## Tool-family reference

### Name-mapping table (legacy name → live tool + view)

The server consolidated a one-tool-per-dataset surface into view-parameter
tools. Older notes, scripts, deliverables and earlier versions of this skill
use the left-hand names; nothing on the left exists on the server any more.
**Check this table against the announced schema in one glance at the start of
any session that reuses old material** — it is the part of this file most
likely to be stale, and a stale entry fails as "tool not found", which is at
least loud.

| Legacy name | Live call |
|---|---|
| `domain_visindex` | `domain view=visindex` |
| `domain_visindex_overview` | `domain view=visindex_overview` |
| `domain_kwcount_seo` | `domain view=kwcount` |
| `domain_keywordcount_top10` | `domain view=kwcount_top10` |
| *(new — no legacy name)* | `domain view=urlcount` |
| `domain_competitors_seo` | `domain view=competitors` |
| `domain_ideas` | `domain view=ideas` |
| `domain_opportunities` | `domain view=opportunities` |
| `domain_ranking_distribution` | `domain view=ranking_distribution` |
| `domain_traffic_estimation` | `domain view=traffic_estimation` |
| `keyword_seo` | `keyword view=seo` |
| `keyword_seo_metrics` | `keyword view=metrics` |
| `keyword_seo_traffic` | `keyword view=traffic` |
| `keyword_seo_competition` | `keyword view=competition` |
| `keyword_seo_searchintent` | `keyword view=searchintent` |
| `keyword_seo_serpfeatures` | `keyword view=serpfeatures` |
| `keyword_seo_traffic_estimation` | `keyword view=traffic_estimation` |
| `keyword_domain_seo` | `keyword_domain_seo` (unchanged, still its own tool) |
| `links_list` | `links view=list` |
| `links_linktexts` | `links view=linktexts` |
| `links_linktargets` | `links view=linktargets` |
| `project_overview` | `project view=overview` |
| `project_competitors` | `project view=competitors` |
| `project_ranking` | `project view=ranking` |
| `project_visibilityindex` | `project view=visibilityindex` |
| `project_keyword_serps` | `project view=keyword_serps` |
| `project_onpage` (metadata) | `project_onpage view=meta` |
| `project_onpage_overview` / `_crawl` / `_issue` / `_urls` / `_links` / `_resources` / `_resources_usage` / `_cookies` | `project_onpage view=overview` / `crawl` / `issue` / `urls` / `links` / `resources` / `resources_usage` / `cookies` |
| `ai_entity` | **Not exposed by this MCP at last check** (see the AI section). Use `ai_check` — its `overview`, `competitors`, `prompts`, `prompts_count` and `sources` views cover the same ground for brands and domains |
| `ai_top` | **Not exposed by this MCP at last check** (see the AI section). No equivalent on the announced surface; don't substitute a tracker aggregate, it answers a different question |
| `ai_tracker` / `ai_tracker_overview` / `ai_prompt_answers` / `ai_topicresearch` | unchanged (each already took a `view` or needed none) |

**The failure this table exists to prevent, stated generally.** A companion
skill keyed on **tool names** rots silently when the server consolidates its
surface — every lookup instruction ("match by suffix") stops resolving, and
the skill still reads as authoritative. Key the documentation on the
**concept** — the dataset, the view, the question it answers — and keep a
name-mapping table as the only place old names live. Then a consolidation
costs one table edit rather than a rewrite, and the mapping can be checked
against the live schema in a single glance.

### Domain (`domain`)

**Two underlying keyword datasets.** The domain tools do not all draw on the
same keyword universe. Visibility Index and `domain view=ranking_distribution` are
computed over SISTRIX's **standard (VI-bearing) keyword set**;
`domain view=kwcount` and `keyword_domain_seo` draw on the much larger
**extended index** full of zero-volume long-tail. Numbers across these tools
can flatly contradict each other (e.g. after an update: page-1 keywords
collapsing 20→3 in `domain view=ranking_distribution` while `domain view=kwcount`
stays flat at ~85k). That's not a bug — name the dataset behind each number
before comparing anything across tools.

**UI exports and the API are different universes too.** A SISTRIX Toolbox UI
keyword export may carry a user-applied filter (e.g. **volume > 0**) that is
invisible in the exported file — so the API extended index can contain ranking
keywords (and dated positions) the UI export silently lacks. Two consequences:
(1) **never treat an MCP extended-index pull and a UI export as the same
population**; (2) when joining MCP data against a UI export, ask which filters
were applied at export time before interpreting any non-match. See the join-
validation step in the historical visibility-loss forensics recipe.

- `domain view=visindex` — current Visibility Index value + date. `history=true` for
  the weekly timeline, `daily=true` for daily, `date` for a point in time.
  - **`history=true` returns only ~100 most recent weeks by default.** Pass
    `limit=250` to extend the series years further back. But a 250-row history
    response is ~10k tokens inline, so per-domain histories **don't scale across
    a portfolio** — for many-domain work, prefer point-`date` calls (one cheap
    call per domain at the date you care about) over pulling full histories.
  - **`daily=true` is a rolling partial-sample series, not a daily census.**
    SISTRIX does not scrape all keywords every day, so a daily VI value only
    reflects the subset scraped that day. A sharp ranking shock therefore appears
    as a **smeared multi-day slide** as keywords rotate through the scraping
    schedule, not a clean step. Use **weekly VI for magnitude and event
    analysis**; never promise event-boundary attribution from daily VI. For
    timing an overlapping/ambiguous update, rely on cross-domain / cross-market
    patterns and loss-composition signatures (see the cross-domain
    natural-experiment recipe) rather than daily-VI timing.
  - **A single domain's series records that domain, not the business.** A VI
    curve that decays towards zero over a year reads as continuing damage and is
    just as consistent with the market having been moved to another host,
    domain or ccTLD. Establish the domain set before reading the shape (see
    the forensics recipe, step 0).
  - **Confirmed working on the consolidated surface** (recorded so a later
    session doesn't re-derive the call shape): `domain view=visindex
    history=true limit=140`, and `domain view=ranking_distribution` both with
    and without a `date`.
- `domain view=visindex_overview` — **a series-extent view, not a current
  value. Verified against the live server at last check — re-run the call
  to confirm.** It reports how much
  history exists and where the all-time extremes sit, and nothing else:
  `sichtbarkeitsindex_overview: [{domain, weeks}]` (the count of weekly data
  points on record), plus `sichtbarkeitsindex_overview_min` and
  `sichtbarkeitsindex_overview_max`, each a `{value, date}` pair. It **does
  NOT return the current value** — for that, call `domain view=visindex`.
  - *The evidence.* One call returned `weeks: 592` with a min dated 2015 and a
    max dated 2024, and no present-day figure of any kind. The control, same
    domain and same country in the same minute, `domain view=visindex`,
    returned a dated current value (`{date, value}`) that the overview view
    does not carry. One call plus one control, under a minute.
  - **The announced description is wrong on both counts.** It promises
    "current value + country/SERP breakdown": the current value is absent, and
    **no breakdown of any kind is returned** — not by country, not by SERP
    feature, not by anything. Document this view from its rows, never from its
    description (see "This server's tool descriptions over-promise" in the
    principles section — this is one of the three dated instances behind that
    rule).
  - *What it is good for.* Establishing how far back the series runs before
    asking for history (so a `limit` is chosen against a known span rather
    than guessed), and getting the all-time peak and trough with their dates
    in a single cheap call.
- `domain view=urlcount` — how many of the domain's **URLs** rank in the
  organic top 10 and top 100. Takes `history`, `date`, `limit`. This counts
  **ranking URLs, not keywords**, so it is the breadth measure that answers
  "how much of the site earns rankings" rather than "how many terms do we
  rank for" — pair it with `kwcount`/`kwcount_top10` to separate a footprint
  that grew by ranking more pages from one that grew by ranking the same pages
  for more terms. Which keyword universe it is computed over is **not
  verified**; name it as unverified rather than asserting standard or extended
  index when reporting alongside the counts above.
- `domain view=kwcount` — total ranking keywords (extended index).
  `domain view=kwcount_top10` — subset ranking in the top 10. Both take
  `history=true`. `domain view=kwcount` also accepts a historical `date` param
  (not in all schema docs) for point-in-time counts.
- `domain view=ranking_distribution` — share/count of keywords by results page
  (`page01`..`page10`); `percent=true` for shares. Also accepts a historical
  `date` param for point-in-time distributions (standard keyword set — the
  right tool for "how did our page-1 footprint change").
- `domain view=traffic_estimation` — top URLs by estimated clicks
  ({path, click_estimation, share, keywords, estimated_value}).
- `domain view=competitors` — a plain array of competitor domain strings.
- `domain view=ideas` — a plain array of keyword strings. **Noisy** — it returns
  loose topical-context terms that are often off-topic (a bicycle shop returned
  "lasagne"). Treat as brainstorming fuel, not vetted suggestions; filter with
  `regex_keyword`.
- `domain view=opportunities` — keywords with upside ({gain, keyword, position, url,
  competition}).
- `keyword_domain_seo` — every keyword a domain ranks for ({keyword, position,
  url}). Rich filters: `from_pos`/`to_pos`, `serp_feature`, `ai_answer`,
  `regex_keyword`/`regex_url`/`regex_host`. This is the domain-centric workhorse.
  Filter gotchas:
  - **`ai_answer=true` can't be verified from the response** — the returned rows
    carry no `ai_answer` flag, so a filtered result looks identical to the
    unfiltered top keywords. Treat with caution; cross-check via
    `keyword view=serpfeatures` or `ai_check view=prompts` rather than trusting
    the filter silently applied.
  - **`regex_keyword` is confirmed working on the consolidated surface** — and
    an `Error (1000)` on a tight regex is only readable as a real absence once
    a **control call returns rows**. An exact-match regex that returns
    `Error (1000): no result` while an identical call with a looser
    `regex_keyword` (or none) returns rows is evidence the keyword genuinely
    isn't in the domain's ranking set; the same error with no control is
    equally consistent with a malformed pattern, a wrong country or an empty
    scope. Run the control in the same turn, not after someone questions the
    finding.
  - **`serp_feature` expects the UPPERCASE feature tokens** returned by
    `keyword view=serpfeatures` (e.g. `ORGANIC`, `PRODUCTS`, `RELATED_QUESTION`,
    `VIDEO`), not lowercase snake_case. An invalid value throws a **server-side
    `Error (1007): invalid SERP-Feature` that does NOT list the valid values** —
    look the tokens up with `keyword view=serpfeatures` first.
  - `from_pos`/`to_pos` works correctly for position-range / striking-distance
    filtering.
  - **Undocumented `date` param, honoured for historical pulls** — but dated
    results are **position-sorted over the extended index**, so the top rows
    are zero-volume ultra-long-tail rankings that survive updates untouched.
    Before/after top-100 pulls can look nearly identical even across an ~89%
    VI collapse. Dated pulls are **qualitative pattern samples only** — you
    cannot build a historical volume-weighted "top loser" list from this tool
    (data ceiling; the SISTRIX Toolbox UI has that view). See the
    historical visibility-loss forensics recipe, including the mandatory
    undated control call.

### Keyword (`keyword`)

- `keyword view=seo` — the live organic SERP for a term ({domain, position, url}).
  Optionally filter by `domain`/`url`.
  - **Always run a control call without the `domain` filter before interpreting
    absence.** `keyword view=seo` with a `domain` filter returns `Error (1000): no
    result` in **two indistinguishable cases**: the domain isn't in the SERP,
    AND the keyword has no SERP data at all. An undated, unfiltered control call
    proves whether the data source itself has content before you conclude "the
    domain doesn't rank."
  - **For historical positions of a *specific* keyword, prefer
    `keyword_domain_seo(date, search="<exact keyword>")` over
    `keyword view=seo(date, domain)`.** `keyword view=seo` with `date` + `domain` returns
    `Error (3001): no keyword history` for low/zero-volume keywords — per-keyword
    SERP history isn't stored outside a higher-volume keyword set. The same
    historical positions **are** retrievable via the domain-centric extended-index
    route `keyword_domain_seo` with `date` + `search` (single-row bare-object
    response). Know both routes before declaring the data unavailable.
- `keyword view=metrics` — the one-call summary: {competition (0–100), cpc,
  traffic, clicks, desktop_distribution, mobile_distribution}.
- `keyword view=competition` — just {competition}.
- `keyword view=searchintent` — {intent_website, intent_know, intent_visit,
  intent_do}. **These are four independent 0–100 scores, NOT a distribution —
  they do not sum to 100. Verified against the live server at last check;
  the call below reproduces it.** One
  call — a generic German product term, `keyword(keyword="laufschuhe",
  view="searchintent", country="de")` — returned
  `{intent_website: 11, intent_know: 28, intent_visit: 0, intent_do: 100}` —
  **sum 139**. That sum is the whole proof: a distribution cannot exceed its
  own total. The dimension scored 100 is the dominant intent; read the four
  relatively, never as percentages.
  - **The error the shape invites.** Four numbers under 100 in one object look
    like a mix, so the reflex is to normalise them (139 → divide through) and
    report "39 % informational". Both steps are wrong: the normalised figures
    are arithmetic on unrelated scales, and the resulting percentages will be
    quoted back as if a share of searches carried that intent. Report the raw
    scores with the scale named ("intent scores, 0–100 each, independent"), or
    report the dominant intent and the runner-up — never a pie.
  - The schema describes this view as a classification "with shares". It is
    not; the shares it promises do not exist. Another of the three dated
    instances behind "This server's tool descriptions over-promise" in the
    principles section.
- `keyword view=serpfeatures` — a feature→value map (ORGANIC, PRODUCTS,
  RELATED_QUESTION, VIDEO, …). `show_all_types=true` also lists untriggered
  feature types.
- `keyword view=traffic` — {traffic, clicks, country}. No `history` param;
  use `show_all=true` for the timeline and `show_all_countries=true` for
  cross-country volume. **`show_all_countries=true` is a heavy payload** —
  200+ rows including many zero-traffic countries, **duplicate country codes**
  (e.g. `th`, `nz`, `tw` appear more than once), and codes **outside** the
  documented 52-code SEO set (`kr`, `li`, `ba`, `al`, `ge`, `by`, `hk`, …).
  Sort by traffic and filter client-side before using it.
- `keyword view=traffic_estimation` — estimated clicks per SERP position.

**Coverage is not demand — two readings, one for the `keyword` family and
one for its AI sibling.**

- **`Error (1000): no result` on a volume/traffic call for a term with
  obvious demand means the keyword is not in that country's database yet,
  not that nobody searches it.** The keyword index is built from what has
  already been crawled and clustered, so a term from a category the market
  only recently acquired a vocabulary for can be absent in one country while
  carrying solid volume in another. Observed twice: on an emerging product
  category a few months old, where two category terms returned Error 1000 in
  one market while the same strings carried four- and five-figure volumes in
  another; and on a compliance term in one European market, which returned
  Error 1000 in that country's language while its direct equivalent carried
  four-figure volume in a neighbouring market — and the site's own
  search-console data showed real clicks in the language that returned
  nothing. **The control:** run the same call with `show_all_countries=true`
  (or the equivalent cross-country view) to establish whether the term exists
  in other countries' databases at all, and run a sibling term of known
  demand in the same country. Solid volume in one country plus Error 1000 in
  another is database lag, not absent demand; a sibling that returns while
  the category term does not confirms the gap is coverage. Never present such
  a no-result as a cross-market demand comparison — label it "not covered",
  and take the demand read from a source that observes the market directly
  (search-console data in that market's language, site search, community
  questions).
- **The same shape appears on `ai_topicresearch`: an overview returning 0
  topics for an emerging entity or category describes the database, not the
  market.** Topic research is built over what has already been crawled and
  clustered, so a young category name or a recently-introduced entity can
  return an empty topic set while the category demonstrably has content
  demand — seen alongside the keyword Error 1000s above, in the same runs.
  Apply the same control and the same wording: check a sibling or parent term
  that the database does cover, and report zero topics as "not covered yet",
  never as "no topics exist". The general rule behind both readings: a demand
  database measures what it has already indexed, so the newer the category,
  the more its absences describe the database rather than the market. For any
  category younger than roughly a year, establish coverage before using these
  numbers to weight or allocate anything across markets.

### Links (`links`) — domain only, no country

- `links view=linktargets` — most-linked URLs ({url, links, domains, hosts, nets,
  ips}).
- `links view=linktexts` — plain array of anchor-text strings, returned **raw**
  (includes spam/SEO-vendor anchors verbatim — don't treat presence as
  endorsement).
- `links view=list` — individual backlinks. Note the **dotted keys**: `url.from`,
  `url.to`, plus `text`. Filter with `regex_link_from`/`regex_link_to`/
  `regex_link_text`.

**Two cross-cutting rules for any spam/link-profile analysis:**

- **Classify spam by where links point, not what they say.** A spam classifier
  keyed on niche terms in **anchor text** (`links view=linktexts` /
  `regex_link_text`) systematically undercounts: spammy links' anchors are
  often generic SEO-tool-flavoured strings, while the niche strings live in the
  **target URLs**. Classify by target-URL patterns first (`links view=list` with
  `regex_link_to`; `links view=linktargets` URL slugs); treat anchor-text composition
  as a secondary campaign signature. Don't assume a spam campaign uses
  niche-related anchors.
- **The `links` views return current snapshots only.** Historical link profiles
  cannot be reconstructed from them. Any cross-domain spam-share comparison used
  to explain a *past* outcome therefore assumes link accumulation/persistence
  over time — state that assumption explicitly whenever a link profile informs
  historical attribution.

### AI visibility (`ai_*`)

- `ai_tracker_overview` — list your AI trackers (hash + name). Call first.
- `ai_check` — **the ad-hoc, tracker-free entry point**, and the one to reach
  for when a brand has no tracker configured. Pass any number of `brands`
  and/or `domains` (single string or list, up to 100 each; at least one of the
  two is required) and pick a `view`: `overview` (analysed scope, prompt count,
  per-model and per-country breakdown), `competitors` (competing brands),
  `prompts` (the most relevant prompts including model and answer text),
  `prompts_count` (historical prompt counts per day — `limit` = history days),
  `sources` (cited sources, with `source_type` = `host` | `domain` | `url`).
  View-specific optional params (`model`, `source_type`, `limit`, `page`) are
  **silently ignored** when the chosen view doesn't accept them, so a call can
  look scoped and not be. Two things to note before treating it as a drop-in
  for the old per-entity tool: it is **brand/domain-keyed, not entity-keyed**,
  and its view set has **no `environment` view** — for the environment/
  co-occurrence reading, use `ai_tracker view=environment` on a configured
  tracker.
- **`ai_top` and `ai_entity` — not callable through this MCP at last
  check.** The announced AI surface, read off the server's own tool list at
  that check, is exactly five tools: `ai_check`, `ai_prompt_answers`,
  `ai_topicresearch`, `ai_tracker`, `ai_tracker_overview`. Neither `ai_entity`
  nor `ai_top` appears.
  - **Be precise about what that establishes.** A tool listing settles
    *addressability through this MCP* and nothing more: these two are not
    callable here, which is the fact that matters to anyone following this
    skill. Whether the underlying SISTRIX API endpoints still exist is a
    different question that no tool listing can answer — don't report them as
    "retired from the API", and don't tell a user their non-MCP script is
    broken on this evidence.
  - **Where to go instead.** For the per-entity readings, `ai_check` is the
    equivalent — its `overview`, `competitors`, `prompts`, `prompts_count` and
    `sources` views line up with the old `ai_entity` views of the same names,
    with the keying and coverage caveats noted above. For the global Top-N
    there is no equivalent on this surface: say so rather than substituting a
    tracker aggregate, which answers a different question.
  - The readings preserved below were written against the older surface and
    are kept **only** so an agent reading an older script or report knows what
    the call used to mean and where it maps to now. Do not issue them.
  - Note the general reading, from "an enumeration proves a thing is
    addressable, not populated or current": a name's absence from the schema
    is the loud, cheap failure — the dangerous case is a name that still
    resolves and no longer means what the documentation says.
- `ai_top` *(not on the MCP surface — historical, see above)* — global Top-N.
  `view`: `sources` |
  `entities` | `brands`. **The top-level response key is always `prompts`**
  regardless of view (misleading); rows are {host/brand/entity, rank, amount,
  visindex}. The `lang` param's filtering effect is unverified — results have
  looked global; confirm before relying on language scoping.
- `ai_entity` *(not on the MCP surface — historical, see above)* — per-entity analysis. `view`:
  `overview` (just a `prompt_count`),
  `prompts` (the actual AI answers mentioning the entity), `prompts_count`,
  `sources`, `competition`, `environment`. View-specific params (`model`,
  `source_type`, `expand_models`) are silently ignored when the chosen view
  doesn't use them.
  - **`view=competition` is a co-occurring-entity list, not a clean competitor
    set.** It mixes real competitor brands with topical concepts/standards AND
    includes the queried entity itself (e.g. for a coffee roaster it
    returned a 50/50 mix of rival roasters and abstract terms like "Arabica",
    "Roast", "Espresso"). Filter the concepts out — or cross-check against a
    known-competitor list / `ai_tracker` competitors — before presenting it as
    competitors. For a vetted competitor ranking, prefer a configured
    `ai_tracker`'s competitors view.
  - **`view=overview` `prompt_count` is an entity-salience signal, not just a
    volume.** An inflated `prompt_count` paired with an off-topic `environment`
    (see below) signals a **homonym** — the model's entity for that token isn't
    your brand. Short/common brand tokens are most at risk.
- **Per-country counts need a no-country baseline.** The `country` filter on
  `ai_*` tools can silently no-op: `ai_entity` with `country=nl` returned a
  `prompt_count` identical to the all-countries aggregate on repeated calls
  (both `overview` and `prompts_count` views), while `de`, `es`, `it`, `fr`
  and `us` returned distinct, plausible values. A per-country value **exactly
  equal to the aggregate is a silent-fallback artefact, not data** — treat
  such a match as "market split unavailable" rather than reporting it as a
  market footprint. General rule: when an API filter can silently no-op,
  filtered == unfiltered is the tell — pull the unfiltered baseline before
  trusting any segmented number.
- `ai_prompt_answers` — the generated answer text for a given prompt. Filter by
  `model` / `country`. **Critical scope limit: it only serves SISTRIX's
  global/standard crawled prompt pool, NOT the custom prompts configured inside
  an AI tracker.** A custom tracker prompt returns `SISTRIX API Error (1000): no
  result` here even though it's fully measured via `ai_tracker view=prompts`
  (a control prompt from the global pool, e.g. "best running shoes uk", returns a
  full answer). Consequence: you **cannot** reconstruct per-prompt, provider-by-provider
  share-of-voice or answer text for tracker prompts from the MCP. Don't burn
  calls discovering this, and don't promise a per-prompt SOV the data can't
  support — for tracker prompts the only competitor SOV the MCP exposes is the
  `ai_tracker competitors` aggregate (see its caveats below).
- `ai_tracker` — per-tracker views. Pass a tracker hash. `view`: `prompts`
  ({prompt, brand_found, avg_pos, tags, models}), `competitors` ({name, visindex,
  mentions, prompts}), `environment` ({misc, prompt_count, mentions}), and
  `sources_domains` | `sources_urls` | `sources_hosts` ({url | domain | host,
  prompt_count, mentions, brands} — the identifier field is named for the
  view; `view=sources_urls` was confirmed to return `{url, prompt_count,
  mentions, brands: [...]}`). **Note: `prompts` AND the `sources_*` views both
  nest under a top-level `source` key** — don't key off the field name, key off
  the view you asked for.
  - **`brands` on a source row is answer-level, and it is not a clean entity
    list.** It is computed over the **citing answers**, not over the source
    page, so it says nothing about what the cited URL contains. Worse, the
    observed values were not brands at all but raw substring fragments of the
    answer text: a bare category word with a trailing space, a run-together
    title-and-date string, two concatenated URLs with brand text stuck to
    them, and a truncated word. It is a token dump. **Never use `brands` as a
    brand-presence test for a page** — not to decide that a page already
    covers the brand, not to build a gap list, not to route a page into a
    workflow that depends on what the page says. Calibration from one pull:
    of 12 cited URLs carrying the brand token in `brands`, fetching them
    showed **3 of the 12 did not mention the brand anywhere**. Treat the field
    as a ranking hint and confirm by fetching.
  - **`view=prompts` gives `brand_found` and `avg_pos` only — no execution
    counts — so per-answer visibility is not derivable from the API.**
    `brand_found` is a per-question **ever**-flag (true if the brand was named
    in any single answer at any point in the tracker's window), which is an
    upper bound on the share of answers naming the brand, not that share. Never
    present a `brand_found` tally as visibility. The per-answer figure lives
    only in the Toolbox UI — see
    `sistrix-tracking-strategy-builder/references/sistrix-implementation.md`
    for the UI route and the extraction recipe.
  - **Dedupe `view=prompts` before tallying.** At high `limit` the response can
    repeat prompts — a canonical tagged set plus a trailing block of the same
    prompt strings with `tags:[]`, a reduced model-coverage slice, and
    sometimes different `brand_found`/`avg_pos`. Counting rows naively
    double-counts and corrupts presence rates. Deduplicate by prompt string
    (and decide whether the multi-engine row or a per-engine row is canonical)
    before computing any presence/gap statistic.
  - **`competitors` is an all-prompts aggregate that structurally favours the
    tracked brand**, and it **cannot be filtered by tag or prompt type via the
    API** — so it can't be reduced to a clean discovery-only ranking. See the
    branded-vs-discovery rule below before reporting anything off it.
  - **The three `sources_*` views are the same kind of aggregate** — they take
    only `model`, no tag or prompt filter — and the branded-prompt bias lands
    on the tracked brand's **own domain** row (plus its Wikipedia entry and
    third-party profile pages). Same rule below applies before ranking any
    source off them.
- **`ai_tracker`'s `model` vocabulary — `aio`, `aimode`, `chatgpt`,
  `perplexity`. Verified against the live server at last check.** These are the
  identifiers seen in live responses *and* the ones the filter accepts:
  `aio` (Google AI Overviews), `aimode` (Google AI Mode), `chatgpt`,
  `perplexity`.
  - *The evidence, three calls on one tracker's `competitors` view in the same
    minute.* `model="gpt"` → `SISTRIX API Error (1002): invalid parameters`.
    `model="chatgpt"` → succeeded, and the figures moved against the
    unfiltered control (top brand 33 mentions across 6 prompts filtered,
    versus 133 across 22 unfiltered). No `model` → unfiltered. So the filter
    is genuinely applied, not silently ignored, and `gpt` is rejected outright
    rather than quietly matching nothing.
  - **The schema's `'gpt'` / `'gemini'` examples are wrong for this tool.**
    `gpt` is rejected by the API. Don't copy a model identifier out of a
    parameter description; take it from a live response or from the list
    above.
  - **Scope this to `ai_tracker`.** `ai_check` has its own `model` parameter,
    whose description offers `'gpt'` and `'aio'` as examples. That parameter
    was **not tested** at that check and its accepted vocabulary is
    **untested** — do not assume `ai_tracker`'s result transfers. If you need
    a model filter on `ai_check`, run the same two-call probe first and record
    what you find.
  - Other engines (e.g. `deepseek`) may appear depending on account/coverage
    but were **not observed** — don't filter on a model name you haven't
    confirmed is present.
  - **The technique, worth reusing anywhere.** A filtered call on its own
    cannot tell you the filter was applied: a silently-ignored filter returns
    a perfectly plausible result. **Run an unfiltered control through the
    identical call in the same minute.** A number that *moves* proves the
    filter was applied; a number *identical* to the unfiltered baseline proves
    nothing and should be read as "silently ignored" until shown otherwise.
    Same minute matters — it removes data refresh as the explanation for the
    difference. This pairs with the `filtered == unfiltered` rule in the
    principles section, which covers the failure this control detects.

> **Token-cost trap.** `ai_check view=prompts` (and `ai_entity view=prompts` if
> it is reachable on an older surface) and `ai_prompt_answers` return
> full AI answers as large HTML blobs (`<highlight>`, embedded source lists,
> hundreds of lines each). A `limit` of 5 can be tens of thousands of tokens.
> Keep `limit` to 3–5, and if you only need counts use `view=overview` /
> `prompts_count` instead. Strip HTML before quoting answer text to the user.

### Project / Optimizer (`project`)

- `project view=overview` — list projects (hash + name). Call first.
- `project_onpage view=meta` — project metadata ({hash, name, tags}). Note that
  this metadata view sits on the **onpage** tool rather than the `project` tool,
  even though it is not crawl data — the legacy name for it was the bare
  `project_onpage`, which is why older notes describe a metadata tool "despite
  the name".
- `project view=ranking` — tracked rankings. **Deeply nested:**
  `optimizer.rankings[0].optimizer.ranking[]`, each row {keyword, position, url,
  tags, device, country, traffic, searchengine, date}. Filter by `tag` or
  `regex_keyword`.
- `project view=visibilityindex` — project domain's VI timeline ({host, date, value}).
  `competitors=true` is intended to add competitor curves, but on a current-only
  call it may return just the host's value — combine with `date`/history if you
  need the competitor series.
- `project view=competitors` — configured competitor set ({domain, match, visindex});
  includes the project's own domain at `match=100`.
- `project view=keyword_serps` — the live SERP ({pos, url}) for a keyword in the
  project's market; supports `city`, `device`, `searchengine`.
- **Onpage crawl views** — `project_onpage` with `view` = `overview`, `crawl`,
  `issue`, `urls`, `links`, `resources`, `resources_usage`, `cookies` (plus
  `meta` above). These only
  return data if the project has a completed onpage crawl. If onpage crawling
  isn't enabled, `view=overview` returns empty and the others return
  `SISTRIX API Error (1000): no result`. That's a **data-availability** signal,
  not a broken tool — confirm the project actually runs onpage crawls before
  concluding the tool failed. Shapes below are **verified** against a completed
  crawl of a large e-commerce site (~138k pages). Note that nearly every tool
  nests the rows under a per-tool key inside an `optimizer.project` (or
  `optimizer.onpage.*`) envelope — read the actual key, don't assume `result`.
  - `project_onpage view=overview` → `{optimizer.onpage.overview: [{time, pages,
    error, warnings, notice}]}`. One row = the latest crawl's totals. There is a
    **third severity tier, `notice`**, alongside `error`/`warnings` — by far the
    largest bucket in practice (here 69,963 notices vs 3,116 errors). Despite
    its tool description ("error/warning totals"), it does surface notices.
  - `project_onpage view=crawl` → a **bare top-level array** of
    `{type, name, count}`, `type` ∈ `error` | `warning` | `notice`. The `name`
    values **are** the issue keys you pass to `project_onpage view=issue` (e.g.
    `incorrect_url_format`, `alt_attribute_missing`, `same_h1`,
    `broken_external_link`). Call this first to discover which issue keys
    actually have rows before drilling in — it saves blind `view=issue` calls.
  - `project_onpage view=issue` → `{optimizer.onpage.issue: [{count,
    optimizer.onpage.issue: [{url_score, url}]}]}`. **Key reuse / double
    nesting**: the outer `optimizer.onpage.issue` is a one-element array whose
    element holds `count` plus an inner array **under the same key name** with
    the rows. Rows are **minimal — just `{url_score, url}`**, no title/h1/etc.
    To get page detail for a flagged URL, join against `project_onpage view=urls`.
    Takes one of 105 `issue` keys (e.g. `title_tag_missing`, `missing_h1`,
    `duplicate_content`, `broken_external_link`, `noindex`); an invalid key is
    rejected **client-side** with a validation error listing all valid keys, so
    a wrong key is cheap — read the list and retry.
  - `project_onpage view=urls` → `{optimizer.project: [{hash, name, count,
    optimizer.project.urls: [{...}]}]}`, `count` = total crawled URLs. Rows are
    rich: `url_score` (0–100), `url`, `title`, `title_num`, `size` (bytes),
    `time` (seconds, float), `content_type`, `charset`, `http_code`, `level`
    (click depth), `links_int`, `links_ext`, `meta_index` (1/0),
    `meta_description`, `canonical`, `h1`, `h1_num`, `h2`, `h2_num`, `h3`,
    `h3_num`, `source`. The workhorse for page-level onpage analysis.
  - `project_onpage view=links` → `{optimizer.project: [{hash, name, count,
    optimizer.project.resources.links: [{from, to, type, status_code,
    source_status_code, follow, text, content_type}]}]}`. `type` ∈ `INT`/`EXT`;
    `follow` is an int flag. **Flat `from`/`to` keys here** — contrast the
    backlink tool `links view=list`, which uses dotted `url.from`/`url.to`. `count`
    is the *total* link count and can be enormous (5.5M+ on a large site), so
    always paginate — this family uses `page`, **not** `offset`.
  - `project_onpage view=resources` → `{optimizer.project: [{hash, name, count,
    optimizer.project.resources: [{type, url, status_code, size}]}]}`. `type` ∈
    `IMG`/`JS`/`CSS`/… Returned largest-`size`-first — handy for finding heavy
    assets.
  - `project_onpage view=resources_usage` → `{optimizer.project: [{hash, name, count,
    optimizer.project.resources.usage: [{type, url, status_code, size, time,
    source, image_alt, image_title}]}]}`. Maps each resource to the `source`
    page that loads it (plus `image_alt`/`image_title`), so `count` is the
    total resource-on-page usage rows (1.47M here) — much larger than the
    distinct-resource count from `view=resources`. Paginate with `page`.
  - `project_onpage view=cookies` → can **still return `SISTRIX API Error (1000): no
    result` even on a completed crawl** (observed on a live crawl at last
    check). Cookie capture is
    gated separately from the main crawl — treat a 1000 here as "no cookie data
    for this crawl", not evidence the crawl is unfinished.

---

## Response shape — assume nothing

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

## Error taxonomy & resilience

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

## Workflows

### Domain visibility snapshot (any market)
1. `domain view=visindex` (current value) + `domain view=visindex_overview`
   (history span, all-time min/max — and **only** those; the overview view
   carries no current value and no breakdown, see the Domain section).
2. `domain view=kwcount` and `domain view=kwcount_top10` for breadth.
3. `domain view=ranking_distribution` (`percent=true`) for quality of positions.
4. `domain view=competitors` for the competitive set, `domain view=traffic_estimation`
   for where traffic concentrates.
Repeat per `country` for cross-market comparison — change only the country code,
and double-check each code is in the supported set.

**Across a portfolio of many domains, probe before you drill.** Run a single
`domain view=visindex` point-`date` call per domain first (the event/snapshot week) to
eliminate dead or zero-VI domains cheaply, then pull breadth/history only for the
survivors. Full per-domain `domain view=visindex history=true limit=250` pulls are
~10k tokens each and don't scale across a portfolio — point-date calls do.

### Keyword research
`keyword view=metrics` (volume/CPC/competition/device split) →
`keyword view=searchintent` (intent mix, scored 0–100 each) →
`keyword view=serpfeatures` (what the SERP looks like) →
`keyword view=seo` (who ranks). Use `keyword view=traffic show_all_countries=true` to
find the strongest market for a term.

### AI-visibility audit
For a brand without a tracker, use `ai_check` (see its caveats; the older
`ai_entity` route was not on the MCP surface at last check):
`ai_check view=overview`
(footprint) → `view=competitors` (who else shows up) → `view=prompts limit=3`
(sample answers, mind the token cost). For a tracked brand: `ai_tracker_overview` → `ai_tracker
view=prompts` (per-prompt brand_found / avg_pos; dedupe before tallying) →
`view=competitors` (share of voice — all-prompts aggregate, see caveats) →
`view=sources_domains` (which sources AI cites — same all-prompts aggregate; the
own-domain row is inflated by branded prompts). Note which `models` each prompt
covers. **Before reporting any number, split branded vs discovery prompts** (see
that section) and, for "best/top [X]" prompts, run the
"Why doesn't our directory rank convert" recipe.

### Optimizer project monitoring
`project view=overview` → `project view=visibilityindex` (trend) → `project view=ranking`
(movements; remember the nested shape) → `project view=competitors`. Attempt onpage
(`project_onpage view=overview`) only after confirming the project runs crawls.

---

## AI visibility — branded vs discovery (read before reporting)

Before reporting any AI visibility / share-of-voice number, **classify the
prompts into branded vs discovery and report them separately.** This is a hard
rule, not a nicety — skipping it produces a flattering, wrong headline.

- **Branded prompts** name the brand or are head-to-head ("is X a good provider",
  "X vs Y", "what is X known for"). The tracked brand is essentially guaranteed
  to appear at position 1 on these, and no competitor has equivalent branded
  prompts in a tracker built around X.
- **Discovery prompts** are category/need-based and brand-agnostic ("best [X]
  for [need]", "top [category] providers"). **Discovery is the meaningful
  measure for new-business visibility.**

Why it matters:

- The `ai_tracker competitors` view aggregates across **all** prompts, so the
  home brand is structurally advantaged. Reporting "we're #1 by visibility
  index" off that aggregate overstates standing. And the view **cannot be
  tag/prompt-filtered via the API**, so a clean discovery-only competitor
  ranking can't be produced from that endpoint at all — caveat it, or rebuild
  share-of-voice from per-prompt data where possible (but note `ai_prompt_answers`
  won't serve tracker prompts, so per-prompt provider-level SOV often can't be built
  from SISTRIX alone).
- Compute the brand's own discovery presence from `ai_tracker view=prompts`
  filtered to non-branded prompts (tags + brand-name detection), after deduping
  (see the `ai_tracker` note above).
- **The `sources_domains` / `sources_urls` / `sources_hosts` views carry the
  same contamination, and it lands on a different row.** They take only
  `model` — no tag or prompt filter — so they aggregate over every prompt in
  the tracker, branded ones included. Branded prompts ("is X a good provider",
  "X vs Y", "what is X known for") are answered from X's own About pages, its
  Wikipedia entry and third-party profile pages, so the tracked brand's **own
  domain** rises in the citation ranking. Third-party rows (directories, rival
  sites) are barely affected; the own-domain row is the one that lies. In one
  ~100-prompt tracker with four branded prompts, the brand's own domain came out
  as "the most-cited site of its kind on every engine" and was stated to the
  user before the contamination was noticed. Two remedies: read the own-domain
  row only from the Toolbox UI sources report with the branded tag excluded, or
  drop the own-domain row from any cross-engine or cross-source ranking. Either
  way, bound the contamination first — branded prompts × engines × answers per
  prompt — before quoting anything off these views. Third-party source rows may
  be quoted, with the caveat that the period is the tracker's whole window.
- **The UI remedy has its own silent failure: reconcile the UI's totals
  against the API's before trusting a tag-filtered sources figure.** The
  Toolbox sources report is the only tag-filterable route to these views, but
  its multi-select tag filter drops tags whose names contain an ampersand —
  the name is HTML-entity-encoded before URL-encoding, so it matches nothing
  and is removed from the selection without a warning or an empty state. The
  result is a smaller, plausible number: an "everything except the branded
  tag" selection of fourteen tags applied only the six ampersand-free ones and
  returned roughly 45% of every unfiltered count across three different source
  rows. So when a UI sources figure and the `ai_tracker` `sources_*` output
  disagree, **a partly-ignored tag filter is a live hypothesis before any
  story about a smaller population**. Three checks, in order: (1) add a tag
  known to contribute and confirm the totals move — if filtered == unfiltered,
  nothing was applied; (2) reconcile the filtered figure against the
  unfiltered API total and ask whether the shortfall matches the prompts you
  meant to exclude or the tags the filter could encode; (3) fall back to one
  tag at a time via the report's per-tag URL, which applies correctly. Do not
  read the tag chips displayed in the UI as the filter that ran — they
  describe the selection, not the query.
- **Separate "found" from "prominent".** A brand can be present but mid-pack
  (avg_pos 6–7) on the highest-value broad prompts — that is not the same as
  leading them.

AI-visibility numbers are only comparable across brands once branded prompts
are removed; otherwise the brand the tracker is built around always looks like
the leader. The general form: a view that aggregates over the whole prompt set
inherits every bias in the prompt set, and the bias lands on whichever row the
biased prompts feed — the tracked brand's mention count in `competitors`, the
tracked brand's own domain in `sources_*`. Before ranking anything off an
unfilterable aggregate, name the row the contaminating subset feeds and either
filter it elsewhere or leave it out.

---

## Analysis recipes

The MCP's highest-value use is combining tools into analyses no single tool
delivers. These are client-agnostic; substitute your own domain/market.

**Visibility-drop diagnosis (don't read keyword-count deltas naively).**
`domain view=visindex` history + `domain view=ranking_distribution` history (counts) +
`domain view=kwcount` history. Attribute a decline to **position slippage** (page-1
footprint shrinking) vs **long-tail attrition** (losses concentrated in page 3+).
A "keyword count halved" headline overstates the visibility impact when the lost
keywords were low-VI deep-ranking long-tail — the decomposition tells you which.
For drops in the past (update post-mortems), use the historical forensics
recipe below.

**Striking-distance / quick wins.** `keyword_domain_seo from_pos=11 to_pos=20`
(+ `domain view=opportunities`) for near-page-1 terms, sized with
`keyword view=metrics` volume and the `keyword view=traffic_estimation`
CTR-by-position curve to rank by realistic upside.

**Cross-market prioritisation.** `keyword view=traffic show_all_countries=true`
(sorted, filtered — heavy payload, see its note) to rank markets by demand,
paired with per-country `domain view=visindex` to spot under-served markets.

**AI × classic bridge.** `keyword_domain_seo` (`serp_feature` / `ai_answer`) +
`ai_check view=sources` to see where existing rankings co-occur with AI answers and
who gets cited (mind the `ai_answer`-verification caveat).

**AI share-of-voice + citation target list.** `ai_tracker` competitors (share of
voice, with the branded-vs-discovery caveat) + `sources_domains` (which
intermediary domains to earn citations from).

**SEO competitor gap.** `domain view=competitors` + per-competitor
`keyword_domain_seo`, diffed client-side — there is **no native keyword-gap
tool**.

**Backlink anchor / toxicity scan.** `links view=linktexts` + `links view=list` with
`regex_link_*` filters.

**Sensitive/undesired-query audits.** When checking whether a domain ranks for
adult or otherwise unwanted queries (e.g. a retail domain whose product titles
spawn adult-term keyword combinations): SISTRIX has **no blanket adult-keyword
filter** — `keyword view=seo` returns full SERPs and `keyword view=metrics` returns
volume for unambiguous adult head terms. But the domain-level footprint for
long-tail undesired queries is **structurally under-represented**:
`regex_keyword` on the ranking set may find a handful of matches across tens of
thousands of keywords while GSC shows the domain ranking for many such queries,
because SISTRIX only observes rankings for keywords inside its own keyword
universe, and title-generated ultra-long-tail combos are rarely in any tool's
database. Absence in SISTRIX is evidence about the tool's database, not about
the domain's actual rankings. **Size such problems with GSC query data; use
`regex_keyword` only as a cheap spot-check.**

### Recipe: Stress-test a third-party report / URL-pattern claim

For independently verifying another report's headline claims — positions, which
URL type ranks, URL-pattern cannibalisation — from a different data source than
the one the report used. An independent source plus **pattern-level queries**
(regex over the ranking set) is faster and more convincing than re-checking each
cited figure one by one: it both confirms the pattern and reveals its true
prevalence (often far more pervasive than the report states).

1. **Control-call rule: run the unfiltered query first.** Establish that the data
   source has live content before reading anything into a filtered absence (the
   same control rule as `keyword view=seo` above).
2. **Test URL-pattern cannibalisation at domain scale with
   `keyword_domain_seo` + `regex_url=<pattern>`.** A pattern like
   `regex_url=p=[0-9]` surfaces every paginated `?p=N` URL the domain ranks for,
   per market — confirming a pagination/parameter phenomenon AND exposing how
   widespread it is (frequently many such URLs ranking in the top positions, not
   the handful a report implies). Works equally for other parameter patterns and
   blog/path patterns.
3. **Expose the URL and position behind a headline claim with `keyword view=seo`,
   NO domain filter, raised `limit`** (e.g. 15–20). The full live SERP shows the
   exact ranking URL and its position, plus the surrounding competitive/intent
   context — e.g. whether the SERP carries dictionaries / Wikipedia / Reddit,
   which signals mixed or informational intent behind the term.
4. **Reproduce the diagnosis, not the exact number.** Expect small position
   deltas between different SEO tools; the goal is to confirm the pattern, not
   match a competitor tool's figure to the decimal.

### Recipe: Historical visibility-loss forensics (update post-mortems)

For "what did the [algorithm update] cost us?" questions answered from
historical SISTRIX data. Several domain tools accept a historical `date` param
(`keyword_domain_seo`, `domain view=kwcount`, `domain view=ranking_distribution` —
not all documented in the schemas), but the datasets behind them differ, so
follow the sequence and its verification rules:

0. **Establish the domain set before reading any single series — and name the
   structural alternatives before naming a cause.** A visibility series is the
   joint output of the market and of every structural change to what SISTRIX
   was measuring: a domain migration or consolidation, a market moved from one
   ccTLD to another, a host or protocol switch, a subfolder folded into a
   subdomain. `domain view=visindex` on one domain cannot distinguish "this market
   lost visibility" from "this market left this domain" — a curve decaying to
   near zero over a year is the same curve in both cases. So, before step 1:
   ask the user (or check the client dossier) for every domain and host that has
   carried this market in the window; pull `domain view=visindex` for each of them
   (point-`date` probes first, per the history-limit note); and read the
   **combined** series. A migration shows up as a mirror-image handover between
   two curves with a flat or recovering sum; a real loss shows up in the sum.
   Only the sum is evidence about the business. Then, in the write-up, list the
   structural facts that could have generated the shape (migration, domain
   move, seasonality, keyword-set or country changes) and say which of them were
   checked — and run that check in the same turn as the observation, never
   report the decay first and the migration check afterwards. A finding
   delivered before its control has already been read by the time the control
   arrives, and a retraction costs more than the extra call would have.
1. **Bracket the drop.** `domain view=visindex` with `history=true` (or `date` point
   calls) to pin the window and size the VI loss.
2. **Decompose with before/after point-in-time calls.**
   `domain view=ranking_distribution` (standard/VI-bearing set) + `domain view=kwcount`
   (extended index) at dates either side of the drop. Expect contradictions:
   page-1 counts can crater (e.g. 20→3, top-100 ~2,990→~490) while the extended
   kwcount stays flat (83k→87k). Both are correct for their dataset — **name the
   dataset behind every number you report.**
3. **Dated keyword pulls = qualitative samples only.** `keyword_domain_seo` with
   `date` is position-sorted over the extended index; the top rows are
   zero-volume ultra-long-tail that updates don't touch, so before/after top-N
   pulls can look nearly identical despite a massive VI collapse. Use them to
   sample patterns (which page types / topics appear), never to build a
   volume-weighted "top loser" list — that's a **data ceiling** of the MCP
   (the SISTRIX Toolbox UI has that view; send the user there if they need it).
4. **Control-call rule.** Before trusting any before/after diff built on
   `date`, run an **undated control call** with otherwise identical params —
   it must return a visibly different (current) set, proving the param is
   honoured. A suspiciously similar before/after diff is a verification
   trigger, not a conclusion.
5. **Validate join non-match semantics before reporting "not ranked" percentages.**
   A common claim — "~98% of declined keywords don't rank today" — is often built
   by joining a keyword list against a current UI export and treating absence as
   "not ranked". But a UI export may carry an invisible **volume > 0** filter, so
   it omits ranking keywords whose recorded volume is currently 0 — **absence from
   a filtered export is not absence from rankings.** Before publishing any such
   percentage, test a handful of keywords **known to rank** (e.g. from an MCP
   extended-index pull) against the export; if any are missing, the export's
   filter is dropping rankers. Then report **conservative bounds** (">95%", not
   "97.9%") and state the filter caveat. MCP extended-index pulls and UI exports
   have different filters/universes — the interpretation of a non-match is defined
   by the filters of whatever you joined against, so confirm with known positives
   first.
6. **Current-state cross-check before anything ships.** Before a historical
   loss claim goes to a stakeholder, pull the **current** ranking set
   (`keyword_domain_seo`, undated) for the named keywords and flag which "lost"
   items have since recovered — then remove them or note the recovery
   explicitly. Recipients spot-check historical claims against live SERPs
   today; one recovered keyword on a "disappeared" list discredits the whole
   analysis. Prefer precise, falsifiable wording ("dropped out of the top
   results during the update window") over absolutes ("disappeared", "lost").

### Recipe: Cross-domain natural experiment (update attribution across a portfolio)

For "which update hit us, and why?" answered by comparing many domains — e.g. a
portfolio of sibling ccTLD domains, or a treatment domain against controls that
share its content/template. Three design rules keep it honest:

1. **Enumerate all outcome branches *before* pulling any data.** Framing one
   branch as the expected outcome biases the read of everything that follows.
   Write down the full set of possible explanations first, then let the data
   choose — don't pre-select a favourite and look for confirmation.
2. **Verify treatment isolation across controls — or model treatment intensity
   instead of presence.** If every "control" domain shares the same content/links
   as the treatment, a binary "has spam links / doesn't" can't differentiate
   anything. The differentiating variable is usually the **ratio** of spam to
   legitimate link equity (a trust-buffer measure), which the `links` views can
   proxy per domain. Prefer **magnitude-vs-exposure correlation across many
   domains** over a binary affected/unaffected framing.
3. **State the snapshot-only limitation of the `links` views.** They return current
   link profiles only — historical profiles can't be reconstructed — so any
   cross-domain spam-share comparison used to explain a past event assumes link
   accumulation/persistence and must say so.

Cheap-failing call sequence for the portfolio:

- **Probe one date first (the event week) to eliminate dead domains.** A single
  `domain view=visindex` point-date call per domain weeds out domains with zero VI
  before you spend any further calls on them — far cheaper than pulling full
  histories for the whole portfolio (see the `domain view=visindex` history-limit note).
- Drill into the surviving domains only, pairing per-domain VI magnitude against
  per-domain link-profile exposure.

### Recipe: "Why doesn't our directory rank convert to AI visibility?"

A diagnostic for "best/top [X]" recommendation prompts where a provider with
strong directory rankings is absent from AI answers in some topics but present
in others. **Run the steps; then respect the causal-method discipline below —
this is the area where confident wrong conclusions are easiest to reach.**

Diagnostic steps:

1. **`keyword view=seo` on the specialism/category head term** → the organic candidate
   pool. Is it generalist-directory-led (the big ranking directories) or
   specialist/boutique-led? In boutique-dominated discourse, a broad
   full-service provider structurally loses regardless of its directory band.
2. **`keyword_domain_seo` with `regex_keyword`** → does the provider have
   **on-topic** content ranking now (vs dated updates / navigational own-brand
   pages)?
3. **`ai_check view=overview` (plus `ai_tracker view=environment` where a
   tracker exists)** → entity salience and
   disambiguation. An inflated `prompt_count` with an **off-topic environment**
   signals a **homonym** — the model's entity for that brand token isn't the
   provider. Compare against a distinctively-named competitor that binds cleanly.
4. **For "best/top [X]" prompts, always pull `ai_tracker sources_domains` (or
   `ai_check view=sources`).** These answers are usually grounded in a small set of
   **third-party directories/listicles/rankings**, not the candidates' own
   sites. If so, the highest-leverage action is improving the provider's
   standing **within those intermediaries** for the weak topics — not publishing
   more owned content.

What actually drives "best [X]" inclusion (corrected — it is **not** the
provider's own page depth or backlinks; an on-page driver hypothesis was
disconfirmed by a same-URL natural experiment and uniformly negligible
backlinks):

1. **Directory BAND, matched to segment and label.** Top-of-table (Band 1 /
   Tier 1) converts; mid-band usually doesn't, because "best provider" answers
   compress to the top band. But the band is **segment- and taxonomy-specific**:
   a mid-market Band 1 does not win a premium-coded generic prompt, and a Band 1
   in an adjacent/parent specialism does not transfer to an emerging query label.
   Map prompt → the exact directory sub-ranking/segment before reading a band as
   predictive.
2. **Head-term organic pool composition** (step 1 above) — generalist-directory
   vs specialist/boutique-led.
3. **External corroboration breadth**, including entity disambiguation —
   often the cheapest big win, because an ambiguous brand token silently dilutes
   every topical signal the brand earns (fix via Wikidata/Wikipedia, `sameAs`,
   disambiguating context).

Explicitly warn the user: owned-page quality/backlinks and **even a position-1
organic ranking for the provider's own page** do not predict AI "best [X]"
inclusion.

### Causal-method discipline for GEO/AI-visibility diagnosis

The candidate explanatory variables (owned content depth, organic rank,
directory band, genuine strength) are usually **collinear** — they all reflect
underlying quality — so naive contrasts mislead. Rules:

- **Verify entity-to-URL cardinality before any page-level driver analysis.**
  Two entities with opposite AI outcomes that resolve to the **same URL** (merged
  service pages like "Home + Contents", "Fleet + Haulage" are common on
  large-org sites) instantly falsify a page-level hypothesis — identical page,
  identical backlinks. Extract real URLs from the live nav; never assume 1:1
  entity-to-URL mapping.
- **Verify a "natural experiment" premise before leaning on it.** "Same URL" can
  hide dedicated sub-pages and unequal topical emphasis. Fetch the page; confirm
  the asset is truly shared and treatment is symmetric. Avoid "proves it, full
  stop" language for observational correlations — state residual confounds and
  the test that would settle them.
- **Isolate causes with divergence cases, not collinear pairs.** To attribute AI
  presence to one factor, find a case where that factor **diverges** from the
  others — e.g. strong owned ranking + mid directory band, or strong band + thin
  page. Collinear pairs can't isolate causation.
- **Calibrate claim strength to n.** Two divergence cases are anecdotes, not a
  necessity/sufficiency law. State n, the confounds, and the **data ceiling**:
  SISTRIX exposes Google rankings only and no per-competitor, per-specialism AI
  presence, so a fully controlled multi-provider causal test can't be built from
  SISTRIX alone. Downgrade to "directional/suggestive" when n is small.
- **Never assert an AI engine's retrieval backend from priors** — it changes and
  is post-cutoff; verify with a web search. Treat "engine X cites source Y" as
  reflecting Y's ranking on X's underlying index (Google for AIO/AIMode, Bing for
  ChatGPT Search, hybrid/own for Perplexity) unless you can show Y is cited
  **despite not ranking** on that index — which needs that index's ranking data.
  Cross-engine directory dominance is therefore **consistent with** "AI inherits
  each index's rankings", not evidence of ranking-free authority. Default GEO
  framing for "best [X]" prompts: AI visibility largely **inherits** search
  rankings across the indices the engines use (GEO ≈ SEO across Google + Bing +
  own), plus directories.

---

## Pre-flight checklist

Before running SISTRIX MCP calls, verify:

0. **Tool names resolved against the announced schema, not from memory.** The
   server exposes view-parameter tools (`domain`, `keyword`, `links`,
   `project`, `project_onpage`, `ai_check`, `ai_tracker`, `ai_tracker_overview`,
   `ai_prompt_answers`, `ai_topicresearch`), so the dataset you want is a
   **tool + `view`** pair. Anything shaped like `domain_visindex`,
   `keyword_seo_metrics` or `project_onpage_urls` is a legacy name — resolve it
   through the name-mapping table. A call built on `ai_entity` or `ai_top` is
   not runnable here — neither was on the announced surface at last check;
   re-map it through the table (`ai_entity` → `ai_check`; `ai_top` has no
   equivalent) rather than issuing it.
1. **Country validated** against the supported set, lowercase, `uk` not `gb`.
   (Wrong codes don't error — they mislead.)
2. **IDs discovered**, not guessed — `project view=overview` / `ai_tracker_overview`
   first; confirm any user-supplied hash and that it matches the right family.
3. **`mobile` not set** (deprecated/ignored).
4. **AI-answer limits kept low** (3–5) to control token cost; prefer count views
   when you don't need answer text.
5. **Response shape inspected** on first call before scripting against it
   (including single-row bare-object unwrapping on `keyword_domain_seo`).
6. **Timeouts retried once**; large batches split into smaller parallel groups.
7. **Credit budget** agreed with the user for broad exploration.
8. **AI-visibility reporting:** branded and discovery prompts separated before
   any share-of-voice claim; `ai_tracker view=prompts` deduped before tallying;
   `brand_found` never presented as visibility (it is a per-question ever-flag,
   and the API carries no execution counts — take the per-answer share from the
   UI route in
   `sistrix-tracking-strategy-builder/references/sistrix-implementation.md`);
   no causal "X drives AI visibility" claim without a divergence case and a
   stated n/confounds. (See "AI visibility — branded vs discovery" and the
   causal-method discipline.)
9. **Historical (`date`) analysis:** undated control call run before trusting
   any before/after diff; dataset (standard vs extended index) named behind
   every number; current rankings cross-checked before any "lost keywords"
   claim reaches a stakeholder. (See the historical forensics recipe.)
10. **Single-domain series read as a business trend:** every domain and host
    that carried the market in the window pulled and combined before the shape
    of any one `domain view=visindex` curve is called a loss, a recovery or
    continuing damage; the structural alternatives (migration, domain move,
    seasonality, country or keyword-set change) named with which were checked,
    in the same message as the observation. (See forensics recipe, step 0.)

A 20-second pass over this list prevents the failure modes that otherwise only
surface after a wasted batch of calls.

---

## Rules that aren't about SISTRIX — the portability test

Four of the rules above are not about SISTRIX at all. They are epistemics for
querying a remote data source you cannot see inside, and they hold for any
tool companion of this kind:

- **Control call before any absence claim** — here: the undated, unfiltered
  `keyword view=seo` / `keyword_domain_seo` control (error taxonomy, historical
  forensics recipe step 4, pre-flight item 9).
- **Inspect the actual response shape before scripting** — here: "Response
  shape — assume nothing", including single-row bare-object unwrapping on
  `keyword_domain_seo` (pre-flight item 5).
- **`filtered == unfiltered` is the silent-no-op tell** — here: `ai_*` view
  parameters accepted and ignored, and a silently mis-served `country` code.
  Its harder sibling: **a filter can be applied in part**, and then the result
  moves, looks filtered, and is wrong. A multi-select whose members pass
  through an encoding layer (the Toolbox sources report's tag filter and
  ampersands, above) can drop some members and keep others, so a filtered
  figure that differs from expectation says the filter was partly ignored at
  least as readily as it says the population is smaller. The generic rule: a
  filtered count is trustworthy only after a **positive control** — add a
  member known to contribute and confirm the total moves by roughly what that
  member carries — and after the filtered and unfiltered totals have been
  reconciled against each other. Distrust the interface's echo of your
  selection; it renders what you chose, not what was queried.
- **Full-payload views are token traps; prefer count and point-date views** —
  here: `domain view=visindex history=true limit=250` (~10k tokens) versus
  point-`date` probes, and `ai_*` answer limits of 3–5 (pre-flight item 4).

A fifth belongs with them: **verify any count against an independent
derivation** — here, naming the dataset (standard vs extended index) behind
every number and validating join non-match semantics before quoting a
percentage.

A sixth: **a metric series records the world and every change to how the
world was measured, and cannot separate the two** — here, one domain's
`domain view=visindex` curve read as a market's fate when the market had moved
domains (forensics recipe, step 0). Name the structural alternatives, run the
control in the same turn, and never report the movement before the check.

Five more of the same family, each stated in its SISTRIX form:

- **An enumeration proves a thing is addressable, not populated or
  current.** Tool schemas and parameter lists are accretions — a documented
  parameter can be retired while still accepted (`mobile`, deprecated and
  ignored), and a listed capability can hold nothing behind it. A zero-data
  probe on an advertised parameter or dataset has three readings — populated
  but not matching, live but empty, or retired — and the API cannot
  distinguish them. Report an unpopulated advertised surface as an
  observation with a question attached ("this returns nothing — still in
  use?"), never as a defect finding; where a working sibling tool covers the
  same concept, retired is the leading hypothesis, not broken.
- **Human-authored metadata is a claim, not behaviour.** Tool names,
  descriptions and schema wording travel with the data and drift from what
  the query actually does — this file already catalogues the local
  instances: `domain view=visindex_overview`'s description promising a
  current value and a breakdown it does not return,
  the onpage tool's `meta` view being project metadata rather than crawl
  data "despite the name",
  `project_onpage view=overview` surfacing notices despite its
  "error/warning totals" description. The same applies to *your own*
  account objects: tracker names, project names and tag labels describe
  intent at authoring time, not current contents. Where a description and
  the returned rows disagree, the rows win — verify any scope taken from a
  name or description against the rows before quoting it.
- **This server's tool descriptions over-promise — so a schema description
  here is a hypothesis to test, never a source to document from.** The rule
  above is stated as a general caution. On this server it is no longer a
  caution but a verified finding, and the direction of the error is
  consistent: three independent checks run against the live server at last
  verification each found the description
  claiming *more* than the API delivers, and in all three the skill's existing
  behaviour-derived claim was right and the metadata was wrong.
  - `domain view=visindex_overview` was described as returning a current value
    and a country/SERP breakdown. It returns neither — only a week count and
    the all-time min/max.
  - `keyword view=searchintent` was described as a classification with shares.
    The four scores summed to 139 on the probe; there are no shares.
  - `ai_tracker`'s `model` parameter offered `'gpt'` and `'gemini'` as
    examples. `gpt` is rejected by the API with error 1002.
  Three for three, all in the same direction. So on this server, write
  documentation from rows and status codes only; treat every description as a
  claim to be checked, and when a description and an observation conflict,
  stop looking for a reading that reconciles them — the description is the
  more likely error.
  - **The test is cheap, which is the point.** One call plus one control
    settled all three in under a minute: the call you intend to make, plus
    either the sibling view that would carry the promised field
    (`visindex` against `visindex_overview`) or the unfiltered baseline
    through the identical call (`model` unset against `model` set). Run it
    before quoting a view you have not personally seen return rows.
  - **Hold the scope.** This is a dated finding about *this* server, from
    three probes on one day. It is not a claim that MCP tool descriptions are
    unreliable in general, and it is not a licence to disregard schema text
    elsewhere. Elsewhere, the general rule above applies at its general
    strength. Re-run the probes before treating this as still true of a much
    later version of the server.
- **The strongest null-result control is a positive control on a
  known-populated scope through the identical call.** An unfiltered control
  (the `keyword view=seo` rule above) proves the data source has content; it
  does not prove *your scope* is loaded — a selectable object (a
  just-created tracker, a project whose crawl finished hours ago) can be
  enumerable and empty, because the registry that lists objects and the
  store that serves their data are different systems with a lag. An empty
  scope and a clean scope render identically, always as good news. Run the
  identical call against a scope known to hold data (an older tracker, a
  previous crawl), and treat any object whose run completed within hours as
  the leading suspect for an unprocessed scope. Read the zeros'
  distribution: zero in every view means the scope is empty; zero in one
  filtered view means the filter matched nothing.
- **Before engineering a workaround for a predicate the API can't express,
  check what the platform precomputes.** SISTRIX computes derived answers
  as first-class objects — the Optimizer's 105-key onpage issue catalogue
  (`project_onpage view=crawl` lists which keys hold rows, with counts), SERP
  features, intent scores — and these are computed inside the platform,
  unbound by the API's filter set. A platform-computed flag carries the
  vendor's definition and is comparable across crawls; an ad-hoc
  client-side reconstruction is neither. Ask "has SISTRIX already answered
  this?" before building around a missing filter — the workaround is the
  interesting route, which is exactly why attention flows to it.

And one about error reading: **a value that passes validation can still
fail at execution, and the two failures carry different information.** The
client-side Pydantic rejection listing valid keys is a cheap discovery
probe; a server-side error (`1007` with no valid values listed) means the
engine refused something the schema accepted — back out rather than retry
variants. When a failure's blast radius exceeds the thing you changed,
suspect your own parameter before the tool: an error larger than its cause
misdirects by default.

And three more, all about reading a response for more than it says:

- **A default you didn't set is still a filter, and it shows up in neither
  the request nor the response.** Omitting a scoping parameter is not "no
  scoping" — it is whatever scope the server picked. On the `ai_*` family,
  leaving `country` unset aggregates across every country rather than
  returning an unscoped truth; `domain view=visindex history=true` returns only
  the ~100 most recent weeks unless `limit` says otherwise, so "the full
  history" is a window nobody asked for; every paginated tool has a `limit`
  you did not choose. Re-reading the call cannot reveal a default (it isn't
  there) and inspecting the result cannot either (it isn't there either) —
  the only detector is to vary the parameter deliberately. Before quoting any
  total or aggregate, run the same call twice with the scoping parameter at
  its extremes and compare; if the figure moves, label every number with the
  scope that produced it. Treat a row count, a `counts` value or an exhausted
  page loop as a report on the scope you **requested**, never the scope
  SISTRIX holds — and treat a per-row date that predates your window as no
  evidence at all about the window's width. Exhaustive-looking retrieval is
  the dangerous case: having paged to the end supplies the feeling of
  completeness that would otherwise prompt the check.
  And even a window you *did* set — or one the tool fixes and states plainly —
  is still an aggregate that destroys recency. This is a distinct failure from
  the unset default above: there the scope was chosen for you; here the scope
  was known, correct and reported, and the number was still wrong to act on. A
  rate answers "how often, across this period" and is read as "how often,
  now", and the two diverge exactly when the condition has already stopped —
  the case where acting on the number is most wasteful. A presence rate
  computed from `ai_tracker view=prompts` `brand_found`, a
  `mentions`/`prompt_count` ratio off `environment` or the `sources_*` views,
  or a VI delta quoted across a quarter can each describe something that ran
  through the first half of the period and ended weeks before the report was
  written: arithmetically correct, operationally wrong, and the action item it
  produces sends someone hunting a live cause that no longer exists. **So
  segment the window before any rate carries a recommendation.** On the SEO
  side this is cheap and already documented here: `domain view=visindex` point-`date`
  probes (or `history=true` / `daily=true`), the `project view=visibilityindex`
  timeline, and the dated `keyword_domain_seo` / `domain view=kwcount` routes
  give the bands directly; where only a since-date is available, nested windows
  differenced against each other cost one call per band (3, 7, 14, 21, 30 days)
  and need no time-series dimension at all. The `ai_*` views are the hard case
  — they expose no window parameter this file has confirmed, so their rates
  cannot be banded: report them as statements about the tracker's own period,
  never as "this is happening now", and take the recency claim from a surface
  that can be dated. Two things belong in any deliverable that does band: which
  band the finding actually lives in, and whether it is still occurring in the
  most recent one. The bands also recover the mechanism the aggregate destroyed
  — a component present in the older bands and exactly zero in every band of 14
  days or less names both the cause and roughly when it stopped. Two riders:
  check the denominator, because a dramatic percentage over a handful of
  prompts or keywords is not a finding (and dedupe `view=prompts` before
  computing any presence rate); and where a UI attaches a headline rate to a
  named keyword, URL or prompt, read it as scoped to that row until proven
  otherwise, not as a domain total that happens to be displayed beside an
  example.
- **Test an identifier before using it as a join key across two calls.** A
  field shaped like a key is an implicit promise about persistence, and the
  promise is untested until the same scope is fetched twice and the values
  compared. Here the only identifiers with confirmed durability are the
  project and tracker `hash` values, and even those live in separate spaces
  (see Mental model); row-level identity is weaker than it looks —
  `ai_tracker view=prompts` can return the same prompt several times in one
  response with different tags and `brand_found`, so row position and row
  count are not identity at all. Key any before/after comparison on a
  **natural key** — the keyword string, the prompt string, `domain`,
  `url.from`/`url.to` — deduplicate first, and diff the substantive fields
  explicitly rather than inferring change from set membership. An unstable
  key manufactures a difference in exactly the operation used to detect
  difference, so the instrument's noise arrives wearing the costume of the
  signal.
- **A mention figure or a brand list attached to a source row describes the AI
  answers, not the source page.** The `sources_domains` / `sources_urls` /
  `sources_hosts` views of `ai_tracker` return `{url | domain | host,
  prompt_count, mentions, brands}`: both `mentions` and `brands` are computed
  over the *answers* that cited the source, and both are
  silent about what the cited page contains. Read `ai_check view=sources`
  the same way unless its payload proves otherwise. A cited URL that shows no brand mention is therefore not evidence
  the brand is absent from that page, which is exactly the inference a
  citation-gap or outreach list wants to make — and the converse fails too:
  a URL whose `brands` carries the brand token is not evidence the brand is
  present on that page (3 of 12 such URLs on one measured pull did not
  mention it anywhere). Treat any target list derived
  from such a field as a hypothesis about the source pages until a sample is
  fetched and checked — one static fetch per URL, which is also the bytes an
  AI crawler sees. The fetch separates two findings with nothing in common:
  brand absent from the page, and brand present but not selected, or present
  only under a legal-entity or alternative name — a coverage problem and an
  authority/entity-disambiguation problem, calling for opposite
  interventions. The sample's disagreement rate is the list's error rate.
- **The semantics of a source row's brand field differ by platform, and
  cannot be inferred from the field's shape or name.** Two citation platforms
  measured here expose a per-source-row brand array that looks identical in
  the payload and means opposite things. One is **page-level and
  substring-matched** — it records the tracked brands whose name or alias
  appears in the cited page's own content, confirmed by a positive and a
  negative control on live data.
  SISTRIX's `brands` is **answer-level and a raw token dump**, as above. So
  no such field is a presence test until a fetch has confirmed its polarity
  on the project in hand. The disproof is cheap and works for any tool:
  **fetch a page you know does *not* mention the brand but whose citing
  answers do, and see which way the field falls.** The portable half of this:
  an undocumented field that arrives shaped like the answer to an expensive
  question will be read as that answer, so find out which question it
  answers before it drives spend.

The test when adding any new rule to this file: could the sentence survive
having "SISTRIX" removed? If yes, it is a rule about querying any opaque data
source — keep it phrased generically, so it can be lifted into whatever
companion skill documents the next tool.
