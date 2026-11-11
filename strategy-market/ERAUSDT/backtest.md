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
