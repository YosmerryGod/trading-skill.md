# AI Response Protocol — Trading Backtest

## Tujuan

Dokumen ini mengatur bagaimana AI merespons permintaan backtest. Aturan strategi, data, dan perhitungan teknis tetap mengikuti `backtest-skill.md`. Jika kedua dokumen berbeda, aturan strategi terbaru yang disepakati di `backtest-skill.md` menjadi acuan teknis; format respons mengikuti dokumen ini.

## 1. Saat pengguna mengirim simbol

- Jika simbol jelas, langsung mulai mengambil data dan menyiapkan baseline backtest. Jangan bertanya apakah pengguna ingin analisis atau backtest.
- Jangan memilih simbol sendiri. Pengguna harus menentukan simbol.
- Jika kontrak ambigu, tanyakan hanya apakah yang dimaksud Binance USDⓈ-M perpetual.
- Jangan mengarang hasil. Jika data atau tool tidak tersedia, katakan bahwa backtest belum dapat dijalankan dan jelaskan bagian yang kurang.
- Gunakan periode evaluasi 1 hari, 7 hari, dan 30 hari yang sudah selesai (UTC).
- Jalankan baseline lebih dahulu. Jangan otomatis menjalankan eksperimen ATR dinamis atau eksperimen tambahan lain sebelum baseline selesai atau diminta pengguna.

## 2. Baseline yang wajib

Untuk setiap simbol, baseline mencakup:

- Strategi Parabolic SAR Follow Trend.
- Filter tren 1H dan sinyal entry 5M sesuai `backtest-skill.md`.
- LONG dan SHORT aktif.
- Empat konfigurasi tetap:
  - TP 3% / SL 2%.
  - TP 5% / SL 3%.
  - TP 7% / SL 5%.
  - TP 10% / SL 7%.
- Periode 1, 7, dan 30 hari.
- Modal awal 100 USDT, leverage 3x, Cross Margin, menggunakan 100% saldo tersedia sebagai margin per trade, compounding.
- Satu posisi terbuka pada satu waktu.
- Biaya, slippage, funding, exchange filters, dan liquidation dimodelkan sesuai `backtest-skill.md`.

TP/SL dinamis berbasis ATR adalah eksperimen tambahan, bukan bagian baseline. Kolom dinamis hanya ditampilkan setelah eksperimen dinamis benar-benar dijalankan.

## 3. Format respons — hanya dua tabel utama

Jangan membuat banyak tabel terpisah, kartu statistik, atau bagian panjang yang mengulang hasil. Respons backtest standar hanya memiliki dua tabel utama, ditambah catatan singkat yang diperlukan untuk menjelaskan status dan keterbatasan data.

### Tabel 1 — Ringkasan periode

Satu baris untuk masing-masing periode: 1 hari, 7 hari, dan 30 hari. Untuk setiap konfigurasi TP/SL, tampilkan tiga metrik: Net PnL, jumlah trade, dan win rate. Tambahkan Max Drawdown untuk ringkasan periode bila tersedia.

Contoh struktur:

| Periode | TP3/SL2 PnL | Trades | Win rate | TP5/SL3 PnL | Trades | Win rate | TP7/SL5 PnL | Trades | Win rate | TP10/SL7 PnL | Trades | Win rate |
|---|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|
| 1 hari | ... | ... | ... | ... | ... | ... | ... | ... | ... | ... | ... | ... |
| 7 hari | ... | ... | ... | ... | ... | ... | ... | ... | ... | ... | ... | ... |
| 30 hari | ... | ... | ... | ... | ... | ... | ... | ... | ... | ... | ... | ... |

- PnL yang ditampilkan adalah Net PnL setelah fee, slippage, dan funding yang berhasil dimodelkan.
- Jangan isi angka jika perhitungan belum selesai. Gunakan status yang jelas, bukan angka perkiraan.
- Jika eksperimen dinamis sudah diminta dan dijalankan, tambahkan grup kolom Dinamis: PnL, Trades, Win rate.
- Jangan menyebut satu TP/SL sebagai yang terbaik hanya dari PnL. Pertimbangkan trade count, win rate, drawdown, profit factor, expectancy, dan kualitas data dalam catatan singkat.

### Tabel 2 — Breakdown harian berdasarkan volatilitas

Tabel kedua adalah breakdown harian. Untuk setiap hari yang diminta, kelompokkan hasil berdasarkan rentang volatilitas aktual dalam persen dan tiga mode:

1. **ALL TRADING** — tidak memakai filter volatilitas; semua sinyal valid boleh diperdagangkan.
2. **HANYA TRADING (ONLY)** — hanya entry ketika volatilitas pada saat sinyal berada dalam rentang yang ditampilkan.
3. **TIDAK TRADING (EXCLUDE)** — melewati entry ketika volatilitas berada dalam rentang yang ditampilkan, tetapi tetap trading pada rentang lainnya.

Gunakan rentang volatilitas angka aktual dengan interval 5 poin persentase, misalnya:
- 0% sampai <5%.
- 5% sampai <10%.
- 10% sampai <15%.
- 15% sampai <20%.
- Lanjutkan interval 5% yang sama sampai mencakup nilai tertinggi yang ditemukan.
- Tampilkan batas interval secara eksplisit. Nilai tepat di batas masuk ke interval yang dimulai pada batas tersebut.
- Jangan mengubah volatilitas menjadi label V1/V2/V3/V4.
- Jika nilai volatilitas tidak tersedia, tandai sebagai `VOLATILITY_UNAVAILABLE`; jangan masukkan ke interval buatan.

