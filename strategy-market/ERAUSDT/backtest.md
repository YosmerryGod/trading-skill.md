# ERAUSDT Backtest

Dokumen ini menyimpan hasil dan status pengujian strategi untuk market **ERAUSDT**. Parameter strategi utama dibaca dari `config.json`; jangan mengubah parameter di laporan ini secara terpisah dari konfigurasi.

## 1. Konfigurasi yang Diuji

| Parameter | Nilai |
|---|---:|
| Symbol | ERAUSDT |
| Strategi | PSAR Follow Trend |
| PSAR Step | 0.01 |
| PSAR Maximum | 0.10 |
| Trend timeframe | 1H |
| Entry timeframe | 5M |
| Position notional | 100 USDT |
| Compounding | Tidak |
| Take Profit | 7% |
| Stop Loss | 5% |
| Volatility minimum | 5% |
| Volatility maximum | 30% |
| Loss streak trigger | 3 loss berturut-turut |
| Pause setelah trigger | 24 jam |
| Timezone | UTC |

## 2. Periode Backtest

- Mulai: 2026-08-01 00:00:00 UTC
- Selesai: 2026-09-01 00:00:00 UTC (batas akhir eksklusif)
- Periode laporan: 1–31 Agustus 2026.
- Posisi awal: flat/tidak ada posisi terbuka.
- Posisi yang masih terbuka pada akhir periode harus dilaporkan terpisah dan tidak dianggap sebagai realized PnL.

## 3. Ringkasan Hasil Awal

> **Status: INCOMPLETE_DATA — hasil awal/provisional, belum tervalidasi penuh.**

| Metrik | Hasil |
|---|---:|
| Hari dalam periode | 31 |
| Hari memenuhi filter volatilitas | 22 |
| Total trade tertutup | 8 |
| Win | 3 |
| Loss | 5 |
| Win rate | 37.5% |
| Gross PnL | -4.00 USDT |
| Fee estimasi | -0.80 USDT |
| Funding historis terhitung | -0.93 USDT |
| Net PnL estimasi | **-5.73 USDT** |
| Jeda 24 jam terpicu | 1 kali |
| Posisi terbuka di akhir | Tidak ada |
| Slippage | Belum dihitung |
| Max drawdown | Belum dihitung |
| Profit factor | Belum divalidasi |

Perhitungan net pada simulasi awal:

`Net PnL = Gross PnL - Fee + Funding`

`-4.00 - 0.80 - 0.93 = -5.73 USDT`

Fee menggunakan asumsi 0.05% per sisi pada notional 100 USDT, yaitu sekitar 0.10 USDT untuk entry dan exit per trade. Nilai funding berasal dari riwayat funding yang tersedia dan estimasi arah posisi selama periode pendanaan.

## 4. Hari yang Memenuhi Filter Volatilitas

Formula rentang volatilitas harian:

`((Daily High - Daily Low) / Daily Open) * 100`

| Tanggal UTC | Volatilitas |
|---|---:|
| 2026-08-02 | 6.69% |
| 2026-08-03 | 11.55% |
| 2026-08-04 | 11.62% |
| 2026-08-05 | 8.72% |
| 2026-08-06 | 19.34% |
| 2026-08-07 | 11.39% |
| 2026-08-08 | 6.44% |
| 2026-08-10 | 7.27% |
| 2026-08-11 | 7.63% |
| 2026-08-14 | 8.18% |
| 2026-08-17 | 7.77% |
| 2026-08-18 | 5.95% |
| 2026-08-19 | 8.39% |
| 2026-08-20 | 8.18% |
| 2026-08-21 | 9.62% |
| 2026-08-22 | 17.22% |
| 2026-08-23 | 8.80% |
| 2026-08-24 | 17.78% |
| 2026-08-25 | 8.18% |
| 2026-08-28 | 7.72% |
| 2026-08-30 | 22.66% |
| 2026-08-31 | 13.42% |

Hari lainnya tidak memenuhi filter 5–30% atau tidak termasuk daftar hari eligible.

## 5. Aturan Eksekusi yang Digunakan

