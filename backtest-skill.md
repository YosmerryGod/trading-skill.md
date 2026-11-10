---
name: parabolic-sar-follow-trend-backtest
description: Reproducibly backtest a Parabolic SAR trend-following breakout strategy on Binance USD-M Futures using 1h trend, 5m candle-close entries, 1m exit sequencing, loss-streak cooldowns, TP/SL variants, and volatility-regime comparisons.
---

# Backtest Skill — Parabolic SAR Follow Trend

## Tujuan

Mengukur secara jujur performa strategi **Parabolic SAR Follow Trend** pada data historis Binance USDⓈ-M Futures. Skill ini wajib menjalankan simulasi kronologis dan melaporkan seluruh hasil, termasuk loss, periode tanpa trade, data yang tidak lengkap, dan keterbatasan resolusi candle.

Skill ini bukan alat untuk membuktikan strategi pasti profit. Hasil historis tidak menjamin hasil masa depan.

## 1. Strategi yang diuji — jangan mengubah aturan diam-diam

### 1.1 Instrumen dan sumber data

- Default instrumen: simbol Binance USDⓈ-M perpetual yang diminta pengguna. Jika pengguna belum menentukan simbol, tanyakan satu pertanyaan.
- Sumber utama: data historis Binance Futures.
- Simpan waktu internal dalam UTC. Tampilkan waktu lokal hanya jika zona waktu disebutkan.
- Jangan mencampur data spot dan perpetual futures.
- Catat simbol, contract type, exchange, waktu pengambilan data, rentang waktu, dan endpoint/sumber.

### 1.2 Indikator Parabolic SAR

Gunakan satu implementasi PSAR yang konsisten untuk seluruh pengujian.

Parameter default penelitian ini:
- Step / acceleration factor: 0.01.
- Maximum acceleration factor: 0.10.

Jika pengguna meminta parameter lain, buat versi eksperimen baru. Jangan mengganti parameter di tengah pengujian.

Catat:
- Formula dan implementasi/library beserta versinya.
- Parameter PSAR.
- Warm-up candle yang digunakan.
- Cara menangani reversal dan initial trend.
- Apakah hasil sudah dibandingkan dengan implementasi referensi seperti Binance/TradingView. Jangan mengaku identik jika belum diverifikasi.

PSAR dihitung dari candle yang sudah selesai saja. Dilarang memakai nilai high, low, atau close candle yang belum selesai.

### 1.3 Filter arah tren — timeframe 1H

Gunakan candle 1H terakhir yang sudah benar-benar close pada waktu sinyal 5M.

- Bullish trend: close 1H terakhir > PSAR 1H terakhir.
- Bearish trend: close 1H terakhir < PSAR 1H terakhir.
- Jika close sama dengan PSAR atau data 1H belum tersedia: tidak ada sinyal.

Jangan memakai candle 1H yang masih berjalan. Simpan timestamp candle 1H yang menjadi filter untuk setiap trade.

### 1.4 Sinyal entry — timeframe 5M

Sinyal baru sah setelah candle 5M selesai.

**LONG**
1. Filter tren 1H bullish.
2. Candle 5M sebelumnya belum memenuhi kondisi body sepenuhnya di atas PSAR.
3. Candle 5M sinyal memiliki seluruh body di atas PSAR terakhir: `min(open, close) > PSAR`.
4. Ini harus merupakan breakout/cross baru, bukan setiap candle lanjutan yang tetap berada di atas PSAR.

**SHORT**
1. Filter tren 1H bearish.
2. Candle 5M sebelumnya belum memenuhi kondisi body sepenuhnya di bawah PSAR.
3. Candle 5M sinyal memiliki seluruh body di bawah PSAR terakhir: `max(open, close) < PSAR`.
4. Ini harus merupakan breakout/cross baru, bukan setiap candle lanjutan yang tetap berada di bawah PSAR.

Jika candle menyeberangi PSAR tetapi body tidak seluruhnya berada di sisi yang sesuai, jangan hitung sebagai sinyal valid.

