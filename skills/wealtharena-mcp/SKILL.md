---
name: wealtharena-mcp
description: "Use Wealth Arena Model Context Protocol (MCP) tools for Indian financial markets (NSE/BSE): live quotes, option chains with Greeks, instrument search, expiry dates, account tier details, and credit quota usage."
---

# Wealth Arena MCP Tools — Agent Operating Guide

The Wealth Arena Model Context Protocol (MCP) server provides autonomous AI agents (Claude, Cursor, Codex, Gemini CLI, Antigravity) with structured, real-time access to Indian financial markets (NSE & BSE equities, indices, and derivatives).

---

## 1. Core Operating Principles

1. **Fail-Honest Mandate (Zero Synthetic Reconstruction)**:
   - Synthesizing, interpolating, or hallucinating missing financial metrics, option strikes, Greeks, or market quotes is strictly prohibited.
   - If upstream broker connectivity, exchange feeds, or contract data are unavailable or empty, Wealth Arena tools report honest empty states or error notices.
   - Always present these states verbatim to the user; never attempt to fabricate plausible placeholder values.

2. **Token Efficiency (GFM Markdown Contract)**:
   - All tool responses return compact, token-dense GitHub Flavored Markdown (GFM) tables, cards, and key-value ribbons.
   - Eliminates 60%–80% of token overhead compared to raw JSON envelopes while preserving full numeric fidelity.

3. **Compute Credit Economy & Quota Awareness**:
   - Wealth Arena accounts operate on tiered monthly compute credit quotas.
   - Control plane queries (liveness, session, quota, plan) are **0 credits** (free).
   - Lightweight market lookups consume **1 credit**.
   - Heavy analytical computations (full option chain snapshots with Greeks) consume **5 credits**.
   - Proactively monitor remaining balance via `get_wa_usage_status` when running multi-step analyses.

---

## 2. Complete MCP Tool Catalog

| Tool Name | Title & Purpose | Credit Weight | Required Arguments | Optional Arguments |
|:---|:---|:---:|:---|:---|
| **`get_wa_status`** | Connection health, broker session, and platform liveness | **0 Credits** | *None* | *None* |
| **`get_wa_plan`** | Active subscription plan tier, monthly quota, and rate limits | **0 Credits** | *None* | *None* |
| **`get_wa_usage_status`** | Current credit consumption, remaining balance, cycle renewal date, and daily pacing budget | **0 Credits** | *None* | *None* |
| **`get_wa_market_quote`** | Real-time quote (LTP, change, volume, prev close) for a single instrument | **1 Credit** | `symbol` *(string)* | *None* |
| **`search_wa_instruments`** | Search tradable stocks, indices, and derivatives by name or keyword | **1 Credit** | `query` *(string)* | `limit` *(int, default 10)* |
| **`get_wa_expiries`** | Active option and futures expiry calendar for an underlying | **1 Credit** | `underlying` *(string)* | *None* |
| **`get_wa_chain_snapshot`** | High-density option chain with spot, strikes, OI, PCR, Max Pain, and Greeks (Delta, Gamma, Theta, Vega) | **5 Credits** | `symbol` *(string)* | `strikes` *(int, default 10)*, `expiry` *(YYYY-MM-DD)* |

### Feature-Gated Tools (Coming Soon)
The following execution and risk management tools are currently gated behind backend feature flags (`kill:mcp-tool-*`) and return an honest Coming Soon indicator:
- `wa_broker_status`: Multi-broker connection matrix and session telemetry.
- `wa_paper_margin`: Virtual simulated paper trading margin and cash balance.
- `wa_live_margin`: Live broker margin and collateral status.
- `wa_halt_status`: Emergency platform trading halt circuit breaker.

---

## 3. Symbol Notation & Formatting Rules

Indian equity and derivatives markets follow strict identifier conventions:

### Index Underlyings
Always use the broker-prefixed index identifier:
- **NIFTY 50**: `"NSE:NIFTY50-INDEX"`
- **BANK NIFTY**: `"NSE:BANKNIFTY-INDEX"`
- **FINNIFTY**: `"NSE:FINNIFTY-INDEX"`
- **MIDCAP NIFTY**: `"NSE:MIDCPNIFTY-INDEX"`

