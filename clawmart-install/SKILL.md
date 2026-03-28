---
name: clawmart-install
description: "Search and install an OpenClaw configuration pack from the ClawMart marketplace. Use when: (1) installing a personality pack, soul, agents, memory, or skill bundle from ClawMart, (2) searching for community OpenClaw configs, (3) user says 帮我安装 or install from clawmart or 从ClawMart下载. NOT for: uploading your own config (use clawmart-upload)."
metadata:
  {"openclaw": {"emoji": "🛒", "homepage": "https://clawmart-gray.vercel.app"}}
---

# ClawMart Install

You are helping the user find and install an OpenClaw configuration pack from ClawMart. Follow these steps exactly.

## Configuration

- ClawMart API base URL: `https://clawmart-gray.vercel.app`
- Config file: `~/.openclaw/clawmart-config.json`
- Install target: `~/.openclaw/workspace/` (config files)
- Skills target: `~/.openclaw/skills/` (skill folders)
- Backup directory: `~/.openclaw/backups/`

---

## Step 1: Check API Token

Read `~/.openclaw/clawmart-config.json`. If the file does not exist or `token` is empty:

Tell the user:
> 需要一个 ClawMart API Token 才能下载。请访问 https://clawmart-gray.vercel.app/dashboard/tokens 生成一个 Token，然后把它粘贴给我。

Once the user provides a token (format: `cm_` followed by hex characters), save:

```json
{
  "token": "<user_provided_token>",
  "base_url": "https://clawmart-gray.vercel.app"
}
```

Write to `~/.openclaw/clawmart-config.json`.

---

## Step 2: Determine Search Query

Extract the pack name from the user's message. If the user said「帮我安装深度研究分析师」, the query is「深度研究分析师」.

If no name was mentioned:
> 你想安装哪个 Pack？请输入关键词搜索。

---

## Step 3: Search ClawMart

```
GET {base_url}/api/packs/search?q={query}&limit=8
```

If `packs` array is empty:
> 没有找到与「{query}」相关的 Pack。请尝试其他关键词。

Then stop.

Otherwise display numbered results:

```
找到以下 Pack：

1. 深度研究分析师 Pro  ⭐ 4.8 · ↓ 1.2K
   @researcher_li · 「专注于学术文献和市场研究的全面分析配置」

2. 投研助手 Lite  ⭐ 4.5 · ↓ 856
   @quant_wang · 「轻量版投研配置，适合日常使用」

请选择要安装的 Pack（输入序号，或输入 0 取消）：
```

Wait for user input.

---

## Step 4: Show Pack Details

Fetch and display the selected pack's details, then ask for confirmation:

```
Pack 详情：
─────────────────────────────
标题：深度研究分析师 Pro
创作者：@researcher_li · 版本 v2.1.0
评分：⭐ 4.8 (127 评分) · ↓ 1,234 次下载

包含内容：
  配置文件：SOUL, AGENTS, BOOT, MEMORY
  Skills（自制）：deep-research/, my-translator/

描述：
专注于学术文献和市场研究的全面分析配置...
─────────────────────────────

确认安装？(y/n)
```

---

## Step 5: Check for Conflicts

Check files in `~/.openclaw/workspace/` and `~/.openclaw/skills/`.

If any files would be overwritten, list them:

```
以下文件已存在，安装后将覆盖（原文件自动备份到 ~/.openclaw/backups/2026-03-28-143022/）：

  workspace/claude.soul.md   ← 当前文件将备份
  skills/deep-research/      ← 新内容将覆盖

确认继续？(y/n)
```

---

## Step 6: Download Pack

```
POST {base_url}/api/packs/{slug}/download
Authorization: Bearer {token}
X-Download-Mode: signed-url
Content-Type: application/json
```

Parse the `url` and `filename` from the response. Download the zip to `/tmp/clawmart-install-{random}.zip`.

---

## Step 7: Backup Conflicting Files

If conflicts found:

1. Create `~/.openclaw/backups/{YYYY-MM-DD-HHmmss}/`
2. Copy each conflicting file/folder there
3. Confirm: `已备份 {n} 个文件/目录到 ~/.openclaw/backups/{timestamp}/`

---

## Step 8: Extract and Install

Unzip. For each item:

- Files at the **root** of the zip → `~/.openclaw/workspace/`
- Folders inside `skills/` subdirectory → `~/.openclaw/skills/{folder-name}/`

After extracting each skill folder, write an origin marker so future uploads know this skill came from ClawMart (not user-created):

```json
// ~/.openclaw/skills/{folder-name}/.clawhub/origin.json
{
  "source": "clawmart",
  "pack_slug": "{slug}",
  "pack_title": "{title}",
  "installed_at": "{ISO timestamp}",
  "url": "{base_url}/packs/{slug}"
}
```

Create all directories as needed.

---

## Step 9: Confirm Installation

```
✅ 安装完成！

已安装到 ~/.openclaw/workspace/：
  · claude.soul.md
  · research.agents.md
  · deep_analysis.boot.md
  · memory_projects.json

已安装到 ~/.openclaw/skills/：
  · deep-research/
  · my-translator/

备份位置：~/.openclaw/backups/2026-03-28-143022/

重启 OpenClaw 后新配置即可生效。
```

---

## Step 10: Cleanup

Delete `/tmp/clawmart-install-{random}.zip`.

---

## Notes

- Origin markers in skill folders tell `clawmart-upload` that these are external skills, preventing accidental re-upload
- If token is rejected (401): `{base_url}/dashboard/tokens`
- Pack detail page: `{base_url}/packs/{slug}`
