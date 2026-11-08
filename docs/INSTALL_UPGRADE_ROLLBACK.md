# 安装、升级、回滚与卸载

## 前置条件

1. Windows 10/11 x64，已解锁的交互式桌面
2. Node.js 24.15.0+ x64，位于 PATH
3. 已取得官方 KimiCU 插件 payload（`-SourcePath` 指向的 62 个文件），二选一：
   - 已安装 Kimi Code：使用 `%USERPROFILE%\.kimi-code\plugins\managed\kimi-cu-win`（只读引用，不要写入）。
   - 未安装 Kimi Code：下载官方版本化包 `https://cdn.kimi.com/kimi-computer-use-windows/0.3.10/kimi-cu-win-plugin.zip` 并解压，解压前核对 ZIP 的 SHA-256 为 `aae21d2b5b8eb601f8f89b29e3ccc3ffab32db95f009a1e7e0a5409eefe4cea9`（取自官方 `0.3.10/version.json`）。

   注意：官方安装脚本 `latest/setup_windows.cmd` 只安装 native runtime（`%LOCALAPPDATA%\KimiCU`），不会产出此 payload；runtime 仍是运行前提。仓库清单固定 0.3.10，`latest` 已更新到 0.3.13 且 8 个文件哈希不同，直接用最新版会在校验处中止。
4. 与 payload 版本匹配的 `payload-lock.json`。官方 0.3.10 可直接使用仓库根目录附带的清单。

随附清单根据已安装的官方 0.3.10 完整 62 文件集生成，并与其中 `bundle.json` 的官方文件哈希逐项交叉校验。其他版本必须重新生成并复核清单；哈希校验不等同于代码签名或上游背书。

## 安装

```powershell
# 预览：校验版本、文件集、大小与 SHA-256，不写状态
powershell.exe -NoProfile -File scripts/setup.ps1 `
  -SourcePath "<官方 payload 目录>" `
  -LockPath "<payload-lock.json>" -DryRun

# 安装到 %LOCALAPPDATA%\KimiCU-Community
powershell.exe -NoProfile -File scripts/setup.ps1 `
  -SourcePath "<官方 payload 目录>" `
  -LockPath "<payload-lock.json>"

# 诊断
powershell.exe -NoProfile -File scripts/setup.ps1 -Doctor
```

安装器不下载 payload，不改写宿主配置。任何文件缺失、大小或哈希不符、源目录含额外文件或重解析点都会中止。

## 升级

使用同一命令指向新版本 payload 与对应清单即可。新版本写入独立的版本化目录，成功后原子切换 `current.json`，上一版本保留在 `previous.json`。

## 回滚

```powershell
powershell.exe -NoProfile -File scripts/setup.ps1 -Rollback
```

回滚前会重新校验目标 payload；失败则保持当前版本不变。

## 卸载

```powershell
# 只删除状态中引用的 current/previous payload 与状态文件
powershell.exe -NoProfile -File scripts/setup.ps1 -Uninstall

# 移除宿主中的本项目节点
node scripts/apply-host-config.mjs --host workbuddy `
  --config-path "$env:USERPROFILE\.workbuddy\mcp.json" --remove
```

更早的版本目录不会自动清理，见 docs/KNOWN_LIMITATIONS.md。

## 备份与恢复

写入宿主配置前会在同目录生成 `<文件名>.kimi-cu-backup-<时间戳>`，并输出恢复命令。备份是原配置的完整副本，可能包含其他 MCP 服务的密钥；确认无需恢复后请自行删除。

## 失败处理

- 状态缺失、schema 不符、路径越界或哈希不符：停止，不写宿主配置
- 宿主配置不是合法 JSON：停止，不覆盖原文件
- 配置文件被宿主占用：有限次退避重试后失败，原文件保持不变
