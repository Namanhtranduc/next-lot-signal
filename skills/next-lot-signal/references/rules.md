# next-lot-signal rules

**In short:**
1. Project stock day by day with no new lot: inbound first, then the waiting queue, then new orders. Peak uses the recent average; after the peak, three scenarios (slow / mid / fast = post-peak rate −40% / as set / +40%).
2. arrival(X) = order day + factory days + transit days; an order placed during the Lunar New Year (Tết / CNY) closure counts from the day the factory is back to full capacity.
3. Full-buffer deadline = the latest order day whose arrival still leaves `buffer_days` before the stock-out.
4. The mid scenario drives the status, unless the fast scenario's deadline is the Tết cut-off and earlier; then that cut-off drives it.
5. Status by days left: 43+ "Time left", 15–42 "Approaching decision window", 0–14 "In decision window"; plus "Past deadline", "Enough stock until next peak", "Not enough data".
6. Ask the CEO only the first time in a lot cycle that the window opens, the deadline passes, a shortage becomes unavoidable, or a defect signal fires; attach a decision record.

Numbers marked *[design parameter]* were chosen to make the skill work. Only `band_pct` is in `config.md`; change the others by editing this file.

---

## Definitions and input checks

**New orders per day:** orders received that day, minus those of them that were later cancelled. Count a cancellation against the day the order was received, not the day it was cancelled; an order received before the 14-day window and cancelled inside it changes nothing in the window. Shipping older orders does not count.

**Usable day for measuring the sales rate:** a day when the store accepted orders. Pre-order days count even if there was no stock. Exclude only days when the store stopped taking orders because it was sold out, and promotion days.

