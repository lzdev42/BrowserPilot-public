# BrowserPilot 🚀

**The Next-Gen AI Browser Automation Engine. | 下一代 AI 浏览器自动化引擎。**

> [!IMPORTANT]
> **Operational Status / 运行状态说明**
> - **MCP Server**: Fully operational with all tools verified in testing. | 已完全可用，各项工具已通过功能测试验证。
> - **Native Agent**: Functional but in early-access; optimization is ongoing. | 处于早期可用状态，功能尚在完善中。
>
> **Model Capability Benchmark (Field Tested) / 模型能力实测建议**
> Field tests: NVIDIA NIM open-source models and GLM-free models **completely fail** the "selecting a county-level city" task on Boss Zhipin. Gemini 2.5/3 Flash **succeeded**. Other models have not been tested. | 实测：NVIDIA NIM 平台上的开源模型及 GLM 免费版在 Boss 直聘“选择县级市”任务中**全军覆没**。Gemini 2.5/3 Flash **测试成功**，其余模型未实测。

[English](#english) | [中文](#中文)

---

<a name="english"></a>
## English

BrowserPilot is a high-performance, **native desktop application** designed to bridge the gap between AI Agents and the Web. Unlike traditional script-based automation (Python/Node.js), BrowserPilot provides a zero-setup, stealthy, and powerful environment for automated browsing.

### 🚀 Development Roadmap & Status
- [x] **MCP Server**: Full Model Context Protocol support.
- [x] **Dual-Mode Parity**: Agent and MCP Server functional alignment.
- [x] **Autonomous Scripting**: Auto-sensing and script generation for repetitive tasks.
- [x] **Stealth & Anti-Detection**: Verified bypass of anti-bot systems (e.g., Boss Zhipin).
- [ ] **Agent Optimization**: Refinement of autonomous execution. (**In Progress**)
- [ ] **Sharing Ecosystem**: Collaborative platform for sharing automation logic.

### ✨ Key Features
- **🖥️ Native Desktop Experience**: Built with Compose Multiplatform. No Python/Node environment required—just download and run.
- **🛡️ Advanced Anti-Detect**: Integrated with [Camoufox](https://github.com/daijro/camoufox) to provide state-of-the-art fingerprint obfuscation and bot-detection bypass.
- **🤖 AI Native (MCP)**: Native support for Model Context Protocol (MCP), allowing AI Agents like Claude or GPT to interact with the web through a standardized interface.

---

<a name="中文"></a>
## 中文

BrowserPilot 是一款高性能的**原生桌面应用程序**，旨在连接 AI 代理与 Web 世界。与传统的基于脚本（如 Python/Node.js）的自动化工具不同，BrowserPilot 提供了一个零配置、高隐匿且功能强大的自动化浏览器环境。

### 🚀 开发路线图与项目进度
- [x] **MCP Server**: 完整支持模型上下文协议。
- [x] **双模式对齐**: Agent 与 MCP Server 功能对齐。
- [x] **自主脚本**: 重复性任务自动感知与脚本生成。
- [x] **反检测**: Boss 直聘等严苛环境下的自动化操作验证。
- [ ] **Agent 完善**: 持续优化执行稳定性。 (**进行中**)
- [ ] **分享生态**: 规划中的脚本与提示词共享平台。

### ✨ 核心特性
- **🖥️ 原生桌面体验**：基于 Compose Multiplatform 构建。无需安装 Python 或 Node.js 环境，下载即用。
- **🛡️ 强力反检测**：深度集成 [Camoufox](https://github.com/daijro/camoufox)，提供顶级的浏览器指纹混淆能力，轻松绕过机器人检测。
- **🤖 AI 原生支持 (MCP)**：原生支持模型上下文协议 (MCP)，让 AI Agent 能够通过标准化接口直接操控网页。

---

## 📥 Downloads | 下载

BrowserPilot is distributed as pre-compiled binaries. | BrowserPilot 以预编译二进制文件形式发布。

- **Latest Stable**: Check the **[Releases](https://github.com/lzdev42/BrowserPilot-public/releases)** section.
- **Latest Dev Build**: You can download the latest automated builds from the **[GitHub Actions](https://github.com/lzdev42/BrowserPilot-public/actions)** page (click on a successful run and scroll down to "Artifacts").

### Supported Platforms | 支持平台:
- **macOS**: `.dmg` (Apple Silicon & Intel)
- **Linux**: `.AppImage` (x64 & ARM64)
- **Windows**: `.zip` (x64)

---

## 📜 License & Disclaimer | 许可与免责声明

BrowserPilot is provided for automation and testing purposes. Users are responsible for complying with the terms of service of the websites they interact with.
BrowserPilot 仅供自动化和测试使用。用户需自行负责遵守所交互网站的服务条款。

---
*Built with ❤️ by [lzdev42](https://github.com/lzdev42)*
