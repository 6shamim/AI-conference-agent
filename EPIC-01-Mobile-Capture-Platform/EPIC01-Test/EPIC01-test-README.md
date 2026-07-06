# EPIC-01 Mobile Capture Platform - Comprehensive Test Suite

## Document Overview

This document provides a comprehensive guide to the test suites for all 10 features of EPIC-01 (Mobile Capture Platform). It includes:

- Test architecture and execution framework
- Individual test case coverage for all 10 features
- Test matrix mapping tests to user stories and acceptance criteria
- Setup instructions and dependencies
- Execution guides for different test types
- Performance targets and success metrics

---

## Test Suite Organization

### Test Categories

```
EPIC01-Test/
├── EPIC01-feature-1-test-cases.md    (Unit, Integration, E2E)
├── EPIC01-feature-2-test-cases.md    (Unit, Integration, E2E)
├── EPIC01-feature-3-test-cases.md    (Unit, Integration, E2E)
├── EPIC01-feature-4-test-cases.md    (Unit, Integration, E2E)
├── EPIC01-feature-5-test-cases.md    (Unit, Integration, E2E)
├── EPIC01-feature-6-test-cases.md    (Unit, Integration, E2E)
├── EPIC01-feature-7-test-cases.md    (Unit, Integration, E2E)
├── EPIC01-feature-8-test-cases.md    (Unit, Integration, E2E) ✅ NEW
├── EPIC01-feature-9-test-cases.md    (Unit, Integration, E2E) ✅ NEW
├── EPIC01-feature-10-test-cases.md   (Unit, Integration, E2E) ✅ NEW
├── EPIC01-test-README.md             (This file) ✅ NEW
└── mocks/
    ├── api.ts
    ├── permissions.ts
    ├── audioRecorder.ts
    ├── camera.ts
    └── server.ts
```

---

## Test Matrix: Features 8-10

### FEATURE-08: Quick Interaction Tagging

**User Stories**: 3 (1 tag behavior scenario each)

| Test Case ID | Test Type | Acceptance Criteria | Coverage | Priority |
|---|---|---|---|---|
| F8-UNIT-001 | Unit | AC-F8-001 (Functional) | Preset tag application | High |
| F8-UNIT-002 | Unit | AC-F8-001, AC-F8-003 (Technical) | Custom tag creation | High |
| F8-UNIT-003 | Unit | AC-F8-002 (UX) | Voice note tagging | High |
| F8-UNIT-004 | Unit | AC-F8-004 (Functional) | Tag edit/remove | Medium |
| F8-INT-001 | Integration | AC-F8-003, AC-F8-005 (Technical) | Tag sync & retry | High |
| F8-INT-002 | Integration | AC-F8-005 (Technical) | Tag metadata storage | Medium |
| F8-UX-001 | User Interaction | AC-UX-001, AC-UX-002 | UI flow completeness | High |
| F8-ACC-001 | Accessibility | AC-UX-003 (Accessibility) | WCAG AA compliance | Medium |
| F8-EDGE-001 | Edge Case | AC-F8-006 (Error handling) | Error scenarios | High |

**Acceptance Criteria Mapping for Feature 8**:
```
Functional Criteria:
  ✓ Feature launches successfully (F8-UNIT-001)
  ✓ User interaction completes within expected workflow (F8-UX-001)
  ✓ System handles offline and online states (F8-INT-001)
  ✓ Errors are logged correctly (F8-EDGE-001)

UX Criteria:
  ✓ Workflow requires minimal user interaction (F8-UX-001)
  ✓ UI feedback visible within 1 second (F8-UNIT-001)
  ✓ Accessibility supported (F8-ACC-001)

Technical Criteria:
  ✓ API responses handled gracefully (F8-INT-001)
  ✓ Local persistence supported (F8-INT-002)
  ✓ Retry logic implemented (F8-INT-001)
```

---

### FEATURE-09: Push-to-Capture Mode

**User Stories**: 3 (Privacy-conscious user, Attendee recap scenarios)

