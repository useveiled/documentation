# Paying per Request with x402

x402 is an open standard for payments between machines, whether that's one AI agent paying another or an agent paying a service. It is built on HTTP's `402 Payment Required` status code. Veiled agent accounts speak it natively, so an agent can buy API calls, compute, or any other service without a credit card, an OAuth dance, or a prepaid balance. The payment travels with the HTTP request.

Solana suits this kind of traffic. Slots are roughly 400ms, fees are a fraction of a cent, and it carries deep, natively issued stablecoin liquidity, which is what payments at machine speed need. x402 supplies the shared vocabulary on top: a service states the price in its response, and the agent pays it inside the same exchange.

---

## One request, start to finish

```
1. The agent calls an x402-enabled endpoint
2. The service answers 402 Payment Required, with headers naming the amount, asset, and recipient address
3. The agent's Veiled wallet checks that request against its spend policy
4. If the policy allows it, the agent signs and submits a USDG transfer
5. The payment settles on Solana, normally within a slot or two (about 400ms)
6. The agent repeats the original call, attaching the transaction signature as proof of payment
7. The service confirms the payment on-chain and returns what was asked for
```

No person is involved unless the amount is over the agent's approval threshold, which you set in its [spend policy](spending-guardrails.md). In that case the payment waits in your approval queue. The program also records an invoice reference against each agent spend, so an x402 payment can be traced back to the invoice it settled.

---

## Billing patterns

| Pattern | How it works |
|---|---|
| Pay per call | Usually the default: a fraction of a dollar is settled alongside every single call |
| Metered | The service asks for incremental payments as usage builds up, instead of one charge at the start |
| Subscription style | A recurring x402 charge at a fixed interval, checked against the spend policy every time it comes round |

---

## Privacy at high volume

An active agent can make many small x402 payments every minute. If all of them were readable on a public chain, anyone watching could reconstruct the agent's usage and its costs, which is valuable intelligence for a competitor. That is the reason Veiled supports confidential settlement in USDG, USDC, and SOL over Token-2022 confidential balances, the same amount-hiding mechanism used for ordinary transfers (see [Encrypted Balances and Private Payments](../confidentiality/encrypted-balances.md)).

Know the current limits, though. Payments that the Veiled program settles from an agent's vault under its spend policy are checked against the policy's limits in the clear, and their amounts appear in the instruction and in the program's `AgentPaymentExecuted` event. Addresses are public for every payment.

---

## Turning it on

There is nothing to switch on. Every agent account supports x402 from the moment it is created. The one thing to get right is the spend policy: its allowed recipients must include the services you expect the agent to pay. The API side of that is covered in [Managing Agents and Policies](../developer-api/managing-agents.md).

---

## Keep reading

- [Spending Guardrails](spending-guardrails.md)
- [Reacting to Events](reacting-to-events.md)
