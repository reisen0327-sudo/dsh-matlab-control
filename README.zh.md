# dsh-matlab-control

**DeepSeek Harness（DSH）的 MATLAB/Simulink 控制工程助手插件。**
[English](README.md) | 中文

本插件**不实现** MATLAB/Simulink 通信层：桥接复用 MathWorks 官方 **MATLAB MCP Server**，
Simulink 建模工具复用官方 **Simulink Agentic Toolkit** 的工具注册表，插件只做配置与技能注册，
再用 4 个技能把 Agent 的工作流特化到 Simulink 仿真、App Designer 与控制系统工程。

- 对模型暴露 14 个工具：5 个 MATLAB（`evaluate_matlab_code`、`run_matlab_file`、`check_matlab_code`、
  `run_matlab_test_file`、`detect_matlab_toolboxes`）+ 9 个 Simulink（`model_overview`、`model_read`、
  `model_edit`、`model_check`、`model_query_params`、`model_resolve_params`、`model_read_diagnostics`、
  `model_scan`、`model_test`）。
- 工具在会话中统一命名为 `mcp__matlab__<tool>`。
- 领域知识放在技能文件里，不塞进系统提示词。

## 三大能力

| 领域 | Agent 会做什么 |
|---|---|
| **Simulink 仿真** | 读模型结构（`model_overview`、`model_read`）、查连接完整性与 Edit-Time 检查（`model_check`）、跑仿真并把信号/指标整理成结构化摘要，而不是回传原始命令行输出 |
| **App Designer / GUI** | 生成现代 UI 组件（`uifigure`/`uiaxes`/`uidropdown`/`uibutton`）的 App 类，回调中调用 `sim()` 把结果实时画进 `uiaxes`，并用 `check_matlab_code` 校验 |
| **控制科学与工程** | 先判断意图再选函数：`bode`/`nyquist`/`margin`/`allmargin`、`step`/`impulse`/`lsim`、`ss`/`tf`/`zpk`、`pidtune`/`pidTuner`、`robuststab`/`robgain`；返回增益裕度、相位裕度、超调、调节时间、稳态误差与闭环稳定性 |

## 实现方式

`cordis.patch.yml` 就是全部运行时——两行配置，没有 JavaScript 代码：

| 行 id | 依赖包 | 作用 |
|---|---|---|
| `mcp-matlab` | `@deepseek-ai/dsh-mcp-client` | 以 stdio 拉起 MathWorks MATLAB MCP Server，把工具注册为 `mcp__matlab__*` |
| `matlab-control-skills` | `@deepseek-ai/dsh-skill-filesystem` | 把 `skills/` 注册为所有 Agent 会话的技能源 |

