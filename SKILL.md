---
name: deepseek-v4-pro-deploy
description: >-
  Deploy DeepSeek V4 PRO as the underlying model for all three major AI
  coding CLI tools (Claude Code, Codex CLI, Gemini CLI) with automated
  environment detection, configuration, and proxy setup. Use when user
  wants to (1) Set up or switch any CLI tool to DeepSeek V4 PRO (2) Install
  or configure Claude Code, Codex CLI, Gemini CLI with DeepSeek
  (3) Deploy the ccswitch-deepseek protocol translation proxy
  (4) Troubleshoot DeepSeek connectivity issues in CLI tools
  (5) Set up Nyro or CCSwitch model switching tools.
  Bundles the ccswitch-deepseek proxy server script.
---
# DeepSeek V4 PRO Triple CLI Deployment

Deploy DeepSeek V4 PRO across Claude Code, Codex CLI, and Gemini CLI. The AI agent must autonomously execute every step without asking for permission per action.

## Core Rules for the AI Agent

- Proactively execute all commands, create files, edit configs. Never ask "should I run this?" — just do it and report results.
- The only user input needed: DeepSeek API Key. Everything else is auto-detected and auto-executed.
- Follow phases in order: Pre-setup → Claude Code → Codex CLI → Gemini CLI → Verification.
- Detect environment before each phase (OS, Node.js version, existing installs, port conflicts). Skip or execute based on results.
- On errors: reference the troubleshooting table below before stopping to ask the user.
- Use `update_plan` to track phases as tasks.

## Architecture Summary

```
Claude Code ──▶ api.deepseek.com/anthropic ──▶ DeepSeek V4 PRO (direct, no proxy)
Codex CLI   ──▶ Nyro :19530 ──▶ api.deepseek.com ──▶ DeepSeek V4 PRO
Gemini CLI  ──▶ ccswitch-deepseek :11435 ──▶ api.deepseek.com ──▶ DeepSeek V4 PRO
```

## Phase 1: Pre-setup

### 1.1 Verify Node.js
```powershell
node --version   # Expect v18.x or v20.x
npm --version
```
If missing: direct user to https://nodejs.org/ (LTS).

### 1.2 Get DeepSeek API Key
Ask user: "请提供你的 DeepSeek API Key（sk-xxx），在 https://platform.deepseek.com/api_keys 获取。最低充值 ¥10。"
Once received, store as `$DEEPSEEK_KEY` for all subsequent steps.

### 1.3 Install CLI tools (skip if already installed)
```powershell
npm install -g @anthropic-ai/claude-code
npm install -g @openai/codex
npm install -g @google/gemini-cli
```

## Phase 2: Claude Code

### 2.1 Install CCSwitch (model switcher)
- URL: https://github.com/farion1231/cc-switch/releases
- Windows: download `CC-Switch-v3.14.1-Windows.msi`, run it.
- macOS/Linux: download corresponding binary.

### 2.2 Configure Claude Code to use DeepSeek
Create/edit `~/.claude/.env` (or `%USERPROFILE%\.claude\.env` on Windows):
```env
ANTHROPIC_BASE_URL=https://api.deepseek.com/anthropic
ANTHROPIC_API_KEY=<DEEPSEEK_KEY>
```

### 2.3 Verify
```powershell
claude
# Type: "你好，你是什么大模型" → should respond DeepSeek V4 PRO
```

## Phase 3: Codex CLI

### 3.1 Install Nyro (AI gateway)
- Windows: `irm https://raw.githubusercontent.com/nyroway/nyro/master/scripts/install/install.ps1 | iex`
- macOS: `brew tap nyroway/nyro && brew install --cask nyro`
- Web UI: http://localhost:19531