- Filter tren memakai candle 1H terakhir yang sudah closed.
- Sinyal entry memakai fresh PSAR cross pada candle 5M yang sudah closed.
- Entry disimulasikan pada close candle sinyal 5M.
- Maksimal satu posisi terbuka pada satu waktu.
- Setelah posisi ditutup, candle yang sama tidak digunakan untuk membuka posisi baru.
- Re-entry searah menunggu reset PSAR dan fresh cross berikutnya.
- Posisi dikelola sampai TP atau SL; perubahan PSAR 5M atau tren 1H tidak menutup posisi.
- Setelah tiga loss berturut-turut, entry baru dihentikan selama 24 jam. Sinyal yang muncul selama jeda diabaikan.

## 6. Keterbatasan dan Validasi yang Masih Diperlukan

Hasil pada bagian 3 bukan hasil final tervalidasi karena:

1. **Resolusi exit:** simulasi awal memeriksa TP/SL menggunakan OHLC candle 5M. Konfigurasi final mensyaratkan pemeriksaan candle 1M agar urutan sentuh TP dan SL dapat diketahui lebih baik.
2. **Ambiguitas TP/SL:** jika TP dan SL tersentuh dalam candle 1M yang sama, gunakan asumsi konservatif SL lebih dahulu dan beri label `AMBIGUOUS_1M_SL_FIRST`.
3. **Look-ahead bias volatilitas:** filter memakai high dan low dari candle harian penuh. Nilai akhir hari belum diketahui pada awal hari, sehingga filter ini hanya untuk eksperimen backtest dan tidak aman untuk entry real-time pada hari yang sama.
4. **Slippage:** belum dimodelkan. Jangan menganggap slippage bernilai nol.
5. **Drawdown dan profit factor:** belum dihitung dari equity curve/trade ledger yang tervalidasi.
6. **Funding:** perlu direkonsiliasi dengan timestamp entry, exit, dan aturan funding exchange secara tepat.
7. **Likuidasi dan leverage:** belum dimodelkan. Position notional 100 USDT bukan berarti margin atau leverage sudah ditetapkan.

## 7. Rencana Validasi Berikutnya

- [ ] Jalankan ulang Agustus dengan data candle 1M untuk validasi TP/SL.
- [ ] Simpan trade ledger lengkap: waktu entry/exit, arah, harga, TP, SL, alasan exit, fee, funding, dan net PnL.
- [ ] Hitung LONG/SHORT, TP hit, SL hit, loss streak terpanjang, dan jumlah pause.
- [ ] Hitung max drawdown dari equity curve dan profit factor.
- [ ] Tambahkan model slippage yang eksplisit atau tandai tetap tidak tersedia.
- [ ] Uji filter volatilitas tanpa look-ahead bias, misalnya memakai volatilitas hari sebelumnya atau range intraday yang sudah terjadi saat entry.
- [ ] Pastikan hasil akhir sesuai dengan `config.json` dan `strategy-skill-backtest.md`.

## 8. Aturan Pemeliharaan Dokumen

- Setiap backtest baru harus menambahkan periode, versi konfigurasi, sumber data, asumsi, dan hasilnya.
- Jangan menimpa hasil periode lama tanpa menyimpan histori perubahan.
- Jika parameter di `config.json` berubah, catat versi atau snapshot konfigurasi yang digunakan untuk setiap hasil.
- Jangan menyebut hasil sebagai DATA LENGKAP sebelum seluruh komponen penting divalidasi.


---

# Riwayat Backtest September–November 2026

Bagian ini menyimpan eksperimen yang sebelumnya sudah dijalankan untuk ERAUSDT. Hasil di bawah adalah **hasil historis/provisional dari simulasi terdahulu**, bukan hasil final tervalidasi. Jangan menghapus atau menimpa hasil Agustus di atas.

## 9. Parameter dan Cakupan Eksperimen

