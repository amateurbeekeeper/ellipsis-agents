# ellipsis-agents

Cloud AI agents for the [Ellipsis](https://github.com/amateurbeekeeper/ellipsisandco) repo, running on Cloudflare via [moltworker](https://github.com/cloudflare/moltworker).

Interact via Discord. Each thread routes to a different agent with its own role and persona. Agents clone the Ellipsis repo, do work, and open PRs. You review locally.

**This repo never runs locally.** Deploy only — via `npm run deploy`.

---

## Agents

| Discord thread | Agent | Scope |
|---|---|---|
| `#agent-backend` | Backend Engineer | `apps/api/` — NestJS, TypeORM, tests |
| `#agent-frontend` | Frontend Engineer | `apps/web/`, `apps/admin/` — Next.js, Tailwind |
| `#agent-product` | Product Advisor | `docs/` — prioritisation, requirements (no code) |

Persona files: `agents/<role>/SOUL.md` (personality) and `AGENTS.md` (rules).

---

## Architecture

```
Discord (threads)
    ↓
Cloudflare Worker (moltworker proxy)
    ↓
Cloudflare Sandbox container (OpenClaw gateway)
    ↓
OpenClaw agents → git clone ellipsisandco → open PRs
```

This repo is a fork of [cloudflare/moltworker](https://github.com/cloudflare/moltworker) with:
- Agent persona files added (`agents/`)
- `Dockerfile` extended to COPY agent files into the container
- `start-openclaw.sh` patched to inject multi-agent + Discord thread bindings

---

## Cost

~$34.50/mo running 24/7 (Workers Paid $5 + Cloudflare Container compute).
Set `SANDBOX_SLEEP_AFTER=10m` to sleep when idle — reduces to ~$10/mo for light use.

---

## Prerequisites

- Cloudflare account on **Workers Paid plan** ($5/mo)
- Cloudflare Containers enabled: dash.cloudflare.com → Workers → Containers
- Discord bot with **Message Content Intent** and **Server Members Intent** enabled
- Anthropic API key
- GitHub fine-grained token with read/write on `ellipsisandco` (for agents to push PRs)

---

## Setup

### 1. Install dependencies

```bash
npm install
```

### 2. Set secrets

```bash
# Required: AI
npx wrangler secret put ANTHROPIC_API_KEY

# Required: gateway token (generate one and save it)
export TOKEN=$(openssl rand -hex 32)
echo "Save this: $TOKEN"
echo "$TOKEN" | npx wrangler secret put MOLTBOT_GATEWAY_TOKEN

# Required: Discord
npx wrangler secret put DISCORD_BOT_TOKEN

# Required: Cloudflare Access (protects admin UI)
npx wrangler secret put CF_ACCESS_TEAM_DOMAIN   # e.g. myteam.cloudflareaccess.com
npx wrangler secret put CF_ACCESS_AUD           # from your Access application

# Required: GitHub token for agents to push PRs
npx wrangler secret put GITHUB_TOKEN

# Optional: sleep when idle to save cost
npx wrangler secret put SANDBOX_SLEEP_AFTER     # e.g. 10m

# Optional: R2 persistence (recommended — keeps paired devices + history across restarts)
npx wrangler secret put R2_ACCESS_KEY_ID
npx wrangler secret put R2_SECRET_ACCESS_KEY
npx wrangler secret put CF_ACCOUNT_ID
```

### 3. Set up Cloudflare Access

Protects the admin UI at `/_admin/`. Steps:
1. Go to Workers & Pages dashboard → select your worker → Settings → Domains & Routes
2. Click `...` next to the `workers.dev` row → Enable Cloudflare Access
3. In Zero Trust → Access → Applications, add your email to the allow list
4. Copy the AUD tag → set as `CF_ACCESS_AUD` secret above
5. Redeploy after setting secrets

### 4. Deploy

```bash
npm run deploy
```

First deploy takes 2-3 minutes. Open the Control UI after:

```
https://ellipsis-agents.<your-subdomain>.workers.dev/?token=YOUR_GATEWAY_TOKEN
```

### 5. Discord setup

1. Go to [discord.com/developers](https://discord.com/developers/applications)
2. Create application → Bot
3. Enable **Message Content Intent** + **Server Members Intent** under Bot → Privileged Gateway Intents
4. Copy bot token → set as `DISCORD_BOT_TOKEN` secret
5. Invite bot: OAuth2 → URL Generator → scopes: `bot`, `applications.commands` → permissions: Send Messages, Read Message History, Create Public Threads
6. In your Discord server, create three threads named exactly:
   - `agent-backend`
   - `agent-frontend`
   - `agent-product`
7. Pair your Discord account via admin UI at `/_admin/`

### 6. GitHub token for agents

Create a fine-grained personal access token at github.com/settings/tokens with:
- Repository access: `ellipsisandco` only
- Permissions: Contents (read+write), Pull requests (read+write), Metadata (read)

Set it: `npx wrangler secret put GITHUB_TOKEN`

Each agent's AGENTS.md already has the repo URL. They'll use this token to clone and push branches.

---

## Daily use

Message in a Discord thread → the bound agent responds and can open PRs.
You work locally in Claude Code as normal. Review PRs in GitHub.

The agents never touch your local machine. They run entirely in the Cloudflare container.

---

## Updating agent personas

Edit `agents/<role>/SOUL.md` or `agents/<role>/AGENTS.md`, then:

```bash
npm run deploy
```

---

## Secrets reference

| Secret | Required | Description |
|---|---|---|
| `ANTHROPIC_API_KEY` | Yes | Anthropic API key |
| `MOLTBOT_GATEWAY_TOKEN` | Yes | Token for Control UI access |
| `DISCORD_BOT_TOKEN` | Yes | Discord bot token |
| `GITHUB_TOKEN` | Yes | Fine-grained token for ellipsisandco repo |
| `CF_ACCESS_TEAM_DOMAIN` | Yes | Cloudflare Access team domain |
| `CF_ACCESS_AUD` | Yes | Cloudflare Access application audience |
| `SANDBOX_SLEEP_AFTER` | No | e.g. `10m` — sleep when idle to reduce cost |
| `R2_ACCESS_KEY_ID` | No | R2 for persistence across restarts |
| `R2_SECRET_ACCESS_KEY` | No | R2 secret key |
| `CF_ACCOUNT_ID` | No | Cloudflare account ID (needed for R2) |

---

## Links

- [ellipsisandco](https://github.com/amateurbeekeeper/ellipsisandco) — the product repo
- [moltworker](https://github.com/cloudflare/moltworker) — upstream (this is a fork)
- [OpenClaw docs](https://docs.openclaw.ai/) — agent framework docs
- [Cloudflare Sandbox docs](https://developers.cloudflare.com/sandbox/)
