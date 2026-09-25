# Tool-family reference (§3)

Part of the **sistrix-mcp** skill (CC BY 4.0 — Eoghan Henn / [rebelytics.com](https://rebelytics.com)). Section numbers are global across `SKILL.md` and `references/` — the section map in `SKILL.md` says where each § lives.

**Load trigger:** Read before the first SISTRIX tool call of a session, and whenever a tool name, view, parameter or field has to be resolved — especially from older notes, scripts or deliverables that use the one-tool-per-dataset names.

## Contents

- 3. Tool-family reference
  - 3.1 Name-mapping table (legacy name → live tool + view)
  - 3.2 Domain (`domain`)
  - 3.3 Keyword (`keyword`)
  - 3.4 Links (`links`) — domain only, no country
  - 3.5 AI visibility (`ai_*`)
  - 3.6 Project / Optimizer (`project`)

---

## 3. Tool-family reference

### 3.1 Name-mapping table (legacy name → live tool + view)

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

### 3.2 Domain (`domain`)

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

### 3.3 Keyword (`keyword`)

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

**Coverage is not demand — three readings, two for the `keyword` family and
one for its AI sibling.**

- **A single `keyword view=traffic` reading is a statement about that exact
  string, not about the concept it names.** SISTRIX appears to put a
  cluster's volume on one lead variant and leave the siblings near the floor
  (10), and which form leads is not predictable — sometimes the plural,
  sometimes the singular, sometimes a qualified form. The shape it takes,
  one country, one day (illustrative figures):

  | First string tried | Volume | Sibling variant | Volume |
  |---|---|---|---|
  | standing desk | 30 | standing desks | **2,550** |
  | hiking boots | 10 | hiking boot | **1,300** |
  | espresso machine | 350 | home espresso machine | **1,350** |

  Reading one variant is a coin toss that mostly comes up "no demand", and
  the failure is silent: a four-figure concept gets listed as weak on the
  strength of a two-figure sibling. **The rule:** before
  calling a concept low-demand, query at least the singular, the plural and
  the common qualified form (the head term plus its usual category or
  market qualifier), take the **max** as the concept's demand, and report
  **which variant carried it** — the lead string is also the one to target.
  Read a 10 as "this string isn't the lead", never as "nobody searches
  this". This is a separate failure from the Error-1000 reading below: the
  string is in the database and returns a number; the number is simply not
  the concept's.
- **`Error (1000): no result` on a volume/traffic call for a term with
  obvious demand means the keyword is not in that country's database yet,
  not that nobody searches it.** The keyword index is built from what has
  already been crawled and clustered, so a term from a category the market
  only recently acquired a vocabulary for can be absent in one country while
  carrying solid volume in another. Two shapes: a product category only a few
  months old, whose terms return Error 1000 in one market while the same
  strings carry four- and five-figure volumes in another; and a specialist
  term that returns Error 1000 in one country's language while its direct
  equivalent carries volume in a neighbouring market — with the site's own
  search-console data showing real clicks in the language that returned
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

### 3.4 Links (`links`) — domain only, no country

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

### 3.5 AI visibility (`ai_*`)

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
  - **`view=prompts` carries no country and no language, so an API re-read
    cannot verify a tracker's market settings.** The row is `{prompt,
    brand_found, avg_pos, tags, models}` and nothing else; country and
    language are set in the UI at upload and never surface in the payload.
    A post-upload re-read therefore confirms prompt text and tags — the
    fields the CSV carried — and is structurally blind to the one setting
    the CSV does not carry. The consequence when it goes wrong: prompt
    sets run under the account-default country and language, measuring
    the wrong market, while the API verification passes. Verify market
    settings in the UI by-prompts view (flag and language column per
    prompt); the route is in
    `sistrix-tracking-strategy-builder/references/sistrix-implementation.md`
    §12. Generic form: a verification step can only confirm the fields its
    read returns — when the write has settings the read cannot see, add a
    channel that can see them, or the check certifies the part that was
    never at risk.
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

### 3.6 Project / Optimizer (`project`)

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

