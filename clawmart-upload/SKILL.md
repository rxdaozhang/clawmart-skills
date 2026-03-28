---
name: clawmart-upload
description: "Upload your original OpenClaw configuration to the ClawMart marketplace. Use when: (1) sharing your own agent personality, soul, agents, or self-written skills with others, (2) publishing a new version of an existing pack you own. NOT for: installing packs from others (use clawmart-install), re-uploading skills downloaded from ClawHub or GitHub."
metadata:
  {"openclaw": {"emoji": "📦", "homepage": "https://clawmart-gray.vercel.app"}}
---

# ClawMart Upload

You are helping the user package and upload their **original** OpenClaw configuration to the ClawMart marketplace. Follow these steps exactly.

## Configuration

- ClawMart API base URL: `https://clawmart-gray.vercel.app`
- Config file: `~/.openclaw/clawmart-config.json`
- API endpoint: `POST {base_url}/api/packs`

---

## Step 1: Check API Token

Read `~/.openclaw/clawmart-config.json`. If the file does not exist or `token` is empty:

Tell the user:
> 需要一个 ClawMart API Token 才能上传。请访问 https://clawmart-gray.vercel.app/dashboard/tokens 生成一个 Token，然后把它粘贴给我。

Once the user provides a token (format: `cm_` followed by hex characters), save it:

```json
{
  "token": "<user_provided_token>",
  "base_url": "https://clawmart-gray.vercel.app"
}
```

Write this to `~/.openclaw/clawmart-config.json`.

---

## Step 2: Scan Workspace Files

### 2a. Config files

Scan `~/.openclaw/workspace/` for these patterns:

| Pattern | Type |
|---------|------|
| `*.soul.md` | SOUL |
| `*.agents.md` | AGENTS |
| `*.boot.md` | BOOT |
| `*.heartbeat.md` | HEARTBEAT |
| `memory_*.json` or `memory-*.json` | MEMORY |

### 2b. Skill folders — distinguish original vs. external

Scan `~/.openclaw/skills/` for all subdirectories containing a `SKILL.md` file.

For each skill folder:
- If the folder contains `.clawhub/origin.json` → it was **installed from ClawHub/GitHub** — mark as **[外部]** and **exclude by default**
- If no `.clawhub/origin.json` exists → it is **user-created** — mark as **[自制]** and **include by default**

Always exclude `clawmart-upload` and `clawmart-install` regardless.

### 2c. Show the complete file list to the user

Show exactly which files/folders will be included or excluded:

```
找到以下 OpenClaw 配置文件：

配置文件（默认全部包含）：
  ✓ SOUL      claude.soul.md
  ✓ AGENTS    coding.agents.md, research.agents.md
  ✓ BOOT      startup.boot.md
  ✓ MEMORY    memory_projects.json

Skills：
  ✓ [自制]  deep-research/      ← 将上传
  ✓ [自制]  my-translator/      ← 将上传
  ✗ [外部]  coding-agent/       ← 跳过（来自 ClawHub，非原创）
  ✗ [外部]  clawmart-install/   ← 跳过（系统 skill）

是否全部包含？或者想调整某个文件？(全部包含/输入要排除或添加的文件名)
```

Wait for user confirmation. Adjust the list based on user input.

**Important**: If the user insists on including an [外部] skill, warn them:
> ⚠️ {skill-name} 不是你原创的（来自外部安装），上传他人 skill 可能违反原作者版权。建议只上传自制 skill。是否确认包含？(y/n)

---

## Step 3: Sensitive Information Check

Scan all included non-SKILL files for sensitive patterns:

- Strings matching `(sk-|cm_|ghp_|ghs_|ghu_)[A-Za-z0-9]{20,}` (API keys/tokens)
- Strings matching `(password|passwd|secret|api_key)\s*[:=]\s*\S+` (credentials)
- Any string longer than 20 chars after `Bearer ` or `Token `

If any sensitive pattern is found, show the file name, line number, and masked value, and ask:
> 在 {filename} 第 {line} 行发现疑似敏感信息：`{masked_value}`。建议移除后再上传。是否继续上传（y）或先编辑文件（n）？

Only proceed if user says yes.

---

## Step 4: Collect Pack Metadata

Ask the user for:

1. **标题 (Title)**：这个 Pack 的名称是什么？（例如：深度研究分析师）
2. **描述 (Description)**：简单介绍一下这个 Pack 的用途和特点（可选）
3. **版本 (Version)**：版本号是多少？（默认：`1.0.0`）

Check ClawMart for an existing pack with the same title:
```
GET {base_url}/api/packs/search?q={title}
Authorization: Bearer {token}
```

If a matching pack you own already exists, ask:
> 找到同名 Pack「{title}」。是否上传为新版本 {new_version}？(y/n)

---

## Step 5: Create ZIP Package

Create a ZIP file at `/tmp/clawmart-{random}.zip` with this structure:

```
pack.zip
├── claude.soul.md          ← config files at root
├── coding.agents.md
├── startup.boot.md
├── memory_projects.json
└── skills/
    ├── deep-research/      ← each self-made skill as a folder
    │   └── SKILL.md
    └── my-translator/
        └── SKILL.md
```

Rules:
- Config files (SOUL, AGENTS, BOOT, HEARTBEAT, MEMORY) go at the **root** of the zip
- Skill folders go inside a `skills/` subdirectory, preserving the folder structure
- Verify the zip contains no `../` path traversal

---

## Step 6: Upload to ClawMart

```
POST {base_url}/api/packs
Authorization: Bearer {token}
Content-Type: multipart/form-data

Fields:
  file: <the zip file>
  title: <user provided title>
  description: <user provided description>
  version: <version>
```

**On success** (HTTP 201):
> ✅ 上传成功！
>
> Pack 「{title}」已提交审核，通常在 24 小时内完成。
> 查看状态：{base_url}/dashboard/packs

**On error**: show the error message and stop.

---

## Step 7: Cleanup

Delete the temporary zip file.

---

## Notes

- Only upload files you created yourself
- The token is stored locally at `~/.openclaw/clawmart-config.json` and reused
- If the token is rejected (401), generate a new one at `{base_url}/dashboard/tokens`
