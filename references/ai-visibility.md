# AI visibility — branded vs discovery (§8)

Part of the **sistrix-mcp** skill (CC BY 4.0 — Eoghan Henn / [rebelytics.com](https://rebelytics.com)). Section numbers are global across `SKILL.md` and `references/` — the section map in `SKILL.md` says where each § lives.

**Load trigger:** Read before reporting any AI visibility, share-of-voice, competitor or cited-source number taken from `ai_tracker` or `ai_check` — the branded-vs-discovery split is a hard rule, and the source views carry the same contamination.

## Contents

- 8. AI visibility — branded vs discovery (read before reporting)

---

## 8. AI visibility — branded vs discovery (read before reporting)

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

