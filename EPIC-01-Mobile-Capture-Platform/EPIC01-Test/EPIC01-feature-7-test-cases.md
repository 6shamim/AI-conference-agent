# EPIC01 Feature 7: Lock Screen Controls - Comprehensive Test Cases

## Feature Overview
Lock Screen Controls allow users to manage capture without unlocking the phone through pause/resume, mark moments, time display, and quick stop functionality via iOS Live Activities or Android notifications.

---

## Part 1: Unit Test Scenarios

### Test Suite: LockScreenControlsManager

#### UT-7.1: Control Button Rendering
**Objective**: Verify lock screen controls render correctly
**Prerequisites**: iOS 16.1+ (Live Activity API) or Android 12+ (notification controls)
**Test Steps**:
1. Start recording
2. Render lock screen widget/live activity
3. Verify pause button visible and tappable
4. Verify resume button state (hidden initially)
5. Verify mark button visible
6. Verify session name displayed
7. Verify elapsed time formatted (mm:ss)
8. Verify stop button with confirmation

**Expected Result**: All controls visible, properly sized, accessible

**Code Sample**:
```typescript
describe('LockScreenControlsManager', () => {
  let controlManager: LockScreenControlsManager;
  let recordingService: RecordingService;

  beforeEach(() => {
    controlManager = new LockScreenControlsManager();
    recordingService = new RecordingService();
  });

  test('should render lock screen controls correctly', async () => {
    await recordingService.startRecording({ session: 'Demo Keynote' });
    const liveActivity = await controlManager.createLiveActivity();

    expect(liveActivity).toBeDefined();
    expect(liveActivity.buttons).toContain({ action: 'pause', label: 'Pause' });
    expect(liveActivity.buttons).toContain({ action: 'mark', label: 'Mark' });
    expect(liveActivity.sessionName).toBe('Demo Keynote');
    expect(typeof liveActivity.elapsedTime).toBe('string');
  });
});
```

#### UT-7.2: Pause Button Functionality
**Objective**: Verify pause control pauses recording
**Test Steps**:
1. Recording active
2. User taps pause on lock screen
3. Verify recording state changes to paused
4. Verify pause button hidden
5. Verify resume button visible
6. Verify timestamp marker created
7. Verify pause reason logged

**Expected Result**: Recording paused, state updated, marker saved

#### UT-7.3: Resume Button Functionality
**Objective**: Verify resume control resumes recording
**Test Steps**:
1. Recording paused (via lock screen)
2. User taps resume on lock screen
3. Verify recording state changes to active
4. Verify resume button hidden
5. Verify pause button visible
6. Verify gap time calculated
7. Telemetry logged

**Expected Result**: Recording resumed, gap duration tracked

#### UT-7.4: Mark Moment Functionality
**Objective**: Verify mark button creates moment marker
**Test Steps**:
1. Recording active
2. User taps mark button on lock screen
3. Verify marker created with timestamp
4. Verify marker_type = "important"
5. Verify current session_id captured
6. Verify no recording interruption
7. Verify user receives haptic feedback

**Expected Result**: Marker saved, recording continues uninterrupted

**Code Sample**:
```typescript
test('should create marker when mark button tapped', async () => {
  const sessionId = 'session-123';
  await recordingService.startRecording({ sessionId });

  const markerTimestamp = Date.now();
  const marker = await controlManager.createMarker(sessionId, 'important');

  expect(marker).toBeDefined();
  expect(marker.session_id).toBe(sessionId);
  expect(marker.marker_type).toBe('important');
  expect(Math.abs(marker.timestamp - markerTimestamp)).toBeLessThan(100); // Within 100ms
  expect(marker.created_at).toBeLessThanOrEqual(Date.now());
});
```

#### UT-7.5: Stop Button with Confirmation
**Objective**: Verify stop requires confirmation to prevent accidental stops
**Test Steps**:
1. Recording active
2. User taps stop button
3. Confirmation dialog appears on lock screen
4. Verify "Cancel" and "Stop Recording" options
5. User taps Cancel - recording continues
6. User taps Stop - recording stops
7. Verify finalization (metrics, telemetry)

