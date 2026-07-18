# cloud-itonami-isco-8155

Open Occupation Blueprint for **ISCO-08 8155**: Fur and Leather Preparing
Machine Operators.

This repository designs a forkable OSS business for a fur and leather
preparing plant scheduling and logistics coordination practice: a plant
scheduling and supply-coordination robot manages crew/task records under a
governor-gated actor, so a fur and leather preparing machine-operator crew
keeps its own operating records instead of renting a closed
workforce-management SaaS.

**Maturity: `:implemented`.** `src/furleathercoord/` implements the
`FurLeatherCoordActor` as a `langgraph.graph/state-graph`
(`furleathercoord.actor`) wired to a `Fur and Leather Preparing Plant
Scheduling Coordination Advisor` (`furleathercoord.advisor`) and an
independent `FurLeatherCoordGovernor` (`furleathercoord.governor`),
following the itonami actor pattern (ADR-2607121000): `:intake -> :advise
-> :govern -> :decide -+-> :commit (:ok? true) +-> :request-approval
(:escalate? true, human-in-the-loop interrupt) +-> :hold (:hard? true)`.
HARD invariants (always hold, never overridable): operator provenance,
plant provenance, no-actuation (`:effect` must be `:propose`), a closed
op-allowlist (`:log-work-record`, `:schedule-crew-operation`,
`:flag-safety-concern`, `:coordinate-supply-order` — nothing else may ever
be proposed), and a permanent, unconditional block on any proposal that
would directly finalize a processing-operation-execution decision (e.g.
deciding to proceed with a specific tanning/dressing machine run) or a
chemical-safety-clearance decision (e.g. declaring the plant safety cleared
for chemical processing), or that would override a plant safety officer's
judgment. Always-escalate paths (human sign-off regardless of confidence,
mapping this repo's Trust Controls in
[`docs/business-model.md`](docs/business-model.md)): `:flag-safety-concern`
(always) and `:coordinate-supply-order` above the registered cost
threshold.

## Robotics premise

All cloud-itonami verticals are designed on the premise that a **robot performs
the physical domain work**. Here a plant scheduling/logistics coordination
robot performs crew scheduling, production-run/inventory/progress-record
logging and processing-chemicals/raw-hide supply-order coordination for a
fur and leather preparing machine-operator crew, under an actor that
proposes actions and an independent **Fur and Leather Preparing Plant
Scheduling Coordination Governor** that gates them. The governor never
dispatches hardware itself, never operates fur/leather preparing
(tanning/dressing) machinery on the plant floor, and never finalizes a
processing-operation-execution decision or a chemical-safety-clearance
decision, and never overrides a plant safety officer's judgment;
`:high`/`:safety-critical` actions (such as a flagged chemical-exposure/
heavy-machinery/equipment-condition concern, or an above-threshold supply
order) require human sign-off. **This actor coordinates PLANT SCHEDULING/
LOGISTICS ONLY — it never operates fur/leather preparing machinery itself,
and it never makes a chemical-safety-clearance decision itself.**

Fur and Leather Preparing Machine Operators run tanning/dressing machinery
(fleshing, tanning-drum, dyeing, splitting and staking machines) using
chemical processing agents, alongside heavy-machinery hazard (crush/
entanglement from rollers and drums). This is a real chemical-exposure and
heavy-machinery hazard domain; this actor never operates that equipment and
never clears it as safe — it only schedules and logs around it, and always
routes chemical-exposure/machinery-hazard/equipment-condition concerns to a
human plant safety officer.

## Core Contract

```text
crew roster + plant registration + safety-reporting policy
        |
        v
Fur and Leather Preparing Plant Scheduling Coordination Advisor -> FurLeatherCoordGovernor -> log/schedule/coordinate, or human sign-off
        |
        v
robot actions (gated) + operating records + audit ledger
```

No automated advice can dispatch a robot action the governor refuses,
finalize a processing-operation-execution decision, finalize a
chemical-safety-clearance decision, override a plant safety officer's
judgment, suppress an operating record, or disclose sensitive data without
governor approval and audit evidence.

## Capability layer

Resolves via [`kotoba-lang/occupation`](https://github.com/kotoba-lang/occupation)
(ISCO-08 `8155`). Required capabilities:

- :robotics
- :identity
- :audit-ledger

See [`docs/business-model.md`](docs/business-model.md) and
[`docs/operator-guide.md`](docs/operator-guide.md).

## License

AGPL-3.0-or-later.