| Test Case ID | Test Type | Acceptance Criteria | Coverage | Priority |
|---|---|---|---|---|
| F9-UNIT-001 | Unit | AC-F9-001 (Functional) | Press-and-hold audio | High |
| F9-UNIT-002 | Unit | AC-F9-002 (Technical) | Audio processing/upload | High |
| F9-UNIT-003 | Unit | AC-F9-001 (Functional) | Quick photo capture | Medium |
| F9-UNIT-004 | Unit | AC-F9-003 (Technical) | Note creation | High |
| F9-UNIT-005 | Unit | AC-F9-004 (UX) | Discard functionality | Medium |
| F9-INT-001 | Integration | AC-F9-002, AC-F9-003 | Offline queueing/sync | High |
| F9-INT-002 | Integration | AC-F9-003 | Permission handling | High |
| F9-UX-001 | User Interaction | AC-UX-001, AC-UX-002 | UI flow/feedback | High |
| F9-ACC-001 | Accessibility | AC-UX-003 | Accessibility features | Medium |
| F9-EDGE-001 | Edge Case | AC-F9-006 | Error scenarios | High |

**Acceptance Criteria Mapping for Feature 9**:
```
Functional Criteria:
  ✓ Feature launches successfully (F9-UNIT-001)
  ✓ User interaction completes within expected workflow (F9-UX-001)
  ✓ System handles offline and online states (F9-INT-001)
  ✓ Errors are logged correctly (F9-EDGE-001)

UX Criteria:
  ✓ Workflow requires minimal user interaction (F9-UX-001)
  ✓ UI feedback visible within 1 second (F9-UX-001)
  ✓ Accessibility supported (F9-ACC-001)

Technical Criteria:
  ✓ API responses handled gracefully (F9-INT-002)
  ✓ Local persistence supported (F9-INT-001)
  ✓ Retry logic implemented (F9-INT-001)
```

---

### FEATURE-10: Conference Session Switching

**User Stories**: 3 (Session switch, Ad hoc creation, Media reassignment)

| Test Case ID | Test Type | Acceptance Criteria | Coverage | Priority |
|---|---|---|---|---|
| F10-UNIT-001 | Unit | AC-F10-001 (Functional) | Session selection | High |
| F10-UNIT-002 | Unit | AC-F10-002 (Functional) | Calendar import | High |
| F10-UNIT-003 | Unit | AC-F10-003 (Functional) | Ad hoc creation | High |
| F10-UNIT-004 | Unit | AC-F10-004 (Functional) | Media reassignment | High |
| F10-UNIT-005 | Unit | AC-F10-005 (UX) | Auto-suggest session | Medium |
| F10-INT-001 | Integration | AC-F10-002 | Context persistence | High |
| F10-INT-002 | Integration | AC-F10-005 | Offline management | Medium |
| F10-UX-001 | User Interaction | AC-UX-001, AC-UX-002 | UI flow/header display | High |
| F10-ACC-001 | Accessibility | AC-UX-003 | Keyboard/screen reader | Medium |
| F10-EDGE-001 | Edge Case | AC-F10-006 | Error scenarios | High |

**Acceptance Criteria Mapping for Feature 10**:
```
Functional Criteria:
  ✓ Feature launches successfully (F10-UNIT-001)
  ✓ User interaction completes within expected workflow (F10-UX-001)
  ✓ System handles offline and online states (F10-INT-002)
  ✓ Errors are logged correctly (F10-EDGE-001)

UX Criteria:
  ✓ Workflow requires minimal user interaction (F10-UX-001)
  ✓ UI feedback visible within 1 second (F10-UNIT-001)
  ✓ Accessibility supported (F10-ACC-001)

Technical Criteria:
  ✓ API responses handled gracefully (F10-INT-001)
  ✓ Local persistence supported (F10-INT-002)
  ✓ Retry logic implemented (F10-INT-002)
```

---

## Test Type Breakdown - All 10 Features

### By Test Type Distribution

