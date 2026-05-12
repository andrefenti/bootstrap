# Bootstrap — Claude Code Reinstall Manual

> Full reinstall guide for **andrefenti** (email: andrefenti2@gmail.com).
> If you lose access to your computer and have only Claude Code, follow this guide top to bottom.
> **Last updated: 2026-05-12**

---

## 0. Overview of What This Machine Runs

| System | Where | Purpose |
|--------|-------|---------|
| **Helvora** frontend | `C:\Users\Acer\Helvora` | Production SaaS (helvora.store) — React/Vite/TypeScript |
| **Awawa Bridge** | `C:\Users\Acer\Helvora\awawa-bridge` | AI agent bridge server (10 agents powered by Claude Code) |
| **TacBox** | local (clone from GitHub) | Private side project |
| **Claude Catalog** | `~/claude-catalog` | Auto-syncing skills catalog |
| **Claude Code** | `~/.local/bin/claude` | AI CLI — the brain of everything |

---

## 1. Install Required Tools

### Node.js (v24.x LTS)
```
Download from https://nodejs.org — install LTS (v24+)
Verify: node --version && npm --version
```

### Git (2.52+)
```
Download from https://git-scm.com
Verify: git --version
```

### GitHub CLI
```bash
winget install GitHub.cli
# Then authenticate:
gh auth login
# Choose: GitHub.com → HTTPS → authenticate via browser
gh auth refresh -h github.com -s delete_repo
```

### Claude Code
```bash
npm install -g @anthropic-ai/claude-code
# Or download from https://claude.ai/code
# Verify install:
claude --version
# Authenticate (will open browser):
claude auth login
```

### Cloudflared (for Awawa Bridge tunnel)
```
Download from https://developers.cloudflare.com/cloudflare-one/connections/connect-networks/downloads/
Install to a folder on PATH (e.g. C:\cloudflared\cloudflared.exe)
Verify: cloudflared --version
```

### Python 3.14 (for settings.json hooks)
```
Download from https://python.org — install to C:\Python314\
Verify: C:\Python314\python --version
```

### Vercel CLI
```bash
npm install -g vercel
vercel login
# Authenticate with andrefenti2@gmail.com
```

### Supabase CLI (via npx — no global install needed)
```bash
# Test it works:
npx --yes supabase projects list
# Should show hcadzyaazewogwabggms and others — will prompt for login if not authed
npx --yes supabase login
```

---

## 2. Clone All Repositories

```bash
# Helvora (main SaaS)
git clone https://github.com/andrefenti/helvora C:\Users\%USERNAME%\Helvora
cd C:\Users\%USERNAME%\Helvora
npm install

# Awawa Bridge (AI agent server — SEPARATE private repo)
git clone https://github.com/andrefenti/awawa-bridge C:\Users\%USERNAME%\Helvora\awawa-bridge
cd C:\Users\%USERNAME%\Helvora\awawa-bridge
npm install

# Claude Catalog (skills auto-sync)
git clone https://github.com/andrefenti/claude-catalog ~/claude-catalog
```

> **NEVER clone or push to `andrefenti/helvora-811d1241`** — that is the dead Lovable repo.

---

## 3. Set Up Environment Variables

### Helvora frontend — `C:\Users\%USERNAME%\Helvora\.env`
```env
VITE_SUPABASE_URL=https://hcadzyaazewogwabggms.supabase.co
VITE_SUPABASE_PUBLISHABLE_KEY=<get from Supabase dashboard → Settings → API → anon/public key>
```

### Awawa Bridge — `C:\Users\%USERNAME%\Helvora\awawa-bridge\.env`
```env
AWAWA_API_KEY=51ad715757bc401188554091db4c6fa6e187646a044536b83f2b1384018a57f4
PORT=3141
```

### Supabase Edge Function secrets (set via CLI, not .env)
```bash
# After Supabase CLI is authed, restore all edge function secrets:
npx --yes supabase secrets list --project-ref hcadzyaazewogwabggms
# Re-set any missing ones:
npx --yes supabase secrets set KEY=value --project-ref hcadzyaazewogwabggms
```

