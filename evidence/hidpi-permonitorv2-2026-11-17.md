# 默认高分屏支持（2026-11-17）

## 实现

- `src/supervisor/DropBridge.manifest` 声明 Windows 10/11 使用 `PerMonitorV2`，兼容回退为 `dpiAware=true/pm`。
- `Build-Supervisor.ps1` 用 `/win32manifest` 将 manifest 嵌入 `DropBridge.exe`。
- `DeviceNameDialog` 使用 `AutoScaleMode.Dpi`。
- 托盘图标继续按当前显示器的物理小图标尺寸选择 ICO 帧，避免高缩放下放大低分辨率帧。

## 离线验证

`Build-Supervisor.ps1 -Test` 使用 Windows SDK `mt.exe` 从生成的 EXE 提取资源 `#1`，断言包含 `PerMonitorV2` 与 `true/pm`。信道序列、托盘明暗图标、设置测试全部通过。

## 本机运行验证

- 显示缩放：175%；
- 新包运行 `DropBridge.exe --auto-exit 90`；
- 六个项目进程全部启动；
- TCP 49900、49901、8771 与 UDP 15353 全部监听；
- 自动退出后进程与端口无残留；
- radio 日志包含 `shutdown: done`，exit 0。

## 尚未覆盖

- 在两个不同 DPI 的显示器之间移动窗口；
- 任务栏位于非底部位置；
- 多个接收弹窗在混合 DPI 多显示器上的排列。
