# Ovra Skills

Payment skills for AI agents. Zero-knowledge checkout — the agent never sees card data.

## Install

```bash
npx skills add AlecFritsch/ovra-skills
```

Or manually copy `ovra-agentic-payments/SKILL.md` into your skills directory.

## What this enables

Say "buy X" to your AI agent and it handles the entire checkout securely:

1. Declares intent (what to buy, how much, which merchant)
2. Policy engine checks spending rules
3. Fills the checkout form via CDP — agent never sees PAN/CVV
4. Attaches receipt for audit trail
5. Verifies transaction matches intent

Works with Claude Code, Cursor, OpenClaw, Codex, and any MCP-compatible agent.

## Setup

Connect to Ovra's MCP server:

```json
{
  "mcpServers": {
    "ovra": {
      "url": "https://mcp.getovra.com/mcp",
      "headers": { "Authorization": "Bearer sk_test_YOUR_KEY" }
    }
  }
}
```

Get your API key at [getovra.com/dashboard/keys](https://getovra.com/dashboard/keys).

## Available Skills

| Skill | Description |
|-------|-------------|
| [ovra-agentic-payments](./ovra-agentic-payments/SKILL.md) | Complete payment flow: intents, checkout, evidence, disputes |

## Why Ovra?

Every other agent payment solution gives the agent your card number. Ovra doesn't. The agent gets `{ success: true }` and nothing else. Card data stays on Ovra's server, typed directly into the checkout form via Chrome DevTools Protocol.

## Links

- [Website](https://getovra.com)
- [Documentation](https://docs.getovra.com)
- [MCP Server](https://mcp.getovra.com/mcp)
- [Dashboard](https://getovra.com/dashboard)
