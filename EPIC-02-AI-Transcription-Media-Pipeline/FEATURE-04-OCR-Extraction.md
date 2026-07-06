# FEATURE-04 — OCR Extraction

## Epic
EPIC-02 — AI Transcription & Media Pipeline

---

# 1. Objective

Extract machine-readable text from badges, slides, business cards, booth signs, and documents.

---

# 2. Problem Statement

Important conference information often exists only in images and is unusable unless converted into structured text.

---

# 3. Feature Overview

OCR extraction receives image assets, preprocesses them, extracts text, detects document type, and stores raw and normalized OCR outputs.

---

# 4. Key Functionalities

## Image Preprocessing
Improve contrast, crop boundaries, correct rotation.

## Text Extraction
Extract raw text and bounding boxes.

## Document Type Detection
Classify badge, slide, business card, booth, or document.

## Structured Field Extraction
Extract name, title, company, email, URL, session title where possible.

---

# 5. Primary Use Cases

## Badge capture
User photographs a badge and gets a contact draft.

## Slide capture
User photographs a slide and links it to session notes.

---

# 6. User Stories

## User Story 1
As a conference attendee,  
I want capture text from a photo,  
so that I do not manually type contact or session information.

### Acceptance Criteria
- OCR result is generated after image upload
- Text is editable
- Extracted fields include confidence scores

## User Story 2
As a AI pipeline,  
I want receive normalized OCR data,  
so that contacts and sessions can be enriched automatically.

### Acceptance Criteria
- OCR output links to image_asset_id
- Bounding boxes are stored
- Structured fields are available via API

---

# 7. User Workflow

1. ImageUploaded event received
2. Image preprocessing executed
3. OCR model extracts text
4. Document type classified
5. Structured fields extracted
6. OCRCompleted event emitted

---

# 8. UI / UX Requirements

- Show OCR preview
- Allow field correction
- Show confidence warnings

---

# 9. Technical Requirements

## Frontend
- Show OCR preview
- Allow field correction
- Show confidence warnings

## Backend
- OCR worker
- Image preprocessing service
- OCR result persistence

## AI/ML
- OCR model
- Document classifier
- Named entity extraction

## Infrastructure
- Image processing queue
- Model endpoint
- Object storage

---

# 10. APIs / Integrations

| Integration / API | Purpose |
|---|---|
| POST /images/{id}/ocr | Run OCR |
| GET /images/{id}/ocr | Retrieve OCR result |
| PATCH /ocr/{id} | Correct extracted fields |

---

# 11. Data Model

| Entity | Fields |
|---|---|
| ocr_result | id, image_asset_id, raw_text, document_type, confidence |
| ocr_field | id, ocr_result_id, field_name, field_value, confidence, bounding_box |

---

# 12. Security & Privacy

- Protect personal data from badges/business cards
- Encrypt OCR results
- Audit manual edits

---

# 13. Performance Requirements

| Metric | Target |
|---|---|
| OCR processing time | <5 sec per image |
| OCR retrieval | <300 ms |
| Field extraction target | >85% for clear images |

---

# 14. Edge Cases

- Blurred image
- Glare
- Partial badge
- Handwritten notes
- Non-English text

---

# 15. Dependencies

- Image Upload
- Image Enhancement
- Contact Intelligence

---

# 16. Risks

- Incorrect contact creation
- OCR inaccuracies from poor images
- Privacy concerns for captured attendees

---

# 17. Telemetry & Analytics

Track:
- ocr_started
- ocr_completed
- ocr_failed
- ocr_field_corrected
- document_type_detected

---

# 18. Success Metrics

| Metric | Goal |
|---|---|
| OCR success | >95% clear images |
| Structured field accuracy | >85% target |
| Manual correction rate | <25% |

---

# 19. Future Enhancements

- Multilingual OCR
- QR code extraction
- Batch image OCR

---

# 20. Open Questions

- Which OCR provider is preferred?
- Should OCR run on-device for privacy-sensitive captures?

