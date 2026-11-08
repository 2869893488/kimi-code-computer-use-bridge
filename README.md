# Kimi Code Computer Use Bridge

一个让「官方 KimiCU 的电脑操作能力」能在其他 AI 客户端里复用的社区转接器。官方 KimiCU 以 Kimi Code 原生插件的形式提供桌面控制（computer use）能力，让模型可以读取屏幕、激活窗口、接管键鼠；如果想在 Antigravity、WorkBuddy 或 Claude Desktop 里用上同一套能力，就需要把本机已安装的官方 payload 接到这些宿主的 MCP 配置中。本仓库把官方 payload 按哈希锁定并安装到 `%LOCALAPPDATA%\KimiCU-Community`，再把名为 `kimi-cu` 的 stdio MCP server 合并进宿主配置文件；宿主重启后即可加载 `js`、`js_reset`、`js_add_node_module_dir` 等工具。Codex 使用 TOML，需按[宿主接入指南](docs/HOST_INTEGRATION_GUIDE.md)手动添加。

本仓库**不下载、不分发**任何官方运行时、EXE、ZIP 或插件源码，官方组件需由你自行从官方渠道获取，见[获取官方组件](#获取官方组件)；脚本只做校验、事务化安装（`-DryRun` / `-Doctor` / 回滚 / 卸载）和「写入前备份 + 原子替换」的配置合并，并明确拒绝写入官方 managed 插件目录；目前仅面向 Windows 10/11 x64 的交互式桌面，仍处于预览阶段，各宿主的验证程度以[支持矩阵](docs/SUPPORT_MATRIX.md)为准。

> 预览版本 `0.1.0-preview.1`：接口和安装流程可能变化。

非官方社区项目，与 Moonshot AI、Kimi、Google Antigravity、Anthropic、OpenAI 不存在隶属、授权或背书关系。

本仓库不包含、也不分发官方 KimiCU 运行时、EXE、ZIP 或官方插件源码。它只提供：

- 本地已安装官方 payload 的哈希锁定与事务化安装脚本
- 把缓存 payload 安全写入宿主 MCP 配置的适配器
- 不依赖真实桌面的自动化测试

## 风险

桌面控制会读取屏幕内容、激活窗口并接管键盘鼠标。截图可能包含敏感信息。请仅在受控环境或独立测试账户中使用，并在执行前审查模型生成的代码。本项目不提供任何担保。

## 要求

- Windows 10/11 x64，已解锁的交互式桌面
- Node.js 24.15.0 或更新版本，x64
- PowerShell 5.1 或 7
- 用户自行安装的官方 KimiCU payload。仓库附带基于官方 0.3.10 本地安装生成的 `payload-lock.json`；哈希只证明文件集内容，不证明上游身份。其他版本必须自行生成并复核清单。

## 获取官方组件

需要区分两类官方组件，它们来源不同：

- **Native runtime**（`kimi-cu.exe`）：由官方安装脚本 `https://cdn.kimi.com/kimi-computer-use-windows/latest/setup_windows.cmd` 装到 `%LOCALAPPDATA%\KimiCU`。该脚本**只安装 runtime**，不会产出下面的插件 payload；runtime 是 payload 运行的前提。
- **插件 payload**（`-SourcePath` 指向的 62 个文件），二选一：
  1. 已安装 Kimi Code：直接使用 `%USERPROFILE%\.kimi-code\plugins\managed\kimi-cu-win`（不要写入该目录，只读引用）。
  2. 未安装 Kimi Code：从官方版本化地址下载并解压 `https://cdn.kimi.com/kimi-computer-use-windows/0.3.10/kimi-cu-win-plugin.zip`，解压前核对 ZIP 的 SHA-256 为 `aae21d2b5b8eb601f8f89b29e3ccc3ffab32db95f009a1e7e0a5409eefe4cea9`（取自官方 `0.3.10/version.json`）。

仓库附带的 `payload-lock.json` 固定为官方 0.3.10。`latest` 会持续漂移（本说明撰写时 `latest/version.json` 已是 0.3.13，文件集相同但有 8 个文件哈希不同），直接使用最新版会在哈希校验处中止；使用 0.3.10 以外的版本必须自行生成并复核清单。

## 安装

```powershell
# 1. 预览，不写入状态
powershell.exe -NoProfile -File scripts/setup.ps1 -SourcePath "<官方 payload 目录>" -LockPath "<payload-lock.json>" -DryRun

# 2. 校验并安装到 %LOCALAPPDATA%\KimiCU-Community
powershell.exe -NoProfile -File scripts/setup.ps1 -SourcePath "<官方 payload 目录>" -LockPath "<payload-lock.json>"

# 3. 诊断
powershell.exe -NoProfile -File scripts/setup.ps1 -Doctor
```

```powershell
# 4. 写入宿主配置（示例：WorkBuddy）
node scripts/apply-host-config.mjs --host workbuddy `
  --config-path "$env:USERPROFILE\.workbuddy\mcp.json" `
  --install-root "$env:LOCALAPPDATA\KimiCU-Community"
```

JSON 合并适配器覆盖 Antigravity、WorkBuddy 和 Claude Desktop。Antigravity 与 WorkBuddy 为实验性，Claude Desktop 尚未真机验证；Codex 使用 TOML，需要手动配置。详见 [支持矩阵](docs/SUPPORT_MATRIX.md) 与 [宿主接入指南](docs/HOST_INTEGRATION_GUIDE.md)。

## 用 agent 安装

把下面这段直接复制发给你的 agent，由它代你完成安装。前提是本机已有官方组件（获取方式见上文「获取官方组件」）；本仓库不下载、不分发官方组件。

```text
帮我装 Kimi Code computer use 转接器：https://github.com/2869893488/kimi-code-computer-use-bridge
前提：本机已有官方 KimiCU runtime（没有先跑 https://cdn.kimi.com/kimi-computer-use-windows/latest/setup_windows.cmd，它只装 runtime）
和 kimi-cu-win 插件 payload（已装 Kimi Code 的在 %USERPROFILE%\.kimi-code\plugins\managed\kimi-cu-win；
否则下载 https://cdn.kimi.com/kimi-computer-use-windows/0.3.10/kimi-cu-win-plugin.zip 解压，ZIP 的 SHA-256 应为 aae21d2b5b8eb601f8f89b29e3ccc3ffab32db95f009a1e7e0a5409eefe4cea9）。

先问我：目标宿主（claude-desktop/workbuddy/antigravity/codex）、宿主配置绝对路径、官方 payload 目录、版本是否 0.3.10。

然后照做，任一步失败就停：
clone 仓库 → powershell -NoProfile -File scripts/setup.ps1 -SourcePath "<payload>" -DryRun，过了再去掉 -DryRun → -Doctor 记下版本和 payloadRoot →
node scripts/apply-host-config.mjs --host <宿主> --config-path "<配置>" --install-root "$env:LOCALAPPDATA\KimiCU-Community"，先 --dry-run 再正式写 → 重启宿主确认 js/js_reset/js_add_node_module_dir 已加载。
codex 是 TOML，脚本不改，照 docs/HOST_INTEGRATION_GUIDE.md 手动加。

别写 .kimi-code\plugins\managed\，别手工绕过脚本报错。装完报版本、备份路径、加载出的工具。
```

## 测试

```powershell
npm test
```

默认测试不访问真实桌面，不要求已安装 payload。

## 文档

- 安装、升级、回滚、卸载：[docs/INSTALL\_UPGRADE\_ROLLBACK.md](docs/INSTALL_UPGRADE_ROLLBACK.md)
- 支持矩阵：[docs/SUPPORT\_MATRIX.md](docs/SUPPORT_MATRIX.md)
- 已知限制：[docs/KNOWN\_LIMITATIONS.md](docs/KNOWN_LIMITATIONS.md)
- 安全报告：[SECURITY.md](SECURITY.md)
- 发布与撤回：[docs/RELEASE\_PROCESS.md](docs/RELEASE_PROCESS.md)
- 变更记录：[CHANGELOG.md](CHANGELOG.md)
- 许可证：[LICENSE](LICENSE)

## 许可证

本仓库原创代码使用 MIT。官方组件不包含在本仓库中，继续受其原始许可证和条款约束；本仓库不对官方组件授予任何许可。
