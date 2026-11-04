---
name: simulink-model-check
description: Audit a Simulink model's structure and explain its architecture. Use when the user says 检查模型 / 看看这个模型有什么问题 / model health / why does the build or compile fail, or before editing a model you have not seen.
whenToUse: The user points at a .slx/.mdl file and asks whether it is OK, what is wrong with it, or how it is structured.
---

# Simulink model structure check

All Simulink access goes through the bridged MathWorks MCP tools (`mcp__matlab__*`). Never open, unzip, or parse `.slx` bytes yourself.

## Tool routing

| Question | Tool call |
|---|---|
| What is in this model? | `mcp__matlab__model_overview(model=<path>, scope='root', detail='tree')` |
| Ports and signal flow | `mcp__matlab__model_overview(..., detail='full')` then `mcp__matlab__model_read(model=<path>, scope='root', depth=<int>)` |
| Structural health | `mcp__matlab__model_check(model=<path>, scope='root', checks='["all"]')` |
| Compile/runtime diagnostics | `mcp__matlab__model_read_diagnostics(model=<path>, severity=...)` |
| Parameter values | `mcp__matlab__model_query_params(model=<path>, targets=..., params=...)` |
| Behavioural test | `mcp__matlab__model_test(...)` (requires Simulink Test) |

`checks` is a JSON *string*: `'["all"]'` or a subset of `unconnected_ports`, `unconnected_lines`, `stateflow_lint`.

## Workflow

1. Resolve the model to an absolute path **inside the session workspace**; refuse paths outside it.
2. `model_overview` with `detail='tree'` — establishes subsystem hierarchy and block counts.
3. `model_check` with `checks='["all"]'` — this is the authoritative structural verdict (unconnected ports, dangling lines, Stateflow edit-time lint).
4. If issues exist, call `model_read` for the affected scope and translate each block ID into its block path and a one-line cause. Do not invent issues the tool did not report.
5. Return a report with this shape:

```
Model      : <name> (<n> blocks, <m> subsystems)
Verdict    : PASS | <k> issues
unconnected_ports : <count>
unconnected_lines : <count>
stateflow_lint    : <count>
Issues     : severity | block path | message   (one row per issue)
Next action: <concrete edit or "none">
```

6. Report `0` only when the tool returned an empty issue list. If a tool call errors, report the error text and the likely cause — never silently claim the model is clean.

## When a fix is needed

Load the official Simulink workflow before editing:

- `<plugin>/thirdparty/simulink-agentic-toolkit/skills-catalog/model-based-design-core/building-simulink-models/SKILL.md` — model construction and rewiring rules.
- `<plugin>/thirdparty/simulink-agentic-toolkit/skills-catalog/model-based-design-core/simulink-sanitize-model/SKILL.md` — cleanup of saved models.
- `<plugin>/thirdparty/simulink-agentic-toolkit/skills-catalog/model-based-design-core/managing-simulink-projects/SKILL.md` — project/path handling.

`<plugin>` is the directory that contains this skill's package (the bundle directory you installed). Apply edits only through `mcp__matlab__model_edit`, then re-run `model_check` to prove the fix.
