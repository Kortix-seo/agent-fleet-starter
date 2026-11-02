---
name: fleet-registry
description: Keep the fleet's register of agents, owners, jobs and budgets as one file. Use when an agent is added, retired or changes owner, or when the foreman needs to know which agent may do what.
---

# Fleet registry

In an open-source CompanyOS, an agent fleet is a management problem before it is a model problem. The
management lives in one file, `memory/fleet/registry.md`, and it answers four
questions for every agent: who owns it, what job it does, which connector
actions it may take, and what it may spend.

## The row

One row per agent, in this order:

| Column | What goes in it |
|---|---|
| Agent | The name in `kortix.yaml`. |
| Owner | One person's name. Not a team, not a shared inbox. |
| Job | What it produces, as a file. |
| Connector actions | The actions it may take without asking, if any. |
| Budget | Sessions per week and a spend ceiling, or `none`. |
| Since | The date the row was added. |

## Adding an agent

Adding an agent is three edits and one change request:

1. The agent's block in `kortix.yaml` (prompt, grants, connector policy set).
2. Its prompt file under `agents/`.
3. Its row here, with an owner named before the agent ever runs.

No row, no agent: an agent without an owner cannot be held responsible for
anything, and an incident with no owner becomes nobody's.

## Retiring an agent

Set `enabled: false` in its block, keep its row and add a `Retired` date with
the reason. Deleting the row deletes the only record that the agent ever
existed, which is exactly what an audit needs to keep.

## Rules

- One owner per agent. An agent two people answer for is an agent nobody
  answers for.
- A new agent starts with no write access. Widen it after the first week of
  runs, in a change request that says what went wrong without it.
- The registry and the manifest name the same agents. When they drift, fix the
  registry, not the memory of the person reading it.