- Market: ERAUSDT USDT-M Futures.
- Notional tetap: 100 USDT per posisi, tanpa compounding.
- Filter volatilitas: 5–30% berdasarkan rentang high-low harian terhadap open harian.
- Strategi dasar: PSAR Follow Trend, trend 1H dan entry 5M.
- Periode yang dicoba:
  - September 2026: 1–30 September.
  - Oktober 2026: 1–31 Oktober.
  - November 2026: 1–10 November; 11 November tidak dimasukkan karena hari belum selesai.
- Konfigurasi TP/SL yang dibandingkan: 3%/2%, 5%/3%, 7%/5%, dan 10%/7%.
- Setiap periode bulanan dimulai dari kondisi flat secara independen. Hasil akumulasi di bawah adalah penjumlahan PnL periode independen, bukan simulasi satu akun berkelanjutan.

## 10. Perbandingan September 2026

Hari yang lolos filter volatilitas: **16 hari**.

| TP / SL | Trade tertutup | Win | Loss | Win rate | Gross PnL | Fee | Funding | Net PnL |
|---|---:|---:|---:|---:|---:|---:|---:|---:|
| 3% / 2% | 32 | 16 | 16 | 50.0% | +16.00 | -3.20 | -0.05 | **+12.75** |
| 5% / 3% | 15 | 8 | 7 | 53.3% | +19.00 | -1.50 | -0.23 | **+17.27** |
| 7% / 5% | 9 | 6 | 3 | 66.7% | +27.00 | -0.90 | -0.08 | **+26.02** |
| 10% / 7% | 5 | 3 | 2 | 60.0% | +16.00 | -0.50 | +0.31 | **+15.81** |

Posisi yang masih terbuka pada akhir periode dan tidak masuk net PnL:
- TP/SL 3%/2%: LONG, entry 0.06664.
- TP/SL 5%/3%: SHORT, entry 0.06859.
- TP/SL 7%/5%: SHORT, entry 0.06791.
- TP/SL 10%/7%: SHORT, entry 0.07080.

## 11. Perbandingan Oktober 2026

Hari yang lolos filter volatilitas: **25 hari**.

| TP / SL | Trade tertutup | Win | Loss | Win rate | Gross PnL | Fee | Funding | Net PnL |
|---|---:|---:|---:|---:|---:|---:|---:|---:|
| 3% / 2% | 82 | 40 | 42 | 48.8% | +36.00 | -8.20 | -1.23 | **+26.57** |
| 5% / 3% | 59 | 25 | 34 | 42.4% | +23.00 | -5.90 | -0.28 | **+16.82** |
| 7% / 5% | 39 | 22 | 17 | 56.4% | +69.00 | -3.90 | -1.90 | **+63.20** |
| 10% / 7% | 25 | 12 | 13 | 48.0% | +29.00 | -2.50 | -5.10 | **+21.40** |

Posisi yang masih terbuka pada akhir periode dan tidak masuk net PnL:
- TP/SL 3%/2%: SHORT, entry 0.06690.
- TP/SL 5%/3%: LONG, entry 0.06849.
- TP/SL 7%/5%: SHORT, entry 0.06986.
- TP/SL 10%/7%: SHORT, entry 0.06986.

## 12. Perbandingan November 1–10, 2026

Hari yang lolos filter volatilitas: **10 hari**. Data 11 November merupakan candle harian parsial dan dikecualikan.

| TP / SL | Trade tertutup | Win | Loss | Win rate | Gross PnL | Fee | Funding | Net PnL | Pause 24 jam |
|---|---:|---:|---:|---:|---:|---:|---:|---:|---:|
| 3% / 2% | 27 | 10 | 17 | 37.0% | -4.00 | -2.70 | +0.32 | **-6.38** | 4 |
| 5% / 3% | 23 | 8 | 15 | 34.8% | -5.00 | -2.30 | +1.11 | **-6.19** | 2 |
| 7% / 5% | 15 | 6 | 9 | 40.0% | -3.00 | -1.50 | +1.27 | **-3.23** | 1 |
| 10% / 7% | 7 | 2 | 5 | 28.6% | -15.00 | -0.70 | -2.02 | **-17.72** | 1 |

Tidak ada posisi terbuka pada cutoff 10 November.

