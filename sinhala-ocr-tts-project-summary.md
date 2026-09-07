# Sinhala OCR-to-TTS Accessibility Tool — Project Summary

## Goal
Build a tool to help blind Sinhala-speaking users understand text in the world around them (signs, labels, menus, documents) by extracting Sinhala text from images and reading it aloud.

**Scope decision:** v1 is a focused **Sinhala text reader** (OCR + TTS), not a full scene-description assistant like Microsoft Seeing AI or Google Lookout. Full scene description (objects, people, navigation) is a possible future direction, not part of this build.

## Pipeline (planned)
1. Image capture (phone camera)
2. **Text detection + recognition** ← current focus
3. Post-processing / reading-order cleanup
4. Sinhala text-to-speech
5. Audio playback

---

## Decisions made so far

### Why not Florence-2 (or VLMs generally, for this task)
- Tested Florence-2-base on a real Sinhala image; it **hallucinated a fluent but completely wrong English caption** instead of reading the Sinhala text or failing visibly.
- VLMs generate text autoregressively based on learned visual-language patterns; when the script is out-of-distribution, they don't have a mechanical fallback — they produce whatever is statistically plausible, which can look confident and be totally wrong.
- Dedicated OCR systems (detection + recognition stages trained specifically on glyph shapes) are the right tool for "extract this text accurately," not general VLMs.

### OCR model research (Sept 2026 search)
Benchmarked on real-world Sri Lankan legislative documents (202-sample test set):

| Model | CER | WER | Notes |
|---|---|---|---|
| **LightOnOCR-2-1B** (fine-tuned) | 1.05% | 5.63% | Best overall, fine-tuned via QLoRA |
| Google Document AI | 2.06% | 8.26% | Best commercial, no fine-tuning needed |
| DeepSeek-OCR V1 (fine-tuned) | 3.02% | 12.27% | |
| DeepSeek-OCR V2 (fine-tuned) | 6.94% | 17.23% | |
| Surya-OCR (out of box) | 8.84% | 26.64% | Best open-source non-fine-tuned baseline |
| Tesseract v5 | 10.69% | — | |

**Caveat:** Surya performs much better on clean/synthetic Sinhala images than on real-world degraded documents — a gap that matters since our users will submit real phone photos, not scans.

### Model chosen: LightOnOCR-2-1B
- 1B-parameter end-to-end vision-language OCR model, Apache 2.0 licensed (free, commercial use OK)
- Recommend fine-tuning from the `-base` variant (per LightOn's own guidance), not the RLVR-refined variant
- Requires `transformers` installed from source (not yet in stable pip release)
- Runs efficiently even on modest GPUs (1B params)

### Dataset chosen for first fine-tuning pass
- **`Ransaka/sinhala_synthetic_ocr-large`** — 6,969 image-text pairs, Hugging Face, Apache-licensed
- Rendered using 5 Sinhala font families: Noto Sans Sinhala, Gemunu Libre, Noto Serif Sinhala, Yaldevi, Abhaya Libre
- Same dataset used in a peer-reviewed zero-shot OCR comparison paper (Sinhala/Tamil low-resource OCR study) — a known benchmark in the field
- **Known limitation:** synthetic, clean, computer-rendered text — no real-world noise (skew, blur, lighting, background clutter)

### Plan going forward
1. **Now:** Fine-tune LightOnOCR-2-1B-base on the synthetic dataset as a first pass.
2. **Next:** Build a custom real-world Sinhala dataset (actual phone photos of signs/labels/menus/documents) to fine-tune further for real-world robustness — this is the gap identified between "works on clean data" and "works on what a blind user's phone camera actually captures."
3. **Later:** Reading-order logic from bounding boxes, confidence-based fallback behavior, Sinhala TTS integration (Meta MMS is a candidate, not yet evaluated).

---
*Last updated: reflects conversation through the start of LightOnOCR-2-1B fine-tuning on synthetic data.*
