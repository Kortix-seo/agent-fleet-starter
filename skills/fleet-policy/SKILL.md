---
name: fleet-policy
description: Decide and record which connector actions run, which wait for a person and which never run, and keep that list true to the manifest. Use before an agent makes a connector call, or when someone asks why a call was held.
---

# Fleet policy

The fleet's policy in this open-source CompanyOS is the list of what its agents may do
to the systems around them. Kortix applies it on every connector call, in this order: the project's
`policies`, then the calling agent's `policy_sets`, then the connector's own
base rules. The first rule that matches decides.

## The three answers

- **Run** (`always_run`): the call happens at once.
- **Wait for a person** (`require_approval`): the call is held until someone
  approves or rejects it, and it expires after `approvals.ttl`.
- **Never** (`block`): the call does not happen. A block rule still wins when
  the call is finally run.

## Writing a rule

Rules live in `kortix.yaml`; this file is the human-readable copy and the
reason each rule exists.

| Rule | What it protects |
|---|---|
| `warehouse.*delete*` → block | Nothing in a fleet deletes a record unattended. |
| `warehouse.update_stock` → require approval | Stock changes move money downstream, so a person sees each one. |
| Reads (`get_*`) → run | Reading is how an agent finds out; holding it stops work for no gain. |

## What a hold looks like

An approval covers one exact action. If a detail changes — an amount, a
recipient, a row — it is a new call and a new approval. There is no "approve
everything for this session"; an action that must run with nobody present needs
a rule that says so, and that rule is a decision someone signs.

## Rules

- A rule is written where the person who owns it can find it, and it says why.
  A policy nobody can explain gets loosened at 2am and never tightened again.
- Tighten first, loosen later: start a new agent on reads, add writes after a
  week of runs, and never widen a rule for a one-off.
- When a hold fires more than once for the same action, that is a decision the
  fleet has not made yet, not a nuisance. Record it in the run record.
