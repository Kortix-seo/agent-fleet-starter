# Fleet policy

The human-readable copy of the rules in this open-source CompanyOS's `kortix.yaml`. Kortix applies these on
every connector call; when this file and the manifest disagree, the manifest
wins and this file is the one to fix.

## Decisions

| Rule | Action | Why |
|---|---|---|
| `warehouse.*delete*` | never runs | Nothing in the fleet deletes a record unattended. |
| `warehouse.update_stock` | waits for a person | A stock change moves money downstream. |
| `warehouse.get_*` | runs | Reading is how an agent finds out. |
| any other call no rule matches | a read runs, a write or a delete waits | The project default. |

A held call expires after 4 hours (`approvals.ttl: 4h`). An approval covers one
exact action: a changed amount, row or recipient is a new call and a new
approval.

## The worker's set

The `worker` agent gets the `read-only` policy set on the `warehouse`
connector: reads run, everything else is blocked. A policy set can loosen a
connector's base rule for its agent; it can never loosen a project rule above.

## Changing a rule

A rule change is a change request with a line in the reason: which rule, what
fired it, and who decided. Loosening a rule without that line is how a fleet
ends up with permissions nobody can explain.
