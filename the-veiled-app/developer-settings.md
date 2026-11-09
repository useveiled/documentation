# Developer Settings

The **API Keys** page, under **Developer** in the sidebar, is where you create credentials and register webhook endpoints from the dashboard instead of through API calls. Once your integration is running, [Making Authenticated Requests](../developer-api/making-requests.md) covers the API side.

---

## Issuing a key from the dashboard

1. Press **+ New key**.
2. Give the key a name, for example `Production` or `staging-bot`.
3. Choose the environment:

   | Environment | Key starts with | Network |
   |---|---|---|
   | **live** (default) | `vl_live_` | Solana mainnet, real USDG |
   | **test** | `vl_test_` | Solana devnet, nothing of value moves |

4. Press **Generate**.

The key is generated in your browser: the prefix followed by 48 random hexadecimal characters. Veiled stores only a SHA-256 hash of the key, the prefix and the last four characters. The complete key appears once, in a panel with a **Copy** button. That panel says the key will be shown once and can't be retrieved later. Copy the key into your secret manager before you close the panel, because nobody can retrieve it afterwards, Veiled included. If you lose a key, revoke it and create another one.

A quick check that a new test key works with the SDK:

```ts
import { Veiled } from "@useveiled/sdk";

// Read the key from the environment instead of hard-coding it
const veiled = new Veiled({ apiKey: process.env.VEILED_API_KEY! });
console.log(veiled.environment); // prints "test" for a vl_test_ key
```

---

## The key list

Each key in the list shows:

- A status badge: **Active** or **Revoked**.
- A masked version of the key, made of the prefix, four dots and the last four characters (for example `vl_live_••••9f2c`). This is enough to tell keys apart.
- **Created** and **Last used** dates.

### Revoking

Press **Revoke** on an active key. It stops working on the next request, and you can't undo this. The revoked key stays in the list labelled **Revoked**, so your records still show every key that has existed.

---

## Webhook endpoints

### Adding one

Press **+ Add endpoint** in the Webhooks section, then:

1. Enter the HTTPS URL that should receive events, for example `https://hooks.example.com/veiled`.
2. Select at least one event type:

   | Event | Meaning |
   |---|---|
   | `transfer.settled` | A transfer finalised on Solana |
   | `transfer.failed` | The payment ended without moving funds |
   | `payment.received` | Money arrived in one of your accounts |
   | `policy.limit_reached` | An agent reached a limit in its spend policy |

3. Press **Add webhook**. The button stays disabled until there's a URL and at least one event selected.

An endpoint gets only the events you selected for it. You can register several endpoints for different consumers, for example one that forwards transfers to a chat channel and another that sends only `payment.received` to your billing service.

### Pausing and removing

Each endpoint has an **Active** or **Inactive** button. Click it to switch the endpoint to the other state. An inactive endpoint stops receiving deliveries but keeps its URL and event list, which is useful while you fix the service on the receiving end. **Remove** deletes the endpoint entirely.

---

## Payloads and signatures

This page only covers creating and managing keys and endpoints. For the full event list, payload formats and how to verify signatures, see [Reacting to Events](../autonomous-agents/reacting-to-events.md) and [Webhook Delivery and Signatures](../developer-api/webhook-delivery.md).

---

## Related pages

- [Making Authenticated Requests](../developer-api/making-requests.md)
- [Reacting to Events](../autonomous-agents/reacting-to-events.md)
