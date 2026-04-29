# AWP/Mineworks Discord Status Monitor — Technical Guide

> Version 2.0.0 | Deep Health Monitoring + Inline Explanations

## Overview

Monitor ini memantau kesehatan infrastruktur AWP dan Mineworks secara mendalam, lalu menampilkan hasilnya sebagai rich embed di Discord channel. Berbeda dengan versi sebelumnya yang hanya cek HTTP 200, v2.0.0 melakukan **deep health check** menggunakan JSON-RPC untuk mendeteksi masalah spesifik seperti database down, Redis failure, chain keeper stall, dan low gas balance.

## Arsitektur

```
┌──────────────────┐     ┌────────────────────┐
│   monitor.py     │────▶│   Discord API      │
│   (main loop)    │     │   (embed + ping)   │
├──────────────────┤     └────────────────────┘
│   probes.py      │
│   (deep health)  │
├──────────────────┤     ┌────────────────────┐
│  GatewayPresence │────▶│   Discord Gateway  │
│  (WebSocket)     │     │   (🟢 online)      │
└──────────────────┘     └────────────────────┘
         │
         ▼
┌────────────────────────────────────────┐
│  Data Sources:                         │
│  1. GET  api.awp.sh         (HTTP)     │
│  2. POST api.awp.sh/v2      (RPC)     │
│  3. GET  api.awp.sh/api/health (REST) │
│  4. GET  api.minework.net   (HTTP)     │
│  5. GET  minework.net/api/miners (REST)│
└────────────────────────────────────────┘
```

## File Structure

| File | Deskripsi |
|------|-----------|
| `monitor.py` | Main script — loop monitoring, Discord API, Gateway, state management |
| `probes.py` | Deep health probes, formatting helpers, issue detection, embed builders |
| `.env` | Konfigurasi (token, channel ID, URLs) |
| `.env.example` | Template konfigurasi |
| `.monitor_state.json` | Auto-generated state file untuk crash recovery |

## Data Sources & Yang Dipantau

### 1. Basic HTTP Probe (`probe_all()`)
- **URL:** `GET https://api.awp.sh` + `GET https://api.minework.net`
- **Fungsi:** Cek apakah server merespon HTTP (200/4xx = alive, 5xx/timeout = down)
- **Interval:** Setiap 2 menit

### 2. Deep Health — `health.detailed` (JSON-RPC)
- **URL:** `POST https://api.awp.sh/v2`
- **Method:** `health.detailed`
- **Data:** Database status, Redis status, per-chain indexer block, keeper alive, relayer balance
- **Issue detection:** DB down, Redis down, keeper stall, low gas

### 3. Protocol Stats — `stats.global` (JSON-RPC)
- **URL:** `POST https://api.awp.sh/v2`
- **Method:** `stats.global`
- **Data:** Total users, total staked, active subnets, chain count

### 4. Epoch Info — `/api/health` (REST)
- **URL:** `GET https://api.awp.sh/api/health`
- **Data:** Current epoch per chain, paused status, server version

### 5. Mineworks Miners — `/api/miners` (REST)
- **URL:** `GET https://minework.net/api/miners`
- **Data:** Total miners, online count, top reward, average score
- **Interval:** Setiap 5 cycle (~10 menit) — response besar (~10MB)

## Embed Layout (Opsi B — Inline Hints)

Setiap baris punya penjelasan singkat dalam kurung agar user awam mengerti:

