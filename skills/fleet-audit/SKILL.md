---
name: fleet-audit
description: Write the run record for a job, then the weekly audit from those records. Use when a job finishes and someone has to be able to say later what the agents did, what it cost and who decided.
---

# Fleet audit

In an open-source CompanyOS a run record is what makes a fleet auditable after the fact. It is written at
the end of every job, by the agent that did the work, and it names files rather
than impressions.

## The run record

`runs/<yyyy-mm-dd>-<slug>.md`:

```
# <job> — <yyyy-mm-dd>

Agent: <name from memory/fleet/registry.md>   Owner: <person>
Asked by: <person>   Change request: <number or none>

## What ran
- <step>: <file it wrote or action it took>

## Connector actions
| Action | Result |
|---|---|
| warehouse.get_stock | ran |
| warehouse.update_stock | held for approval, decided by <person> at <time> |
| warehouse.delete_row | blocked by policy |

## Files changed
- <path> — <one line on what changed>

## What a person still has to decide
- <decision, with the file it lives in>
```

## The weekly audit

`runs/audit-<yyyy-mm-dd>.md`, written from the records of the last seven days:
what ran, what needed a person, what it cost, and at most three things to
change. Every item names an agent and its owner.

## Rules

- This file is an audit, so a missing number is `not recorded` and never a
  reconstruction from a total or an average.
- An action with no result recorded is `not recorded`, not `ran`. The
  difference is the whole point of keeping the record.
- Count a hold once per action. An approval that was withdrawn and asked again
  is two decisions, and the person who made them can tell you why.
- The record is committed with the work it describes, in the same change
  request. A record written later is a story about the work, not evidence of it.
