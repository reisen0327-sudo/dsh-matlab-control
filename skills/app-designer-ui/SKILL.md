---
name: app-designer-ui
description: Generate MATLAB App Designer / programmatic-UI code (uifigure, uiaxes, uidropdown, uibutton) and wire callbacks that simulate a Simulink model and plot results into uiaxes. Use when the user asks for .mlapp code, 界面, 开个窗口, a control panel, or a GUI around a model.
whenToUse: The user wants an app, dashboard, or interactive UI for MATLAB/Simulink work, or asks to review/repair existing App Designer code.
---

# App Designer / GUI generation

## Hard rules

- Use **modern UI components**: `uifigure`, `uipanel`, `uigridlayout`, `uiaxes`, `uidropdown`, `uibutton`, `uislider`, `uieditfield`, `uilabel`, `uitable`. Never use legacy `figure`, `axes`, `subplot`, `guide`, `uicontrol`, or `waitforbuttonpress` in generated code.
- Produce a **code-only app class** (`classdef MyApp < matlab.apps.AppBase`) saved as `MyApp.m`. `.mlapp` is a binary container; a text class file is reviewable, diffable, and satisfies the same requirement. Say so explicitly when the user asked for `.mlapp`, and give the one-line packaging step (`appdesigner` / "Save As .mlapp") if they want the binary.
- Every callback that launched work must report failure in the UI (`uialert` or a status label), never only to the command window.
- Keep one `uigridlayout` as the layout root; size the `uiaxes` by the layout, not by absolute pixels.

## Callback + Simulink pattern

```matlab
function RunButtonPushed(app, ~)
    app.StatusLabel.Text = "Running...";
    drawnow
    try
        in = Simulink.SimulationInput(app.ModelDropDown.Value);
        in = in.setModelParameter("StopTime", string(app.StopTimeField.Value));
        out = sim(in);
        y = out.yout.getElement(1).Values.Data;
        t = out.yout.getElement(1).Values.Time;
        plot(app.UIAxes, t, y, LineWidth=1.5);
        grid(app.UIAxes, "on");
        app.StatusLabel.Text = sprintf("Done: %d samples, final=%.4g", numel(y), y(end));
    catch err
        uialert(app.UIFigure, err.message, "Simulation failed");
        app.StatusLabel.Text = "Failed: " + err.identifier;
    end
end
```

`sim()` must be called with a `Simulink.SimulationInput` when the app changes model parameters, so the base workspace is not mutated.

## Workflow

1. Read the official App Building guidance first:
   - `<user home>\.matlab\agentic-toolkits\matlab\skills-catalog\matlab-app-building\matlab-build-app\SKILL.md` — layout archetypes, component lifecycle, `.mlapp` vs `.m` decision.
   - `...\matlab-app-building\matlab-build-chart\SKILL.md` for chart/axes wiring, `...\matlab-apply-theme\SKILL.md` for styling.
   - Interaction patterns: the **App Building** category of <https://github.com/matlab/prompts> (MathWorks' official prompt collection).
2. Confirm the model path and its logged outputs (see `simulink-simulation-run`).
3. Write the class to the workspace (`.m`), then verify with `mcp__matlab__check_matlab_code(script_path=<abs path>)` and fix every reported syntax error. Static-analysis warnings should be explained, not ignored.
4. Optionally smoke-test construction in MATLAB with `app = MyApp; delete(app)` inside `evaluate_matlab_code` and report the result.
5. Return: file path, component tree, callback list, verification result (tool output), and how to launch it.

## Do not

- Do not scaffold `guide`-style `.fig` files or `uicontrol` layouts.
- Do not leave placeholder callbacks that do nothing.
- Do not plot inside the base workspace without a `uiaxes` target.