| Test Type | Count | Focus Area |
|---|---|---|
| Unit Tests | ~50 | Component logic, API calls, state management |
| Integration Tests | ~20 | API interactions, offline/online sync, data persistence |
| User Interaction (UX) | ~15 | UI flows, user workflows, feedback mechanisms |
| Accessibility Tests | ~10 | WCAG compliance, screen readers, keyboard nav |
| Edge Case Tests | ~15 | Error scenarios, boundary conditions, recovery |
| **Total Test Cases** | **~110** | **Complete coverage across all 10 features** |

### By Feature Distribution

| Feature | Unit | Integration | UX | Accessibility | Edge Case | Total |
|---|---|---|---|---|---|---|
| FEATURE-01: One-Tap Conference | 5 | 2 | 2 | 1 | 2 | 12 |
| FEATURE-02: Background Audio | 5 | 2 | 2 | 1 | 2 | 12 |
| FEATURE-03: Real-Time Indicator | 4 | 2 | 2 | 1 | 2 | 11 |
| FEATURE-04: Camera Capture | 5 | 2 | 2 | 1 | 2 | 12 |
| FEATURE-05: Offline Buffering | 4 | 3 | 2 | 1 | 2 | 12 |
| FEATURE-06: Battery Optimization | 4 | 2 | 2 | 1 | 2 | 11 |
| FEATURE-07: Lock Screen Controls | 4 | 2 | 2 | 1 | 2 | 11 |
| **FEATURE-08: Quick Tagging** | **5** | **2** | **1** | **1** | **1** | **10** |
| **FEATURE-09: Push-to-Capture** | **5** | **2** | **1** | **1** | **1** | **10** |
| **FEATURE-10: Session Switching** | **5** | **2** | **1** | **1** | **1** | **10** |
| **TOTAL (Features 8-10)** | **15** | **6** | **3** | **3** | **3** | **30** |

---

## Setup & Installation

### Prerequisites

```bash
# Node.js & npm
node --version  # v18+ required
npm --version   # v9+ required

# React Native CLI
npm install -g react-native-cli

# Detox CLI (for E2E testing)
npm install -g detox-cli
```

### Repository Setup

```bash
# Clone repository
git clone <repo-url>
cd EPIC-01-Mobile-Capture-Platform

# Install dependencies
npm install

# Install iOS dependencies (macOS only)
cd ios && pod install && cd ..

# Install Detox dependencies
detox build-framework-cache
```

### Test Framework Configuration

**Jest Configuration** (`jest.config.js`):
```javascript
module.exports = {
  preset: 'react-native',
  testEnvironment: 'node',
  setupFilesAfterEnv: ['<rootDir>/src/__tests__/setup.ts'],
  collectCoverageFrom: [
    'src/**/*.{ts,tsx}',
    '!src/**/*.d.ts',
    '!src/**/__tests__/**'
  ],
  coverageThreshold: {
    global: {
      branches: 80,
      functions: 90,
      lines: 85,
      statements: 85
    }
  }
};
```

**Detox Configuration** (`.detoxrc.json`):
```json
{
  "testRunner": "jest",
  "apps": {
    "ios": {
      "type": "ios.app",
      "binaryPath": "ios/build/Build/Products/Release-iphonesimulator/ConferenceApp.app",
      "build": "xcodebuild -workspace ios/ConferenceApp.xcworkspace..."
    }
  },
  "configurations": {
    "ios.sim.debug": {
      "device": { "type": "iPhone 14" },
      "app": "ios"
    }
  }
}
```

### Mock Services Setup

**MSW Server** (`mocks/server.ts`):
```typescript
import { setupServer } from 'msw/node';
import { http, HttpResponse } from 'msw';

export const server = setupServer(
  http.post('/api/tags/apply', () => {
    return HttpResponse.json({ success: true });
  }),
  http.post('/api/media/upload', () => {
    return HttpResponse.json({ id: 'media-123' });
  }),
  http.post('/api/sessions/switch', () => {
    return HttpResponse.json({ success: true });
  })
);
```

