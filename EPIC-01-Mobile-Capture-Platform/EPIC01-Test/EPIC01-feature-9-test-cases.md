# EPIC01 Feature 9 - Push-to-Capture Mode - Comprehensive Test Cases

## Feature Overview
Feature-09 provides a privacy-preserving alternative to continuous recording by allowing users to press-and-hold or tap controls to capture short audio/image notes only when needed.

---

## Test Execution Environment

### Setup & Dependencies
- **Platform**: React Native (iOS/Android)
- **Test Framework**: Jest, Detox (E2E), React Testing Library
- **Mock Services**:
  - React Native Audio Recorder
  - Camera API mock
  - MSW for API mocking
- **Dependencies**:
  - `@react-native-community/audio-toolkit`
  - `react-native-camera`
  - `jest`
  - `detox`

### Environment Variables
```
TEST_ENV=development
API_BASE_URL=http://localhost:3000
MAX_CLIP_DURATION=60000
DEFAULT_CLIP_DURATION=30000
```

---

## Unit Test Scenarios

### 1.1 Press-and-Hold Audio Capture

**Test Case ID**: F9-UNIT-001  
**Acceptance Criteria**: AC-F9-001 (Functional - Feature launches successfully)

**Description**: Verify press-and-hold gesture captures audio correctly

**Test Steps**:
```typescript
describe('Push-to-Capture Mode - Press-and-Hold Audio', () => {
  beforeEach(() => {
    jest.clearAllMocks();
  });

  test('should start audio capture on press', async () => {
    const { getByTestId } = render(
      <PushToCaptureComponent sessionId="session-123" />
    );
    
    const captureButton = getByTestId('btn-capture');
    fireEvent.pressIn(captureButton);
    
    // Verify recorder started
    await waitFor(() => {
      expect(mockAudioRecorder.start).toHaveBeenCalledWith(
        expect.objectContaining({
          duration: 60000, // Default max
          quality: 'high'
        })
      );
    });
  });

  test('should show countdown timer during capture', async () => {
    const { getByTestId, getByText } = render(
      <PushToCaptureComponent sessionId="session-123" />
    );
    
    fireEvent.pressIn(getByTestId('btn-capture'));
    
    await waitFor(() => {
      expect(getByText('0:01')).toBeTruthy();
    });
  });

  test('should stop capture on release', async () => {
    const { getByTestId } = render(
      <PushToCaptureComponent sessionId="session-123" />
    );
    
    const captureButton = getByTestId('btn-capture');
    fireEvent.pressIn(captureButton);
    
    await waitFor(() => {
      expect(mockAudioRecorder.start).toHaveBeenCalled();
    });
    
    fireEvent.pressOut(captureButton);
    
    // Verify recorder stopped
    await waitFor(() => {
      expect(mockAudioRecorder.stop).toHaveBeenCalled();
    });
  });

  test('should auto-stop when max duration reached', async () => {
    const { getByTestId, getByText } = render(
      <PushToCaptureComponent 
        sessionId="session-123"
        maxDuration={5000}
      />
    );
    
    fireEvent.pressIn(getByTestId('btn-capture'));
    
    // Wait for max duration
    await waitFor(() => {
      expect(getByText('0:05')).toBeTruthy();
    }, { timeout: 6000 });
    
    // Should auto-stop
    await waitFor(() => {
      expect(mockAudioRecorder.stop).toHaveBeenCalled();
    });
  });

  test('should provide haptic feedback on press and release', async () => {
    const { getByTestId } = render(
      <PushToCaptureComponent sessionId="session-123" />
    );
    
    const captureButton = getByTestId('btn-capture');
    fireEvent.pressIn(captureButton);
    
    expect(mockHaptics.trigger).toHaveBeenCalledWith('medium');
    
    fireEvent.pressOut(captureButton);
    
    expect(mockHaptics.trigger).toHaveBeenCalledWith('light');
  });
});
```

**Expected Result**:
- Audio capture starts on press
- Countdown timer displayed
- Capture stops on release
- Max duration auto-stop works
- Haptic feedback provided

---

### 1.2 Audio Processing and Upload

**Test Case ID**: F9-UNIT-002  
**Acceptance Criteria**: AC-F9-002 (Technical - API responses handled gracefully)

