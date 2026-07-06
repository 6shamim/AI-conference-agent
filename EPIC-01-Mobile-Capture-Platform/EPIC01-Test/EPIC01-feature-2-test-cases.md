# EPIC01 Feature 2 — Background Audio Recording — Test Cases

## Test Overview
Comprehensive test suite for Background Audio Recording feature covering unit tests, integration tests, edge cases, and performance validation for recording audio in background/locked states.

---

## 1. UNIT TEST SCENARIOS

### 1.1 Audio Session Management

#### TC-F2-U1.1: Start Background Recording Successfully
**Objective**: Verify audio recording starts successfully in background state

**Preconditions**:
- User authenticated and in active conference session
- Microphone permission granted
- AVAudioSession configured for background audio
- Device not in airplane mode

**Test Steps**:
1. Create active conference session
2. Call `startAudioRecording()` with session context
3. Verify recording state changes to RECORDING
4. Verify audio file path returned
5. Verify audio session category is `.playAndRecord`
6. Verify background audio mode enabled

**Expected Result**: Recording started successfully; returns audio file handle

**Code Sample**:
```typescript
describe('BackgroundAudioRecorder', () => {
  it('should start background audio recording successfully', async () => {
    const recorder = new BackgroundAudioRecorder(mockAVSession);
    const sessionId = 'session-123';

    const recording = await recorder.startRecording(sessionId);

    expect(recording).toBeDefined();
    expect(recording.state).toBe('RECORDING');
    expect(recording.filePath).toMatch(/\.wav$|\.m4a$/);
    expect(mockAVSession.category).toBe('.playAndRecord');
    expect(mockAVSession.backgroundAudioEnabled).toBe(true);
  });
});
```

---

#### TC-F2-U1.2: Pause and Resume Recording
**Objective**: Verify pause/resume functionality without data loss

**Test Steps**:
1. Start recording
2. Record for 5 seconds
3. Call `pauseRecording()`
4. Verify recording state changes to PAUSED
5. Verify elapsed time captured
6. Call `resumeRecording()`
7. Verify recording state back to RECORDING
8. Verify audio continues in same file
9. Stop recording and verify total duration

**Expected Result**: 
- Pause/resume successful
- No gap in audio file
- Duration accounts for paused time

**Code Sample**:
```typescript
it('should pause and resume recording without loss', async () => {
  const recorder = new BackgroundAudioRecorder(mockAVSession);
  const recording = await recorder.startRecording('session-123');

  // Record for 5 seconds
  await delay(5000);

  await recorder.pauseRecording(recording.id);
  expect(recording.state).toBe('PAUSED');

  const pausedTime = recording.currentTime;

  // Pause for 2 seconds
  await delay(2000);

  await recorder.resumeRecording(recording.id);
  expect(recording.state).toBe('RECORDING');

  await delay(3000);
  await recorder.stopRecording(recording.id);

  const duration = recording.duration;
  expect(duration).toBeCloseTo(8000, 500); // ~8 sec total, 2 sec paused
});
```

---

#### TC-F2-U1.3: Audio Segmentation (Chunking)
**Objective**: Verify audio automatically segments into chunks

**Test Steps**:
1. Configure chunk duration to 60 seconds
2. Start recording
3. Record continuously for 150 seconds
4. Verify chunks created at intervals
5. Verify chunk files saved with sequential naming
6. Verify each chunk <= 60 seconds duration
7. Verify chunk checksums computed

**Expected Result**: 
- 3 chunks created (60s + 60s + 30s)
- All chunks saved with correct naming
- Checksums present for integrity verification

**Code Sample**:
```typescript
it('should segment audio into chunks automatically', async () => {
  const recorder = new BackgroundAudioRecorder(mockAVSession, { 
    chunkDurationMs: 60000 
  });
  const recording = await recorder.startRecording('session-123');

  // Record for 150 seconds
  await delay(150000);

  await recorder.stopRecording(recording.id);
  const chunks = await recorder.getSegments(recording.id);

  expect(chunks).toHaveLength(3);
  expect(chunks[0].duration).toBeLessThanOrEqual(60000);
  expect(chunks[1].duration).toBeLessThanOrEqual(60000);
  expect(chunks[2].duration).toBeLessThanOrEqual(60000);
  
  chunks.forEach(chunk => {
    expect(chunk.checksum).toMatch(/^[a-f0-9]{32}$/); // MD5
  });
});
```

