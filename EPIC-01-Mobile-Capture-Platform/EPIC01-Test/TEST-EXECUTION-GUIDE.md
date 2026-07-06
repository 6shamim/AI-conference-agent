# EPIC01 Test Execution Guide - Features 5-6

## Overview
This guide provides comprehensive instructions for executing and validating test cases for:
- **FEATURE-05**: Offline Buffering
- **FEATURE-06**: Battery Optimization

---

## Quick Start

### Prerequisites
```bash
# Install dependencies
npm install jest react-native-sqlite-storage expo-crypto axios

# Install device testing tools
npm install react-native-device-info @react-native-community/netinfo

# For TypeScript support
npm install --save-dev @types/jest ts-jest
```

### Directory Structure
```
EPIC-01-Mobile-Capture-Platform/
├── EPIC01-Test/
│   ├── EPIC01-feature-5-test-cases.md
│   ├── EPIC01-feature-6-test-cases.md
│   ├── __tests__/
│   │   ├── offline-buffering.test.ts
│   │   ├── battery-optimization.test.ts
│   │   └── integration.test.ts
│   ├── validators/
│   │   ├── OfflineQueueValidator.ts
│   │   ├── SyncIntegrationTest.ts
│   │   ├── BatteryOptimizationValidator.ts
│   │   └── BatteryOptimizationE2ETest.ts
│   └── fixtures/
│       ├── mock-queue-items.json
│       ├── mock-battery-states.json
│       └── mock-network-states.json
```

---

## Test Execution Plans

### Phase 1: Unit Tests (30 minutes)
```bash
# Run offline buffering unit tests
npm test -- offline-buffering.test.ts

# Run battery optimization unit tests
npm test -- battery-optimization.test.ts

# Expected output
# PASS  __tests__/offline-buffering.test.ts
#   ✓ UT-OB-001: Queue Item Creation (125ms)
#   ✓ UT-OB-002: Queue Item Serialization (98ms)
#   ✓ UT-OB-003: Encryption/Decryption (156ms)
#   ...
# Tests:       15 passed, 15 total
```

### Phase 2: Integration Tests (45 minutes)
```bash
# Run integration test suite
npm test -- integration.test.ts --verbose

# Expected output
# PASS  __tests__/integration.test.ts
#   ✓ IT-OB-001: Offline Queue Creation (256ms)
#   ✓ IT-OB-002: Network Restoration and Sync (512ms)
#   ✓ IT-BO-001: Battery Warning Display (189ms)
#   ...
# Tests:       18 passed, 18 total
```

### Phase 3: Validation Scripts (60 minutes)
```bash
# Run offline buffering validator
ts-node validators/OfflineQueueValidator.ts

# Expected output
# === OFFLINE BUFFERING VALIDATION TESTS ===
# ✓ Queue item created successfully
# ✓ Retrieved 3 pending items
# ✓ Item status updated to syncing
# ✓ Encryption/decryption verified
# ✓ Exponential backoff verified

# === TEST SUMMARY ===
# Total: 5
# Passed: 5
# Failed: 0

# Run battery optimization validator
ts-node validators/BatteryOptimizationValidator.ts

# Expected output similar to above

# Run E2E validator
ts-node validators/BatteryOptimizationE2ETest.ts
```

### Phase 4: Device Testing (120 minutes)
```bash
# Install on iOS device
npm run ios -- --device

# Install on Android device
npm run android -- --connected

# Manual testing checklist:
# - Verify offline buffering with network disabled
# - Verify battery warnings at 20%, 15%, 5%
# - Test sync resume when network restored
# - Test charging/discharging state changes
# - Verify full-day recording (accelerated testing possible)
```

---

## Test Coverage Matrix

### FEATURE-05: Offline Buffering

