# EPIC-01 Test Execution Guide

## Overview
This guide provides comprehensive instructions for executing all test cases for EPIC-01 Mobile Capture Platform Features 1-4.

**Total Test Cases**: 106
- Feature 1 (One-Tap Conference Mode): 30 test cases
- Feature 2 (Background Audio Recording): 26 test cases
- Feature 3 (Real-Time Capture Indicator): 27 test cases
- Feature 4 (Camera Capture Workflow): 23 test cases

---

## Test File Structure

```
EPIC01-Test/
├── EPIC01-feature-1-test-cases.md          (One-Tap Conference Mode)
├── EPIC01-feature-2-test-cases.md          (Background Audio Recording)
├── EPIC01-feature-3-test-cases.md          (Real-Time Capture Indicator)
├── EPIC01-feature-4-test-cases.md          (Camera Capture Workflow)
└── TEST_EXECUTION_GUIDE.md                 (This file)
```

---

## 1. FEATURE 1: One-Tap Conference Mode (30 Tests)

### Purpose
Verify that conference capture sessions can be created and managed with minimal user interaction.

### Test Coverage

#### Unit Tests (12 tests)
- **Session Initialization** (3 tests)
  - TC-F1-U1.1: Successful session creation with metadata
  - TC-F1-U1.2: Permission validation before session start
  - TC-F1-U1.3: Session state machine transitions

- **Local Storage** (2 tests)
  - TC-F1-U2.1: Session data persisted locally
  - TC-F1-U2.2: Offline session creation with sync queue

- **UI State Management** (1 test)
  - TC-F1-U3.1: UI reflects active session state

#### Integration Tests (6 tests)
- **Authentication** (1 test)
  - TC-F1-I1.1: Authentication required before session creation

- **API Integration** (2 tests)
  - TC-F1-I2.1: Session API communication
  - TC-F1-I2.2: Retry logic on API failure

- **Permissions** (1 test)
  - TC-F1-I3.1: Permission request & grant flow

#### Edge Cases (6 tests)
- **Network Edge Cases** (2 tests)
  - TC-F1-E1.1: Network loss during session creation
  - TC-F1-E1.2: Slow network connection

- **Permission Edge Cases** (2 tests)
  - TC-F1-E2.1: User denies permission mid-flow
  - TC-F1-E2.2: Revoked permissions during active session

- **Device State** (2 tests)
  - TC-F1-E3.1: Low battery state
  - TC-F1-E3.2: App backgrounded unexpectedly

#### Performance Tests (6 tests)
- **Session Creation Performance** (2 tests)
  - TC-F1-P1.1: Session creation < 2 seconds
  - TC-F1-P1.2: UI responsiveness during session start

- **Memory Performance** (1 test)
  - TC-F1-P2.1: Session memory footprint

- **Database Performance** (1 test)
  - TC-F1-P3.1: Local storage write performance

### Execution

```bash
# Run all Feature 1 tests
npm test -- EPIC01-feature-1-test-cases.md

# Run specific test category
npm test -- EPIC01-feature-1-test-cases.md --grep "Unit Test"
npm test -- EPIC01-feature-1-test-cases.md --grep "Integration Test"
npm test -- EPIC01-feature-1-test-cases.md --grep "Edge Case"
npm test -- EPIC01-feature-1-test-cases.md --grep "Performance"
```

### Success Criteria
- Session creation < 2 seconds ✓
- All unit tests pass ✓
- No crashes on edge cases ✓
- Memory per session < 50KB ✓

---

## 2. FEATURE 2: Background Audio Recording (26 Tests)

### Purpose
Verify reliable audio capture in background/locked state with proper handling of interruptions and uploads.

### Test Coverage

#### Unit Tests (9 tests)
- **Audio Session Management** (3 tests)
  - TC-F2-U1.1: Start background recording successfully
  - TC-F2-U1.2: Pause and resume recording
  - TC-F2-U1.3: Audio segmentation/chunking

- **Recording State** (2 tests)
  - TC-F2-U2.1: Handle phone call interruption
  - TC-F2-U2.2: Handle Bluetooth microphone disconnection

- **Audio Quality** (1 test)
  - TC-F2-U3.1: Audio quality settings

#### Integration Tests (4 tests)
- **Background Upload** (2 tests)
  - TC-F2-I1.1: Audio chunks upload after recording stops
  - TC-F2-I1.2: Resumable upload on network failure

- **Transcription Pipeline** (1 test)
  - TC-F2-I2.1: Trigger transcription after upload

#### Edge Cases (7 tests)
- **Storage Edge Cases** (2 tests)
  - TC-F2-E1.1: Recording when storage nearly full
  - TC-F2-E1.2: Handle storage write errors

- **Audio Quality** (2 tests)
  - TC-F2-E2.1: Recording in noisy environment
  - TC-F2-E2.2: Handle audio session interruption (alarm/timer)

- **Network Edge Cases** (1 test)
  - TC-F2-E3.1: Multiple upload retries on persistent failure

#### Performance Tests (6 tests)
- **Recording Performance** (2 tests)
  - TC-F2-P1.1: Minimal CPU impact while recording
  - TC-F2-P1.2: Audio encoding performance

