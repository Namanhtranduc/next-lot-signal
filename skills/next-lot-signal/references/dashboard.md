# Dashboard

Every run also produces a one-page dashboard: the status, the report line, a chart of projected stock for the three scenarios, the scenario table, inbound lots and data age. It shows the same numbers as the report line, drawn as a picture. It is illustrative and based on the data entered, like the report line.

The viewer can point at the chart to read any day, hide or show a scenario, highlight one, and mark a key date. As an artifact in Claude it also has an "Ask Claude" box: Claude explains the numbers on the page and never works out new dates (a "what if" needs a new run of this skill). The page does all of this itself from the data object below; you only fill the data.

## How to make it
1. Do it inside the same throwaway calculation as the projection (SKILL.md, step 2). Build the data object below **from the computed values in code**. Never type the numbers by hand.
2. Read `assets/dashboard.html` from the skill folder. Replace only the text between `<script id="nls-data" type="application/json">` and the next `</script>` with the data object as JSON. Change nothing else in the template.
3. Write the result as `next-lot-dashboard.html` in the folder named in the request (`sample/` included), overwriting last week's file. Never write it at the skill root, in `assets/` or in `references/`.
4. Check: read the JSON back from the written file. Confirm that `status`, `driving_deadline` and `days_left` match what you will report, that `report_line` is character for character the line in your answer (`**` included), and that each scenario's dates and day counts match the numbers in your answer. If you change the report line or any number after writing the dashboard, write the dashboard again.
5. Show it. The file on disk is always kept; showing it is an extra step.
   - **Ask before showing anything.** End the answer with the path and one question. In a scheduled or unattended run, never show or publish it; just give the path.
   - **If you have an Artifact tool** (a tool that publishes a page as a private artifact in Claude), ask: *"Dashboard saved: `<folder>/next-lot-dashboard.html`. Show it as a private artifact here in Claude? The page, with these numbers, is stored in your Claude account, and it has a box where you can ask Claude about them (each question uses your own Claude usage)."* On a yes:
     - Publish the saved file exactly as it is, with that tool's plain publish call and `file_path` set to it. It is already a finished, self-contained page: do not rewrite, restyle or rebuild it, and do not start from an artifact template or type instead. Give it the icon `chart` and the title "Next-lot check".
     - On every publish, declare the capability that lets the page ask Claude: `capabilities: {"sample": {}}`. The page's "Ask Claude" box appears only with it. Declare nothing else. The page's code is already written; if your Artifact tool asks you to read a guide first, read it, but still publish the file unchanged.
     - Look in the folder's `config.md`, section *Dashboard link*. If `artifact_url` holds a link, publish to that link (the tool's `url` field), so the user keeps one link that is updated each week. If the tool refuses because it has not read that artifact in this session, read it with the tool, then publish the new file to the same link. If the link no longer works, publish a new artifact.
     - Write the link the tool returns into `artifact_url`, and give it to the user. Never make the artifact public or share it; that is the user's choice.
   - **If you have no Artifact tool**, or the publish fails, ask instead: *"Dashboard saved: `<folder>/next-lot-dashboard.html`. Open it in your web browser now?"* On a yes, open it in their default web browser with the operating system's own command:
     - Windows: in PowerShell `Start-Process "<path>"`, or in cmd `start "" "<path>"`;
     - macOS: `open "<path>"`;
     - Linux: `xdg-open "<path>"`.
   - **Do not open the file in an app's side panel, file viewer or preview pane.** Those show HTML without running its script, so the page shows only dashes and a note. The page draws its numbers and charts with a small script; an artifact and a real browser both run it. In a browser the page works the same, without the "Ask Claude" box.
   - If neither works (for example on a remote machine), say so and tell the user to double-click the file.

   Put this after the report line, never inside it.

Make the dashboard for every status, including "Not enough data". In that case, set `series` to null, `scenarios` to `[]`, and fill `missing`.

The page builds its own labels, key-date list and next-peak reminder from these fields. Do not repeat them in `notes`.

## Data object
Dates are `yyyy-mm-dd`, since the page writes them as d/m/yyyy. Use `null` for anything that does not apply. Never use 0 to mean "not applicable".