Important edge function secrets include: `AWAWA_BRIDGE_URL`, `AWAWA_API_KEY`, `STRIPE_SECRET_KEY`, `STRIPE_WEBHOOK_SECRET`, `RESEND_API_KEY`, `WAPI_API_KEY`, `WAPI_BASE_URL`, `SUPABASE_SERVICE_ROLE_KEY`.

---

## 4. Configure Claude Code Settings

Create/overwrite `~/.claude/settings.json` with:

```json
{
  "permissions": {
    "allow": [
      "Bash(vercel:*)",
      "Bash(gh:*)",
      "Bash(git:*)",
      "Bash(pm2:*)",
      "Bash(npm:*)",
      "Bash(bun:*)",
      "Bash(npx:*)",
      "Bash(cloudflared:*)",
      "Bash(ngrok:*)",
      "Bash(supabase:*)",
      "Bash(curl:*)",
      "Bash(mkdir:*)",
      "Bash",
      "Read",
      "Edit",
      "Write",
      "WebFetch",
      "Grep",
      "Glob",
      "NotebookEdit",
      "WebSearch",
      "Agent",
      "mcp__*"
    ],
    "defaultMode": "bypassPermissions"
  },
  "model": "opusplan",
  "hooks": {
    "Setup": [
      {
        "matcher": "*",
        "hooks": [
          {
            "type": "command",
            "command": "_R=\"$HOME/.claude/plugins/marketplaces/thedotmack/plugin\"; \"$_R/scripts/setup.sh\"",
            "timeout": 300,
            "windowsHide": true
          }
        ]
      }
    ],
    "SessionStart": [
      {
        "matcher": "*",
        "hooks": [
          {
            "type": "command",
            "command": "curl -sf -X POST http://localhost:3141/bridge/pause -H 'Content-Type: application/json' -H 'x-api-key: 51ad715757bc401188554091db4c6fa6e187646a044536b83f2b1384018a57f4' -d '{\"reason\":\"Terminal session active\"}' >/dev/null 2>&1 || true",
            "timeout": 5,
            "windowsHide": true
          }
        ]
      },
      {
        "matcher": "startup|clear|compact",
        "hooks": [
          {
            "type": "command",
            "command": "_R=\"$HOME/.claude/plugins/marketplaces/thedotmack/plugin\"; node \"$_R/scripts/smart-install.js\" 2>/dev/null || true",
            "timeout": 300,
            "windowsHide": true
          },
          {
            "type": "command",
            "command": "_R=\"$HOME/.claude/plugins/marketplaces/thedotmack/plugin\"; node \"$_R/scripts/bun-runner.js\" \"$_R/scripts/worker-service.cjs\" start 2>/dev/null || true",
            "timeout": 60,
            "windowsHide": true
          }
        ]
      }
    ],
    "PostToolUse": [
      {
        "matcher": "Write|Edit",
        "hooks": [
          {
            "type": "command",
            "command": "FILE=$(/c/Python314/python -c \"import sys,json; d=json.load(sys.stdin); print(d.get('tool_input',{}).get('file_path',''))\" 2>/dev/null); case \"$FILE\" in *\".claude/skills\"*) cd \"$HOME/claude-catalog\" && bash sync.sh 2>/dev/null || true;; esac",
            "timeout": 30,
            "async": true,
            "windowsHide": true
          }
        ]
      }
    ],
    "SessionEnd": [
      {
        "hooks": [
          {
            "type": "command",
            "command": "curl -sf -X POST http://localhost:3141/bridge/resume -H 'Content-Type: application/json' -H 'x-api-key: 51ad715757bc401188554091db4c6fa6e187646a044536b83f2b1384018a57f4' >/dev/null 2>&1 || true",
            "timeout": 5,
            "windowsHide": true
          }
        ]
      }
    ]
  },
  "extraKnownMarketplaces": {
    "thedotmack": {
      "source": {
        "source": "github",
        "repo": "thedotmack/claude-mem"
      }
    }
  },
  "autoUpdatesChannel": "latest",
  "skipDangerousModePermissionPrompt": true
}
```

