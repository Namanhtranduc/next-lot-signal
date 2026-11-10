# next-lot-signal config

Fill this in once and review it monthly. Leave a field blank if you don't know it yet. A blank field is never treated as 0.

## Data sources
| Data | Where the agent reads it |
|---|---|
| Sellable stock and its daily history | |
| Inbound lots: quantity and arrival date | |
| Accepted orders not yet shipped | |
| New orders per day, net of cancellations | |
| Promotion days (optional) | |
| Customer-support data with delivery dates (optional) | |

## Settings
| Field | Required? | Value | Meaning |
|---|---|---|---|
| `factory_days` | yes | | Days from placing the order until goods leave the factory |
| `transit_days` | yes | | Days from leaving the factory until stock is sellable |
| `buffer_days` | yes | | Safety buffer in days. Base it on the factory's or carrier's worst recent delay, not a round guess. Make it at least 7, because the skill runs weekly and the buffer has to cover the gap between two runs |
| `post_peak_daily_orders` | yes | | Expected new orders per day after the peak season |
| `peak_end` | yes | | Last day of the current peak season |
| `next_peak_start` | yes | | First day of the next peak season |
| `tet_last_order_date` | recommended | | Last order date for goods to leave the factory before Tết, as confirmed by the factory |
| `tet_full_capacity_date` | recommended | | Date the factory is back to full capacity after Tết |
| `peak_daily_orders` | no | | Fallback peak rate, used when there are fewer than 7 usable days of data |
| `preorder_promise_date` | no | | Ship date promised to pre-order customers |
| `band_pct` | no | 40% | Slow and fast scenarios: ± this % around the post-peak rate |
| `status_scenario` | no | auto | The scenario that drives the status. "auto" = mid, unless the fast scenario hits the Tết cut-off (see Rule 3) |
| `lost_or_wait` | no | lost | What happens to new orders placed while out of stock, outside the pre-order phase. Keep "lost" if the store stops selling at zero stock (Shopify's default). Use "wait" if the store keeps selling when out of stock |
| `moq` | no | | Minimum order quantity |
| `doa_threshold` | if defect tracking is on | | DOA rate that triggers the defect signal |
| `defect_threshold` | if defect tracking is on | | Defect return / warranty rate that triggers the defect signal |
| `as_of_date` | no | | For testing only: replaces "today" |


## Dashboard link
Filled in by the agent the first time it shows the dashboard as an artifact in Claude, so later runs update the same link. Clear it to start a new link.

| Field | Value |
|---|---|
| `artifact_url` | |
