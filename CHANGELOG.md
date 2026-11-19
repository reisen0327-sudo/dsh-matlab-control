# Changelog

## 0.1.3 — 2026-11-20

Kernel compatibility re-verification for DeepSeek Harness `0.2.2-rc.5`.

- Re-verified the two Loader rows against the rc.5 package sources. `@deepseek-ai/dsh-mcp-client`
  still accepts every key the patch sets — `transport: stdio`, `serverName`, `command`, `args`,
  `toolCallTimeoutMs`, `maxInstructionBytes`, `failOnStartupError` (`src/index.ts`, Config stdio
  branch) — and `@deepseek-ai/dsh-skill-filesystem` still accepts `providerName`,
  `includeDefaultRoots`, `customSkillDirs` (`src/index.ts`, `Config`). No used key was removed or
  renamed.
- Re-verified the four `skills/*/SKILL.md` blocks against the rc.5 parser: each frontmatter block
  parses, `name` is kebab-case and matches its directory, and `description` is present and
  non-empty. The optional `whenToUse` field is still accepted.
- Re-verified the manifest and display surface: `exports` still provides `./locale/*.json` and
  `./icon`, `dsh.bundle.patch` still points at `cordis.patch.yml`, and `files` covers every runtime
  file. Locale metadata still uses `meta.title`/`meta.description`; no `meta.detail` is declared and
  the README fallback remains the documented option, so no detail document was added.
- Checked the `v0.2.2-rc.3` and `v0.2.2-rc.4` upgrade guides: they change the DeepSeek refusal error
  code and the default web-search provider, neither of which touches a configuration-only MATLAB
  bundle. No upgrade guide applies, and no patch, skill, or tool-surface change was needed.
- Confirmed the upgrade behaviour is still present on `0.2.2-rc.5`: the desktop profile keeps the
  `link:` dependency but its `dsh.profile.bundles` no longer lists `dsh-matlab-control`. Recovery is
  unchanged: `plugin_manager { action: "set_bundle", target: "dsh-matlab-control", enabled: true }`.
- Re-verified by configuration/manifest recomposition on the rc.5 kernel: installing this checkout
  into a throwaway profile (`dsh plugin --profile <name> add link:<checkout>`) auto-selected the
  bundle, and `dsh --profile <name> --dump-config` composed both rows and every config key without
  error.
  Live mounting on a desktop profile after re-enabling was not re-run (no headless app activation in
  this environment), so `application: applied`, the 14 `mcp__matlab__*` tools, and
  `model_check`/`pidtune` results are not claimed for `0.2.2-rc.5`; they remain for the user to
  confirm after re-enabling.
- Version bumped to `0.1.3`; both READMEs extended with `0.2.2-rc.5`.

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