### 1.5 Waktu entry dan fill

- Sinyal diketahui setelah candle 5M close.
- Untuk menghindari look-ahead, entry simulasi dilakukan pada open candle 1M pertama setelah waktu close candle sinyal 5M.
- Jika data 1M pada waktu tersebut tidak ada, tandai sinyal tidak dapat dieksekusi dan jangan mengarang harga fill.
- Catat harga close candle sinyal 5M dan harga fill 1M secara terpisah.
- Jangan menganggap bisa mengisi order pada harga yang sudah diketahui setelah candle close jika harga tersebut tidak lagi tersedia untuk dieksekusi.

### 1.6 Posisi dan aturan risiko

- Maksimal satu posisi terbuka pada satu waktu.
- LONG dan SHORT sama-sama diperbolehkan sesuai filter tren.
- TP/SL dihitung dari harga fill aktual simulasi, bukan dari close sinyal.
- Setelah posisi ditutup, cari sinyal berikutnya secara kronologis.
- Setelah **3 kerugian beruntun**, kunci entry baru selama **24 jam sejak trade ketiga yang loss selesai**.
- Selama cooldown, jangan membuka trade baru. Data tetap dipantau.
- Posisi yang sudah terbuka tetap dikelola sampai TP, SL, atau akhir data. Cooldown tidak menutup posisi yang sedang berjalan.
- Win memutus loss streak. Trade yang belum selesai di akhir dataset tidak boleh dihitung sebagai win atau loss.
- Catat waktu mulai dan akhir cooldown, loss streak sebelum/sesudah setiap trade, serta sinyal yang dilewati karena cooldown.

## 2. Variasi TP dan SL

Jalankan keempat konfigurasi secara terpisah. Jangan menggabungkan hasilnya.

| Eksperimen | Take profit | Stop loss | Reward:risk nominal |
|---|---:|---:|---:|
| A | +3% | -2% | 1.5:1 |
| B | +5% | -3% | 1.67:1 |
| C | +7% | -5% | 1.4:1 |
| D | +10% | -7% | 1.43:1 |

Untuk LONG:
- TP = entry × (1 + TP%).
- SL = entry × (1 - SL%).

Untuk SHORT:
- TP = entry × (1 - TP%).
- SL = entry × (1 + SL%).

Persentase di atas adalah pergerakan harga terhadap entry. Jangan menyebutnya return modal/margin atau ROI leverage. Jika PnL dolar ditampilkan, nyatakan notional, leverage, margin, dan biaya secara terpisah. Default pembanding nominal historis boleh memakai notional 100 USDT per trade, tetapi hanya jika disebutkan jelas dan tidak dianggap sebagai ukuran akun pengguna.

## 3. Resolusi exit 1 menit — aturan wajib

**Jangan menentukan urutan TP/SL dari candle 5M.** Entry memakai sinyal 5M, tetapi pemantauan exit harus menggunakan candle 1M.

Setelah entry:
1. Baca candle 1M secara berurutan mulai dari candle pertama setelah fill.
2. Periksa high/low candle 1M terhadap level TP dan SL sesuai arah posisi.
3. Jika hanya TP tersentuh, tutup sebagai TP.
4. Jika hanya SL tersentuh, tutup sebagai SL.
5. Jika satu candle 1M menyentuh TP dan SL sekaligus, urutan intramenit tidak diketahui dari OHLC. Gunakan asumsi konservatif **SL lebih dulu**, lalu beri label `AMBIGUOUS_1M_SL_FIRST`.
6. Jangan menyimpulkan urutan sebenarnya dari open/close candle 1M. OHLC 1M tetap tidak menunjukkan tick-by-tick path.
7. Jika pengguna membutuhkan kepastian urutan intramenit, jelaskan bahwa diperlukan trade/tick data atau data dengan resolusi lebih tinggi. Jangan menyebut 1M sebagai bukti urutan yang pasti.
8. Jika ada gap melewati SL/TP, jangan mengasumsikan fill pada level yang tidak tersedia. Gunakan harga open yang dapat dieksekusi sesuai aturan simulasi dan tandai gap/slippage.
9. Jika data 1M hilang selama posisi terbuka, tandai trade sebagai `DATA_GAP`; jangan menebak TP/SL mana yang lebih dulu. Keluarkan trade tersebut dari metrik final yang membutuhkan urutan exit pasti, tetapi laporkan jumlah dan detailnya.

