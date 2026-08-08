# UNI-Based Deep Learning Framework for Automated Urine Cytology Analysis and Paris System Classification

Internship project, Computer Science and Engineering, NIT Calicut
Author: Anu Thomson · Supervisor: Dr. Jayaraj P B

## Overview

This repository contains the implementation for an automated urine cytology
analysis pipeline. The system detects and localizes individual cells in urine
cytology microscopy images using an anchor-free CenterNet detector built on a
pathology-pretrained UNI (ViT-Large) backbone, and extends detection into
nucleus/cytoplasm segmentation and N/C-ratio-based classification under the
Paris System for Reporting Urinary Cytology (NHGUC / AUC / SHGUC).

Full methodology, experiments, and results are documented in `docs/report.pdf`
(see the "Report" section below).

---

##  Repository Structure

```text
uni-cytology-detect/
│
├── docs/
│   └── report.pdf             # Project report and technical documentation
│
└── notebooks/
    ├── cell_segmentation.ipynb    # Cell segmentation and preprocessing pipelines
    ├── centernet_300.ipynb        # Baseline CenterNet architecture 
    ├── centernet_uni.ipynb        # CenterNet model adapted with UNI backbone
    ├── rcnn_300.ipynb             # Baseline R-CNN pipeline
    ├── uni_corner_net.ipynb       # CornerNet implementation utilizing UNI
    ├── uni_fcos.ipynb             # FCOS detector using UNI backbone
    └── uni_rcnn.ipynb             # Faster R-CNN pipeline powered by UNI

## Models Implemented

| Notebook | Architecture | Role |
|---|---|---|
| `centernet_uni.ipynb` | CenterNet + UNI ViT-Large | **Primary/production model** |
| `uni_rcnn.ipynb` | Faster R-CNN + UNI ViT-Large | Anchor-based baseline comparison |
| `uni_fcos.ipynb` | FCOS + UNI ViT-Large | Anchor-free comparison (centerness-based) |
| `uni_corner_net.ipynb` | CornerNet + UNI ViT-Large + FPN | Anchor-free comparison (corner-pairing) |
| `cell_segmentation.ipynb` | Multi-Otsu + watershed | Downstream N/C ratio & Paris System classification |

All detectors share the same UNI ViT-Large backbone (DINOv2-pretrained on
~100,000 histopathology whole-slide images), so results across notebooks are
comparable at the level of detection head/loss design rather than feature
extraction.

## Key Results (validation set, corrected evaluation)

| Model | AP@0.5 | Best F1 |
|---|---|---|
| **CenterNet-UNI (primary, fine-tuned)** | **0.8755** | **0.8339** |
| CenterNet-UNI (FPN neck variant) | 0.8650 | 0.8296 |
| CenterNet-300 (dataset-partition ablation) | 0.8401 | 0.7838 |
| FCOS-UNI | 0.826 | 0.758 |
| Faster R-CNN | 0.824 | 0.776 |
| CornerNet-UNI | 0.702 | 0.637 |

The primary CenterNet-UNI model met the project's AP@0.5 target (>0.85) after
an evaluation implementation bug was identified and corrected. Precision/recall
targets (>0.90 each) were not simultaneously met at any single operating
threshold — see `docs/report.pdf`, Chapter 5–6, for the full discussion and
future work.

## Dataset

Two dataset organizations, both derived from the same underlying urine cytology cell annotations, were used across experiments:

**mcell-detection dataset** (primary — official train/validation split)

| Split | Images | Description |
|---|---|---|
| Train | 2,469 | Augmented training set |
| Validation | 332 | Model selection, threshold tuning |
| Test | 332 | Held-out, evaluated once |

**COCO-style dataset** (secondary — used only for dataset-partition ablation experiments, e.g. `CenterNet-300`, `Faster R-CNN-300`)

| Split | Images | GT Boxes | Zero-Ann | Multi-Ann |
|---|---|---|---|---|
| Train | 245 | 1,232 | 96 | 127 |
| Validation | 21 | 115 | 8 | 13 |
| Test | 22 | 93 | 9 | 12 |
| **Total** | **288** | **1,440** | **113** | **152** |

On-the-fly augmentation (random horizontal/vertical flip, brightness/contrast/saturation/hue jitter) is applied to the training set only. Full dataset design and rationale for the two-split setup are in `docs/report.pdf`, Section 3.3 and Chapter 4.

## Report

The complete write-up — background, design, implementation details for each
architecture, full experimental results, and conclusions — is in:

```
docs/report.pdf
```

