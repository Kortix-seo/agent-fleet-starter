# Fleet memory

The facts and rules this fleet works from, kept as files in an open-source CompanyOS
the company owns. Read this file first, then the files it points at.

## The fleet

| File | Holds |
|---|---|
| `memory/fleet/registry.md` | One row per agent: owner, job, connector actions, budget. |
| `memory/fleet/policy.md` | Which connector actions run, wait for a person, or never run, and why. |

## Where work lives

| Folder | Holds |
|---|---|
| `agents/` | One prompt file per agent, the text the session loads. |
| `skills/` | One folder per skill, each with a `SKILL.md`. |
| `runs/` | One run record per job, and the weekly audit. |

## How work lands here

Every change arrives on a session branch and lands through a change request. A
person reads the diff and merges it. That is the only path to `main`, and it is
the same rule for the agents in this repository and the people who own them.