Catat untuk setiap trade:
- Waktu entry, harga sinyal, harga fill.
- Level TP dan SL.
- Waktu sentuh pertama yang teramati pada 1M.
- Harga exit simulasi.
- Hasil: TP / SL / AMBIGUOUS_1M_SL_FIRST / DATA_GAP / OPEN_AT_END.
- Jumlah candle 1M sampai exit.
- Apakah TP dan SL sama-sama tersentuh dalam satu candle 1M.
- Timestamp candle 1M yang menentukan exit.

## 4. Pengukuran volatilitas

Volatilitas adalah konteks pasar untuk mengelompokkan trade, bukan sinyal arah.

Default definisi yang dipakai untuk eksperimen ini:
- Timeframe pengukuran: 5M.
- Lookback: 288 candle 5M yang telah selesai (24 jam).
- Formula:

`volatility_24h_pct = (highest_high_288 - lowest_low_288) / first_open_288 × 100`

- Untuk keputusan pada waktu sinyal, hanya gunakan 288 candle 5M yang selesai sebelum candle sinyal. Jangan memasukkan candle masa depan atau candle sinyal yang belum close.
- Catat nilai persentase aktual pada setiap sinyal dan trade.
- Jika 288 candle belum tersedia, label `VOLATILITY_UNAVAILABLE` dan jangan masukkan ke bucket.

### Bucket volatilitas yang diminta

Gunakan batas berikut persis:

| Bucket | Rentang |
|---|---:|
| V1 | 0% sampai <10% |
| V2 | 15% sampai <25% |
| V3 | 30% sampai <45% |
| V4 | >50% |

Rentang yang tidak disebut pengguna **tidak boleh dipaksakan masuk ke bucket lain**:
- 10% sampai <15%.
- 25% sampai <30%.
- 45% sampai 50% (termasuk 50%).
- Nilai di luar/kurang data.

Labeli sebagai `UNCATEGORIZED_GAP` atau `VOLATILITY_UNAVAILABLE`. Laporkan jumlah sinyal dan trade pada kategori tersebut. Jangan menghapusnya dari laporan keseluruhan.

Jika pengguna mengubah batas atau formula, buat versi eksperimen baru dan jangan mencampur hasil dengan versi ini.

## 5. Eksperimen volatilitas: trading vs tidak trading

Untuk setiap TP/SL, jalankan seluruh eksperimen berikut secara independen. Setiap eksperimen harus menjalankan ulang simulasi kronologis, termasuk aturan satu posisi dan cooldown 3 loss/24 jam. Jangan menjumlahkan hasil per bucket untuk membuat hasil portofolio karena urutan posisi dan cooldown dapat berubah.

### A. ALL_MARKETS — baseline

Trade pada semua kondisi volatilitas, termasuk bucket yang ditentukan, gap bucket, dan kategori data volatilitas yang tersedia. Pisahkan hasil trade dengan label volatilitasnya. Sinyal dengan data volatilitas yang tidak tersedia tetap boleh masuk baseline jika data lain lengkap; tandai labelnya.

### B. ONLY_THIS_BUCKET — hanya trading pada bucket

Jalankan empat simulasi terpisah:
- Hanya entry ketika volatilitas V1 (0–10%).
- Hanya entry ketika volatilitas V2 (15–25%).
- Hanya entry ketika volatilitas V3 (30–45%).
- Hanya entry ketika volatilitas V4 (>50%).

Sinyal di luar bucket terpilih dilewati. Posisi yang sudah terbuka tetap dikelola meskipun volatilitas berubah bucket setelah entry.