**Description**: Verify audio processing and API upload

**Test Steps**:
```typescript
test('should process audio after capture', async () => {
  const { getByTestId } = render(
    <PushToCaptureComponent sessionId="session-123" />
  );
  
  fireEvent.pressIn(getByTestId('btn-capture'));
  await new Promise(resolve => setTimeout(resolve, 2000));
  fireEvent.pressOut(getByTestId('btn-capture'));
  
  // Verify audio processing
  await waitFor(() => {
    expect(mockAudioProcessor.normalize).toHaveBeenCalled();
    expect(mockAudioProcessor.compress).toHaveBeenCalled();
  });
});

test('should upload audio clip to backend', async () => {
  const { getByTestId } = render(
    <PushToCaptureComponent sessionId="session-123" />
  );
  
  fireEvent.pressIn(getByTestId('btn-capture'));
  await new Promise(resolve => setTimeout(resolve, 1500));
  fireEvent.pressOut(getByTestId('btn-capture'));
  
  await waitFor(() => {
    expect(mockMediaAPI.uploadClip).toHaveBeenCalledWith(
      expect.objectContaining({
        sessionId: 'session-123',
        mediaType: 'audio',
        duration: expect.any(Number),
        fileName: expect.stringMatching(/^capture-\d+\.m4a$/)
      })
    );
  });
});

test('should handle upload failure with retry', async () => {
  let callCount = 0;
  mockMediaAPI.uploadClip.mockImplementation(() => {
    callCount++;
    if (callCount < 2) {
      return Promise.reject({ status: 500 });
    }
    return Promise.resolve({ success: true });
  });
  
  const { getByTestId } = render(
    <PushToCaptureComponent sessionId="session-123" />
  );
  
  fireEvent.pressIn(getByTestId('btn-capture'));
  fireEvent.pressOut(getByTestId('btn-capture'));
  
  await waitFor(() => {
    expect(mockMediaAPI.uploadClip).toHaveBeenCalledTimes(2);
  }, { timeout: 5000 });
});
```

**Expected Result**:
- Audio processed correctly
- Uploaded to backend
- Retry logic handles failures

---

### 1.3 Quick Photo Capture

**Test Case ID**: F9-UNIT-003  
**Acceptance Criteria**: AC-F9-001 (Functional)

**Description**: Verify quick photo capture functionality

**Test Steps**:
```typescript
test('should capture photo on tap', async () => {
  const { getByTestId } = render(
    <PushToCaptureComponent 
      sessionId="session-123"
      captureType="photo"
    />
  );
  
  fireEvent.press(getByTestId('btn-photo-capture'));
  
  await waitFor(() => {
    expect(mockCamera.takePhoto).toHaveBeenCalled();
  });
});

test('should save photo to local gallery', async () => {
  const { getByTestId } = render(
    <PushToCaptureComponent 
      sessionId="session-123"
      captureType="photo"
    />
  );
  
  fireEvent.press(getByTestId('btn-photo-capture'));
  
  await waitFor(() => {
    expect(mockCameraRoll.save).toHaveBeenCalledWith(
      expect.any(String),
      'photo'
    );
  });
});
```

**Expected Result**:
- Photo captured successfully
- Saved to gallery
- Metadata attached

---

### 1.4 Immediate Note Creation

**Test Case ID**: F9-UNIT-004  
**Acceptance Criteria**: AC-F9-003 (Technical - Local persistence supported)

**Description**: Verify quick note creation after capture

**Test Steps**:
```typescript
test('should create note from captured clip', async () => {
  const { getByTestId, getByText } = render(
    <PushToCaptureComponent sessionId="session-123" />
  );
  
  // Capture audio
  fireEvent.pressIn(getByTestId('btn-capture'));
  await new Promise(resolve => setTimeout(resolve, 2000));
  fireEvent.pressOut(getByTestId('btn-capture'));
  
  // Wait for transcription
  await waitFor(() => {
    expect(mockTranscriptionAPI.transcribe).toHaveBeenCalled();
  });
  
  // Wait for note creation
  await waitFor(() => {
    expect(mockNoteAPI.create).toHaveBeenCalledWith(
      expect.objectContaining({
        sessionId: 'session-123',
        type: 'quick-capture',
        content: expect.stringMatching(/\w+/) // Has transcribed text
      })
    );
  });
});

test('should generate summary for captured clip', async () => {
  const mockClip = { id: 'clip-123', duration: 2000 };
  const { getByTestId } = render(
    <PushToCaptureComponent sessionId="session-123" />
  );
  
  fireEvent.pressIn(getByTestId('btn-capture'));
  fireEvent.pressOut(getByTestId('btn-capture'));
  
  await waitFor(() => {
    expect(mockSummaryAPI.generate).toHaveBeenCalledWith(
      expect.objectContaining({
        clipId: expect.any(String),
        sessionId: 'session-123'
      })
    );
  });
});
```

