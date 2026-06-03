# SSL for Histopathological Image Analysis

> A data-efficient, two-stage framework for breast cancer histopathology combining **GAN-based data synthesis** and **SimCLR self-supervised learning** with multi-source dataset harmonization.

**Research Intern Project** | Amrita Vishwa Vidyapeetham | Nov 2025 – Jan 2026  
**Status:** Paper under review

---

## Overview

Annotating histopathological images requires expert pathologists, making large labeled datasets expensive and scarce. This project addresses that bottleneck through two complementary strategies:

1. **GAN-based synthesis** — Three GAN architectures (DCGAN, WGAN-GP, CycleGAN) are trained on PatchCamelyon and BreakHis to synthesize realistic tissue patches, augmenting the training pool without additional annotation.
2. **SimCLR self-supervised learning** — A ResNet-18 encoder is pre-trained contrastively on unlabeled images (real + GAN-synthesized) and evaluated via linear probing, achieving strong downstream classification without any labeled pre-training data.

A five-stage **dataset harmonization pipeline** (patch standardization → stain normalization → SSL pre-training → ComBat feature correction → downstream evaluation) is proposed to address the severe domain shift between PCam and BreakHis.

---

## Pipeline

```
PatchCamelyon + BreakHis
        │
        ▼
┌─────────────────────┐
│  GAN-based Synthesis │  DCGAN / WGAN-GP / CycleGAN
│  (evaluated: FID,   │
│   SSIM, MMD)        │
└────────┬────────────┘
         │ 70% real + 30% GAN
         ▼
┌─────────────────────┐
│  Harmonization      │  Patch standardization
│  Pipeline           │  Vahadane stain normalization
│                     │  ComBat feature correction
└────────┬────────────┘
         │
         ▼
┌─────────────────────┐
│  SimCLR Pre-training│  ResNet-18 encoder
│  (NT-Xent loss)     │  τ = 0.07, 512-dim embeddings
└────────┬────────────┘
         │ Frozen encoder
         ▼
┌─────────────────────┐
│  Linear Probe       │  In-domain + cross-dataset eval
│  Evaluation         │  Accuracy, ROC-AUC, F1
└─────────────────────┘
```

---

## Results

### GAN Evaluation — PatchCamelyon

| Model    | FID ↓  | SSIM ↑ | MMD ↓    |
|----------|--------|--------|----------|
| DCGAN    | 70.81  | 0.0580 | 0.002007 |
| WGAN-GP  | 96.71  | 0.0435 | 0.000979 |
| CycleGAN | 173.33 | 0.0327 | 0.087345 |

### GAN Evaluation — BreakHis

| Model    | FID ↓  | SSIM ↑ | MMD ↓  |
|----------|--------|--------|--------|
| DCGAN    | 85.88  | 0.1481 | 0.0099 |
| WGAN-GP  | 100.60 | 0.1240 | 0.0099 |
| CycleGAN | 66.98  | 0.8189 | 0.0104 |

### SSL Linear Probe — In-Domain

| Dataset  | Accuracy | ROC-AUC | Macro F1 |
|----------|----------|---------|----------|
| BreakHis | ~91%     | 0.9245  | ~0.91    |
| PCam     | 85.0%    | 0.9263  | 0.850    |

### SSL Linear Probe — Cross-Dataset Transfer

| Train → Test      | Accuracy | ROC-AUC |
|-------------------|----------|---------|
| PCam → BreakHis   | ~56%     | 0.4410  |
| BreakHis → PCam   | ~45%     | 0.4005  |

Cross-dataset degradation motivates the harmonization pipeline.

---

## Repository Structure

```
SSL-for-Histopathology/
│
├── README.md
├── requirements.txt
├── .gitignore
│
├── notebooks/
│   ├── 01_data_exploration/
│   │   └── camelyon_data_visualization.ipynb
│   │
│   ├── 02_gan_patchcamelyon/
│   │   ├── dcgan_on_patchcam.ipynb
│   │   ├── wgan_gp_on_patchcam.ipynb
│   │   └── cyclegan_on_patchcam.ipynb
│   │
│   └── 03_gan_breakhis/
│       ├── dcgan_on_breakhis.ipynb
│       ├── wgan_gp_on_breakhis.ipynb
│       └── cyclegan_on_breakhis.ipynb
│
└── assets/
    └── pipeline_overview.png
```

> **Note:** SimCLR pre-training and linear probe notebooks will be added upon paper acceptance.

---

## Datasets

| Dataset | Description | Source |
|---------|-------------|--------|
| PatchCamelyon (PCam) | 327,680 lymph node patches, 96×96px, binary labels | [Kaggle](https://www.kaggle.com/competitions/histopathologic-cancer-detection) |
| BreakHis | 7,909 H&E breast tissue images, 4 magnifications, 82 patients | [Kaggle](https://www.kaggle.com/datasets/ambarish/breakhis) |

Datasets are not included in this repository. Download them from the links above and update the `DATA_DIR` / `ROOT_DIR` paths in each notebook accordingly.

---

## Setup

```bash
git clone https://github.com/Si-ra-kri/SSL-for-Histopathology.git
cd SSL-for-Histopathology
pip install -r requirements.txt
```

Then open any notebook in `notebooks/` via Jupyter or upload directly to Kaggle.

---

## Requirements

```
torch>=2.0
torchvision>=0.15
numpy
matplotlib
h5py
Pillow
scipy
scikit-image
```

---

## Key Findings

- **DCGAN** achieves the best distributional fidelity on PCam (FID = 70.81); **CycleGAN** best preserves structure on BreakHis (SSIM = 0.82) due to cycle-consistency.
- **SimCLR** representations converge early — performance at epoch 30 is nearly identical to epoch 65 — reducing the need for long pre-training runs.
- **Domain shift** between PCam and BreakHis is severe; cross-dataset ROC-AUC drops below chance, motivating the Vahadane + ComBat harmonization pipeline.
- SSL features are **highly linearly separable** in-domain, achieving 91% accuracy on BreakHis without any labeled pre-training.

---

## Methods

| Component | Details |
|-----------|---------|
| GAN architectures | DCGAN, WGAN-GP (λ=10), CycleGAN |
| SSL framework | SimCLR with ResNet-18 backbone |
| Loss function | NT-Xent (τ = 0.07) |
| Projection head | 2-layer MLP → 128-dim embeddings |
| Stain normalization | Vahadane (sparse NMF) |
| Feature harmonization | ComBat (empirical Bayes) |
| Evaluation | Linear probe, FID, SSIM, MMD, ROC-AUC |
| Framework | PyTorch |
| Training platform | Kaggle (GPU) |

---

## Citation

If you find this work useful, please consider citing it once published. Preprint/DOI will be updated here upon availability.

---

## Author

**Sivaramakrishnan K**  
B.Tech + M.Tech (Dual Degree) in Engineering Physics — NIT Agartala  
BS in Data Science and Applications — IIT Madras  
[GitHub](https://github.com/Si-ra-kri) · [LinkedIn](https://www.linkedin.com/in/sivaramakrishnan-kannan)
