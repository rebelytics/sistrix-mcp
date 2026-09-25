# Workflows and analysis recipes (§6–§7)

Part of the **sistrix-mcp** skill (CC BY 4.0 — Eoghan Henn / [rebelytics.com](https://rebelytics.com)). Section numbers are global across `SKILL.md` and `references/` — the section map in `SKILL.md` says where each § lives.

**Load trigger:** Read when the task combines tools into an analysis — a domain visibility snapshot, keyword research, an AI-visibility audit, Optimizer project monitoring, a visibility-drop or update post-mortem, cross-domain attribution, stress-testing a third-party report, or diagnosing why a directory rank does not convert to AI visibility.

## Contents

- 6. Workflows
  - 6.1 Domain visibility snapshot (any market)
  - 6.2 Keyword research
  - 6.3 AI-visibility audit
  - 6.4 Optimizer project monitoring
- 7. Analysis recipes
  - 7.1 Recipe: Stress-test a third-party report / URL-pattern claim
  - 7.2 Recipe: Historical visibility-loss forensics (update post-mortems)
  - 7.3 Recipe: Cross-domain natural experiment (update attribution across a portfolio)
  - 7.4 Recipe: "Why doesn't our directory rank convert to AI visibility?"
  - 7.5 Causal-method discipline for GEO/AI-visibility diagnosis

---

## 6. Workflows

### 6.1 Domain visibility snapshot (any market)
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

### 6.2 Keyword research
`keyword view=metrics` (volume/CPC/competition/device split) →
`keyword view=searchintent` (intent mix, scored 0–100 each) →
`keyword view=serpfeatures` (what the SERP looks like) →
`keyword view=seo` (who ranks). Use `keyword view=traffic show_all_countries=true` to
find the strongest market for a term.

### 6.3 AI-visibility audit
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

### 6.4 Optimizer project monitoring
`project view=overview` → `project view=visibilityindex` (trend) → `project view=ranking`
(movements; remember the nested shape) → `project view=competitors`. Attempt onpage
(`project_onpage view=overview`) only after confirming the project runs crawls.

---

## 7. Analysis recipes

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

### 7.1 Recipe: Stress-test a third-party report / URL-pattern claim

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

### 7.2 Recipe: Historical visibility-loss forensics (update post-mortems)

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

### 7.3 Recipe: Cross-domain natural experiment (update attribution across a portfolio)

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

### 7.4 Recipe: "Why doesn't our directory rank convert to AI visibility?"

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

### 7.5 Causal-method discipline for GEO/AI-visibility diagnosis

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