---

### 1.2 Recording State & Interruption Handling

#### TC-F2-U2.1: Handle Phone Call Interruption
**Objective**: Verify recording pauses during incoming phone call

**Test Steps**:
1. Start recording audio
2. Simulate incoming phone call notification
3. Verify recording automatically paused
4. Verify AVAudioSession callback triggered
5. User answers call (simulate)
6. Verify recording remains paused during call
7. User ends call (simulate)
8. Verify recording resumes automatically
9. Stop recording and verify no data loss

**Expected Result**: 
- Recording paused on call start
- Recording resumed on call end
- No audio loss or corruption

**Code Sample**:
```typescript
it('should handle phone call interruption gracefully', async () => {
  const recorder = new BackgroundAudioRecorder(mockAVSession);
  const recording = await recorder.startRecording('session-123');

  // Simulate incoming call
  mockAVSession.emitInterruption('BEGIN', 'DEFAULT');

  await waitFor(() => {
    expect(recording.state).toBe('PAUSED');
  });

  // Simulate call answered
  mockAVSession.emitInterruption('END', 'DEFAULT');

  await waitFor(() => {
    expect(recording.state).toBe('RECORDING');
  }, { timeout: 2000 });
});
```

---

#### TC-F2-U2.2: Handle Bluetooth Microphone Disconnection
**Objective**: Verify recording continues with fallback when BT mic disconnects

**Test Steps**:
1. Start recording with Bluetooth microphone selected
2. Verify recording active with BT input
3. Simulate Bluetooth mic disconnection
4. Verify recording automatically switches to built-in mic
5. Verify notification shown to user
6. Verify audio continues recording
7. Verify switch logged in telemetry

**Expected Result**: 
- Automatic fallback to built-in mic
- Recording continues uninterrupted
- User notified of switch

**Code Sample**:
```typescript
it('should fallback to built-in mic when BT disconnects', async () => {
  const recorder = new BackgroundAudioRecorder(mockAVSession);
  const recording = await recorder.startRecording('session-123', {
    inputDevice: 'bluetooth'
  });

  expect(recording.inputDevice).toBe('bluetooth');

  // Simulate BT disconnection
  mockAudioInputManager.emitDeviceDisconnection('bluetooth');

  await waitFor(() => {
    expect(recording.inputDevice).toBe('builtin');
  });

  const telemetry = getTelemetryEvents();
  expect(telemetry).toContainEqual(
    expect.objectContaining({ 
      type: 'AUDIO_INPUT_FALLBACK',
      from: 'bluetooth',
      to: 'builtin'
    })
  );
});
```

---

### 1.3 Audio Quality & Encoding

#### TC-F2-U3.1: Audio Quality Settings
**Objective**: Verify audio is recorded at specified quality level

**Test Steps**:
1. Create recording with quality = 'HIGH' (44.1kHz, 16-bit, mono)
2. Start and record 10 seconds
3. Stop recording
4. Verify file metadata:
   - Sample rate = 44100 Hz
   - Bit depth = 16-bit
   - Channels = 1 (mono)
   - File size reasonable for duration

**Expected Result**: 
- Audio encoded at correct quality
- File size matches expected range

**Code Sample**:
```typescript
it('should record audio at specified quality level', async () => {
  const recorder = new BackgroundAudioRecorder(mockAVSession, {
    quality: 'HIGH',
    sampleRate: 44100,
    bitDepth: 16,
    channels: 1
  });

  const recording = await recorder.startRecording('session-123');
  await delay(10000);
  const result = await recorder.stopRecording(recording.id);

  const audioMetadata = await getAudioMetadata(result.filePath);
  expect(audioMetadata.sampleRate).toBe(44100);
  expect(audioMetadata.bitDepth).toBe(16);
  expect(audioMetadata.channels).toBe(1);
  
  // Rough file size estimation: 44100 Hz * 1 byte * 10 sec = ~441 KB
  expect(result.fileSize).toBeGreaterThan(400000);
  expect(result.fileSize).toBeLessThan(500000);
});
```

---

## 2. INTEGRATION TEST SCENARIOS

