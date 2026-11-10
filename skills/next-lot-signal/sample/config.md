# Sample config (illustrative, not real company figures)

All values below are made up to show how the skill works.

## Data sources
| Data | Where |
|---|---|
| Sellable stock and its daily history | `data.md` (this folder), section Stock |
| Inbound lots | `data.md` (this folder), section Inbound |
| Accepted orders not yet shipped | `data.md` (this folder), section Open orders |
| New orders per day | `data.md` (this folder), section Daily new orders |
| Promotion days | none |
| Customer-support data | none (defect tracking off) |

## Settings
| Field | Value |
|---|---|
| `factory_days` | 70 |
| `transit_days` | 30 |
| `buffer_days` | 20 |
| `post_peak_daily_orders` | 4 |
| `peak_end` | 2026-12-25 |
| `next_peak_start` | 2027-11-01 |
| `tet_last_order_date` | 2026-12-01 |
| `tet_full_capacity_date` | 2027-03-13 |
| `peak_daily_orders` | (blank) |
| `preorder_promise_date` | (blank) |
| `band_pct` | 40% |
| `status_scenario` | auto |
| `lost_or_wait` | lost |
| `moq` | 1000 |
| `doa_threshold` | (off) |
| `defect_threshold` | (off) |
| `as_of_date` | 2026-10-26 |


## Dashboard link
Filled in by the agent the first time it shows the dashboard as an artifact in Claude, so later runs update the same link. Clear it to start a new link.

| Field | Value |
|---|---|
| `artifact_url` | |