**Stock and the waiting queue:**
- Use stock on hand, including units already promised to waiting orders. If a source gives stock already net of those orders (for example Shopify's "Available" = "On hand" − "Committed"), use "On hand" instead, so the queue is not counted twice.
- Stock is never below 0.
- Accepted orders not yet shipped (also called open orders) form the **waiting queue**. They always wait, and they are shipped first as soon as stock arrives.

**New orders that cannot be filled on the day:**
- **During the pre-order phase:** they join the waiting queue, whatever `lost_or_wait` says. The phase runs from today until `preorder_promise_date`.
  - The phase includes `preorder_promise_date` itself.
  - If `preorder_promise_date` is blank and today's stock is smaller than the unshipped orders, treat it as a pre-order phase anyway. The phase ends the day before the earliest inbound lot arrives. On the arrival day the normal order applies: add the lot, ship the queue, then fill that day's orders. Remind the user to fill in the field.
  - If there is no inbound lot either, the phase has no end. Report the status "Not enough data", missing field: promise date or inbound lot.
- **Outside the pre-order phase:** follow `lost_or_wait`. With "lost", drop the order. With "wait", add it to the waiting queue.

**Shortage day:** a day when demand is not fully met. Judge it after steps 1–3 of that day (Rule 1): the waiting queue is still not empty, or some of the day's new orders could not be filled.
- A day that ends at exactly 0 stock with all demand met is **not** a shortage day.
- Days inside the pre-order phase are **not** shortage days.
- Shortage days are counted on projected demand, even on days when the store is closed for orders because it sold out. Use the rate of that season (peak or post-peak).

**Simulation start:** the stock and queue figures are taken as the end of today. The simulation starts tomorrow. Today's orders are already counted as unshipped orders. If a source is older than today, still start from its figures and mark it stale; do not simulate the missing days. For testing, `as_of_date` in `config.md` replaces "today".

**Counting days:** days left = deadline − today, so a deadline of today is 0 days. Example: 1/12/2026 − 26/10/2026 = 36 days.

**Decimals and rounding:** keep daily rates as decimals in the projection (2.4 a day stays 2.4). Do not round daily orders. Treat anything below 0.001 as 0. Round only the final units needed, always up to a whole unit. Months of sales = days ÷ 30.4, one decimal.

**Dates in the output:** write d/m/yyyy in the report line and the decision record.

**Seasons:**
- Peak runs from today until `peak_end`, including `peak_end`. From the day after `peak_end` it is post-peak. That is normal and needs no reminder.
- Once `next_peak_start` has passed, treat both season dates as blank. Report "Not enough data" and remind the user to update both dates together.

**Tết dates:**
- If `tet_full_capacity_date` is earlier than today, treat both Tết dates as blank.
- If `tet_last_order_date` has passed but `tet_full_capacity_date` has not, keep both. arrival(X) below already gives the right answer: any order from today until `tet_full_capacity_date` arrives on the same date.
- If only one of the two Tết dates is filled, treat both as blank.

**arrival(X), the arrival date if ordered on day X:**
- If X is after `tet_last_order_date` and before `tet_full_capacity_date`: arrival(X) = `tet_full_capacity_date` + `factory_days` + `transit_days`.
- Otherwise: arrival(X) = X + `factory_days` + `transit_days`.
- If the Tết dates are blank, always use "otherwise", and note in the report: "no Tết dates; the deadline may be wrong". Write this even when no current deadline depends on Tết.

**Inbound lots:**
- Inbound = lots in the source data, plus lots recorded in `decisions.md`.
- Drop a `decisions.md` lot once it appears in the source data, so it is not counted twice. It matches when the quantity is the same and the expected arrival is within 7 days. Mark it "in source data" in the Status column. If only one of the two matches, count the lot once and ask the user to confirm.
- Assume a stock count does not include an overdue lot, unless the source says it does. If the count is dated before the lot's expected arrival, it cannot include it: do not ask; only ask the user to confirm the lot was received.

**Input checks:**
- A blank field is not 0.
- Quantities must be 0 or more.
- `band_pct` must be above 0% and below 100%.
- `buffer_days` below 7: warn, because a weekly run can miss up to 6 days.
- An inbound lot past its expected date: add it to stock before the first simulated day, and flag it "overdue".
- An inbound lot with no date: leave it out and flag it.

**Data age:**
- Show the age of every input: today − the date the source was last updated.
- Mark an input "stale" if it is older than 8 days *[design parameter]*, but still produce the result. For stale stock, add: "stock may be overstated by sales since the count".
- If a source gives no update date, show "age unknown" and treat it as stale.

---

## Rule 1: day-by-day projection

Each simulated day, in this order:
1. add inbound arriving that day;
2. ship the waiting queue first;
3. then fill that day's new orders. If stock covers only part of them, fill that part; the rest is handled as described under "New orders that cannot be filled".

**Rate of new orders:**
- **Peak:** the average of the usable days among the last 14 calendar days, ending with today *[design parameter]*. The same peak rate applies in all three scenarios.
  - Note any day above twice the average *[design parameter]*.
  - With fewer than 7 usable days, use `peak_daily_orders`.
  - If that field is blank too, report "Not enough data: sales" and give no date.
- **Post-peak:** use `post_peak_daily_orders`.
  - Mid scenario = that value. Slow = −`band_pct`. Fast = +`band_pct`.
  - `band_pct` defaults to 40% *[design parameter]* and applies to post-peak only.

**Outputs of the projection.** Run the projection without any new lot:
- **Unavoidable shortage:** the shortage days before arrival(today). No new lot can fix them. Report the count. This includes days before an existing inbound lot arrives (example: sold out today, lot due in 10 days → about 9 unavoidable days).
- **Stock-out date for the new lot:** the first shortage day on or after arrival(today). If arrival(today) is itself a shortage day, the stock-out date is arrival(today), so no date keeps a buffer (Rule 3: "Past deadline").
- If there is no shortage day before `next_peak_start` (from arrival(today) on), the scenario has "Enough stock until next peak" (Rule 3). Give no date.

---

## Rule 2: deadlines

**Full-buffer deadline:** the latest day X, from today on, with arrival(X) ≤ stock-out date − `buffer_days`.
- This handles Tết automatically. If ordering after `tet_last_order_date` would arrive too late, the deadline becomes `tet_last_order_date`.

**Smaller-buffer deadline:** the latest X with arrival(X) ≤ stock-out date.
- Report it with the buffer days left = stock-out date − arrival(X). This is usually 0, but can be more when Tết pushes the next possible arrival later.
- Report it only when its date differs from the full-buffer deadline. When there is no full-buffer deadline, report it only if it is later than today (the Tết case: the last order date that still arrives on arrival(today)); otherwise leave it out.

**If the full-buffer deadline is missed:** report arrival(the day after it). If that arrival is not later than the stock-out date, report "no shortage" and the buffer days left (stock-out date − arrival). If it is later, report the shortage days = arrival − stock-out date (the arrival day itself is not short). Example: 21/6/2027 − 4/6/2027 = 17.

**If no X qualifies:** the status is "Past deadline". Report arrival(today), the buffer days left (can be 0), and any unavoidable shortage. If the factory is closed for Tết, also report the last order date that still arrives on arrival(today) (that is `tet_full_capacity_date`): waiting until then costs nothing, and each day after it adds one day of delay.

The buffer never multiplies the Tết closure.

Planning the lot for the **next peak season** is out of scope. The skill prints the reminder line instead (see SKILL.md).

---

## Rule 3: status and when to ask the CEO

**Evaluation order.** Go top to bottom and stop at the first condition that is true:
1. A required input is missing → **"Not enough data"**. Name the missing field. Do not ask the CEO; report it in the line only.
2. The status scenario has no shortage day before `next_peak_start` (from arrival(today) on) → **"Enough stock until next peak"**. Add the reminder line. Do not ask the CEO, unless the status scenario has unavoidable shortage.
3. A stock-out date exists but there is no full-buffer deadline → **"Past deadline"**.
4. Otherwise, count the days from today to the deadline that drives the status (see "Which deadline drives the status" below). The thresholds are *[design parameters]*:
   - 43 days or more → **"Time left"**;
   - 15–42 days → **"Approaching decision window"**;
   - 0–14 days → **"In decision window"**.

**Status rules:**
- **Which deadline drives the status:**
  - By default, the mid scenario's full-buffer deadline.
  - **Exception:** if the fast scenario's full-buffer deadline *is* `tet_last_order_date` (Tết is what sets it) *and* it is earlier than the mid deadline, use `tet_last_order_date` instead. This also applies when the mid scenario has no deadline because it has enough stock: the fast scenario then becomes the status scenario.
  - Why: missing the Tết cut-off cannot be undone, because the stock arrives weeks later. Deciding early only ties up cash. The cost of running short is much larger than the cost of ordering early.
  - The counting thresholds and the ask-the-CEO triggers below all use this deadline. "Status scenario" below means the scenario whose deadline drives the status.
  - `status_scenario` in `config.md`: "auto" (default) = the rule above. "slow", "mid" or "fast" = always use that scenario, with no exception.
  - If the fast scenario has no full-buffer deadline but the mid scenario does, the status still follows the mid scenario. Mention it in the line ("Fast scenario: already past its full-buffer deadline; earliest arrival [arrival(today)]"). This alone does not trigger an ask.
- **Unavoidable shortage:** if the status scenario has any, the status is never "Time left": when step 4 applies, it is at least "Approaching decision window". It is an ask trigger (see below). Show the shortage days next to any status, including "Enough stock until next peak". Unavoidable shortage in another scenario is shown in the line and the decision record, but does not by itself trigger an ask.
- **Dead band (3 days, *[design parameter]*):** move to a more urgent level at once. Move to a less urgent level only when the count is more than 3 days beyond the boundary: 19 days or more (back to "Approaching decision window", since 15 + 3 = 18) or 47 days or more (back to "Time left", since 43 + 3 = 46). Take last week's status from the Run log in `decisions.md`.

**Ask the CEO, only the first time each happens in a lot cycle, when:**
- the status scenario enters "In decision window" or "Past deadline";
- the status scenario has unavoidable shortage;
- a defect signal fires (Rule 5).

**End of a lot cycle:** the cycle ends when the CEO replies "decided" with a quantity and an expected arrival. That lot counts as inbound from the next run.

**Run log:** before working out the status, read the Run log in `decisions.md`: the last row for the dead band, and every row since the lot cycle began for the first-time rule. After each run, add one row. If the log is empty, this is the first run of the cycle.

---

## Rule 4: updating the post-peak rate

When there are 28 usable days after `peak_end` *[design parameter]*, propose replacing `post_peak_daily_orders` with the actual average. Skip the 3 days after each restock *[design parameter]*. Change the value only after the CEO approves. With fewer than 28 days, say nothing about this rule.

---

## Rule 5: defect signal

This rule runs only when customer-support data with delivery dates is available.

**How to measure:** by groups of units delivered in the same week.
- DOA (dead on arrival): count groups delivered at least 7 days ago.
- Defect returns or warranty claims: count groups delivered at least 30 days ago.
- Fire only when the counted groups hold at least 100 units and at least 5 defect cases *[design parameters]*.
- "Changed my mind" returns are not defects.
- A weekly group counts once its **last** delivery day is at least 7 (or 30) days before today.
- Rate = total cases ÷ total units of all counted groups (pooled, not a per-week average). Apply the 100-unit and 5-case minimums to each measure separately.

**When it fires:** the first time in a lot cycle that `doa_threshold` or `defect_threshold` is exceeded (check the Run log), tell the CEO: "worth reviewing before deciding the next lot". Summarise the reasons customers gave. Do not conclude a cause.

**Without the data:** the rule is off, and the line says "Defects not tracked."

**With the data:** the line says "Defects: below thresholds." or "Defect signal raised: see the note to the CEO."
A defect-only ask needs no decision record: send the note and the report line.

---

## Decision record

The record contains:
- the full-buffer deadline, the smaller-buffer deadline, the buffer days, and the unavoidable shortage days;
- **units needed** for the next lot;
- the result of waiting 14 days.

**Units needed for the next lot:**
- For each scenario, assume ordering on that scenario's own full-buffer deadline. If it has none, assume ordering today.
- Units needed = (expected new orders from arrival to `next_peak_start`) + (waiting queue left on arrival) − (projected stock on arrival) − (other inbound arriving between arrival and `next_peak_start`). Never below 0.
- Counting window: from the arrival day itself through the day **before** `next_peak_start`. Do not count `next_peak_start`.
- "On arrival" means the start of the arrival day, before that day's orders and before the new lot is added.
- Work it out for each scenario.
- If MOQ is larger, state the excess in units and in months of sales, using the mid post-peak rate.
- A scenario with 0 units needed is listed as "no lot needed in this scenario". If it is 0 in every scenario, say "no lot is needed before the next peak season" and skip the MOQ comparison.
- Rounding: units needed may come out as 12.0000001 because of decimals; round to 3 decimals first, then round up. Other inbound arriving on the arrival day counts as "other inbound", not as stock on arrival.
- Always state: "does not include stock for the next peak season".

**If we wait 14 days *[design parameter]*:** skip this when the status scenario has no stock-out date. Otherwise, for the status scenario, recompute with the order placed 14 days after the assumed order date above (its full-buffer deadline, or today if there is none). It is not today + 14. State the change in shortage days and in buffer days (stock-out date − arrival, not `buffer_days`), and whether the new order date is after `tet_last_order_date`.