**Expected Result**: Confirmation required, accidental stops prevented

#### UT-7.6: Time Display Updates
**Objective**: Verify elapsed time updates every second
**Test Steps**:
1. Start recording at t=0
2. Verify display shows 00:00
3. Wait 5 seconds
4. Verify display shows 00:05
5. Wait 1 minute
6. Verify display shows 01:00
7. Verify format consistency

**Expected Result**: Time updates accurate, no drift

---

## Part 2: Integration Test Scenarios

### Test Suite: Lock Screen Control Pipeline

#### IT-7.1: Complete Lock Screen Workflow
**Objective**: Verify end-to-end lock screen interaction
**Prerequisites**: 
- Device locked
- Recording active
- Live Activity/Notification visible
**Test Steps**:
1. Start recording (phone unlocked)
2. Lock phone
3. Verify live activity appears on lock screen
4. Tap pause on lock screen
5. Verify app receives pause command
6. Verify recording paused
7. Verify UI updated (if unlocked mid-interaction)
8. Tap mark on lock screen
9. Verify marker saved
10. Tap resume
11. Verify recording resumes
12. Unlock phone
13. Verify UI consistent with lock screen state

**Expected Result**: All commands executed, state synchronized

**Code Sample**:
```typescript
describe('Lock Screen Control Pipeline', () => {
  let controlManager: LockScreenControlsManager;
  let recordingService: RecordingService;
  let deviceService: DeviceService;

  beforeEach(async () => {
    controlManager = new LockScreenControlsManager();
    recordingService = new RecordingService();
    deviceService = new DeviceService();
    await recordingService.startRecording({ session: 'Test Session' });
  });

  test('complete lock screen workflow', async () => {
    // Lock device
    deviceService.lockDevice();
    expect(deviceService.isLocked()).toBe(true);

    // Pause from lock screen
    let result = await controlManager.handleLockScreenAction('pause');
    expect(result.success).toBe(true);
    let state = await recordingService.getRecordingState();
    expect(state).toBe('paused');

    // Mark moment
    result = await controlManager.handleLockScreenAction('mark');
    expect(result.success).toBe(true);
    let markers = await recordingService.getSessionMarkers();
    expect(markers.length).toBe(1);

    // Resume from lock screen
    result = await controlManager.handleLockScreenAction('resume');
    expect(result.success).toBe(true);
    state = await recordingService.getRecordingState();
    expect(state).toBe('active');

    // Unlock and verify sync
    deviceService.unlockDevice();
    await new Promise(resolve => setTimeout(resolve, 500)); // UI sync
    const uiState = await controlManager.getUIState();
    expect(uiState.recordingState).toBe('active');
  });
});
```

#### IT-7.2: Multiple Markers from Lock Screen
**Objective**: Verify user can mark multiple moments
**Test Steps**:
1. Start recording
2. Mark at t=10s
3. Mark at t=30s
4. Mark at t=60s
5. Verify all 3 markers saved
6. Verify timestamps accurate
7. Verify no performance degradation
8. Review markers in app UI

**Expected Result**: All markers preserved, accessible

#### IT-7.3: Pause/Resume Cycle
**Objective**: Verify multiple pause/resume cycles work correctly
**Test Steps**:
1. Record 10s, pause, record 10s, resume, record 10s
2. Pause, mark moment, resume
3. Pause, wait 30s, resume
4. Verify total recording time accurate
5. Verify all gaps tracked
6. Verify markers still present

**Expected Result**: Accurate total duration, all state transitions logged

#### IT-7.4: Live Activity Refresh Rate
**Objective**: Verify time display and state updates at correct interval
**Test Steps**:
1. Start recording, lock device
2. Monitor live activity updates
3. Verify elapsed time updates every 1 second
4. Verify no updates more frequent than 1 second
5. Verify state changes reflected immediately (<100ms)
6. Monitor memory/battery impact