- **Battery Impact** (1 test)
  - TC-F2-P2.1: Battery drain rate while recording

- **Storage Performance** (1 test)
  - TC-F2-P3.1: Local cache space efficiency

### Execution

```bash
# Run all Feature 2 tests
npm test -- EPIC01-feature-2-test-cases.md

# Run with coverage
npm test -- EPIC01-feature-2-test-cases.md --coverage

# Run integration tests only
npm test -- EPIC01-feature-2-test-cases.md --grep "Integration"
```

### Success Criteria
- Recording starts in < 500ms ✓
- Audio loss < 1% ✓
- Successful recording sessions > 95% ✓
- CPU usage < 15% ✓
- Battery drain < 5% per hour ✓

---

## 3. FEATURE 3: Real-Time Capture Indicator (27 Tests)

### Purpose
Verify status indicator displays capture state accurately and updates in real-time.

### Test Coverage

#### Unit Tests (9 tests)
- **Indicator State Management** (3 tests)
  - TC-F3-U1.1: Display active recording status
  - TC-F3-U1.2: Display paused recording status
  - TC-F3-U1.3: Display error status

- **Timer & Duration** (2 tests)
  - TC-F3-U2.1: Timer accuracy and precision
  - TC-F3-U2.2: Handle long recording sessions (hours)

- **Status Transitions** (1 test)
  - TC-F3-U3.1: Smooth status transitions

#### Integration Tests (6 tests)
- **Recording Service Integration** (1 test)
  - TC-F3-I1.1: Indicator reflects recording service state

- **Upload Status Integration** (1 test)
  - TC-F3-I2.1: Indicator shows upload progress

- **Notification Integration** (1 test)
  - TC-F3-I3.1: Permission warning notifications

#### Edge Cases (6 tests)
- **Visual & Display Edge Cases** (2 tests)
  - TC-F3-E1.1: Indicator visible on lock screen
  - TC-F3-E1.2: Indicator visible in dynamic island (iOS 16+)

- **State Mismatch Edge Cases** (1 test)
  - TC-F3-E2.1: Recover from service-UI state mismatch

- **Battery & Resource Edge Cases** (1 test)
  - TC-F3-E3.1: Indicator update throttling on low battery

#### Performance Tests (6 tests)
- **Rendering Performance** (2 tests)
  - TC-F3-P1.1: Indicator re-render performance
  - TC-F3-P1.2: Memory usage for indicator

- **Update Latency** (1 test)
  - TC-F3-P2.1: Status update latency < 50ms

- **Battery Impact** (1 test)
  - TC-F3-P3.1: Indicator minimizes battery drain

### Execution

```bash
# Run all Feature 3 tests
npm test -- EPIC01-feature-3-test-cases.md

# Run performance tests
npm test -- EPIC01-feature-3-test-cases.md --grep "Performance"

# Run with visual regression testing
npm test -- EPIC01-feature-3-test-cases.md --visualRegressionTesting
```

### Success Criteria
- State update latency < 50ms ✓
- Timer accuracy ±1 second ✓
- State accuracy > 99% ✓
- Rendering efficient with < 5ms avg ✓

---

## 4. FEATURE 4: Camera Capture Workflow (23 Tests)

### Purpose
Verify image capture, classification, and metadata management workflow.

### Test Coverage

#### Unit Tests (9 tests)
- **Image Capture & Processing** (3 tests)
  - TC-F4-U1.1: Successful image capture
  - TC-F4-U1.2: Image type auto-detection
  - TC-F4-U1.3: Image compression

- **Image Preview & Retake** (2 tests)
  - TC-F4-U2.1: Image preview before save
  - TC-F4-U2.2: Retake functionality

- **Metadata & Linking** (1 test)
  - TC-F4-U3.1: Image linked to session & interaction

#### Integration Tests (4 tests)
- **Vision Pipeline Integration** (2 tests)
  - TC-F4-I1.1: Trigger OCR after image upload
  - TC-F4-I1.2: Auto-contact creation from business card

#### Edge Cases (6 tests)
- **Image Quality Edge Cases** (2 tests)
  - TC-F4-E1.1: Blurry image detection and warning
  - TC-F4-E1.2: Poor lighting detection

- **Storage & Permission Edge Cases** (2 tests)
  - TC-F4-E2.1: Camera permission denied
  - TC-F4-E2.2: Storage full when saving image

- **Camera Session Edge Cases** (1 test)
  - TC-F4-E3.1: Camera interrupted by phone call

#### Performance Tests (4 tests)
- **Camera Operation Performance** (2 tests)
  - TC-F4-P1.1: Camera launch < 1 second
  - TC-F4-P1.2: Image save time < 2 seconds

- **Memory Performance** (1 test)
  - TC-F4-P2.1: Camera memory usage

### Execution

```bash
# Run all Feature 4 tests
npm test -- EPIC01-feature-4-test-cases.md

# Run camera performance tests
npm test -- EPIC01-feature-4-test-cases.md --grep "Performance"

# Run edge case tests
npm test -- EPIC01-feature-4-test-cases.md --grep "Edge Case"
```