Simulink 的 9 个工具来自官方 [Simulink Agentic Toolkit](https://github.com/matlab/simulink-agentic-toolkit)
的工具注册表，通过 `--extension-file` 交给同一个 MCP Server。**本仓库不打包**该 toolkit，
也不打包 MCP Server 二进制——它们都是 MathWorks 交付物，遵循各自的许可条款。

## 前置条件

- **MATLAB R2023a 或更高** + **Simulink**（toolkit 要求 R2023a+；实测于 R2024b）。
- **MATLAB MCP Server** 可执行文件（v0.13.0+）；Windows 上 MATLAB Agentic Toolkit 会装到
  `<用户目录>\.matlab\agentic-toolkits\bin\matlab-mcp-server.exe`。
- **MATLAB MCP Server Toolbox**，每个 MATLAB 版本安装一次：`matlab-mcp-server.exe --setup-matlab`。
- 可选工具箱：Control System Toolbox（`pidtune`、`allmargin`）、Simulink Test（`model_test`）、
  Robust Control Toolbox（`robuststab`、`robgain`）。

## 安装

1. **在本插件目录内拉取官方 Simulink Agentic Toolkit**（不入库）：

   ```bash
   git clone --depth 1 https://github.com/matlab/simulink-agentic-toolkit.git \
     thirdparty/simulink-agentic-toolkit
   ```

2. **把配置改成你机器上的路径**，编辑 `cordis.patch.yml` 中这几处：

   | 字段 | 填什么 |
   |---|---|
   | `command` | `matlab-mcp-server.exe` 的绝对路径 |
   | `--matlab-root=` | MATLAB 安装根目录（含 `bin` 的那一层，不要带 `\bin`） |
   | `--extension-file=` | `<插件目录>/thirdparty/simulink-agentic-toolkit/tools/tools.json` |
   | `customSkillDirs` | `<插件目录>/skills` |

3. **装进 DSH profile**：

   ```text
   plugin_manager { action: "install_bundle", target: "<本目录绝对路径>" }
   ```

   本地目录安装是 link（不复制），只要目录不移动，第 2 步的绝对路径就一直有效。

4. 新建会话后即可使用 `mcp__matlab__*` 工具与 4 个技能。

### 会话模式

示例配置省略了 `--matlab-session-mode`，即 MathWorks 默认的 `auto`：优先连执行过
`shareMATLABSession()` 的 MATLAB 会话，找不到就自己启动一个。若要严格绑到桌面 MATLAB 会话，
先在桌面执行 `shareMATLABSession()`，再给 `args` 加 `--matlab-session-mode=existing`。

## 内核兼容性

| 组件 | 验证过的版本 |
|---|---|
| DeepSeek Harness 内核 | `0.2.1-rc.5`、`0.2.2-rc.1` |
| MATLAB | R2024b（24.2.0.2712019） |
| Simulink、Control System Toolbox | R2024b |
| MATLAB MCP Server | v0.13.0 |
| Simulink Agentic Toolkit 注册表 | 1.0.32 |

本插件**刻意不声明** `engines.dsh` 范围，也不声明 `@deepseek-ai/dsh-*` peerDependencies：它只是
配置型 bundle（两行 Loader 配置，依赖的都是稳定包），内核的兼容性门控不会跳过它。因此内核升级后
应“实测验证”，而不是靠版本区间钉死：

```text
plugin_manager { action: "list_bundles" }                                    # 是否 enabled？
plugin_manager { action: "set_bundle", target: "dsh-matlab-control", enabled: true }
```

DSH 升级会重写 `dsh.profile.bundles`；没被重新列出的第三方 bundle 会以 `enabled: false` 挂载，
于是工具与技能从会话里消失（包其实还在）。重新 enable 立即生效（已在 0.2.2-rc.1 实测），不需要重装。

## 技能

| 技能 | 触发语句 | 行为 |
|---|---|---|
| `simulink-model-check` | “检查模型”“看看有什么问题” | `model_overview` → `model_check` → 逐条解释问题并给结论 |
| `simulink-simulation-run` | “跑仿真”“看响应” | 通过 `evaluate_matlab_code` 跑 `SimulationInput` + `sim()`，输出结构化信号/指标摘要 |
| `app-designer-ui` | “生成 .mlapp 代码”“做个界面” | 现代组件 App 类 + `sim()` 回调绘图，`check_matlab_code` 校验 |
| `control-design` | “PID 整定”“频域/时域分析”“状态空间”“鲁棒性” | 意图路由到对应分析函数，输出结构化闭环指标 |

技能正文只说明“先读哪个官方 `SKILL.md`，再按它的工作流做事”，索引见
[`reference/official-skills-index.md`](reference/official-skills-index.md)。toolkit 自带的技能
不复制进本仓库，请从你 clone 的目录中读取。

## 示例提问

```text
检查这个 Simulink 模型有什么问题：<路径>.slx
跑 demo_plant 仿真 10 秒，给出超调、调节时间与终值。
做一个 App Designer 界面，能跑模型并把结果画在 uiaxes 上。
给 tf(1,[1 1]) 整定 PID，并在 Simulink 里验证闭环。
```

## 目录结构

```
dsh-matlab-control/
├── cordis.patch.yml                 # 全部运行时：MCP 客户端 + 技能源注册
├── package.json                     # bundle 清单（dsh.bundle.patch）
├── icon.svg, locale/{en,zh}.json    # 插件列表标题、描述、图标
├── skills/<name>/SKILL.md           # 4 个特化技能
├── reference/official-skills-index.md
├── README.md, README.zh.md
└── LICENSE
```

`thirdparty/`（你的 toolkit clone）与本地测试产物已被 git 忽略。

## 常见问题

| 现象 | 原因 / 处理 |
|---|---|
| 第一次工具调用要等 1–3 分钟 | MCP Server 正在启动 MATLAB，之后会很快 |
| `model_*` 报“函数不存在” | 当前 MATLAB 会话没加 toolkit 路径：在 MATLAB 里执行 `addpath(genpath('<插件>/thirdparty/simulink-agentic-toolkit'))`（用 `evaluate_matlab_code` 也行） |
| `pidtune` / `robuststab` 未定义 | 缺 Control System / Robust Control Toolbox，用 `detect_matlab_toolboxes` 确认 |
| 安装后没有 `mcp__matlab__*` 工具 | `cordis.patch.yml` 里的 MCP Server 路径不对，或该 MATLAB 版本没装 MCP Server Toolbox |
| DSH 升级后工具/技能消失 | 升级把该 bundle 从 `dsh.profile.bundles` 里剔除了，重新 enable 即可（见「内核兼容性」），包本身没坏 |
| 安装返回 `application: restart-required` | DSH 已热挂载两行；重启一次可清掉标志并走正常插件清单加载 |

## 状态与后续

在 DeepSeek Harness `0.2.1-rc.5`、`0.2.2-rc.1` + MATLAB R2024b + Simulink + Control System Toolbox
上验证通过。这是最小可用版本：模型搜索、批处理、报告导出、多 MATLAB 会话池等增强功能暂未包含。

## 许可

MIT，见 [LICENSE](LICENSE)。MATLAB MCP Server、MathWorks MATLAB MCP Server Toolbox 与
Simulink Agentic Toolkit 均为 MathWorks 软件，**不**在本许可覆盖范围内，各自遵循其原始条款，
且未打包进本仓库。
