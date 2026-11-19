# dsh-matlab-control

[![Listed on dsh-plugin.org](https://dsh-plugin.org/badges/listed.svg)](https://dsh-plugin.org/plugins/reisen0327-sudo/dsh-matlab-control)


**MATLAB/Simulink control-engineering assistant for DeepSeek Harness (DSH).**
English | [中文](README.zh.md)

This plugin does not implement a MATLAB/Simulink transport. It reuses the official
**MathWorks MATLAB MCP Server** as the bridge and the official **Simulink Agentic Toolkit**
tool registry for model tooling, then adds four agent skills that specialise the workflow for
Simulink simulation, App Designer, and control-system design.

- 14 model-facing tools: 5 MATLAB (`evaluate_matlab_code`, `run_matlab_file`, `check_matlab_code`,
  `run_matlab_test_file`, `detect_matlab_toolboxes`) + 9 Simulink
  (`model_overview`, `model_read`, `model_edit`, `model_check`, `model_query_params`,
  `model_resolve_params`, `model_read_diagnostics`, `model_scan`, `model_test`).
- All tools appear to the agent as `mcp__matlab__<tool>`.
- Domain knowledge lives in skill files, not in a system prompt.

## Features

| Area | What the agent does |
|---|---|
| **Simulink simulation** | Reads model structure (`model_overview`, `model_read`), validates connectivity and Edit-Time checks (`model_check`), runs simulations and reports signal/metric summaries instead of raw command output |
| **App Designer / GUI** | Generates modern `uifigure`/`uiaxes`/`uidropdown`/`uibutton` app classes, wires callbacks that call `sim()` and plot into `uiaxes`, and verifies them with `check_matlab_code` |
| **Control engineering** | Routes an intent to `bode`/`nyquist`/`margin`/`allmargin`, `step`/`impulse`/`lsim`, `ss`/`tf`/`zpk`, `pidtune`/`pidTuner`, `robuststab`/`robgain`, and returns gain/phase margin, overshoot, settling time, steady-state error and stability as a structured summary |

## How it works

`cordis.patch.yml` is the entire runtime — two rows, no JavaScript:

| Row | Package | Purpose |
|---|---|---|
| `mcp-matlab` | `@deepseek-ai/dsh-mcp-client` | Launches the MathWorks MATLAB MCP Server over stdio and registers its tools as `mcp__matlab__*` |
| `matlab-control-skills` | `@deepseek-ai/dsh-skill-filesystem` | Registers `skills/` as a skill source for every agent session |

The Simulink tool surface comes from the official
[Simulink Agentic Toolkit](https://github.com/matlab/simulink-agentic-toolkit) registry, passed to
the same MCP server with `--extension-file`. Neither the toolkit nor the MCP server binary is
bundled in this repository; both are MathWorks deliverables and stay under their own terms.

## Requirements

- **MATLAB R2023a or later** with **Simulink** (the toolkit requires R2023a+; tested with R2024b).
- **MATLAB MCP Server** binary, v0.13.0 or later. The MATLAB Agentic Toolkit installs it at
  `<user>\.matlab\agentic-toolkits\bin\matlab-mcp-server.exe` on Windows.
- **MATLAB MCP Server Toolbox**, installed once per MATLAB release:
  `matlab-mcp-server.exe --setup-matlab`.
- Optional products: Control System Toolbox (`pidtune`, `allmargin`), Simulink Test
  (`model_test`), Robust Control Toolbox (`robuststab`, `robgain`).

## Installation

1. **Get the official Simulink Agentic Toolkit** into the plugin directory (not tracked here):

   ```bash
   git clone --depth 1 https://github.com/matlab/simulink-agentic-toolkit.git \
     thirdparty/simulink-agentic-toolkit
   ```

2. **Point the bundle at your machine.** Edit these values in `cordis.patch.yml`:

   | Field | Set it to |
   |---|---|
   | `command` | absolute path of `matlab-mcp-server.exe` |
   | `--matlab-root=` | your MATLAB installation folder (the one that contains `bin`), no trailing `\bin` |
   | `--extension-file=` | `<plugin dir>/thirdparty/simulink-agentic-toolkit/tools/tools.json` |
   | `customSkillDirs` | `<plugin dir>/skills` |

3. **Install the bundle into your DSH profile:**

   ```text
   plugin_manager { action: "install_bundle", target: "<absolute path to this directory>" }
   ```

   A local-directory install links the checkout instead of copying it, so the absolute paths from
   step 2 stay valid as long as the directory does not move.

4. Start a new session; the `mcp__matlab__*` tools and the four skills are then available.

### Session mode

The example configuration omits `--matlab-session-mode`, which selects the MathWorks default
`auto`: attach to a MATLAB session that ran `shareMATLABSession()`, otherwise start one. To bind
strictly to your desktop MATLAB session, add `--matlab-session-mode=existing` to `args` after
running `shareMATLABSession()` there.

## Compatibility

| Component | Tested versions |
|---|---|
| DeepSeek Harness kernel | `0.2.1-rc.5`, `0.2.2-rc.1`, `0.2.2-rc.3`, `0.2.2-rc.5` |
| MATLAB | R2024b (24.2.0.2712019) |
| Simulink, Control System Toolbox | R2024b |
| MATLAB MCP Server | v0.13.0 |
| Simulink Agentic Toolkit registry | 1.0.32 |

The bundle deliberately declares **no** `engines.dsh` range and **no** `@deepseek-ai/dsh-*`
peer dependencies: it is configuration-only (two Loader rows over stable packages), so the kernel's
compatibility gate never skips it. Verify behaviour after a kernel upgrade instead of pinning a range:

```text
plugin_manager { action: "list_bundles" }                                    # enabled?
plugin_manager { action: "set_bundle", target: "dsh-matlab-control", enabled: true }
```

A DSH upgrade rewrites `dsh.profile.bundles`. Third-party bundles that are not re-listed are mounted
as `enabled: false`, so the tools and skills disappear from the session even though the package is
still installed. This happened on `0.2.2-rc.1`, `0.2.2-rc.3` and `0.2.2-rc.5`; re-enabling takes
effect immediately and needs no reinstall.

`0.2.2-rc.5` was re-verified by configuration/manifest recomposition: installing this checkout into
a throwaway profile on the rc.5 kernel auto-selected the bundle, and
`dsh --profile <name> --dump-config` composed both rows and every config key. Live mounting on a
desktop profile after re-enabling remains for the user to confirm.

## Skills

| Skill | Triggers on | Does |
|---|---|---|
| `simulink-model-check` | "check this model", "what's wrong with it" | `model_overview` → `model_check` → per-issue explanation and a verdict |
| `simulink-simulation-run` | "run the simulation", "show the response" | `Simulink.SimulationInput` + `sim()` via `evaluate_matlab_code`, structured signal and metric summary |
| `app-designer-ui` | "generate .mlapp code", "build a UI" | App class with modern UI components, `sim()` callback plotting into `uiaxes`, verified with `check_matlab_code` |
| `control-design` | "tune a PID", "Bode plot", "state space", "robustness" | Intent routing to the right analysis functions, structured closed-loop metrics |

Each skill body names the official `SKILL.md` to read before acting; see
[`reference/official-skills-index.md`](reference/official-skills-index.md). The toolkit's own
skills are not copied into this repository — read them from your clone.

## Example prompts

```text
Check this Simulink model and tell me what is wrong with it: <path>.slx
Run demo_plant for 10 s and summarise overshoot, settling time and final value.
Generate a small App Designer app that runs a model and plots the result in a uiaxes.
Tune a PID for tf(1,[1 1]) and verify the closed loop in Simulink.
```

## Repository layout

```
dsh-matlab-control/
├── cordis.patch.yml                 # the whole bridge + skill registration
├── package.json                     # bundle manifest (dsh.bundle.patch)
├── icon.svg, locale/{en,zh}.json    # plugin-list title, description, icon
├── skills/<name>/SKILL.md           # four specialisation skills
├── reference/official-skills-index.md
├── README.md, README.zh.md
└── LICENSE
```

`thirdparty/` (your toolkit clone) and local test artifacts are ignored by git.

## Troubleshooting

| Symptom | Cause / fix |
|---|---|
| First tool call takes 1–3 minutes | The MCP server is starting MATLAB. Later calls are fast. |
| `model_*` tools report "function not found" | The toolkit is not on the MATLAB path for this session. Run `addpath(genpath('<plugin>/thirdparty/simulink-agentic-toolkit'))` in MATLAB (`evaluate_matlab_code` also works). |
| `pidtune` / `robuststab` undefined | Missing Control System / Robust Control Toolbox. Check with `detect_matlab_toolboxes`. |
| No `mcp__matlab__*` tools after install | The MATLAB MCP Server binary path in `cordis.patch.yml` is wrong, or the MCP Server Toolbox is not installed for that MATLAB release. |
| Tools and skills vanished after a DSH upgrade | The upgrade dropped this bundle from `dsh.profile.bundles`; re-enable it (see [Compatibility](#compatibility)) — the package itself is unaffected. |
| `application: restart-required` after install | DSH hot-applies the rows; restart once to clear the flag and load the bundle through the normal plugin list. |

## Status and roadmap

Kernel compatibility was re-verified on DeepSeek Harness `0.2.2-rc.5` by configuration
recomposition; the live tool surface with MATLAB R2024b, Simulink and Control System Toolbox was
last confirmed on `0.2.2-rc.3`. This is a minimal first version: model search/batch processing,
report export and multi-session MATLAB pooling are intentionally not included yet.

## License

MIT — see [LICENSE](LICENSE). The MATLAB MCP Server, the MathWorks MATLAB MCP Server Toolbox and the
Simulink Agentic Toolkit are MathWorks software and are **not** covered by this license; they are
distributed under their own terms and are not bundled here.
