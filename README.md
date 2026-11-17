# Segmentasi Semantik Polip pada Citra Kolonoskopi (Kvasir-SEG): U-Net dari Nol vs U-Net dengan Encoder ResNet-34 Pretrained

Tugas Kelompok mata kuliah **Computer Vision**. Proyek ini membandingkan dua model segmentasi semantik untuk memisahkan polip dari latar pada citra kolonoskopi, lalu menelaah informasi yang hilang bila masalah yang sama diselesaikan dengan klasifikasi atau deteksi.

## Ringkasan

- **Dataset:** [Kvasir-SEG](https://datasets.simula.no/kvasir-seg/) (1.000 citra kolonoskopi beserta mask biner polip).
- **Model A:** U-Net standar 4 tingkat, dilatih dari nol.
- **Model B:** U-Net dengan encoder ResNet-34 pretrained ImageNet (`segmentation_models_pytorch`).
- **Metrik:** Dice dan IoU per citra pada data uji, lalu dirata-rata.
- **Analisis tambahan:** dari prediksi Model B diturunkan label citra, *bounding box*, mask semantik, dan pewarnaan per objek untuk membahas perbedaan segmentasi, klasifikasi, deteksi, dan *instance*.

## Hasil

Kinerja pada data uji (150 citra):

| Model | Dice uji | IoU uji | Dice validasi (epoch terbaik) | Epoch terbaik |
|---|---|---|---|---|
| A. U-Net dari nol | 0,5216 | 0,3858 | 0,5043 | 12 |
| B. U-Net + ResNet-34 pretrained | 0,8506 | 0,7773 | 0,8615 | 12 |

Kedua model berhenti lewat *early stopping* pada epoch ke-20 (patience 8, maksimum 40 epoch).

## Konfigurasi

| Komponen | Model A | Model B |
|---|---|---|
| Arsitektur | U-Net standar, 4 tingkat encoder–decoder | U-Net, encoder ResNet-34 |
| Inisialisasi | Acak | Bobot pretrained ImageNet |
| Pustaka | PyTorch | PyTorch + `segmentation_models_pytorch` |
| Ukuran input | 256 × 256 piksel | 256 × 256 piksel |
| Loss | Dice loss + BCE (1 : 1) | Sama |
| Optimizer | Adam, learning rate 1e-3, batch 16 | Adam, learning rate 1e-4, batch 16 |
| Augmentasi (hanya data latih) | Flip horizontal, rotasi ±15°, kecerahan/kontras ±0,2 | Sama |
| Penghentian | *Early stopping* pada Dice validasi, patience 8 | Sama |

Pembagian data 70/15/15 (700 latih, 150 validasi, 150 uji) dengan seed 42, dilakukan per kelompok citra mirip (*difference hash*) sebagai pengganti pembagian per pasien karena Kvasir-SEG tidak memiliki ID pasien.

## Struktur Repositori

```
.
├── Tugas_Kelompok_Computer_Vision_v4.ipynb   # notebook utama (jalan dari awal sampai akhir)
├── Laporan_Segmentasi_Polip_Kvasir-SEG.pdf   # laporan (maks. 4 halaman)
└── README.md
```

> Ganti nama berkas di atas bila nama berkas Anda berbeda.

## Cara Menjalankan

Notebook dirancang berjalan dari awal sampai akhir tanpa intervensi manual. Seed acak ditetapkan di sel kode pertama, dan dataset diunduh otomatis lewat `kagglehub`.

1. Buka notebook di Google Colab (disarankan runtime GPU) atau Jupyter lokal dengan GPU CUDA.
2. Jalankan **Run all**. Sel pertama memasang pustaka yang dibutuhkan:
   ```bash
   pip install -q kagglehub segmentation-models-pytorch albumentations==2.0.8 scikit-image
   ```
3. Dataset `debeshjha1/kvasirseg` diunduh otomatis. Di luar Colab, `kagglehub` mungkin memerlukan akun dan kredensial Kaggle.

Keluaran yang dihasilkan notebook: Tabel 1, `Gambar_1_Kurva_Training.png`, `Gambar_2_Terbaik_Terburuk.png`, `Gambar_3_Perbandingan_Tugas.png`, `Gambar_4_Kegagalan_Semantik_vs_Instance.png`, serta bobot terbaik `Model_A_best.pth` dan `Model_B_best.pth`.

## Catatan dan Keterbatasan

- Hasil berasal dari **satu run dengan satu seed** dan satu pembagian data, tanpa uji signifikansi atau pengulangan.
- *Learning rate* kedua model berbeda (1e-3 vs 1e-4), sehingga selisih kinerja tidak sepenuhnya dapat dikaitkan dengan *pretraining* saja.
- Pembagian per kelompok citra mirip adalah pendekatan pengganti dan tidak menjamin bebas kebocoran antar-pasien.
- Hasil di GPU dapat sedikit berbeda antar-run walau seed sudah ditetapkan (jenis GPU, versi pustaka).
- Di data uji, tidak ditemukan mask prediksi yang menyatukan dua objek *ground truth*; kasus objek yang saling bersentuhan belum teruji.

## Referensi

- Jha, D., dkk. (2020). Kvasir-SEG: A segmented polyp dataset. *MultiMedia Modeling (MMM 2020)*.
- Ronneberger, O., Fischer, P., & Brox, T. (2015). U-Net: Convolutional networks for biomedical image segmentation. *MICCAI 2015*.
- He, K., Zhang, X., Ren, S., & Sun, J. (2016). Deep residual learning for image recognition. *CVPR*.
- Deng, J., dkk. (2009). ImageNet: A large-scale hierarchical image database. *CVPR*.

## Kelompok