### Success Criteria
- Camera launch < 1 second ✓
- Image save < 2 seconds ✓
- Successful image captures > 95% ✓
- OCR-ready quality > 85% ✓

---

## Test Execution Workflow

### Step 1: Setup Test Environment

```bash
# Install dependencies
npm install

# Setup test database/mocks
npm run setup:test

# Verify environment
npm run verify:environment
```

### Step 2: Run All Tests

```bash
# Run complete test suite for all 4 features
npm test -- EPIC01-Test/

# Run with detailed output
npm test -- EPIC01-Test/ --verbose

# Run with coverage report
npm test -- EPIC01-Test/ --coverage
```

### Step 3: Review Results

```bash
# Generate test report
npm run test:report

# Generate coverage report
npm run coverage:report

# Compare against baseline
npm run test:compare-baseline
```

### Step 4: Document Results

| Feature | Unit Tests | Integration | Edge Cases | Performance | Total | Status |
|---------|-----------|-------------|-----------|-------------|-------|--------|
| Feature 1 | 12 | 6 | 6 | 6 | 30 | TBD |
| Feature 2 | 9 | 4 | 7 | 6 | 26 | TBD |
| Feature 3 | 9 | 6 | 6 | 6 | 27 | TBD |
| Feature 4 | 9 | 4 | 6 | 4 | 23 | TBD |
| **TOTAL** | **39** | **20** | **25** | **22** | **106** | TBD |

---

## Test Execution Modes

### Quick Test (5 minutes)
Tests critical happy path only:
```bash
npm test -- EPIC01-Test/ --mode=quick
```

### Standard Test (30 minutes)
All unit and integration tests:
```bash
npm test -- EPIC01-Test/ --mode=standard
```

### Full Test (60+ minutes)
All tests including performance and stress testing:
```bash
npm test -- EPIC01-Test/ --mode=full
```

### Continuous Integration
Automated testing on every commit:
```bash
npm test -- EPIC01-Test/ --mode=ci
```

---

## Key Test Metrics to Monitor

### Performance Thresholds
- ✓ Session creation: < 2 seconds
- ✓ Camera launch: < 1 second
- ✓ Image save: < 2 seconds
- ✓ Status update latency: < 50ms
- ✓ UI refresh rate: 60 FPS minimum

### Reliability Targets
- ✓ Successful recording sessions: > 95%
- ✓ Recovered interruptions: > 90%
- ✓ State accuracy: > 99%
- ✓ User-reported issues: < 2%

### Resource Usage
- ✓ CPU during recording: < 15%
- ✓ Battery drain: < 5% per hour
- ✓ Memory per session: < 50KB
- ✓ Storage cache: < 300MB

### Data Integrity
- ✓ Audio loss: < 1%
- ✓ Image corruption: 0%
- ✓ Missing metadata: 0%

---

## Test Failure Troubleshooting

### Common Issues

#### 1. Permission Tests Failing
- **Cause**: Mock permission service not initialized
- **Solution**: Run `npm run setup:test:permissions`

#### 2. Network Tests Failing
- **Cause**: Network mock not intercepting calls
- **Solution**: Verify network mock configuration in jest.setup.js

#### 3. Performance Tests Timing Out
- **Cause**: Device too slow or background processes
- **Solution**: Close unnecessary apps, run tests in isolation

#### 4. Camera Tests Failing
- **Cause**: No camera device available
- **Solution**: Use camera simulator: `npm run setup:test:camera`

---

## Continuous Integration Setup

### GitHub Actions
```yaml
# .github/workflows/test.yml
name: Test EPIC-01

on: [push, pull_request]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2
      - run: npm install
      - run: npm test -- EPIC01-Test/ --mode=ci
```

### Pre-commit Hook
```bash
#!/bin/sh
npm test -- EPIC01-Test/ --mode=quick
```

---

## Test Report Template

### Summary
- Test Run Date: [DATE]
- Environment: [DEVICE/SIMULATOR]
- Test Mode: [quick/standard/full]
- Total Duration: [TIME]

### Results
- Passed: [X] / [Y]
- Failed: [X]
- Skipped: [X]
- Coverage: [X]%

### Performance
- Average session creation: [TIME]ms
- Average camera launch: [TIME]ms
- Memory peak: [MB]
- Battery drain: [%]/hour

### Issues
- Critical: [X]
- High: [X]
- Medium: [X]
- Low: [X]

---

## Next Steps

1. **Execute all test cases** in EPIC01-Test/
2. **Document results** with coverage metrics
3. **Identify gaps** and add supplementary tests
4. **Performance tune** any failing threshold tests
5. **Integrate** into CI/CD pipeline
6. **Monitor** test metrics over time

---

## References

- Feature Specifications: `EPIC-01-Mobile-Capture-Platform/FEATURE-*.md`
- User Stories: `EPIC-01-Mobile-Capture-Platform/EPIC01-UserStories/`
- Testing Framework: Jest / React Testing Library
- Code Coverage Target: > 80%