Hari eligible dan volatilitas November:
| Tanggal UTC | Volatilitas |
|---|---:|
| 2026-11-01 | 11.92% |
| 2026-11-02 | 7.95% |
| 2026-11-03 | 10.17% |
| 2026-11-04 | 11.28% |
| 2026-11-05 | 12.81% |
| 2026-11-06 | 11.56% |
| 2026-11-07 | 9.26% |
| 2026-11-08 | 13.20% |
| 2026-11-09 | 10.01% |
| 2026-11-10 | 18.84% |

## 13. Akumulasi PnL September–November 10

Penjumlahan net PnL dari tiga pengujian bulanan independen:

| TP / SL | September | Oktober | November 1–10 | Total net PnL |
|---|---:|---:|---:|---:|
| 3% / 2% | +12.75 | +26.57 | -6.38 | **+32.94 USDT** |
| 5% / 3% | +17.27 | +16.82 | -6.19 | **+27.90 USDT** |
| 7% / 5% | +26.02 | +63.20 | -3.23 | **+85.99 USDT** |
| 10% / 7% | +15.81 | +21.40 | -17.72 | **+19.49 USDT** |

Catatan interpretasi:
- Pada hasil historis ini, konfigurasi TP 7% / SL 5% memiliki akumulasi net PnL tertinggi, yaitu +85.99 USDT.
- November 1–10 negatif untuk keempat konfigurasi.
- Total di atas bukan return akun yang telah dikompaun dan tidak memasukkan floating PnL posisi akhir September/Oktober.
- Tidak boleh dianggap sebagai bukti bahwa TP 7% / SL 5% akan menghasilkan profit di masa depan.

## 14. Daftar Hari Eligible September dan Oktober

Hasil arsip sebelumnya mencatat jumlah hari eligible, tetapi daftar volatilitas per hari untuk dua bulan ini belum disalin ke laporan. Jangan membuat atau mengisi angka yang belum tersedia. Data tersebut dapat ditambahkan setelah diambil ulang dari candle harian historis.

| Periode | Hari eligible |
|---|---:|
| September 2026 | 16 |
| Oktober 2026 | 25 |
| November 1–10, 2026 | 10 |

## 15. Audit Wajib Sebelum Menggunakan Hasil September–November

**Status bagian ini: LEGACY_PROVISIONAL / INCOMPLETE_DATA.**

Saat meninjau ulang implementasi simulasi terdahulu, ditemukan risiko bahwa setelah posisi ditutup oleh TP/SL, loop dapat mengevaluasi sinyal entry pada candle 5M yang sama. Perilaku ini berpotensi membuat entry ulang terlalu cepat dan menggelembungkan jumlah trade, terutama pada konfigurasi TP/SL yang sempit. Hasil Oktober dengan 82 trade pada TP 3%/SL 2% perlu diperiksa secara khusus.

Sebelum angka-angka ini dipakai untuk memilih parameter final:
1. Perbaiki dan audit aturan exit agar tidak membuka entry baru pada candle yang sama dengan candle exit.
2. Terapkan reset PSAR setelah exit sesuai arah posisi dan wajibkan fresh cross baru.
3. Validasi urutan TP/SL menggunakan candle 1M; gunakan SL-first bila kedua level tersentuh dalam candle 1M yang sama.
4. Jalankan ulang September, Oktober, dan November menggunakan implementasi yang sama dan konfigurasi yang sama.
5. Untuk hasil lintas bulan yang berkelanjutan, bawa posisi terbuka melintasi batas bulan. Jangan memulai setiap bulan dari flat jika ingin menyebutnya sebagai simulasi akun kontinu.
6. Hitung ulang drawdown, profit factor, loss streak, pause, unrealized PnL, fee, funding, dan slippage.
7. Uji ulang filter volatilitas tanpa look-ahead bias. Filter high-low harian penuh hanya boleh dipakai sebagai eksperimen historis, bukan sinyal yang tersedia pada awal hari.

Jangan menggabungkan hasil lama dan hasil audit ulang dalam satu tabel tanpa label versi. Pertahankan hasil lama sebagai arsip dan beri label jelas jika hasil telah dihitung ulang.
