# Model Card: MST Classification (DINOv2 Vision Transformer)

## Model Details

| Field | Value |
|-------|-------|
| **Name** | ODELIA MST (Multi-Scale Transformer) |
| **Architecture** | DINOv2-based Vision Transformer |
| **Task** | 3-class classification per breast: No lesion / Benign / Malignant |
| **Output** | Bilateral classification with confidence scores + attention map overlays |
| **Source** | [ODELIA-AI/MST on HuggingFace](https://huggingface.co/ODELIA-AI/MST) (gated model) |
| **License** | Research-only use (see HuggingFace model card for full terms) |
| **Service port** | 5556 |

## Intended Use

Research-only analysis of breast MRI with interpretability via attention maps. Not approved for clinical decision-making.

## Input Requirements

### Modality & Anatomy

- **Modality:** MRI (breast MRI, single 3D series)
- **Anatomy:** Breast

### Required DICOM Tags

| Tag | Keyword | Required | Purpose |
|-----|---------|----------|---------|
| `(0020,000D)` | StudyInstanceUID | Yes | Study-level retrieval |
| `(0020,000E)` | SeriesInstanceUID | Yes | Series-level retrieval |
| `(0020,0100)` | TemporalPositionIdentifier | Recommended | Primary tag for temporal phase detection |
| `(0018,1060)` | TriggerTime | Fallback | Used if TemporalPositionIdentifier is absent |
| `(0020,1041)` | SliceLocation | Recommended | Primary tag for spatial slice ordering |
| `(0020,0032)` | ImagePositionPatient | Fallback | Z-coordinate used if SliceLocation is absent |
| `(0020,0013)` | InstanceNumber | Fallback | Used if both spatial tags are absent |
| `(7FE0,0010)` | PixelData | Yes | Image data |

### Temporal Handling (4D Series)

- If multiple temporal phases are detected, **only the first temporal position** is used
- Single-phase series are used as-is

### Conversion Pipeline

1. DICOM files are read and grouped by temporal position
2. First temporal phase is selected (or all files if single-phase)
3. Files are sorted by `SliceLocation` (descending, matching GDCM default superior-to-inferior)
4. Series is converted to a single NIfTI file via SimpleITK
5. NIfTI is loaded as a TorchIO `ScalarImage`
6. Further preprocessing and inference is handled by the HuggingFace model code (`predict_attention.py`)

## Input Limitations & Failure Modes

| Condition | Behavior |
|-----------|----------|
| No `.dcm` files in folder | `ValueError` raised |
| GDCM cannot recognize series | `ValueError` raised |
| Missing `SliceLocation` and `ImagePositionPatient` | All slices default to position 0.0 — incorrect 3D volume ordering, **silent corruption** |
| Missing `TemporalPositionIdentifier` and `TriggerTime` | All files grouped as single phase (temporal position 0) — safe for single-phase input, but multi-phase data will be merged incorrectly |
| Non-breast anatomy | Model will still run but predictions are meaningless |
| Very small volumes | Depends on model's internal handling (from HuggingFace repo) |

## Authentication

Requires a valid HuggingFace token (`HF_TOKEN` environment variable) to download the gated model. The model license must be accepted at https://huggingface.co/ODELIA-AI/MST before first use.