**Expected Result**:
- Note created from clip
- Transcription generated
- Summary produced

---

### 1.5 Discard Functionality

**Test Case ID**: F9-UNIT-005  
**Acceptance Criteria**: AC-F9-004 (UX - Accessibility supported)

**Description**: Verify quick discard option for captures

**Test Steps**:
```typescript
test('should show discard option after capture', async () => {
  const { getByTestId, getByText } = render(
    <PushToCaptureComponent sessionId="session-123" />
  );
  
  fireEvent.pressIn(getByTestId('btn-capture'));
  fireEvent.pressOut(getByTestId('btn-capture'));
  
  await waitFor(() => {
    expect(getByTestId('btn-discard')).toBeTruthy();
    expect(getByText('Discard')).toBeTruthy();
  });
});

test('should discard capture on button tap', async () => {
  const { getByTestId, queryByTestId } = render(
    <PushToCaptureComponent sessionId="session-123" />
  );
  
  fireEvent.pressIn(getByTestId('btn-capture'));
  fireEvent.pressOut(getByTestId('btn-capture'));
  
  await waitFor(() => {
    fireEvent.press(getByTestId('btn-discard'));
  });
  
  // Verify capture cleared
  expect(queryByTestId('capture-preview')).toBeFalsy();
});

test('should track discard rate', async () => {
  const { getByTestId } = render(
    <PushToCaptureComponent sessionId="session-123" />
  );
  
  fireEvent.pressIn(getByTestId('btn-capture'));
  fireEvent.pressOut(getByTestId('btn-capture'));
  
  fireEvent.press(getByTestId('btn-discard'));
  
  await waitFor(() => {
    expect(mockTelemetry.track).toHaveBeenCalledWith(
      'push_capture_discarded',
      expect.any(Object)
    );
  });
});
```

**Expected Result**:
- Discard option visible
- Capture discarded correctly
- Telemetry tracked

---

## Integration Test Scenarios

### 2.1 Offline Capture and Sync

**Test Case ID**: F9-INT-001  
**Acceptance Criteria**: AC-F9-002 (Technical - Retry logic implemented)

**Description**: Verify offline capture queueing and sync

**Test Steps**:
```typescript
describe('Push-to-Capture - Offline Sync', () => {
  test('should queue capture when offline', async () => {
    mockNetworkAPI.setOnline(false);
    
    const { getByTestId } = render(
      <PushToCaptureComponent sessionId="session-123" />
    );
    
    fireEvent.pressIn(getByTestId('btn-capture'));
    fireEvent.pressOut(getByTestId('btn-capture'));
    
    // Verify queued locally
    const queue = await AsyncStorage.getItem('capture-queue');
    expect(JSON.parse(queue)).toHaveLength(1);
  });

  test('should sync queued captures when online', async () => {
    mockNetworkAPI.setOnline(false);
    
    const { getByTestId, rerender } = render(
      <PushToCaptureComponent sessionId="session-123" />
    );
    
    fireEvent.pressIn(getByTestId('btn-capture'));
    fireEvent.pressOut(getByTestId('btn-capture'));
    
    // Go online
    mockNetworkAPI.setOnline(true);
    rerender(<PushToCaptureComponent sessionId="session-123" />);
    
    await waitFor(() => {
      expect(mockMediaAPI.uploadClip).toHaveBeenCalled();
    }, { timeout: 5000 });
  });

  test('should preserve capture metadata during offline queue', async () => {
    mockNetworkAPI.setOnline(false);
    
    const { getByTestId } = render(
      <PushToCaptureComponent sessionId="session-123" />
    );
    
    fireEvent.pressIn(getByTestId('btn-capture'));
    await new Promise(resolve => setTimeout(resolve, 1500));
    fireEvent.pressOut(getByTestId('btn-capture'));
    
    const queue = await AsyncStorage.getItem('capture-queue');
    const captured = JSON.parse(queue)[0];
    
    expect(captured).toMatchObject({
      sessionId: 'session-123',
      duration: expect.any(Number),
      timestamp: expect.any(Number)
    });
  });
});
```

