# When Similarity Masks Instability: Evaluating 3D Medical Foundation Model Representations Across MRI Acquisitions

## Abstract

Robustness of medical foundation models is commonly assessed through downstream performance or a single representation
analysis metric, although neither reveals how acquisition changes affect embeddings. We compare ten frozen 3D encoders
using matched T1 and FLAIR brain MRI from 20 participants scanned on four systems. Across vendor, orientation, and
resolution comparisons, we suggest quantifying preservation of between-subject geometry using debiased linear centered
kernel alignment (CKA), feature agreement measured by intraclass correlation (ICC), and the relative contributions of
subject, acquisition condition, and residual variability. Orientation changes generally preserve embedding geometry,
whereas vendor-related sensitivity varies substantially across models. Resolution changes reveal a marked
disagreement: Brain2Vec and nnFoundation-ViT maintain CKA of 0.98 and 0.99 but ICC of only 0.01 and 0.10. Effective
rank varies independently of these scores. Model-specific patterns further motivate hypotheses about pretraining
diversity, training objectives, and preprocessing. The findings argue for complementary representation measures rather
than single-score robustness rankings.

## Data

[ON-Harmony](https://openneuro.org/datasets/ds004712) (OpenNeuro ds004712, CC0; Warrington et al., *Scientific Data*
2025): 20 healthy travelling participants, each scanned on the four 3 T systems common to all participants: two Philips
systems (Achieva and Ingenia; Nottingham) and two Siemens Prisma systems (Oxford). From each session, the 1 mm isotropic
T1-weighted (T1) and T2-FLAIR volumes were used (160 volumes in total). Volumes were skull-stripped with HD-BET,
background values were set to the fifth percentile of the original volume, and images were cropped to 192³ around the
brain. Neither registration nor bias-field correction was applied.

## Pretrained 3D encoders

**Table 1.** Pretrained 3D encoders evaluated in this study: backbone, pretraining objective, domain and data, and
the frozen representation used as the embedding, with its dimensionality *d* in parentheses. n.s.: not specified.

<img src="figures/table_encoders.svg" alt="Table 1" width="100%">

## Preprocessing

**Table 2.** Preprocessing of the 3D encoders, (a) as specified by the authors in the paper and released code, and
(b) as applied in this study to the skull-stripped 192³ volumes described in Data. –: step not used; n.s.: step
applied but not specified.

<img src="figures/table_preprocessing.svg" alt="Table 2" width="100%">

## Robustness evaluation

Each comparison uses the embeddings of the **same 20 participants** under two conditions that differ only in one
factor:

| Factor | Compared inputs | Repeated for |
|---|---|---|
| Vendor | one Philips vs. one Siemens scanner (RAS) | the four Philips–Siemens scanner pairs |
| Orientation | RAS vs. LAS | each of the four scanners |
| Resolution | standard vs. 2 mm input (RAS) | each of the four scanners |
| Modality | T1 vs. FLAIR (RAS) | each of the four scanners |

Vendor, orientation and resolution are evaluated separately for T1 and FLAIR. Each metric is computed for each of the
four scanners or scanner pairs and then averaged.

### Resolution inputs

Standard and 2 mm inputs of each model in the resolution comparison.

| Model | Standard input | 2 mm input |
|---|---|---|
| Neuro-SimCLR, BrainFM, nnFoundation-CNN | 192³ at 1 mm | 192³ downsampled to 96³ |
| NeuroVFM | resampled internally to 1×1×4 mm | 96³ input, resampled internally to 1×1×4 mm |
| BrainAnytime, Brain2Vec, SAM-Med3D | 128³ at 1.5 mm | padded to 256³, then resized to 128³ |
| 3DINO | 112³ at 1.71 mm | padded to 224³, then resized to 112³ |
| BrainIAC | 96³ at 2 mm | eight 96³ patches at 1 mm, features averaged (the comparison input is at 1 mm) |
| nnFoundation-ViT | 192³ at 1 mm | downsampled to 96³ and upsampled back to 192³ (fixed input size) |

### Measures

- **Debiased CKA, RSA and SVCCA**: representational similarity between the two conditions, i.e. how well the
  arrangement of the subjects is preserved.
- **ICC(2,1)**: agreement of the embeddings between the two conditions, computed per embedding dimension; the median
  over dimensions is reported.
- **Variance decomposition**: the share of the embedding variance due to differences between subjects (subject), a
  common difference between the two conditions (shift), and subject-specific changes (residual).
