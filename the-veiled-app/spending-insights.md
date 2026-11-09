# Spending Insights

The **Analytics** page turns your transaction history into charts. It shows how money moves through your own account and through each agent that spends for you, and it uses only your data.

---

## Choosing a window

Pick a time window with the buttons along the top of the page (**7d**, **30d** and **90d** are available). The default is 7d. Every number and chart on the page uses that window, and each headline figure is compared with the window of the same length immediately before it. For example, the 30-day view compares the last 30 days with the 30 days before those.

---

## Headline figures

| Figure | How it's calculated | Change shown as |
|---|---|---|
| **Total transfers** | Number of payments in the window, across all your accounts | % change from the previous window |
| **Total volume** | Sum of those payments' amounts | % change |
| **Avg transfer amount** | Average size of a transfer (volume over count) | % change |
| **Settled rate** | Percentage of those payments with status `settled` | Change in percentage points (`pts`) |

If the previous window had no activity, the change is left blank. A percentage measured against zero wouldn't tell you anything useful, so the page doesn't show one.

---

## Charts

### Daily transfer chart

This is a bar chart with one bar per day in the window. Hover over a bar to see that day's transfer count and dollar volume. On the 7d view the axis shows weekday names. On longer views it shows dates, labelling every fifth day on the 30d view and every tenth day on the 90d view so the labels don't overlap.

### Which accounts move the most

This chart splits the window's volume across your accounts, personal and agent alike, and leaves out accounts that had no activity. Use it to see whether one agent is responsible for most of your spending.

### Split across privacy layers

This chart shows how much of your volume settled at each layer, in this order: **Confidential** (where most payments land by default, backed by Token-2022 confidential balances), then **Shielded**, then **Public**. What each layer hides is explained in [Encrypted Balances and Private Payments](../confidentiality/encrypted-balances.md).

### When you're most active

This is a grid with seven rows for the days of the week (Monday first) and 24 columns for the hours, where darker cells mean more transfers. Hover over a cell to see its exact count. The heatmap is useful for checking agents: an agent that should only run during certain hours shows up in only those cells, and a scheduled job that is running on time shows a regular pattern.

---

## Privacy

Every figure is calculated in your browser from transactions on accounts you own. Nobody else's data appears here, and opening the page has no effect on what the chain reveals. It only presents data you already have.

---

## Related pages

- [Activity Log](activity-log.md)
- [Agent Wallets](../autonomous-agents/agent-wallets.md)
