# sistrix-mcp

Open-source **companion skill for the SISTRIX MCP server** — SEO and AI-visibility data via the SISTRIX API. Load it into any MCP-capable AI agent (Claude, Cursor, Codex, n8n, etc.) alongside a connected SISTRIX MCP.

## What this skill does

A field guide to how the SISTRIX MCP tools actually behave — parameter rules, inconsistent response shapes, error patterns and token-cost traps — so a session using the MCP is structured and avoids the common mistakes. It does not replace the MCP's own tool descriptions; it adds the operational knowledge those descriptions omit: the mental model of the tool families, universal parameter rules, a tool-family reference with a legacy-name mapping table, response-shape rules, an error taxonomy, workflows and analysis recipes, the branded-vs-discovery rule for AI-visibility reporting, and a pre-flight checklist.

## When it triggers

Working with SISTRIX through an MCP — Visibility Index, keyword and ranking data, backlinks, competitor sets, domain traffic estimates, AI-visibility data (AI Overviews / AI Mode / ChatGPT / Perplexity mentions, sources and trackers), Optimizer project work, and any multi-step SISTRIX analysis or cross-market comparison. Triggers on SISTRIX, Sichtbarkeitsindex, Visibility Index, "sistrix mcp", AI tracker, the `domain` / `keyword` / `links` / `project` / `ai_*` tools and their `view` parameters, and the older `domain_*` / `keyword_*` / `ai_entity` / `ai_top` names.

## What this repo contains

- `SKILL.md` — the whole guide, in one file.
- `LICENSE` — CC BY 4.0.

## Install

Copy the skill directory into your MCP-capable agent's skills directory, following the client-specific path, and restart the client. Tool names in the skill are written without the MCP server prefix; match by the suffix.

## Related

[`sistrix-tracking-strategy-builder`](https://github.com/rebelytics/sistrix-tracking-strategy-builder) builds an AI visibility tracking strategy on SISTRIX's custom prompt tracking and uses this skill for the read side.

## Contributing

Open an issue or a PR at [github.com/rebelytics/sistrix-mcp](https://github.com/rebelytics/sistrix-mcp). Real-world breakage is the best signal for improving the guide.

## Credits

- Original author: [Eoghan Henn](https://www.rebelytics.com) / [LinkedIn](https://www.linkedin.com/in/eoghanhenn)
- Not affiliated with SISTRIX.

## License

CC BY 4.0 — see [LICENSE](./LICENSE). Use it, fork it, adapt it, monetise it. Keep the attribution.
