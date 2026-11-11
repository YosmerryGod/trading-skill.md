# Strategy Skill Backtest

Dokumen ini mendefinisikan strategi **PSAR Follow Trend** dan aturan backtest entry/exit. Aturan klasifikasi atau filter volatilitas belum dimasukkan; volatilitas akan dibahas dan diuji secara terpisah.

## 1. Strategi

- Nama strategi: PSAR Follow Trend.
- Indikator: Parabolic SAR (PSAR).
- PSAR Step: 0.01.
- PSAR Maximum: 0.10.
- Arah posisi: LONG dan SHORT aktif.
- Timeframe tren: 1H.
- Timeframe sinyal entry: 5M.
- Gunakan hanya candle yang sudah selesai/closed.

## 2. Filter Tren 1H

Gunakan candle 1H terakhir yang sudah selesai.

- **LONG diperbolehkan** jika close candle 1H terakhir berada di atas nilai PSAR 1H.
- **SHORT diperbolehkan** jika close candle 1H terakhir berada di bawah nilai PSAR 1H.
- Jangan menggunakan candle 1H yang masih berjalan.
- Jika candle 1H terakhir yang selesai sudah berumur 45 menit, tetap gunakan candle tersebut sampai candle 1H berikutnya selesai.
- Catat timestamp candle 1H yang digunakan sebagai filter tren.

## 3. Sinyal Entry 5M

Sinyal hanya valid setelah candle 5M selesai.

### LONG

1. Candle 5M sebelumnya close di bawah atau sama dengan PSAR sebelumnya.
2. Candle 5M terbaru close di atas PSAR terbaru.
3. Filter tren 1H harus menunjukkan LONG.

### SHORT

1. Candle 5M sebelumnya close di atas atau sama dengan PSAR sebelumnya.
2. Candle 5M terbaru close di bawah PSAR terbaru.
3. Filter tren 1H harus menunjukkan SHORT.

Ketentuan:

- Syarat entry menggunakan **harga close candle**, bukan posisi seluruh body candle.
- Entry hanya pada cross baru (fresh cross).
- Jangan membuka posisi hanya karena harga masih berada di sisi yang sama dari PSAR.
- LONG dan SHORT menggunakan aturan yang simetris.

## 4. Eksekusi Entry

- Gunakan asumsi market order segera setelah candle sinyal 5M selesai.
- Harga entry backtest menggunakan close candle sinyal 5M.
- Catat asumsi ini sebagai harga eksekusi simulasi; harga eksekusi nyata dapat berbeda karena latency, spread, dan slippage.
- Maksimal satu posisi terbuka pada satu waktu.
- Jika posisi lama ditutup dan pada candle 5M yang sama terdapat sinyal valid arah berlawanan, tutup posisi lama terlebih dahulu, lalu evaluasi sinyal baru satu kali setelah posisi sudah flat. Sinyal baru tetap harus memenuhi syarat fresh cross.

## 5. Take Profit dan Stop Loss

Uji empat konfigurasi TP/SL berikut secara terpisah. Jangan menggabungkan hasil antar-konfigurasi.

| Konfigurasi | Take Profit | Stop Loss |
|---|---:|---:|
| A | 3% | 2% |
| B | 5% | 3% |
| C | 7% | 5% |
| D | 10% | 7% |

Ketentuan perhitungan:

- Persentase TP dan SL dihitung dari pergerakan harga aset terhadap harga entry, bukan persentase keuntungan atau kerugian margin setelah leverage.
- LONG: TP berada di atas entry dan SL berada di bawah entry.
- SHORT: TP berada di bawah entry dan SL berada di atas entry.
- Level TP dan SL ditetapkan ketika posisi dibuka dan tidak berubah selama posisi berjalan.
- Posisi ditutup ketika TP atau SL tersentuh.
- Jika TP dan SL sama-sama tersentuh dalam candle 1M yang sama dan urutan sentuh tidak dapat dipastikan, gunakan asumsi konservatif **SL lebih dahulu** dan beri label `AMBIGUOUS_1M_SL_FIRST`.

## 6. Aturan Selama Posisi Terbuka

- Jika tren 1H berubah arah, jangan tutup posisi karena perubahan tersebut. Tetap kelola posisi sampai TP atau SL.
- Jika PSAR 5M berbalik arah saat posisi masih terbuka, abaikan sinyal tersebut. Tetap kelola posisi sampai TP atau SL.
- Jangan membuka posisi tambahan selama masih ada posisi aktif.

## 7. Reset Sinyal Setelah Posisi Ditutup

Setelah posisi ditutup karena TP maupun SL, jangan langsung masuk lagi ke arah yang sama.

- Setelah posisi LONG ditutup, tunggu candle 5M close kembali di bawah atau sama dengan PSAR. Setelah itu, tunggu fresh cross close ke atas PSAR untuk sinyal LONG berikutnya.
- Setelah posisi SHORT ditutup, tunggu candle 5M close kembali di atas atau sama dengan PSAR. Setelah itu, tunggu fresh cross close ke bawah PSAR untuk sinyal SHORT berikutnya.
- Aturan reset berlaku setelah TP maupun SL.

## 8. Batas Kerugian Beruntun dan Jeda Trading

- Hitung loss secara berurutan berdasarkan hasil posisi yang sudah ditutup.
- Setelah **3 kali loss berturut-turut**, hentikan pembukaan posisi baru selama **24 jam**, dihitung sejak waktu penutupan posisi loss ketiga.
- Selama jeda 24 jam, jangan membuka posisi baru, termasuk ketika muncul sinyal valid.
- Posisi yang sudah terbuka sebelum jeda dimulai tetap dikelola sampai TP atau SL.
- Jika trade menghasilkan profit sebelum mencapai 3 loss beruntun, reset hitungan loss beruntun menjadi nol.
- Setelah 24 jam berakhir, entry boleh dilakukan kembali jika muncul sinyal valid yang memenuhi aturan strategi. Tidak perlu reset PSAR tambahan hanya karena jeda berakhir.
- Sinyal yang muncul selama masa jeda diabaikan; jangan digunakan sebagai entry tertunda setelah jeda selesai.
- Terapkan aturan jeda yang sama pada semua konfigurasi TP/SL.

## 9. Ketentuan Backtest

- Jalankan setiap konfigurasi TP/SL secara independen dengan aturan strategi dan data historis yang sama.
- Uji periode 1 hari, 7 hari, dan 30 hari.
- Gunakan candle historis yang sudah selesai dan periode UTC yang lengkap.
- LONG dan SHORT sama-sama aktif.
- Maksimal satu posisi terbuka pada satu waktu.
- Sertakan hari tanpa transaksi dalam rincian harian.
- Catat minimal: jumlah trade, win rate, jumlah win/loss, loss beruntun terpanjang, gross PnL, fee, slippage, funding, net PnL, dan drawdown.
- Jelaskan asumsi data dan eksekusi. Jangan menyebut hasil final tervalidasi jika komponen penting seperti funding, biaya, slippage, atau risiko likuidasi belum dapat dihitung dengan memadai.
- Jika data yang diperlukan tidak tersedia, tandai hasil sebagai `INCOMPLETE_DATA` dan jelaskan bagian yang belum dihitung.

## 10. Di Luar Cakupan Dokumen Ini

Filter, klasifikasi, dan eksperimen volatilitas **belum menjadi bagian dari aturan strategi ini**. Jangan menerapkan filter volatilitas atau mengelompokkan hasil berdasarkan volatilitas secara otomatis. Tambahkan aturan tersebut hanya setelah ada keputusan terpisah.
