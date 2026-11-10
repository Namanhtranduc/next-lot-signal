# Setup and finding the data

Users can set things up by hand (copy the templates, fill in `config.md`, list each file under "Data sources"), or let the agent do it. Both ways work. Whatever the user has filled in by hand always wins: only fill gaps, never overwrite a value the user wrote.

## 1. The folder
- If the request names a folder, use it.
- If it names none and there is exactly one folder in the repo with a next-lot-signal `config.md`, use that one and say which.
- Otherwise suggest `next-lot/` at the repo root and create it once the user agrees.
- If the folder has no `config.md` or `decisions.md`, copy the empty templates from the skill root into it. Never write into the skill root.

## 2. Guided setup (when required settings are blank)
If any required setting in `config.md` is blank, offer to fill it in now. Ask, in **one** message, only the questions that are still open. Under each question, add a short hint on **where to find the answer**, so a user who doesn't know it yet knows where to look:

| Setting | Question to ask | Where to find it (hint to show) |
|---|---|---|
| `factory_days` | How many days from placing an order until the goods leave the factory? | The factory's quote or the last purchase order: order date to ex-factory date. If unsure, ask the factory for its current lead time; components such as circuit boards can stretch it. |
| `transit_days` | How many days from leaving the factory until the stock can be sold? | The freight forwarder's quote, or the last shipment: ex-factory date to the day the stock showed up as sellable (for example, in Shopify inventory). Include customs and warehouse receiving. |
| `buffer_days` | How many days of safety margin do you want? | Look at your last few shipments and take the worst delay against the promised date. With no history yet, many teams start with 2–3 weeks. At least 7. |
| `post_peak_daily_orders` | After the peak season, about how many units a day do you expect to sell? | Last year's sales for the same months (Shopify Analytics → Sales over time), a similar product's off-season sales, or your own plan. |
| `peak_end` | What is the last day of the current peak season? | Your marketing calendar, or the last shipping day for gift delivery (often around 20–24 December for Christmas). |
| `next_peak_start` | When does the next peak season start? | When sales jumped last year (Shopify Analytics), or the start of your next big campaign (for US holiday sales, often early to mid November). |
| `tet_last_order_date`, `tet_full_capacity_date` (recommended) | Has the factory told you its last order date before Lunar New Year (Tết / CNY) and the date it is back to full capacity? | Only the factory knows. Offer the drafted factory question (SKILL.md, step 7). For reference, Lunar New Year 2027 is on 6 February; factories often stop new production weeks earlier and need 4–6 weeks after it to reach full output. |
| `moq` (optional) | What is the factory's minimum order quantity? | The factory's quote or supply contract. |

**Suggesting a value:** if the user says they don't know, you may suggest a starting value from their own data, and say clearly how you got it. Examples:
- for `post_peak_daily_orders`, about a quarter of the current peak rate, shown as a number;
- for `buffer_days`, 14 days.

Use a suggested value only after the user confirms it. In `config.md`, mark it "(suggested, confirmed by user)".

Write each answer into `config.md` and leave unanswered fields blank. Never guess a value silently. If the user does not answer a required one, run anyway and report "Not enough data", naming the field and its hint. If the Tết dates are unknown, draft the factory question (SKILL.md, step 7).

## 3. Finding the data files
For each data item that "Data sources" in `config.md` leaves blank, look in the folder for a file that holds it:
- **Shopify orders export:** a CSV with columns such as `Name`, `Created at`, `Lineitem quantity`, `Lineitem sku`, `Fulfillment Status`, `Cancelled at`. It gives the accepted orders not yet shipped and the new orders per day.
- **Shopify inventory export:** a CSV with columns such as `SKU`, `Location`, `On hand`, `Available`. It gives stock on hand.
- **Other files:** use one only if its headings make clear what it holds. If they don't, ask.

Then:
- **Product:** if the exports contain more than one SKU and `config.md` does not name the product, list the SKUs with their units sold in the last 14 days and ask which one to plan. Do not pick one yourself.
- **Data date:** for an orders export, use the latest `Created at` in the file. For an inventory export, use the date in its file name, or else ask. Record the date in the answer.
- **Write back:** record what you found in the "Data sources" table (file name, SKU, the date rule) so the next run does not ask again. Tell the user what you wrote.

Next week, the user only replaces the export files and asks again.

## 4. Lots told in conversation
The user may simply say, for example, "a lot of 2,000 units arrives on 15/3/2027". Add it to the Lot decisions table in `decisions.md` with Status "reported, already on order". It counts as inbound like any recorded lot, and the matching rule in `references/rules.md` (Inbound lots) still prevents double counting.
- It does **not** start a new lot cycle.
- Only a "decided, X units, expected arrival Y" reply to a CEO note does that.

A short table of lots kept by hand and listed under "Data sources" works too.