**Expected Result**: 1-second update cadence, minimal overhead

#### IT-7.5: Remote Command Handling
**Objective**: Verify lock screen commands execute safely
**Test Steps**:
1. Send pause command while network-dependent task running
2. Verify pause executes (higher priority)
3. Send resume during low battery
4. Verify resume executes with policy applied
5. Send mark while disk full
6. Verify mark still created (critical)

**Expected Result**: Commands execute with appropriate priority

---

## Part 3: Resource Monitoring Tests

### Battery Impact Tests

#### BT-7.1: Live Activity Battery Impact
**Objective**: Measure battery cost of lock screen controls
**Test Steps**:
1. Start recording, no lock screen widget
2. Monitor battery over 1 hour
3. Create lock screen live activity
4. Monitor battery over 1 hour
5. Compare drain rates

**Expected Result**: Live activity adds <0.3% per hour drain

**Code Sample**:
```typescript
test('live activity battery impact', async () => {
  const batteryMonitor = new BatteryMonitor();

  // Baseline without live activity
  const initialBattery = await batteryMonitor.getBatteryPercentage();
  await new Promise(resolve => setTimeout(resolve, 3600000)); // 1 hour
  const baselineDrain = initialBattery - await batteryMonitor.getBatteryPercentage();

  // With live activity
  const beforeActiveBattery = await batteryMonitor.getBatteryPercentage();
  await controlManager.createLiveActivity();
  await new Promise(resolve => setTimeout(resolve, 3600000)); // 1 hour
  const activeDrain = beforeActiveBattery - await batteryMonitor.getBatteryPercentage();

  const additionalDrain = activeDrain - baselineDrain;
  expect(additionalDrain).toBeLessThan(0.3); // Less than 0.3%
});
```

#### BT-7.2: Marker Creation Battery Cost
**Objective**: Verify minimal battery impact of marking
**Test Steps**:
1. Monitor battery
2. Create 100 markers rapidly
3. Measure battery drain
4. Verify <0.1% drain for 100 markers

**Expected Result**: <0.1% drain for rapid marking operations

### Memory Impact Tests

#### MT-7.1: Live Activity Memory
**Objective**: Verify live activity doesn't cause memory leaks
**Test Steps**:
1. Record baseline memory
2. Create live activity
3. Update 1000 times (1000 seconds at 1Hz update)
4. Destroy live activity
5. Verify memory recovered

**Expected Result**: <10MB peak growth, recovered after cleanup

#### MT-7.2: Marker Storage
**Objective**: Verify marker list doesn't bloat memory
**Test Steps**:
1. Create 1000 markers
2. Measure memory growth
3. Verify <5MB for 1000 markers
4. Query markers (should stay fast)

**Expected Result**: Linear memory growth, <5MB for 1000 items

### CPU Impact Tests

#### CT-7.1: Lock Screen Response Time
**Objective**: Verify lock screen controls respond quickly
**Test Steps**:
1. User taps button on lock screen
2. Measure time to execute command
3. Measure time to update UI state
4. Verify <500ms latency (perceived instant)

**Expected Result**: <500ms latency, <100ms for local state update

#### CT-7.2: Live Activity Update CPU
**Objective**: Verify time display doesn't spike CPU
**Test Steps**:
1. Monitor CPU while live activity updates
2. Measure CPU per update cycle
3. Run for 1 minute (60 updates)
4. Verify average CPU <2%

**Expected Result**: <2% average CPU for time updates

---

## Part 4: Edge Case Validation

### EC-7.1: Live Activity Expires
**Objective**: Handle live activity timeout (iOS: 8 hours)
**Test Steps**:
1. Create live activity
2. Simulate passage of 8+ hours
3. Live activity expires on lock screen
4. Verify app still recording
5. Recording still accessible in app UI
6. User can continue without lock screen control

**Expected Result**: Seamless fallback to app UI controls