```json
{
  "title": "Next-lot check",
  "as_of": "today (or as_of_date)",
  "status": "one of the six statuses, exactly as in the report line",
  "status_scenario": "slow | mid | fast | null",
  "driving_deadline": "the deadline that drives the status, or null",
  "days_left": "days from today to driving_deadline, or null",
  "driver_reason": "12 words or fewer, one of: \"a later order would hit Tết\" | \"keeps the full [n]-day buffer\" | \"no order date keeps the full buffer\" (add \"; the Tết cut-off has passed\" when it has) | \"no stock-out before the next peak in this scenario\" | null for Not enough data",
  "report_line": "the report line, word for word",
  "ask_ceo": "true if a note to the CEO is attached this run",
  "missing": ["missing input names; an empty list unless the status is Not enough data"],
  "stock_now": "starting stock used in the projection (on hand, plus overdue or today's inbound)",
  "queue_now": "starting waiting queue (accepted orders not yet shipped)",
  "settings": {
    "buffer_days": 20, "factory_days": 70, "transit_days": 30, "moq": 1000,
    "peak_end": "…", "next_peak_start": "…",
    "tet_last_order_date": "… or null", "tet_full_capacity_date": "… or null"
  },
  "scenarios": [
    {
      "name": "slow | mid | fast",
      "post_peak_rate": 2.4,
      "stockout_date": "stock-out date for the new lot (Rule 1), or null",
      "first_shortage_date": "first shortage day with no new lot, counting unavoidable shortage days; null if none before next_peak_start",
      "full_buffer_deadline": "… or null",
      "days_left": "days to full_buffer_deadline, or null",
      "smaller_buffer_deadline": "only when it differs from the full-buffer deadline, else null",
      "smaller_buffer_days": "buffer days left at the smaller-buffer deadline, or null",
      "if_missed_arrival": "only when this scenario has a full-buffer deadline: arrival(the day after it); else null",
      "if_missed_shortage_days": "only with if_missed_arrival: shortage days for that arrival (0 if none); else null",
      "order_now_arrival": "only when this scenario has a stock-out date but no full-buffer deadline: arrival(today); else null",
      "unavoidable_shortage_days": 0,
      "units_needed": "from the decision record when one is made this run (0 = no lot needed in this scenario), else null",
      "note": "only \"Enough stock until next peak\" or null"
    }
  ],
  "series": {
    "start": "first simulated day (the day after today)",
    "stock": { "slow": [], "mid": [], "fast": [] },
    "queue": { "slow": [], "mid": [], "fast": [] }
  },
  "inbound": [ { "qty": 500, "arrival": "…", "flag": "overdue | in source data | no date | \"\"" } ],
  "data_age": [ { "input": "Stock | Inbound | Open orders | Daily new orders | Customer support (one entry each, no file names)", "updated": "… or null if the source gives no date", "days": 2, "stale": false } ],
  "defects": [ { "measure": "DOA | defect returns", "cases": 15, "units": 400, "rate_pct": 3.75, "threshold_pct": 3, "fired": true } ],
  "notes": ["see the list below"]
}
```

**`notes`:** only these, in this order, and nothing else (no method notes, no data age, no field names; dates as d/m/yyyy):
1. the Tết sentence from the report line, word for word, if there is one;
2. the defect sentence from the report line, word for word;
3. one line for each thing the user must act on: fill in the pre-order promise date (when it is blank and a pre-order phase was assumed or orders are marked as pre-orders), send the drafted factory question. Overdue lots are shown by the page itself.

Example: `["Tết does not set any of these dates.", "Defects not tracked.", "Please fill in the pre-order promise date."]`

**`defects`:** one entry per measure computed under Rule 5; an empty list when there is no customer-support data.

**`series`:** for each scenario, the stock and the waiting queue at the **end** of each simulated day, after steps 1–3 of Rule 1. Give one value per day, from `start` through the day before `next_peak_start`. Round to 1 decimal. Inbound that is overdue or arrives today is already in the starting stock. The page draws stock minus waiting queue, so pre-order backlogs show below 0.

List the scenarios in the order slow, mid, fast.
