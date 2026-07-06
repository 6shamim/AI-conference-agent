# FEATURE-06 — Slide Extraction

## Epic
EPIC-02 — AI Transcription & Media Pipeline

---

# 1. Objective

Detect, extract, normalize, and link presentation slides from captured photos or video frames.

---

# 2. Problem Statement

Slides contain key insights but are difficult to search, summarize, or connect to transcript moments when stored only as raw photos.

---

# 3. Feature Overview

The pipeline identifies slide regions, corrects perspective, extracts text and visual metadata, and links the slide to a session timestamp.

---

# 4. Key Functionalities

## Slide Boundary Detection
Detect slide content inside captured image.

## Slide Normalization
Crop and perspective-correct the slide.

## Slide Metadata Extraction
Extract title, bullet text, chart labels, and visual type.

## Transcript Linking
Associate slide with nearest transcript timestamp/session.

---

# 5. Primary Use Cases

## Keynote slide archive
User captures slides and gets a searchable slide library.

## Slide-to-topic mapping
Slides are linked to discussion topics in summaries.

---

# 6. User Stories

## User Story 1
As a conference attendee,  
I want save clean slide images,  
so that I can review presentation content later.

### Acceptance Criteria
- Slide image is cropped and normalized
- OCR text is extracted
- Slide links to session

## User Story 2
As a summary generator,  
I want use slide content with transcript,  
so that summaries include visual context.

### Acceptance Criteria
- Slide timestamp is stored
- Slide OCR is available
- Slide is attached to session summary

---

# 7. User Workflow

1. ImageUploaded event received
2. Slide boundary detected
3. Slide crop generated
4. OCR extracted
5. Timestamp/session linked
6. SlideExtracted event emitted

---

# 8. UI / UX Requirements

- Slide gallery
- Session timeline slide markers
- Manual slide link correction

---

# 9. Technical Requirements

## Frontend
- Slide gallery
- Session timeline slide markers
- Manual slide link correction

## Backend
- Slide extraction worker
- Slide asset persistence
- Session linking service

## AI/ML
- Slide/document detection
- OCR
- Visual content classification

## Infrastructure
- Image workers
- Object storage
- Event queue

---

# 10. APIs / Integrations

| Integration / API | Purpose |
|---|---|
| GET /sessions/{id}/slides | List session slides |
| PATCH /slides/{id}/link | Correct session/timestamp link |

---

# 11. Data Model

| Entity | Fields |
|---|---|
| slide_asset | id, image_asset_id, session_id, timestamp_ms, title, text, slide_uri, confidence |

---

# 12. Security & Privacy

- Protect confidential slide content
- Respect retention policies
- Support delete/export

---

# 13. Performance Requirements

| Metric | Target |
|---|---|
| Slide extraction | <5 sec per image |
| Slide listing | <500 ms |
| Timestamp link confidence | >80% target |

---

# 14. Edge Cases

- Multiple slides in image
- Screen glare
- Presenter blocking slide
- Chart-heavy slide
- No timestamp available

---

# 15. Dependencies

- Image Enhancement
- OCR Extraction
- Transcript Segmentation

---

# 16. Risks

- Incorrect slide-session linking
- Poor chart interpretation
- Copyright/confidential conference content

---

# 17. Telemetry & Analytics

Track:
- slide_detected
- slide_extracted
- slide_linked_to_session
- slide_link_corrected

---

# 18. Success Metrics

| Metric | Goal |
|---|---|
| Slide detection accuracy | >90% clear slide photos |
| OCR extraction success | >85% |
| Manual link correction rate | <15% |

---

# 19. Future Enhancements

- Chart/table extraction
- Video frame slide extraction
- Slide deck reconstruction

---

# 20. Open Questions

- Should V1 support video-based slide capture?
- How should confidential slides be marked?

