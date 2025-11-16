# Benchmarking Deep Learning for Multi-Label Ocular Disease Detection on ODIR-5K

This repository implements and reproduces the full experimental pipeline from:

**“Benchmarking Deep Learning for Multi-Label Ocular Disease Detection on ODIR-5K:  
A Foundation for ADRD Retinal Biomarker Research” — Eric Yang**

It includes dataset preparation, unified preprocessing, model training, evaluation, and ensemble analysis across four major architectures (ResNet-18, EfficientNet-B3, Swin-Tiny, CNN-GNN) plus a post-hoc ensemble.

---

## 1. Overview

Alzheimer’s Disease and Related Dementias (ADRD) can manifest subtle retinal signals due to shared neurovascular and neurodegenerative pathways between retina and brain. Before analyzing ADRD-specific retinal biomarkers, we first benchmark standard deep learning models on **ODIR-5K**, a multilabel ocular disease dataset.

This establishes:
- A **robust baseline** of disease-recognition models.
- Insight into which ocular findings are easy vs difficult.
- A reference point for future **foundation-model–based ADRD** pipelines.

---

## 2. Dataset

**ODIR-5K (Ocular Disease Intelligent Recognition)**  
~5,000 fundus photographs, each labeled with 0–8 disease tags:

1. Normal  
2. Diabetes  
3. Glaucoma  
4. Cataract  
5. Age-related Macular Degeneration (AMD)  
6. Hypertension  
7. Myopia  
8. Other abnormalities  

Text labels are converted into **8-dimensional multi-hot vectors**, e.g.:
["Diabetes"] → [0,1,0,0,0,0,0,0]

Each row of `ODIR-5K-clean.csv` includes:
- `filepath`  
- `labels` (raw text list)  
- `target` (multi-hot vector as Python list)  

---

## 3. Data Preparation Pipeline

### 3.1. Cleaning metadata
- Remove dataset-directory prefixes from Kaggle-style file paths.
- Save unified CSV: `ODIR-5K-clean.csv`.

### 3.2. PyTorch Dataset
The dataset class:
- Loads the fundus image in RGB.
- Converts `target` into a float tensor of shape `[8]`.
- Applies the shared transform.

### 3.3. Preprocessing Transform
Applied identically across all models:

- Resize to **224×224**
- Convert to tensor
- Normalize with ImageNet mean/std

### 3.4. Train / Validation Split
- Random split: **70% train / 30% validation**
- Batch size: **32**

---

## 4. Models

All models output **8 logits** and use **BCEWithLogitsLoss**.

### 4.1. ResNet-18
- Pretrained ImageNet backbone
- Final FC replaced with 8-logit classifier

### 4.2. EfficientNet-B3
- ImageNet pretrained
- Final linear layer replaced with 8-logit classifier

### 4.3. Swin Transformer-Tiny
- Created via `timm`
- Strongest individual model in experiments

### 4.4. CNN + GNN Hybrid
- CNN backbone → image feature vector  
- Label embeddings → graph nodes  
- GCNConv layers propagate disease-relationship structure  
- Outputs 8 logits (1 per disease)

### 4.5. Ensemble (ResNet-18 + EfficientNet-B3 + Swin-Tiny)
- Collect validation predictions for each model
- Perform a 2-D grid search:
  - `w1` = ResNet weight (0 to 1)
  - `w2` = EfficientNet weight (0 to 1)
  - `w3 = 1 - w1 - w2`
- Best macro AUROC achieved at:
  - ResNet-18: **0.2**
  - EfficientNet-B3: **0.1**
  - Swin-Tiny: **0.7**

---

## 5. Training Procedure

### 5.1. Shared Setup
- Loss: **BCEWithLogitsLoss**
- Optimizer: **Adam**, lr = **1e-4**
- Epochs: **5**
- Batch size: **32**

### 5.2. Runtime (approx)
- ResNet-18: ~30 min  
- EfficientNet-B3: ~54 min  
- Swin-Tiny: ~55 min  
- CNN-GNN Hybrid: ~36 min  
- Ensemble (post-hoc weighting): ~3 min  

---

## 6. Metrics

We use multilabel metrics:

- **Macro AUROC**
- **Macro AUPRC**
- **Per-class AUROC**
- **Per-class AUPRC**

macro = unweighted mean across 8 classes (accounts for imbalance)

---

## 7. Results

### 7.1. Macro AUROC by Epoch

