<p align="center">
    <img src="stacer-logo.svg" width="120" alt="Stacer-X">
</p>

<h1 align="center">Stacer-X</h1>

<p align="center">简体中文 · <a href="README.md">English</a></p>

<h3 align="center">采用 Fluent 风格重新设计的现代 Linux 系统优化工具</h3>

<p align="center">
    <img alt="平台（GNU/Linux）" src="https://img.shields.io/badge/platform-GNU/Linux-blue.svg"/>
    <img alt="界面（Qt Quick / QML）" src="https://img.shields.io/badge/UI-Qt%20Quick%20%2F%20QML-41cd52.svg"/>
    <img alt="风格（Microsoft PC Manager）" src="https://img.shields.io/badge/style-PC%20Manager%20Fluent%20dark-5cc4ef.svg"/>
</p>

---

本公开仓库是 [s4rt4/Stacer-x](https://github.com/s4rt4/Stacer-x) 的分支，增加了
简体中文 / 英文切换，以及可选的系统托盘后台监控。相关更新已提交至
[上游 PR #1](https://github.com/s4rt4/Stacer-x/pull/1)。

**Stacer-X** 基于 [QuentiumYT/Stacer](https://github.com/QuentiumYT/Stacer)，使用
**Qt Quick / QML** 重写前端界面，并参考 **Microsoft PC Manager** 的 Fluent 深色风格。
项目复用了原有的 `stacer-core` C++ 后端，系统数据仍来自原有的采集工具；界面则重新设计，
包括固定大小的无边框窗口、图标导航栏、实时图表和自定义标识。

> 原有 Qt Widgets 应用仍可作为 `stacer` 目标构建。新的 QML 界面是独立的 `stacer-x`
> 目标：两者共用 `stacer-core`，其他部分相互独立。

## 功能介绍

- **仪表盘**：实时查看 CPU、内存、各核心负载和运行时间；通过“系统加速”释放可回收缓存，
  并显示释放量。状态横幅会根据当前负载提供一键处理入口，例如释放内存或转到清理页面。
- **清理**：扫描缓存、回收站、软件包缓存、日志和崩溃报告；清理前可通过“详情”预览待删除内容。
- **服务**：搜索并启动、停止、启用或禁用 systemd 服务。
- **进程**：搜索和排序进程，调整优先级或结束进程。
- **启动项**：管理 `~/.config/autostart` 中的启动项。
- **卸载程序**：卸载软件包，可选清理残留配置（`~/.config`、`~/.cache`、
  `~/.local/...`）以及自动移除不再需要的依赖。
- **Hosts**：启用、添加或删除 `/etc/hosts` 条目。
- **资源监控**：通过实时图表查看 CPU、集成显卡、**NVIDIA 显卡**（通过 `nvidia-smi`）、
  内存、磁盘 I/O 和网络吞吐量。
- **磁盘**：通过环形图和图例查看各分区的使用情况。
- **设置**：配置 CPU、内存和磁盘使用率告警阈值、登录时启动、主题强调色和中英文界面语言；
  相关设置会被保存。
- **系统托盘**：默认关闭窗口后继续在托盘中运行。点击托盘图标或选择“显示 Stacer-X”可恢复窗口，
  选择“退出”可完全退出。可在 **设置 → 关闭时隐藏到系统托盘** 中关闭此行为；
  如果桌面没有可用托盘，关闭窗口时程序会正常退出。
- 无边框固定大小窗口、自定义标识，以及桌面启动器和 Dock 集成。

## 从源码构建

### 环境要求

需要 Qt 6.5 或更新版本，并安装 Quick / QML、Widgets 和 Linguist Tools 模块，
以及 CMake 和 Ninja。中文翻译文件会在构建时编译进应用程序。Ubuntu 22.04 标准软件源中的
Qt 6 版本低于要求，需要另行准备合适的新版本 Qt SDK。

**Fedora：**

```bash
sudo dnf install cmake ninja-build gcc-c++ \
  qt6-qtbase-devel qt6-qtdeclarative-devel qt6-qt5compat-devel \
  qt6-qtsvg-devel
```

**Debian / Ubuntu：**

```bash
sudo apt update
sudo apt install cmake ninja-build g++ \
  qt6-base-dev qt6-declarative-dev qt6-declarative-private-dev qt6-tools-dev-tools \
  libqt6core5compat6-dev libqt6svg6-dev qml6-module-*
```

### 编译和运行

```bash
cmake -S . -B build -G Ninja
cmake --build build --target stacer-x -j $(nproc)
./build/stacer-qml/stacer-x
```

需要管理员权限的操作（系统加速、清理系统目录、编辑 `/etc/hosts`、卸载软件包）
会在执行时通过 **polkit**（`pkexec`）请求提权；Stacer-X 本身以普通用户身份运行。

### 语言与后台监控

在 **设置 → 语言** 中选择 **English** 或 **简体中文**。切换立即生效，并在下次启动时保留；
首次运行时根据系统语言选择。应用名称、服务名称和操作系统信息仍由系统提供。

启用 **关闭时隐藏到系统托盘** 后，关闭窗口仍会继续进行资源监控及已配置的告警。
通过托盘菜单中的“退出”可完全退出程序。GNOME 需要可用的系统托盘或 AppIndicator 扩展；
如果没有可用托盘，关闭窗口会直接退出，避免窗口隐藏后无法恢复。

## 项目结构

```text
stacer-core/     后端：系统数据采集与工具（共用，沿用上游 Stacer）
stacer/          原有 Qt Widgets 应用（构建目标：stacer）
stacer-qml/      Stacer-X 的 Qt Quick / QML 前端（构建目标：stacer-x）
  *.{h,cpp}        连接 stacer-core 与 QML 的控制器（上下文属性）
  qml/             主题、可复用组件和页面
  qml/icons/       运行时重新着色的单色 SVG 图标
icon/            原始图标集
stacer-logo.svg  应用标识
```

## 致谢

Stacer-X 基于 Quentin Lienhardt（QuentiumYT）的
**[Stacer](https://github.com/QuentiumYT/Stacer)**，该项目最初由 Oguzhan Inan 创建。
后端系统逻辑来自该项目；Stacer-X 为现有工具提供 QML / Fluent 前端和额外的交互体验。

界面设计参考了 **Microsoft PC Manager**。

## 许可证

与上游 Stacer 使用相同的许可证，详见 [LICENSE](LICENSE)。
