# Expected output for `sample/` (illustrative)

Worked out day by day from the rules. If the dates and day counts match, the rules are being applied correctly; wording may differ.

## Key numbers per scenario
| Scenario | Post-peak rate | Stock-out date for new lot | Full-buffer deadline | Days left | Notes |
|---|---|---|---|---|---|
| Slow | 2.4/day | none before 1/11/2027 | — | — | Enough stock until next peak (156 units left on 31/10/2027) |
| Mid | 4/day | 8/8/2027 | 10/4/2027 | 166 | Smaller-buffer deadline 30/4/2027 (0 days buffer) |
| Fast | 5.6/day | 4/6/2027 | **1/12/2026** (Tết cut-off) | 36 | If missed: earliest arrival 21/6/2027, 17 days short |

- **Deadline that drives the status:** 1/12/2026. The fast scenario's full-buffer deadline is the Tết cut-off and is earlier than the mid deadline (10/4/2027), so the exception in Rule 3 applies.
- **Status:** "Approaching decision window" (36 days). The 3-day dead band does not matter here, since the Run log is empty (first run).
- **Ask the CEO?** Not yet. 36 days is outside the 0–14 day decision window, and there is no unavoidable shortage. The CEO is asked once the count reaches 14 days or fewer.

## Expected report line
> Approaching decision window. Nearest date: **1/12/2026** (36 days) if sales follow the fast scenario, because a later order would hit Tết. If missed, the earliest arrival is about 21/6/2027, roughly 17 days short. Slow scenario (does not drive the status): no stock-out before 1/11/2027. Mid scenario (does not drive the status): decide by 10/4/2027 to keep the full 20-day buffer. Based on the data you entered; stock updated 2 days ago. Tết sets the fast date: the factory's last order date is 1/12/2026 and it is back to full capacity on 13/3/2027. Defects not tracked.

## Sample decision record
Shown here only to illustrate the format. The CEO would not be asked on this run.
- **Mid:**
  - Full-buffer deadline: 10/4/2027.
  - Smaller-buffer deadline: 30/4/2027 (0 days buffer).
  - Arrival if ordered 10/4/2027: 19/7/2027, with 80 units still in stock.
  - Units needed from 19/7/2027 through 31/10/2027 (105 days): 420 − 80 = **340**.
  - MOQ is 1000, so the excess is 660 units, about 5.4 months at 4 per day.
- **Fast:**
  - Deadline: 1/12/2026.
  - Arrival: 11/3/2027, with 480 units still in stock.
  - Units needed from 11/3/2027 through 31/10/2027 (235 days): 1316 − 480 = **836** (after rounding to 3 decimals, as in the rules).
  - MOQ is 1000, so the excess is 164 units, about 1.3 months at 4 per day.
- **If we wait 14 days (fast, the status scenario; 14 days after 1/12/2026, not after today):** waiting moves the order past the Tết cut-off. Arrival becomes 21/6/2027, giving 17 days of shortage instead of 0, and the buffer drops from 85 days to none.
- Does not include stock for the next peak season.
