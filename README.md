<div align="center">

# 🐍 ViperStrike

### LLM-Driven Automated Penetration Testing System

[![Python 3.10+](https://img.shields.io/badge/Python-3.10+-3776AB?style=flat-square&logo=python&logoColor=white)](https://python.org)
[![License: MIT](https://img.shields.io/badge/License-MIT-00CCCC?style=flat-square)](LICENSE)
[![Kali Linux](https://img.shields.io/badge/Platform-Kali_Linux-557C94?style=flat-square&logo=kalilinux&logoColor=white)](https://kali.org)
[![Flask](https://img.shields.io/badge/Web-Flask-000000?style=flat-square&logo=flask)](https://flask.palletsprojects.com/)

使用大语言模型驱动三模块协作架构实现端到端自动化渗透测试

**[快速开始](#-快速开始) · [Web 界面](#-web-仪表盘) · [架构设计](#-系统架构) · [配置说明](#-配置) · [安全声明](#-安全声明)**

</div>

---

## ✨ 核心特性

| 特性 | 说明 |
|------|------|
| 🧠 **三模块 LLM 协作** | 推理（策略规划）+ 生成（命令构造）+ 解析（输出分析），各自维护独立上下文 |
| 🌳 **渗透测试任务树 (PTT)** | 树形结构管理全局攻击状态，UCB1 算法智能选择下一步任务 |
| 🖥️ **赛博朋克 Web 仪表盘** | 实时进度、任务树可视化、发现列表、操作日志、图表统计、系统监控 |
| 🔧 **Kali 工具链集成** | nmap、gobuster、nikto、sqlmap、hydra、metasploit、searchsploit 等 |
| 🛡️ **多层安全控制** | 命令黑名单 + 范围校验 + 风险分级审批 |
| ⏸️ **中断恢复** | PTT 持久化至 JSON，支持暂停 / 恢复 / 人工指导 |
| 📊 **自动化报告** | 一键生成 Markdown 渗透测试报告，Web 界面直接下载 |
| ⚙️ **在线配置** | Web 界面实时修改 config.yaml，热重载无需重启 |
| 🤖 **多 LLM 支持** | OpenAI 兼容 API（GPT-4o / DeepSeek / Claude）+ Ollama 本地模型 |

---

## 🏗️ 系统架构

```
┌─────────────────────────────────────────────────────────┐
│                    用户交互层                             │
│         Web Dashboard (Flask + SSE)  │  CLI / TUI        │
└──────────────────────┬──────────────────────────────────┘
                       │
┌──────────────────────▼──────────────────────────────────┐
│               Orchestrator 协调控制器                     │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐               │
│  │ Reasoning │  │Generation│  │ Parsing  │  ← 三模块协作  │
│  │ 推理模块  │  │ 生成模块  │  │ 解析模块  │               │
│  └────┬─────┘  └────┬─────┘  └────┬─────┘               │
│       │              │              │                     │
│  ┌────▼──────────────▼──────────────▼─────┐              │
│  │   LLM Provider (OpenAI / Ollama)       │              │
│  │   + ContextManager (会话压缩)           │              │
│  └────────────────────────────────────────┘              │
│       │                                                   │
│  ┌────▼────────────┐  ┌──────────────────┐               │
│  │ CommandRunner    │  │ PTT 任务树        │               │
│  │ (安全执行+超时)   │  │ (UCB1 调度)       │               │
│  └────┬────────────┘  └──────────────────┘               │
│       │                                                   │
│  ┌────▼────────────┐  ┌──────────────────┐               │
│  │ Validator        │  │ Database (SQLite) │               │
│  │ (命令+范围校验)   │  │ FileStore (JSON)  │               │
│  └─────────────────┘  └──────────────────┘               │
└─────────────────────────────────────────────────────────┘
```

### 执行流程

```
1. Reasoning 分析 PTT → 选择最优任务
           ↓
2. Generation 生成可执行命令（适配 OS）
           ↓
3. Validator 校验命令安全性 + 范围合规
           ↓
4. CommandRunner 执行命令（超时控制）
           ↓
5. Parsing 解析工具输出 → 提取关键发现
           ↓
6. Reasoning 更新 PTT → 回到步骤 1
```

---

## 🖥️ Web 仪表盘

ViperStrike 内置赛博朋克风格 Web 仪表盘，提供：

- **实时进度追踪** — 命令执行进度条、耗时统计
- **PTT 任务树可视化** — 树形展示所有渗透任务及状态
- **发现列表 (Findings)** — 漏洞、开放端口、凭据等实时更新
- **操作日志** — 所有命令执行记录
- **图表统计** — 漏洞严重性分布、任务状态、执行时间线
- **系统监控** — CPU / 内存 / 网络实时指标
- **项目管理** — 创建 / 加载 / 删除项目
- **在线配置** — 实时修改所有配置参数
- **报告管理** — 生成报告 + 查看历史报告 + 下载

---

## 🚀 快速开始

### 环境要求

| 项目 | 要求 |
|------|------|
| **操作系统** | Kali Linux（推荐）/ Ubuntu / macOS |
| **Python** | 3.10+ |
| **LLM** | OpenAI 兼容 API 或 Ollama 本地模型 |
| **工具链** | nmap, gobuster, nikto 等（Kali 自带） |

### 安装

```bash
git clone https://github.com/kk12-30/ViperStrike.git
cd ViperStrike
pip install -r requirements.txt
```

### 配置

编辑 `config/config.yaml`：

```yaml
llm:
  provider: "openai"                    # openai 或 ollama
  api_key: "${OPENAI_API_KEY}"          # API Key（推荐用环境变量）
  base_url: "https://api.openai.com/v1" # 兼容 SiliconFlow / Azure 等
  model: "gpt-4o"
  temperature: 0.3
  max_tokens: 4096
  context_window: 128000

  # 使用 Ollama 本地模型时：
  # provider: "ollama"
  # ollama:
  #   base_url: "http://localhost:11434"
  #   model: "qwen2.5:32b"

execution:
  default_timeout: 300                  # 命令默认超时 5 分钟
  max_timeout: 1800                     # 命令最大超时 30 分钟
  auto_approve_risk_levels: ["low"]     # 自动审批低风险命令
  parallel_workers: 2                   # 并行执行步骤数

security:
  network_scope_enforcement: true       # 启用范围校验
```

设置 API Key：

```bash
export OPENAI_API_KEY="your-api-key-here"
```

### 启动

#### Web 模式（推荐）

```bash
python -m src.main web --port 5000
```

浏览器打开 `http://localhost:5000`，在 Web 界面中管理项目和执行扫描。

#### 命令行模式

```bash
# 创建新项目并启动扫描
python -m src.main start -n "TestProject" -t 192.168.1.100 -s 192.168.1.0/24

# 恢复已有项目
python -m src.main resume -p <project_id>

# 查看项目列表
python -m src.main projects

# 查看 PTT 状态
python -m src.main tree -p <project_id>

# 生成报告
python -m src.main report -p <project_id>
```

#### 交互式模式

```bash
python -m src.main interactive

viperstrike > new TestProject 192.168.1.100
viperstrike > start
viperstrike > tree
viperstrike > guide "尝试对80端口进行SQL注入"
viperstrike > pause
viperstrike > resume
viperstrike > findings
viperstrike > report
viperstrike > quit
```

---

## 🔑 关键技术

### 渗透测试任务树 (PTT)

PTT 是 ViperStrike 的核心数据结构，用树形结构维护整个渗透测试的全局状态：

- **根节点**：渗透目标（IP / 域名 / 网段）
- **子节点**：侦察、枚举、漏洞利用等阶段任务
- **叶节点**：具体可执行的原子操作
- **UCB1 调度**：平衡"探索未尝试任务"与"利用高回报路径"

### 上下文窗口管理

LLM 的上下文窗口有限，ViperStrike 采用：
- 每个模块维护**独立会话**
- 超限时自动**压缩旧消息为摘要**
- 始终保留 system prompt + 摘要 + 最近 N 轮对话

### 安全执行

- **命令黑名单**：阻止 `rm -rf`、`dd if=`、`mkfs` 等危险操作
- **范围校验**：确保目标 IP/域名在授权范围内
- **风险分级审批**：low / medium / high，可配置自动审批级别
- **超时控制**：双层限制（default_timeout + max_timeout）

---

## ⚠️ 安全声明

> **ViperStrike 仅用于已获授权的合法安全测试。未经授权的渗透测试是违法行为。**

使用前请确保：

1. ✅ 已获得目标系统所有者的**明确书面授权**
2. ✅ 测试范围已明确定义并配置在 `scope` 中
3. ✅ 已启用 `network_scope_enforcement: true`
4. ✅ 理解并接受相关法律责任

---

## 📜 License

[MIT License](LICENSE)

---

<div align="center">

**Star ⭐ this repo if you find it useful!**

Made with 🐍 by [kk12-30](https://github.com/kk12-30)

</div>
