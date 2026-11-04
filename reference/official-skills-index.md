# Official skill index (read the SKILL.md before acting)

Domain knowledge lives in skill files, not in a system prompt. Read the matching file, then
follow its workflow.

## Already registered in this Harness (call the `skill` tool by name)

| Skill | Use for |
|---|---|
| `matlab-debug-code` | errors, breakpoints, workspace inspection |
| `matlab-review-code` | code quality / coding standards review |
| `matlab-run-tests` | running suites, filtering, coverage |
| `matlab-write-tests` | writing `matlab.unittest` tests |
| `matlab-read-documentation` | exact official syntax before guessing |
| `matlab-list-products` / `matlab-install-products` | installed products, installs |

These come from the MathWorks MATLAB Agentic Toolkit (`~/.matlab/agentic-toolkits/matlab/skills-catalog`) and are
already in the session catalog — do not re-create them.

## Vendored Simulink Agentic Toolkit (read by absolute path)

Root: `<plugin dir>/thirdparty/simulink-agentic-toolkit/skills-catalog` (your clone of the official toolkit)

| Area | File |
|---|---|
| Build / restructure models | `model-based-design-core\building-simulink-models\SKILL.md` |
| Simulate and read results | `model-based-design-core\simulating-simulink-models\SKILL.md` |
| Layout / readability | `model-based-design-core\arranging-simulink-layouts\SKILL.md` |
| Clean up a saved model | `model-based-design-core\simulink-sanitize-model\SKILL.md` |
| Projects, paths, data dictionaries | `model-based-design-core\managing-simulink-projects\SKILL.md` |
| Requirements drafts | `model-based-design-core\generate-requirement-drafts\SKILL.md` |
| MBD specifications | `model-based-design-core\specifying-mbd-algorithms\SKILL.md` |
| Tests / verification | `verification-validation-and-test\testing-simulink-models\SKILL.md` |
| Plant models | `model-based-design-core\specifying-plant-models\SKILL.md` |
| Linearization | `control-systems\simulink-linearize\SKILL.md` |
| Frequency response in Simulink | `control-systems\simulink-frequency-response\SKILL.md` |

Group directories under `skills-catalog` are: `model-based-design-core`, `model-based-system-engineering`,
`verification-validation-and-test`, `simulink-simulation`, `simulink-modeling`, `control-systems`,
`simulink-environment-fundamentals`, `signal-processing`, `physical-modeling`, `code-generation`,
`real-time-simulation-and-testing`, `simulink-block-and-blockset-authoring`.

## MATLAB App Building

`<user home>\.matlab\agentic-toolkits\matlab\skills-catalog\matlab-app-building\` — `matlab-build-app`, `matlab-build-chart`,
`matlab-apply-theme` (installed but not registered as skills — read the `SKILL.md` files directly when building an app).
The upstream prompt category is the `App Building` folder of <https://github.com/matlab/prompts>.