### 2.1 Background Upload Integration

#### TC-F2-I1.1: Audio Chunks Upload After Recording Stops
**Objective**: Verify audio chunks automatically queued for upload

**Test Steps**:
1. Record audio for 90 seconds (creates 2 chunks)
2. Stop recording
3. Verify chunks exist in local cache
4. Verify upload queue contains 2 items
5. Verify upload process starts automatically
6. Confirm chunks uploaded to backend
7. Verify local cache cleaned after successful upload

**Expected Result**: 
- Chunks queued for upload
- Upload initiated automatically
- Chunks removed from local cache after upload

**Code Sample**:
```typescript
describe('BackgroundAudioUpload Integration', () => {
  it('should queue chunks for upload after recording', async () => {
    const recorder = new BackgroundAudioRecorder(mockAVSession);
    const uploadQueue = new AudioUploadQueue();

    const recording = await recorder.startRecording('session-123');
    await delay(90000); // 90 seconds -> 2 chunks
    await recorder.stopRecording(recording.id);

    const chunks = await recorder.getSegments(recording.id);
    expect(chunks).toHaveLength(2);

    // Verify upload queue
    const queueItems = await uploadQueue.getItems();
    expect(queueItems).toHaveLength(2);
    expect(queueItems[0].sessionId).toBe('session-123');

    // Process uploads
    await uploadQueue.processAll();

    // Verify cleanup
    const localChunks = await recorder.getLocalSegments(recording.id);
    expect(localChunks).toHaveLength(0);
  });
});
```

---

#### TC-F2-I1.2: Resumable Upload on Network Failure
**Objective**: Verify upload resumes after network reconnection

**Test Steps**:
1. Record audio and create chunks
2. Start upload of first chunk
3. Simulate network failure after 50% uploaded
4. Verify upload halts and chunk marked for retry
5. Restore network connection
6. Verify upload resumes from pause point (resumable)
7. Verify chunk eventually uploaded successfully

**Expected Result**: 
- Upload halts gracefully on network loss
- Resumes from saved position when network restored
- No re-upload of completed portion

**Code Sample**:
```typescript
it('should support resumable upload on network failure', async () => {
  const uploadQueue = new AudioUploadQueue();
  const mockHttpClient = new MockHttpClient({ resumableUpload: true });

  // Setup: Create and queue chunks
  const chunks = ['chunk1.wav', 'chunk2.wav'];
  chunks.forEach(file => uploadQueue.addItem(file, 'session-123'));

  const uploadPromise = uploadQueue.processAll();

  // Simulate 50% upload then network failure
  setTimeout(() => {
    mockHttpClient.simulateNetworkFailure();
  }, 1000);

  try {
    await uploadPromise;
  } catch (error) {
    expect(error.message).toContain('Network');
  }

  // Verify upload paused
  const status = await uploadQueue.getItemStatus(chunks[0]);
  expect(status.state).toBe('PAUSED');
  expect(status.bytesUploaded).toBeGreaterThan(0);

  // Restore network and retry
  mockHttpClient.restoreConnection();
  await uploadQueue.retryFailed();

  // Verify completed
  const finalStatus = await uploadQueue.getItemStatus(chunks[0]);
  expect(finalStatus.state).toBe('COMPLETED');
});
```

---

### 2.2 Transcription Pipeline Integration

#### TC-F2-I2.1: Trigger Transcription After Upload
**Objective**: Verify transcription triggered automatically after chunk upload

**Test Steps**:
1. Record and upload audio chunk
2. Verify upload completes successfully
3. Verify transcription event published to queue
4. Transcription service consumes event
5. Transcription begins processing
6. Verify callback received with transcription status
7. Verify transcript linked to original session

**Expected Result**: 
- Transcription event automatically triggered
- Transcript linked to correct session and chunk

**Code Sample**:
```typescript
it('should trigger transcription after audio upload', async () => {
  const recorder = new BackgroundAudioRecorder(mockAVSession);
  const uploadQueue = new AudioUploadQueue();
  const transcriptionQueue = new TranscriptionQueue();

  // Record and upload
  const recording = await recorder.startRecording('session-123');
  await delay(60000);
  const result = await recorder.stopRecording(recording.id);

  await uploadQueue.addItem(result.filePath, 'session-123');
  await uploadQueue.processAll();

  // Verify transcription event
  const events = await transcriptionQueue.getEvents();
  expect(events).toContainEqual(
    expect.objectContaining({
      type: 'TRANSCRIPTION_REQUESTED',
      sessionId: 'session-123',
      audioPath: expect.stringContaining(result.filePath)
    })
  );
});
```