### 3.2 Configure Nyro
In Nyro Web UI (http://localhost:19531):

**Add Provider**: Providers → New
| Field | Value |
|-------|-------|
| Name | `DeepSeek V4 Pro` |
| Protocol | `OpenAI Compatible` |
| Base URL | `https://api.deepseek.com/v1` |
| API Key | `<DEEPSEEK_KEY>` |

**Add Route**: Routes → New
| Field | Value |
|-------|-------|
| Virtual Model | `deepseek-v4-pro` |
| Provider | `DeepSeek V4 Pro` |
| Target Model | `deepseek-v4-pro` |

**Sync**: Connect → find route → click Sync next to Codex CLI.

### 3.3 Manual fallback (if Nyro unavailable)
Create/edit `~/.codex/config.toml`:
```toml
model_provider = "nyro"
model = "deepseek-v4-pro"
model_reasoning_effort = "high"
disable_response_storage = true

[model_providers.nyro]
name = "Nyro Gateway"
base_url = "http://localhost:19530/v1"
wire_api = "responses"
requires_openai_auth = true
```
> CRITICAL: `base_url` must NOT end with `/responses` — Codex auto-appends it, causing `/responses/responses` → 404.

### 3.4 Verify
```powershell
codex
# Type: "你好，你是什么大模型" → should respond DeepSeek V4 PRO
```

## Phase 4: Gemini CLI

Gemini CLI uses Google GenerateContent protocol — DeepSeek doesn't support it natively. The bundled `ccswitch-deepseek.js` proxy handles protocol translation.

### 4.1 Deploy ccswitch-deepseek proxy
Copy from this skill's bundled script:
```powershell
mkdir L:\00-projects\ccswitch-deepseek -Force
Copy-Item "{SKILL_DIR}\scripts\ccswitch-deepseek.js" "L:\00-projects\ccswitch-deepseek\index.js"
Copy-Item "{SKILL_DIR}\scripts\package.json" "L:\00-projects\ccswitch-deepseek\package.json"
cd L:\00-projects\ccswitch-deepseek
npm install
```

Create `.env`:
```env
api_key=<DEEPSEEK_KEY>
```

### 4.2 Start proxy
```powershell
cd L:\00-projects\ccswitch-deepseek
Start-Process node -ArgumentList "index.js" -WindowStyle Hidden
```

### 4.3 Configure Gemini CLI
Create/edit `~/.gemini/.env`:
```env
GOOGLE_GEMINI_BASE_URL=http://localhost:11435/v1beta
GEMINI_API_KEY=<DEEPSEEK_KEY>
GEMINI_MODEL=deepseek-v4-pro
```

Ensure `~/.gemini/settings.json`:
```json
{
  "security": {
    "auth": {
      "selectedType": "gemini-api-key"
    }
  }
}
```

### 4.4 Verify
```powershell
gemini
# Type: "hello" → should respond normally
```

## Phase 5: Final Verification

Run all three checks:
```powershell
# Claude Code
claude -c "你好，你是什么大模型"

# Codex CLI
codex -c "你好，你是什么大模型"

# Gemini CLI
gemini -c "你好，你是什么大模型"
```
All three should identify as DeepSeek V4 PRO.

## Troubleshooting

| Problem | Fix |
|---------|-----|
| Claude Code connection failed | Check `~/.claude/.env` API Key; verify `api.deepseek.com` reachable |
| Codex CLI 404 | Nyro running? `netstat -ano \| findstr "19530"`; `config.toml` `base_url` must end with `/v1` |
| Gemini CLI `type: null` error | Must route through ccswitch-deepseek (`:11435/v1beta`), not Nyro |
| ccswitch-deepseek won't start | Check `.env` API Key; kill old process on port 11435: `taskkill /F /PID <pid>` |
| Node.js too old | Upgrade to v18+ from https://nodejs.org/ |
| Port 11435 occupied | `netstat -ano \| findstr "11435"` → `taskkill /F /PID <pid>` |

## Quick Model Switching
- **Claude Code**: Use CCSwitch tray icon to select different provider.
- **Codex CLI**: In Nyro Web UI, modify route target or create new route + sync.
- **Gemini CLI**: Edit `~/.gemini/.env` `GOOGLE_GEMINI_BASE_URL` to point to different proxy.

## Bundled Resources

### scripts/ccswitch-deepseek.js
Node.js HTTP proxy that translates: Gemini GenerateContent ↔ OpenAI Chat Completions, and Codex Responses API ↔ Chat Completions. Handles SSE streaming, tool schema fixes (`type: null` → `type: "object"`), and full message format translation.

### scripts/package.json
Project manifest with `dotenv` dependency and `"type": "module"`.