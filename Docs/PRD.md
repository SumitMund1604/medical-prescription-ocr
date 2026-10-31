# Medical Prescription OCR — Product Requirements Document (PRD)

**Project type:** Class project / academic demo
**Status:** Draft for review
**Last updated:** 2026-10-31

---

## Plain-English Summary

Doctors often write prescriptions by hand, and the handwriting is hard to read. We are building a small web app where a user uploads a photo of a prescription, and the app reads the image and pulls out three things:

1. **Medicine name**
2. **Dosage** (for example, "500 mg")
3. **Frequency** (for example, "twice a day")

The goal is a working class demo, not a hospital-ready product. It shows how OCR (software that reads text from images) plus simple text parsing can turn a paper prescription into structured data.

---

## Problem Statement

Patients, pharmacy staff, and students often cannot quickly read handwritten prescriptions. This causes confusion, manual re-typing, and possible medication errors. For this class project, the problem is narrowed to: **given an image of a prescription, can we automatically extract the medicine name, dosage, and frequency in a way that is easy to demo and explain?**

---

## Evidence

- Handwritten prescription recognition is an active research problem because handwriting styles vary widely and image quality is often poor (blur, lighting, angles). Source: Ameer et al., *Frontiers in Medicine*, 2026 — https://doi.org/10.3389/fmed.2026.1856485
- Printed prescriptions reach low character error rates with modern OCR, while handwritten text is much harder (roughly 4%–20% error depending on legibility). Same source.
- Public datasets exist for training and testing, e.g. RxHandBD (5,578 handwritten prescription word images) — https://data.mendeley.com/datasets/dsb5r6vskg/2 and the Kaggle "Doctor's Handwritten Prescription BD" dataset — https://www.kaggle.com/datasets/mamun1113/doctors-handwritten-prescription-bd-dataset
- **Chosen dataset:** `chinmays18/medical-prescription-dataset` on Hugging Face — 1,000 synthetic prescription images with JSON ground-truth annotations, split 800 train / 100 validation / 100 test. Link: https://huggingface.co/datasets/chinmays18/medical-prescription-dataset
- **Assumption (needs validation):** Students and demo audiences will judge the project mainly on whether the end-to-end flow works on a few clear sample images, not on clinical-grade accuracy.

---

## Proposed Solution

Build a Python web app using **Streamlit** for the interface and **Tesseract OCR / EasyOCR** for text recognition. The user uploads an image, the app preprocesses it (resize, grayscale, threshold), runs OCR, then uses simple parsing rules (regular expressions + a small medicine-name list) to identify medicine, dosage, and frequency. Results are shown in a table, and the raw OCR text is shown for transparency so the user can verify the output. For handwritten images, results are best-effort and the UI clearly warns the user to double-check.

**Why this approach:** It is free, runs locally, needs no model training, and is easy to explain in a class presentation. A deep-learning model (e.g. TrOCR) would perform better but adds training time, data needs, and complexity that are out of scope.

---

## Key Hypothesis

We believe **automatic OCR + rule-based extraction** will **reduce manual reading and typing of prescriptions** for **students and demo users**.
We'll know we're right when **the app correctly extracts medicine, dosage, and frequency from at least 7 out of 10 clear printed sample prescriptions, and produces a usable result on at least 3 out of 5 handwritten samples.**

---

## What We're NOT Building

- **A clinical or diagnostic tool** — not validated for real medical use.
- **A full EMR/EHR system** — no patient records, login, or history.
- **A trained custom handwriting model** — out of scope for one class project.
- **Medicine interaction or safety checking** — not a pharmacy safety system.
- **Support for every language or every prescription format** — English printed + simple handwritten samples only.
- **Cloud deployment at scale** — local demo is enough.

---

## Success Metrics

| Metric | Target | How Measured |
|--------|--------|--------------|
| Printed prescription extraction accuracy | ≥ 70% (7/10 samples) | Compare app output to the ground-truth JSON in the dataset test split |
| Handwritten sample usability | ≥ 60% (3/5 samples) | Output contains a plausible medicine + dosage + frequency |
| End-to-end demo time | Under 10 seconds per image | Timer during demo |
| Demo reliability | Works 9/10 tries without crashing | Manual test runs |
| Presentation clarity | Audience understands flow in < 2 minutes | Class feedback / Q&A |

---

## Open Questions

- [x] Which sample images will we use for the demo? → Hugging Face `chinmays18/medical-prescription-dataset` test split, plus a few own photos to sanity-check real-world performance.
- [ ] Do we need a medicine-name dictionary, and where do we get it?
- [ ] How much handwriting accuracy is "good enough" for grading?
- [ ] Should the app allow the user to correct extracted fields before export?
- [ ] Is a CSV/JSON export required, or is an on-screen table enough?

---

## Users & Context

