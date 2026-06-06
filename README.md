# Ni-WC SEM Microstructure Segmentation and Quantitative Phase Characterization

Research workflow for semantic segmentation and quantitative characterization of Ni-WC metal matrix composite SEM microstructures.

**Authors:** Omar Faruk, Israt Jahan Keya, and Musfique Mahmud Foysal

## Scope

The project performs four-class semantic segmentation:

| Raw mask ID | Model class ID | Class name |
|---:|---:|---|
| 1 | 0 | Matrix |
| 3 | 1 | Carbide particles |
| 4 | 2 | Dilution band |
| 5 | 3 | Reprecipitated carbides |

Raw label `2` is treated as unlabeled/ignored and is excluded from optimization and metric computation.

## Main Notebook

Run the complete workflow from:

```text
niwc_microstructure_segmentation.ipynb
```

The notebook is organized as a research pipeline:

1. Environment and configuration
2. Dataset manifest and label remapping
3. Fixed train/validation/test split
4. Dataset diagnostics
5. Model registry
6. Augmentation, class weighting, and loaders
7. Loss functions and evaluation metrics
8. Training and checkpointing
9. Result aggregation
10. Manuscript figures
11. Final-model material characterization

## Dataset Layout

Expected folder structure:

```text
data/
  AugmentedImages/
    sem600_x*_y*.bmp
    sem700_x*_y*.bmp
    sem800_x*_y*.bmp
    sem1000_x*_y*.bmp
  AugmentedMasks/
    sem600_x*_y*.bmp
    sem700_x*_y*.bmp
    sem800_x*_y*.bmp
    sem1000_x*_y*.bmp
```

Filename convention:

```text
sem<MAGNIFICATION>_x<X>_y<Y>[_augmentation].bmp
```

The filename is used to infer magnification source, patch identity, and augmentation status.

## Experimental Protocol

The primary protocol is a fixed cross-magnification held-out test:

- Training/validation sources: `sem1000`, `sem800`, `sem700`
- Held-out test source: `sem600`
- Validation split: every sixth independent patch from the training sources
- Seeds: `1, 2, 3, 4, 5`

This design evaluates deployment to an unseen magnification while preventing patch leakage between train, validation, and test sets.

## Models

The notebook includes only the models used in the manuscript comparison:

- U-Net ResNet34, ImageNet
- U-Net ResNet34, MicroNet
- U-Net EfficientNet-B3, ImageNet
- U-Net EfficientNet-B3, MicroNet
- U-Net++ ResNet34, ImageNet
- U-Net++ ResNet34, MicroNet
- U-Net++ EfficientNet-B3, ImageNet
- U-Net++ EfficientNet-B3, MicroNet
- DeepLabV3+ ResNet34, ImageNet
- DeepLabV3+ ResNet34, MicroNet
- DeepLabV3+ EfficientNet-B3, ImageNet
- DeepLabV3+ EfficientNet-B3, MicroNet
- SegFormer MiT-B0
- Final U-Net++ ResNet34 with EMA checkpointing and geometric test-time augmentation

Exploratory branches that were not retained in the manuscript workflow are intentionally excluded from the clean notebook.

## Training Configuration

Shared settings:

- Input size: `512 x 512`
- Batch size: `8`
- Evaluation batch size: `64`
- Epochs: `200`
- Early stopping patience: `15`
- Optimizer: AdamW
- Initial learning rate: `2e-4`
- Scheduler: cosine annealing to `1e-6`
- Weight decay: `1e-4`
- Loss: weighted focal loss + soft Dice loss
- Mixed precision: CUDA bfloat16 autocast
- CUDA speed settings: TF32 enabled, cuDNN benchmark enabled
- Data handling: image and mask tensors cached on GPU when running on CUDA

Training augmentation:

- Horizontal flip
- Vertical flip
- Rotation by multiples of 90 degrees
- Brightness jitter
- Contrast jitter
- Gamma jitter
- Gaussian noise

## Resume Behavior

The notebook is checkpoint-aware at the model/seed level.

For each model and seed, it checks for:

```text
test_metrics.csv
history.csv
per_class_metrics.csv
best_model.pt
```

If these files exist, the run is loaded and skipped. If a run was interrupted before completion, that exact model/seed is retrained. This allows the notebook to continue from the first incomplete run after an interruption.

## Outputs

All outputs are written to:

```text
paper_outputs/
  checkpoints/
  figures/
  predictions/
  tables/
```

Key CSV outputs:

- `paper_outputs/tables/manifest.csv`
- `paper_outputs/tables/split_manifest.csv`
- `paper_outputs/tables/raw_label_audit.csv`
- `paper_outputs/tables/split_audit.csv`
- `paper_outputs/tables/class_distribution_by_split.csv`
- `paper_outputs/tables/model_registry.csv`
- `paper_outputs/tables/class_weights.csv`
- `paper_outputs/tables/training_sample_weights.csv`
- `paper_outputs/tables/all_training_history.csv`
- `paper_outputs/tables/seed_level_test_metrics.csv`
- `paper_outputs/tables/seed_level_per_class_metrics.csv`
- `paper_outputs/tables/paper_model_leaderboard.csv`
- `paper_outputs/tables/unetpp_resnet34_final_training_curve.csv`
- `paper_outputs/tables/unetpp_resnet34_final_phase_fraction_by_image.csv`
- `paper_outputs/tables/unetpp_resnet34_final_particle_morphology.csv`
- `paper_outputs/tables/unetpp_resnet34_final_phase_spatial_profiles.csv`

Figures are displayed in the notebook and exported as:

```text
*.png
*.tiff
*.svg
```

at 600 dpi using the manuscript plotting style.

At the end of the workflow, the complete output folder is archived as:

```text
paper_outputs.zip
```

## Running the Workflow

Create or activate the experiment environment, then open the notebook:

```bash
conda activate thesiswork
jupyter lab niwc_microstructure_segmentation.ipynb
```

The notebook assumes a CUDA-capable GPU runtime for the final training run.

## Recommended GitHub Upload

Recommended files to upload:

```text
README.md
niwc_microstructure_segmentation.ipynb
requirements_l40s_abi.txt
fix_l40s_numpy_abi.sh
make_*.py                         # optional figure/table helper scripts, if needed
```

Large datasets, trained checkpoints, and generated outputs should be uploaded only if permitted by the dataset and manuscript data-availability policy. Otherwise, provide them through a release archive, institutional repository, Zenodo, OSF, or journal supplementary data.

## Reproducibility Notes

- Five seeds are used for the reported deep learning runs.
- The test set is fixed and source-held-out.
- All model histories, per-class metrics, confusion matrices, prediction masks, and material-characterization tables are saved.
- The final model is reported separately because EMA and TTA are inference-stabilization steps applied after model selection.


```text
Faruk, O.; Keya, I. J.; Foysal, M. M. Ni-WC SEM Microstructure Segmentation and Quantitative Phase Characterization. GitHub repository, 2026.
```

