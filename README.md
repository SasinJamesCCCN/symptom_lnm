# sLNM Method Validation

Code for the analyses in:

> **Do Symptoms Matter? Investigating Symptom-Based Lesion Network Mapping.**  
> Treeratana et al., *[in prep]*

This repository contains four analysis notebooks demonstrating that symptom-based Lesion Network Mapping (sLNM) systematically converges toward principal gradient of the reference connectome rather than identifying disease-specific circuits.

---

## Directory Structure

```
slnm_method/
├── code/
│   ├── neuroimage_analysis.py            # Core analysis utilities
│   ├── 01_clinical_slnm.ipynb            # Analysis 1: clinical sLNM (TMS & Broca)
│   ├── 02_simulated_slnm.ipynb           # Analysis 2: simulated datasets
│   ├── 03_clinical_predictions.ipynb     # Analysis 3: sLNM vs Gradient 1 predictions
│   └── 04_connectome_randomization.ipynb # Analysis 4: eigenswap analysis
├── data/
│   ├── schaefer_300_fcmap/               # Schaefer 300 parcel FC maps (ground truth)
│   ├── sim_lesions/                      # Simulated lesion FC maps (n=500)
│   ├── matrix/                           # GSP1000 connectivity matrix
│   └── templates/                        # Brain mask and gradients
│       └── surf/                         # Schaefer-1000 fsaverage5 surface and label files
└── results/                              # Pickled sLNM outputs and CIFTI brain maps
```

---

## Requirements

To run these project scripts, create a virtual environment using `uv sync`, details are in the `pyproject.toml` file.

---

## Data

All data required to reproduce the analyses #2 and #4 are included in the `data/` directory. No additional downloads are needed.

> **Note:** Clinical datasets (TMS-depression and Broca's aphasia) are not included in this repository as they contain patient data. To reproduce analyses #1 and #3, please obtain the data directly from the original publications (see Datasources below).

> **Note:** Surface files in `data/templates/surf/` are sourced from the [CBIG repository](https://github.com/ThomasYeoLab/CBIG) and are distributed under the MIT License.

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

---

## Analysis Overview

### 1. Clinical sLNM (`01_clinical_slnm.ipynb`)
Computes sLNM maps from TMS-depression and Broca's aphasia datasets and tests whether they converge using a symptom-permutation test. Requires clinical data obtained from the original publications.

### 2. Simulated sLNM (`02_simulated_slnm.ipynb`)
Tests sLNM statistical properties using synthetic datasets with known ground truth networks at η² ∈ {0.0, 0.3, 0.99}.

### 3. Clinical Predictions (`03_clinical_predictions.ipynb`)
Compares leave-one-out sLNM prediction against Gradient 1 (PC1 of the GSP1000 healthy reference connectome) for both clinical datasets. Requires clinical data obtained from the original publications.

### 4. Connectome Randomization (`04_connectome_randomization.ipynb`)
Dissociates the degree map from Gradient 1 via eigenvalue swapping and tests which structure dominates sLNM outputs.

---

## Results

Pre-computed results are provided in the `results/` directory:

- Pickled outputs from 1000 sLNM runs at each effect size (η² = 0.0, 0.3, 0.99)
- NIFTI `.nii.gz` and CIFTI `.dscalar.nii` files for the clinical sLNM maps, 

---

## Datasources

**GSP1000 normative fMRI data:** https://doi.org/10.7910/DVN/ILXIKS

**TMS-Depression dataset:** Weigand et al. (2018). Biological Psychiatry, 84(1), 28–37.

**Aphasia Recovery Cohort:** Gibson et al. (2024). Scientific Data, 11, 970. — https://github.com/neurolabusc/AphasiaRecoveryCohortDemo

**MNI Brain template:** Taylor et al. (2023). Nature Human Behaviour, 7(3), 420–429. — https://github.com/nimlab/NHB_Taylor2023