### Cash Equities
Format as `"NSE:<SYMBOL>-EQ"`:
- Reliance Industries: `"NSE:RELIANCE-EQ"`
- Tata Consultancy Services: `"NSE:TCS-EQ"`
- HDFC Bank: `"NSE:HDFCBANK-EQ"`
- State Bank of India: `"NSE:SBIN-EQ"`

### Expiry Date Formatting
- Always pass dates in standard ISO `YYYY-MM-DD` format (Indian Standard Time, IST).
- Example: `"2026-11-26"`.
- If the exact weekly or monthly expiry date is unknown, query `get_wa_expiries` before guessing.

### Instrument Discovery
If the user mentions an unfamiliar company or ticker, call `search_wa_instruments(query: "...")` first to resolve the exact canonical symbol.

---

## 4. Multi-Turn Agent Workflow Recipes

### Recipe 1: Pre-Flight Health & Quota Verification
Before executing multiple market queries, verify platform reachability and remaining credit quota:
1. Call `get_wa_status()` (0 credits) to confirm Wealth Arena backend is connected and healthy.
2. Call `get_wa_usage_status()` (0 credits) to inspect remaining monthly credits and daily pacing budget.

### Recipe 2: Single Spot Price / Quote Lookup
When the user asks for the current price or performance of a stock or index:
```json
// Call get_wa_market_quote
{
  "symbol": "NSE:NIFTY50-INDEX"
}
```
*Output: Compact GFM card containing Last Traded Price (LTP), net change, percentage change, trading volume, and previous close.*

### Recipe 3: Complete Option Chain & Greeks Analysis
When the user requests option chain analysis, Put-Call Ratio (PCR), Max Pain, or Greek positioning:
1. **Discover Available Expiries**:
   ```json
   // Call get_wa_expiries
   {
     "underlying": "NSE:NIFTY50-INDEX"
   }
   ```
2. **Fetch Chain Snapshot**:
   Select the desired expiry (usually near-week or monthly) and request a focused strike window around ATM:
   ```json
   // Call get_wa_chain_snapshot
   {
     "symbol": "NSE:NIFTY50-INDEX",
     "strikes": 10,
     "expiry": "2026-11-26"
   }
   ```
3. **Interpret Output**:
   - The response includes spot price, overall Put-Call Ratio (PCR), Max Pain strike, and a structured strike table.
   - For each strike: Call OI, Call Delta/Gamma/Theta/Vega, Call LTP, Strike Price, Put LTP, Put Delta/Gamma/Theta/Vega, and Put OI.
   - Use strike concentration and PCR to evaluate bullish/bearish dealer positioning.

### Recipe 4: Resolving Unfamiliar Symbols
When a user refers to an ambiguous company or derivative:
```json
// Call search_wa_instruments
{
  "query": "TATAMOTORS",
  "limit": 5
}
```
Select the corresponding `NSE:TATAMOTORS-EQ` or derivative contract identifier.

---

## 5. Market Hours & Operating Boundaries

- **Regular Trading Hours**: Monday through Friday, **09:15 to 15:30 IST** (excluding exchange holidays).
- **Pre-Open Market Session**: **09:00 to 09:15 IST**.
- **Platform Operating Window**: **06:30 to 23:30 IST daily**.
- **Off-Hours Queries**: During market off-hours or weekends, requests return the last finalized closing session prices and settled option chain data. The server notes off-market status honestly.

---

## 6. Client Configuration

### Remote Streamable HTTP / SSE MCP (Recommended for Cloud & Web Agents)
- **Endpoint URL**: `https://mcp.wealtharena.in/mcp`
- **Authentication**:
  - OAuth 2.1 PKCE (Claude native connector, OAuth authorization endpoint: `https://dev.wealtharena.in/oauth/authorize`).
  - Or direct Bearer token / header secret: `Authorization: Bearer <WA_ACCESS_TOKEN>`.

### Local Stdio MCP (Cursor, Claude Desktop, Windsurf)
Add to your client configuration file (`claude_desktop_config.json` or `cursor.json`):
```json
{
  "mcpServers": {
    "wealtharena": {
      "command": "npx",
      "args": ["-y", "wealtharena-cli", "mcp"],
      "env": {
        "WA_ACCESS_TOKEN": "<YOUR_ACCESS_TOKEN>"
      }
    }
  }
}
```
*(Or authenticate once via terminal `wac login`; the stdio server automatically reads your stored credentials).*