### EC-7.2: Device Locked During Control Interaction
**Objective**: Handle lock event during button tap
**Test Steps**:
1. Start recording, lock device
2. Tap pause button (race condition: unlock happens mid-tap)
3. Button tap processed
4. Verify state consistent
5. No duplicate commands

**Expected Result**: Atomic operation, consistent state

### EC-7.3: Recording Stops Externally
**Objective**: Handle recording stop from app UI while lock screen active
**Test Steps**:
1. Recording active, live activity visible
2. User taps stop in app UI (device locked)
3. Live activity updates to stopped state
4. User unlocks phone
5. Verify app shows stopped state

**Expected Result**: State synchronized across lock screen and app

### EC-7.4: Session Change with Lock Screen Active
**Objective**: Handle session switch while lock screen controls visible
**Test Steps**:
1. Recording session A, live activity visible
2. User switches to session B
3. Live activity updates to show session B name
4. User taps mark on lock screen
5. Verify marker saved to session B
6. No confusion between sessions

**Expected Result**: Session context maintained, markers correct

### EC-7.5: Accidental Button Presses
**Objective**: Prevent accidental control activation
**Test Steps**:
1. Live activity visible on lock screen
2. Simulate repeated random taps
3. Verify only intended button responds
4. Verify drag-to-dismiss doesn't activate controls
5. Verify minimum tap size requirement

**Expected Result**: Accidental activation <0.1% of random taps

**Code Sample**:
```typescript
test('should prevent accidental button presses', async () => {
  const liveActivity = await controlManager.createLiveActivity();
  
  // Simulate taps at button boundaries
  const accidentalTaps = await simulateRandomTaps(1000, {
    x_range: [0, 400],
    y_range: [0, 400],
  });

  let accidentalActivations = 0;
  for (const tap of accidentalTaps) {
    const action = controlManager.getTapAction(tap.x, tap.y);
    if (action && action !== 'none') {
      accidentalActivations++;
    }
  }

  const accidentalRate = accidentalActivations / 1000;
  expect(accidentalRate).toBeLessThan(0.001); // <0.1%
});
```

### EC-7.6: No Sensitive Content on Lock Screen
**Objective**: Verify transcript not exposed on lock screen
**Test Steps**:
1. Recording active with transcript generation
2. Lock device
3. Live activity visible
4. Inspect live activity content
5. Verify only: session name, elapsed time, controls
6. Verify no transcript text
7. Verify no participant names
8. Verify no sensitive metadata

**Expected Result**: Only public information displayed

---

## Part 5: Telemetry and Logging

### Telemetry Events
```
- lock_screen_activity_created { platform, session_id }
- lock_screen_activity_destroyed { session_id, reason, duration }
- lock_control_used { action, session_id, timestamp }
  - action: pause, resume, mark, stop
- marker_created_lock_screen { session_id, timestamp, type }
- lock_screen_command_failed { action, error_code, session_id }
- recording_paused_lock_screen { session_id, timestamp }
- recording_resumed_lock_screen { session_id, timestamp }
- accidental_stop_prevented { gesture_type }
```

### Log Levels
```
DEBUG: Live activity updates, button tap detection
INFO: Control action executed, marker created
WARNING: Live activity expiration, command latency
ERROR: Command execution failed, state mismatch
```

### Metrics Dashboard
```typescript
{
  "sessionLockScreenInteractions": 12,
  "pauseCount": 4,
  "resumeCount": 4,
  "markCount": 4,
  "averageCommandLatency": 245, // ms
  "failedCommands": 0,
  "accidentalStopsBlocked": 0,
  "liveActivityDuration": 1845, // seconds
}
```

---

## Part 6: Performance Benchmarks