**Expected Result**:
- Captures queued offline
- Synced when online
- Metadata preserved

---

### 2.2 Permission Handling

**Test Case ID**: F9-INT-002  
**Acceptance Criteria**: AC-F9-003 (Technical - API responses handled)

**Description**: Verify permission requests and handling

**Test Steps**:
```typescript
test('should request microphone permission before audio capture', async () => {
  mockPermissions.check.mockResolvedValueOnce('denied');
  
  const { getByTestId, getByText } = render(
    <PushToCaptureComponent sessionId="session-123" />
  );
  
  fireEvent.pressIn(getByTestId('btn-capture'));
  
  await waitFor(() => {
    expect(mockPermissions.request).toHaveBeenCalledWith(
      expect.stringMatching(/RECORD_AUDIO|MICROPHONE/)
    );
  });
});

test('should prevent capture if permission denied', async () => {
  mockPermissions.check.mockResolvedValueOnce('denied');
  mockPermissions.request.mockResolvedValueOnce('denied');
  
  const { getByTestId, getByText } = render(
    <PushToCaptureComponent sessionId="session-123" />
  );
  
  fireEvent.pressIn(getByTestId('btn-capture'));
  
  await waitFor(() => {
    expect(getByText(/permission.*denied/i)).toBeTruthy();
    expect(mockAudioRecorder.start).not.toHaveBeenCalled();
  });
});

test('should request camera permission for photo capture', async () => {
  mockPermissions.check.mockResolvedValueOnce('denied');
  
  const { getByTestId } = render(
    <PushToCaptureComponent 
      sessionId="session-123"
      captureType="photo"
    />
  );
  
  fireEvent.press(getByTestId('btn-photo-capture'));
  
  await waitFor(() => {
    expect(mockPermissions.request).toHaveBeenCalledWith(
      expect.stringMatching(/CAMERA/)
    );
  });
});
```

**Expected Result**:
- Permissions requested correctly
- Capture prevented if denied
- User informed of permission requirements

---

## User Interaction Tests

### 3.1 UI Flow - Capture and Save

**Test Case ID**: F9-UX-001  
**Acceptance Criteria**: AC-UX-001 (UX - Minimal user interaction)

**Description**: End-to-end flow for capturing and saving

**Test Steps**:
```typescript
describe('UI Flow - Push-to-Capture', () => {
  test('should display prominent capture button', async () => {
    const { getByTestId } = render(
      <PushToCaptureComponent sessionId="session-123" />
    );
    
    const captureButton = getByTestId('btn-capture');
    expect(captureButton).toBeTruthy();
    
    const styles = window.getComputedStyle(captureButton.parentElement);
    expect(styles.position).toBe('relative');
  });

  test('should show visual feedback during capture', async () => {
    const { getByTestId, getByText } = render(
      <PushToCaptureComponent sessionId="session-123" />
    );
    
    fireEvent.pressIn(getByTestId('btn-capture'));
    
    await waitFor(() => {
      expect(getByText('Recording...')).toBeTruthy();
      expect(getByTestId('recording-indicator')).toHaveProp('animated', true);
    });
  });

  test('should display countdown timer with proper formatting', async () => {
    const { getByTestId, getByText } = render(
      <PushToCaptureComponent 
        sessionId="session-123"
        maxDuration={5000}
      />
    );
    
    fireEvent.pressIn(getByTestId('btn-capture'));
    
    await waitFor(() => {
      expect(getByText('0:01')).toBeTruthy();
    });
  });

  test('should show save/discard options after capture', async () => {
    const { getByTestId, getByText } = render(
      <PushToCaptureComponent sessionId="session-123" />
    );
    
    fireEvent.pressIn(getByTestId('btn-capture'));
    fireEvent.pressOut(getByTestId('btn-capture'));
    
    await waitFor(() => {
      expect(getByText('Save')).toBeTruthy();
      expect(getByText('Discard')).toBeTruthy();
    });
  });

  test('should provide preview of captured content', async () => {
    const { getByTestId, getByText } = render(
      <PushToCaptureComponent sessionId="session-123" />
    );
    
    fireEvent.pressIn(getByTestId('btn-capture'));
    await new Promise(resolve => setTimeout(resolve, 1500));
    fireEvent.pressOut(getByTestId('btn-capture'));
    
    await waitFor(() => {
      expect(getByTestId('capture-preview')).toBeTruthy();
      expect(getByText(/duration:/i)).toBeTruthy();
    });
  });
});
```

