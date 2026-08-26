# Gonad-MRI_SegAI

Automated segmentation and volumetric analysis of ovaries, ovarian cysts, and testicles on MRI using nnU-Net.

This repository accompanies the published study:

> **Artificial Intelligence-Based Approach for Automated Gonad Volume Quantification Using Magnetic Resonance Imaging in Healthy Adolescents Across Puberty**  
> Haque F, Harmon SA, Kumnick A, et al. *Diagnostics*. 2026;16(9):1357.  
> DOI: [10.3390/diagnostics16091357](https://doi.org/10.3390/diagnostics16091357)

Pretrained model checkpoints and example data are available from the associated [Zenodo record](https://zenodo.org/records/16414788).

> **Research use only:** The models and scripts in this repository are intended for research and reproducibility. They are not medical devices and have not been validated for clinical diagnosis, treatment planning, or patient-care decisions.

## Overview

Gonad-MRI_SegAI provides nnU-Net-based models for automated gonadal segmentation on T2-weighted MRI. The released workflows cover:

- Ovary and ovarian cyst segmentation using fat-saturated T2-weighted MRI, conventional T2-weighted MRI, or both modalities.
- Testicular segmentation using T2-weighted MRI.
- Post-inference utilities for segmentation statistics and volume extraction.

The associated study used longitudinal MRI data from healthy adolescents and evaluated automated gonadal volume quantification across puberty. On the in-house test set, the reported Dice similarity coefficients were **0.86 for ovaries**, **0.69 for ovarian cysts**, and **0.90 for testicles**.

## Model variants

The release uses the following nnU-Net dataset identifiers. The names are retained exactly because they are part of the model-package metadata.

### Ovary and ovarian cyst models

| Dataset identifier | MRI input | Description |
|---|---|---|
| `Dataset101_MRI-Ovary-Cyst` | FS-T2W + T2W | Dual-modality model. T2W is spatially resampled/aligned to the FS-T2W image before inference. |
| `Dataset102_MRI-Ovary-Cyst` | FS-T2W | Single-modality fat-saturated T2-weighted MRI model. |
| `Dataset103_MRI-Ovary-Cyst` | T2W | Single-modality conventional T2-weighted MRI model. |

The ovary/cyst analysis utilities treat label `1` as ovary and label `2` as ovarian cyst.

### Testicular models

| Dataset identifier | MRI input | Description |
|---|---|---|
| `Dataset101_MRI-testie` | T2W | Whole-testicle segmentation model. |
| `Dataset102_MRI-testie` | T2W | Side-separated testicular segmentation model. |

For exact label definitions, trainer/plans configuration, folds, and checkpoint locations, use the `dataset.json`, `plans.json`, and model directory structure distributed with the corresponding pretrained package.

## Repository contents

The GitHub repository contains analysis utilities and documentation. Large pretrained checkpoints and example imaging data are hosted separately on Zenodo.

```text
Gonad-MRI_SegAI/
|-- README.md
|-- LICENSE
|-- dicom_to_nifti_conversion.py
|-- Ovary-Cyst_AI_statistics.py
|-- Ovary-cyst_volume_extraction.py
|-- Testicular_AI_statistics.py
`-- Testicular_AI_volume_extraction.py
```

The inference shell scripts referenced in earlier versions of this README are not currently tracked on the GitHub `main` branch. Use the inference workflow and model metadata distributed with the downloaded Zenodo package, or run nnU-Net v2 directly using the package-specific dataset, trainer, plans, configuration, and fold information.

## Installation

Python 3.10 is recommended for compatibility with the original workflow.

```bash
git clone https://github.com/HFahmida/Gonad-MRI_SegAI.git
cd Gonad-MRI_SegAI

conda create -n mri_gonad python=3.10 -y
conda activate mri_gonad
```

Install PyTorch for the CUDA version available on your system, then install nnU-Net v2 and the packages used by the analysis scripts:

```bash
python -m pip install --upgrade pip
python -m pip install nnunetv2 SimpleITK numpy pandas connected-components-3d openpyxl
```

For nnU-Net installation and configuration details, see the official [nnU-Net repository](https://github.com/MIC-DKFZ/nnUNet).

If you run nnU-Net directly rather than through a package-specific inference script, configure the required nnU-Net paths according to the model archive and your local installation:

```bash
export nnUNet_raw=/absolute/path/to/nnUNet_raw
export nnUNet_preprocessed=/absolute/path/to/nnUNet_preprocessed
export nnUNet_results=/absolute/path/to/nnUNet_results
```

## Download pretrained models

Download the pretrained checkpoints and example files from:

**Zenodo:** [https://zenodo.org/records/16414788](https://zenodo.org/records/16414788)

Extract the downloaded model package in a location with sufficient storage. Preserve the nnU-Net directory structure and metadata supplied with the archive. In particular, do not rename dataset directories or checkpoint folders unless you also update the corresponding nnU-Net configuration.

## Input preparation

### NIfTI format

nnU-Net expects input images in NIfTI format. Compressed NIfTI files (`.nii.gz`) are recommended.

Each case must have a unique case identifier. The channel suffix determines which modality is supplied to nnU-Net.

### Dual-modality ovary/cyst model

For `Dataset101_MRI-Ovary-Cyst`:

- FS-T2W is channel 0: `CASE_ID_0000.nii.gz`
- T2W is channel 1: `CASE_ID_0001.nii.gz`

Example:

```text
imagesTs/
|-- id-023_2014-09_0000.nii.gz   # FS-T2W
`-- id-023_2014-09_0001.nii.gz   # T2W
```

The two images for a case must represent the same subject/time point and must be spatially aligned. The workflow described in the original release resamples the T2W image to the FS-T2W image geometry before use with the dual-modality model.

### Single-modality models

For a single-modality FS-T2W, T2W, or testicular model, the image is channel 0:

```text
imagesTs/
|-- case001_0000.nii.gz
|-- case002_0000.nii.gz
`-- case003_0000.nii.gz
```

Do not use `_0001.nii.gz` for a single-modality model unless its `dataset.json` explicitly defines a second input channel.

## DICOM-to-NIfTI conversion

The repository includes `dicom_to_nifti_conversion.py` as a research helper. The current script contains dataset-specific assumptions and should be reviewed and validated against your DICOM directory organization before use.

For reproducible inference, confirm after conversion that:

- Image orientation is correct.
- Voxel spacing and physical coordinates are preserved.
- Multi-modality images are registered/resampled as required by the selected model.
- Output filenames follow nnU-Net channel naming conventions.

## Inference

Use the pretrained package corresponding to the desired anatomy and MRI input. The exact `nnUNetv2_predict` arguments depend on the dataset identifier, trainer, plans, configuration, and trained fold stored with that package.

A general nnU-Net v2 prediction command has the form:

```bash
nnUNetv2_predict \
  -i /absolute/path/to/imagesTs \
  -o /absolute/path/to/predictions \
  -d DATASET_ID \
  -c CONFIGURATION \
  -tr TRAINER \
  -p PLANS \
  -f FOLD \
  -device cuda
```

Replace each placeholder with the values distributed with the selected model. Do not assume that all model variants use identical trainer/plans or fold settings.

Use `-device cpu` when GPU inference is unavailable. A CUDA-compatible GPU is recommended for 3D full-resolution inference.

## Post-inference analysis

The repository contains four research utilities:

| Script | Purpose |
|---|---|
| `Ovary-Cyst_AI_statistics.py` | Computes connected-component-based statistics for ovary/cyst masks. |
| `Ovary-cyst_volume_extraction.py` | Computes total, right, and left ovary/cyst volumes from segmentation masks. |
| `Testicular_AI_statistics.py` | Computes segmentation comparison statistics for testicular predictions and ground truth. |
| `Testicular_AI_volume_extraction.py` | Computes testicular volumetric measurements. |

These scripts were developed for the study workflow and contain hard-coded or study-specific paths in their `__main__` sections. They are not generalized command-line applications. Review the source code and replace the input, prediction, ground-truth, and output paths before running them on a new dataset.

Volume calculations use the NIfTI voxel spacing and convert voxel volume from mm³ to cm³. Therefore, preserving correct image geometry is essential.

Example invocation after adapting a script's paths:

```bash
python Ovary-cyst_volume_extraction.py
```

or

```bash
python Testicular_AI_statistics.py
```

For independent inference without ground-truth annotations, use the volume-extraction portions that operate on AI prediction masks. Ground-truth-dependent evaluation metrics require reference segmentations and should not be interpreted as available for unlabeled deployment data.

## Reported performance

The associated publication reports the following Dice similarity coefficients on the in-house test set:

| Target | Dice similarity coefficient |
|---|---:|
| Ovary | 0.86 |
| Ovarian cyst | 0.69 |
| Testicle | 0.90 |

These values describe performance in the study cohorts and do not guarantee equivalent performance on MRI acquired with different scanners, protocols, populations, anatomy coverage, or preprocessing pipelines. External validation is recommended before applying the models to a new research cohort.

## Citation

If you use this repository or the pretrained models, please cite the associated publication:

> Haque F, Harmon SA, Kumnick A, Soliman M, Berman KF, Yanovski JA, Turkbey EB, Nieman LK, Gomez-Lobo V, Wei S-M, Schmidt PJ, Turkbey B. **Artificial Intelligence-Based Approach for Automated Gonad Volume Quantification Using Magnetic Resonance Imaging in Healthy Adolescents Across Puberty.** *Diagnostics*. 2026;16(9):1357. https://doi.org/10.3390/diagnostics16091357

BibTeX:

```bibtex
@article{haque2026gonad_mri_segai,
  title   = {Artificial Intelligence-Based Approach for Automated Gonad Volume Quantification Using Magnetic Resonance Imaging in Healthy Adolescents Across Puberty},
  author  = {Haque, Fahmida and Harmon, Stephanie A. and Kumnick, Allison and Soliman, Mary and Berman, Karen F. and Yanovski, Jack A. and Turkbey, Evrim B. and Nieman, Lynnette K. and Gomez-Lobo, Veronica and Wei, Shau-Ming and Schmidt, Peter J. and Turkbey, Baris},
  journal = {Diagnostics},
  year    = {2026},
  volume  = {16},
  number  = {9},
  pages   = {1357},
  doi     = {10.3390/diagnostics16091357}
}
```

Please also cite [nnU-Net](https://github.com/MIC-DKFZ/nnUNet) as appropriate and reference the [Zenodo model record](https://zenodo.org/records/16414788) when using the released checkpoints.

## License

The source code in this GitHub repository is distributed under the [GNU General Public License v3.0](LICENSE).

Model weights and example data distributed through Zenodo may have separate terms. Consult the Zenodo record before redistribution or reuse.

## Acknowledgments

The associated work was supported by the Intramural Research Program of the National Institutes of Health and used the computational resources of the NIH HPC Biowulf cluster.

## Contact and issues

For reproducibility questions, unexpected model behavior, or documentation problems, please open a GitHub issue and include the model variant, MRI modality, nnU-Net version, relevant input naming, and the error message or unexpected output.
