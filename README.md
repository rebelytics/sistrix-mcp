# sistrix-mcp

Open-source **companion skill for the SISTRIX MCP server** — SEO and AI-visibility data via the SISTRIX API. Load it into any MCP-capable AI agent (Claude, Cursor, Codex, n8n, etc.) alongside a connected SISTRIX MCP.

## What this skill does

A field guide to how the SISTRIX MCP tools actually behave — parameter rules, inconsistent response shapes, error patterns and token-cost traps — so a session using the MCP is structured and avoids the common mistakes. It does not replace the MCP's own tool descriptions; it adds the operational knowledge those descriptions omit: the mental model of the tool families, universal parameter rules, a tool-family reference with a legacy-name mapping table, response-shape rules, an error taxonomy, workflows and analysis recipes, the branded-vs-discovery rule for AI-visibility reporting, and a pre-flight checklist.

## When it triggers

Working with SISTRIX through an MCP — Visibility Index, keyword and ranking data, backlinks, competitor sets, domain traffic estimates, AI-visibility data (AI Overviews / AI Mode / ChatGPT / Perplexity mentions, sources and trackers), Optimizer project work, and any multi-step SISTRIX analysis or cross-market comparison. Triggers on SISTRIX, Sichtbarkeitsindex, Visibility Index, "sistrix mcp", AI tracker, the `domain` / `keyword` / `links` / `project` / `ai_*` tools and their `view` parameters, and the older `domain_*` / `keyword_*` / `ai_entity` / `ai_top` names.

## What this repo contains

- `SKILL.md` — the mental model, the universal parameter rules, the pre-flight checklist, the portability test, and a section map saying when to load each reference file.
- `references/tool-surface.md` — the tool-family reference (§3): the legacy-name mapping table and the `domain` / `keyword` / `links` / `ai_*` / `project` families with their views and parameter rules.
- `references/response-shapes-and-errors.md` — response-shape rules (§4) and the error taxonomy incl. credits (§5).
- `references/recipes.md` — workflows (§6) and analysis recipes (§7).
- `references/ai-visibility.md` — the branded-vs-discovery rule for AI-visibility reporting (§8).
- `LICENSE` — CC BY 4.0.

## Install

Install the whole skill directory — `SKILL.md` and `references/` must travel together — into your MCP-capable agent's skills directory, following the client-specific path, and restart the client. Tool names in the skill are written without the MCP server prefix; match by the suffix.

## Related

[`sistrix-tracking-strategy-builder`](https://github.com/rebelytics/sistrix-tracking-strategy-builder) builds an AI visibility tracking strategy on SISTRIX's custom prompt tracking and uses this skill for the read side.

## Contributing

Open an issue or a PR at [github.com/rebelytics/sistrix-mcp](https://github.com/rebelytics/sistrix-mcp). Real-world breakage is the best signal for improving the guide.

## Credits

- Original author: [Eoghan Henn](https://www.rebelytics.com) / [LinkedIn](https://www.linkedin.com/in/eoghanhenn)
- Not affiliated with SISTRIX.

## License

CC BY 4.0 — see [LICENSE](./LICENSE). Use it, fork it, adapt it, monetise it. Keep the attribution.
