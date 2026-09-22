---
id: lesson-0001-a5cdddaf8a344715bda570fafd2c5edb
type: lesson_learned
timestamp: 2026-09-22T11:11:15Z
author: 
tags: []
---

## Sirius diagram-open NullPointerException after capella-fabric patch (commit 8afe71b0)

**Symptom:** after pulling the state/transition description patch, opening `[MSM] TR88 PackML 7 state` in Capella throws:
```
java.lang.NullPointerException: Cannot invoke "org.eclipse.sirius.common.tools.api.interpreter.IInterpreter.evaluateBoolean(...)" because "acceleoInterpreter" is null
  at FilterService.checkExpression / isVisible / getAppliedFilters
  at RefreshDiagramOnOpeningCommand.doExecute
```

**Analysis:** the NPE happens inside Sirius's graphical-filter evaluation during diagram-open refresh, not in Capella's semantic model. This is a recognized Sirius interpreter-session-timing issue, not typically tied to model content. However, `apply_model_patch` (capella-fabric, backed by py-capellambse) writes directly to the model's XMI files rather than through Capella's own session/UI lifecycle — a write path that bypasses Sirius's normal session handling is a plausible (not yet confirmed) trigger if it touches files Sirius expects to control.

**Troubleshooting path given to the user, in order:**
1. Close/reopen the diagram tab (common fix for this exact race condition, unrelated to content)
2. Restart Capella and reopen the project
3. Check Eclipse Error Log for companion errors (isolates session-timing vs. actual file corruption)
4. Test whether the diagram opens cleanly on the commit before 8afe71b0, to confirm/rule out causality
5. If isolated to the patch, Cartenza can revert it via capella-fabric and redo descriptions differently (e.g. avoid HTML-wrapped text in description fields)

**Status:** unresolved / awaiting user's test results. If confirmed as a real capella-fabric write-path issue, this is worth flagging to Anthropic/Cartenza maintainers as a tooling gap — `apply_model_patch` on `description` fields of State/StateTransition objects may need to go through a path that keeps Sirius session state in sync.
