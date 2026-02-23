# Model Limitations

> **All AI models in ODELIA Viewer are for research use only.**
> Outputs may be inaccurate and must not be used for clinical decision-making.

## Integrated Models

| Model | Architecture | Task | Classes | Input |
|-------|-------------|------|---------|-------|
| [Breast Cancer Classification](model_card_breast_cancer_classification.md) | ResNet-18 | Binary classification | Cancerous / Not Cancerous | Bilateral breast MRI (DCE, Pre + Post) |
| [MST Classification](model_card_mst_classification.md) | DINOv2 Vision Transformer | 3-class classification | No lesion / Benign / Malignant | Breast MRI (single 3D series) |
| [MedGemma MRI](model_card_medgemma_mri.md) | Vision-Language Model | 3-class classification | No lesion / Benign / Malignant | Breast MRI (single 3D series) |
| [Chat Middleware](model_card_chat_middleware.md) | Vision-Language Model (Ollama) | Free-form chat | N/A | Any DICOM series |

## Common Input Requirements

All models receive DICOM data via WADO-RS retrieval. The following tags are universally required:

| Tag | Keyword | Purpose |
|-----|---------|---------|
| `(0020,000D)` | StudyInstanceUID | Study-level retrieval |
| `(0020,000E)` | SeriesInstanceUID | Series-level retrieval |
| `(7FE0,0010)` | PixelData | Image data |

## Critical DICOM Tags by Model

| Tag | Keyword | BCC | MST | MedGemma | Chat |
|-----|---------|-----|-----|----------|------|
| `(0018,1060)` | TriggerTime | **Required** | Fallback | Fallback | — |
| `(0020,0100)` | TemporalPositionIdentifier | — | Primary | Primary | — |
| `(0020,1041)` | SliceLocation | — | Primary | Primary | Via GDCM |
| `(0020,0032)` | ImagePositionPatient | — | Fallback | Fallback | Via GDCM |
| `(0020,0013)` | InstanceNumber | — | Fallback | Fallback | Via GDCM |

**BCC** = Breast Cancer Classification, **MST** = MST Classification, **MedGemma** = MedGemma MRI, **Chat** = Chat Middleware

## Key Limitations

### All Models

- **Research use only** — not validated for clinical use
- **Breast MRI only** — all classification models assume breast MRI input; other anatomies will produce unreliable results
- **No demographic bias testing** — model performance across patient demographics has not been systematically evaluated
- **No performance metrics published** — evaluation results are not tracked or displayed within the viewer

### Breast Cancer Classification

- Requires `TriggerTime` DICOM tag to separate dynamic contrast phases (Pre / Post)
- Requires both pre-contrast and post-contrast phases; will fail if either is missing
- Assumes bilateral axial acquisition with left/right breasts on opposite halves of a 512-pixel width dimension
- Volume is CropOrPad to 512 × 512 × 32 — acquisitions significantly different in size may produce degraded results

### MST Classification

- Downloads model weights from HuggingFace on first run (requires network access and `HF_TOKEN`)
- For 4D temporal series, only the first temporal phase is used; other phases are discarded
- Missing spatial ordering tags (`SliceLocation`, `ImagePositionPatient`) cause **silent volume corruption** — slices default to position 0

### MedGemma MRI

- Only 5 slices (from the central 60% of the volume) are sent to the model — the rest of the volume is not analyzed
- Confidence scores are **self-reported by the language model**, not calibrated probability distributions
- Output depends on JSON parsing of free-text generation — malformed model responses cause errors
- Requires HuggingFace token and license acceptance for the gated model

### Chat Middleware

- Free-form text responses — no structured classification guarantee
- No explicit 4D temporal series handling; multi-phase series may be merged incorrectly
- Depends on an external Ollama server being available and having the model loaded
- Any confidence or certainty expressed in chat responses is not statistically calibrated
