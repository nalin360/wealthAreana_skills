---
name: wealtharena-mcp
description: "Use Wealth Arena Model Context Protocol (MCP) tools for Indian financial markets (NSE/BSE): live quotes, historical OHLCV candlestick data, multi-timeframe technical analysis, institutional FII/DII/Pro/Client positioning & Volume Profile (POC/VAH/VAL), option chains with Greeks and historical replay, Call vs Put OI buildup, instrument search, expiry dates, account tier details, and credit quota usage."
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
   - Lightweight market lookups consume **1 credit** (`get_wa_market_quote`, `search_wa_instruments`, `get_wa_expiries`).
   - Analytical & historical candlestick computations consume **5 credits** (`get_wa_chain_snapshot`, `get_wa_historical_chart_data`, `get_wa_technical_analysis`, `get_wa_institutional_positioning`).
   - Bulk Call vs Put OI rollups consume **10 credits** (`get_wa_oi_buildup`).
   - Proactively monitor remaining balance via `get_wa_usage_status` when running multi-step analyses.

---

## 2. Complete MCP Tool Catalog

| Tool Name | Title & Purpose | Credit Weight | Required Arguments | Optional Arguments |
|:---|:---|:---:|:---|:---|
| **`get_wa_status`** | Connection health, broker session, and platform liveness | **0 Credits** | *None* | *None* |
| **`get_wa_plan`** | Active subscription plan tier, monthly quota, and rate limits | **0 Credits** | *None* | *None* |
| **`get_wa_usage_status`** | Current credit consumption, remaining balance, cycle renewal date, and daily pacing budget | **0 Credits** | *None* | *None* |
| **`get_wa_market_quote`** | Real-time quote (LTP, change, volume, prev close) for a single instrument | **1 Credit** | `symbol` *(string)* | *None* |
| **`search_wa_instruments`** | Search tradable stocks, indices, and derivatives by name or keyword | **1 Credit** | `query` or `underlying` *(string)* | `instrument_type` *(exact: `EQ` \| `INDEX` \| `FUTIDX` \| `FUTSTK` \| `OPTIDX` \| `OPTSTK` \| `COMMODITY` \| `All`)*, `limit` *(int, default 15, max 50)* |
| **`get_wa_expiries`** | Active option and futures expiry calendar for an underlying | **1 Credit** | `underlying` *(string)* | *None* |
| **`get_wa_historical_chart_data`** | Historical or intraday OHLCV candlestick bars (1m, 3m, 5m, 15m, 30m, 1h, 2h, 4h, 1D) with session range and volume | **5 Credits** | `symbol` *(string)* | `resolution` *(string, default "15m")*, `date` *(YYYY-MM-DD, omit for latest session)*, `from` *(string)*, `to` *(string)*, `limit` *(int, max 300)* |
| **`get_wa_chain_snapshot`** | High-density option chain with spot, strikes, OI, PCR, Max Pain, and Greeks (Delta, Gamma, Theta, Vega); pass `date`/`time` to replay a stored historical chain | **5 Credits** | `symbol` *(string)* | `strikes` *(int, default 10)*, `expiry` *(YYYY-MM-DD)*, `date` *(YYYY-MM-DD, omit for live chain)*, `time` *(HH:MM IST, default "15:30")* |
| **`get_wa_oi_buildup`** | Call vs Put Open Interest buildup from stored snapshots: CE/PE OI, PCR, window deltas, and prior-session settlement change across 1m–1h rollups | **10 Credits** | `symbol` *(string)* | `timeframe` *("1m" \| "3m" \| "5m" \| "15m" \| "30m" \| "1h", default "15m")*, `date` *(YYYY-MM-DD, omit for latest stored session)*, `expiry` *(YYYY-MM-DD)*, `from` *(HHMM, default "0915")*, `to` *(HHMM, default "1530")*, `range_start` *(number)*, `range_end` *(number)*, `limit` *(int, default 20, max 100)* |
| **`get_wa_technical_analysis`** | Multi-timeframe technical indicator & regime breakdown: MA Consensus, Trend Horizons (EMA 9-200), Momentum (RSI/MACD/ROC), Swings, Pivots, Option Chain context | **5 Credits** | `symbol` *(string)* | `timeframe` *(string, default "15m")*, `date` *(YYYY-MM-DD)*, `include_option_chain` *(bool, default true)* |
| **`get_wa_institutional_positioning`** | NSE institutional & participant derivatives positioning (FII/DII/Pro/Client OI, long/short ratios, cash flows, India VIX, F&O ban list), monthly leaderboard, participant time series, and intraday Volume Profile (POC/VAH/VAL) | **5 Credits** | *None* | `view` *(string: `"overview"` \| `"summary"` \| `"time_series"` \| `"leaderboard"` \| `"volume_profile"`)*, `date` *(YYYY-MM-DD or `"latest"`)*, `participant` *(`"FII"` \| `"DII"` \| `"PRO"` \| `"CLIENT"`)*, `month` *(YYYY-MM)*, `symbol` *(string, default `"NSE:NIFTY50-INDEX"`)*, `limit` *(int, max 60)* |

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
- **BANK NIFTY**: `"NSE:NIFTYBANK-INDEX"`
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