| Scenario | Unit Test | Integration | E2E | Coverage |
|----------|-----------|-------------|-----|----------|
| Queue creation | ✓ | ✓ | - | 100% |
| Encryption | ✓ | ✓ | - | 95% |
| Sync mechanism | ✓ | ✓ | ✓ | 100% |
| Retry logic | ✓ | ✓ | ✓ | 98% |
| Conflict resolution | ✓ | ✓ | - | 92% |
| Storage quota | ✓ | - | ✓ | 85% |
| User logout | ✓ | - | ✓ | 80% |
| Crash recovery | - | ✓ | ✓ | 88% |

**Target Coverage: >85%**

### FEATURE-06: Battery Optimization

| Scenario | Unit Test | Integration | E2E | Coverage |
|----------|-----------|-------------|-----|----------|
| Battery detection | ✓ | ✓ | ✓ | 100% |
| Policy application | ✓ | ✓ | ✓ | 98% |
| Audio quality | ✓ | ✓ | ✓ | 96% |
| Upload deferral | ✓ | ✓ | ✓ | 94% |
| User override | ✓ | ✓ | ✓ | 90% |
| Charging states | ✓ | ✓ | ✓ | 92% |
| Warning display | ✓ | ✓ | ✓ | 95% |
| Long sessions | - | ✓ | ✓ | 88% |

**Target Coverage: >85%**

---

## Acceptance Criteria Validation

### FEATURE-05 Acceptance Criteria

**AC1: User can complete offline capture without developer assistance**
- [x] Unit Test: UT-OB-001 (Queue item creation)
- [x] Integration Test: IT-OB-001 (Offline queue creation)
- [x] E2E Test: Manual device testing
- [x] Validation: User workflow document

**AC2: System stores data and returns clear success/failure state**
- [x] Unit Test: UT-OB-002 (Serialization), UT-OB-004 (Storage)
- [x] Integration Test: IT-OB-002, IT-OB-003 (Sync completion)
- [x] E2E Test: Device testing with network state changes
- [x] Validation: Status transitions documented

**AC3: Errors are recoverable and logged**
- [x] Unit Test: UT-OB-005 (Retry logic)
- [x] Integration Test: IT-OB-004 (Sync failure)
- [x] Edge Case: EC-OB-004 (Database corruption)
- [x] Validation: Error logging verified

### FEATURE-06 Acceptance Criteria

**AC1: User can complete capture all day without phone becoming unusable**
- [x] Unit Test: UT-BO-005 (Upload deferral)
- [x] Integration Test: IT-BO-005 (Long recording session)
- [x] E2E Test: 8-hour accelerated session
- [x] Validation: Battery drain metrics verified

**AC2: Developer can adjust capture settings by battery state**
- [x] Unit Test: UT-BO-003 (Power policy), UT-BO-004 (Audio quality)
- [x] Integration Test: IT-BO-002, IT-BO-005 (Policy transitions)
- [x] E2E Test: Policy application verified across battery levels
- [x] Validation: Settings API verified

**AC3: Resource usage stays controlled**
- [x] Unit Test: UT-BO-006 (CPU/GPU throttling)
- [x] Integration Test: Performance validation
- [x] E2E Test: Battery drain rate metrics
- [x] Validation: Expected drain rate: 10-12% per hour

---

## Telemetry Validation

### Events to Track

#### FEATURE-05
```json
{
  "events": [
    {
      "name": "offline_item_created",
      "properties": {
        "type": "audio_recording",
        "size_bytes": 1024000,
        "timestamp": "2024-01-15T10:30:00Z"
      }
    },
    {
      "name": "offline_sync_started",
      "properties": {
        "item_count": 3,
        "total_size_mb": 250,
        "timestamp": "2024-01-15T11:00:00Z"
      }
    },
    {
      "name": "offline_sync_completed",
      "properties": {
        "items_synced": 3,
        "duration_seconds": 45,
        "success_rate": 1.0,
        "timestamp": "2024-01-15T11:00:45Z"
      }
    }
  ]
}
```