> **`model: "opusplan"`** means Opus plans, Sonnet executes. This is correct — it's not a downgrade.
> **`defaultMode: "bypassPermissions"`** is the equivalent of `--dangerously-skip-permissions` for every session.
> **Update the Python path** (`/c/Python314/python`) if Python was installed elsewhere.

---

## 5. Create the CLAUDE.md for Helvora

File: `C:\Users\%USERNAME%\Helvora\CLAUDE.md` — this is already in the git repo so it will be there after cloning. Read it carefully — it contains mandatory rules about:
- Edge function deployment (ALWAYS deploy after any `supabase/functions/**` change)
- Trigger column rename safety
- Stripe SDK v18 `constructEventAsync` pattern
- Email trigger rules (order_created vs new_customer_welcome)
- WAPI table schema rules (what goes in `order_wapi_snapshots`)
- Build verification before push

---

## 6. Set Up the Memory System

Memory files live at: `~/.claude/projects/C--Users-<username>\memory\`

After Claude Code is running, paste this into chat to restore all memories:

```
Read the HANDOVER.md from https://github.com/andrefenti/bootstrap and restore my full memory system from it. 
Create all memory files in ~/.claude/projects/C--Users-<username>/memory/ and populate MEMORY.md index.
```

### Key memories to restore manually if needed:

**User identity:**
- Name: Andre Fenti Yamamoto (andrefenti / andrefenti2@gmail.com)
- Claude is called "Awawa" and is the CEO of Helvora

**Project facts:**
- Helvora is a live SaaS at helvora.store (Brazil/PT-BR market)
- Supabase project ref: `hcadzyaazewogwabggms` (North EU/Stockholm) — this is the ACTIVE one
- Old Lovable-managed Supabase ref `ayypmxiesqjihkxtrkjn` is no longer accessible/used
- GitHub repo: `andrefenti/helvora` (NEVER `andrefenti/helvora-811d1241`)
- Vercel project ID: `prj_F7SmNv1tYi0qzS4XaTkV4RhkZP9l`
- Vercel team ID: `team_PYgzgTY4bUVpF0ONkPyGdiFf`

**Behavioral rules Claude must know:**
- Always show task progress: ✅ done, 🔄 in progress, ⬚ pending
- dev-browser always runs `--headless` unless user asks to see it or must interact
- Screenshots save to `C:\Users\%USERNAME%\screenshots\screenshot_YYYYMMDD_HHMMSS.png`
- Never suggest Lovable, OpenRouter, Groq, OpenAI, or Anthropic API — all AI goes through Claude Code (Awawa bridge)
- Email templates ONLY changed via /admin/email UI — never direct SQL
- Ship gate: never say "done" based on {success:true} alone — always verify in real system
- New tables in AdminOrdersMultiSheet need 3 registrations: useAdminData + dataByTable + loadingByTable
- All TABLE_COLS labels must be prefixed: helvora_, wapi_, stripe_
- Chat history (messages + images) must ALWAYS be visible for both customer and admin — never break this
- If blocked on permission prompt, ask immediately — never hang silently

---

## 7. Install Claude Code Skills

Skills live at `~/.claude/skills/`. After Claude Code is running, install via Claude chat:

```
Install my skills from the claude-catalog at ~/claude-catalog and from andrefenti/claude-catalog on GitHub.
```

Core mandatory skills (must be present):
- `init` — session bootstrap, sets bypassPermissions
- `fewer-permission-prompts` — auto-updates allowlist from transcripts
- `github-sync` — auto git add/commit/push after changes
- `vercel-deploy` — deploy to Vercel
- `self-healing` — writes skills for recurring patterns
- `know-me` — applies user context silently
- `dev-browser` — headless browser automation (from sawyerhood)
- `ship-gate` — verify before claiming done
- `email-template-update` — email template change workflow
- `column-label-prefix` — TABLE_COLS label enforcement
- `cs-fix` — customer support fix workflow
- `wapi-table-rule` — WAPI table schema enforcement

Additional skills to reinstall (from the catalog):
- All architect skills: `arquiteto-supabase`, `arquiteto-postgresql`, `arquiteto-stripe`, `arquiteto-oauth`, `arquiteto-openai`, `otimizador-postgresql`
- Security skills: `security`, `pentest-advisor`, `sqli-test`, `xss-test`, `api-keys`, `wordlist`, `webshell-detect`, `bug-bounty-hunter`, `ctf-assistant`, `auditor-seguranca`, `defesa-prompt-injection`, `gestor-segredos`
- Frontend skills: `frontend-design`, `designer-ux-acessibilidade`, `design-system-figma`, `design-for-ai`
- Operations: `researcher`, `customer-support`, `scalability`, `cost-reducer`, `revisor-codigo`, `observabilidade-incidentes`, `motor-conteudo`, `otimizador-tokens`, `scraping-etico`, `n8n`, `trigger-dev`, `operador-godaddy-dns`
- Shannon (AI pentester): `https://github.com/KeygraphHQ/shannon`

