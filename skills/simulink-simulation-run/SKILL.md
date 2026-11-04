---
name: simulink-simulation-run
description: Run a Simulink simulation through MATLAB and return a structured interpretation of the logged signals instead of raw command output. Use when the user says 跑仿真 / simulate / 看响应 / run the model and tell me what happens.
whenToUse: The user wants simulation results, step-response behaviour from a model, or a check that a model runs without errors.
---

# Run and interpret a Simulink simulation

Simulation runs as MATLAB code through the bridged MATLAB tools; the model itself is never parsed directly.

## Workflow

1. Read the official guidance first:
   `<plugin>/thirdparty/simulink-agentic-toolkit/skills-catalog/model-based-design-core/simulating-simulink-models/SKILL.md`
2. Optionally validate structure with `mcp__matlab__model_check` (see the `simulink-model-check` skill) before a long run.
3. Execute with `mcp__matlab__evaluate_matlab_code`. Preferred pattern — one script, deterministic outputs, no interactive plotting:

```matlab
mdl = 'MyModel';
in  = Simulink.SimulationInput(mdl);
in  = in.setModelParameter('StopTime', '10', 'SaveOutput', 'on', 'SaveFormat', 'StructureWithTime');
out = sim(in);
% logged signals: out.yout (Dataset) or out.get('<ToWorkspaceVar>')
```

4. Collect, in the same call: `out.yout` element names, sample count, final value, min/max, steady-state error against a reference if one exists, overshoot and settling time for step-like responses, and the warning/error count from the run.
5. Never paste the raw MATLAB command echo. Return this summary:

```
Model / stop time : MyModel / 10 s
Signals           : <name>(<n samples>) ...
Final values      : <sig>=<v> ...
Range             : <sig>: [min, max]
Step metrics      : overshoot=<%>, settling(2%)=<s>, steady-state error=<v>
Run status        : completed in <t s>, warnings=<n>, errors=<n>
Notes             : <anything the user must fix>
```

If the simulation fails, return the MATLAB error identifier and message plus the smallest fix (missing variable, unconnected port, solver settings). Do not re-run blindly more than twice.

## Scope log data deliberately

- `logsout` / `yout` datasets: use `out.get(name)` or `out.yout.getElement(name)`.
- `To Workspace` blocks in `Array`/`Structure` format: read the variable captured from `out.<varname>`.
- Scope blocks alone are not queryable — ask for a `To Workspace` sink or add one via `mcp__matlab__model_edit` when the user wants numbers.
