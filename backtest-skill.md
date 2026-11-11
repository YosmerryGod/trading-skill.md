---
name: parabolic-sar-follow-trend-backtest
description: Reproducibly backtest the Parabolic SAR Follow Trend strategy on Binance USD-M perpetual futures with completed 1H trend candles, 5M close-cross signals, market-at-close fill assumptions, 1M exits, realistic costs, account compounding, cooldowns, and volatility-filter experiments.
---

# Backtest Skill — Parabolic SAR Follow Trend

## Tujuan dan sumber kebenaran

Mengukur performa strategi Parabolic SAR Follow Trend secara kronologis dan dapat diaudit. Laporkan hasil baik, buruk, nol trade, data tidak lengkap, liquidation, dan keterbatasan simulasi. Backtest tidak membuktikan profit masa depan.

- Aturan strategi, data, dan perhitungan teknis dalam dokumen ini adalah acuan teknis.
- Format respons diatur oleh `respon-ai.md`.
- Jangan mengubah aturan secara diam-diam.
- Jangan mengarang data atau mengklaim backtest sudah dijalankan jika baru membuat rencana.

## 1. Instrumen dan data

- Gunakan simbol Binance USDⓈ-M perpetual yang diminta pengguna. Jangan memilih simbol sendiri.
- Jangan mencampur data spot dan perpetual.
- Sumber utama: data resmi Binance Futures.
- Gunakan data candle 1M, 5M, 1H, historical funding rates, leverage brackets/maintenance margin, exchange filters, dan fee information yang tersedia.
- Simpan seluruh timestamp internal dalam UTC.
- Catat simbol, contract type, exchange, waktu pengambilan data, periode evaluasi, endpoint/sumber, jumlah candle, gap, duplikasi, dan candle tidak lengkap.
- Ambil data warm-up sebelum periode evaluasi untuk PSAR dan volatilitas.
- Jika data penting tidak tersedia, tandai dan jangan mengarang nilainya.

## 2. PSAR dan filter tren

### Parameter PSAR
- Step: 0.01.
- Maximum: 0.10.
- Gunakan implementasi PSAR yang sama untuk semua pengujian.
- Catat formula/library dan versinya, warm-up, initial trend, reversal handling, dan status validasi terhadap referensi.
- Hanya gunakan candle yang telah selesai.

### Filter tren 1H
Pada waktu sinyal 5M, gunakan candle 1H terakhir yang sudah selesai:
- LONG trend jika close 1H > PSAR 1H.
- SHORT trend jika close 1H < PSAR 1H.
- Jika sama atau data tidak tersedia, tidak ada sinyal.
- Candle 1H yang masih berjalan tidak boleh digunakan, walaupun hampir selesai.
- Catat timestamp candle 1H yang digunakan. Candle completed terakhir boleh berusia 45 menit atau lebih; jangan menggantinya dengan candle yang belum close.

## 3. Sinyal entry 5M

Sinyal hanya valid setelah candle 5M selesai. Kriteria adalah **close candle menyeberang PSAR**, bukan seluruh body candle harus berada di sisi lain PSAR.

### LONG
1. Filter tren 1H bullish.
2. Close candle 5M sebelumnya <= nilai PSAR pada candle sebelumnya.
3. Close candle 5M sinyal > nilai PSAR terbaru pada candle sinyal.
4. Ini harus cross baru dari bawah/di PSAR ke atas PSAR. Candle lanjutan yang tetap close di atas PSAR bukan sinyal baru.

### SHORT
1. Filter tren 1H bearish.
2. Close candle 5M sebelumnya >= nilai PSAR pada candle sebelumnya.
3. Close candle 5M sinyal < nilai PSAR terbaru pada candle sinyal.
4. Ini harus cross baru dari atas/di PSAR ke bawah PSAR. Candle lanjutan yang tetap close di bawah PSAR bukan sinyal baru.

Sinyal harus searah filter tren 1H. LONG dan SHORT sama-sama aktif.

### Entry execution
- Asumsi eksekusi yang dipilih pengguna: market order segera setelah sinyal 5M terkonfirmasi pada candle close.
- Catat close sinyal 5M dan harga fill simulasi secara terpisah.
- Harga close sinyal bukan otomatis harga fill yang terjamin. Modelkan slippage; jika data bid/ask atau trade yang sesuai tersedia, gunakan data tersebut. Jika tidak, gunakan asumsi slippage 0.05% per sisi dan labeli sebagai estimasi.
- Jangan memakai data yang belum tersedia pada saat entry.
- Jika implementasi hanya memiliki OHLC dan tidak memiliki quote/tick tepat setelah close, nyatakan keterbatasan fill dengan jelas; jangan mengklaim fill persis di close.
- Satu posisi terbuka maksimum. Jangan membuka posisi baru ketika posisi masih aktif.

