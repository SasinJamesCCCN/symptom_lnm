# sLNM Method Validation

Code for the analyses in:

> **Connectome Principal Component Drives Cross-Dataset Replication and Clinical Prediction in Symptom Lesion Network Mapping.**
> Treeratana et al., *[under review]*
> https://doi.org/10.64898/2026.03.04.709716

This repository contains five analysis notebooks demonstrating that symptom-based Lesion Network Mapping (sLNM) systematically converges toward PC1 of the reference connectome.

---

## Directory Structure

```
symptom_lnm/
├── code/
│   ├── neuroimage_analysis.py            # Core analysis utilities
│   ├── 01_clinical_slnm.ipynb            # Analysis 1: clinical sLNM (TMS & Broca)
│   ├── 02_latent_space.ipynb             # Analysis 2: latent space (PC alignment)
│   ├── 03_connectome_randomization.ipynb # Analysis 3: eigenswap analysis
│   ├── 04_simulated_slnm.ipynb           # Analysis 4: convergence test on simulated datasets
│   └── 05_clinical_predictions.ipynb     # Analysis 5: sLNM vs PC1 controlled predictions
├── data/
│   ├── schaefer_300_fcmap/               # Schaefer 300 parcel FC maps (ground truth)
│   ├── sim_lesions/                      # Simulated lesion FC maps (n=500)
│   ├── pca/                              # Voxel-wise GSP1000 PC maps
│   ├── matrix/                           # GSP1000 connectivity matrix
│   ├── arc_dataset/                      # Aphasia Recovery Cohort (Broca's aphasia)
│   ├── tms_dataset/                      # TMS-depression dataset
│   ├── control_maps/                     # Control maps used to predict TMS response
│   └── templates/                        # Brain mask and surface files
└── results/                              # Pickled sLNM outputs and NIFTI brain maps
```

---

## Requirements

To run these project scripts, create a virtual environment using `uv sync`, details are in the `pyproject.toml` file.

---

## Data

Simulated data required to reproduce all analyses are included in the `data/` directory.

