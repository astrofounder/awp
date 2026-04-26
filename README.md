# ⛏️ AWP Mine Bot v3.1.0

> Autonomous multi-wallet mining bot for [AWP (Autonomous Work Protocol)](https://awp.network) with smart LLM enrichment, self-healing workers, anti-sybil proxy routing, and real-time Telegram monitoring.

---

## ✨ Features

### Core Mining
- **Multi-wallet support** — mine with unlimited wallets simultaneously
- **Background workers** — each wallet runs in its own thread
- **Dead thread auto-restart** — crashed workers recover automatically
- **run-worker daemon fallback** — bypasses platform API when `agent-start` times out
- **Duplicate daemon cleanup** — `pkill` prevents process accumulation on restarts
- **Exponential backoff** — smart error recovery with jitter
- **Graceful shutdown** — `SIGTERM`/`Ctrl+C` stops cleanly
- **Crash recovery** — resumable state on restart

### LLM Enrichment (v3.1.0)
- **Smart gateway detection** — auto-finds CLIProxyAPI on `localhost:8317`
- **Multi-API-key rotation** — comma-separated keys, auto-rotate on quota hit
- **Cross-provider fallback** — `gemini-3-flash` → `gemini-3.1-pro-high` → `gemini-3.1-pro-low`
- **Preflight health check** — validates gateway + model before mining starts
- **Auto-detect model** — queries `/v1/models` and picks the best available

### Anti-Sybil & Proxy
- **Smart proxy routing** — mining traffic through residential proxy, registration direct
- **`NO_PROXY` bypass** — `api.awp.sh` (registration) and LLM gateway go direct
- **Platform via proxy** — `api.minework.net` MUST go through proxy (datacenter IPs blocked)
- **Relay-400 patch** — auto-patches AWP skill to handle "already registered" errors gracefully
- **Per-wallet IP diversity** — rotating residential proxy with session-based assignment

### Monitoring
- **Telegram notifications** — startup, shutdown, errors, milestones
- **Hourly reports** — periodic summary of all wallet stats
- **Credit milestones** — alerts when credit score increases
- **Key rotation alerts** — notified when API key quota exhausted
- **Live dashboard** — real-time terminal monitoring (`--dashboard`)

### Developer Experience
- **Interactive setup wizard** — `--setup` generates `.env` in 5 steps
- **CLIProxyAPI auto-install** — offered when gateway not detected
- **Dry-run mode** — validate setup without starting mining
- **File logging** — `awp_mining.log` with rotation

---

## 🚀 Quick Start

### 1. First-Time Setup (Interactive)

```bash
python3 awpv3.py --setup
```

The wizard walks you through:
1. **LLM Gateway** — detect/install CLIProxyAPI, select model
2. **Wallet** — single mnemonic or multi-wallet file
3. **Registration** — funder private key (or gasless relay)
4. **Telegram** — bot token + chat ID for notifications
5. **Proxy** — optional rotating proxy configuration

### 2. Manual Setup

```bash
cp .env.example .env
nano .env          # Edit your configuration
```

### 3. Start Mining

```bash
# Normal run
python3 awpv3.py

# Test setup first (no mining)
python3 awpv3.py --dry-run

# Skip dependency installation
python3 awpv3.py --skip-deps

# Override wallet count
python3 awpv3.py --wallet-count 5
```

---

## ⚙️ Configuration

### Environment Variables (`.env`)

| Variable | Required | Default | Description |
|----------|:--------:|---------|-------------|
| `MNEMONIC` | ✅ | — | Wallet mnemonic (12/24 words) |
| `MINE_GATEWAY_BASE_URL` | — | auto-detect | CLIProxyAPI URL (e.g. `http://localhost:8317/v1`) |
| `MINE_GATEWAY_TOKEN` | — | — | API key(s), comma-separated for multi-key |
| `MINE_ENRICH_MODEL` | — | auto-detect | LLM model (e.g. `gemini-3-flash`) |
| `BASE_RPC_URL` | — | `https://mainnet.base.org` | Base Mainnet RPC (Alchemy recommended) |
| `FUNDER_PRIVATE_KEY` | — | — | ETH private key for on-chain registration |
| `TG_BOT_TOKEN` | — | — | Telegram bot token from @BotFather |
| `TG_CHAT_ID` | — | — | Telegram chat ID |
| `NODE_NAME` | — | — | Multi-VPS node identifier |

<details>
<summary><b>All Variables</b></summary>

| Variable | Default | Description |
|----------|---------|-------------|
| `WALLET_MODE` | `single` | `single` or `multi` |
| `WALLET_COUNT` | `1` | Number of wallets to mine |
| `WALLETS_FILE` | `wallets.txt` | Path to multi-wallet file |
| `USE_PROXY` | `false` | Enable rotating proxy |
| `PROXY_HOST` | — | Proxy hostname |
| `PROXY_PORT` | — | Proxy port |
| `PROXY_USER_PREFIX` | — | Proxy username prefix |
| `PROXY_PASS` | — | Proxy password |
| `BASE_RPC_URL` | `https://mainnet.base.org` | Base Mainnet RPC endpoint |

</details>

### Multi-Key Rotation

Support multiple API keys for quota rotation:

```env
MINE_GATEWAY_TOKEN=sk-KEY1,sk-KEY2,sk-KEY3
```

When one key hits rate limits, the bot automatically rotates to the next key with a 5-minute cooldown.

### Model Fallback Chain

When a model is unavailable, the bot falls back through this chain:

```
gemini-3-flash → gemini-3.1-pro-high → gemini-3.1-pro-low
```

Google and Claude models use **separate quota pools**, so rotating between them effectively doubles your available quota.

### RPC Configuration

The bot uses a single Base Mainnet RPC for on-chain registration checks. Set in `.env`:

```env
# Private Alchemy RPC (recommended)
BASE_RPC_URL=https://base-mainnet.g.alchemy.com/v2/YOUR_KEY

# Or public fallback (rate-limited)
BASE_RPC_URL=https://mainnet.base.org
```

If `BASE_RPC_URL` is not set, falls back to the public `mainnet.base.org` endpoint.

---

## 📡 CLI Commands

| Command | Description |
|---------|-------------|
| `python3 awpv3.py` | Start mining |
| `python3 awpv3.py --setup` | Interactive setup wizard |
| `python3 awpv3.py --dashboard` | Live monitoring dashboard |
| `python3 awpv3.py --dry-run` | Test setup without mining |
| `python3 awpv3.py --get-chat-id` | Find Telegram chat ID |
| `python3 awpv3.py --test-telegram` | Send test notification |
| `python3 awpv3.py --test-proxy` | Test proxy connectivity |
| `python3 awpv3.py --skip-deps` | Skip dependency install |
| `python3 awpv3.py --skip-onchain` | Skip on-chain registration |
| `python3 awpv3.py --no-proxy` | Disable proxy |
| `python3 awpv3.py --wallet-count N` | Override wallet count |
| `python3 awpv3.py --batch-size N` | Override batch size |
| `python3 awpv3.py --proxy-file PATH` | Override proxy file path |
| `python3 awpv3.py --wallets-file PATH` | Override wallets file path |
| `python3 awpv3.py --monitor-interval N` | Override monitor interval (seconds) |
| `python3 awpv3.py --verbose` | Debug logging |

---

## 📊 Live Dashboard

```bash
python3 awpv3.py --dashboard
```

```
============================================================
  AWP v3.1.0 Dashboard | 2026-04-26 12:30:00 | Cycle #12
============================================================

  ⚡ LLM Gateway: ✓ OK (7 models)
  📡 URL: http://localhost:8317/v1
  🤖 Model: gemini-3-flash
  🔑 Key: #1/3 | Rotations: 2
  ⏱  Uptime: 3h 45m

  ────────────────────────────────────────────────────────
  Wallet           State       Epoch  Submit  Credit
  ────────────────────────────────────────────────────────
  wallet1          ✓ running   42/80      156      12
  wallet2          ✓ running   38/80      134       9
  wallet3          ○ idle       0/80        0       0

  ────────────────────────────────────────────────────────
  Refreshing in 30s... (Ctrl+C to exit)
```

---

## 📱 Telegram Notifications

| Emoji | Event | When |
|:-----:|-------|------|
| 🚀 | **Bot Started** | Script starts with wallet/gateway summary |
| 🛑 | **Shutting Down** | Ctrl+C or SIGTERM received |
| ⚠️ | **Worker Restart** | Dead thread detected & auto-restarted |
| ❌ | **All Failed** | Every wallet failed setup |
| 📊 | **Hourly Report** | Periodic summary (credits, submissions) |
| 💰 | **Milestone** | Credit score increased |
| 🔄 | **Key Rotated** | API key quota exhausted, switched |
| ⚡ | **Gateway Warning** | LLM gateway test failed |

---

## 🔧 LLM Gateway Setup

The bot uses [CLIProxyAPI](https://github.com/brokechubb/cliproxyapi-installer) to access LLM models for free via OAuth.

### Install CLIProxyAPI

```bash
curl -fsSL https://raw.githubusercontent.com/brokechubb/cliproxyapi-installer/refs/heads/master/cliproxyapi-installer | bash
cd ~/cliproxyapi && ./cli-proxy-api --login
```

### Verify

```bash
curl -s http://localhost:8317/v1/models | python3 -m json.tool
```

### Management Panel

Access at `http://YOUR_VPS_IP:8317/management.html` to:
- View/add API keys
- Configure quota fallback
- Monitor usage statistics

---

## 🛡️ Network & Proxy Architecture

The bot uses a smart proxy routing strategy for anti-sybil compliance:

```
┌─────────────────┐     ┌──────────────────────┐
│  awpv3.py       │     │  Residential Proxy    │
│                 │     │  (rotating sessions)  │
│  ┌───────────┐  │     └────────┬─────────────┘
│  │ wallet1   │──┼──────────────┤
│  │ wallet2   │──┼──────────────┤  → api.minework.net (mining)
│  │ wallet3   │──┼──────────────┤  → wikipedia, arxiv, etc (crawling)
│  └───────────┘  │              │
│                 │     ┌────────┴─────────────┐
│  NO_PROXY ──────┼─────┤  Direct (bypass)     │
│                 │     │  → api.awp.sh        │  registration/relay
│                 │     │  → localhost:8317     │  LLM gateway
│                 │     └──────────────────────┘
└─────────────────┘
```

| Destination | Route | Reason |
|-------------|-------|--------|
| `api.minework.net` | **Proxy** | Platform blocks datacenter IPs |
| `wikipedia.org`, `arxiv.org`, etc | **Proxy** | Crawling targets — anti-sybil |
| `api.awp.sh` | **Direct** | Registration/relay — fast, no anti-sybil |
| LLM gateway | **Direct** | Local service, no proxy needed |

---

## 📁 Project Structure

```
awp/
├── awpv3.py             # Main bot script
├── .env                 # Configuration (secrets)
├── wallets.txt          # Multi-wallet mnemonics (optional)
├── proxies.txt          # Proxy list (optional)
├── awp_mining.log       # Runtime logs (auto-created)
├── mine-skill-template/ # Shared mine-skill template (auto-cloned)
└── mine-skill-wallet*/  # Per-wallet working directories (auto-created)
    ├── .venv/           # Python virtual environment
    ├── scripts/         # AWP mine-skill scripts
    ├── worker.log       # run-worker daemon log (fallback mode)
    └── ...
```

---

## 🛡️ Security

- Mnemonics hidden from `ps aux` (not passed as CLI args)
- `.env` file for secrets (never committed to git)
- RPC API keys in `.env` (not hardcoded in source)
- `shell=False` for subprocess calls with user input
- Per-wallet isolation (separate working directories)
- Proxy bypass for local gateway traffic (`NO_PROXY`)

---

## 📋 Requirements

- **Python 3.11+** (auto-installed if missing)
- **Node.js 18+** (auto-installed if missing)
- **Linux VPS** recommended (tested on Ubuntu 22.04+)
- **CLIProxyAPI** for free LLM enrichment (optional but recommended)

---

## ⚡ Startup Flow

```
python3 awpv3.py
    │
    ├── Load .env
    ├── Install Python 3.11 + Node.js (if needed)
    ├── ⚡ Preflight Gateway Check
    │   ├── Auto-detect CLIProxyAPI
    │   ├── Query available models
    │   ├── Select best model
    │   └── Test chat completion
    ├── On-chain registration (if FUNDER_PRIVATE_KEY set)
    │   └── Uses BASE_RPC_URL (Alchemy) for Base Mainnet
    ├── Setup wallets (clone mine-skill, patch relay-400, unlock, doctor)
    ├── 🚀 Telegram: "Bot Started"
    ├── Start mining threads
    │   ├── Try agent-start (3 attempts)
    │   └── Fallback: spawn run-worker daemon (if platform unresponsive)
    └── Monitor loop
        ├── Dead thread auto-restart
        ├── Duplicate daemon cleanup (pkill)
        ├── 📊 Hourly Telegram report
        └── 💰 Credit milestone alerts
```

---

## 📝 Changelog

### v3.1.0 — Smart Gateway, Anti-Sybil & Resilient Workers
- LLM Gateway preflight health check
- Auto-detect model from CLIProxyAPI `/v1/models`
- Multi-API-key rotation (`GatewayKeyManager`)
- Cross-provider model fallback (Google → Claude)
- Enhanced Telegram (hourly reports, milestones, key rotation alerts)
- Smart `.env` defaults (auto-detect gateway URL & model)
- Interactive setup wizard (`--setup`)
- Live dashboard (`--dashboard`)
- CLIProxyAPI auto-install
- **Anti-sybil proxy routing** — smart `NO_PROXY` bypass for registration
- **Relay-400 patch** — auto-patches AWP skill for "already registered" handling
- **run-worker daemon fallback** — bypasses platform timeout with direct worker spawn
- **Duplicate daemon cleanup** — `pkill` prevents process accumulation
- **RPC in `.env`** — `BASE_RPC_URL` replaces hardcoded multi-RPC list

### v3.0.0 — Production Resilience
- `.env` file support
- CLI arguments
- File logging with rotation
- Telegram notifications
- Earnings tracking
- Dead thread auto-restart
- Exponential backoff
- Graceful shutdown
- Crash recovery
- Security hardening

---

## 📄 License

Private use only. Not for redistribution.
