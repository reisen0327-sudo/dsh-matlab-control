# Changelog

## 0.1.2 — 2026-11-14

Kernel compatibility re-verification for DeepSeek Harness `0.2.2-rc.3`.

- Verified on `0.2.2-rc.3`: the bundle mounts after re-enabling (`application: applied`), all 14
  `mcp__matlab__*` tools and the four skills appear, `model_check` returns `healthy`, and
  `pidtune`/`stepinfo` reproduce the earlier metrics (Kp=1.4316, Ki=2.5948, overshoot 6.08 %,
  settling 3.165 s).
- Confirmed the upgrade behaviour is repeatable: `0.2.2-rc.3` again rewrote `dsh.profile.bundles`
  and dropped this bundle (and several other third-party bundles) to `enabled: false`; no code or
  configuration change was needed to restore it.
- Locale metadata follows the newer bundle-details guidance: `meta.description` now states how the
  plugin is triggered and its prerequisites/limits, so the plugin list explains itself without the
  README fallback. No `meta.detail` or preview image is declared (optional for configuration-only
  bundles).
- READMEs: compatibility matrix extended with `0.2.2-rc.3`; the upgrade note now records that the
  bundle drop was reproduced on both `0.2.2-rc.1` and `0.2.2-rc.3`.
- No change to `cordis.patch.yml`, the skills, or the tool surface.

## 0.1.1 — 2026-11-06

Kernel compatibility re-verification for DeepSeek Harness `0.2.2-rc.1`.

- Verified on DSH `0.2.1-rc.5` and `0.2.2-rc.1`: the bundle mounts (`application: applied`),
  all 14 `mcp__matlab__*` tools and the four skills appear, `model_check` returns `healthy`,
  and `pidtune`/`stepinfo` produce unchanged closed-loop metrics
  (Kp=1.4316, Ki=2.5948, overshoot 6.08 %, settling 3.165 s).
- Documented in both READMEs: the plugin is configuration-only, declares no `engines.dsh` or
  peer-dependency gate on purpose, and is verified per kernel release instead of version-pinned.
- Documented the upgrade behaviour discovered on 0.2.2-rc.1: a DSH upgrade rewrites
  `dsh.profile.bundles`, and a third-party bundle that is not re-listed mounts as `enabled: false`
  (tools and skills silently disappear while the package stays installed). Recovery is
  `plugin_manager { action: "set_bundle", target: "dsh-matlab-control", enabled: true }`,
  which applies immediately and needs no reinstall.
- Added a troubleshooting row for that symptom in both READMEs.
- No change to `cordis.patch.yml`, the skills, or the tool surface.

## 0.1.0 — 2026-11-04

Initial release.

- Configuration-only bundle: `mcp-matlab` (`@deepseek-ai/dsh-mcp-client`, stdio) plus
  `matlab-control-skills` (`@deepseek-ai/dsh-skill-filesystem`).
- Simulink tool registry passed via the official Simulink Agentic Toolkit `--extension-file`;
  14 `mcp__matlab__*` tools (5 MATLAB + 9 Simulink).
- Four skills: `simulink-model-check`, `simulink-simulation-run`, `app-designer-ui`, `control-design`.
- English README with a linked Chinese translation, MIT license, original icon.