**Expected Result**:
- Clear UI for capture control
- Visual feedback during recording
- Timer displayed
- Save/discard options shown
- Preview available

---

### 3.2 Accessibility and Keyboard Navigation

**Test Case ID**: F9-ACC-001  
**Acceptance Criteria**: AC-UX-003 (UX - Accessibility supported)

**Description**: Verify accessibility features

**Test Steps**:
```typescript
describe('Accessibility - Push-to-Capture Mode', () => {
  test('should have accessible labels for capture controls', async () => {
    const { getByLabelText } = render(
      <PushToCaptureComponent sessionId="session-123" />
    );
    
    expect(getByLabelText('Press and hold to capture audio')).toBeTruthy();
    expect(getByLabelText('Capture photo')).toBeTruthy();
  });

  test('should announce capture state to screen reader', async () => {
    const { getByTestId, getByText } = render(
      <PushToCaptureComponent sessionId="session-123" />
    );
    
    fireEvent.pressIn(getByTestId('btn-capture'));
    
    await waitFor(() => {
      expect(getByTestId('aria-live-region')).toHaveTextContent('Recording');
    });
  });

  test('should support keyboard activation for photo capture', async () => {
    const { getByTestId } = render(
      <PushToCaptureComponent 
        sessionId="session-123"
        captureType="photo"
      />
    );
    
    const button = getByTestId('btn-photo-capture');
    fireEvent.keyDown(button, { key: 'Enter' });
    
    expect(mockCamera.takePhoto).toHaveBeenCalled();
  });

  test('should have sufficient color contrast for recording indicator', async () => {
    const { getByTestId } = render(
      <PushToCaptureComponent sessionId="session-123" />
    );
    
    fireEvent.pressIn(getByTestId('btn-capture'));
    
    const indicator = getByTestId('recording-indicator');
    const styles = window.getComputedStyle(indicator);
    
    expect(getContrastRatio(styles.color, styles.backgroundColor)).toBeGreaterThan(4.5);
  });
});
```

**Expected Result**:
- All controls have accessible labels
- Screen reader announcements work
- Keyboard navigation supported
- Color contrast meets standards

---

## Edge Case Validation

### 4.1 Error Scenarios

**Test Case ID**: F9-EDGE-001  
**Acceptance Criteria**: AC-F9-006 (Functional - Errors logged correctly)

**Description**: Verify error handling for edge cases