### C. EXCLUDE_THIS_BUCKET — tidak trading pada bucket

Jalankan empat simulasi terpisah:
- Tidak entry pada V1; bucket lain tetap boleh.
- Tidak entry pada V2; bucket lain tetap boleh.
- Tidak entry pada V3; bucket lain tetap boleh.
- Tidak entry pada V4; bucket lain tetap boleh.

Sinyal di kategori gap dan data volatilitas tidak tersedia tetap mengikuti kebijakan baseline dan harus dilaporkan terpisah. Jangan diam-diam menganggap kategori gap sebagai salah satu bucket.

**Makna perbandingan:** ONLY_THIS_BUCKET menjawab "bagaimana jika hanya trading di kondisi ini"; EXCLUDE_THIS_BUCKET menjawab "bagaimana jika menghindari kondisi ini dan tetap trading di kondisi lain". Keduanya wajib dibandingkan dengan ALL_MARKETS.

## 6. Rentang waktu backtest

Ambil data yang cukup untuk warm-up indikator dan lookback volatilitas sebelum awal periode evaluasi.

Jalankan laporan untuk:
- 1 hari terakhir yang lengkap.
- 7 hari terakhir yang lengkap.
- 30 hari terakhir yang lengkap.

Definisi:
- Semua tanggal dan batas waktu evaluasi menggunakan UTC.
- Gunakan jendela waktu yang benar-benar sudah selesai; jangan memasukkan candle yang masih berjalan.
- Untuk periode 7 dan 30 hari, sertakan breakdown **setiap hari kalender UTC**.
- Laporan 30 hari wajib memiliki 30 baris harian per konfigurasi, termasuk hari dengan nol trade.
- Laporan 7 hari wajib memiliki 7 baris harian.
- Laporan 1 hari wajib memiliki 1 baris harian.
- Tampilkan juga ringkasan kumulatif masing-masing jendela 1, 7, dan 30 hari.
- Hari kalender tanpa trade tetap ditulis dengan trade = 0, bukan dihilangkan.
- Untuk menghindari hasil parsial, sebutkan timestamp awal/akhir dan status kelengkapan data.

Jika pengguna meminta tanggal tertentu, tanggal tersebut menggantikan jendela relatif. Jangan mengasumsikan timezone selain UTC untuk perhitungan.

## 7. Metrik wajib

Untuk setiap kombinasi periode × TP/SL × mode volatilitas, hitung dan laporkan:

- Jumlah sinyal valid.
- Sinyal dilewati karena filter tren.
- Sinyal dilewati karena volatilitas.
- Sinyal dilewati karena cooldown.
- Total trade yang benar-benar dibuka.
- TP count.
- SL count.
- Ambiguous 1M count.
- Data-gap count.
- Open-at-end count.
- Win rate = TP / (TP + SL) × 100. Jelaskan bahwa ambiguous/data-gap/open tidak masuk denominator.
- Gross PnL dalam persen-R atau USDT sesuai notional yang dinyatakan.
- Total gross profit.
- Total gross loss.
- Net PnL setelah biaya jika biaya benar-benar dimodelkan.
- Profit factor = gross profit positif / nilai absolut gross loss.
- Expectancy per trade.
- Rata-rata dan median hasil trade.
- Maximum drawdown dari equity curve kronologis.
- Loss streak maksimum.
- Jumlah cooldown dan total jam terkunci.
- Exposure time / waktu berada dalam posisi.
- Fee, slippage, dan funding yang dimodelkan atau dinyatakan tidak tersedia/tidak dihitung.

Jangan menyebut gross PnL sebagai net profit. Jika fee, slippage, atau funding belum tersedia, tampilkan hasil gross dan tulis dengan jelas bahwa hasil belum dikurangi komponen tersebut.

### Perhitungan PnL

Untuk notional tetap:
- LONG gross return = (exit - entry) / entry.
- SHORT gross return = (entry - exit) / entry.
- Gross PnL USDT = gross return × notional.