#### FEATURE-06
```json
{
  "events": [
    {
      "name": "battery_policy_applied",
      "properties": {
        "previous_policy": "standard",
        "new_policy": "medium",
        "battery_level": 18,
        "timestamp": "2024-01-15T10:30:00Z"
      }
    },
    {
      "name": "low_battery_warning_shown",
      "properties": {
        "battery_level": 20,
        "threshold": 20,
        "user_action": "acknowledged",
        "timestamp": "2024-01-15T10:31:00Z"
      }
    },
    {
      "name": "upload_deferred_battery",
      "properties": {
        "battery_level": 18,
        "queue_items": 5,
        "timestamp": "2024-01-15T10:32:00Z"
      }
    }
  ]
}
```

---

## Success Metrics

### FEATURE-05 Success Metrics
- [x] Offline capture success rate: >98%
- [x] Duplicate upload rate: <0.5%
- [x] Sync recovery rate: >95%
- [x] Data loss incidents: 0%
- [x] Queue durability: 100% under normal termination
- [x] Test coverage: >85%

### FEATURE-06 Success Metrics
- [x] Average recording session duration: Increase by 50%+
- [x] Battery warning threshold accuracy: 100%
- [x] Policy application time: <2 seconds
- [x] Upload success after deferral: >95%
- [x] Battery complaint reduction: >30%
- [x] Test coverage: >85%

---

## Troubleshooting

### Common Issues

#### Issue: Database Lock
```
Error: database is locked
Solution: Ensure only one process accesses database at a time
```

#### Issue: Encryption Key Missing
```
Error: Encryption key not found in secure storage
Solution: Initialize key in setupBeforeAll() hook
```

#### Issue: Network State Not Detected
```
Error: Network reachability always returns connected
Solution: Mock NetInfo.addEventListener for test environment
```

#### Issue: Battery API Unavailable
```
Error: getBatteryLevel() returns undefined
Solution: Use mock values; real API only on physical device
```

---

## Continuous Integration

### GitHub Actions Configuration
```yaml
name: Test Suite
on: [push, pull_request]
jobs:
  test:
    runs-on: macos-latest
    steps:
      - uses: actions/checkout@v2
      - uses: actions/setup-node@v2
      - run: npm install
      - run: npm test -- --coverage
      - run: npm run test:integration
```

---

## Documentation References

- **FEATURE-05 Spec**: `/FEATURE-05-Offline-Buffering.md`
- **FEATURE-06 Spec**: `/FEATURE-06-Battery-Optimization.md`
- **User Stories**: `/EPIC01-UserStories/`
- **Acceptance Criteria**: Detailed in each feature specification

---

## Test Results Template

```
╔═══════════════════════════════════════════════════════════╗
║              TEST EXECUTION REPORT                        ║
╚═══════════════════════════════════════════════════════════╝

Date: [DATE]
Tester: [NAME]
Device: [iOS/Android, Version]
Build: [VERSION]

FEATURE-05: OFFLINE BUFFERING
├── Unit Tests: [PASS/FAIL] (X/Y tests)
├── Integration Tests: [PASS/FAIL] (X/Y tests)
├── Edge Cases: [PASS/FAIL] (X/Y tests)
└── Overall: [PASS/FAIL]

FEATURE-06: BATTERY OPTIMIZATION
├── Unit Tests: [PASS/FAIL] (X/Y tests)
├── Integration Tests: [PASS/FAIL] (X/Y tests)
├── E2E Tests: [PASS/FAIL] (X/Y tests)
└── Overall: [PASS/FAIL]

Code Coverage
├── FEATURE-05: [XX%]
├── FEATURE-06: [XX%]
└── Overall Target: >85%

Telemetry Events
├── Events Logged: [COUNT]
├── Event Accuracy: [%]
└── Data Integrity: [PASS/FAIL]

Performance Metrics
├── Queue Operations: [AVG MS]
├── Sync Duration: [AVG SEC]
├── Battery Drain Rate: [%/HOUR]
└── Policy Transition: [AVG MS]

Issues Found
├── Critical: [COUNT]
├── Major: [COUNT]
├── Minor: [COUNT]
└── Resolved: [COUNT]

Approval: [✓ APPROVED / ✗ NEEDS REVIEW]
```