**Primary User (class project)**
- **Who:** A student presenting the project, and classmates/teacher watching the demo.
- **Current behavior:** Reads the prescription manually and types the details by hand.
- **Trigger:** A prescription image needs to be turned into structured text quickly.
- **Success state:** Upload image → see clean table of medicine, dosage, frequency → explain how it works.

**Secondary (realistic) User**
- **Who:** Pharmacy staff or a patient who wants a quick readable transcription.
- **Current behavior:** Squints at handwriting, asks a colleague, or types it manually.
- **Trigger:** Receiving a handwritten prescription.
- **Success state:** A readable digital summary they can verify.

**Job to Be Done**
When I have a photo of a prescription, I want to automatically extract the medicine, dosage, and frequency, so I can understand or record it without manually deciphering the handwriting.

**Non-Users**
- Hospitals needing certified, legally valid prescription processing.
- Doctors wanting e-prescribing software.
- Users needing languages other than English in v1.

---

## Solution Detail

### Core Capabilities (MoSCoW)

| Priority | Capability | Rationale |
|----------|------------|-----------|
| Must | Upload an image (JPG/PNG) | Entry point for the whole flow |
| Must | Run OCR on printed prescriptions | Core proof of concept |
| Must | Extract medicine name, dosage, frequency | The three required output fields |
| Must | Show extracted results in a table | Easy for the audience to read |
| Must | Show raw OCR text | Builds trust; helps debug errors |
| Should | Attempt handwritten prescriptions | User requested; set expectations honestly |
| Should | Image preprocessing (grayscale, threshold, resize) | Big accuracy improvement for little effort |
| Could | Export results as CSV/JSON | Useful but not needed for demo |
| Could | Editable result fields | Lets user fix OCR mistakes |
| Won't | Model training / fine-tuning | Too much time for a class project |
| Won't | Login, database, patient history | Not relevant to the hypothesis |

### MVP Scope

A Streamlit app that:
1. Accepts one image upload.
2. Preprocesses and OCRs the image.
3. Extracts medicine, dosage, and frequency with regex/rules.
4. Displays the results and the raw text.
5. Shows a clear "verify manually" warning for handwritten input.

### User Flow

1. Open the Streamlit app.
2. Upload a prescription image.
3. App preprocesses the image and runs OCR.
4. App parses the text into medicine / dosage / frequency.
5. User sees a results table + raw OCR text.
6. User can re-upload another image if needed.

---

## Technical Approach

**Feasibility:** MEDIUM
- Printed text: HIGH confidence with Tesseract/EasyOCR + preprocessing.
- Handwriting: LOW–MEDIUM confidence; public research shows it is genuinely hard.

**Architecture Notes**
- **UI:** Streamlit (single-page app).
- **OCR:** EasyOCR as primary (better out-of-the-box), Tesseract as fallback/comparison. Both run locally and free.
- **Preprocessing:** OpenCV/Pillow — grayscale, resize, adaptive threshold, denoise.
- **Extraction:** Regex patterns for dosage (`\d+\s?(mg|ml|mcg|g|tablet|capsule)`) and frequency (`OD`, `BD`, `TDS`, `QID`, `once`, `twice daily`, etc.) + a small medicine-name list for matching.
- **Dataset:** Hugging Face `chinmays18/medical-prescription-dataset` — 1,000 synthetic images with JSON ground truth (800 train / 100 validation / 100 test). Use the test split for evaluation and the validation split for tuning parsing rules. No model training needed for the MVP.
- **No database** — everything happens per upload.

**Technical Risks**

| Risk | Likelihood | Mitigation |
|------|------------|------------|
| Handwritten text OCR is inaccurate | High | Set expectations in UI; demo printed samples first; allow manual correction |
| Poor image quality (blur, shadow) | High | Preprocessing + ask user to upload clear images |
| Medicine names misspelled by OCR | High | Fuzzy matching against a small medicine list |
| Dosage/frequency formats vary | Medium | Cover common abbreviations; document limitations |
| Tesseract install issues | Medium | Use EasyOCR as fallback; document setup steps |
| Chosen dataset is synthetic, so results may not match real prescriptions | High | Test on a few real photos too; report the gap as a limitation |

---

## Implementation Phases

| # | Phase | Description | Status | Parallel | Depends | PRP Plan |
|---|-------|-------------|--------|----------|---------|----------|
| 1 | Setup & samples | Repo setup, install libs, load HF dataset + own photos | pending | - | - | - |
| 2 | Image preprocessing | Grayscale, resize, threshold, denoise module | pending | - | 1 | - |
| 3 | OCR integration | EasyOCR + Tesseract wrappers, raw text output | pending | with 4 | 2 | - |
| 4 | Field extraction | Regex + medicine list for medicine/dosage/frequency | pending | with 3 | 2 | - |
| 5 | Streamlit UI | Upload, results table, raw text, warnings | pending | - | 3, 4 | - |
| 6 | Evaluation & report | Test on samples, record accuracy, write demo/report | pending | - | 5 | - |

