# Alert Rules

The **Alerts** page (in the **Developer** group of the sidebar) lists events on your account as they happen. The top of the page is a feed of notifications, and below it are the rules that decide which kinds of notice you want.

---

## Four categories

Each alert carries a coloured tag showing its category:

| Tag | Raised when | Example title |
|---|---|---|
| **Transfer** | A payment arrives in one of your accounts | Transfer received |
| **Policy** | One of your agents reaches 80% or more of its daily spending limit | Agent account nearing spending limit |
| **Security** | Access to your data is granted, for example a view key created by a compliance disclosure | View key access granted |
| **System** | Notices from the platform, such as the one you get on signup | Welcome to Veiled |

The tag tells you which alerts need attention. A **Policy** alert usually does, while **Transfer** and **System** alerts are mostly for your information. Alert bodies give the details, for example "Incoming confidential transfer of $25 USDG from @maria" or "Agent account @research-bot has used 85% of its daily spending limit".

---

## Using the feed

- Unread alerts have a lit dot on the left and a slightly lighter background.
- Clicking an alert you haven't seen yet clears its unread marker.
- **Mark all read** marks every unread alert as read in one go. It's greyed out when there's nothing unread.
- The **all** and **unread** tabs switch the view. The unread tab shows how many are waiting, for example **unread (3)**.
- Timestamps are relative ("Just now", "12 min ago", "3 hr ago", "2 days ago").

---

## Choosing which alerts you get

The **Notification preferences** section below the feed has one switch per rule. New accounts start with these rules:

| Rule | Default |
|---|---|
| Notify me when a transfer is received | On |
| Notify me when an agent account nears its spending limit | On |
| Notify me when a view key is used | On |
| Notify me on new Veiled platform updates | Off |

Each switch saves as soon as you click it. A switch affects alerts from that point onward. It doesn't remove alerts already in your feed.

### A switch only controls the notice

Turning a rule off only stops the notice. It has no effect on whatever produces the notice. For example, if you turn off the agent limit rule, the agent's spending limit stays exactly where it was. You just stop hearing when the agent gets close to it. Spending limits are enforced by the Veiled agent program on-chain, whether or not you're receiving alerts. [Spending Guardrails](../autonomous-agents/spending-guardrails.md) explains how that enforcement works.

---

## Related pages

- [Spending Guardrails](../autonomous-agents/spending-guardrails.md)
- [Profile and Preferences](profile-and-preferences.md)
