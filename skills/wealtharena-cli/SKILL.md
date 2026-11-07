---
name: wealtharena-cli
description: "Use the Wealth Arena CLI (wac/wa) for Indian financial markets (NSE/BSE): live quotes, Option Chain with Greeks, GEX gamma flips, algorithmic backtests, paper trading, and quota usage."
---

# Wealth Arena CLI (`wac` / `wa`) — Agent Operating Guide

The Wealth Arena CLI is a high-performance terminal interface and Model Context Protocol (MCP) companion designed for quantitative traders and autonomous AI coding agents (Claude Code, Gemini CLI, Cursor, Antigravity, OpenCode).

Both `wa` and `wac` (and `wealtharena-cli`) are identical, interchangeable binary aliases.

---

## 1. Core Operating Principles

1. **Token Efficiency (`--md` Default)**:
   - By default (or with `--md`), `wac` formats all command outputs as token-dense GitHub Flavored Markdown (GFM) tables and cards.
   - Conserves **60%–80% context tokens** compared to verbose raw JSON envelopes.
   - Pass `--json` only when piping directly into `jq`, custom shell pipelines, or automated JSON parsers.
   - AI-tagged calls (`--ai` or `--actor <name>`) with `--json` return a reduced envelope (status/message dropped on success, data intact; errors verbatim).

2. **Data Authenticity (Fail-Honest Mandate)**:
   - Reconstructing, synthesizing, or interpolating missing market data is strictly forbidden.
   - If market snapshots, Greek baselines, or contract metrics are missing or corrupted at the source, surface honest empty states (`-` or `N/A`). Never fabricate placeholder values.

3. **Execution Safety (Live Gate)**:
   - Paper trading is free and unrestricted for experimentation: `wac paper ...`
   - Live broker order placement strictly refuses without `--live --yes` and margin pre-checks: `wac live ...`

---

## 2. Compute Credit Weights

Every authenticated API request consumes credits based on backend compute cost. Free control-plane requests are never billed:
*(Note: Credit weights reflect backend compute tiers and operational costs; refer to `wac usage` for active quota status.)*

| Credit Weight | API & CLI Category | Representative Commands |
|:---|:---|:---|
| **0 Credits** | Control Plane & Quota | `wac login`, `wac logout`, `wac usage`, `wac plans`, `wac flags list`, `wac skill` |
| **1 Credit** | Lightweight Lookups | `wac market quote`, `wac market depth`, `wac me`, `wac broker status`, `wac market search` |
| **5 Credits** | Analytical Compute | `wac chain snapshot`, `wac market history`, Pre-Open Gap analysis |
| **10 Credits** | Bulk Analytics | Bulk multi-strike scans, Call vs Put OI time-series analysis |
| **20 Credits** | High Compute Engine | Quantitative strategy backtesting runs (`wac backtest run`) |

---

## 3. High-Frequency Command Recipes

```bash
# 1. Check Remaining Compute Credits & Cycle Pacing
wac usage

# 2. Inspect Active Session Identity & Roles
wac me

# 3. Real-Time Market Quote
wac market quote --symbol NSE:NIFTY50-INDEX

# 4. Market Depth (5-Level Level 2 Orderbook)
wac market depth --symbol NSE:RELIANCE-EQ

# 5. Live Option Chain with Greeks (10 strikes around ATM)
wac chain snapshot --symbol NIFTY --strikes 10

# 6. Specific Expiry Option Chain Snapshot
wac chain snapshot --symbol NIFTY --expiry 2026-11-26 --strikes 10

# 7. Historical Daily OHLCV Candle Bars
wac market history --symbol NSE:NIFTY50-INDEX --resolution 1D

# 8. Check Broker Connectivity (Fyers, Zerodha, INDmoney)
wac broker status

# 9. Paper Trading Capital Reset (₹10,00,000 default)
wac paper reset --capital 1000000

# 10. View Canonical Catalog of Subscription Tiers & Capabilities
wac plans
```

---

## 4. Exit Codes & Error Recovery

The CLI communicates execution states deterministically via standard process exit codes:

| Exit Code | Classification | Meaning & Recommended Agent Recovery |
|:---:|:---|:---|
| **0** | `SUCCESS` | Command completed successfully. Read output from stdout. |
| **2** | `USAGE_ERROR` | Invalid command syntax, unrecognized flags, or mutually exclusive flags (`--md` and `--json` passed together). |
| **4** | `AUTH_REQUIRED` | Missing or expired token. Recovery: Run `wac login`, set `WA_ACCESS_TOKEN`, or pass `--token <jwt>` in-memory. |
| **5** | `RATE_LIMITED` | Request throttled by burst/sliding rate limiters. Recovery: Sleep for interval specified in `Retry-After` header. |
| **6** | `SERVICE_OUTAGE` | Broker disconnected, billing store unavailable, or platform off-hours (platform operating window: 06:00–23:30 IST daily). Never fabricate synthetic numbers on exit 6. |
| **7** | `GATE_REFUSED` | Feature flag killed (`kill:*`), live gate refusal without `--live --yes`, or emergency trading halt active (`wa halt`). |
| **8** | `QUOTA_EXHAUSTED` | Monthly compute credits exhausted (`HTTP 402 QUOTA_EXHAUSTED`) or plan past due. Recovery: Inspect remaining balance with `wac usage`. |
| **1** | `GENERIC_ERROR` | Internal server or network transport error. |