---

## Test Execution Guide

### 1. Run Unit Tests Only

```bash
# Run all unit tests
npm run test:unit

# Run specific feature (e.g., Feature 8)
npm run test:unit -- EPIC01-feature-8-test-cases.spec.ts

# Run with coverage
npm run test:unit -- --coverage

# Run in watch mode (development)
npm run test:unit -- --watch

# Generate coverage report
npm run test:unit -- --coverage --collectCoverageFrom='src/**/*.{ts,tsx}'
```

### 2. Run Integration Tests

```bash
# Run all integration tests
npm run test:integration

# Run specific feature
npm run test:integration -- EPIC01-feature-9-test-cases.spec.ts

# Run with detailed logging
npm run test:integration -- --verbose

# Run integration tests with network simulation
MOCK_NETWORK=true npm run test:integration
```

### 3. Run E2E Tests

```bash
# Build Detox framework
detox build-framework-cache

# Build the app for testing
detox build-app --configuration ios.sim.debug

# Run E2E tests
detox test --configuration ios.sim.debug --cleanup

# Run specific E2E test file
detox test e2e/tagging.e2e.ts --configuration ios.sim.debug

# Run with detailed recording
detox test --configuration ios.sim.debug --record logs
```

### 4. Run Complete Test Suite

```bash
# Run all tests (unit + integration + E2E)
npm run test:all

# Run all tests with coverage
npm run test:all -- --coverage

# Run in CI mode (single pass, no watch)
npm run test:ci

# Generate test report
npm run test:report
```

### 5. Run Accessibility Tests

```bash
# Run only accessibility tests
npm run test:a11y

# Run with accessibility audit
npm run test:a11y -- --audit

# Check WCAG compliance
npm run test:wcag
```

---

## Coverage Targets & Metrics

### Coverage Goals for Features 8-10

| Metric | Target | Features 8-10 |
|---|---|---|
| **Line Coverage** | >85% | Target: >88% |
| **Branch Coverage** | >80% | Target: >85% |
| **Function Coverage** | >90% | Target: >92% |
| **Statement Coverage** | >85% | Target: >88% |

### Performance Requirements

| Feature | Metric | Target | Test Case |
|---|---|---|---|
| F8 (Tagging) | Tag apply time | <1 sec | F8-UNIT-001 |
| F9 (Push-Capture) | Start latency | <500 ms | F9-UNIT-001 |
| F9 (Push-Capture) | Save latency | <2 sec | F9-UNIT-002 |
| F10 (Session) | Switch latency | <1 sec | F10-UNIT-001 |
| F10 (Session) | Ad hoc creation | <5 sec | F10-UNIT-003 |

### Success Metrics (From Feature PRDs)

**Feature 8 - Quick Interaction Tagging**:
- Tag adoption: >50% active users
- Average tag time: <2 sec
- Error rate: <0.5%

**Feature 9 - Push-to-Capture Mode**:
- Clip save success: >98%
- Discard rate: Tracked
- Average capture duration: 15-45 sec

**Feature 10 - Conference Session Switching**:
- Correct session attachment: >90%
- Ad hoc creation success: >95%
- Reassignment reliability: >99%

---

## CI/CD Integration

### GitHub Actions Workflow

```yaml
name: EPIC-01 Tests

on: [push, pull_request]

jobs:
  test:
    runs-on: macos-latest
    steps:
      - uses: actions/checkout@v3
      - uses: actions/setup-node@v3
        with:
          node-version: '18'
      
      - name: Install dependencies
        run: npm ci
      
      - name: Run unit tests
        run: npm run test:unit -- --coverage
      
      - name: Run integration tests
        run: npm run test:integration
      
      - name: Build E2E app
        run: detox build-app --configuration ios.sim.debug
      
      - name: Run E2E tests
        run: detox test --configuration ios.sim.debug --cleanup
      
      - name: Upload coverage
        uses: codecov/codecov-action@v3
        with:
          files: ./coverage/coverage-final.json
```