Jika menggunakan leverage, jangan mengalikan return harga dan menyebutnya ROI akun tanpa menghitung margin dan likuidasi. Leverage tidak mengubah persentase pergerakan harga TP/SL.

Jika fee/slippage/funding tidak dihitung, beri label `GROSS — COSTS EXCLUDED`.

## 8. Format laporan wajib

### 8.1 Metadata pengujian

Laporkan:
- Symbol/contract.
- Exchange dan sumber data.
- Timestamp UTC awal/akhir.
- Jumlah candle 1H, 5M, dan 1M.
- Data gap, duplikasi, dan candle yang tidak lengkap.
- PSAR parameter, implementasi, warm-up, dan status verifikasi.
- Formula/lookback volatilitas.
- Asumsi entry, exit, fill, fee, slippage, funding, dan notional.
- Versi kode/backtest engine dan waktu eksekusi.

### 8.2 Ringkasan per TP/SL

Tampilkan ALL_MARKETS untuk keempat TP/SL pada masing-masing periode 1, 7, dan 30 hari.

### 8.3 Perbandingan volatilitas

Untuk setiap TP/SL, tampilkan ALL_MARKETS, empat ONLY_THIS_BUCKET, dan empat EXCLUDE_THIS_BUCKET. Jangan hanya menampilkan strategi terbaik. Semua eksperimen yang dijalankan harus terlihat.

### 8.4 Breakdown harian sampai 30 hari

Buat tabel per hari UTC untuk setiap TP/SL dan setiap mode volatilitas. Minimal kolom:

| Date UTC | Signals | Trades | TP | SL | Ambiguous | Win rate | Gross PnL | Net PnL | Loss streak max | Cooldown hours |
|---|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|

- Tampilkan baris nol trade.
- Jangan menggabungkan hasil LONG dan SHORT tanpa tetap menyediakan breakdown arah.
- Sertakan ringkasan LONG dan SHORT secara terpisah.
- PnL harian harus mengikuti trade yang selesai pada hari itu. Trade yang melintasi tengah malam tetap dicatat pada tanggal exit; sediakan juga tanggal entry untuk audit.

### 8.5 Trade log lengkap

Simpan seluruh trade dalam CSV/JSON, satu baris per trade, minimal berisi:
- trade_id, symbol, direction.
- signal_time_5m, signal_open/high/low/close, signal_psar.
- trend_time_1h, trend_close_1h, trend_psar_1h, trend_direction.
- volatility_pct, volatility_bucket.
- entry_time_1m, entry_price, notional.
- tp_pct, sl_pct, tp_price, sl_price.
- exit_time_1m, exit_price, exit_reason.
- ambiguous_1m, data_gap, bars_held_1m.
- gross_return_pct, gross_pnl, fee, slippage, funding, net_pnl.
- loss_streak_before, loss_streak_after, cooldown_start, cooldown_end.
- data_quality_flags, backtest_version.

Simpan juga daftar sinyal yang dilewati dan alasan skip, supaya hasil bisa diaudit. Ringkasan tabel bukan pengganti trade log.

## 9. Validasi data dan pemeriksaan anti-bias

Sebelum menyatakan backtest selesai:

1. Pastikan candle 1M, 5M, dan 1H berasal dari kontrak dan exchange yang sama.
2. Pastikan timestamp candle konsisten dan candle 1M yang diperlukan untuk exit tersedia.
3. Pastikan tidak ada candle masa depan yang masuk ke filter 1H atau volatilitas.
4. Pastikan candle 1H yang dipakai sudah close sebelum waktu sinyal 5M.
5. Pastikan sinyal 5M hanya diketahui setelah close.
6. Pastikan entry tidak diisi pada harga yang sudah berlalu.
7. Pastikan TP/SL diputuskan memakai 1M, bukan high/low 5M.
8. Audit candle 1M yang menyentuh TP dan SL sekaligus.
9. Audit loss streak dan timestamp cooldown 24 jam.
10. Jalankan ulang satu sampel trade secara manual untuk memastikan hasil simulator cocok dengan data mentah.
11. Jalankan ulang eksperimen dengan konfigurasi yang sama dan pastikan hasil identik.
12. Jangan menghapus data gap atau hasil buruk tanpa mengungkapkannya.