| Metric | Target | Tolerance |
|--------|--------|-----------|
| Live activity creation | <500ms | ±100ms |
| Live activity update | <100ms | ±20ms |
| Command execution latency | <500ms | ±100ms |
| Marker creation | <100ms | ±20ms |
| Pause/resume latency | <200ms | ±50ms |
| Live activity memory | <10MB | ±2MB |
| Battery drain (per hour) | <0.3% | ±0.1% |
| CPU overhead (updates) | <2% | ±0.5% |
| Accidental tap rate | <0.1% | ±0.05% |

---

## Part 7: Platform-Specific Tests

### iOS-Specific (Live Activities)

#### iOS-7.1: Live Activity API Availability
**Test Steps**:
1. Check iOS version >= 16.1
2. Query live activity support
3. Request necessary entitlements
4. Create activity if supported
5. Fallback if unavailable

**Expected Result**: Graceful fallback on older iOS

#### iOS-7.2: Focus Engine Integration
**Test Steps**:
1. Recording active with focus (Do Not Disturb)
2. Pause/mark/resume on lock screen
3. Verify actions work despite focus
4. Verify notification not sent (respects focus)

**Expected Result**: Controls work in any focus state

#### iOS-7.3: Dynamic Island Support
**Test Steps**:
1. Test on iPhone 14+ with Dynamic Island
2. Verify live activity expands in island
3. Verify controls accessible
4. Verify time display visible

**Expected Result**: Optimized UX on Dynamic Island devices

### Android-Specific (Notification Controls)

#### Android-7.1: Notification Control Buttons
**Test Steps**:
1. Create notification with action buttons
2. Verify pause, mark, resume actions
3. Verify actions accessible from lock screen
4. Verify foreground service running

**Expected Result**: Controls available in Android 12+

#### Android-7.2: Notification Permission
**Test Steps**:
1. App requests notification permission
2. User denies permission
3. Verify lock screen controls unavailable
4. Verify app UI controls still work
5. Show notification permission prompt in UI

**Expected Result**: Graceful degradation

---

## Part 8: Accessibility Tests

#### A11Y-7.1: VoiceOver/TalkBack Support
**Test Steps**:
1. Enable VoiceOver (iOS) or TalkBack (Android)
2. Verify buttons announced correctly
3. Verify controls navigable with gestures
4. Verify button functions clear

**Expected Result**: Fully accessible with screen readers

#### A11Y-7.2: Haptic Feedback
**Test Steps**:
1. Tap button on lock screen
2. Verify haptic feedback (vibration)
3. Verify feedback consistent with action
4. Test with haptics disabled

**Expected Result**: Haptic feedback enabled by default, disableable

---

## Part 9: Test Environment Setup

### Mock Recording Service
```typescript
class MockRecordingService {
  private state = 'idle';
  private sessionId = '';
  private startTime = 0;

  async startRecording(config: any) {
    this.sessionId = config.sessionId || `session-${Date.now()}`;
    this.state = 'active';
    this.startTime = Date.now();
  }

  async pauseRecording() { this.state = 'paused'; }
  async resumeRecording() { this.state = 'active'; }
  async stopRecording() { this.state = 'stopped'; }

  getRecordingState() { return this.state; }
  getElapsedTime() { return this.state !== 'idle' ? Date.now() - this.startTime : 0; }
}
```

### Mock Live Activity
```typescript
class MockLiveActivity {
  async createActivity(config: any) {
    return {
      id: `activity-${Date.now()}`,
      sessionName: config.sessionName,
      buttons: ['pause', 'mark', 'stop'],
      createdAt: Date.now(),
    };
  }

  async updateActivity(id: string, state: any) {
    return { success: true, updated: Date.now() };
  }

  async dismissActivity(id: string) {
    return { success: true };
  }
}
```

---

## Execution Notes
- Test on physical iOS devices (simulator limitations with Live Activities)
- Test on physical Android devices with Android 12+
- Verify with device locked and on lock screen
- Test with various screen unlock methods (Face ID, Touch ID, PIN, Pattern)
- Validate haptic feedback on devices that support it
- Test with different notification permissions states
- Measure real battery and CPU impact (not just theoretical)
- Test notification persistence across reboots
