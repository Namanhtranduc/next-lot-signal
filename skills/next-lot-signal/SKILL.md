---
name: next-lot-signal
description: Weekly check that tells a hardware founder the latest date to decide the next production lot while keeping a safety buffer, including the factory cut-off before Lunar New Year (Tết / CNY), and how many days of stock-out a missed date would cause, with a one-page dashboard. Use it whenever someone asks when to reorder, when to place or commit to the next production run, whether Tết or Chinese New Year affects the next lot, whether a longer factory or component lead time (for example PCB) moves the decision date, how long current stock will last, how many units are needed until the next peak season, or wants a weekly stock-and-reorder line in the morning report, even if they do not name this skill. Covers one product's next lot; it does not plan the peak-season lot or set safety stock across many SKUs or warehouses.
---

# next-lot-signal

**Question it answers:** *"What is the latest date to decide the next production lot and still keep the buffer, including any pre-Tết cut-off? And if that date is missed, how many days will we be out of stock?"*

**Language:** answer in English by default. If the user writes in Vietnamese, answer in Vietnamese, including the setup questions and the report line. If the user asks for a language, use that one. The note to the CEO follows the same language unless the user asks otherwise. The dashboard's labels stay in English; its report line follows the answer's language.

The agent prepares the decision; a person decides. It never places orders or contacts suppliers. Every result is "based on the data you entered".

## Try it in 2 minutes
About 2 minutes of your time, plus about 3–5 minutes while the agent works and fills the dashboard.
1. Install it (the install command in the repo README, or copy this folder to `.claude/skills/next-lot-signal/`), then start a new session.
2. Type `/next-lot-signal sample` (or ask "run next-lot-signal on its sample data"). The agent uses the `sample/` folder inside this skill folder, wherever it is installed.
3. Compare the dates and day counts with `sample/expected-output.md`. If they match, the rules are being applied correctly; wording may differ.
4. Say yes when the agent offers to show the dashboard, as an artifact in Claude or in your web browser. The file is also saved as `sample/next-lot-dashboard.html`. It shows the same numbers as a chart and a table; point at the chart to read a day. As an artifact, it also has a box to ask Claude about the numbers.

## What the agent reads each week
These come from the sources listed in `config.md` (for example, orders from Shopify, and stock and inbound from the warehouse file or system). If a source is not listed, the agent can find Shopify exports in the folder itself, and the user can simply tell it about lots on order (`references/setup.md`).
- **Required:**
  - sellable stock, with its daily history;
  - inbound lots (quantity and arrival date);
  - accepted orders not yet shipped (the waiting queue);
  - new orders per day, net of cancellations.
- **Optional:**
  - promotion days;
  - customer-support data with delivery dates (for the defect signal).

**Which files to use:** take `config.md`, the data and `decisions.md` from the folder named in the request (for example `sample/`). A request for just `sample` means the `sample/` folder inside this skill folder. The `config.md` and `decisions.md` at the skill root are empty templates: never run on them and never write to them. If no folder is named, or the folder has no `config.md` / `decisions.md` yet, follow `references/setup.md`: it creates them from the templates and can fill the settings by asking the user a few questions. Values the user filled in by hand always win.

## How to run it
1. Read `config.md`, the data sources, and both tables in `decisions.md` (Lot decisions and Run log), all from the folder named in the request. If required settings are blank, data sources are not listed, or the user mentions a lot on order, follow `references/setup.md` first. Check the inputs and the data age, as described in `references/rules.md` (Definitions).
2. Apply Rules 1–5 in `references/rules.md` for each scenario (slow / mid / fast). Do the day-by-day projection with a short throwaway calculation in your own code tool, not by mental arithmetic. Dates are easy to get wrong by hand. Do not save that calculation into the repo.
3. Work out the status in the order given in Rule 3.
4. Write the **report line** for the morning report (format below).
5. Make the **dashboard** as described in `references/dashboard.md`: fill `assets/dashboard.html` from the same calculation, save it as `next-lot-dashboard.html` in that folder, give its path, and ask whether to show it as a private artifact in Claude, or, without an Artifact tool, to open it in the user's web browser (show it only on a yes).
6. Only if a Rule 3 trigger fires for the first time in this lot cycle (check the Run log): ask the CEO, and attach the decision record (format below; not needed for a defect-only ask).
7. If the Tết dates are missing or out of date, draft a short question to the factory for a person to send. Do not send it yourself.
8. Add one row to the Run log in that folder's `decisions.md`.

