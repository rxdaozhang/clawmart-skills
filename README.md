# ClawMart Skills for OpenClaw

Two OpenClaw skills for uploading and installing configuration packs from [ClawMart](https://clawmart-gray.vercel.app) — the community marketplace for OpenClaw AI personality configurations.

## Skills

| Skill | Description |
|-------|-------------|
| `clawmart-install` | Search and install a config pack from ClawMart |
| `clawmart-upload` | Package and upload your OpenClaw config to ClawMart |

## Install via OpenClaw conversation

Once OpenClaw is running, just say:

> 帮我安装 clawmart-install 和 clawmart-upload

Or use clawhub CLI directly:

```bash
clawhub install clawmart-install
clawhub install clawmart-upload
```

## Setup

1. Sign in at [clawmart-gray.vercel.app](https://clawmart-gray.vercel.app) with GitHub or Google
2. Go to **Dashboard → API Tokens** and generate a token
3. The first time you use either skill, it will ask for your token and save it to `~/.openclaw/clawmart-config.json`

## Usage

### Install a pack

Tell your OpenClaw agent:
> 帮我安装深度研究分析师

### Upload your config

Tell your OpenClaw agent:
> 把我的配置上传到 ClawMart

## Token Security

Your API token is stored locally in `~/.openclaw/clawmart-config.json`. It is never uploaded or shared. Revoke tokens anytime at [clawmart-gray.vercel.app/dashboard/tokens](https://clawmart-gray.vercel.app/dashboard/tokens).