---

## Test Maintenance & Best Practices

### Test Review Cadence

- **Weekly**: Run full test suite and review failure trends
- **Bi-weekly**: Code review of new test cases
- **Monthly**: Update mocks to match API changes, refactor flaky tests
- **Quarterly**: Full test architecture review, update performance targets

### Common Issues & Solutions

| Issue | Cause | Solution |
|---|---|---|
| Flaky async tests | Insufficient timeout | Increase timeout in CI (3-5s for network tests) |
| Mock drift | API changes not reflected | Keep mock specifications in sync with backend |
| Storage conflicts | AsyncStorage not cleared | Clear storage in beforeEach hook |
| Permission errors | Mock permissions misconfigured | Use setupTests to properly mock platform permissions |
| Detox timeouts | Device overloaded | Run tests serially in CI, parallel locally |

### Test Best Practices

1. **Isolation**: Each test should be independent and not rely on others
2. **Mocking**: Always mock external APIs and platform APIs
3. **Async Handling**: Use `waitFor` with appropriate timeouts
4. **Cleanup**: Clear mocks and storage after each test
5. **Descriptive Names**: Use clear, specific test descriptions
6. **Single Assertion**: Focus each test on a single behavior
7. **Edge Cases**: Always test error paths and boundary conditions

---

## Telemetry & Analytics Integration

### Events Tracked Across Features 8-10

**Feature 8 Events**:
```
- tag_applied (tagId, tagType, duration)
- tag_removed (tagId, reason)
- custom_tag_created (tagName)
- voice_tag_created (duration)
- tag_sync_success / tag_sync_failed
```

**Feature 9 Events**:
```
- push_capture_started (captureType)
- push_capture_saved (duration, size, offline)
- push_capture_discarded (duration)
- transcription_complete (accuracy)
- transcription_failed (reason)
```

**Feature 10 Events**:
```
- session_switched (sessionId, fromSessionId)
- adhoc_session_created (title)
- media_reassigned (count, sourceSession, targetSession)
- session_suggestion_accepted / rejected
```

### Verifying Telemetry in Tests

```typescript
test('should track tag application event', async () => {
  const { getByTestId } = render(<TaggingComponent />);
  
  fireEvent.press(getByTestId('tag-preset-investor'));
  
  await waitFor(() => {
    expect(mockTelemetry.track).toHaveBeenCalledWith(
      'tag_applied',
      expect.objectContaining({
        tagId: 'tag-investor',
        tagType: 'preset',
        duration: expect.any(Number)
      })
    );
  });
});
```

---

## Test Data & Fixtures

### Sample Test Data

**Feature 8 - Tags**:
```typescript
const mockTags = {
  preset: [
    { id: 'tag-investor', label: 'Investor', type: 'preset' },
    { id: 'tag-customer', label: 'Customer', type: 'preset' },
    { id: 'tag-meeting', label: 'Meeting', type: 'preset' },
    { id: 'tag-panel', label: 'Panel', type: 'preset' },
    { id: 'tag-booth', label: 'Booth', type: 'preset' }
  ],
  custom: [
    { id: 'tag-custom-1', name: 'VIP Prospect', type: 'custom' }
  ]
};
```

**Feature 9 - Capture Clips**:
```typescript
const mockCaptures = {
  audio: {
    id: 'clip-audio-1',
    type: 'audio',
    duration: 2500,
    fileSize: 125000,
    transcription: 'Meeting notes about Q1 goals'
  },
  photo: {
    id: 'clip-photo-1',
    type: 'photo',
    fileSize: 2500000,
    timestamp: Date.now()
  }
};
```

**Feature 10 - Sessions**:
```typescript
const mockSessions = [
  {
    id: 'session-keynote',
    title: 'Keynote',
    type: 'scheduled',
    startTime: new Date('2024-01-15T09:00'),
    endTime: new Date('2024-01-15T10:00'),
    location: 'Main Hall'
  },
  {
    id: 'session-hallway-123',
    title: 'Hallway Meeting',
    type: 'adhoc',
    createdAt: Date.now()
  }
];
```

