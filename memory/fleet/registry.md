# Fleet registry

Every agent the fleet runs in this open-source CompanyOS, with the person who answers
for it. The foreman
reads this before handing out work; an agent with no row here does not exist.

| Agent | Owner | Job | Connector actions | Budget | Since |
|---|---|---|---|---|---|
| foreman | Replace with a person | Splits a job, hands the parts to the worker, lands the result as a change request | reads only | 20 sessions/week, none | 2026-11-02 |
| worker | Replace with a person | Does one job end to end and writes its run record | reads only (`read-only` policy set) | 40 sessions/week, none | 2026-11-02 |
| auditor | Replace with a person | Writes the weekly audit from the run records | none | 4 sessions/week, none | 2026-11-02 |

## Notes

- `Owner` is one person's name. A row with a placeholder is an agent nobody has
  accepted yet: fill it before the first session.
- `Budget` is sessions per week and a spend ceiling. `none` means no ceiling
  was agreed, which is a decision to make, not a permission to spend.
- Adding an agent is three edits and one change request — see the
  `fleet-registry` skill.
