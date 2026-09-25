---
name: "sistrix-mcp"
description: "Companion skill for the SISTRIX MCP server (SEO + AI-visibility data via the SISTRIX API). Load this whenever the user works with SISTRIX through an MCP — pulling Visibility Index, keyword/ranking data, backlinks, competitor sets, domain traffic estimates, or AI Overviews / AI Mode / ChatGPT / Perplexity mentions, sources and trackers. Also load for SISTRIX Optimizer project work (project rankings, onpage crawls) and any cross-market comparison or report built on SISTRIX data. Trigger on mentions of Sichtbarkeitsindex, ai_check/ai_tracker, the domain/keyword/links/project tools and their view parameters, or older ai_entity/ai_top/domain_/keyword_ names — even when the tools aren't named but SISTRIX data is clearly needed. Teaches the real behaviour of the MCP, including response-shape quirks and the gotchas tool descriptions omit."
version: 1.5.0
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

## 1. Mental model

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

## 2. Universal parameter rules

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

## Reference files — load on demand

This skill uses progressive disclosure. This file holds what applies on every
invocation — the mental model, the universal parameter rules, the pre-flight
checklist and the portability rules. The tool surface, response shapes,
workflows and reporting rules live in `references/` and are read when their
episode comes up. The load triggers are mandatory, not suggestions: calling a
tool without having read the tool-surface file, or reporting an AI-visibility
number without the branded-vs-discovery file, is how the confidently-wrong
outputs this skill exists to prevent get produced. If you notice an episode
was handled without its reference loaded, treat that as a failure to report,
not a shortcut that worked.

- `references/tool-surface.md` (§3) — **read before the first SISTRIX tool
  call of a session**, and whenever a tool name, view, parameter or field has
  to be resolved: the name-mapping table for legacy names, every view per
  family, filter gotchas, per-view response keys, the `ai_*` model vocabulary
  and the token-cost trap on answer-text views.
- `references/response-shapes-and-errors.md` (§4–§5) — **read before
  scripting against any response, and on any error, timeout or empty
  result**: observed top-level keys, single-row unwrapping, the error-code
  table with the retry/no-retry decision per code, and credits.
- `references/recipes.md` (§6–§7) — **read when the task combines tools into
  an analysis**: domain visibility snapshot, keyword research, AI-visibility
  audit, Optimizer monitoring, visibility-drop diagnosis and update
  post-mortems, cross-domain attribution, stress-testing a third-party
  report, "best [X]" AI-inclusion diagnosis, and the causal-method
  discipline that governs all of them.
- `references/ai-visibility.md` (§8) — **read before reporting any AI
  visibility, share-of-voice, competitor or cited-source number** off
  `ai_tracker` or `ai_check`: the branded-vs-discovery split and the
  source-view contamination rules.

## Section map — where each § lives

Section numbers are global across the file set, so a heading referred to by
name or number anywhere — in this skill, in a companion skill, or in older
notes — resolves through this table. Cross-references written before the
split ("see the Domain section", "the rule below", "the `ai_tracker` note")
resolve here by heading; "the principles section" means §10.

| § | Heading | File |
|---|---|---|
| 1 | Mental model | `SKILL.md` |
| 2 | Universal parameter rules | `SKILL.md` |
| 3 | Tool-family reference | `references/tool-surface.md` |
| 3.1 | Name-mapping table (legacy name → live tool + view) | `references/tool-surface.md` |
| 3.2 | Domain (`domain`) | `references/tool-surface.md` |
| 3.3 | Keyword (`keyword`) — incl. "Coverage is not demand" and the lead-variant rule (one string ≠ the concept's demand) | `references/tool-surface.md` |
| 3.4 | Links (`links`) — domain only, no country | `references/tool-surface.md` |
| 3.5 | AI visibility (`ai_*`) — incl. the `ai_tracker` note (no execution counts, no country/language in `view=prompts`), `model` vocabulary, token-cost trap | `references/tool-surface.md` |
| 3.6 | Project / Optimizer (`project`) — incl. onpage crawl views | `references/tool-surface.md` |
| 4 | Response shape — assume nothing | `references/response-shapes-and-errors.md` |
| 5 | Error taxonomy & resilience — incl. credits | `references/response-shapes-and-errors.md` |
| 6 | Workflows | `references/recipes.md` |
| 6.1 | Domain visibility snapshot (any market) | `references/recipes.md` |
| 6.2 | Keyword research | `references/recipes.md` |
| 6.3 | AI-visibility audit | `references/recipes.md` |
| 6.4 | Optimizer project monitoring | `references/recipes.md` |
| 7 | Analysis recipes — incl. visibility-drop diagnosis, striking distance, cross-market, AI × classic bridge, competitor gap, backlink scan, sensitive-query audits | `references/recipes.md` |
| 7.1 | Recipe: Stress-test a third-party report / URL-pattern claim | `references/recipes.md` |
| 7.2 | Recipe: Historical visibility-loss forensics (update post-mortems) | `references/recipes.md` |
| 7.3 | Recipe: Cross-domain natural experiment (update attribution across a portfolio) | `references/recipes.md` |
| 7.4 | Recipe: "Why doesn't our directory rank convert to AI visibility?" | `references/recipes.md` |
| 7.5 | Causal-method discipline for GEO/AI-visibility diagnosis | `references/recipes.md` |
| 8 | AI visibility — branded vs discovery (read before reporting) | `references/ai-visibility.md` |
| 9 | Pre-flight checklist | `SKILL.md` |
| 10 | Rules that aren't about SISTRIX — the portability test (the "principles section") | `SKILL.md` |

---

## 9. Pre-flight checklist

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

## 10. Rules that aren't about SISTRIX — the portability test

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

A seventh: **when a data source attributes an aggregate to one
representative member, a low reading on any other member says nothing
about the aggregate** — here: `keyword view=traffic` volume sitting on one
lead variant of a term cluster (singular, plural or qualified form, not
predictably which) while the siblings read 10 (§3.3, "Coverage is not
demand"). Query the plausible representatives and take the max before
reporting absence, and name the member that carried it.

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