## 4. Re-entry, posisi aktif, dan perubahan tren

- Setelah posisi ditutup, arah yang sama tidak boleh entry lagi sampai PSAR reset: LONG perlu setidaknya satu completed 5M close <= PSAR lalu fresh close cross ke atas; SHORT perlu close >= PSAR lalu fresh close cross ke bawah.
- Aturan reset ini berlaku setelah TP, SL, atau liquidation.
- Ketika posisi aktif, perubahan tren 1H tidak menutup posisi. Posisi tetap dikelola sampai TP, SL, liquidation, atau akhir data.
- Flip PSAR 5M saat posisi aktif diabaikan sebagai exit signal; posisi tetap berjalan sampai exit yang ditentukan.
- Jika posisi lama ditutup pada candle 5M yang sama dengan sinyal valid arah sebaliknya, tutup posisi lama lebih dahulu, kemudian evaluasi sinyal baru setelah posisi kosong. Sinyal baru tetap harus memenuhi filter tren dan cross baru; jangan membuat dua posisi bersamaan.
- Sinyal yang muncul selama cooldown dilewati dan tidak boleh digunakan sebagai sinyal baru setelah cooldown selesai.

## 5. TP/SL baseline dan dinamis

### Baseline fixed TP/SL
Jalankan masing-masing konfigurasi secara independen:
| Konfigurasi | TP | SL |
|---|---:|---:|
| A | 3% | 2% |
| B | 5% | 3% |
| C | 7% | 5% |
| D | 10% | 7% |

Persentase adalah pergerakan harga underlying dari entry, bukan return margin atau ROI leverage.

LONG:
- TP = entry × (1 + TP%).
- SL = entry × (1 - SL%).

SHORT:
- TP = entry × (1 - TP%).
- SL = entry × (1 + SL%).

TP dan SL dikunci pada saat entry. Jangan menggeser level saat posisi terbuka.

### Dynamic ATR — eksperimen tambahan
Bukan bagian baseline awal. Jalankan hanya jika diminta setelah baseline:
- ATR timeframe 5M, period 14, Wilder's RMA.
- Gunakan nilai ATR dari candle 5M terakhir yang sudah selesai sebelum entry.
- SL multiplier: 1.0, 1.5, 2.0, 2.5.
- TP Risk/Reward: 1:1, 1:1.5, 1:2, 1:3.
- Total 16 kombinasi.
- Tidak ada maximum TP/SL cap.
- Level TP/SL ditetapkan saat entry dan tidak berubah selama posisi aktif.
- Simpan hasil setiap kombinasi terpisah; jangan hanya menyimpan kombinasi yang profit.

## 6. Account, sizing, dan compounding

Asumsi akun baseline yang disetujui:
- Starting balance: 100 USDT.
- Leverage: 3x.
- Margin mode: Cross Margin.
- Gunakan 100% available account balance sebagai initial margin per trade.
- Initial notional pada saldo 100 USDT: 300 USDT.
- Compounding: ukuran posisi berikutnya dihitung dari saldo akun terbaru.
- Maksimal satu posisi terbuka.
- Sizing aktual tetap harus mematuhi quantity step, tick size, minimum notional, margin availability, dan exchange filters.
- Jika saldo/margin/quantity tidak cukup untuk memenuhi aturan minimum order Binance, hentikan simulasi pada waktu tersebut dan tandai `INSUFFICIENT_BALANCE`. Jangan mengecilkan order diam-diam.
- Catat saldo sebelum trade, margin, notional, quantity, leverage, realized PnL, dan saldo setelah trade.
- Jangan menganggap leverage mengubah persentase pergerakan harga TP/SL.

## 7. Fees, slippage, funding, dan liquidation

### Fee
- Gunakan taker fee 0.05% per sisi sebagai asumsi pengguna untuk baseline jika fee akun aktual tidak tersedia.
- Fee ini adalah asumsi model, bukan klaim bahwa akun Binance pengguna memiliki fee resmi tersebut.
- Hitung fee entry dan exit atas notional yang relevan. Catat fee terpisah dan kurangi dari PnL.

### Slippage
- Prioritaskan historical bid/ask atau data fill yang tersedia.
- Jika tidak tersedia, asumsi 0.05% per sisi.
- Tandai nilai estimasi dan jangan campur dengan data aktual tanpa label.

### Funding
- Gunakan historical Binance funding rate yang sesuai dengan waktu posisi terbuka.
- Terapkan tanda cashflow yang benar untuk LONG/SHORT dan waktu funding aktual.
- Jika funding data tidak tersedia, tandai hasil funding incomplete; jangan mengarang.
- Catat funding setiap trade.