Contoh struktur untuk satu hari:

| Volatilitas | Mode | TP3/SL2 PnL | Trades | Win rate | TP5/SL3 PnL | Trades | Win rate | TP7/SL5 PnL | Trades | Win rate | TP10/SL7 PnL | Trades | Win rate |
|---|---|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|
| 0–5% | ALL TRADING | ... | ... | ... | ... | ... | ... | ... | ... | ... | ... | ... | ... |
| 0–5% | HANYA TRADING | ... | ... | ... | ... | ... | ... | ... | ... | ... | ... | ... | ... |
| 0–5% | TIDAK TRADING | ... | ... | ... | ... | ... | ... | ... | ... | ... | ... | ... | ... |
| 5–10% | ALL TRADING | ... | ... | ... | ... | ... | ... | ... | ... | ... | ... | ... | ... |
| 5–10% | HANYA TRADING | ... | ... | ... | ... | ... | ... | ... | ... | ... | ... | ... | ... |
| 5–10% | TIDAK TRADING | ... | ... | ... | ... | ... | ... | ... | ... | ... | ... | ... | ... |

- Untuk setiap rentang, ALL TRADING adalah baseline tanpa filter dan boleh ditampilkan berulang sebagai pembanding; nilainya harus konsisten untuk hari/periode yang sama.
- ONLY dan EXCLUDE adalah dua simulasi independen yang dijalankan ulang dari saldo awal yang sama. Jangan menghitungnya dengan mengurangi atau memisahkan PnL ALL secara aritmetis.
- Setiap simulasi tetap menerapkan satu posisi terbuka, cooldown, compounding, biaya, funding, dan liquidation.
- Posisi yang sudah terbuka tetap dikelola sampai TP/SL/liquidation meskipun volatilitas kemudian berpindah rentang.
- PnL, jumlah trade, dan win rate dalam sel harus berasal dari simulasi mode dan rentang yang benar, bukan hasil agregasi yang tidak sebanding.
- Pada tabel kedua, tampilkan satu hari saja jika pengguna meminta satu hari tertentu. Jika backtest mencakup 7 atau 30 hari, tampilkan breakdown untuk setiap hari pada periode yang diminta, termasuk hari tanpa trade.
- Jika tabel terlalu panjang untuk satu pesan, pecah berdasarkan hari tanpa menghilangkan baris atau menggabungkan hasil berbeda.

### Cara membaca win rate

Win rate = jumlah trade TP dibagi jumlah trade selesai yang diklasifikasikan TP atau SL. Liquidation, ambiguous, data-gap, dan posisi open-at-end harus dihitung serta dilaporkan terpisah dan tidak diam-diam dimasukkan sebagai win atau loss.

## 4. Eksperimen dinamis ATR

Jangan jalankan pada baseline awal kecuali pengguna meminta.

Konfigurasi eksperimen:
- ATR timeframe 5M, periode 14, Wilder's RMA.
- Nilai ATR berasal dari candle 5M terakhir yang sudah selesai sebelum entry.
- SL = ATR × 1.0, 1.5, 2.0, atau 2.5.
- TP ditentukan dengan Risk/Reward 1:1, 1:1.5, 1:2, atau 1:3.
- Total 16 kombinasi.
- Tidak ada batas maksimum TP/SL.
- Level TP dan SL dikunci pada saat entry.

Jika dinamis diminta, tampilkan hasilnya dalam grup kolom Dinamis pada dua tabel yang sama. Jelaskan kombinasi ATR multiplier dan RR yang diuji dalam catatan singkat, jangan membuat tabel ketiga.

## 5. Catatan wajib setelah tabel

Setelah dua tabel, tambahkan catatan singkat saja yang berisi:
- Status backtest: `NOT_RUN`, `INCOMPLETE_DATA`, `RUN_COMPLETE_GROSS`, `RUN_COMPLETE_NET`, atau `INCONCLUSIVE`.
- Sumber dan kelengkapan data.
- Asumsi biaya/slippage/funding yang belum dapat diverifikasi.
- Jumlah liquidation, ambiguous 1M SL-first, data gap, dan posisi open-at-end jika ada.
- Lokasi file riwayat hasil jika laporan benar-benar sudah disimpan.

Jangan menambahkan tabel ketiga untuk LONG/SHORT, trade log, daily summary, atau biaya. Detail trade log dan skip log tetap disimpan sebagai artefak audit/riwayat, dan dapat ditampilkan jika pengguna memintanya.

## 6. Penyimpanan hasil

Setelah backtest benar-benar selesai:
- Simpan atau perbarui `history-backtest/{SYMBOL}.md`.
- Ikuti format dan aturan di `history-backtest/README.md`.
- Simpan parameter, timestamp UTC, hasil dua tabel, data quality, dan referensi trade log.
- Jangan membuat file seolah-olah hasil backtest telah ada jika simulasi belum dijalankan.

## 7. Larangan

- Jangan mengarang angka atau menyatakan backtest selesai sebelum data benar-benar diuji.
- Jangan otomatis memilih simbol.
- Jangan menjalankan eksperimen dinamis sebelum baseline kecuali diminta.
- Jangan mengubah strategi atau parameter tanpa persetujuan pengguna.
- Jangan menampilkan lebih dari dua tabel utama pada respons standar.
- Jangan menyamakan gross PnL dengan net PnL.
- Jangan menyebut hasil 1, 7, atau 30 hari sebagai bukti strategi pasti profitable.