---

## 8. Start the Awawa Bridge

The bridge runs locally and is exposed via Cloudflare Tunnel.

```bash
cd C:\Users\%USERNAME%\Helvora\awawa-bridge
npm install
node server.js
```

Or use the npm script from the Helvora root:
```bash
cd C:\Users\%USERNAME%\Helvora
npm run bridge
```

**In a separate terminal**, start the Cloudflare tunnel:
```bash
cloudflared tunnel --url http://localhost:3141
```

Cloudflare will print a public URL like `https://xxx-xxx.trycloudflare.com`.

Copy that URL into `C:\Users\%USERNAME%\Helvora\.env` as:
```env
VITE_AWAWA_BRIDGE_URL=https://xxx-xxx.trycloudflare.com
```

Also set it as a Supabase edge function secret:
```bash
npx --yes supabase secrets set AWAWA_BRIDGE_URL=https://xxx-xxx.trycloudflare.com --project-ref hcadzyaazewogwabggms
```

### Bridge architecture
- **Port:** 3141
- **API key:** `51ad715757bc401188554091db4c6fa6e187646a044536b83f2b1384018a57f4`
- **10 agents:** awawa (CEO/Delegator), sage (Content), axion (BI), linkor (IT), chessie (Strategy), forexa (Finance), jayjay (Customer Service), brainda (Marketing), tico (Tax & Compliance), opsidium (Operations)
- All agents use `claude -p` (Claude Code CLI) — no third-party AI
- Awawa never executes tasks — always delegates to department heads

### Bridge endpoints
```
GET  /health              — no auth required, returns all agent statuses
POST /delegate            — Awawa picks agent + queues task
POST /awawa               — direct chat with Awawa
POST /agent/:agentId      — direct chat with any agent
GET  /tasks               — all agent statuses
GET  /task/:taskId        — task details + result
DELETE /task/:taskId      — cancel queued task
```
All endpoints except `/health` require header: `x-api-key: <AWAWA_API_KEY>`

---

## 9. Deploy Workflow

### Normal path (push to test branch → CI deploys)
```bash
git add <files>
git commit -m "..."
git push origin test
# GitHub Actions (.github/workflows/alias-test-domain.yml) runs vercel deploy + alias
# Wait ~2 min, verify at https://helvora-test.vercel.app
```

### Manual fallback
```bash
cd C:\Users\%USERNAME%\Helvora
npx --yes vercel --prod --yes
# Copy the deployment URL from output, then:
npx --yes vercel alias <deployment-url> helvora-test.vercel.app
```

### Deploy Supabase Edge Functions (MANDATORY after any function change)
```bash
npx --yes supabase functions deploy <function-name> --project-ref hcadzyaazewogwabggms
```

