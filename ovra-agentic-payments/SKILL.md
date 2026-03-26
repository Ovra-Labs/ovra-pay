---
name: ovra-agentic-payments
description: >
  Use this skill when the agent needs to buy something online, pay for a
  service, complete a checkout, manage spending budgets, or track purchases.
  Enables secure payments where the agent never sees card data. Works with
  any website that accepts Visa. Use even if the user just says "buy",
  "order", "purchase", "subscribe", or "pay for".
license: MIT
compatibility: Requires Ovra MCP server connection and API key.
metadata:
  version: "1.0.0"
  author: "Ovra"
  website: "https://getovra.com"
  docs: "https://docs.getovra.com"
  mcp: "https://mcp.getovra.com/mcp"
---

# Ovra Agentic Payments

Secure payments for AI agents. The agent never touches card data.

## Setup

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

Keys at https://getovra.com/dashboard/keys. Sandbox keys: `sk_test_*`.

## Payment Flow

Every purchase follows this sequence. Do not skip steps.

```
1. ovra_policy_simulate  → will it pass?
2. ovra_intent_create    → declare what to buy
3. ovra_checkout_token   → get single-use fill token
4. ovra_checkout_detect  → find payment form on page
5. ovra_checkout_fill    → fill card fields (agent sees nothing)
6. ovra_evidence         → attach receipt
7. ovra_intent_action    → verify actual vs expected
```

### Step 1: Simulate

Before anything, check if the payment would pass policy:

```
ovra_policy_simulate({ agentId, amount: 49.99, merchant: "amazon.de" })
```

If it fails, stop and tell the user why. Do not proceed.

### Step 2: Declare intent

```
ovra_intent_create({
  agentId, purpose: "Buy wireless keyboard",
  expectedAmount: 49.99, expectedMerchant: "amazon.de"
})
```

If status is `pending_approval`: **stop and wait**. Tell the user their approval is needed in the dashboard. Do not retry or work around this.

### Step 3–4: Get token and detect form

```
ovra_checkout_token({ intentId })
ovra_checkout_detect({ cdpBaseUrl: "http://localhost:9222" })
```

The token expires in 60 seconds and works only once, only on the matching merchant domain.

### Step 5: Fill checkout

```
ovra_checkout_fill({ token, cdpBaseUrl: "http://localhost:9222" })
```

Ovra connects to the browser via CDP and types card data directly into the form. The agent never sees PAN or CVV.

### Step 6: Attach receipt

Scrape the confirmation page and attach it:

```
ovra_evidence({
  action: "attach", intentId,
  type: "receipt",
  description: "Order #12345",
  amountEuros: 49.99,
  merchant: "amazon.de"
})
```

### Step 7: Verify

```
ovra_intent_action({
  intentId, action: "verify",
  actualAmountEuros: 49.99,
  actualMerchant: "amazon.de"
})
```

## When the merchant has a payment API

Some merchants expose an API instead of a browser form. Use the proxy — same flow but replace steps 4–5:

```
ovra_checkout_proxy({
  token,
  request: {
    url: "https://api.shop.de/pay",
    method: "POST",
    body: { card: "{{PAN}}", exp: "{{EXP_SHORT}}", cvv: "{{CVC}}" }
  }
})
```

Ovra replaces placeholders server-side and scrubs card data from the response. Available: `{{PAN}}`, `{{CVC}}`, `{{EXP_MONTH}}`, `{{EXP_YEAR}}`, `{{EXP_SHORT}}`, `{{HOLDER}}`, `{{LAST4}}`.

## Gotchas

- `ovra_card_get_sensitive` exists but **never call it**. It exposes raw PAN/CVV. Always use `ovra_checkout_fill` or `ovra_checkout_proxy`.
- `ovra_checkout_execute` is for simple API-level checkouts without a browser. Prefer `ovra_checkout_fill` for real websites.
- Intents expire (default 24h). If expired, create a new one.
- The fill token expires in 60 seconds. Get it right before you need it, not earlier.
- `pending_approval` means a human must approve. The agent cannot approve its own intents when above the auto-approve threshold.
- If `ovra_checkout_detect` finds no payment form, the page might still be loading. Wait and retry once.
- Card data must never appear in your output, logs, or context. If you somehow see a 16-digit number starting with 4, do not repeat it.

## Agent vs Human boundaries

**Agent does autonomously:** simulate, create intents, get tokens, detect forms, fill checkout, attach evidence, verify, list transactions, file disputes, check balances.

**Agent asks human first:** create/delete agents, fund wallets, change policies, freeze/unfreeze cards, manage API keys, approve high-value intents.

## Checking balance and status

```
ovra_agent_get({ agentId })           → balance, status
ovra_policy_get({ agentId })          → current spending rules
ovra_transactions({ action: "list" }) → recent purchases
ovra_intents_list({ status: "pending_approval" }) → waiting for human
```

## Filing a dispute

If a purchase went wrong:

```
ovra_dispute_create({
  transactionId,
  reason: "not_received",
  description: "Package not delivered after 14 days"
})
```

Reasons: `unauthorized`, `fraud`, `not_received`, `duplicate`, `incorrect_amount`, `canceled`.

## Merchant intelligence

Use before setting up policies to find the right merchant names and MCC codes:

```
ovra_merchant_intel({ action: "suggest", query: "cloud hosting" })
ovra_merchant_intel({ action: "resolve", query: "hetzner.com" })
```