### Liquidation
- Gunakan leverage brackets, maintenance margin, dan aturan liquidation Binance yang berlaku untuk simbol dan ukuran posisi pada waktu simulasi.
- Pertimbangkan Cross Margin dan equity akun, bukan hanya isolated-position formula.
- Jika liquidation terjadi sebelum TP/SL, exit reason = `LIQUIDATION`, bukan SL.
- Terapkan liquidation fee bila relevan dan datanya tersedia.
- Jika aturan historis atau data bracket tidak dapat dipastikan, tandai liquidation modeling incomplete dan jangan menyatakan hasil net final sepenuhnya tervalidasi.

## 8. Cooldown dan loss streak

- Setelah 3 loss berturut-turut, hentikan entry baru selama 24 jam sejak posisi ketiga yang loss selesai.
- Win mereset loss streak menjadi nol.
- Cooldown berlaku untuk setiap simulasi TP/SL dan mode volatilitas secara independen.
- Posisi yang sudah terbuka tetap dikelola sampai exit; cooldown tidak menutup posisi aktif.
- Setelah 24 jam berakhir, entry baru boleh dilakukan segera jika ada sinyal fresh yang valid setelah cooldown. Sinyal yang muncul selama cooldown tidak boleh dipakai.
- Cooldown tidak dengan sendirinya memaksa reset PSAR tambahan. Namun aturan reset PSAR setelah posisi ditutup tetap berlaku.
- Catat loss streak, cooldown start/end, sinyal yang dilewati, dan total jam cooldown.

## 9. Resolusi exit 1M

Gunakan candle 1M secara kronologis untuk memeriksa TP/SL setelah entry.
- Jika hanya TP tersentuh, exit TP.
- Jika hanya SL tersentuh, exit SL.
- Jika TP dan SL sama-sama tersentuh dalam candle 1M yang sama, asumsikan SL terjadi lebih dahulu dan beri label `AMBIGUOUS_1M_SL_FIRST`. Ini asumsi konservatif, bukan fakta urutan tick.
- Jika terjadi gap melewati level, jangan mengasumsikan fill pada level yang tidak dapat dieksekusi; gunakan harga open yang relevan dan catat slippage/gap.
- Jika candle 1M hilang ketika posisi terbuka dan urutan exit tidak dapat dipastikan, tandai `DATA_GAP`; jangan menebak.
- Posisi yang masih terbuka pada akhir periode diberi label `OPEN_AT_END`, bukan TP/SL.
- Catat timestamp candle 1M yang menentukan exit, entry/exit price, TP/SL levels, exit reason, bars held, dan quality flags.

## 10. Volatilitas aktual

Volatilitas adalah nilai numerik yang dihitung pada saat sinyal, bukan label kategori tetap.

Formula:
`volatility_24h_pct = (highest_high_288 - lowest_low_288) / first_open_288 × 100`

- Timeframe: 5M.
- Lookback: 288 completed candles (24 jam).
- Gunakan 288 candle 5M yang sudah selesai sebelum waktu sinyal. Tidak boleh ada future leakage.
- Catat angka persentase aktual pada setiap sinyal.
- Jika data belum cukup, tandai `VOLATILITY_UNAVAILABLE`.

Untuk breakdown gunakan interval 5 percentage points: `[0,5)`, `[5,10)`, `[10,15)`, `[15,20)`, dan seterusnya sampai mencakup volatilitas tertinggi yang ditemukan. Batas bawah termasuk, batas atas tidak termasuk. Contoh nilai 5.00% masuk ke 5–10%, bukan 0–5%. Jangan ubah angka menjadi V1/V2/V3/V4.

### Mode volatilitas
Untuk setiap TP/SL dan setiap rentang volatilitas, jalankan tiga simulasi independen:

1. `ALL TRADING`: tidak menggunakan filter volatilitas; semua sinyal valid boleh masuk. Hasil baseline ini ditampilkan sebagai pembanding pada setiap rentang.
2. `HANYA TRADING (ONLY)`: hanya entry jika volatilitas saat sinyal masuk ke rentang yang sedang diuji.
3. `TIDAK TRADING (EXCLUDE)`: lewati entry jika volatilitas saat sinyal masuk ke rentang yang sedang diuji; rentang lain tetap boleh trading.

Setiap simulasi dimulai dari saldo awal yang sama dan menjalankan ulang urutan trade, compounding, one-position rule, cooldown, fee, slippage, funding, dan liquidation. Jangan menghitung ONLY/EXCLUDE dengan mengurangi atau membagi PnL ALL.

Posisi yang sudah terbuka tetap dikelola meskipun volatilitas berpindah rentang setelah entry. Sinyal dengan volatility unavailable tetap boleh masuk ALL jika data strategi lengkap; untuk ONLY/EXCLUDE jangan masukkan ke rentang buatan, dan laporkan terpisah.