**Test Steps**:
```typescript
describe('Edge Cases - Push-to-Capture Error Scenarios', () => {
  test('should handle audio capture with no microphone available', async () => {
    mockAudioRecorder.start.mockRejectedValueOnce({
      message: 'No microphone available'
    });
    
    const { getByTestId, getByText } = render(
      <PushToCaptureComponent sessionId="session-123" />
    );
    
    fireEvent.pressIn(getByTestId('btn-capture'));
    
    await waitFor(() => {
      expect(getByText(/microphone.*not.*available/i)).toBeTruthy();
    });
  });

  test('should handle capture interruption gracefully', async () => {
    const { getByTestId } = render(
      <PushToCaptureComponent sessionId="session-123" />
    );
    
    fireEvent.pressIn(getByTestId('btn-capture'));
    await new Promise(resolve => setTimeout(resolve, 500));
    
    // Simulate system interrupt
    fireEvent(document, 'appBackground');
    
    // Should save state
    const savedState = await AsyncStorage.getItem('capture-state');
    expect(savedState).toBeTruthy();
  });

  test('should handle storage full scenario', async () => {
    mockStorage.getAvailableSpace.mockResolvedValueOnce(100); // 100 bytes
    
    const { getByTestId, getByText } = render(
      <PushToCaptureComponent sessionId="session-123" />
    );
    
    fireEvent.pressIn(getByTestId('btn-capture'));
    fireEvent.pressOut(getByTestId('btn-capture'));
    
    await waitFor(() => {
      expect(getByText(/storage.*full/i)).toBeTruthy();
    });
  });

  test('should handle low battery during capture', async () => {
    mockDeviceAPI.getBatteryLevel.mockResolvedValueOnce(5);
    
    const { getByTestId, getByText } = render(
      <PushToCaptureComponent sessionId="session-123" />
    );
    
    fireEvent.pressIn(getByTestId('btn-capture'));
    
    await waitFor(() => {
      expect(getByText(/low battery/i)).toBeTruthy();
    });
  });

  test('should handle upload timeout', async () => {
    mockMediaAPI.uploadClip.mockImplementationOnce(
      () => new Promise((_, reject) =>
        setTimeout(() => reject({ message: 'Timeout' }), 31000)
      )
    );
    
    const { getByTestId, getByText } = render(
      <PushToCaptureComponent sessionId="session-123" />
    );
    
    fireEvent.pressIn(getByTestId('btn-capture'));
    fireEvent.pressOut(getByTestId('btn-capture'));
    
    await waitFor(() => {
      expect(getByText(/timeout/i)).toBeTruthy();
    }, { timeout: 35000 });
  });

  test('should handle failed transcription gracefully', async () => {
    mockTranscriptionAPI.transcribe.mockRejectedValueOnce({
      status: 400,
      message: 'Invalid audio format'
    });
    
    const { getByTestId, getByText } = render(
      <PushToCaptureComponent sessionId="session-123" />
    );
    
    fireEvent.pressIn(getByTestId('btn-capture'));
    fireEvent.pressOut(getByTestId('btn-capture'));
    
    await waitFor(() => {
      // Should create note anyway without transcription
      expect(mockNoteAPI.create).toHaveBeenCalledWith(
        expect.objectContaining({
          content: 'Audio capture - no transcription'
        })
      );
    });
  });
});
```

**Expected Result**:
- All edge cases handled without crashes
- Meaningful error messages shown
- Graceful degradation when possible

---

## Test Code Samples

### Sample Mock Audio Recorder
```typescript
// mocks/audioRecorder.ts
export const mockAudioRecorder = {
  start: jest.fn().mockResolvedValue({ success: true }),
  stop: jest.fn().mockResolvedValue({ filePath: '/tmp/audio.m4a' }),
  pause: jest.fn().mockResolvedValue({ success: true }),
  resume: jest.fn().mockResolvedValue({ success: true }),
  getDuration: jest.fn().mockResolvedValue(2000),
  deleteRecording: jest.fn().mockResolvedValue({ success: true }),
};
```

### Sample E2E Test
```typescript
// e2e/push-to-capture.e2e.ts
describe('Push-to-Capture E2E', () => {
  it('should capture and save audio clip', async () => {
    await element(by.id('btn-capture')).multiTap();
    
    await waitFor(() => 
      element(by.text('Recording...')).toBeVisible()
    ).withTimeout(1000);
    
    // Simulate 2-second press
    await new Promise(resolve => setTimeout(resolve, 2000));
    
    await element(by.id('btn-capture')).multiTap();
    
    await waitFor(() =>
      element(by.text('Save')).toBeVisible()
    ).withTimeout(2000);
    
    await element(by.text('Save')).tap();
    
    await expect(element(by.text('Clip saved'))).toBeVisible();
  });
});
```

---

## Test Execution Guide

### Running Tests
```bash
npm run test:unit -- EPIC01-feature-9-test-cases.spec.ts
npm run test:integration -- EPIC01-feature-9-test-cases.spec.ts
detox test e2e/push-to-capture.e2e.ts --configuration ios.sim.debug
```

### Coverage Target
- **Line Coverage**: >85%
- **Branch Coverage**: >80%
- **Function Coverage**: >90%

---

## Sign-Off Checklist

- [ ] All unit tests pass
- [ ] Integration tests pass
- [ ] E2E tests pass (iOS & Android)
- [ ] Accessibility tests pass
- [ ] Edge cases validated
- [ ] Performance targets met
- [ ] Telemetry verified
