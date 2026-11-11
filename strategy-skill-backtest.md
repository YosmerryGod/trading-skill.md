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

## 11. Format Respons Backtest

Setiap hasil backtest yang disampaikan kepada user harus menggunakan tabel yang ringkas, mudah dibandingkan, dan memakai penanda warna secara konsisten. Tampilkan ringkasan terlebih dahulu, lalu detail risiko dan kesimpulan.

### 11.1 Kode warna

Gunakan badge warna di antarmuka yang mendukungnya. Jika hanya Markdown/GitHub yang tersedia, gunakan emoji warna berikut:

- 🟢 **Hijau**: hasil positif, profit, atau trade menang.
- 🔴 **Merah**: hasil negatif, loss, atau drawdown.
- 🟡 **Kuning**: peringatan, loss streak, risiko, atau data belum lengkap.
- 🔵 **Biru**: informasi periode, modal, leverage, dan parameter.
- ⚪ **Abu-abu**: netral, nol, tidak tersedia, atau tidak berlaku.

Warna harus mencerminkan nilai sebenarnya. Jangan memberi warna hijau pada konfigurasi hanya karena nilainya paling baik jika hasilnya masih negatif.

### 11.2 Ringkasan utama

Ringkasan utama harus memberi gambaran performa yang lebih lengkap, tetapi tetap mudah dibaca. Pecah menjadi tiga tabel agar tidak terlalu lebar. Tampilkan empat konfigurasi TP/SL secara berdampingan dan gunakan data backtest aktual. Jangan mengisi angka contoh atau perkiraan seolah-olah hasil nyata.

**A. Aktivitas dan hasil trade**

| TP / SL | Total Trade | LONG | SHORT | Win | Loss | TP Hit | SL Hit | Win Rate |
|---|---:|---:|---:|---:|---:|---:|---:|---:|
| 3% / 2% | — | — | — | — | — | — | — | — |
| 5% / 3% | — | — | — | — | — | — | — | — |
| 7% / 5% | — | — | — | — | — | — | — | — |
| 10% / 7% | — | — | — | — | — | — | — | — |

**B. Profit dan biaya**

| TP / SL | Gross Profit | Gross Loss | Fee | Slippage | Funding | Net PnL | Return |
|---|---:|---:|---:|---:|---:|---:|---:|
| 3% / 2% | — | — | — | — | — | — | — |
| 5% / 3% | — | — | — | — | — | — | — |
| 7% / 5% | — | — | — | — | — | — | — |
| 10% / 7% | — | — | — | — | — | — | — |

**C. Risiko dan kualitas performa**

| TP / SL | Max Drawdown | Loss Streak Terpanjang | Jeda 24 Jam | Profit Factor | Avg. Win | Avg. Loss | Posisi Terbuka di Akhir |
|---|---:|---:|---:|---:|---:|---:|---:|
| 3% / 2% | — | — | — | — | — | — | — |
| 5% / 3% | — | — | — | — | — | — | — |
| 7% / 5% | — | — | — | — | — | — | — |
| 10% / 7% | — | — | — | — | — | — | — |

Definisi dan aturan:
- **LONG / SHORT** adalah jumlah trade berdasarkan arah posisi.
- **TP Hit / SL Hit** adalah jumlah posisi yang ditutup oleh masing-masing target. Jika ada penutupan lain, tampilkan sebagai kategori tersendiri; jangan paksa masuk TP atau SL.
- **Gross Profit** adalah total hasil positif trade sebelum biaya; **Gross Loss** adalah total hasil negatif trade sebelum biaya.
- **Net PnL** harus memperhitungkan fee, slippage, dan funding yang tersedia. Tampilkan komponen biaya secara terpisah.
- **Return** adalah Net PnL dibagi modal awal, dalam persen.
- **Max Drawdown** harus menjelaskan apakah dihitung dari realized equity saja atau termasuk unrealized/floating PnL.
- **Profit Factor** adalah Gross Profit dibagi nilai absolut Gross Loss. Jika Gross Loss nol, tampilkan N/A, bukan angka tak terhingga.
- **Avg. Win / Avg. Loss** adalah rata-rata PnL per trade menang dan per trade kalah sebelum atau sesudah biaya harus disebutkan secara eksplisit. Utamakan nilai net setelah biaya.
- **Jeda 24 Jam** menampilkan berapa kali aturan jeda setelah 3 loss beruntun terpicu.
- **Posisi Terbuka di Akhir** wajib disebutkan karena posisi yang belum ditutup belum menjadi hasil final; tampilkan floating PnL secara terpisah bila tersedia.
- Jika suatu metrik tidak bisa dihitung, tampilkan N/A dan jelaskan alasannya. Jangan mengarang atau menyimpulkan nilai yang tidak tersedia.

