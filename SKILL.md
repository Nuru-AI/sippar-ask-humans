---
name: ask-humans
description: Ask 1-5 real, verified humans any question and get answers in minutes — human-in-the-loop judgment, verification, QA of AI outputs, surveys, ground-truth labels. Pay per answer ($0.33) via x402: no API key, no escrow, no account. Workers are paid on Celo the moment they answer. Use when your agent needs a human opinion, a second opinion, a real-world fact check, a tiebreak between two outputs, or honest human reactions to a name/idea/text.
---

# Ask Humans — real human answers for AI agents

When your agent can't decide alone, ask real humans. 1–5 verified people answer
your question in their own words, usually within minutes. You pay $0.33 per
answer with a plain USDC transfer on any major chain — **no API key, no escrow,
no account**. The humans are paid on Celo the second their answer is accepted.

Good for:
- **Verification votes** — "Is this business real / open / correct?" 3–5
  independent human confirmations for under $1.65.
- **Judgment / tiebreaks on AI output** — "Which of these two responses is
  better?" "Is this reply appropriate to send?"
- **Ground-truth sampling** — buy N human labels per run to score your agent.
- **Honest human reactions** — names, ideas, copy, prices ("would you pay $X?").

## How to use

Base URL: `https://sippar.network/api/sippar/paysh/compose/ask-humans`

### 1. Request a run (you'll get a 402 with payment instructions)

```bash
curl -X POST https://sippar.network/api/sippar/paysh/compose/ask-humans/agent \
  -H "Content-Type: application/json" \
  -d '{"question": "Would you trust an app called Aurora for banking? Why or why not?", "respondents": 3, "chain": "base"}'
```

The 402 response contains `paymentRequirements`: the exact USDC amount
(`amountBaseUnits`, 6 decimals) and the treasury address (`payTo`) on your
chosen chain. Supported chains: `base`, `arbitrum`, `optimism`, `polygon`,
`bnb`, `celo`, `solana`, `stellar`.

### 2. Pay

Transfer the USDC amount to `payTo` on your chain. Any wallet you control
works — the payment IS the authentication.

### 3. Retry with the tx hash

```bash
curl -X POST https://sippar.network/api/sippar/paysh/compose/ask-humans/agent \
  -H "Content-Type: application/json" \
  -H "X-PAYMENT: 0xYOUR_TX_HASH" \
  -d '{"question": "Would you trust an app called Aurora for banking? Why or why not?", "respondents": 3, "chain": "base"}'
```

Payment is verified on-chain (ICP threshold verification via the x402_payments
canister — one tx funds exactly one run). The response includes `runId` and a
`poll` URL.

### 4. Poll for answers (humans take minutes, not milliseconds)

```bash
curl https://sippar.network/api/sippar/paysh/compose/ask-humans/<runId>
```

Poll every 30–60 seconds. Answers stream into `answers[]` as real people
reply; `status` becomes `complete` when all respondents have answered. Expect
first answers within minutes, full panels can take longer. Runs expire after
24 hours.

## Rules

- `question`: 10–300 characters, no links or contact details.
- `respondents`: 1–5. Price is $0.33 × respondents.
- Answers shorter than 15 characters are auto-rejected and never charged.
- Treat answers as anonymous human opinion, not expert advice.

## Why this instead of a human-task marketplace

No account to create, no credits to top up, no escrow lifecycle to manage, no
API key to store. One HTTP request, one payment, one poll loop. Workers are
verified humans (not AI personas) paid instantly on Celo — every answer is
backed by an on-chain payout. Operated by Sippar HireWire, ERC-8004 agent
#9274 on Celo (https://8004scan.io/agents/celo/9274), whose wallet has no
private key (ICP threshold signatures).
