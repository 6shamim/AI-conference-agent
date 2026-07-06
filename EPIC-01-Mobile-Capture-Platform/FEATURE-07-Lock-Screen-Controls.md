# FEATURE-07 — Lock Screen Controls

## Epic
EPIC-01 — Mobile Capture Platform

---

# 1. Objective

Allow users to manage capture without unlocking the phone.

---

# 2. Problem Statement

Conference interactions happen quickly; unlocking the phone interrupts the conversation.

---

# 3. Feature Overview

Provide lock-screen controls for pause/resume, mark moment, and quick capture status where platform policies allow.

---

# 4. Key Functionalities

## Pause/resume recording
Implementation-ready behavior required for V1.

## Mark important moment
Implementation-ready behavior required for V1.

## Show elapsed time
Implementation-ready behavior required for V1.

## Show current session name
Implementation-ready behavior required for V1.

## Quick stop
Implementation-ready behavior required for V1.

---

# 5. Primary Use Cases

## Use Case 1
User pauses recording before private conversation

## Use Case 2
User marks a quote during a panel

## Use Case 3
User checks status while phone is locked

---

# 6. User Stories

## User Story 1
As a conference attendee,
I want control recording from lock screen,
so that I avoid awkward phone interaction.

### Acceptance Criteria
- User can complete the action without developer/support assistance.
- System stores required data and returns clear success/failure state.
- Errors are recoverable and logged for diagnostics.

## User Story 2
As a executive,
I want mark important moments quickly,
so that I can review them later.

### Acceptance Criteria
- User can complete the action without developer/support assistance.
- System stores required data and returns clear success/failure state.
- Errors are recoverable and logged for diagnostics.

---

# 7. User Workflow

1. Recording active
2. Lock screen widget/live activity visible
3. User taps pause/mark/resume
4. App updates recording state
5. Telemetry and timestamp markers saved

---

# 8. UI / UX Requirements

- Large controls
- Accidental tap protection
- Clear active status
- No sensitive transcript text displayed

---

# 9. Technical Requirements

## Frontend
Live Activities/Now Playing style controls where permitted; remote command handling

## Backend
Marker API; recording state update

## AI/ML
No inference at lock screen

## Infrastructure
Background event handling

---

# 10. APIs / Integrations

| Integration | Purpose |
|---|---|
| Recording Service | Pause/resume |
| Marker API | Save moment |
| Notification/Live Activity | Expose controls |

---

# 11. Data Model

| Entity | Fields |
|---|---|
| CaptureMarker | id, session_id, timestamp, marker_type, note, created_at |

---

# 12. Security & Privacy

- No sensitive content on lock screen
- Require unlocked device for destructive delete
- Honor recording consent

---

# 13. Performance Requirements

| Metric | Target |
|---|---|
| Control latency | <1 sec |
| Marker save | <500 ms local |
| Accidental stop prevention | Confirm for stop |

---

# 14. Edge Cases

- Live Activity expired
- Device locked after app termination
- Remote command unavailable

---

# 15. Dependencies

- Recording service
- iOS permissions
- Marker service

---

# 16. Risks

- Platform limitations
- Accidental pause/stop

---

# 17. Telemetry & Analytics

Track:
- `lock_control_used`
- `marker_created_lock_screen`
- `recording_paused_lock_screen`

---

# 18. Success Metrics

| Metric | Goal |
|---|---|
| Lock control success | >95% |
| Marker usage | Tracked |
| Accidental stop reports | <1% |

---

# 19. Future Enhancements

- Apple Watch controls
- AirPods tap-to-mark

---

# 20. Open Questions

- Which controls are allowed under current iOS policy?
