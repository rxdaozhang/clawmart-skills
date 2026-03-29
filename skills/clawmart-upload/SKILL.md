---
name: clawmart-upload
description: Upload your current OpenClaw configuration to the ClawMart marketplace
version: 1.1.0
triggers:
  - "upload to clawmart"
  - "share my config on clawmart"
  - "publish to clawmart"
  - "upload my pack"
---

# ClawMart Upload Skill

You are helping the user upload their OpenClaw configuration to the ClawMart marketplace. Follow these steps exactly and in order.

## Configuration

- ClawMart API base URL: `https://clawmart-gray.vercel.app`
- Config file: `~/.openclaw/clawmart-config.json`
- API endpoint: `POST {base_url}/api/packs`

---

## Step 1: Check API Token

Read `~/.openclaw/clawmart-config.json`. If the file does not exist or `token` is empty:

Tell the user:
> You need a ClawMart API Token to upload. Please visit https://clawmart-gray.vercel.app/dashboard/tokens to generate one, then paste it here.

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

Look for OpenClaw configuration files in the current directory and `~/.openclaw/workspace/`. Collect all files matching these patterns:

| Pattern | Type |
|---------|------|
| `*.soul.md` | SOUL |
| `*.agents.md` | AGENTS |
| `*.boot.md` | BOOT |
| `*.heartbeat.md` | HEARTBEAT |
| `memory_*.json` or `memory-*.json` | MEMORY |
| `*.skill.md` | LOCAL SKILLS |

**Exclude** any file named `clawmart-upload.skill.md` or `clawmart-install.skill.md` from the list.

### External Skills Detection

Also scan for externally installed skills from these locations:
- `~/.claude/skills/` — user-installed skills
- `~/.claude/plugins/*/skills/` — plugin-provided skills

For each external skill found, read its `SKILL.md` frontmatter and extract:
- `name` — skill name
- `source` — origin (plugin package name or directory path)
- `version` — version field (default to `unknown` if missing)

These are collected as metadata only — their file contents are **not** included in the zip.

Show the user a summary:

```
Found the following OpenClaw configuration files:

SOUL:          claude.soul.md
AGENTS:        coding.agents.md, research.agents.md
BOOT:          startup.boot.md
MEMORY:        memory_projects.json
LOCAL SKILLS:  my-workflow.skill.md, my-tools.skill.md
EXTERNAL SKILLS (metadata only):
  - jd-interview-prep  (source: ~/.claude/skills/jd-interview-prep, v1.0.0)
  - nextjs             (source: plugin:vercel, v5.0.6)
  - ai-sdk             (source: plugin:vercel, v5.0.6)

Include all? Or exclude specific files? (all / enter filenames to exclude)
```

Wait for user confirmation. Adjust the file list based on user input.

---

## Step 3: Sensitive Information Check

Before packaging, scan the content of all non-SKILLS files for sensitive patterns:

- Strings matching `(sk-|cm_|ghp_|ghs_|ghu_)[A-Za-z0-9]{20,}` (API keys/tokens)
- Strings matching `(password|passwd|secret|api_key)\s*[:=]\s*\S+` (credentials)
- Any string longer than 20 chars after `Bearer ` or `Token `

If any sensitive pattern is found, tell the user exactly which file and line, and ask:
> Sensitive information detected in {filename} at line {line}: `{masked_value}`. It is recommended to remove it before uploading. Continue anyway? (y/n)

Only proceed if user says yes.

---

## Step 4: Collect Pack Metadata

Ask the user for:

1. **Title**: What is the name of this Pack? (e.g., Deep Research Analyst)
2. **Description**: Brief description of the Pack's purpose and features (optional)
3. **Version**: Version number? (default: `1.0.0`)

Check ClawMart if the user already has a pack with the same title:
```
GET {base_url}/api/packs/search?q={title}
Authorization: Bearer {token}
```

If a matching pack already exists, ask:
> A pack named "{title}" already exists. Upload as new version {new_version}? (y/n)

If yes, note this for the upload.

---

## Step 5: Create ZIP Package

Create a ZIP file in a temporary location (e.g., `/tmp/clawmart-{random}.zip`) containing:

- All confirmed local files from Step 2, placed at the root of the zip
- **No subdirectories** for non-skill files
- LOCAL SKILLS files placed in a `skills/` subdirectory within the zip
- A `skills-manifest.json` file in the zip root listing external skills metadata:

```json
{
  "external_skills": [
    {
      "name": "jd-interview-prep",
      "source": "~/.claude/skills/jd-interview-prep",
      "version": "1.0.0"
    },
    {
      "name": "nextjs",
      "source": "plugin:vercel",
      "version": "5.0.6"
    }
  ]
}
```

Verify the zip does not contain any `../` path traversal patterns.

---

## Step 6: Upload to ClawMart

Send the upload request:

```
POST {base_url}/api/packs
Authorization: Bearer {token}
Content-Type: multipart/form-data

Fields:
  file:        <the zip file>
  title:       <user provided title>
  description: <user provided description>
  version:     <version>
```

**On success** (HTTP 201), tell the user:
> Pack "{title}" has been submitted for review. It is typically approved within 24 hours.
> View status: {base_url}/dashboard/packs

**On error**, show the error message and stop.

---

## Step 7: Cleanup

Delete the temporary zip file created in Step 5.

---

## Notes

- Local skill file **contents** are included in the zip under `skills/`
- External skills are recorded in `skills-manifest.json` — name, source, and version only, no file content
- The token is stored locally and reused on future uploads
- If the token is rejected (401), ask the user to generate a new one at `{base_url}/dashboard/tokens`
