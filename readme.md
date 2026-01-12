

# BMap Voice Studio

**BMap Voice Studio** 是一个用于制作百度地图个性化语音包的 Windows 桌面自动化工具。

它通过 ADB 控制安卓模拟器，利用本地大模型语音服务（IndexTTS），实现从 **文本识别 -> 语音合成 -> 虚拟内录 -> 自动提交 -> 提取打包** 的全自动化闭环。

![Version](https://img.shields.io/badge/version-1.2-blue.svg) ![Platform](https://img.shields.io/badge/platform-Windows-lightgrey.svg) ![Python](https://img.shields.io/badge/python-3.x-green.svg)

## ✨ 特性 (v1.2)

*   **全自动化工作流**：无需人工干预，自动完成地图录音任务。
*   **IndexTTS 深度集成**：
    *   支持 **Upload Mode**：可直接使用本地任意位置的 `.wav/.mp3` 作为参考音频，无需复制文件。
    *   支持 **Hybrid Select**：既可自动关联项目 `prompts` 目录，也可手动浏览任意文件。
*   **原生打包**：内置 Python Zip/Tar 逻辑，自动生成 `config.yaml` 配置文件，完美兼容 MultiTTS 导入。
*   **零脚本依赖**：纯 Python 实现，不再依赖外部 `.bat` 批处理。

## 🛠️ 环境依赖 (Prerequisites)

运行前请确保满足以下硬性条件：

1.  **音频驱动 (必装)**：安装 [VB-Audio Virtual Cable](https://vb-audio.com/Cable/) 并重启。
    *   安装后需配置**侦听功能**（否则录制时听不到TTS音频）：
        1.  右键任务栏音量图标 -> **声音** -> **录制**选项卡
        2.  右键 `CABLE Output` -> **属性** -> **侦听**选项卡
        3.  勾选 **侦听此设备**，下方选择你的播放设备（如耳机/USB声卡/扬声器）
        4.  点击**确定**保存
2.  **IndexTTS 服务**：本地部署并可运行。
3.  **安卓模拟器 (雷电/MuMu)**：
    *   **ROOT 权限**：必须开启 (用于提取 `/data` 下的语音文件)。
    *   **分辨率**：建议 `720x1280` (DPI 320)。
    *   **App**：安装百度地图并登录。

> [!IMPORTANT]
> **⚠️ 核心注意事项 (必读)**
>
> 模拟器的音频输入通道会在重启后重置。**每次启动模拟器后，必须手动执行以下操作：**
> 1. 进入模拟器设置 -> **音频设置**。
> 2. 将 **麦克风输入 (Mic)** 修改为 `CABLE Output (VB-Audio Virtual Cable)`。
> 3. **保存设置**。
>
> **后果：如果不设置，软件播放的声音无法传入模拟器，导致录制全是静音。**

## 📥 下载与安装 (Installation)

我们提供两个发行版本，请根据需求选择：

| 版本 | 描述 | 适用人群 |
| :--- | :--- | :--- |
| **Full (完整版)** | 包含 `BMVS.exe` 以及完整的 `adb/` 和 `ffmpeg/` 工具包。 | **推荐**。解压即用，无需配置环境。 |
| **Lite (精简版)** | 仅包含 `BMVS.exe` 主程序。 | **极客**。需自行将 `adb` 和 `ffmpeg` 加入系统 PATH 环境变量，或手动放入同级目录。 |

## 🚀 快速开始 (Quick Start)

1.  **启动服务**：
    *   打开软件，在 "IndexTTS 服务配置" 中选择你的启动脚本 (`.bat`)。
    *   点击 **🚀 一键启动**。

2.  **配置音频**：
    *   **参考音频**：选择 `.bat` 后会自动扫描 `prompts` 目录；或者点击 "📂 浏览" 选择任意音频文件。
    *   **虚拟内录设备**：下拉选择 `CABLE Input (VB-Audio Virtual Cable)`。

3.  **准备模拟器**：
    *   确认模拟器麦克风已设为 `CABLE Output`。
    *   打开百度地图 -> 进入 "录语音包" -> "开始定制"之前。

4.  **运行** ⚠️ 顺序很重要：
    *   **先**在模拟器中点击 "开始定制"->"选择模式"->"开始录制" 按钮，进入录音界面。
    *   **后**点击软件上的 **▶ 开始自动录制**（此时语音包名称会复制到剪贴板）。
    *   程序将自动接管后续操作。
    *   录制完成后，点击 **📦 一键提取并打包**。

> [!TIP]
> **默认推荐顺序：** 先操作模拟器再点软件，可以避免大多数用户遇到的自动命名问题。
> 
> **特殊情况：** 如果你发现语音包名称自动命名有误（如名称未同步或显示错误），可尝试**先点击软件上的“开始自动录制”，再操作模拟器**，这样可以让语音包名称提前同步到模拟器，解决剪贴板延迟问题。

## 📱 导入手机

生成的 `.zip` 文件位于软件根目录。

1.  将 ZIP 文件发送到手机。
2.  解压或直接在 MultiTTS 等语音包管理工具中导入。
3.  或手动解压至：`/Android/data/com.baidu.BaiduMap/files/BaiduMap/bnav/voice/` (需视具体 Android 版本权限而定)。

## 🧩 技术栈

*   **GUI**: Python `tkinter` + `ttk` (Grid Layout)
*   **Audio**: `pyaudio` (WASAPI/MME), `pydub`, `ffmpeg`
*   **Control**: `adb` (Android Debug Bridge), `uiautomator`
*   **Packaging**: `zipfile` (With Unix permission injection)
*   **Build**: `Nuitka` (C Compilation, 安全加固)

## 🔨 构建 (Build)

如需自行编译，请确保已安装依赖后执行：

```powershell
# 安装 Nuitka
pip install nuitka

# 编译为单文件 exe
nuitka --standalone --onefile --windows-console-mode=disable --windows-icon-from-ico=app.ico --enable-plugin=tk-inter --include-package-data=pypinyin --include-data-files=text.dat=text.dat --include-data-files=app.ico=app.ico -o BMVS.exe main.py
```

> **注意**：首次编译需要下载 C 编译器（MinGW64），可能耗时较长。

## 📄 License

MIT License