Brain analyses are performed in MNI152 space downloaded from a previous LNM study (https://github.com/nimlab/NHB_Taylor2023), to download the file please run this command:

```bash
curl -L https://github.com/nimlab/NHB_Taylor2023/raw/master/MNI152_T1_2mm_brain_mask_dil.nii.gz -o data/templates/Taylor_NHB_MNI152_T1_2mm_brain_mask_dil.nii.gz
```

> **Note:** Surface files in `data/templates/surf/` are sourced from the [CBIG repository](https://github.com/ThomasYeoLab/CBIG) and are distributed under the MIT License.

### Clinical Datasets

Both clinical datasets are openly and publicly available from their original publications.

#### `tms_dataset/` — TMS-Depression
*Data from Weigand et al. [Biological Psychiatry (2018)](https://www.biologicalpsychiatryjournal.com/article/S0006-3223(17)32158-3/abstract), which recorded MNI coordinates of TMS coil placement in depressive patients along with post-treatment Beck Depression Inventory (BDI) reduction.*

#### `arc_dataset/` — Aphasia Recovery Cohort (Broca's aphasia)
*Data sourced from the open-access [Aphasia Recovery Cohort (ARC)](https://github.com/neurolabusc/AphasiaRecoveryCohortDemo) dataset (Gibson et al., [*Scientific Data*, 2024](https://www.nature.com/articles/s41597-024-03819-7)), which contains stroke lesion masks registered to standard MNI space and speech ability scores measured by the Western Aphasia Battery Aphasia Quotient (WAB-AQ).*

### Control Maps

A full description of the control maps, along with their source publications, is provided in the supplementary material of our preprint.

---

## Core Module: `neuroimage_analysis.py`

### Key Functions

#### `voxel_outcome_correlation(brain_data, outcomes)`
Compute Pearson correlation between each brain voxel and behavioral outcomes. Can handle multiple permuted datasets in parallel.
- `brain_data`: ndarray (n_subjects × n_voxels)
- `outcomes`: ndarray (n_subjects,) or (n_subjects, n_permutations)
- Returns: ndarray (n_voxels,) for single permutation, (n_permutations, n_voxels) for multiple permutations

#### `nifti_getdata(nifti_path, brain_template_path=None)`
Load a NIfTI file and return masked, flattened brain voxels with NaN interpolation.
- `nifti_path`: str
- `brain_template_path`: str (uses default brain mask if None)
- Returns: ndarray (n_brain_voxels,)

#### `pearson_rows(X, y)`
Compute Pearson correlation between each row of `X` and a reference vector `y`. Handles NaN values by computing correlations only on valid pairs.
- `X`: ndarray (n_subjects × n_voxels)
- `y`: ndarray (n_voxels,)
- Returns: ndarray (n_subjects,) — correlation of each row with `y`

#### `gen_dataset(subject_maps, ground_truth_maps, sample_size=100, effect_size=0.3, ground_truth_seed=None, rank=False, z_transform=False, lesion_seed=None)`
Generate a simulated dataset with synthetic behavioral scores. Bootstraps subjects from lesion FC maps and creates scores by correlating each subject's FC map with a ground-truth Schaefer region map, scaled by effect size with added noise.
- `subject_maps`: list of paths to subject FC NIfTI files
- `ground_truth_maps`: list of paths to Schaefer region FC NIfTI files
- `sample_size`: number of subjects to bootstrap (default: 100)
- `effect_size`: η², strength of the brain–behavior relationship (default: 0.3)
- `ground_truth_seed`: index of ground-truth map to use (random if None)
- `z_transform`: if True, Fisher z-transform the correlations
- `lesion_seed`: seed for reproducible lesion bootstrap (random if None)
- Returns: dict with `subject_maps` (n_subjects × voxels), `ground_truth` (voxels,), `scores` (n_subjects,), `ground_truth_seed`



---

## Analysis Overview

### 1. Clinical sLNM (`01_clinical_slnm.ipynb`)
Computes sLNM maps from TMS-depression and Broca's aphasia datasets and tests whether they converge using a symptom-permutation test.

### 2. Latent Space (`02_latent_space.ipynb`)
Examines alignment of clinical sLNM maps with the first three principal components of the GSP1000 connectome.

### 3. Connectome Randomization (`03_connectome_randomization.ipynb`)
Dissociates the degree map from PC1 via eigenvalue swapping and tests which structure dominates sLNM outputs.

### 4. Simulated sLNM (`04_simulated_slnm.ipynb`)
Tests sLNM recovery accuracy using synthetic datasets with known ground-truth networks at η² ∈ {0.0, 0.3, 0.6, 0.99}.

### 5. Clinical Predictions (`05_clinical_predictions.ipynb`)
Compares leave-one-out sLNM prediction against control maps for both clinical datasets.

---

## Results

Pre-computed results are provided in the `results/` directory:

- Pickled outputs from 1,000 convergence test sLNM runs at each effect size (η² = 0.0, 0.3, 0.6, 0.99)
- Pickled output for ground-truth vs sLNM PC1-3 alignments
- NIfTI `.nii.gz` files for the clinical sLNM maps

---

## Datasources

**GSP1000 normative fMRI data:** https://doi.org/10.7910/DVN/ILXIKS

**TMS-Depression dataset:** Weigand et al. (2018). [Biological Psychiatry, 84(1), 28–37.](https://www.biologicalpsychiatryjournal.com/article/S0006-3223(17)32158-3/abstract) Data found in supplementary materials.

**Aphasia Recovery Cohort:** Gibson et al. (2024). Scientific Data, 11, 970. — https://github.com/neurolabusc/AphasiaRecoveryCohortDemo (Distributed under [BSD 2-Clause "Simplified" License](https://github.com/neurolabusc/AphasiaRecoveryCohortDemo/blob/main/LICENSE))

**MNI Brain template:** Taylor et al. (2023). Nature Human Behaviour, 7(3), 420–429. — https://github.com/nimlab/NHB_Taylor2023/blob/master/MNI152_T1_2mm_brain_mask_dil.nii.gz

**Control maps** See supplementary tables S1–3.

**Surface template files** CBIG repository: https://github.com/ThomasYeoLab/CBIG (Distributed under [MIT license](https://github.com/ThomasYeoLab/CBIG?tab=MIT-1-ov-file))