# 📖 Penjelasan Output Discord Monitor

File ini menjelaskan **arti setiap baris** yang muncul di embed Discord.
Dibaca oleh member yang melihat output monitor dan bertanya "ini maksudnya apa?"

---

## Contoh Output Lengkap

```
🌐 Server
├─ AWP API ✅ 312ms
└─ Mineworks API ✅ 189ms

⛏️ Mining Hari Ini
├─ 📅 Epoch  : 2026-04-28 (🟢 OPEN)
│  └ Kemarin: 3.6M sub → 44,089 rewarded
├─ 📊 Live   : 15,230 sub | 4,100 eval
├─ 👷 Miner  : 13,352 online
├─ 🏅 Top    : 142,000 aMINE (miner terbaik)
└─ 📈 Avg Score: 78/100

📦 Dataset Ranking (total submissions)
├─ 🥇 Wikipedia              4.6M ×1
├─ 🥈 arXiv                  3.4M ×1
├─ 🥉 LinkedIn Profiles      397K ×12
├─    Amazon Reviews          47K ×8
├─    Basic Amazon Products    33K ×8
└─    LinkedIn Jobs            4K ×5
  ↳ ×N = reward multiplier (makin besar = cuan)

💰 Info Reward & Kualifikasi
├─ 💎 Reward Split: 41% miner | 41% validator | 18% owner
├─ ⏰ Settlement  : 00:00 UTC (harian)
├─ 📋 Min. Syarat: ≥80 tasks + avg score ≥60
├─ ⚠️ LLM Enrich : WAJIB! Tanpa = max score ~55
└─ 🎯 Credit Tier: novice→100% PoW | good→5% | excel→1%

⚠️ Masalah Terdeteksi
├─ 📡 AWP API: DOWN — Connection refused
└─ ⛓️ Base: keeper cache mati
```

---

## 🌐 Server — Penjelasan Per Baris

```
├─ AWP API ✅ 312ms
```
- **AWP API** = server backend utama (`awp.pro`). Mengurus registrasi, staking, reward.
- **✅** = server merespons HTTP 200. **❌** = mati / tidak bisa dihubungi.
- **312ms** = waktu respons. Di bawah 500ms = bagus. Di atas 2000ms = lambat.

```
└─ Mineworks API ✅ 189ms
```
- **Mineworks API** = server mining (`api.minework.net`). Tempat miner kirim heartbeat, submit data, claim task.
- Kalau ini mati, **miner kamu tetap jalan tapi tidak bisa submit** → 0 reward.

**Kapan muncul tambahan:**
```
├─ 🗃️ DB: ❌ DOWN
├─ 📡 Redis: ❌ DOWN
```
- Baris DB/Redis **hanya muncul saat error**. Kalau sehat, tidak ditampilkan.

---

## ⛏️ Mining Hari Ini — Penjelasan Per Baris

```
├─ 📅 Epoch  : 2026-04-28 (🟢 OPEN)
```
- **Epoch** = satu siklus mining 24 jam (00:00 UTC → 00:00 UTC).
- **🟢 OPEN** = epoch sedang berjalan, kamu bisa kirim submission.
- **✅ COMPLETED** = epoch selesai, reward sudah dihitung.
- **🔄 SETTLING** = sedang menghitung siapa yang dapat reward.

```
│  └ Kemarin: 3.6M sub → 44,089 rewarded
```
- Ringkasan epoch kemarin. **3.6M sub** = 3.6 juta submission total.
- **44,089 rewarded** = 44 ribu miner yang lolos kualifikasi dan dapat aMINE.

```
├─ 📤 Submit : 1,234,567 (890,000 valid)
```
- Hanya muncul kalau epoch sudah punya data.
- **1,234,567** = total submission masuk. **890,000 valid** = yang lolos evaluasi.

```
├─ 🏆 Reward : 44,089 miner dapat reward
```
- Jumlah miner unik yang qualified di epoch ini.

```
├─ 📊 Live   : 15,230 sub | 4,100 eval
```
- Data real-time dari dashboard. **sub** = submission berjalan. **eval** = evaluasi berjalan.
- Hanya muncul kalau nilainya > 0.

```
├─ 👷 Miner  : 13,352 online
```
- Jumlah miner yang heartbeat-nya masih aktif (online < 120 detik terakhir).
- Turun drastis = ada masalah server, maintenance, atau ban massal.

```
├─ 🏅 Top    : 142,000 aMINE (miner terbaik)
```
- Total aMINE dari miner paling produktif. Ini benchmark — targetmu.

```
└─ 📈 Avg Score: 78/100
```
- Rata-rata credit score semua miner. Score ini menentukan PoW probability dan submit limit.
- Di bawah 60 = tier "normal" atau lebih rendah. Di atas 80 = "excellent".

---

## 📦 Dataset Ranking — Penjelasan Per Baris

```
├─ 🥇 Wikipedia              4.6M ×1
```
- **Wikipedia** = nama dataset yang bisa di-mine.
- **4.6M** = total submission kumulatif ke dataset ini (bukan hari ini, tapi sepanjang waktu).
- **×1** = reward multiplier (emission weight). **Semakin besar angka = semakin banyak aMINE per task.**

