# 📖 Panduan Membaca Status Monitor AWP

Dokumen ini menjelaskan arti setiap field yang tampil di embed Discord status monitor.

---

## 🌐 API Health

| Field | Arti |
|-------|------|
| **AWP API** | Server backend utama AWP. Mengurus registrasi miner, staking, distribusi reward, dan semua operasi protokol. Jika down, mining tidak bisa mengirim proof. |
| **Mineworks API** | Server mining pool Mineworks. Tempat miner mengambil task, mengirim heartbeat, dan mencatat skor. Jika down, miner tetap jalan tapi tidak dapat task baru. |
| **Database** | Penyimpanan data permanen AWP (staking records, wallet, history epoch). Jika down tapi HTTP masih 200, API kelihatan online padahal tidak bisa memproses data. |
| **Redis** | Cache dan antrian kerja internal. Mempercepat akses data yang sering dipakai. Jika down, API bisa jadi sangat lambat atau timeout. |
| **Angka `ms`** | Waktu respon server dalam milidetik. Makin kecil makin bagus. Di atas 1000ms berarti lambat. |

---

## ⛓️ Blockchain

AWP beroperasi di 4 blockchain sekaligus:

| Chain | Nama Lengkap | Keterangan |
|-------|-------------|------------|
| **ETH** | Ethereum | Mainnet utama |
| **BSC** | BNB Smart Chain | Chain Binance |
| **Base** | Base | Layer 2 milik Coinbase — chain utama AWP saat ini |
| **ARB** | Arbitrum One | Layer 2 Ethereum |

### Field per chain:

| Field | Arti |
|-------|------|
| **Blok #25.0M** | Block terakhir yang sudah diproses indexer AWP dari blockchain. Jika angka ini berhenti naik selama beberapa cycle, berarti indexer macet/lag. |
| **Keeper ✅ OK / ❌ DOWN** | Keeper = service internal yang mengirim transaksi on-chain (distribusi reward, settlement epoch). Jika DOWN, reward di chain ini tertunda sampai keeper aktif lagi. |
| **Gas: 0.067 ETH** | Saldo native token (ETH/BNB) yang dimiliki relayer AWP untuk membayar biaya transaksi. Jika hampir habis, relayer tidak bisa mengirim TX = reward macet. |

---

## 📊 AWP Network

| Field | Arti |
|-------|------|
| **Total User** | Jumlah seluruh user terdaftar di protokol AWP. |
| **Epoch** | Periode distribusi reward (seperti "gajian"). Nomor epoch naik setiap kali reward didistribusikan. Jika tidak naik dalam waktu lama, ada masalah settlement. |
| **Total Stake** | Total token AWP yang dikunci (staked) oleh seluruh user. Indikator kesehatan ekosistem — turun drastis = banyak yang unstake. |
| **Subnet** | Area kerja spesifik untuk miner. Setiap subnet punya task dan reward pool sendiri. |

---

## ⛏️ Mining Pool

| Field | Arti |
|-------|------|
| **Miner Online** | Berapa miner yang sedang aktif vs total terdaftar. Jika turun drastis (misal 487 → 50), kemungkinan ada masalah server Mineworks atau ban massal. |
| **Top Reward** | Total AWP yang sudah dihasilkan oleh miner dengan pendapatan tertinggi. Berguna untuk benchmarking performa. |
| **Skor Rata²** | Rata-rata quality score semua miner (0-100). Skor ini mempengaruhi reward multiplier. Bandingkan skor miner kamu dengan rata-rata ini. |

---

## ⚠️ Masalah Terdeteksi

Jika ada masalah, monitor otomatis menampilkan section ini dengan penjelasan dampaknya:

| Contoh Issue | Arti & Dampak |
|-------------|---------------|
| 🗃️ Database DOWN | Penyimpanan data mati. Semua operasi yang butuh data (staking, reward) gagal. |
| 📡 Redis DOWN | Cache mati. API jadi lambat, bisa timeout. |
| ⛓️ ARB keeper mati | Keeper di Arbitrum berhenti. Reward untuk chain ini tertunda. |
| 💰 ETH gas hampir habis | Relayer Ethereum kehabisan saldo gas. Tidak bisa kirim transaksi on-chain. |
| ⏸️ Base epoch PAUSED | Epoch di Base di-pause oleh tim (maintenance). Mining tetap jalan tapi reward belum didistribusikan. |
| 📡 AWP API DOWN | Server utama tidak merespon sama sekali. Mining tidak bisa mengirim proof. |

---

## 🎨 Warna Embed

| Warna | Status | Artinya |
|-------|--------|---------|
| 🟢 Hijau | ALL SYSTEMS OPERATIONAL | Semua sehat, tidak ada masalah |
| 🟡 Kuning | PARTIAL DEGRADATION | Ada masalah ringan (keeper down, gas rendah) tapi API masih jalan |
| 🟡 Kuning | PARTIAL OUTAGE | Salah satu API mati, yang lain masih jalan |
| 🔴 Merah | OFFLINE | Semua API tidak merespon |
| 🟢 Hijau | RECOVERED ✅ | Baru pulih dari downtime, disertai ping notifikasi |

---

## ⏱️ Monitor

| Field | Arti |
|-------|------|
| **Mulai** | Kapan monitor mulai berjalan (atau terakhir restart). |
| **Cek** | Waktu pengecekan terakhir. |
| **Uptime** | Berapa lama monitor sudah berjalan sejak mulai. |
| **Check count** | Berapa kali pengecekan sudah dilakukan. |

---

*AWP/Mineworks Discord Status Monitor v2.0.0*
