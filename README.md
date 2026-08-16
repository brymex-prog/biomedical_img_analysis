# Hybrid Biomedical Image-Analysis Pipeline

A compact pipeline for nuclei microscopy image analysis that combines classical computer vision, deep learning, and local LLMs/VLMs (via [Ollama](https://ollama.com)). Built for a university assignment covering multimodal description, classical segmentation, U-Net-based learned segmentation, and a hybrid feature-to-narrative pipeline.

## Overview

| Task | Method | Output |
|---|---|---|
| 1. EDA + image description | `llama3.2-vision` (VLM) | Direct natural-language descriptions of raw microscopy images |
| 2. Numbers-first description | Otsu threshold + `regionprops` + `phi3` (LLM) | Structured feature table → LLM-written summary, grounded in real measurements |
| 3. Learned segmentation | Custom U-Net (~1.9M params) | Trained binary nuclei segmentation, Dice = 0.996 / IoU = 0.992 on validation |
| 4. Hybrid pipeline | U-Net mask → `regionprops` → `phi3` | Per-image structured JSON + narrative on unseen test images, aggregated to CSV |
| 5. Robustness extension | Gaussian blur corruption trace | Compares how Otsu vs. U-Net masks degrade under synthetic image corruption |

## Repository Structure


├── notebook/     # Main Jupyter notebook (Colab-ready, runs end to end)
├── outputs/      # Figures, JSON records, CSVs, and trained checkpoints
├── report/       # Final written report (Word/PDF)
├── README.md
├── LICENSE

## Dataset

Nuclei microscopy dataset (112 images, 256×256, pre-split train/val/test), sourced from [Nickolay-K/Assingnment-3-dataset](https://github.com/Nickolay-K/Assingnment-3-dataset). Each split includes grayscale images and pre-merged binary masks; `metadata.csv` provides per-image density, object count, and intensity statistics.

## Running the Notebook

The notebook is designed to run in **Google Colab** (GPU runtime recommended):

1. Open `notebook/nuclei_pipeline_code.ipynb` in Colab.
2. Run the setup cells — these install dependencies and start a local Ollama server inside the Colab VM.
3. The notebook pulls the required models automatically: `ollama pull llama3.2-vision` and `ollama pull phi3`.
4. Run all cells in order. Each task's outputs (figures, JSON, CSV) are saved under `outputs/` and downloaded at the end of the session, since Colab storage is ephemeral.

## Models Used

- **`llama3.2-vision`** — vision-language model for direct image description (Task 1).
- **`phi3`** — text-only LLM for turning structured numeric features into natural-language summaries (Tasks 2 and 4).

## Key Results

- U-Net achieves **Dice = 0.996, IoU = 0.992** on the validation split after 25 epochs (BCE-Dice loss, 16 base channels).
- The hybrid pipeline runs on all 12 unseen test images, producing an auditable JSON + CSV record per image (object count, mean area, density class, quality flag, narrative).
- The robustness extension shows both classical and learned methods lose the same touching-nuclei pair under heavy Gaussian blur (σ = 4.0), with the feature table correctly reflecting the object-count drop.

## Report

See `report/` for the full write-up, including methodology, figures, evaluation metrics, discussion of limitations, and answers to the assignment's reflection questions.

## License

See [LICENSE](./LICENSE).
