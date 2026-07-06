# FEATURE-03 — Speaker Diarization

## Epic
EPIC-02 — AI Transcription & Media Pipeline

---

# 1. Objective

Identify speaker turns and assign speaker labels to transcript segments.

---

# 2. Problem Statement

Conference conversations and panels lose meaning when transcripts do not distinguish between speakers.

---

# 3. Feature Overview

Diarization processes audio and transcript timestamps to detect speaker changes, assign speaker labels, and support later identity resolution.

---

# 4. Key Functionalities

## Speaker Turn Detection
Detect when the active speaker changes.

## Speaker Label Assignment
Assign Speaker 1, Speaker 2, etc. to transcript segments.

## Confidence Scoring
Attach confidence level to speaker labels.

## Manual Correction Support
Allow corrected speaker labels to override AI labels.

---

# 5. Primary Use Cases

## Panel discussion
Different speakers are separated in the transcript.

## One-on-one meeting
User and contact are separated for follow-up generation.

---

# 6. User Stories

## User Story 1
As a conference attendee,  
I want know who said what,  
so that I can create accurate notes and quotes.

### Acceptance Criteria
- Transcript segments display speaker label
- Speaker changes are timestamped
- Low-confidence segments are flagged

## User Story 2
As a review user,  
I want correct speaker labels,  
so that future summaries use accurate speaker attribution.

### Acceptance Criteria
- Manual edits persist
- Edits update related transcript segments
- Edited labels are audit logged

---

# 7. User Workflow

1. TranscriptCompleted event received
2. Diarization worker fetches audio
3. Model detects speaker turns
4. Speaker labels mapped to segments
5. Confidence scores stored
6. DiarizationCompleted event emitted

---

# 8. UI / UX Requirements

- Speaker label UI
- Inline correction control
- Low-confidence highlighting

---

# 9. Technical Requirements

## Frontend
- Speaker label UI
- Inline correction control
- Low-confidence highlighting

## Backend
- Diarization worker
- Speaker segment mapping
- Correction API

## AI/ML
- Speaker diarization model
- Voice activity detection
- Speaker embedding clustering

## Infrastructure
- Audio processing workers
- Temporary processing storage

---

# 10. APIs / Integrations

| Integration / API | Purpose |
|---|---|
| GET /transcripts/{id}/speakers | Retrieve speaker-labeled transcript |
| PATCH /transcripts/{id}/speakers | Correct speaker labels |

---

# 11. Data Model

| Entity | Fields |
|---|---|
| speaker | id, transcript_id, label, resolved_contact_id, confidence |
| speaker_turn | id, speaker_id, start_ms, end_ms, confidence |

---

# 12. Security & Privacy

- Treat voiceprints as sensitive biometric-adjacent data
- Do not store reusable voice embeddings unless explicitly enabled
- Restrict speaker correction access

---

# 13. Performance Requirements

| Metric | Target |
|---|---|
| Diarization processing time | <1.5x audio duration |
| Speaker turn save latency | <1 sec per batch |
| Manual correction save | <500 ms |

---

# 14. Edge Cases

- Overlapping speakers
- Audience questions
- Poor microphone quality
- Music/background noise
- Speaker count unknown

---

# 15. Dependencies

- Streaming Transcription
- Audio Ingestion Service
- Identity Agent

---

# 16. Risks

- Incorrect attribution damages trust
- Voice privacy sensitivity
- Overlapping speech reduces accuracy

---

# 17. Telemetry & Analytics

Track:
- diarization_started
- speaker_turn_detected
- speaker_confidence_low
- speaker_label_corrected

---

# 18. Success Metrics

| Metric | Goal |
|---|---|
| Speaker turn detection quality | >85% target in clean audio |
| Manual correction rate | <20% of sessions |
| Low-confidence flag accuracy | >90% precision |

---

# 19. Future Enhancements

- Known speaker voice enrollment
- Panel agenda-based speaker matching
- Live diarization

---

# 20. Open Questions

- Should user voice be auto-identified?
- Can voice embeddings be retained under privacy policy?

