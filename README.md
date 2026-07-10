# RiskModels — Claude Code plugin

Institutional equity risk and **PIT-normalized fundamentals derived from SEC filings and
licensed sources**, as a data layer for Claude agents. This plugin bundles the hosted
RiskModels MCP with a set of thin skills, an analyst agent, and a worked cookbook so
Claude can decompose a US stock or portfolio into market / sector / subsector /
stock-specific risk, produce tradeable ETF hedge ratios, pull point-in-time quarterly
fundamentals with a CAPM cost-of-capital layer, and rank names on factor and residual
risk — all from live data, from **$0.005/call**, with no enterprise data license.

Realized/historical data only. RiskModels is an analytical tool, **not an investment
adviser** — see [What this is not](#what-this-is-not).

## What you get

- **5 skills** (thin wrappers over live endpoints, with example prompts):
  `risk-decompose`, `fundamentals-pit`, `cost-of-capital`, `portfolio-hedge`,
  `residual-screen`.
- **An analyst agent** — `riskmodels-analyst`, the institutional risk-analyst persona
  that fans the tool calls out and leads with the answer.
- **A [cookbook](./cookbook.md)** — one name, end to end: fundamentals → WACC →
  decomposition → hedge → residual signal.
- **The hosted MCP** wired in — no separate connection step.

## 1. Get an API key

The plugin talks to the hosted RiskModels MCP, which authenticates per user. Claude Code
plugins have no key-entry prompt, so set the key in your environment **before** installing:

```bash
export RISKMODELS_API_KEY="rm_agent_live_..."
```

Get a key (free tier available; usage billed per call) at
**https://riskmodels.app/get-key**. Install is free — you pay only for the calls you make.

## 2. Install

```
/plugin marketplace add BlueWaterCorp/riskmodels-plugin
/plugin install riskmodels@riskmodels
```

Then, in a fresh session:

```
/risk-decompose NVDA
/fundamentals-pit AAPL
/cost-of-capital MSFT 0.05 10y
```

or just ask the `riskmodels-analyst` agent a question in plain English.

## Already using `@riskmodels/mcp`?

The [`@riskmodels/mcp`](https://www.npmjs.com/package/@riskmodels/mcp) npm package is the
**raw connection** — it registers the RiskModels tools on your MCP client and nothing more.
This plugin is the **workflow layer on top**: the same tools, plus the skills, the analyst
agent, and the cookbook that tell Claude *how* to chain them into an analysis.

They are not competing installs, and you do not need both connections. If you already
added the MCP via `claude mcp add … @riskmodels/mcp`, you can still install this plugin for
the skills and agent — just **skip the plugin's bundled MCP** to avoid a duplicate server
(disable it in the plugin, or remove your standalone `riskmodels` MCP entry and let the
plugin provide it). One connection, one key.

## Skills

| Skill | Use it for | Wraps |
| --- | --- | --- |
| `risk-decompose` | What's driving a name's risk; L1/L2/L3 cascade + ETF hedge legs | `riskmodels_get_hedge_levels`, `riskmodels_decompose`, `get_metrics` |
| `fundamentals-pit` | Point-in-time quarterly fundamentals with `as_of` anti-look-ahead | `riskmodels_get_fundamentals` |
| `cost-of-capital` | CAPM cost of equity, WACC, economic profit; caller ERP + rf-tenor grid | `riskmodels_get_fundamentals` |
| `portfolio-hedge` | ETF hedge legs for a position or book; Lstar residual isolation | `riskmodels_hedge_portfolio`, `riskmodels_hedge_position`, `riskmodels_get_lstar` |
| `residual-screen` | Peer/universe rankings and the residual mean-reversion signal | `riskmodels_get_rankings`, `riskmodels_screen_rankings`, `riskmodels_get_residual_signal` |

Skills are deliberately thin — endpoint wrappers plus prompts. The canonical semantics
(field meanings, units, PIT rules) live in the served API contract, not in this repo.

## What this is not

- **Not investment advice.** Skills and the agent report model outputs — decomposition,
  hedge ratios, ranks, cost of capital — never recommendations, price targets, or
  suitability assessments. No forecasts.
- **Not a full raw-fundamentals feed.** Raw line items appear per cell only where the
  serving value is SEC XBRL (`sec_facts`); other cells are derived. The panel is not "raw
  fundamentals," and nothing here is "IP-free."
- **Not local logic.** Every number comes from a live tool call; the plugin computes
  nothing itself, so it cannot drift from the served contract.

## License

Apache-2.0. This repository is install glue — configuration, prompts, and docs. It carries
no proprietary model logic.