---

## Sign-Off & Release Checklist

### Pre-Release Validation (Features 8-10)

- [ ] **Unit Tests**
  - [ ] All tests pass: `npm run test:unit`
  - [ ] Coverage meets targets (>85% lines, >80% branches)
  - [ ] No console errors or warnings
  - [ ] All mocks properly cleaned up

- [ ] **Integration Tests**
  - [ ] All tests pass: `npm run test:integration`
  - [ ] Offline/online sync verified
  - [ ] API error handling validated
  - [ ] Retry logic tested

- [ ] **E2E Tests**
  - [ ] iOS tests pass: `detox test --configuration ios.sim.debug`
  - [ ] Android tests pass (if applicable)
  - [ ] No flaky tests
  - [ ] User workflows complete successfully

- [ ] **Accessibility**
  - [ ] All tests pass: `npm run test:a11y`
  - [ ] WCAG AA compliance verified
  - [ ] Screen reader compatibility tested
  - [ ] Keyboard navigation works

- [ ] **Performance**
  - [ ] All performance targets met
  - [ ] Load time acceptable
  - [ ] Memory usage reasonable
  - [ ] Battery impact minimal

- [ ] **Documentation**
  - [ ] Test cases documented
  - [ ] Test matrix complete
  - [ ] Setup instructions clear
  - [ ] Troubleshooting guide included

- [ ] **Quality Gates**
  - [ ] No high-severity bugs
  - [ ] All critical paths covered
  - [ ] Error scenarios handled
  - [ ] Telemetry verified

---

## Contact & Support

### Test Development Team

- **Lead QA Engineer**: [Name/Contact]
- **Test Automation**: [Name/Contact]
- **Mobile Testing**: [Name/Contact]

### Resources

- **Test Documentation**: See individual feature test case files
- **Mock API Reference**: `mocks/server.ts`
- **CI/CD Pipeline**: `.github/workflows/`
- **Test Reporting Dashboard**: [URL]

---

## Appendix: Complete Test File Structure

```
/EPIC01-Test
├── EPIC01-feature-1-test-cases.md
├── EPIC01-feature-2-test-cases.md
├── EPIC01-feature-3-test-cases.md
├── EPIC01-feature-4-test-cases.md
├── EPIC01-feature-5-test-cases.md
├── EPIC01-feature-6-test-cases.md
├── EPIC01-feature-7-test-cases.md
├── EPIC01-feature-8-test-cases.md          [NEW]
├── EPIC01-feature-9-test-cases.md          [NEW]
├── EPIC01-feature-10-test-cases.md         [NEW]
├── EPIC01-test-README.md                   [NEW - This Document]
│
├── __tests__/
│   ├── setup.ts                    # Jest setup
│   ├── setupTests.ts               # Test utilities
│   ├── feature-8.spec.ts
│   ├── feature-9.spec.ts
│   ├── feature-10.spec.ts
│   └── ...
│
├── e2e/
│   ├── tagging.e2e.ts
│   ├── push-to-capture.e2e.ts
│   ├── session-switching.e2e.ts
│   └── ...
│
├── mocks/
│   ├── server.ts                   # MSW setup
│   ├── api.ts                      # API mocks
│   ├── permissions.ts              # Permission mocks
│   ├── audioRecorder.ts            # Audio mocks
│   ├── camera.ts                   # Camera mocks
│   └── storage.ts                  # Storage mocks
│
├── fixtures/
│   ├── tags.json
│   ├── captures.json
│   ├── sessions.json
│   └── ...
│
└── utils/
    ├── testHelpers.ts
    ├── mockFactory.ts
    └── assertions.ts
```

---

**Document Version**: 1.0  
**Last Updated**: 2024-01-15  
**Next Review**: 2024-02-15  
**Test Coverage**: Features 8-10 (30 test cases)  
**Status**: Ready for Implementation
