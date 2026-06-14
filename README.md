# Ask Humans — real human answers for AI agents, paid on Celo


**Ask 1–5 verified humans any question and get answers in minutes.** Pay per
answer ($0.33) with a plain USDC transfer via **x402** — no API key, no escrow,
no account. The humans are paid on **Celo** the moment they answer.

Operated by **Sippar HireWire**, ERC-8004 agent
[#9274 on Celo](https://8004scan.io/agents/celo/9274) — a *keyless* agent whose
wallet (`0x07fBca218b0a0a35244e0025a036Fa85a6dC97DC`) has **no private key**:
every payout is signed by ICP threshold cryptography (t-ECDSA), so the key
exists only as distributed shares across the Internet Computer subnet and can
never be drained or leaked.

## What it's for
- **Verification votes** — "Is this real / open / correct?" 3–5 independent human confirmations for <$1.65.
- **Judgment / tiebreaks on AI output** — "Which response is better?" "Safe to send?"
- **Ground-truth labels** — buy N human labels per run to score your agent.
- **Honest reactions** — names, copy, prices ("would you pay $X?").

## How it works (x402)
1. `POST …/ask-humans/agent` → **402** with `paymentRequirements` (USDC amount + `payTo`, your chain).
2. Pay the USDC to `payTo` on your chain (8 supported: base, arbitrum, optimism, polygon, bnb, **celo**, solana, stellar).
3. Retry with `X-PAYMENT: <txHash>` → verified on-chain by an ICP canister → `runId` + poll URL.
4. Poll for `answers[]` (minutes, not ms).

Full protocol in [SKILL.md](./SKILL.md).

> Note on keys: *you* (the caller) pay from your own wallet — that's the only
> private key involved. The Sippar agent that fulfills the run and pays the
> humans is the keyless one.

## Celo example (viem)

```ts
import { createWalletClient, http, parseAbi } from 'viem';
import { celo } from 'viem/chains';
import { privateKeyToAccount } from 'viem/accounts';

const API = 'https://sippar.network/api/sippar/paysh/compose/ask-humans/agent';
const USDC = '0xcebA9300f2b948710d2653dD7B07f33A8B32118C'; // native USDC on Celo (6 decimals)
const body = { question: 'Would you trust an app called "Aurora" for banking? Why?', respondents: 3, chain: 'celo' };

// 1. Probe → 402 with payment requirements
const r402 = await fetch(API, {
  method: 'POST', headers: { 'Content-Type': 'application/json' }, body: JSON.stringify(body),
});
const { paymentRequirements } = await r402.json(); // { payTo, amountBaseUnits, asset, network }

// 2. Pay USDC on Celo to payTo — you control this wallet; the payment IS the auth
const account = privateKeyToAccount(process.env.PRIVATE_KEY as `0x${string}`);
const wallet = createWalletClient({ account, chain: celo, transport: http() });
const txHash = await wallet.writeContract({
  address: USDC,
  abi: parseAbi(['function transfer(address,uint256) returns (bool)']),
  functionName: 'transfer',
  args: [paymentRequirements.payTo, BigInt(paymentRequirements.amountBaseUnits)],
});

// 3. Retry with the tx hash → runId
const r = await fetch(API, {
  method: 'POST',
  headers: { 'Content-Type': 'application/json', 'X-PAYMENT': txHash },
  body: JSON.stringify(body),
});
const { runId } = await r.json();

// 4. Poll for human answers
const poll = `https://sippar.network/api/sippar/paysh/compose/ask-humans/${runId}`;
// fetch(poll) every 30–60s until status === 'complete'
```

## Why this vs a human-task marketplace
RentAHuman, Loopuman, Mavu, sanctifai — all require API keys, credit top-ups, or
escrow lifecycles. This is the **first x402-payable** one: one HTTP request, one
payment, one poll loop. Workers are verified humans (optionally via **Self**
zk-passport proof, whose registry also lives on Celo), paid instantly on Celo,
every answer backed by an on-chain receipt.

## Links
- Agent (ERC-8004 #9274): https://8004scan.io/agents/celo/9274
- Live page (humans): https://sippar.network/ask-humans
- Treasury / on-chain payouts: https://celoscan.io/address/0x07fBca218b0a0a35244e0025a036Fa85a6dC97DC
- Built by **Sippar** — the multi-chain agent payment bridge.

## License
MIT — see [LICENSE](./LICENSE).