## 11. Periode backtest

Baseline wajib:
- 1 hari penuh yang sudah selesai.
- 7 hari penuh yang sudah selesai.
- 30 hari penuh yang sudah selesai.

Semua periode menggunakan UTC, mengecualikan candle belum selesai, dan mengambil warm-up sebelum awal periode.
- Sertakan timestamp awal/akhir yang tepat.
- Breakdown harian 7/30 hari menggunakan tanggal kalender UTC.
- Hari tanpa trade tetap dicatat dengan nol trade.
- Jika pengguna meminta satu tanggal/hari spesifik, tampilkan breakdown untuk hari tersebut saja.
- Jangan menyebut hasil 1/7/30 hari telah diuji jika data belum lengkap.

## 12. Metrik dan klasifikasi

Hitung untuk setiap konfigurasi, periode, rentang volatilitas, dan mode:
- Net PnL USDT setelah biaya yang berhasil dimodelkan.
- Jumlah trade.
- Win rate.
- TP count, SL count, liquidation count.
- Ambiguous 1M, data-gap, dan open-at-end counts.
- Final balance dan max drawdown untuk ringkasan periode.
- Profit factor, expectancy, average/median trade, max loss streak, cooldown count/duration sebagai audit tambahan.
- LONG dan SHORT breakdown tersedia di trade log dan dapat ditampilkan jika diminta.

Win rate = TP count / (TP count + SL count) × 100. Liquidation, ambiguous, data-gap, dan open-at-end tidak boleh diam-diam dihitung sebagai win atau loss; tampilkan jumlahnya terpisah.

PnL:
- LONG gross return = (exit - entry) / entry.
- SHORT gross return = (entry - exit) / entry.
- Gross PnL = gross return × notional.
- Net PnL = gross PnL - fees - slippage costs + funding cashflow, dengan liquidation adjustment bila berlaku.

Jangan menyebut gross sebagai net. Jika salah satu komponen biaya tidak tersedia, beri status incomplete dan jelaskan dampaknya.

## 13. Format respons

Ikuti `respon-ai.md`. Respons standar hanya memiliki dua tabel utama:

1. Ringkasan periode 1, 7, dan 30 hari, dengan PnL, trade count, dan win rate untuk setiap fixed TP/SL.
2. Breakdown harian berdasarkan interval volatilitas aktual 5%, dan mode ALL TRADING / HANYA TRADING / TIDAK TRADING untuk setiap TP/SL.

Kolom ATR dinamis hanya ditambahkan setelah eksperimen dinamis diminta dan dijalankan. Jangan membuat tabel tambahan tanpa diminta. Detail trade log dan skip log disimpan untuk audit dan dapat ditampilkan bila pengguna meminta.

## 14. Data quality dan anti-bias checklist

Sebelum status selesai:
1. Pastikan 1M, 5M, dan 1H berasal dari simbol/kontrak yang sama.
2. Pastikan timestamp terurut, tidak duplikat, dan gap ditandai.
3. Pastikan filter tren hanya memakai 1H completed candle.
4. Pastikan sinyal hanya diketahui setelah 5M close.
5. Pastikan volatilitas hanya memakai 288 completed 5M candle sebelum sinyal.
6. Pastikan entry fill/slippage tidak menggunakan harga yang tidak dapat dieksekusi.
7. Pastikan exit diuji pada 1M dan ambiguous SL-first diberi label.
8. Pastikan re-entry reset, one-position rule, opposite signal handling, dan cooldown benar.
9. Pastikan fees, funding, exchange filters, dan liquidation tidak diabaikan tanpa catatan.
10. Audit sampel trade secara manual dan jalankan ulang konfigurasi untuk memastikan hasil reproduktif.
11. Simpan trade log dan skipped-signal log.
12. Jangan menghapus hasil buruk atau baris hari tanpa trade.

Status:
- `NOT_RUN`
- `INCOMPLETE_DATA`
- `RUN_COMPLETE_GROSS`
- `RUN_COMPLETE_NET`
- `INCONCLUSIVE`

Kesimpulan strategi:
- `PROMISING_FOR_FURTHER_TEST`
- `WEAK`
- `REJECTED`
- `INCONCLUSIVE`

Jangan menyebut strategi `PROFITABLE` atau `ROBUST` hanya dari hasil 1, 7, atau 30 hari. Hasil tersebut hanya indikasi awal dan membutuhkan periode lebih panjang serta out-of-sample validation.

## 15. Riwayat hasil

Setelah backtest benar-benar dijalankan, simpan atau perbarui `history-backtest/{SYMBOL}.md` sesuai `history-backtest/README.md`. Simpan konfigurasi, timestamp UTC, dua tabel respons, data quality, dan referensi trade/skip logs. Jangan membuat riwayat hasil fiktif.
