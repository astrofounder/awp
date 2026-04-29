# AWP/Mineworks Discord Status Monitor

Monitor status API AWP & Mineworks, kirim alert ke Discord channel dengan rich embed + deep health check.

## Fitur

- 🔍 **Deep Health Check** — Cek database, Redis, keeper, gas balance per-chain
- 📊 **Protocol Stats** — Users, epoch, staking, subnets real-time
- ⛏️ **Mining Pool Stats** — Online miners, top reward, average score
- 🔴 **DOWN Alert** — Kirim embed merah saat API mati
- ✏️ **Edit-in-Place** — Selama masih down, edit pesan yang sama (update timestamp & durasi)
- 🟢 **Recovery Alert** — Kirim pesan BARU + role ping saat API kembali online
- 🟡 **Partial Alert** — Deteksi degradasi (keeper down, low gas) meski HTTP masih 200
- 💾 **Crash Recovery** — State disimpan ke file, auto-resume setelah restart
- 🕐 **WIB Timezone** — Semua waktu dalam format WIB (UTC+7)

## Setup

### 1. Buat Discord Bot

1. Buka [Discord Developer Portal](https://discord.com/developers/applications)
2. Klik **New Application** → beri nama → **Create**
3. Klik tab **Bot** → **Reset Token** → Copy token
4. Aktifkan **MESSAGE CONTENT INTENT** (di bagian Privileged Gateway Intents)
5. Klik tab **OAuth2** → **URL Generator**:
   - Scopes: `bot`
   - Bot Permissions: `Send Messages`, `Embed Links`, `Read Message History`
6. Copy URL → buka di browser → pilih server → Authorize

### 2. Konfigurasi

```bash
copy .env.example .env
```

Edit `.env`:
```
DISCORD_BOT_TOKEN=MTQ5NzgzMDYwNDA1NjU1OTcw...
DISCORD_CHANNEL_ID=1497830604056559707
CHECK_INTERVAL=120
```

### 3. Install & Jalankan

```bash
pip install websocket-client
python monitor.py
```

## Output Discord

### All Systems Online (Hijau)
```
🟢 AWP STATUS: ALL SYSTEMS OPERATIONAL

🌐 API Health
├─ 📡 AWP API       → ✅ 189ms (backend utama)
├─ 📡 Mineworks API → ✅ 342ms (server mining)
├─ 🗄️ Database      → ✅ OK    (penyimpanan data)
└─ 📡 Redis         → ✅ OK    (cache/antrian)

⛓️ Blockchain (kondisi tiap chain)
├─ ETH  Blok #25.0M  Keeper ✅ OK Gas:0.28 ETH
├─ BSC  Blok #95.0M  Keeper ✅ OK Gas:12.2 BNB
├─ Base Blok #45.3M  Keeper ✅ OK Gas:0.067 ETH
└─ ARB  Blok #456.9M Keeper ✅ OK Gas:0.001 ETH
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
📖 [Penjelasan tiap field](https://github.com/astrofounder/awp/blob/main/discord-alert/MONITOR.md)

### Partial Degradation (Kuning)
```
🟡 AWP STATUS: PARTIAL DEGRADATION

⚠️ Masalah Terdeteksi
├─ 💰 ETH gas hampir habis (0.000284 ETH)
└─ 💰 ARB gas hampir habis (0.00000152 ETH)
```

### Full Offline (Merah)
```
🔴 AWP STATUS: OFFLINE

⚠️ Masalah Terdeteksi
├─ 📡 AWP API DOWN — Connection refused
└─ 📡 Mineworks API DOWN — timeout
```

> **Note:** Di Discord, baris `📖 Penjelasan tiap field` adalah link clickable yang mengarah ke [MONITOR.md](MONITOR.md).

---

## Arsitektur

```
┌──────────────────┐     ┌─────────────────┐
│   monitor.py     │────▶│  Discord API     │
│   (main loop)    │     │  (embed + ping)  │
├──────────────────┤     └─────────────────┘
│   probes.py      │
│   (deep health)  │
├──────────────────┤     ┌─────────────────┐
│  GatewayPresence │────▶│  Discord Gateway │
│  (WebSocket)     │     │  (🟢 online)     │
└──────────────────┘     └─────────────────┘
         │
         ▼
  Data Sources:
  1. GET  api.awp.sh              (HTTP ping)
  2. POST api.awp.sh/v2           (JSON-RPC)
  3. GET  api.awp.sh/api/health   (REST)
  4. GET  api.minework.net        (HTTP ping)
  5. GET  minework.net/api/miners (REST)
```

### File Structure

| File | Deskripsi |
|------|-----------|
| `monitor.py` | Main script — loop monitoring, Discord API, Gateway, state management |
| `probes.py` | Deep health probes, formatting helpers, issue detection, embed section builders |
| `MONITOR.md` | Panduan user — arti setiap field di output Discord |
| `.env` | Konfigurasi (token, channel ID, URLs) |
| `.monitor_state.json` | Auto-generated state file untuk crash recovery |

---

## Environment Variables

| Variable | Default | Deskripsi |
|----------|---------|-----------|
| `DISCORD_BOT_TOKEN` | — | Bot token dari Discord Developer Portal |
| `DISCORD_CHANNEL_ID` | `1497830604056559707` | Channel target untuk embed status |
| `ALERT_ROLE_ID` | `1211196221830471712` | Role yang di-ping saat recovery |
| `CHECK_INTERVAL` | `120` | Interval check dalam detik |
| `AWP_RPC_URL` | `https://api.awp.sh/v2` | JSON-RPC endpoint |
| `AWP_HEALTH_URL` | `https://api.awp.sh/api/health` | Epoch & version info |
| `MINEWORKS_MINERS_URL` | `https://minework.net/api/miners` | Miner stats |
| `DOCS_URL` | GitHub MONITOR.md | Link docs di embed |

---

## Deployment (VPS + tmux)

```bash
pip install websocket-client --break-system-packages
tmux new -s monitor
cd /path/to/discord-alert
python3 monitor.py
# Detach: Ctrl+B, D | Reattach: tmux attach -t monitor
```

### Troubleshooting

| Masalah | Solusi |
|---------|--------|
| Bot offline di Discord | Pastikan `websocket-client` terinstall |
| Embed tidak muncul | Cek `DISCORD_BOT_TOKEN` dan `DISCORD_CHANNEL_ID` di `.env` |
| Deep health kosong | API mungkin down, monitor tetap jalan dengan basic probe |
| State rusak | Hapus `.monitor_state.json` dan restart |
