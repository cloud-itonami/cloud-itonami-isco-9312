# cloud-itonami-isco-9312

Open Occupation Blueprint for **ISCO-08 9312**: Civil Engineering Labourers.

This repository designs a forkable OSS business for an independent civil engineering labour crew lead: a heavy-lifting exoskeleton and site-prep robot assists with material handling and site clearing under a governor-gated actor, so the crew keeps its own safety and inspection records instead of renting a closed site-management SaaS.

## Robotics premise

All cloud-itonami verticals are designed on the premise that a **robot performs
the physical domain work**. Here a heavy-lifting exoskeleton and site-prep robot assists with material handling and site clearing under an actor that proposes
actions and an independent **Civil Labour Governor** that gates them. The governor never
dispatches hardware itself; `:high`/`:safety-critical` actions (such as
operating near excavations, heavy equipment or other workers on-site) require human sign-off.

A live sample of the operator console (robotics safety console, shared template) is rendered in [docs/samples/operator-console.html](docs/samples/operator-console.html) — pure-data HTML output of `kotoba.robotics.ui`.

## Core Contract

```text
work order + site safety brief + inspection checklist
        |
        v
Site Advisor -> Civil Labour Governor -> labor/clear-site, or human sign-off
        |
        v
robot actions (gated) + operating records + audit ledger
```

No automated advice can dispatch a robot action the governor refuses, suppress
an operating record, or disclose sensitive data without governor approval and
audit evidence.

## Capability layer

Resolves via [`kotoba-lang/occupation`](https://github.com/kotoba-lang/occupation)
(ISCO-08 `9312`). Required capabilities:

- :robotics
- :forms
- :telemetry
- :dmn
- :bpmn
- :audit-ledger

See [`docs/business-model.md`](docs/business-model.md) and
[`docs/operator-guide.md`](docs/operator-guide.md).

## License

AGPL-3.0-or-later.