### URLs
| Environment | URL |
|-------------|-----|
| Production | https://helvora.store |
| Test | https://helvora-test.vercel.app |
| Vercel project | https://helvora-andre-fenti-yamamoto.vercel.app |
| Admin panel | https://helvora.store/admin/watchlist |
| Supabase dashboard | https://supabase.com/dashboard/project/hcadzyaazewogwabggms |

---

## 10. Key Accounts & Services

| Service | Account | Notes |
|---------|---------|-------|
| GitHub | andrefenti | `gh auth login` via browser |
| Supabase | andrefenti2@gmail.com | Active project: `hcadzyaazewogwabggms` (Stockholm) |
| Vercel | andrefenti2@gmail.com / team: andre-fenti-yamamoto | `vercel login` |
| GoDaddy | andrefenti2@gmail.com | DNS for helvora.store — A record `@` → `76.76.21.21` |
| Stripe | — | Keys in Supabase edge function secrets |
| Resend | — | Email delivery — key in Supabase secrets as `RESEND_API_KEY` |
| WAPI | — | Logistics integration — key in Supabase secrets as `WAPI_API_KEY` |
| Cloudflare | — | Tunnel for Awawa Bridge — no account needed (trycloudflare.com) |

---

## 11. Known Architecture Gotchas

### Supabase Edge Functions
- Always use `npx --yes supabase functions deploy <name> --project-ref hcadzyaazewogwabggms`
- Git push does NOT auto-deploy edge functions
- Stripe SDK v18 in Deno: ALWAYS use `await stripe.webhooks.constructEventAsync()` — never the synchronous version

### Column Renaming
- When renaming any DB column, search triggers for references to `OLD.<old_name>` and `NEW.<old_name>` — they fail silently at runtime

### Admin UI — AdminOrdersMultiSheet
- Main file: `src/pages/admin/AdminOrdersMultiSheet.tsx` (4652 lines)
- Adding a new table requires 3 registrations: `useAdminData`, `dataByTable`, `loadingByTable`
- All column labels must be prefixed: `helvora_`, `wapi_`, `stripe_`
- Target is to migrate all admin pages to this MultiSheet BI engine

### Email System
- Templates ONLY via `/admin/email` UI — never direct SQL
- `order_created` fires unconditionally on all successful payments
- `new_customer_welcome` fires ONLY when a brand-new account was created
- `payment_succeeded` event type is RETIRED — do not use

### Payment Flow
- If `paymentError` is not in useEffect guard AND deps → infinite retry loop → blank tab
- Always include error state in both the guard condition and the dependency array

### Chat Support
- Chat history (messages + images) must ALWAYS be visible for both customer and admin
- Admin: `src/pages/admin/CustomerSupportPanel.tsx`
- Customer: `SupportChatbot.tsx`
- Edge function: `supabase/functions/support-chat/index.ts`

---

## 12. Repositories

| Repo | Visibility | Description |
|------|------------|-------------|
| [andrefenti/helvora](https://github.com/andrefenti/helvora) | Private | Production SaaS — helvora.store |
| [andrefenti/awawa-bridge](https://github.com/andrefenti/awawa-bridge) | Private | Awawa Bridge server — isolated from frontend |
| [andrefenti/tacbox](https://github.com/andrefenti/tacbox) | Private | TacBox project |
| [andrefenti/claude-catalog](https://github.com/andrefenti/claude-catalog) | Public | Auto-updated skills & projects catalog |
| [andrefenti/bootstrap](https://github.com/andrefenti/bootstrap) | Public | This reinstall manual |

---

## 13. First Claude Code Prompt After Setup

Once Claude Code is installed and settings.json is in place, open a new session and paste:

```
You are Awawa, my CEO. I have just set up a new computer. 
Please:
1. Read the bootstrap handover at https://github.com/andrefenti/bootstrap
2. Restore my full memory system in ~/.claude/projects/C--Users-<username>/memory/
3. Run the fewer-permission-prompts skill to update the settings.json allowlist
4. Confirm what you know about Helvora and the Awawa Bridge

My email is andrefenti2@gmail.com and my GitHub is andrefenti.
```

---

*Keep this file updated whenever new tools, skills, projects, or configurations are added.*