| Model | Ep1        | Ep2 | Ep3      | Ep4 | Ep5 |
|------|------------|-----|----------|-----|-----|
| ResNet-18 | 0.8210     | 0.8366 | **0.8420** | 0.8302 | 0.8357 |
| EfficientNet-B3 | 0.7759     | 0.8254 | 0.8440   | 0.8444 | **0.8459** |
| Swin-Tiny | 0.8401     | 0.8614 | 0.8506   | **0.8704** | 0.8660 |
| CNN-GNN | **0.5087** | 0.5030 | 0.4899   | 0.4864 | 0.4934 |
| Ensemble | –          | – | –        | – | **0.8814** |

### 7.2. Best Macro Metrics (per model)

| Model | Macro AUROC | Macro AUPRC |
|-------|-------------|--------------|
| ResNet-18 | 0.8420 | 0.5564 |
| EfficientNet-B3 | 0.8459 | 0.5618 |
| Swin-Tiny | 0.8704 | 0.6026 |
| Ensemble | **0.8814** | **0.6272** |

### 7.3. Per-Class AUROC

| Class | R18 | EB3 | Swin | Ens |
|-------|------|------|------|------|
| Normal | 0.7692 | 0.7895 | 0.7979 | **0.8135** |
| Diabetes | 0.7503 | 0.7777 | 0.7920 | **0.8031** |
| Glaucoma | 0.8674 | 0.8679 | 0.9006 | **0.9220** |
| Cataract | 0.9860 | 0.9721 | 0.9868 | **0.9879** |
| AMD | 0.8747 | 0.8763 | 0.8965 | **0.9166** |
| Hypertension | 0.7429 | 0.7413 | 0.7820 | **0.819** |
| Myopia | 0.9916 | 0.9899 | 0.9913 | **0.9915** |
| Other | 0.7038 | 0.7522 | 0.7806 | **0.7974** |

### 7.4. Per-Class AUPRC

| Class | R18 | EB3 | Swin | Ens |
|--------|------|------|-------|-------|
| Normal | 0.7046 | 0.7339 | 0.7365 | **0.7583** |
| Diabetes | 0.5497 | 0.5920 | 0.6101 | **0.6362** |
| Glaucoma | 0.4130 | 0.4176 | 0.5340 | **0.5440** |
| Cataract | 0.8909 | 0.8812 | 0.8857 | **0.9054** |
| AMD | 0.4857 | 0.4477 | 0.5707 | **0.6205** |
| Hypertension | 0.1833 | 0.1801 | 0.1569 | **0.1958** |
| Myopia | 0.9331 | 0.9249 | 0.9216 | **0.9417** |
| Other | 0.2905 | 0.3173 | 0.4053 | **0.4160** |

---

## 8. Interpretation

### 8.1. What the models are good at
- **Cataract** and **Myopia** show near-ceiling AUROC/AUPRC (strong, obvious image patterns).
- **Glaucoma** and **AMD** benefit from Swin’s global attention.

### 8.2. What the models struggle with
- **Hypertension** and **Other** have subtle, heterogeneous features.  
- Low AUPRC highlights difficulty in precision for rare diseases.

### 8.3. Why the ensemble works
- CNNs capture **local textures**.
- Swin captures **global structure**.
- Ensemble blends these complementary strengths → best macro metrics across all classes.

### 8.4. ADRD relevance
- ADRD retinal biomarkers resemble the “subtle” disease categories.
- Since even standard ocular diseases are challenging for CNNs alone, ADRD analysis requires:
  - foundation models (e.g., RETFound),
  - global context modeling,
  - fine-grained vascular representation.

---

## 9. Reproducibility Steps

1. Download ODIR-5K dataset.  
2. Generate `ODIR-5K-clean.csv`.  
3. Load dataset with unified transform.  
4. Train:
   - ResNet-18  
   - EfficientNet-B3  
   - Swin-Tiny  
   - CNN-GNN hybrid (optional)  
5. Save validation logits.  
6. Run ensemble grid search to find (0.2, 0.1, 0.7).  
7. Compute AUROC/AUPRC tables.  
8. Replicate plots (macro curves, per-class bars, ensemble heatmap).

---

## 10. Future Work

- Integrate **RETFound** or other retinal foundation models  
- Evaluate **ADRD biomarker labels** when available  
- Apply **focal loss**, **label smoothing**, or **balanced sampling** to improve rare disease detection  
- Explore **multi-task learning** (ocular diseases + ADRD proxy tasks)  

---


