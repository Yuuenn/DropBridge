# Issue 草稿：首次启动误报通知快捷方式缺少 AppUserModelID

## 标题

`register.ps1` 用原始字节扫描验证 AppUserModelID，导致有效的快捷方式注册被误判为失败

## 环境

- Windows 11 `10.0.26200`
- Windows PowerShell 5.1
- DropBridge 当前 `main`，便携目录运行

## 复现步骤

1. 从便携目录启动 `DropBridge.exe`。
2. 程序调用包内 `tools/register.ps1` 注册通知应用身份、协议和开始菜单快捷方式。
3. 查看会话的 `supervisor.log`。

## 实际结果

日志记录：

```text
notification registration failed: register.ps1 exit 1:
shortcut does not carry the AppUserModelID
```

脚本已经通过 `IPropertyStore.SetValue` 和 `Commit` 写入 `PKEY_AppUserModel_ID`，但随后把 `.lnk` 整体按 UTF-16 解码并搜索 `DropBridge.Tray`。Shell Link 是结构化二进制格式，属性值不保证以可直接搜索的 UTF-16 字符串出现，因此这个检查会产生误报。

同时，原 COM 接口没有保留 `SetValue` / `Commit` 的 HRESULT，真正的写入失败也无法区分。

## 期望结果

- 使用 Shell 属性 API 的返回值判断 `SetValue` 与 `Commit` 是否成功；
- 不扫描 `.lnk` 原始字节；
- 重复运行注册脚本应幂等并返回 0；
- 注册失败不应影响 radio、AWDL、netd 或 receiver 数据面。

## 修复验证

修复后针对同一便携包连续运行两次：

```text
registered exe=...\DropBridge.exe shortcut=...\DropBridge.lnk
registered exe=...\DropBridge.exe shortcut=...\DropBridge.lnk
EXITS=0,0
```

并确认：

- `HKCU\Software\Classes\AppUserModelId\DropBridge.Tray` 的 `DisplayName` 与 `IconUri` 正确；
- `dropbridge://` 协议命令指向当前便携包；
- 开始菜单 `DropBridge.lnk` 存在。

## 边界

本 issue 只处理注册脚本的误报与 HRESULT 检查。Toast 横幅是否被 Windows 通知设置或专注模式显示，需要单独做端到端测试。
