# Trading Skill

Repository untuk riset dan backtest strategi cryptocurrency futures yang reproducible, dapat diaudit, dan tidak membuat klaim profit tanpa data.

## Dokumentasi

| File | Fungsi |
|---|---|
| [respon-ai.md](respon-ai.md) | Alur respons AI dan format laporan backtest dua tabel |
| [backtest-skill.md](backtest-skill.md) | Aturan strategi, data, simulasi, risiko, biaya, dan eksperimen |
| [history-backtest/README.md](history-backtest/README.md) | Aturan penyimpanan laporan historis per simbol |

## Strategi

Strategi utama: **Parabolic SAR Follow Trend** pada Binance USDⓈ-M perpetual futures.

- Filter tren: 1H, menggunakan candle terakhir yang sudah selesai.
- Entry: fresh PSAR close-cross pada candle 5M yang sudah selesai.
- Eksekusi simulasi: market order segera setelah sinyal 5M terkonfirmasi, dengan slippage dan keterbatasan fill dicatat.
- Arah: LONG dan SHORT.
- PSAR: Step 0.01, Maximum 0.10.
- Maksimal satu posisi terbuka.
- Re-entry arah yang sama memerlukan reset PSAR.
- Cooldown: 24 jam setelah 3 loss berturut-turut.
- Baseline TP/SL: 3%/2%, 5%/3%, 7%/5%, dan 10%/7%.
- Periode baseline: 1, 7, dan 30 hari penuh (UTC).
- Volatilitas: range aktual 24 jam berdasarkan 288 candle 5M completed, dikelompokkan dalam interval 5 percentage points.
- Mode volatilitas: ALL TRADING, HANYA TRADING (ONLY), dan TIDAK TRADING (EXCLUDE).
- TP/SL dinamis berbasis ATR adalah eksperimen tambahan setelah baseline, bukan baseline otomatis.

## Model akun baseline

- Saldo awal: 100 USDT.
- Leverage: 3x.
- Margin: Cross Margin.
- Menggunakan 100% saldo tersedia sebagai margin per trade.
- Compounding berdasarkan saldo terbaru.
- Taker fee 0.05% per sisi sebagai asumsi model jika fee akun aktual tidak tersedia.
- Slippage 0.05% per sisi jika data historis bid/ask tidak tersedia.
- Funding historis dan liquidation Binance dimodelkan jika datanya tersedia.

Rincian dan batasan simulasi wajib mengikuti [backtest-skill.md](backtest-skill.md).

## Format laporan AI

Setiap respons backtest standar hanya menggunakan dua tabel utama:

1. **Ringkasan periode:** hasil 1, 7, dan 30 hari; setiap TP/SL menampilkan Net PnL, jumlah trade, dan win rate.
2. **Breakdown harian:** rentang volatilitas numerik dan mode ALL / ONLY / EXCLUDE, dengan PnL, trade count, dan win rate untuk tiap TP/SL.

Ikuti [respon-ai.md](respon-ai.md). Jika pengguna meminta satu hari tertentu, tampilkan breakdown hari tersebut saja. Detail trade log dan skip log disimpan untuk audit dan ditampilkan jika diminta.

## Disiplin riset

1. Bekukan strategi dan parameter sebelum pengujian.
2. Gunakan hanya informasi yang tersedia pada waktu sinyal; hindari future leakage.
3. Validasi candle, timestamp, gap, dan sumber data.
4. Jangan mengarang harga fill, urutan TP/SL, funding, fee, liquidation, atau statistik.
5. Bedakan gross PnL dan net PnL setelah biaya.
6. Jalankan ulang ONLY dan EXCLUDE sebagai simulasi independen; jangan menghitungnya dari PnL ALL secara aritmetis.
7. Laporkan semua konfigurasi, termasuk yang rugi, data tidak lengkap, dan hari tanpa transaksi.
8. Simpan trade log dan sinyal yang dilewati untuk audit.
9. Setelah backtest benar-benar dijalankan, simpan hasil ke `history-backtest/{SYMBOL}.md` mengikuti [history-backtest/README.md](history-backtest/README.md).
10. Jangan menyatakan strategi profitable atau robust hanya dari hasil 1, 7, atau 30 hari.

## Status backtest

Gunakan salah satu status:
- `NOT_RUN`
- `INCOMPLETE_DATA`
- `RUN_COMPLETE_GROSS`
- `RUN_COMPLETE_NET`
- `INCONCLUSIVE`

## Struktur repository

```text
.
├── README.md
├── respon-ai.md
├── backtest-skill.md
└── history-backtest/
    └── README.md
```

File laporan per simbol dibuat hanya setelah backtest simbol tersebut benar-benar dijalankan.

## Disclaimer

Dokumen ini menjelaskan metodologi riset dan simulasi, bukan jaminan performa atau nasihat keuangan personal. Futures memiliki risiko tinggi, termasuk kehilangan seluruh modal yang dialokasikan.
