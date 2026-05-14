# DeepSeek V4 PRO Triple CLI Deployment Skill

一键将 **DeepSeek V4 PRO** 部署为 **Claude Code / Codex CLI / Gemini CLI** 三大 AI 编程工具的底层模型。

## 架构

```
Claude Code ──▶ api.deepseek.com/anthropic ──▶ DeepSeek V4 PRO（直连）
Codex CLI   ──▶ Nyro :19530 ──▶ api.deepseek.com ──▶ DeepSeek V4 PRO
Gemini CLI  ──▶ ccswitch-deepseek :11435 ──▶ api.deepseek.com ──▶ DeepSeek V4 PRO
```

## 前置要求

- Node.js 18.x 或 20.x
- DeepSeek API Key（https://platform.deepseek.com/api_keys）
- Windows / macOS / Linux

## 安装此技能

### 方式一：通过 Codex skill-installer（推荐）

```bash
codex
# 在 Codex 中执行：
安装 $deepseek-v4-pro-deploy 从 https://github.com/<你的用户名>/deepseek-v4-pro-deploy-skill
```

### 方式二：手动复制

将本仓库克隆到 `~/.codex/skills/deepseek-v4-pro-deploy/` 目录。

## 使用

```bash
codex
# 对 Codex 说：
接入 DeepSeek V4 PRO
```

AI 会自动：
1. 检测 Node.js 环境
2. 安装 Claude Code / Codex CLI / Gemini CLI
3. 配置 CCSwitch（Claude Code 模型切换）
4. 配置 Nyro（Codex CLI 模型切换）
5. 部署 ccswitch-deepseek 代理（Gemini CLI 协议翻译）
6. 验证三个工具全部连通

你只需提供 DeepSeek API Key，其余全自动。

## 技能文件结构

```
deepseek-v4-pro-deploy-skill/
├── SKILL.md                      # 技能主指令
├── agents/openai.yaml            # UI 元数据
└── scripts/
    ├── ccswitch-deepseek.js      # Gemini CLI 协议翻译代理
    └── package.json              # Node.js 项目清单
```

## 定价参考（2026年5月）

| 计费项 | 价格 |
|--------|------|
| 输入（缓存命中） | ¥0.001 / 1K tokens |
| 输入（缓存未命中） | ¥0.012 / 1K tokens |
| 输出 | ¥0.024 / 1K tokens |

> 一个上午的编程 Agent 重度使用约消耗 ¥2。

## 相关工具

| 工具 | 用途 |
|------|------|
| Claude Code | Anthropic AI 编程 CLI |
| Codex CLI | OpenAI AI 编程 CLI |
| Gemini CLI | Google AI 编程 CLI |
| CCSwitch | Claude Code 模型切换桌面工具 |
| Nyro | AI 网关桌面工具 |

## 许可证

MIT