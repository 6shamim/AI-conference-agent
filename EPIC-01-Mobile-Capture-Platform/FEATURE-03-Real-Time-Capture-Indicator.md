# FEATURE-03 — Real-Time Capture Indicator

## Epic
EPIC-01 — Mobile Capture Platform

---

# 1. Objective

Show clear, persistent capture status for user trust and compliance.

---

# 2. Problem Statement

Users need to know whether recording/capture is active, paused, failed, or uploading.

---

# 3. Feature Overview

A visible indicator displays recording state, elapsed time, upload state, permission warnings, and compliance status.

---

# 4. Key Functionalities

## Active/paused/error status
Implementation-ready behavior required for V1.

## Elapsed timer
Implementation-ready behavior required for V1.

## Upload status
Implementation-ready behavior required for V1.

## Consent state indicator
Implementation-ready behavior required for V1.

## Battery/storage warning
Implementation-ready behavior required for V1.

---

# 5. Primary Use Cases

## Use Case 1
User quickly confirms recording is active

## Use Case 2
User sees upload backlog after poor network

## Use Case 3
User notices microphone permission issue

---

# 6. User Stories

## User Story 1
As a conference attendee,
I want see capture status instantly,
so that I know whether the system is working.

### Acceptance Criteria
- User can complete the action without developer/support assistance.
- System stores required data and returns clear success/failure state.
- Errors are recoverable and logged for diagnostics.

## User Story 2
As a compliance reviewer,
I want see visible recording indicator,
so that recording behavior is transparent.

### Acceptance Criteria
- User can complete the action without developer/support assistance.
- System stores required data and returns clear success/failure state.
- Errors are recoverable and logged for diagnostics.

---

# 7. User Workflow

1. Recording begins
2. Indicator turns active
3. Timer starts
4. Upload status updates
5. Errors show actionable message

---

# 8. UI / UX Requirements

- Persistent header/banner
- Color/state changes without relying on color only
- Readable on lock screen widget
- Haptic alert on failure

---

# 9. Technical Requirements

## Frontend
SwiftUI status component; state binding to recording/upload services

## Backend
Status API optional for sync; local state primary

## AI/ML
No AI requirement

## Infrastructure
Telemetry event stream

---

# 10. APIs / Integrations

| Integration | Purpose |
|---|---|
| Recording Service | Current capture state |
| Upload Service | Queue status |
| Notification Service | Failure alerts |

---

# 11. Data Model

| Entity | Fields |
|---|---|
| CaptureStatus | session_id, state, elapsed_seconds, upload_queue_count, last_error |

---

# 12. Security & Privacy

- Do not expose sensitive content in notifications
- Show consent state when recording active

---

# 13. Performance Requirements

| Metric | Target |
|---|---|
| State update latency | <500 ms |
| Timer accuracy | ±1 sec |
| Failure alert | <3 sec |

---

# 14. Edge Cases

- State mismatch after crash
- Upload stuck
- Screen locked
- Low power mode

---

# 15. Dependencies

- Recording service
- Upload queue
- Notification permissions

---

# 16. Risks

- False active state
- Notification fatigue

---

# 17. Telemetry & Analytics

Track:
- `capture_indicator_viewed`
- `capture_state_changed`
- `capture_error_displayed`

---

# 18. Success Metrics

| Metric | Goal |
|---|---|
| State accuracy | >99% |
| User-reported missed recordings | <2% |

---

# 19. Future Enhancements

- Dynamic island/live activity support
- Apple Watch companion indicator

---

# 20. Open Questions

- Should indicator remain visible during camera capture?