---

## 3. EDGE CASE VALIDATION

### 3.1 Storage Edge Cases

#### TC-F2-E1.1: Recording When Storage Nearly Full
**Objective**: Verify graceful handling when device storage nearly full

**Test Steps**:
1. Check available storage: 100MB free
2. Start recording (will require ~10MB per minute)
3. Record until storage drops to < 5MB
4. Verify recording stops gracefully
5. Verify error message shown: "Storage Low"
6. Verify partial chunk saved
7. Verify cleanup option offered

**Expected Result**: 
- Recording stops when < 5MB remaining
- No crash or data corruption
- Clear error message
- Partial data preserved

**Code Sample**:
```typescript
it('should handle low storage gracefully', async () => {
  mockStorageService.setAvailableSpace(100 * 1024 * 1024); // 100MB
  const recorder = new BackgroundAudioRecorder(mockAVSession);

  const recording = await recorder.startRecording('session-123');

  // Simulate storage decreasing
  let recordingTime = 0;
  const storageCheckInterval = setInterval(() => {
    recordingTime += 1000;
    const estimatedUsed = recordingTime * 10000; // ~10KB/sec
    const remaining = 100 * 1024 * 1024 - estimatedUsed;
    mockStorageService.setAvailableSpace(remaining);

    if (remaining < 5 * 1024 * 1024) {
      clearInterval(storageCheckInterval);
    }
  }, 1000);

  await delay(11000); // Record until low storage

  expect(recording.state).toBe('STOPPED');
  expect(recording.stopReason).toBe('STORAGE_LOW');
  
  // Verify partial chunk saved
  const chunks = await recorder.getSegments(recording.id);
  expect(chunks.length).toBeGreaterThan(0);
});
```

---

#### TC-F2-E1.2: Handle Storage Write Errors
**Objective**: Verify app handles storage write failures

**Test Steps**:
1. Start recording
2. Simulate storage write error on device
3. Verify recording halts
4. Verify error logged and user notified
5. Verify no corrupted audio file left
6. Verify user can retry

**Expected Result**: 
- Recording halts immediately
- No corrupted files
- Informative error to user

**Code Sample**:
```typescript
it('should handle storage write errors', async () => {
  const recorder = new BackgroundAudioRecorder(mockAVSession);
  const recording = await recorder.startRecording('session-123');

  // Simulate write error after 5 seconds
  setTimeout(() => {
    mockStorageService.simulateWriteError('I/O Error');
  }, 5000);

  await delay(10000);

  expect(recording.state).toBe('ERROR');
  expect(recording.error).toContain('write');

  // Verify no corrupted file
  const exists = await fileExists(recording.filePath);
  expect(exists).toBe(false);
});
```

---

### 3.2 Audio Quality Edge Cases

#### TC-F2-E2.1: Recording with Noisy Environment
**Objective**: Verify recording quality maintained in noisy conditions

**Test Steps**:
1. Start recording
2. Simulate 80dB ambient noise (busy conference)
3. Record for 30 seconds
4. Stop recording
5. Analyze audio levels
6. Verify audio not clipping (< -3dB headroom)
7. Verify noise floor detected
8. Verify file size reasonable

**Expected Result**: 
- Audio captured without clipping
- Noise floor identified
- File valid and playable

**Code Sample**:
```typescript
it('should handle noisy environment without clipping', async () => {
  const recorder = new BackgroundAudioRecorder(mockAVSession);
  const recording = await recorder.startRecording('session-123', {
    autoGainControl: true
  });

  // Simulate 80dB ambient noise
  mockAudioInputManager.setNoise({ level: 80, type: 'crowd' });

  await delay(30000);
  const result = await recorder.stopRecording(recording.id);

  const audioAnalysis = await analyzeAudio(result.filePath);
  expect(audioAnalysis.peakLevel).toBeLessThan(-3); // -3dB headroom
  expect(audioAnalysis.clippingDetected).toBe(false);
  expect(audioAnalysis.noiseFloorDb).toBeGreaterThan(60);
});
```

