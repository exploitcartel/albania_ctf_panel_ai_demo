```
╔══════════════════════════════════════════════════════════════════════╗
║                                                                      ║
║        🦅  ALBANIA CTF PANEL  v3.0                                   ║
║        Attack & Defense Infrastructure · AI-Powered                  ║
║        © Edison Plaku — Built for War                                 ║
║                                                                      ║
╚══════════════════════════════════════════════════════════════════════╝
```

![Python](https://img.shields.io/badge/Python-3.11+-ff2d55?style=for-the-badge&logo=python&logoColor=white)
![FastAPI](https://img.shields.io/badge/FastAPI-0.118-00d4ff?style=for-the-badge&logo=fastapi&logoColor=white)
![Docker](https://img.shields.io/badge/Docker-Ready-00d4ff?style=for-the-badge&logo=docker&logoColor=white)
![AI](https://img.shields.io/badge/AI-Claude%20%7C%20Ollama-7f77dd?style=for-the-badge&logo=anthropic&logoColor=white)
![License](https://img.shields.io/badge/License-MIT-00ff88?style=for-the-badge)
![CTF](https://img.shields.io/badge/CTF-Attack%26Defense-ffd200?style=for-the-badge)

---

## 🎯 What is Attack & Defense CTF?

In **A&D competitions**, every team receives an identical vulnerable server (vulnbox).
Your objectives are two:

> `[1]` **ATTACK** — exploit other teams' services, steal flags, submit for points
> `[2]` **DEFEND** — patch your own services so attackers can't steal your flags

Flags rotate every **tick** (60–120 seconds). Speed is everything.
This panel centralizes everything needed to do both — fast.

---

## ⚡ Quick Start (No Docker)

```bash
# 1. Place all files in a folder
cd albania_ctf_panel

# 2. Run
bash run.sh

# 3. Open browser
http://localhost:9090

# Default credentials (change immediately)
admin / admin
```

---

## 🐳 Docker Deploy

### Local machine

```bash
bash install.sh local
```

Panel starts at `http://localhost:9090`.

### VPS / Remote server

```bash
bash install.sh vps root@YOUR_VPS_IP
```

Panel starts at `http://YOUR_VPS_IP:9090`.

### Docker commands

```bash
bash install.sh stop       # stop everything
bash install.sh restart    # restart containers
bash install.sh logs       # follow container logs
```

> Docker Compose is required. Install with:
> `apt install docker.io docker-compose-plugin -y`

---

## 🗂 Project Structure

```
albania_ctf_panel/
│
├── run.sh                  ← Start without Docker (use this for local dev)
├── install.sh              ← Docker deploy (local or VPS)
│
├── main.py                 ← FastAPI backend — all API routes + AI Exploit routes
├── auth.py                 ← JWT auth, users, audit log
├── config.py               ← Config & secrets I/O
├── tools.py                ← Tulip, S4DFarm, LDF, PCAP, Discovery
├── ai_exploits.py          ← AI Exploit engine (Tulip → AI → exploit → run)
│
├── index.html              ← Entire frontend (single file, no build step)
│
├── config/
│   ├── config.json         ← Competition settings
│   ├── secrets.json        ← Passwords, tokens, API keys (chmod 600)
│   ├── users.json          ← Panel users
│   └── audit.log           ← All actions logged with timestamp + IP
│
└── exploits/               ← Drop .py exploit scripts here
    └── <service>_ai.py     ← AI-generated exploits saved here automatically
```

---

## 🤖 AI Exploits — How It Works

The **AI Exploits** module is the most powerful feature of this panel.
It automates the entire exploit development cycle during competition — from traffic analysis to running exploits against all enemy teams simultaneously.

### The Full Flow

```
┌─────────────────────────────────────────────────────────────────┐
│                     AI EXPLOIT AGENT                            │
│                                                                 │
│  1. FETCH    →  Connects to Tulip, pulls flag-out flows         │
│                 (HTTP/TCP streams where enemy stole your flag)  │
│                                                                 │
│  2. ANALYZE  →  Summarizes flows into compact text              │
│                 (request/response pairs, attacker IP, service)  │
│                                                                 │
│  3. GENERATE →  Sends to Claude API or Ollama with expert       │
│                 CTF system prompt → AI returns Python exploit   │
│                                                                 │
│  4. SAVE     →  exploit saved to exploits/<service>_ai.py       │
│                                                                 │
│  5. DEPLOY   →  Script runs as subprocess against all 87 teams  │
│                 Flags captured live via stdout regex match      │
│                                                                 │
│  6. REPEAT   →  5 slots run simultaneously, each independent   │
└─────────────────────────────────────────────────────────────────┘
```

### Slot System

The AI Exploits tab has **5 independent slots**, each targeting a different service:

| Slot | Purpose |
|------|---------|
| Input: `service_name` + `port` | Identifies which Tulip flows to pull |
| `RUN` | Starts the AI agent (fetch → analyze → generate) |
| `PAUSE / RESUME` | Pauses the running exploit subprocess |
| `STOP` | Kills everything and resets the slot |
| `SEND EXPLOIT` | Deploys generated exploit against all teams |

All 5 slots can run **simultaneously** — one per vulnerable service.

### Tulip Integration

The engine tries two Tulip API strategies automatically:

| Strategy | Endpoint | Tulip version |
|----------|----------|---------------|
| Primary  | `GET /api/flows?service=X&type=flag_out` | Tulip ≥ 2.x |
| Fallback | `GET /api/links?service=X` | Older Tulip |

If Tulip has no flows yet (competition hasn't started), the AI generates a heuristic exploit based on service name and port alone.

### AI Providers

#### Claude API (Recommended)

Uses `claude-sonnet-4-20250514` — best code generation quality.

1. Get API key at [console.anthropic.com](https://console.anthropic.com)
2. In panel → **AI EXPLOITS** → **⚙ API CONFIG** → paste key → **SAVE API KEY**
3. Or add directly to `config/secrets.json`:

```json
{
  "claude_api_key": "sk-ant-api03-..."
}
```

Cost: approximately **$0.01–0.05 per exploit** generated.

#### Ollama (Free / Local)

Runs completely offline on your own machine. No API key, no cost, no internet required during competition.

**Setup:**

```bash
# 1. Install Ollama
curl -fsSL https://ollama.ai/install.sh | sh

# 2. Start the server
ollama serve

# 3. Pull a model (do this before competition — needs internet)
ollama pull codellama       # Best for exploit generation (~3.8 GB)
ollama pull deepseek-coder  # Alternative, strong reasoning (~3.8 GB)
ollama pull mistral         # Lighter option (~4.1 GB)

# 4. Verify it works
ollama run codellama "write a python script to make an HTTP GET request"
```

In the panel → **⚙ API CONFIG** → **TEST CONNECTION** to confirm Ollama is reachable.

> Ollama quality is lower than Claude for complex exploit generation, but works offline and is free.

### Audit Log

All AI actions are logged automatically:

```
2026-05-11 14:02:01 | admin  | AI_EXPLOIT_RUN    | slot:0 svc:web_challenge provider:claude
2026-05-11 14:02:45 | admin  | AI_EXPLOIT_SEND   | slot:0
2026-05-11 14:04:12 | admin  | AI_EXPLOIT_STOP   | slot:0
2026-05-11 14:05:00 | admin  | AI_CONFIG_UPDATED | fields:claude_api_key
```

---

## 🧩 All Modules

### Dashboard
Live EKG health monitor, SLA event log, flag stats, tools status at a glance.

```
GREEN waveform  → All services UP
YELLOW          → Services degraded — investigate
RED flatline    → ALL DOWN — panic mode
```

### Setup Wizard
First thing after login. Configure vulnbox IP, gameserver, team token in 2 minutes.

### Vulnbox
Remote management via SSH directly from browser:

| Feature | What it does |
|---------|-------------|
| Backup | Snapshots `/root/`, `/home/`, `/opt/` before patching |
| Watchdog | Auto-restarts crashed Docker services every 25s |
| Firewall | Blocks other teams' IPs, keeps gameserver open |
| Change Pass | Rotates root password, saves to secrets automatically |

### Tools
- **Tulip** — captures and indexes all traffic; replay streams to understand attacks
- **S4DFarm** — runs exploits against all teams in parallel, submits flags automatically
- **PCAP Sync** — continuously rsyncs `.pcap` files from vulnbox to local machine

### LDF Farm
Controlled flag submission rate (default: 5 flags/second). Prevents gameserver rate-limiting that causes flag submission blocks.

### Discovery / Radar
Ping sweep of all team IPs. Animated radar visualization. Click any host to copy IP.

### Exploits
Drop Python scripts in `exploits/`, run/stop from the panel. Per-exploit logs and running status.

### SSH Console
Direct SSH command execution on vulnbox from browser. Quick buttons for `docker ps`, disk, memory, ports.

---

## 🏁 Competition Day Checklist

```
□ bash run.sh  →  http://localhost:9090
□ Login  →  change password immediately
□ Setup Wizard  →  vulnbox IP, gameserver, team token
□ Vulnbox  →  Backup (snapshot before anything)
□ Vulnbox  →  Watchdog ON (auto-restart services)
□ Tools  →  Start Tulip
□ Tools  →  Start S4DFarm
□ Tools  →  Start PCAP Sync
□ LDF Farm  →  Start LDF
□ Discovery  →  Ping sweep all teams
□ Toolkit  →  Nmap your own vulnbox (know your services)
□ AI EXPLOITS  →  Configure API key or Ollama
□ Dashboard  →  Monitor EKG throughout competition
```

---

## 🔐 Security

| Item | Detail |
|------|--------|
| `secrets.json` | `chmod 600` automatically — never commit to git |
| Default password | Must be changed on first login |
| Token TTL | 24 hours — auto-expire |
| Audit log | All actions: who, what, when, from which IP |
| Viewer role | Read-only — safe to give to teammates |

---

## 🚀 VPS Deploy

```bash
# One command deploy to remote server
bash install.sh vps root@YOUR_VPS_IP

# Panel available at:
http://YOUR_VPS_IP:9090
```

---

## ⚠️ Things to Remove or Reconsider

A few observations on the codebase worth addressing before a high-stakes competition:

**1. `ai_exploits_routes.py` is now redundant.**
The routes were integrated directly into `main.py`. The standalone file was for manual patching and is no longer needed — it can be deleted to avoid confusion.

**2. `run.sh` runs as a single Python process.**
If `main.py` crashes, the panel goes down. For competition use, wrap it in a simple restart loop:
```bash
while true; do python -m uvicorn backend.main:app --host 0.0.0.0 --port 9090; sleep 2; done
```

**3. The `lf` file handle bug in `_runner_thread` was dead code.**
Fixed in v3.0 — `stderr` from exploit subprocesses now properly redirects to the log file instead of being silently discarded.

**4. `secrets.json` default had missing fields.**
Old installs (before v3.0) have `secrets.json` without `claude_api_key` and `ollama_url`. Run this once to patch:
```bash
python3 -c "
import json; f='config/secrets.json'
d=json.load(open(f))
d.setdefault('claude_api_key','')
d.setdefault('ollama_url','http://localhost:11434')
json.dump(d,open(f,'w'),indent=2)
print('Patched OK')
"
```

**5. No HTTPS.**
The panel runs plain HTTP. On a VPS exposed to the internet, add nginx + certbot in front. On a local competition network this is usually acceptable.

**6. Ollama quality vs Claude.**
For complex services (binary exploitation, deserialization, custom protocols), Claude produces significantly better exploits than local Ollama models. Use Ollama as a fallback when internet is unavailable, not as a primary.

---

## 📄 License

```
MIT License

Copyright (c) 2026 Edison Plaku

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in
all copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN
THE SOFTWARE.
```

---

<div align="center">

```
Built by Edison Plaku for the Albanian National Team
Attack & Defense — Play to Win  🦅🇦🇱
CyberChallenge.IT 2026
```

</div>
