# agent-fleet-starter: an open-source agent fleet you can read

This repository is a working, open-source agent fleet on Kortix, small enough to read end to end: three agents, one registry of who owns what, one policy set of connector rules, and a run record committed for every job. Kortix is the open-source CompanyOS: your agents, skills, memory, and connectors in one repo you own.

An agent fleet is a management problem before it is a model problem. The models are rented, and they get better on someone else's schedule. What a company actually owns is the register of which agent does which job, who answers for it, what it may do to the systems around it, and the record of what it did. Those four things are files here, and this repository ships all four written out.

Open source. Read it, fork it, audit it. Session work reaches `main` through a change request. Merge is default-deny for agents. Approval gates you set, off until you set them. Every action is recorded, and reading, exporting and streaming the audit log depend on the plan.

## What is in it

| File | What it gives you |
|---|---|
| `kortix.yaml` | The manifest: `default_agent: foreman`, three agents, one connector with a read-only policy set, the project's connector rules and the approval time-to-live |
| `agents/foreman.md` | The primary agent: reads the registry, splits a job, hands parts to the worker, lands the result as a change request, never merges |
| `agents/worker.md` | The subagent that does one job end to end and writes its run record |
| `agents/auditor.md` | The subagent that writes the weekly audit from the run records |
| `memory/fleet/registry.md` | One row per agent: owner, job, connector actions, budget |
| `memory/fleet/policy.md` | The rules in plain words: what runs, what waits for a person, what never runs, and why |
| `memory/MEMORY.md` | The index the agents read first |
| `skills/fleet-registry/SKILL.md` | How an agent is added: three edits and a change request, with an owner named before it ever runs |
| `skills/fleet-policy/SKILL.md` | How a connector rule is written, and what a held call looks like |
| `skills/fleet-audit/SKILL.md` | The run record and the weekly audit, in the shape a person can read later |
| `.gitignore` | `.kortix/`, `node_modules/`, `.DS_Store` |

The idea behind each file is on the campaign's site: what an agent fleet is at all is answered in [what is an agent fleet](https://runanagentfleet.com/what-is-an-agent-fleet), the day-one setup is in [how to run an agent fleet](https://runanagentfleet.com/how-to-run-an-agent-fleet), and the ongoing job — the register, the policies, the approvals — is in [agent fleet management](https://runanagentfleet.com/agent-fleet-management). This repository is the artifact behind those pages, and [the open-source route](https://runanagentfleet.com/open-source-agent-fleet) is why it is a repository and not a dashboard. If you are still choosing where to run the fleet, [the platforms compared](https://runanagentfleet.com/agent-fleet-platforms-compared) puts the products the answers recommend side by side on the one question that outlives a subscription: where the definition of an agent and its policy actually live.

## The manifest, and the command that checks it

The manifest is the whole configuration. Every grant is explicit: an agent gets a skill, a connector or a permission only when its block names it.

```yaml
policy:
  default_mode: risk          # a read runs; a write or a delete waits for a person

policies:
  - match: "warehouse.*delete*"
    action: block             # nothing in the fleet deletes unattended
  - match: "warehouse.update_stock"
    action: require_approval

approvals:
  ttl: 4h                     # a held call expires instead of waiting forever
```

`kortix validate` reads the manifest and prints what each agent enables. This is its output for the manifest in this repository, run on 2026-11-02 with Kortix CLI `v0.14.14`:

```text
  ✓  kortix.yaml is valid

What each agent enables:
  foreman  skills=[fleet-registry, fleet-policy]  subagents=[worker, auditor]  commands=[none (default-deny)]  extensions=[none (default-deny)]  connectors=[warehouse]  secrets=[none (default-deny)]
  worker  skills=[fleet-policy, fleet-audit]  subagents=[none (default-deny)]  commands=[none (default-deny)]  extensions=[none (default-deny)]  connectors=[warehouse]  secrets=[none (default-deny)]
  auditor  skills=[fleet-audit]  subagents=[none (default-deny)]  commands=[none (default-deny)]  extensions=[none (default-deny)]  connectors=[none (default-deny)]  secrets=[none (default-deny)]
```

The `none (default-deny)` columns are the point. An omitted grant grants nothing, so `worker` cannot start a subagent and `auditor` cannot call a connector at all. Widen one of them and the diff shows exactly what you widened.

## Quickstart

Install the CLI and sign in. It supports macOS and Linux, and Windows through WSL.

```sh
curl -fsSL https://kortix.com/install | bash
kortix login
```

Clone the project, then work inside the clone. The clone carries the manifest, so the agent sees the files you do.

```sh
kortix projects clone <project_id> agent-fleet-starter
cd agent-fleet-starter
```

Fill in `memory/fleet/registry.md` before the first session: every `Replace with a person` becomes a name. Then rename the connector or point it at your own service — `base_url` in `kortix.yaml` is a placeholder (`https://api.example.com/v1`), and the credential lives in Kortix, never in this file.

Start a session with the job you actually have:

```sh
kortix sessions new --prompt "Take the weekly stock exception report from the warehouse, write it under runs/, and open a change request"
```

The foreman reads `memory/fleet/registry.md` first, so it knows which agent owns the work. The worker reads `memory/fleet/policy.md` before its first connector call, so a write is held for approval instead of happening. A session needs a model: any provider with your own keys, or the ChatGPT plan you already pay for.

Read the diff, then merge it once the run record reads the way you would write it.

```sh
kortix cr ls
kortix cr diff 1
kortix cr merge 1
```

`1` is the change request number `kortix cr ls` printed. Session work reaches `main` through a change request, and an agent cannot merge its own without a merge grant that a person wrote into the manifest.

## The weekly audit

The `triggers:` block is the only scheduled thing here. It fires at `cron: "0 7 * * 1"` in `UTC`, Monday morning, and starts the `auditor` agent with a prompt rather than waiting to be asked. The auditor reads every run record from the last seven days, then `git log --since="7 days ago" --stat`, and writes `runs/audit-<yyyy-mm-dd>.md`: what ran, what needed a person, what it cost, and at most three things to change. The report is a file in the repository, so next Monday's audit can read this one.

Every count in it comes from a run record. A number no record holds is written `not recorded` — the report is an audit, and an audit that estimates is a story.

## Make it your fleet

Change the registry before you change anything else. It decides who answers for which agent, and a row with no owner is an agent nobody has accepted yet. Then write the real rules in `memory/fleet/policy.md` and mirror them in `kortix.yaml`; the manifest is what Kortix enforces, and the file is what a person reads.

Adding an agent takes three edits and one change request: its block in `kortix.yaml`, its prompt under `agents/`, and its row in the registry with an owner named. Start it read-only and widen it after a week of runs that show you why. The `fleet-registry` and `fleet-policy` skills carry the rest of the rules.

Where Kortix ends and other surfaces begin is a fair question: [an agent fleet against an agent control plane](https://runanagentfleet.com/agent-fleet-vs-agent-control-plane) takes it on directly, and the answer is what this repository demonstrates — the control plane is the product, and the fleet is what you keep.

## Where to read more

- Kortix documentation, `kortix.yaml` reference: https://kortix.com/docs/developers#kortixyaml-reference
- The guide, on agents, connectors and approvals: https://kortix.com/docs/guide
