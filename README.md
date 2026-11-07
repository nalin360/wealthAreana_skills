# Wealth Arena Agent Skills (`wealthAreana_skills`)

Official repository of **Agent Skills** for [Wealth Arena](https://wealtharena.in) and the [Wealth Arena CLI (`wac` / `wa`)](https://www.npmjs.com/package/wealtharena-cli).

Agent Skills are standardized Markdown documents (`SKILL.md`) that teach autonomous AI coding agents (Claude Code, Gemini CLI, Cursor, Antigravity, OpenCode, Codex) how to interact with the Wealth Arena platform, execute market commands, analyze Option Chains, run algorithmic backtests, and manage compute quotas without trial-and-error hallucinations or context window bloat.

Inspired by the open [Agent Skills Standard](https://agentskills.io) and [Neon Agent Skills](https://github.com/neondatabase/agent-skills).

---

## Available Skills

| Skill | Path | Description |
|:---|:---|:---|
| **`wealtharena-cli`** | [`skills/wealtharena-cli/SKILL.md`](skills/wealtharena-cli/SKILL.md) | Core operating playbook for `wac` / `wa`: real-time market quotes, 5-level depth, Option Chains with Greeks, GEX gamma flips, backtesting, paper trading, and compute quota telemetry. |

---

## Installation & Setup

### Option 1: Via Wealth Arena CLI (`wac`)

If you have `wealtharena-cli` installed globally or available via `npx`:

```bash
# Default install (scaffolds to .agents/skills/wealtharena/SKILL.md)
wac skills install

# For Claude Code (.claude/skills/wealtharena/SKILL.md)
wac skills install --agent claude

# For Cursor (.cursor/rules/wealtharena.mdc)
wac skills install --agent cursor

# Custom workspace directory
wac skills install --dir ./custom-skills/
```

### Option 2: Via `skills` CLI Runner

Install directly from GitHub into your active agent workspace using the universal `skills` CLI:

```bash
# Install all skills from this repository
npx skills add nalinsahani/wealthAreana_skills -y

# For Claude Code
npx skills add nalinsahani/wealthAreana_skills -a claude-code

# For Cursor
npx skills add nalinsahani/wealthAreana_skills -a cursor
```

### Option 3: Manual Project Drop

Copy [`skills/wealtharena-cli/SKILL.md`](skills/wealtharena-cli/SKILL.md) into your project's agent skills directory:

* **Antigravity / Gemini CLI / OpenCode**: `.agents/skills/wealtharena/SKILL.md`
* **Claude Code**: `.claude/skills/wealtharena/SKILL.md`
* **Cursor**: `.cursor/rules/wealtharena.mdc`

### Option 4: On-The-Fly Shell Streaming

AI coding agents with terminal execution tools can stream the playbook directly into their context on demand without writing files:

```bash
wac skill
```

---

## Key Principles for AI Agents

1. **Token Efficiency (`--md` Default)**:
   All `wac` commands format output as clean, token-dense Markdown tables and cards by default, saving **60%–80% context tokens** compared to raw JSON.
2. **Data Authenticity (Fail-Honest)**:
   Never interpolate or synthesize placeholder values. Missing market feeds surface as honest empty states (`-`).
3. **Execution Safety (Live Gate)**:
   Paper trading is free (`wac paper`); live orders require explicit `--live --yes` and margin pre-checks (`wac live`).

---

## License

Apache License 2.0 — see [LICENSE](LICENSE).  
The license covers these skill specifications only; it grants no rights to Wealth Arena API services, market data, or platform trademarks.