```
🌐 API Health
├─ 📡 AWP API       → ✅ 189ms (backend utama)
├─ 📡 Mineworks API → ✅ 342ms (server mining)
├─ 🗄️ Database      → ✅ OK    (penyimpanan data)
└─ 📡 Redis         → ✅ OK    (cache/antrian)

⛓️ Blockchain (kondisi tiap chain)
├─ ETH  Blok #25.0M  Keeper ✅ OK Gas:0.000284 ETH
├─ BSC  Blok #95.0M  Keeper ✅ OK Gas:0.0122 BNB
├─ Base Blok #45.3M  Keeper ✅ OK Gas:0.0666 ETH
└─ ARB  Blok #456.9M Keeper ✅ OK Gas:0.00000152 ETH
  ↳ Keeper = pengirim TX on-chain
  ↳ Gas = saldo biaya transaksi

📊 AWP Network
├─ 👥 Total User  : 198,452
├─ 📦 Epoch       : 21 (periode reward)
├─ 💰 Total Stake : 38.8M AWP (dikunci)
└─ 🏗️ Subnet      : 8 aktif (area kerja miner)

⛏️ Mining Pool
├─ 🟢 Miner Online : 487 / 520
├─ 🏆 Top Reward   : 110,177 AWP
└─ 📊 Skor Rata²   : 80 (kualitas mining)

⏱️ Monitor
├─ 🚀 Mulai  : 29 Apr 2026 • 10:00 WIB
├─ 🔄 Cek    : 29 Apr 2026 • 10:16 WIB
└─ 📈 Uptime : 16 menit (8x check)
```

## Status Levels & Colors

| Status | Warna | Kondisi |
|--------|-------|---------|
| 🟢 ALL SYSTEMS OPERATIONAL | Hijau | Semua sehat, 0 issues |
| 🟡 PARTIAL DEGRADATION | Kuning | Ada issues (keeper down, low gas, redis down) tapi HTTP masih OK |
| 🟡 PARTIAL OUTAGE | Kuning | Salah satu API down, yang lain masih OK |
| 🔴 OFFLINE | Merah | Semua API tidak merespon |
| 🟢 RECOVERED ✅ | Hijau | Baru pulih dari downtime |

## Issue Detection

Monitor otomatis mendeteksi dan menampilkan masalah:

| Issue | Sumber Data | Threshold |
|-------|------------|-----------|
| Database DOWN | `health.detailed` → `database != "ok"` | Langsung alert |
| Redis DOWN | `health.detailed` → `redis != "ok"` | Langsung alert |
| Chain keeper mati | `health.detailed` → `keeperCacheAlive == false` | Langsung alert |
| Gas hampir habis | `health.detailed` → `relayerBalance` | < 0.001 token |
| Epoch paused | `/api/health` → `pausedUntil > 0` | Langsung alert |

## Environment Variables

| Variable | Default | Deskripsi |
|----------|---------|-----------|
| `DISCORD_BOT_TOKEN` | — | Bot token dari Discord Developer Portal |
| `DISCORD_CHANNEL_ID` | `1497830604056559707` | Channel untuk embed status |
| `ALERT_ROLE_ID` | `1211196221830471712` | Role yang di-ping saat recovery |
| `CHECK_INTERVAL` | `120` | Interval check dalam detik |
| `AWP_API_URL` | `https://api.awp.sh` | URL probe AWP |
| `MINEWORKS_API_URL` | `https://api.minework.net` | URL probe Mineworks |
| `AWP_RPC_URL` | `https://api.awp.sh/v2` | URL untuk JSON-RPC calls |
| `AWP_HEALTH_URL` | `https://api.awp.sh/api/health` | URL untuk epoch info |
| `MINEWORKS_MINERS_URL` | `https://minework.net/api/miners` | URL untuk miner stats |

## Deployment

### VPS (dengan tmux)
```bash
# Install dependency
pip install websocket-client --break-system-packages

# Jalankan dalam tmux
tmux new -s monitor
cd /path/to/discord-alert
python3 monitor.py

# Detach: Ctrl+B, D
# Reattach: tmux attach -t monitor
```

### Troubleshooting

| Masalah | Solusi |
|---------|--------|
| Bot offline di Discord | Pastikan `websocket-client` terinstall |
| Embed tidak muncul | Cek `DISCORD_BOT_TOKEN` dan `DISCORD_CHANNEL_ID` di `.env` |
| Deep health kosong | API mungkin down, monitor tetap jalan dengan basic probe |
| Miner stats kosong | Response besar, timeout bisa terjadi. Akan retry di cycle berikutnya |
| State rusak | Hapus `.monitor_state.json` dan restart |