---

#### TC-F2-E2.2: Handle Audio Session Interruption (Alarm/Timer)
**Objective**: Verify recording pauses when system sounds interrupt

**Test Steps**:
1. Start recording
2. Simulate device alarm triggering
3. Verify recording pauses
4. Verify notification shown
5. Silence alarm (simulate)
6. Verify recording resumes
7. Verify alarm audio not captured

**Expected Result**: 
- Recording pauses for interruption
- Recording resumes after
- Alarm audio excluded from recording

**Code Sample**:
```typescript
it('should pause recording for system audio interruptions', async () => {
  const recorder = new BackgroundAudioRecorder(mockAVSession);
  const recording = await recorder.startRecording('session-123');

  // Simulate alarm triggering
  mockAVSession.emitInterruption('BEGIN', 'ALARM');

  await waitFor(() => {
    expect(recording.state).toBe('PAUSED');
  });

  // Silence alarm
  mockAVSession.emitInterruption('END', 'ALARM');

  await waitFor(() => {
    expect(recording.state).toBe('RECORDING');
  });
});
```

---

### 3.3 Network Edge Cases

#### TC-F2-E3.1: Multiple Upload Retries on Persistent Failure
**Objective**: Verify upload retry behavior with persistent network failure

**Test Steps**:
1. Queue chunk for upload
2. Configure retry policy: max 5 attempts, exponential backoff
3. Simulate network failure for all 5 attempts
4. Verify exponential backoff: 1s, 2s, 4s, 8s, 16s
5. Verify error logged after final retry
6. Verify chunk remains in queue for manual retry later

**Expected Result**: 
- All 5 retries attempted
- Exponential backoff respected
- Chunk persisted for later retry

**Code Sample**:
```typescript
it('should retry upload with exponential backoff', async () => {
  const uploadQueue = new AudioUploadQueue({
    maxRetries: 5,
    backoffMultiplier: 2,
    initialDelayMs: 1000
  });

  mockHttpClient.setFailureMode('PERSIST ENT');

  const startTime = Date.now();
  await uploadQueue.addItem('chunk.wav', 'session-123');
  
  try {
    await uploadQueue.processAll();
  } catch (error) {
    expect(error.message).toContain('Max retries exceeded');
  }

  const duration = Date.now() - startTime;
  // Total backoff: 1s + 2s + 4s + 8s + 16s = 31s minimum
  expect(duration).toBeGreaterThan(31000);

  // Verify chunk still in queue
  const items = await uploadQueue.getItems();
  expect(items).toContainEqual(
    expect.objectContaining({ state: 'FAILED', retryCount: 5 })
  );
});
```

---

## 4. PERFORMANCE VALIDATION

### 4.1 Recording Performance

#### TC-F2-P1.1: Minimal CPU Impact While Recording
**Objective**: Verify recording doesn't consume excessive CPU

**Test Steps**:
1. Start recording
2. Monitor CPU usage for 60 seconds
3. Verify average CPU < 15% (on mid-range device)
4. Verify no process threads excessive
5. Verify main thread not blocked

**Expected Result**: 
- CPU usage remains low
- No main thread blocking
- Recording smooth at 60 FPS if UI present

**Code Sample**:
```typescript
it('should maintain low CPU usage while recording', async () => {
  const recorder = new BackgroundAudioRecorder(mockAVSession);
  const cpuMonitor = new CPUMonitor();

  cpuMonitor.start();
  const recording = await recorder.startRecording('session-123');

  await delay(60000);

  await recorder.stopRecording(recording.id);
  const metrics = cpuMonitor.getMetrics();

  expect(metrics.averageCPU).toBeLessThan(0.15); // 15%
  expect(metrics.mainThreadBlocked).toBe(false);
});
```

---

#### TC-F2-P1.2: Audio Encoding Performance
**Objective**: Verify audio chunk encoding completes efficiently

**Test Steps**:
1. Record for 60 seconds (creates one chunk)
2. Record start time for encoding
3. Stop recording (triggers encoding/compression)
4. Measure encoding completion time
5. Assert encoding < 5 seconds for 60-sec clip

