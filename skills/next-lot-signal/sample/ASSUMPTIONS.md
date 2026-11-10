# How the sample was built

The sample data in this folder is made up. But each number was chosen for a reason: some come from public industry benchmarks, some follow from other numbers, and some were picked so the sample shows the case that matters most. This page explains each one, so you can judge the logic and replace any number with your own.

**Labels:**
- **Benchmark:** taken from a public planning guide (sources at the end).
- **Derived:** worked out from other numbers on this page.
- **Design choice:** picked to make the sample show something specific.
- **Assumption:** a reasonable guess with no source. Replace it first with your own figure.

## The story the sample tells

A small hardware company sells one device through its own online store. A Chinese contract factory makes it, and lots ship to the US by sea. It is late October, in the middle of the first Christmas season. Stock looks comfortable, and many orders are still waiting to ship. The question is when to commit to the next production lot.

The sample is built so the answer is **not obvious**:
- At the expected sales rate after Christmas, the next decision is months away (April).
- If sales run faster, the factory's pre-Lunar-New-Year cut-off (1 December) becomes the real deadline, only 36 days away.

This is the trap the skill exists to catch. A simple "stock ÷ daily sales" check would miss it.

## Timing

| Setting | Sample value | Type | Why |
|---|---|---|---|
| `factory_days` | 70 | Benchmark | Electronics production typically takes 45–90 days because of component lead times and testing [2]. Printed circuit boards alone can take about 45 days when components are tight, so 70 sits mid-range. |
| `transit_days` | 30 | Benchmark | Ocean port-to-port to the US West Coast is about 14–30 days; customs and inland delivery add 5–15 more [2][3]. 30 days to sellable stock is the optimistic end. Use your forwarder's real door-to-door time. |
| Total lead time | 100 | Derived | 70 + 30. This is close to the 75–90 days often recommended as a planning horizon from order to sellable stock, plus some contingency [2]. |
| `buffer_days` | 20 | Design choice | It covers three things: the gap between two weekly runs (up to 7 days), common vessel delays of 5+ days, and customs variation of 5–10 days [2]. The skill warns if a buffer is below 7. |

## Lunar New Year

| Setting | Sample value | Type | Why |
|---|---|---|---|
| Lunar New Year 2027 | 6 Feb 2027 | Benchmark | Public calendar. The official holiday runs about 5–12 Feb [1]. |
| `tet_last_order_date` | 1 Dec 2026 | Derived | An order must finish production before the factory closes. 6 Feb 2027 − 70 factory days ≈ 28 Nov 2026, rounded to 1 Dec, the kind of round date a factory announces. Factories stop taking production 1–2 weeks before the holiday [1]. |
| `tet_full_capacity_date` | 13 Mar 2027 | Benchmark | Full output usually returns 4–6 weeks after the holiday, as workers come back slowly [1]. 13 March is about 5 weeks after. |

In real use, these two dates come from your factory, not from a rule of thumb. The skill drafts the question to ask the factory if they are missing.

## Demand

| Setting | Sample value | Type | Why |
|---|---|---|---|
| Stock on hand | 3,000 | Assumption | The size of a typical first production run for a small hardware startup. |
| Orders waiting to ship | 1,200 | Design choice | A large share of the first run is already promised to customers, which is common in a launch season with pre-orders. It also tests that the skill ships waiting orders before new ones, and does not count them twice. |
| Peak rate | 15/day | Design choice | Steady Christmas demand. At this rate the stock lasts through the season but not much beyond it. The 14 daily values vary between 13 and 17, so the skill's average and outlier checks have something to work on. |
| `post_peak_daily_orders` | 4/day | Assumption | Gift products sell far less after Christmas, so this is about a quarter of the peak rate. This is the number that moves the answer most. Replace it first. |
| `band_pct` (slow / fast) | ±40% | Design choice | With no sales history after a peak, the honest range is wide. ±40% gives 2.4 / 4 / 5.6 units a day. It is wide enough that the fast case crosses the Tết cut-off and the slow case lasts the whole year, so all three outcomes appear. |
| `peak_end` | 25 Dec 2026 | Assumption | End of the Christmas gifting season. |
| `next_peak_start` | 1 Nov 2027 | Assumption | Holiday shopping in the US starts in early November. The next peak is when the following season's stock must already be in place. |
| `moq` | 1,000 | Assumption | A plausible minimum for a small electronics run. Ask your factory. |
| `lost_or_wait` | lost | Benchmark | A store that stops selling at zero stock loses those sales rather than queueing them. That is Shopify's default setting. |

## What the sample deliberately leaves out

- **Inbound lots:** none. The decision depends only on current stock, so the logic is easy to follow. Other test cases cover inbound and overdue lots.
- **Defect tracking:** off. It is optional, and it needs customer-support data that a new company may not have.
- **Statistical safety stock (z-scores, standard deviations):** not used. A first-season product has no demand history to estimate variability from. A buffer in days, plus three explicit scenarios, is easier to check and explain.

## What the sample shows

| Scenario | Post-peak rate | Result |
|---|---|---|
| Slow | 2.4/day | Enough stock until the next peak |
| Mid | 4/day | Decide by 10 Apr 2027 to keep the 20-day buffer |
| Fast | 5.6/day | Decide by **1 Dec 2026**, because a later order would wait for the factory to reopen after Tết. If missed, the lot arrives about 21 Jun 2027, roughly 17 days short |

The status follows the 1 December date (36 days away: "Approaching decision window"). Missing the Tết cut-off cannot be undone, while deciding early only ties up cash. Full results are in `expected-output.md`.

## Sources

Planning guides from sourcing and logistics companies. They are not official data, so confirm every figure with your own factory and forwarder.

[1] Lunar New Year 2027 dates and factory closures:
- [DocShipper](https://china.docshipper.com/en/logistics/chinese-new-year-shipping-how-to-plan-china-orders-for-2027/)
- [New Buying Agent](https://www.newbuyingagent.com/resources/chinese-new-year-2027-february-6-the-china-production-schedule-survival-guide)
- [DHL](https://www.dhl.com/discover/en-ng/logistics-advice/import-export-advice/post-chinese-new-year-factory-restart)

[2] Electronics production and sea-freight lead times:
- [Alibaba.com seller guide](https://seller.alibaba.com/blogs/2026/southeast-asia/smart-products/production-lead-time-planning-guide-alibaba-b2b)
- [Alibaba.com sea-freight guide](https://seller.alibaba.com/blogs/2026/southeast-asia/electronics-components/sea-freight-delivery-time-planning-alibaba-guide)

[3] Ocean transit China to the US: [Send From China](https://www.sendfromchina.com/NewsCenter/what-is-transit-time.html)
