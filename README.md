# Prediksi Churn Pelanggan Telekomunikasi (UAS Predictive Analytics, PA-1A)

Project akhir mata kuliah *Predictive Analytics*, Universitas Cakrawala.

**Titanio Yudista** · NIM 24120500031 · Kelas A · Dataset PA-1A Telco Customer Churn (IBM)

## Masalah bisnis

Sekitar satu dari empat pelanggan berhenti berlangganan (churn 26,5%). Tim retensi hanya sanggup menghubungi 20% pelanggan setiap siklus kampanye. Model di project ini memberi skor risiko churn untuk setiap pelanggan agar kapasitas yang terbatas itu dipakai untuk pelanggan yang paling mungkin pergi.

## Hasil utama

Model final adalah **LightGBM (tuned)**, dengan Logistic Regression (tuned) sebagai challenger. Angka di bawah berasal dari data uji (1.409 pelanggan) yang baru dibuka setelah model dan threshold dikunci.

| Metrik | Nilai |
|---|---|
| PR-AUC / ROC-AUC | 0,6642 / 0,8577 |
| Precision / Recall / F1 di top-20% | 0,677 / 0,511 / 0,582 |
| Lift vs acak di top-10% / 20% / 30% | 2,83× / 2,55× / 2,31× |
| Net saved revenue kampanye top-20% | USD 229.338 per siklus (basis 7.043 pelanggan) |
| Selisih vs rule kontrak / vs pemilihan acak | +USD 125.657 / +USD 177.972 |

Faktor yang paling menentukan risiko adalah jenis layanan internet, jenis kontrak, dan lama berlangganan. Nilai bisnis dihitung dari asumsi yang ditulis di sel `BUSINESS_ASSUMPTIONS` (success rate 30%, horizon 12 bulan, biaya penawaran USD 35,15 per kontak). Tabel sensitivitas di notebook menunjukkan kampanye tetap untung walaupun biaya penawaran naik empat kali.

## Isi repository

```
data/1A_telco_customer_churn.csv          dataset resmi PA-1A (7.043 baris, 21 kolom)
notebook/uas_pa_telco_churn.ipynb         notebook analisis lengkap, sudah dieksekusi
submission/
  UAS_PA_A_24120500031_Titanio_PA-1A_Notebook.ipynb
  UAS_PA_A_24120500031_Titanio_PA-1A_Notebook.html
  UAS_PA_A_24120500031_Titanio_PA-1A_Deck.pdf   project deck (15 slide + lampiran)
requirements.txt                          versi library yang dipakai
```

Notebook dibagi menjadi 22 bagian: business understanding, data quality, split train/test sebelum EDA, EDA, feature engineering, perbandingan model, tuning, threshold berbasis kapasitas, evaluasi akhir, error analysis, interpretasi, rekomendasi bisnis, sensitivitas, log eksperimen (termasuk yang gagal), skalabilitas, keterbatasan, dan ringkasan angka kunci.

## Menjalankan ulang

Butuh Python 3.12.

```bash
python -m venv .venv
.venv/bin/pip install -r requirements.txt
cd notebook
../.venv/bin/jupyter nbconvert --to notebook --execute --inplace --ExecutePreprocessor.timeout=1800 uas_pa_telco_churn.ipynb
```

Eksekusi penuh memakan waktu sekitar 3 menit. Notebook harus dijalankan dari folder `notebook/` karena path data ditulis `../data/...`. Grafik disimpan ke `figures/`, yang dibuat otomatis.

Hasilnya reproducible: `RANDOM_STATE = 42` dipakai di semua split, model, dan search, dan LightGBM dijalankan dengan `deterministic=True`. Dengan versi library di `requirements.txt`, semua angka model dan bisnis keluar sama di setiap run. Yang berubah hanya angka waktu (waktu fit dan estimasi waktu scoring), karena bergantung pada beban CPU.

## Metodologi singkat

- **Split:** stratified group 80/20. Baris dengan profil identik masuk ke grup yang sama supaya tidak terpecah ke train dan test.
- **Validasi:** CV 5-fold berbasis grup pada data train. Skor model yang di-tuning diambil dari nested CV.
- **Anti-leakage:** EDA hanya memakai data train. Semua transformasi yang di-fit ada di dalam `sklearn.Pipeline`.
- **Aturan operasional:** hubungi 20% pelanggan dengan skor tertinggi (top-k), sesuai kapasitas tim retensi.

## Catatan penggunaan AI

Bantuan AI (Claude Code) dipakai untuk menyusun kode, merapikan teks, dan melakukan review. Rinciannya, beserta cara verifikasi setiap bagian, ada di slide lampiran A1 pada deck.