### Phase Details

**Phase 1: Setup & Samples**
- **Goal:** Working environment and a small, known test set.
- **Scope:** Install Python deps (`streamlit`, `easyocr`, `pytesseract`, `opencv-python`, `pillow`, `datasets`), create folder structure, download the `chinmays18/medical-prescription-dataset` test split, and add 5–10 own printed/handwritten photos as a reality check.
- **Success signal:** `streamlit run app.py` opens a blank app; dataset test images and their ground-truth JSON are loaded and readable.

**Phase 2: Image Preprocessing**
- **Goal:** Improve OCR input quality.
- **Scope:** Grayscale, resize, adaptive threshold, denoise; before/after preview.
- **Success signal:** Preprocessed images are visibly cleaner and OCR text improves on at least one sample.

**Phase 3: OCR Integration**
- **Goal:** Convert image to text reliably.
- **Scope:** EasyOCR primary, Tesseract optional comparison; return raw text with confidence.
- **Success signal:** Printed samples return readable text; app handles missing Tesseract gracefully.

**Phase 4: Field Extraction**
- **Goal:** Turn raw text into the three required fields.
- **Scope:** Dosage/frequency regex, medicine dictionary + fuzzy match, simple line grouping.
- **Success signal:** Extracts correct fields on at least 7/10 printed samples.

**Phase 5: Streamlit UI**
- **Goal:** Make the flow demo-friendly.
- **Scope:** Upload widget, image preview, results table, raw text expander, handwritten warning, error messages.
- **Success signal:** A classmate can use it without instructions.

**Phase 6: Evaluation & Report**
- **Goal:** Prove the hypothesis and document limits.
- **Scope:** Run all samples, record pass/fail, calculate accuracy, capture screenshots, write short report.
- **Success signal:** Metrics table filled in; demo script ready.

### Parallelism Notes

Phases 3 and 4 can be built in parallel once preprocessing works, because OCR output and the parsing rules can be developed against the same sample text. Phase 5 waits for both. Phase 6 must be last.

---

## Decisions Log

| Decision | Choice | Alternatives | Rationale |
|----------|--------|--------------|-----------|
| OCR engine | EasyOCR primary, Tesseract fallback | TrOCR, PaddleOCR, cloud OCR | Free, local, easy setup; no training needed |
| UI | Streamlit | React, FastAPI, Gradio | Fastest path to a clean demo |
| Extraction method | Regex + medicine list | ML/NER model | Simple, explainable, good enough for MVP |
| Handwriting | Best-effort attempt | Ignore handwriting | User requested it, but promises are kept realistic |
| Data storage | None (per-upload) | Database | Not needed for the hypothesis |
| Dataset | Hugging Face `chinmays18/medical-prescription-dataset` | RxHandBD, Kaggle BD dataset, own photos | Ready-made images + JSON labels + train/val/test split, so no manual labeling needed |

---

## Research Summary

**Market Context**
- Prescription digitization is a real problem, especially in emerging markets where paper prescriptions are still common (ACL Industry Track, 2023 — https://aclanthology.org/2023.acl-industry.76.pdf).
- Research pipelines often combine a classifier (printed vs. handwritten) with specialized OCR engines. Best reported results are strong on printed text and weaker on handwriting.
- Public datasets are available (RxHandBD, Kaggle Doctor's Handwritten Prescription BD). We chose `chinmays18/medical-prescription-dataset` because it ships 1,000 labeled synthetic images with a ready 800/100/100 split.

**Technical Context**
- EasyOCR and Tesseract are practical for a student project: free, offline, pip-installable.
- Preprocessing (thresholding, denoising) matters a lot for phone-camera images.
- Transformer models like TrOCR achieve better handwriting accuracy but require training and heavier setup — deliberately deferred.
- Known failure modes: blurry images, unusual handwriting, medical abbreviations, and inconsistent prescription layouts.

---

## Validation Status

| Section | Status |
|---------|--------|
| Problem Statement | Validated by research; class-project scope assumed |
| User Research | Assumption — no formal interviews done |
| Technical Feasibility | Assessed — printed HIGH, handwritten LOW–MEDIUM |
| Success Metrics | Defined; dataset selected, evaluation not yet run |

---

## Recommended Next Step

Build Phase 1 (setup + load the HF dataset test split) and run a quick OCR test on 3 dataset images and 2 real photos. If printed accuracy looks reasonable, continue to the Streamlit UI; if handwriting fails badly, keep it as a "known limitation" slide instead of blocking the project.

---

*Generated: 2026-10-31*
*Status: DRAFT — needs validation*
