---
id: dec-0015-d7efc986d1ba49eda980d4fd4bbb510e
type: decision
timestamp: 2026-11-23T02:07:59Z
last_updated: 2026-11-23T02:07:59Z
author: Cartenza (Claude), for Tony Komar
contributors: ["Cartenza (Claude), for Tony Komar"]
tags: []
---

## Descriptions approved and applied to logical actors, components and functions; Control Machines description restored

**Status:** Active

Tony Komar approved a proposed set of descriptions for the logical-layer elements ahead of the demo recording (NOTE-0010). They were applied to the model in one patch (capella-fabric commit fd379ab4, pushed to master). `verify_model` on LA reported no issues after the patch.

### Why
A description check on commit 3d4b44b found:
- States, transitions and exchange items were already described, with TR88 text that is consistent across all six state machines (Master, Infeed Load, Filling, Capping, Labeling, Case Packing).
- Components, actors and functions had no descriptions.
- `Control Machines` (Line Control's function, uuid 794c5d6f) carried the wrong text. It held the description of the Stopped-to-Idle transition, not the line-level fault-isolation description recorded in NOTE-0007 (commit caf38632). How it was replaced is unknown.

Scene 5 of the demo asks an agent to reason over the model, so the actors and control functions need descriptions that match the model.

### What was applied (28 elements)
- **4 actors:** Operator, Line Control, Bottle Pallet, Case Receiver.
- **8 components:** Logical System, Production Line, the Master TR88 template, and the five stations.
- **16 functions:** the six `Manage States` functions (identical text), Control Machine, Control Machines, and the nine material-flow functions.

Key statements now in the model:
- Operator is the only actor that can issue `clear`, the only way out of Aborted.
- Line Control suspends downstream machines when one aborts and unsuspends them after the faulted machine is cleared and back in Idle. It never issues `clear`.

### Caveats
- **`Control Machines` text is reconstructed, not recovered.** The original wording was not kept. It was rebuilt from NOTE-0007. Review it if the exact earlier wording mattered.
- **Not checked against the model after patching.** The patch status and `verify_model` were clean, but descriptions were not read back. Check them in Capella after refreshing.
- **"Downstream" is prose only.** The model does not record which machines are downstream of a fault. The behavior is stated only in the `Control Machines` description. An agent reasoning about it on camera is reading text, not structure.

### Observed, not changed
- Infeed Load's Line Control component exchange runs Line Control to station. The other four run station to Line Control.
- Infeed Load's functional exchanges are unprefixed (`Operator Control`, `Line Control`, `Machine Status`). The other stations use `<Station>.<Role>`.
- Two components differ from their exchange prefixes: "Labeling and Inspections Station" uses `LabelingStation.*`, and "Case Packing and Labeling" uses `CasePackingStation.*`.
- All six state machines are named "StateMachine 1". NOTE-0010 refers to the diagram `[MSM] TR88 PackML 7 state`.

### Reminder
Refresh the local Capella copy from the remote before editing. This change landed on master from outside Capella (see DEC-0014).

**Reference:** NOTE-0007 (Line Control fault-isolation description), NOTE-0010 (video script), DEC-0004 (state and transition descriptions), DEC-0012 (control-plane wiring complete), DEC-0014 (previous model check and naming fix), BCFG-0001.
