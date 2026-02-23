# Model Card: Breast Cancer Classification (ResNet-18)

## Model Details

| Field | Value |
|-------|-------|
| **Name** | Breast Cancer Classification |
| **Architecture** | ResNet-18 (MONAI `nets.ResNet`, `"basic"` blocks) |
| **Task** | Binary classification per breast: Cancerous vs. Not Cancerous |
| **Input channels** | 2 (pre-contrast + post-1 contrast) |
| **Output** | Bilateral (left/right) prediction with confidence score (0–100%) |
| **Decision threshold** | Probability > 0.5 → "Cancerous" |
| **Service port** | 5555 |

## Intended Use

Research-only analysis of bilateral breast MRI. Not approved for clinical decision-making.

## Input Requirements

### Modality & Anatomy

- **Modality:** MRI (T1-weighted dynamic contrast-enhanced breast MRI)
- **Anatomy:** Bilateral breast
- **Required contrast phases:** Pre-contrast **and** at least one post-contrast phase (Post_1)

### Required DICOM Tags

| Tag | Keyword | Required | Purpose |
|-----|---------|----------|---------|
| `(0020,000D)` | StudyInstanceUID | Yes | Study-level retrieval |
| `(0020,000E)` | SeriesInstanceUID | Yes | Series-level retrieval |
| `(0018,1060)` | TriggerTime | **Yes** | Separates dynamic phases. Sorted by rank to assign: index 0 = "Pre", index 1 = "Post_1", etc. |
| `(7FE0,0010)` | PixelData | Yes | Image data |

### Geometry Constraints

| Constraint | Value |
|------------|-------|
| CropOrPad target shape | 512 × 512 × 32 |
| Bilateral split axis | First spatial axis (width) |
| Per-side input to model | 256 × 256 × 32 |
| Padding mode | Global minimum voxel value |

The pipeline assumes an axial orientation where left and right breasts occupy opposite halves of the 512-pixel width dimension.

### Preprocessing Pipeline

1. DICOM files are grouped by `TriggerTime` to identify dynamic phases
2. Each phase is converted to a TorchIO `ScalarImage` via temporary NIfTI
3. The volume is CropOrPad to 512 × 512 × 32
4. Foreground detection crops the vertical extent (90th percentile thresholding)
5. The volume is split into left (first 256 columns) and right (last 256 columns)
6. Pre and Post_1 images are concatenated along the channel dimension → 2-channel input
7. Z-normalization with percentile clipping (0.5th–99.5th percentile), masking voxels > 0

## Input Limitations & Failure Modes

| Condition | Behavior |
|-----------|----------|
| Missing `TriggerTime` tag | Phase separation fails — cannot distinguish Pre from Post |
| Unequal slice count across dynamic phases | Series is excluded entirely |
| Missing Pre-contrast phase for a side | Returns error: `"Missing Pre contrast image for {side} side"` |
| Missing Post_1 phase for a side | Returns error: `"Missing Post contrast image for {side} side"` |
| Unilateral / non-bilateral acquisition | The 256px left/right split produces a meaningless crop |
| Non-breast anatomy | Model will still run but output is meaningless |
| Non-axial orientation | Left/right split assumption breaks |
| Volume significantly smaller than 512 × 512 × 32 | Heavy zero-padding may degrade prediction quality |
| No `.dcm` files in folder | `ValueError` raised |

## Known Issues

- Model checkpoint loading is currently commented out in `model_service.py` — the model runs with **random (untrained) weights** unless a checkpoint is provided.