## 10. Kejujuran hasil dan status kesimpulan

Aturan mutlak:
- Jangan membuat angka, trade, win rate, PnL, atau statistik yang tidak dihasilkan dari data.
- Jangan menyebut backtest sudah dijalankan jika baru menulis rencana atau kode.
- Jika data 1M belum tersedia, hasil exit belum tervalidasi; jangan memakai hasil 5M sebagai pengganti tanpa label.
- Jika data yang tersedia hanya 1, 7, atau 30 hari, jangan mengklaim periode lain sudah diuji.
- Jangan menyembunyikan konfigurasi yang rugi atau bucket dengan hasil buruk.
- Jangan memilih strategi terbaik hanya dari PnL; tampilkan jumlah trade, drawdown, profit factor, expectancy, biaya, dan keterbatasan sampel.
- Jangan menganggap periode 30 hari sebagai bukti statistik yang kuat.
- Jika PSAR custom belum dicocokkan dengan referensi, nyatakan secara eksplisit.
- Jika OHLC 1M menyentuh TP dan SL dalam menit yang sama, nyatakan urutan sebenarnya tidak diketahui. Asumsi SL-first adalah konservatif, bukan fakta tick.
- Jangan mengoptimasi parameter dengan data yang sama lalu menyebut hasilnya out-of-sample.

Gunakan status akhir:
- `NOT_RUN`: data/simulasi belum dijalankan.
- `INCOMPLETE_DATA`: data tidak cukup atau ada gap penting.
- `RUN_COMPLETE_GROSS`: simulasi lengkap, tetapi biaya belum dihitung.
- `RUN_COMPLETE_NET`: simulasi lengkap dan biaya yang relevan sudah dimodelkan.
- `INCONCLUSIVE`: hasil belum cukup untuk mengambil kesimpulan.

Keputusan strategi:
- `PROMISING_FOR_FURTHER_TEST`
- `WEAK`
- `REJECTED`
- `INCONCLUSIVE`

Jangan gunakan label `PROFITABLE` atau `ROBUST` hanya karena PnL 1–30 hari positif. Validasi lanjutan pada periode historis lebih panjang dan data out-of-sample diperlukan.

## 11. Alur kerja AI

Saat pengguna meminta backtest:

1. Baca dokumen ini dan `respon-ai.md`.
2. Pastikan symbol dan kontrak. Jika belum disebut, tanyakan symbol.
3. Ambil data 1H, 5M, dan 1M termasuk warm-up sebelum periode evaluasi.
4. Validasi kualitas dan cakupan data sebelum simulasi.
5. Bekukan konfigurasi strategi, PSAR, volatilitas, TP/SL, dan cooldown.
6. Jalankan semua konfigurasi TP/SL dan mode volatilitas yang ditetapkan.
7. Buat ringkasan 1, 7, dan 30 hari serta breakdown harian lengkap.
8. Simpan trade log dan skip log mentah agar dapat direproduksi.
9. Audit sampel trade, ambiguitas 1M, dan cooldown.
10. Laporkan hasil apa adanya, termasuk konfigurasi rugi dan keterbatasan.
11. Jangan mengubah strategi atau parameter tanpa persetujuan pengguna. Jika ada eksperimen baru, simpan sebagai versi terpisah.

## 12. Bahasa laporan

- Gunakan Bahasa Indonesia.
- Jawab langsung dan gunakan tabel untuk perbandingan.
- Bedakan fakta data, asumsi simulasi, dan interpretasi.
- Sertakan angka pembilang/penyebut, bukan win rate saja.
- Cantumkan tanggal, timezone, satuan, dan formula.
- Jangan memberikan rekomendasi entry live hanya karena backtest terlihat positif.