## Report line
A short paragraph. Items 1–3 always come first, in at most 3 sentences; items 4–6 follow only when they apply, each as a short phrase or sentence. In this order:
1. the status;
2. the deadline that drives the status, with days left and why;
3. if it is missed: the earliest arrival and the shortage days;
4. each other scenario in one short phrase, in the order slow, mid, fast (skip the status scenario), marked "(does not drive the status)": its full-buffer deadline if different, "already past its full-buffer deadline; earliest arrival [arrival(today)]", or "no stock-out before [next_peak_start]", with no extra clauses;
5. unavoidable shortage, if any, word for word: "Unavoidable shortage: [scenario(s), or all scenarios], about [n] days out of stock ([from] to [to]), which no new order can prevent." Skip the status scenario here if sentences 1–3 already state its shortage;
6. in this order: "Pre-order phase assumed until [date] (promise date blank)." if one was assumed, then data age, the Tết sentence, and the defect line ("Defects not tracked." / "Defects: below thresholds." / "Defect signal raised: see the note to the CEO.").

Tết sentence, word for word:
- Tết sets a date: "Tết sets the [scenario] date: the factory's last order date is [date] and it is back to full capacity on [date]."
- Tết dates known but setting none of the dates: "Tết does not set any of these dates."
- Past deadline because the Tết closure pushes arrival(today): "Tết closure is why no date is left in the [scenario] scenario: the factory's last order date was [date] and it is back to full capacity on [date]."
- Tết dates missing: "No Tết dates; the deadline may be wrong."

Data age: write "stock updated today" when the age is 0 days, otherwise "stock updated [n] days ago".

Bold only the deadline that drives the status, with `**…**`. When there is none, use no bold.

Keep the tone neutral: no colours, no alarm words. Write dates as d/m/yyyy.

Example shapes (the bracketed parts come from the calculation):

> *Approaching decision window. Nearest date: **[date]** ([n] days) if sales follow the fast scenario, because a later order would hit Tết. If missed, the earliest arrival is about [date], roughly [n] days short. Mid scenario (does not drive the status): decide by [date] to keep the full [buffer]-day buffer. Based on the data you entered; stock updated [n] days ago. Defects not tracked.*

> *Past deadline. The Tết cut-off ([date]) has passed and no order date keeps the [buffer]-day buffer. Ordering now arrives about [date]; ordering any day until [date] arrives the same day, because the factory is closed for Tết. Mid scenario: about [n] days out of stock before then, which no order can prevent. ...*

> *Enough stock until next peak. Mid scenario: no stock-out before [next_peak_start]. Fast scenario (does not drive the status): decide by [date] to keep the full [buffer]-day buffer. ...*

When the status is "Enough stock until next peak", always end the report line with: *"The lot for the next peak season (from `next_peak_start`) is not covered by this skill and needs its own plan."*

## Note to the CEO (only when a trigger fires)
One short note for a person to send: the trigger, the report line, and the decision record (not needed for a defect-only ask). Example openings: *"Next-lot check, [date]: the decision window for the next lot has opened. Details below, based on the data you entered."* or *"Next-lot check, [date]: the full-buffer date for the next lot has passed, and some stock-out days can no longer be avoided. Details below, based on the data you entered."*

## Factory question (only when the Tết dates are missing or out of date)
Draft for a person to send: *"Could you confirm, for the coming Tết: (1) the last order date for goods to leave the factory before Tết, and (2) the date you are back to full capacity after Tết? Thank you."*

## Decision record (only when asking the CEO about stock)
If the user asks how many units are needed, give the units-needed part of the record even when the CEO is not asked.

- The full-buffer deadline, and the smaller-buffer deadline (with the buffer days left).
- Unavoidable stock-out days, if any.
- **Units needed** until `next_peak_start` for each scenario, compared with MOQ. Always state: "does not include stock for the next peak season".
- **What changes if we wait 14 days** after the assumed order date (not after today): the change in shortage days and buffer days, and whether waiting crosses the pre-Tết cut-off. Skip it when the status scenario has no stock-out date.

When the CEO replies "decided, X units, expected arrival Y", record it in that folder's `decisions.md`. Count that lot as inbound from the next run, and start a new lot cycle.

## Files
All paths are relative to the skill folder.
- `references/rules.md`: the full rules, statuses, edge cases and formulas. Read it before every run. It is the source of truth when this page and the rules differ.
- `references/setup.md`: setting up a data folder by asking the user, finding Shopify exports, and recording lots told in conversation.
- `references/dashboard.md`: how to fill the dashboard, and its data format.
- `assets/dashboard.html`: the dashboard template (one self-contained page, no network). Do not change its layout.
- `config.md` (skill root): empty settings template. Copy it next to your data, or let the agent do it.
- `decisions.md` (skill root): empty template for the lots the CEO has decided and the Run log. Copy it next to your data, or let the agent do it.
- `sample/`: illustrative config, data, its own `decisions.md`, and the expected output.
- Test cases live outside the skill, in a separate `evals` folder, so installing the skill does not copy them.