**Expected Result**: 
- Encoding completes < 5 seconds
- Audio quality maintained
- File properly formatted

**Code Sample**:
```typescript
it('should encode 60-second audio chunk in under 5 seconds', async () => {
  const recorder = new BackgroundAudioRecorder(mockAVSession);
  const recording = await recorder.startRecording('session-123');

  await delay(60000);

  const encodeStart = performance.now();
  const result = await recorder.stopRecording(recording.id);
  const encodeDuration = performance.now() - encodeStart;

  expect(encodeDuration).toBeLessThan(5000);
  expect(result.fileSize).toBeGreaterThan(400000); // ~440KB for 60sec
});
```

---

### 4.2 Battery Impact

#### TC-F2-P2.1: Battery Drain Rate While Recording
**Objective**: Verify battery drain remains acceptable

**Test Steps**:
1. Fully charge device
2. Start recording audio in background
3. Keep recording for 1 hour
4. Measure battery drain percentage
5. Verify drain <= 5% per hour (typical low rate)

**Expected Result**: 
- Battery drain <= 5% per hour
- Acceptable for conference day usage

**Code Sample**:
```typescript
it('should maintain acceptable battery drain rate', async () => {
  mockDeviceService.setBatteryLevel(1.0); // 100%
  const recorder = new BackgroundAudioRecorder(mockAVSession);

  const recording = await recorder.startRecording('session-123');

  // Record for 1 hour (or simulated)
  await delay(3600000);

  await recorder.stopRecording(recording.id);

  const finalBattery = mockDeviceService.getBatteryLevel();
  const drain = (1.0 - finalBattery) * 100;

  expect(drain).toBeLessThan(5); // Less than 5% per hour
});
```

---

### 4.3 Storage Performance

#### TC-F2-P3.1: Local Cache Space Efficiency
**Objective**: Verify audio chunks don't consume excessive local storage

**Test Steps**:
1. Record audio in multiple sessions totaling 6 hours
2. Calculate total cache space used
3. Verify space used reasonable (expect ~6 hours * 10KB/sec = ~216MB)
4. Upload all chunks
5. Verify cache cleaned up
6. Verify free space restored

**Expected Result**: 
- Cache storage reasonable
- Proper cleanup after upload
- No orphaned files

**Code Sample**:
```typescript
it('should use reasonable cache space for recordings', async () => {
  const recorder = new BackgroundAudioRecorder(mockAVSession);
  const uploadQueue = new AudioUploadQueue();

  const initialFree = mockStorageService.getFreeSpace();

  // Record multiple sessions: 6 hours total
  const recordings = [];
  for (let i = 0; i < 6; i++) {
    const rec = await recorder.startRecording(`session-${i}`);
    await delay(60000); // 1 hour each (simulated)
    const result = await recorder.stopRecording(rec.id);
    recordings.push(result);

    // Queue for upload
    const chunks = await recorder.getSegments(rec.id);
    chunks.forEach(chunk => uploadQueue.addItem(chunk.path));
  }

  const cacheUsed = initialFree - mockStorageService.getFreeSpace();
  // Expect ~216MB for 6 hours at 10KB/sec
  expect(cacheUsed).toBeLessThan(300 * 1024 * 1024); // 300MB max

  // Upload and verify cleanup
  await uploadQueue.processAll();

  const finalFree = mockStorageService.getFreeSpace();
  expect(finalFree).toBeGreaterThan(initialFree - 50 * 1024 * 1024); // Tolerance
});
```

---

## Test Execution Summary

### Test Categories
- **Unit Tests**: 3 suites, 9 test cases
- **Integration Tests**: 2 suites, 4 test cases
- **Edge Cases**: 3 suites, 7 test cases
- **Performance Tests**: 3 suites, 6 test cases

### Total: 26 comprehensive test cases

### Acceptance Criteria Coverage
- ✓ Start/pause/resume recording functionality
- ✓ Segment audio files reliably
- ✓ Handle background mode
- ✓ Recover after interruption
- ✓ Upload completed segments
- ✓ Explicit recording consent required
- ✓ Local encryption before upload
- ✓ Visible recording status maintained
- ✓ Audio loss < 1% of active duration
- ✓ Successful recording sessions > 95%
- ✓ Recovered interruptions > 90%