- **Normalised effective rank**: how many directions of the embedding space are used (0: one direction, 1: all
  directions equally).
- **Separability in UMAP space**: accuracy of a logistic regression that separates the two conditions in a 2D UMAP of
  the embeddings (0.5: not separated, 1: fully separated).

## Results

**Table 3.** Representation stability across MRI acquisition variations. Debiased CKA measures preservation of
between-subject geometry, and median absolute-agreement ICC measures agreement of corresponding embedding features.
Vendor, orientation and resolution values are averaged over T1 and FLAIR. Normalized effective rank describes the
spread of representation variance and is not a robustness score. Brain-specific and general medical encoders are
separated by a horizontal line. Bold: highest value per column; underlined: second highest.

<img src="figures/table1.svg" alt="Table 3" width="100%">

### Robustness profiles

Representation profiles for T1 (blue) and FLAIR (red), showing debiased CKA under orientation, vendor and resolution
changes (axes 1–3), median cross-vendor ICC (axis 4) and normalized effective rank (axis 5). On axes 1–4, larger
values indicate more stable embeddings. Brain2Vec and nnFoundation-ViT preserve high geometric similarity across
several conditions, whereas other encoders exhibit more factor-dependent responses. Effective rank describes
representation dimensionality, so polygon area should not be interpreted as an overall performance score.

![Radar plots](figures/radar.png)

Separability of the two inputs in UMAP space (logistic regression accuracy) for orientation, vendor and resolution,
for T1 (blue) and FLAIR (red).
In these plots, larger values indicate that the factor separates the embeddings more clearly, i.e. **less** robust
embeddings.

![Accuracy radar plots](figures/radar_accuracy.png)

### Variance decomposition

Subject (blue), shift (red) and residual (grey) fractions of the embedding variance between the two inputs of each
factor. Vendor, orientation and resolution: T1 (upper bar) and FLAIR (lower bar) for each model; modality: T1 vs.
FLAIR.

![Variance decomposition](figures/variance_decomposition_T1_FLARE.png)

### Heatmaps

Similarity between the two conditions of each factor, averaged over scanners or scanner pairs and over T1 and FLAIR:

| Debiased CKA | Plain linear CKA |
|---|---|
| ![Debiased CKA](figures/similarity_CKA_debiased.png) | ![CKA](figures/similarity_CKA.png) |
| **RSA** | **SVCCA** |
| ![RSA](figures/similarity_RSA.png) | ![SVCCA](figures/similarity_SVCCA.png) |

Median ICC(2,1) per factor and sequence:

![Median ICC](figures/icc_median.png)

### UMAP embeddings

UMAP of each model's embeddings: (a, b) vendor, T1 and FLAIR; (c, d) orientation, T1 and FLAIR; (e) modality;
(f) resolution, T1. Click a figure to enlarge it.

<table>
<tr><td align="center"><a href="figures/umap_BrainIAC.png"><img src="figures/umap_BrainIAC.png" width="420"></a><br>BrainIAC</td><td align="center"><a href="figures/umap_NeuroSimCLR.png"><img src="figures/umap_NeuroSimCLR.png" width="420"></a><br>Neuro-SimCLR</td></tr>
<tr><td align="center"><a href="figures/umap_NeuroVFM.png"><img src="figures/umap_NeuroVFM.png" width="420"></a><br>NeuroVFM</td><td align="center"><a href="figures/umap_BrainFM.png"><img src="figures/umap_BrainFM.png" width="420"></a><br>BrainFM</td></tr>
<tr><td align="center"><a href="figures/umap_BrainAnytime.png"><img src="figures/umap_BrainAnytime.png" width="420"></a><br>BrainAnytime</td><td align="center"><a href="figures/umap_Brain2Vec.png"><img src="figures/umap_Brain2Vec.png" width="420"></a><br>Brain2Vec</td></tr>
<tr><td align="center"><a href="figures/umap_3DINO.png"><img src="figures/umap_3DINO.png" width="420"></a><br>3DINO</td><td align="center"><a href="figures/umap_nnFoundationCNN.png"><img src="figures/umap_nnFoundationCNN.png" width="420"></a><br>nnFoundation-CNN</td></tr>
<tr><td align="center"><a href="figures/umap_nnFoundationViT.png"><img src="figures/umap_nnFoundationViT.png" width="420"></a><br>nnFoundation-ViT</td><td align="center"><a href="figures/umap_SAMMed3D.png"><img src="figures/umap_SAMMed3D.png" width="420"></a><br>SAM-Med3D</td></tr>
</table>
