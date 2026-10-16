# Klasifikasi Aktivitas Latihan Otot Punggung (TA Albert)

## Urutan menjalankan

Kernel: **Python (TA Albert)** (`.venv`). Jalankan dari atas ke bawah (*Run All*).

| No | Notebook | Isi | Perkiraan waktu |
|---|---|---|---|
| 1 | `1_preprocessing.ipynb` | Video → keypoints → pembersihan → fitur → split subjek → dataset augmentasi | ± 10 menit (MediaPipe dilewati jika keypoints sudah ada) |
| 2 | `2_training_tanpa_augmentasi.ipynb` | Tuning + training 5 skema dengan data asli | ± 1–1,5 jam |
| 3 | `3_training_dengan_augmentasi.ipynb` | Training 5 skema dengan data asli + augmentasi, lalu perbandingan dengan notebook 2 | ± 1,5–2 jam |
| 4 | `4_perbandingan_model.ipynb` | LSTM vs Bi-LSTM vs ConvLSTM pada Skema C & D (tanpa augmentasi) | ± 45–90 menit |

Notebook 3 dan 4 membutuhkan hasil notebook 2 (`results/tanpa_augmentasi/best_config.csv`).
Progres training dapat dipantau di terminal: `tail -f results/tanpa_augmentasi/progress.log`.

## Struktur folder

```
Tugas Akhir - Albert/
├── 1_preprocessing.ipynb
├── 2_training_tanpa_augmentasi.ipynb
├── 3_training_dengan_augmentasi.ipynb
├── 4_perbandingan_model.ipynb
│
├── raw_dataset/                  video asli: <kelas>/<subjek>/*.mp4 (240 video)
├── models/                       model MediaPipe (pose_landmarker_full.task)
│
├── data/                         dibuat oleh notebook 1
│   ├── metadata.csv              data setiap video (resolusi, FPS, durasi, file kembar)
│   ├── keypoints_raw/            33 titik tubuh per frame hasil MediaPipe
│   ├── keypoints_clean/          setelah interpolasi frame hilang
│   ├── features/                 DATASET TANPA AUGMENTASI
│   │   ├── schemeA/              13 keypoints × (x, y, z, v) per frame
│   │   ├── schemeB/              8 sudut sendi per frame
│   │   └── index.csv             daftar video, label, subjek
│   ├── dataset_augmentasi/       DATASET DENGAN AUGMENTASI
│   │   ├── <kelas>/<video>_asli.csv, _aug1.csv, _aug2.csv
│   │   └── index.csv             daftar file + augmentasi yang diterapkan
│   └── splits.json               pembagian subjek (holdout & LOSO)
│
├── qa/                           bahan pemeriksaan visual (notebook 1)
│   ├── first_frames/             grid frame pertama per kelas
│   ├── missing_report.csv        laporan frame hilang per video
│   ├── overlay/                  video marking keypoints
│   └── augmented/                video kerangka asli vs augmentasi
│
├── results/
│   ├── tanpa_augmentasi/         hasil notebook 2
│   ├── dengan_augmentasi/        hasil notebook 3
│   └── perbandingan_model/       hasil notebook 4 (LSTM, Bi-LSTM, ConvLSTM)
│       (isi umum setiap folder hasil di atas:)
│       ├── best_config.csv       konfigurasi hyperparameter terpilih
│       ├── runs.csv              metrik setiap run (akurasi, presisi, recall, F1)
│       ├── predictions.csv       prediksi setiap video uji (bahan confusion matrix)
│       ├── histories.json        loss & akurasi per epoch (bahan learning curve)
│       ├── baseline.csv          hasil baseline tanpa urutan
│       ├── summary.csv           ringkasan per skema × mode
│       ├── figures/              skema_A.png ... skema_D.png, ringkasan_metrik.png, dll.
│       └── models/               model LSTM holdout (.keras)
│
└── arsip/                        notebook & hasil versi lama (tidak dipakai lagi)
```

## Skema fitur

| Skema | Fitur/frame | Isi |
|---|---|---|
| A | 52 | koordinat mentah (x, y, z, visibility) 13 keypoints |
| A' | 26 | koordinat (x, y) dipusatkan di pinggul, dibagi panjang badan |
| B | 8 | sudut sendi (Angle Directive Rules) |
| C | 16 | perubahan sudut terhadap rata-rata video + kecepatan sudut (gerakan saja) |
| D | 52 | perubahan koordinat A' terhadap rata-rata video + kecepatan (gerakan saja) |