```
├─ 🥉 LinkedIn Profiles      397K ×12
```
- LinkedIn Profiles punya **×12** multiplier — artinya 1 task LinkedIn = 12× reward dibanding 1 task Wikipedia.
- Tapi butuh LinkedIn auth/cookies untuk mining dataset ini.

```
  ↳ ×N = reward multiplier (makin besar = cuan)
```
- Penjelasan singkat untuk member baru.

### Tabel Multiplier Lengkap

| Dataset | ×Weight | Butuh Apa? |
|---------|:-------:|------------|
| LinkedIn Profiles | ×12 | LinkedIn login |
| Amazon Products | ×8 | Playwright + proxy residential |
| Amazon Reviews | ×8 | Playwright + proxy residential |
| Basic Amazon Products | ×8 | Playwright + proxy residential |
| LinkedIn Jobs | ×5 | LinkedIn login |
| LinkedIn Company | ×5 | LinkedIn login |
| LinkedIn Posts | ×5 | LinkedIn login |
| Wikipedia | ×1 | Tidak butuh apa-apa |
| arXiv | ×1 | Tidak butuh apa-apa |

---

## 💰 Info Reward & Kualifikasi — Penjelasan Per Baris

```
├─ 💎 Reward Split: 41% miner | 41% validator | 18% owner
```
- Dari total emisi per epoch: **41%** dibagi ke semua miner, **41%** ke validator, **18%** ke subnet owner.
- Source: protocol-configs dari api-server-spec.

```
├─ ⏰ Settlement  : 00:00 UTC (harian)
```
- Reward dihitung dan dibagikan **setiap 00:00 UTC** (07:00 WIB).

```
├─ 📋 Min. Syarat: ≥80 tasks + avg score ≥60
```
- Untuk dapat reward di satu epoch, kamu **HARUS**:
  - Submit **minimal 80 tasks** (bukan 10!)
  - Punya **rata-rata score ≥60** dari evaluasi validator
- Kurang salah satu = **0 reward** untuk epoch itu.

```
├─ ⚠️ LLM Enrich : WAJIB! Tanpa = max score ~55
```
- **LLM Enrichment** = proses pengayaan data pakai AI (via CLIProxyAPI).
- **Tanpa enrichment**, skor maksimal yang bisa kamu dapat hanya ~55 dari 100.
- Karena minimum syarat adalah score ≥60, artinya **tanpa enrichment = TIDAK MUNGKIN dapat reward**.

```
└─ 🎯 Credit Tier: novice→100% PoW | good→5% | excel→1%
```
- **Credit Tier** menentukan seberapa sering kamu kena PoW (Proof of Work) challenge.
- **Novice** (score 0-19): setiap submit 100% pasti kena PoW.
- **Good** (60-79): hanya 5% chance kena PoW.
- **Excellent** (80-100): 1% chance — hampir tidak pernah kena.

---

## ⚠️ Masalah Terdeteksi — Penjelasan Per Baris

Section ini **hanya muncul kalau ada masalah**. Kalau semua sehat, tidak ditampilkan.

```
├─ 📡 AWP API: DOWN — Connection refused
```
- Server AWP tidak bisa dihubungi. Mining bisa jalan tapi tidak bisa kirim proof/submit.

```
├─ 🗃️ Database: DOWN
```
- Database server mati. Semua operasi gagal.

```
├─ ⛓️ Base: keeper cache mati
```
- Keeper di chain Base berhenti. Ini yang kirim transaksi on-chain. Settlement reward tertunda.

```
├─ 💰 Base ETH gas kritis (0.001 ETH)
```
- Saldo gas relayer hampir habis. Tidak bisa kirim transaksi on-chain untuk settle reward.

---

## 🎨 Warna Embed

| Warna | Judul Embed | Artinya |
|:-----:|-------------|---------|
| 🟢 Hijau | ALL SYSTEMS OPERATIONAL | Semua sehat. Aman mining. |
| 🟡 Kuning | ⚠️ PARTIAL DEGRADATION | Ada masalah ringan (keeper/gas/1 API). Mining mungkin lambat. |
| 🔴 Merah | 🔴 OFFLINE | Semua API mati. **Hentikan mining sementara.** |
| 🟢 Hijau | ✅ RECOVERED | Baru pulih dari outage. Bisa mulai mining lagi. |

---

## ⏱️ Footer

```
Mulai: 29 Apr 06:00 | Cek: 29 Apr 13:10 | Uptime: 7h 10m
```
- **Mulai** = kapan monitor pertama kali dijalankan.
- **Cek** = waktu pengecekan terakhir (di-refresh setiap beberapa menit).
- **Uptime** = berapa lama monitor sudah aktif tanpa restart.

---

*AWP/Mineworks Discord Status Monitor v3.0.1*
*Penjelasan lengkap: https://github.com/astrofounder/awp/blob/main/discord-alert/MONITOR.md*
