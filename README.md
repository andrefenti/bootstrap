# Bootstrap — Claude Code Reinstall Manual

> Full reinstall guide for **andrefenti**. If you lose access to your computer and have only Claude Code, follow this guide top to bottom.

---

## 0. Prerequisites

- Windows 11 (or compatible OS)
- Claude Code installed and authenticated
- GitHub account: `andrefenti`
- Anthropic API key

---

## 1. Authorize Claude Code Permissions

Open Claude Code and run this prompt verbatim:

```
claude config set permissions.allow '["Bash(vercel:*)","Bash(gh:*)","Bash(git:*)","Bash","Read","Edit","Write","WebFetch","Grep","Glob","NotebookEdit","WebSearch"]'
```

Or paste into Claude Code chat:
```
Set Claude Code permissions.allow to: Bash(vercel:*), Bash(gh:*), Bash(git:*), Bash, Read, Edit, Write, WebFetch, Grep, Glob, NotebookEdit, WebSearch — no confirmation needed
```

---

## 2. Install Required Tools

### Git
```bash
# Verify or install from https://git-scm.com
git --version
```

### Bun (JavaScript runtime / package manager)
```powershell
powershell -c "irm bun.sh/install.ps1 | iex"
```

### GitHub CLI
```bash
# Download from https://cli.github.com or via winget:
winget install GitHub.cli
```

### Authenticate GitHub CLI
```bash
gh auth login
# Choose: GitHub.com → HTTPS → authenticate via browser
# Then request delete_repo scope:
gh auth refresh -h github.com -s delete_repo
```

### Vercel CLI
```bash
npm install -g vercel
vercel login
```

### Python (for hooks)
- Download Python 3.x from https://python.org
- Install to `C:\Python314\` (or update hook commands in settings.json to match actual path)

### Node.js / npm
```bash
# Download from https://nodejs.org (LTS version)
node --version
npm --version
```

---

## 3. Clone Active Projects

### Helvora (production SaaS at helvora.store)
```bash
git clone https://github.com/andrefenti/helvora D:/Downloads/Helvora
cd D:/Downloads/Helvora
bun install
```

Set up environment variables (copy from Supabase + Stripe dashboard):
```bash
# Create .env.local with:
VITE_SUPABASE_URL=https://ayypmxiesqjihkxtrkjn.supabase.co
VITE_SUPABASE_ANON_KEY=<from Supabase dashboard → Settings → API>
```

Deploy to Vercel production:
```bash
cd D:/Downloads/Helvora
vercel link --yes
vercel env pull .env.local --yes
vercel --prod --yes
```

> Domain: helvora.store → DNS A record `76.76.21.21` at GoDaddy

### tacbox (private project)
```bash
git clone https://github.com/andrefenti/tacbox <local-path>
```

---

## 4. Install Claude Code Skills

Skills live at `~/.claude/skills/`. Each skill is a folder with a `SKILL.md`.

### Install all skills via Claude Code chat (paste this prompt):
```
Install these skills from skills.sh:
- /create-skill
- /catalog
- /startup
- /github-sync
- /vercel-deploy
- /deployment-vercel
- /arquiteto-supabase
- /arquiteto-postgresql
- /otimizador-postgresql
- /arquiteto-stripe
- /gestor-subscricoes-stripe
- /gestor-webhooks-stripe
- /arquiteto-oauth
- /integrador-openapi
- /arquiteto-openai
- /frontend-design
- /designer-ux-acessibilidade
- /design-system-figma
- /design-for-ai
- /security
- /pentest-advisor
- /sqli-test
- /xss-test
- /api-keys
- /wordlist
- /webshell-detect
- /bug-bounty-hunter
- /ctf-assistant
- /auditor-seguranca
- /defesa-prompt-injection
- /gestor-segredos
- /self-healing
- /researcher
- /customer-support
- /know-me
- /scalability
- /cost-reducer
- /revisor-codigo
- /observabilidade-incidentes
- /motor-conteudo
- /otimizador-tokens
- /scraping-etico
- /integrador-cripto
- /integrador-lovable
- /arquiteto-obsidian
- /game-development
- /pipeline-3d-assets
- /n8n
- /trigger-dev
- /operador-godaddy-dns
- /gestor-subscricoes-stripe
```

### Install Shannon (AI pentester):
```
Add this skill: https://github.com/KeygraphHQ/shannon
```

### Install dev-browser (browser automation):
```
Add this skill: https://skills.sh/sawyerhood/dev-browser/dev-browser
```

---

## 5. Set Up Claude Catalog (auto-sync)

```bash
git clone https://github.com/andrefenti/claude-catalog ~/claude-catalog
```

Verify `~/claude-catalog/sync.sh` exists. If not, restore it from the repo or ask Claude to recreate it.

The catalog auto-sync hook in `~/.claude/settings.json` should look like:
```json
{
  "matcher": "Write|Edit",
  "hooks": [{
    "type": "command",
    "command": "FILE=$(/c/Python314/python -c \"import sys,json; d=json.load(sys.stdin); print(d.get('tool_input',{}).get('file_path',''))\" 2>/dev/null); case \"$FILE\" in *\".claude/skills\"*) cd \"$HOME/claude-catalog\" && bash sync.sh 2>/dev/null || true;; esac",
    "timeout": 30,
    "async": true
  }]
}
```

> If Python path is different, update `/c/Python314/python` to match actual install path.

---

## 6. Configure Memory System

Memory files live at `~/.claude/projects/C--WINDOWS-System32/memory/`. Ask Claude:
```
Restore my memory system from the bootstrap repo and the claude-catalog README
```

---

## 7. Restore Vercel + GoDaddy (for helvora.store)

1. Link Helvora to Vercel: `cd D:/Downloads/Helvora && vercel link --yes`
2. Confirm domain in Vercel dashboard: `helvora.store`
3. At GoDaddy, set DNS A record for `@` → `76.76.21.21`

---

## 8. Key Accounts & Services

| Service | Account | Notes |
|---------|---------|-------|
| GitHub | andrefenti | Token via `gh auth login` |
| Supabase | Helvora project ID: `ayypmxiesqjihkxtrkjn` | Get keys from dashboard |
| Stripe | — | Publishable key hardcoded in `EmbeddedCheckout.tsx` |
| Vercel | andrefenti / andre-fenti-yamamoto | CLI: `vercel login` |
| GoDaddy | — | DNS for helvora.store, nameservers: ns71/72.domaincontrol.com |
| WAPI | — | Logistics integration in Helvora |
| Resend | — | Email service in Helvora |

---

## 9. Useful Claude Code Prompts

```
# Start a work session
/startup

# Sync catalog to GitHub
/catalog

# Deploy helvora to production
/vercel-deploy

# Pentest helvora
/shannon --target https://helvora.store --source D:/Downloads/Helvora

# Automate browser
/dev-browser
```

---

## Projects

| Repo | Visibility | Description |
|------|------------|-------------|
| [andrefenti/helvora](https://github.com/andrefenti/helvora) | Private | Production SaaS — helvora.store |
| [andrefenti/tacbox](https://github.com/andrefenti/tacbox) | Private | TacBox project |
| [andrefenti/claude-catalog](https://github.com/andrefenti/claude-catalog) | Public | Auto-updated skills & projects catalog |
| [andrefenti/bootstrap](https://github.com/andrefenti/bootstrap) | Public | This reinstall manual |

---

*Keep this file updated whenever new tools, skills, projects, or configurations are added.*
