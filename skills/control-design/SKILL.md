---
name: control-design
description: Route a control-engineering request to the right MATLAB analysis (Bode/Nyquist/margins, step/impulse/lsim, ss/tf/zpk, pidtune/pidTuner, robuststab/robgain) and return closed-loop metrics such as gain/phase margin, settling time, overshoot, and stability. Use for PID 整定, 频域分析, 时域响应, 状态空间建模, 鲁棒性, or validating a controller in Simulink.
whenToUse: The user asks to design, tune, analyse, or verify a controller or a plant model.
---

# Control science and engineering

First classify the intent, then call only the matching MATLAB functions. Every numeric claim must come from a real tool result.

| Intent | MATLAB functions |
|---|---|
| Frequency domain | `bode`, `nyquist`, `margin`, `allmargin` |
| Time domain | `step`, `impulse`, `lsim`, `stepinfo` |
| State space / models | `ss`, `tf`, `zpk`, `ss2tf`, `minreal` |
| PID tuning | `pidtune`, `pidtuneOptions`, `pidTuner` (interactive — prefer `pidtune` when the agent must return values) |
| Robustness | `robuststab`, `robgain`, `diskmargin` |
| Closed-loop verification | `feedback`, `sim`, Simulink `model_test` |

## Workflow

1. Restate the plant (numerator/denominator, state-space matrices, or model file) and the design target in the reply so the user can catch a misread.
2. Run the analysis in **one** `mcp__matlab__evaluate_matlab_code` call that computes the model, the controller, and the metrics together:

```matlab
G  = tf(1, [1 1]);
C  = pidtune(G, 'PID');          % or 'PI','PD' per the target
L  = C*G;  T = feedback(L, 1);
S  = allmargin(L);
st = stepinfo(T);
fprintf('Kp=%.4g Ki=%.4g Kd=%.4g\n', C.Kp, C.Ki, C.Kd);
fprintf('Gm=%.2f dB Pm=%.2f deg\n', S.GainMargin, S.PhaseMargin);
fprintf('OS=%.2f%% Ts=%.3f s SSE=%.4g stable=%d\n', ...
    st.Overshoot, st.SettlingTime, abs(1-dcgain(T)), isstable(T));
```

3. Return the structured block, not just a code dump:

```
Plant            : G = 1/(s+1)
Controller       : Kp=..., Ki=..., Kd=...
Gain margin      : <dB> at <rad/s>
Phase margin     : <deg> at <rad/s>      (wcp)
Closed-loop      : stable | unstable
Overshoot        : <%>
Settling (2%)    : <s>
Steady-state err : <value>
Robustness       : robuststab/robgain result when requested
Plot             : offer bode/step figures, do not block on them
```

4. If the controller must be validated in Simulink, use the bridged Simulink tools: implement/parameterise via `mcp__matlab__model_edit`, then confirm with `mcp__matlab__model_test` or a `sim()` run inside `evaluate_matlab_code`, and report the same metrics measured from the model. Before writing tests, read
   `<plugin>/thirdparty/simulink-agentic-toolkit/skills-catalog/verification-validation-and-test/testing-simulink-models/SKILL.md`, and for plant definitions `<plugin>/thirdparty/simulink-agentic-toolkit/skills-catalog/model-based-design-core/specifying-plant-models/SKILL.md`.
5. If a toolbox is missing (`pidtune` needs Control System Toolbox, `robuststab` needs Robust Control Toolbox), detect it with `mcp__matlab__detect_matlab_toolboxes` and report the exact missing product instead of a generic error.

## Guardrails

- Never report a stability claim from an unstable/ill-conditioned computation without checking `isstable` and the condition number (`cond`).
- Distinguish open-loop and closed-loop margins explicitly.
- For discrete-time designs state the sample time; for MIMO plants state which loop was analysed.
