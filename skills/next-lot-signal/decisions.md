# Lot decisions

The agent writes a row here when the CEO replies "decided, X units, expected arrival Y". The lot counts as inbound until it appears in the source data.

Status: blank = waiting to appear in the source data; "in source data" = already counted from the source, do not count again; "reported, already on order" = a lot the user mentioned in conversation (counts as inbound, but does not start a new lot cycle).

| Decided on | Units | Expected arrival | Status |
|---|---|---|---|

## Run log
The agent adds one row after every run. The next run reads the last row for the dead band, and every row since the lot cycle began so that each trigger is raised only once per cycle. A new lot cycle starts after the last decided row above (not after a "reported, already on order" row). If the user's `decisions.md` has no Run log, add this table and treat the run as the first of the cycle. Write "—" where there is no deadline.

| Run date | Status scenario | Status | Days to driving deadline | CEO asked? (trigger) | Defect signal raised? |
|---|---|---|---|---|---|