### Recipe 5: Intraday & Historical Candlestick Chart Data
When a user asks for the latest or recent intraday candles (especially on weekends, exchange holidays, or pre-open before 09:15 IST), **omit `date`** so the most recent completed trading session is returned automatically:
```json
// Call get_wa_historical_chart_data (latest completed session)
{
  "symbol": "NSE:NIFTY50-INDEX",
  "resolution": "15m",
  "limit": 50
}
```
Pass `date: "YYYY-MM-DD"` only when targeting a specific calendar trading day (passing a Saturday, Sunday, or holiday date returns an honest empty-session notice):
```json
// Call get_wa_historical_chart_data (specific trading day)
{
  "symbol": "NSE:NIFTY50-INDEX",
  "resolution": "15m",
  "date": "2026-11-13"
}
```
*Output: Structured OHLCV Markdown table formatted with IST timestamps, Open, High, Low, Close, and Volume in Indian units (Cr, L, K), preceded by session summary metrics (Period, Session High/Low, Net Range).*

### Recipe 6: Multi-Timeframe Technical & Regime Analysis
When a user requests trend direction, moving averages, momentum indicators, or support/resistance pivots:
```json
// Call get_wa_technical_analysis
{
  "symbol": "NSE:NIFTY50-INDEX",
  "timeframe": "15m",
  "include_option_chain": true
}
```
*Output: Comprehensive 6-section technical dossier covering:*
1. **Consensus Verdict**: Aggregate Buy/Sell/Neutral counts across MAs and Oscillators.
2. **Trend & Moving Averages**: Alignment across EMA 9, 21, 50, 200 plus full 12-row SMA/EMA table.
3. **Momentum & Oscillators**: RSI value & zone, MACD stance/histogram, ROC velocity.
4. **Market Regime & Price Action**: Volatility regime and confirmed swing highs/lows.
5. **Pivot Points**: Classic, Fibonacci, and Camarilla S1-S3 / R1-R3 levels.
6. **Option Chain Sentiment**: PCR, ATM strike, Max Call OI resistance, Max Put OI support.

### Recipe 7: Institutional Positioning (FII/DII/Pro/Client OI) & Volume Profile (POC)
When a user asks for FII/DII positioning, institutional derivatives OI, monthly smart-money leaderboard, or session Volume Profile (POC / Value Area):
```json
// Call get_wa_institutional_positioning (full overview)
{
  "view": "overview",
  "symbol": "NSE:NIFTY50-INDEX"
}
```
To inspect multi-session positioning trends for a specific participant (`FII`, `DII`, `PRO`, or `CLIENT`):
```json
// Call get_wa_institutional_positioning (participant time series)
{
  "view": "time_series",
  "participant": "FII",
  "limit": 15
}
```
*Output: Token-dense GFM tables covering participant Index/Stock Futures & Call/Put OI, net long/short ratios, FII/DII cash flows (₹ Cr), India VIX, F&O ban list, monthly win-rate/bias leaderboard, and intraday Volume Profile (POC, VAH, VAL).*

### Recipe 8: Call vs Put OI Buildup (Intraday & Historical)
When a user asks where call/put writers are building or unwinding open interest, asks for PCR trend across the session, or wants window-on-window OI deltas:
```json
// Call get_wa_oi_buildup (latest stored session, 15-minute windows)
{
  "symbol": "NSE:NIFTY50-INDEX",
  "timeframe": "15m",
  "limit": 20
}
```
**Omit `date`** on weekends, exchange holidays, or pre-open (before 09:15 IST) so the most recent stored trading session resolves automatically. Pass `date: "YYYY-MM-DD"` only for a specific session — a Saturday or holiday returns an honest empty-session notice rather than synthetic rows. Narrow the read with `expiry`, `from`/`to` (HHMM IST) or a strike range:
```json
// Call get_wa_oi_buildup (3-minute windows, bounded strike range)
{
  "symbol": "NIFTY",
  "timeframe": "3m",
  "date": "2026-11-18",
  "from": "0915",
  "to": "1200",
  "range_start": 24000,
  "range_end": 24500
}
```
*Output: session summary (date, window, expiry, strike range, latest PCR/OI/sentiment) followed by a chronological table of rolled windows: CE OI, PE OI, CE ΔOI, PE ΔOI, PCR, OI gap, volume, and sentiment.*

For an exact point-in-time chain instead of rolled windows, call `get_wa_chain_snapshot(symbol: "NIFTY", date: "2026-11-18", time: "14:45")` — omitting `date` always returns the live broker chain.

---

## 5. Market Hours & Operating Boundaries

- **Regular Trading Hours**: Monday through Friday, **09:15 to 15:30 IST** (excluding exchange holidays).
- **Pre-Open Market Session**: **09:00 to 09:15 IST**.
- **Platform Operating Window**: **06:30 to 23:30 IST daily**.
- **Off-Hours Queries**: During market off-hours or weekends, queries without a weekend/holiday `date` filter return the last finalized closing session prices, candles, and settled option chain data. Explicit `date` queries for non-trading days report off-market status honestly with zero synthetic candles.

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