Aturan pewarnaan:
- Net PnL dan Return positif: 🟢; negatif: 🔴; nol: ⚪.
- Win dan TP Hit: 🟢; Loss dan SL Hit: 🔴.
- Drawdown, loss streak, dan jeda yang terpicu: 🟡 atau 🔴 sesuai tingkat risikonya.
- Profit Factor di atas 1: 🟢; di bawah 1: 🔴; sama dengan 1: ⚪.
- Jangan memberi warna hijau pada konfigurasi hanya karena nilainya paling baik jika hasilnya masih negatif.
- Win Rate harus dibaca bersama jumlah trade, Net PnL, Profit Factor, dan Max Drawdown.

### 11.3 Detail modal dan risiko

| Metrik | Hasil |
|---|---:|
| Periode backtest | — |
| Modal awal | — |
| Saldo akhir | — |
| Gross PnL | — |
| Total fee | — |
| Total slippage | — |
| Total funding | — |
| Net PnL | — |
| Max drawdown | — |
| Loss streak terpanjang | — |
| Jeda 24 jam terpicu | — |

Gunakan 🟢 untuk saldo akhir atau Net PnL yang bertambah, 🔴 untuk penurunan atau drawdown, 🟡 untuk loss streak dan jeda trading, serta 🔵 untuk informasi parameter. Tampilkan satuan USDT atau persen dengan jelas.

### 11.4 Rincian per konfigurasi dan periode

Jika membandingkan periode 1 hari, 7 hari, dan 30 hari, gunakan tabel terpisah atau satu tabel yang jelas. Jangan mencampur hasil dari periode yang berbeda.

| TP / SL | Periode | Trade | Win Rate | Net PnL | Max DD |
|---|---|---:|---:|---:|---:|
| 3% / 2% | 1 hari | — | — | — | — |
| 3% / 2% | 7 hari | — | — | — | — |
| 3% / 2% | 30 hari | — | — | — | — |

Ulangi baris untuk konfigurasi TP/SL lainnya jika diperlukan. Sertakan rincian harian ketika user meminta atau ketika pola harian penting untuk menjelaskan hasil.

### 11.5 Status kualitas data

Selalu tampilkan status kualitas data:

- 🟢 **DATA LENGKAP**: komponen data dan biaya yang diperlukan tersedia serta dihitung.
- 🟡 **INCOMPLETE_DATA**: ada komponen penting yang belum tersedia atau belum dapat dihitung.
- 🔴 **SIMULASI TIDAK VALID**: data atau proses backtest memiliki kesalahan yang membuat hasil tidak layak digunakan.

Jika status `INCOMPLETE_DATA`, sebutkan dengan jelas komponen yang belum dihitung, misalnya funding, slippage, fee aktual, leverage bracket, maintenance margin, atau likuidasi. Jangan menyajikan hasil simulasi sebagai hasil final tervalidasi.

### 11.6 Kesimpulan AI

Setelah tabel, berikan kesimpulan singkat yang mencakup:

1. Konfigurasi dengan Net PnL tertinggi.
2. Konfigurasi dengan Win Rate tertinggi.
3. Konfigurasi dengan Max Drawdown terendah.
4. Apakah hasil positif atau negatif setelah biaya.
5. Loss streak dan apakah jeda 24 jam terpicu.
6. Keterbatasan data yang dapat memengaruhi kesimpulan.

Jangan menyebut satu konfigurasi sebagai yang terbaik hanya dari Win Rate. Pertimbangkan Net PnL, drawdown, jumlah trade, dan kualitas data secara bersamaan. Jangan menjanjikan profit atau menyatakan strategi pasti berhasil.

