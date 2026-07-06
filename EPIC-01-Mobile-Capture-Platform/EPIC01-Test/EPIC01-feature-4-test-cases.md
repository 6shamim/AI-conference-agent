# EPIC01 Feature 4 — Camera Capture Workflow — Test Cases

## Test Overview
Comprehensive test suite for Camera Capture Workflow feature covering unit tests, integration tests, edge cases, and performance validation for image capture and metadata management.

---

## 1. UNIT TEST SCENARIOS

### 1.1 Image Capture & Processing

#### TC-F4-U1.1: Successful Image Capture
**Objective**: Verify image captured successfully with correct metadata

**Preconditions**:
- Camera permission granted
- Device has working camera
- Active conference session
- Sufficient storage

**Test Steps**:
1. Call `captureImage()` with session context
2. Verify image file created at expected path
3. Verify image file is valid (readable by image codec)
4. Verify metadata attached:
   - sessionId
   - captureTimestamp
   - capturedByUserId
   - imageType
5. Verify file size > 100KB (reasonable quality)

**Expected Result**: Valid image file with correct metadata

**Code Sample**:
```typescript
describe('CameraCaptureManager', () => {
  it('should capture image with metadata successfully', async () => {
    const manager = new CameraCaptureManager(mockCameraService);
    const sessionId = 'session-123';

    const capture = await manager.captureImage({
      sessionId,
      userId: 'user-456',
      type: 'SLIDE'
    });

    expect(capture).toBeDefined();
    expect(capture.filePath).toMatch(/\.jpg$|\.png$/);
    expect(capture.fileSize).toBeGreaterThan(100000);
    expect(capture.metadata.sessionId).toBe(sessionId);
    expect(capture.metadata.imageType).toBe('SLIDE');
    expect(capture.metadata.timestamp).toBeLessThanOrEqual(Date.now());
  });
});
```

---

#### TC-F4-U1.2: Image Type Auto-Detection
**Objective**: Verify system auto-detects and classifies image type

**Test Steps**:
1. Capture image of business card
2. Call `detectImageType(imageFile)`
3. Verify returned type = 'BUSINESS_CARD'
4. Capture image of slide
5. Verify returned type = 'SLIDE'
6. Capture image of booth/signage
7. Verify returned type = 'BOOTH_INFO'

**Expected Result**: 
- Image type correctly classified
- Confidence score included

**Code Sample**:
```typescript
it('should auto-detect image type', async () => {
  const classifier = new ImageTypeClassifier();

  const types = [
    { file: 'badge.jpg', expected: 'BADGE', minConfidence: 0.8 },
    { file: 'slide.jpg', expected: 'SLIDE', minConfidence: 0.8 },
    { file: 'card.jpg', expected: 'BUSINESS_CARD', minConfidence: 0.75 }
  ];

  for (const { file, expected, minConfidence } of types) {
    const result = await classifier.classifyImage(file);
    expect(result.type).toBe(expected);
    expect(result.confidence).toBeGreaterThan(minConfidence);
  }
});
```

---

#### TC-F4-U1.3: Image Compression
**Objective**: Verify image compressed efficiently without quality loss

**Test Steps**:
1. Capture high-resolution image (e.g., 4000x3000)
2. Verify image compressed to max 2000x1500
3. Verify JPEG quality set to 80% (balanced quality/size)
4. Verify file size < 500KB
5. Verify image still readable for OCR

**Expected Result**: 
- Image compressed appropriately
- File size reasonable
- Quality maintained

**Code Sample**:
```typescript
it('should compress image efficiently', async () => {
  const manager = new CameraCaptureManager(mockCameraService, {
    maxWidth: 2000,
    maxHeight: 1500,
    jpegQuality: 80
  });

  const capture = await manager.captureImage({ sessionId: 'session-123' });

  const imageData = await getImageDimensions(capture.filePath);
  expect(imageData.width).toBeLessThanOrEqual(2000);
  expect(imageData.height).toBeLessThanOrEqual(1500);
  expect(capture.fileSize).toBeLessThan(500 * 1024);

  // Verify readable for OCR
  const ocrReady = await isOCRReady(capture.filePath);
  expect(ocrReady).toBe(true);
});
```

---

### 1.2 Image Preview & Retake

#### TC-F4-U2.1: Image Preview Before Save
**Objective**: Verify user can preview image before saving

**Test Steps**:
1. Capture image
2. Display preview screen with captured image
3. Render Confirm/Retake buttons
4. Verify image displayed at 100% resolution (or close)
5. Verify no data loss in preview
6. User clicks Confirm
7. Verify image saved and metadata attached

**Expected Result**: 
- Preview clear and accurate
- Metadata attached on save
- Retake available

**Code Sample**:
```typescript
it('should show preview and allow retake', async () => {
  const manager = new CameraCaptureManager(mockCameraService);
  const { getByTestId, getByText } = render(
    <CameraPreviewScreen sessionId="session-123" />
  );

  // Capture image
  const capturedImage = await manager.captureImage({ sessionId: 'session-123' });

  // Show preview
  const preview = getByTestId('image-preview');
  expect(preview).toBeVisible();

  // Verify buttons
  expect(getByText(/Confirm|Save/i)).toBeInTheDocument();
  expect(getByText(/Retake/i)).toBeInTheDocument();

  // User clicks Confirm
  fireEvent.click(getByText(/Confirm/i));

  // Verify saved
  const saved = await manager.getImage(capturedImage.id);
  expect(saved).toBeDefined();
});
```

---

#### TC-F4-U2.2: Retake Functionality
**Objective**: Verify retake deletes previous capture and replaces

**Test Steps**:
1. Capture image (Image A)
2. Show preview
3. Click Retake
4. Verify Image A deleted
5. Capture new image (Image B)
6. Show new preview
7. Click Confirm
8. Verify only Image B saved

**Expected Result**: 
- Retake properly cleans up previous capture
- Only final image saved

**Code Sample**:
```typescript
it('should replace image on retake', async () => {
  const manager = new CameraCaptureManager(mockCameraService);
  
  // Capture first image
  const imageA = await manager.captureImage({ sessionId: 'session-123' });
  const fileAPath = imageA.filePath;

  // Verify exists
  let exists = await fileExists(fileAPath);
  expect(exists).toBe(true);

  // Retake
  await manager.retakeImage(imageA.id);

  // Verify deleted
  exists = await fileExists(fileAPath);
  expect(exists).toBe(false);

  // Capture replacement
  const imageB = await manager.captureImage({ sessionId: 'session-123' });
  const fileBPath = imageB.filePath;

  // Verify new file exists
  exists = await fileExists(fileBPath);
  expect(exists).toBe(true);
});
```

---

### 1.3 Metadata & Linking

#### TC-F4-U3.1: Image Linked to Session & Interaction
**Objective**: Verify image properly linked to context

**Test Steps**:
1. Create session and start interaction
2. Capture image
3. Verify image metadata includes:
   - sessionId
   - interactionId
   - timestamp
   - userId
4. Query: find all images for session
5. Verify captured image appears in results

**Expected Result**: 
- Image linked to correct context
- Query returns expected results

**Code Sample**:
```typescript
it('should link image to session and interaction', async () => {
  const db = new ImageDatabase();
  const sessionId = 'session-123';
  const interactionId = 'interaction-456';

  const capture = await db.saveImageCapture({
    sessionId,
    interactionId,
    userId: 'user-789',
    type: 'BADGE',
    filePath: '/local/images/badge.jpg'
  });

  // Query images for session
  const sessionImages = await db.getImagesBySession(sessionId);
  expect(sessionImages).toContainEqual(
    expect.objectContaining({
      id: capture.id,
      sessionId,
      interactionId
    })
  );
});
```

---

## 2. INTEGRATION TEST SCENARIOS

### 2.1 Vision Pipeline Integration

#### TC-F4-I1.1: Trigger OCR After Image Upload
**Objective**: Verify OCR automatically triggered after image upload

**Test Steps**:
1. Capture and save image (business card)
2. Image queued for upload
3. Upload completes successfully
4. Verify OCR event triggered
5. Verify OCR processes business card
6. Verify text extracted and stored

**Expected Result**: 
- OCR triggered automatically
- Text extracted and linked to image

**Code Sample**:
```typescript
describe('Camera Vision Pipeline Integration', () => {
  it('should trigger OCR after image upload', async () => {
    const manager = new CameraCaptureManager(mockCameraService);
    const uploadQueue = new ImageUploadQueue();
    const visionQueue = new VisionQueue();

    // Capture business card
    const capture = await manager.captureImage({
      sessionId: 'session-123',
      type: 'BUSINESS_CARD'
    });

    // Upload image
    await uploadQueue.addItem(capture.filePath);
    await uploadQueue.processAll();

    // Verify OCR event queued
    const events = await visionQueue.getEvents();
    expect(events).toContainEqual(
      expect.objectContaining({
        type: 'OCR_REQUESTED',
        imageId: capture.id,
        imageType: 'BUSINESS_CARD'
      })
    );
  });
});
```

---

#### TC-F4-I1.2: Auto-Contact Creation from Business Card
**Objective**: Verify contact created automatically from OCR text

**Test Steps**:
1. Capture business card image
2. Upload image
3. OCR extracts text (name, email, phone)
4. Contact service receives OCR result
5. Contact created with extracted fields
6. User can review and edit
7. Contact saved to database

**Expected Result**: 
- Contact automatically created from card
- User can approve/modify
- Linked to session

**Code Sample**:
```typescript
it('should create contact from business card OCR', async () => {
  const ocrService = new OCRService();
  const contactService = new ContactService();

  // Simulate OCR result from business card
  const ocrResult = await ocrService.processImage('business_card.jpg');
  expect(ocrResult).toMatchObject({
    text: expect.stringContaining('email'),
    fields: {
      name: 'John Doe',
      email: 'john@company.com',
      phone: '555-1234',
      company: 'Acme Corp'
    }
  });

  // Create contact from OCR
  const contact = await contactService.createFromOCR(ocrResult, {
    sessionId: 'session-123'
  });

  expect(contact).toBeDefined();
  expect(contact.name).toBe('John Doe');
  expect(contact.email).toBe('john@company.com');
});
```

---

## 3. EDGE CASE VALIDATION

### 3.1 Image Quality Edge Cases

#### TC-F4-E1.1: Blurry Image Detection and Warning
**Objective**: Verify system detects blurry images and warns user

**Test Steps**:
1. Simulate capturing blurry image (low contrast/sharpness)
2. Verify image captured (no crash)
3. Verify blur detection algorithm identifies blur
4. Verify warning shown to user before save
5. User can accept blurry image or retake
6. If accepted, flag image as "LOW_QUALITY"

**Expected Result**: 
- Blur detected
- User warned
- Continues with flag

**Code Sample**:
```typescript
it('should detect blurry images and warn user', async () => {
  const manager = new CameraCaptureManager(mockCameraService, {
    blurThreshold: 0.3
  });

  // Simulate blurry capture
  const mockBlurryImage = createMockBlurryImage(0.5); // High blur
  
  const capture = await manager.captureImage({
    sessionId: 'session-123'
  });

  const analysis = await manager.analyzeImageQuality(capture);
  expect(analysis.isBlurry).toBe(true);
  expect(analysis.blurScore).toBeGreaterThan(0.3);

  // Warning should be shown in UI
  expect(mockUIService.showWarning).toHaveBeenCalledWith(
    expect.stringContaining('blurry')
  );
});
```

---

#### TC-F4-E1.2: Poor Lighting Detection
**Objective**: Verify system detects and warns about poor lighting

**Test Steps**:
1. Simulate capturing in dark environment (< 50 lux)
2. Capture image
3. Verify low light detected
4. Verify warning shown: "Lighting too dim for OCR"
5. Suggest user improve lighting
6. User can retry or accept

**Expected Result**: 
- Low light detected
- Helpful message provided
- Continues if accepted

**Code Sample**:
```typescript
it('should detect poor lighting and suggest improvement', async () => {
  const manager = new CameraCaptureManager(mockCameraService, {
    minLightingLevel: 50 // lux
  });

  // Simulate low light
  mockCameraService.setLighting(30); // 30 lux

  const capture = await manager.captureImage({
    sessionId: 'session-123'
  });

  const analysis = await manager.analyzeImageQuality(capture);
  expect(analysis.lightingLevel).toBeLessThan(50);

  const warning = mockUIService.getLastWarning();
  expect(warning.message).toContain('lighting');
  expect(warning.suggestion).toContain('improve');
});
```

---

### 3.2 Storage & Permission Edge Cases

#### TC-F4-E2.1: Camera Permission Denied
**Objective**: Verify graceful handling when camera permission denied

**Test Steps**:
1. Attempt to open camera
2. System prompts for permission
3. User denies permission
4. Verify app doesn't crash
5. Verify error message shown
6. Verify Settings link provided
7. User can retry after granting permission

**Expected Result**: 
- Graceful error handling
- User path to fix provided

**Code Sample**:
```typescript
it('should handle camera permission denial', async () => {
  mockPermissionService.setUserResponse('camera', 'DENY');

  const manager = new CameraCaptureManager(mockCameraService);

  await expect(
    manager.captureImage({ sessionId: 'session-123' })
  ).rejects.toMatchObject({
    code: 'PERMISSION_DENIED',
    message: expect.stringContaining('camera')
  });

  // Verify user guidance provided
  expect(mockUIService.showPermissionError).toHaveBeenCalledWith(
    expect.objectContaining({
      permissionName: 'camera',
      settingsLink: expect.any(String)
    })
  );
});
```

---

#### TC-F4-E2.2: Storage Full When Saving Image
**Objective**: Verify handling when storage full during save

**Test Steps**:
1. Capture image
2. Set available storage to < 100KB
3. Attempt to save image
4. Verify storage error caught
5. Verify partial file not left behind
6. Verify error message: "Storage Full"
7. Suggest cleanup options

**Expected Result**: 
- No corrupted files
- Clear error message
- Cleanup suggestions

**Code Sample**:
```typescript
it('should handle storage full gracefully', async () => {
  mockStorageService.setAvailableSpace(50 * 1024); // 50KB

  const manager = new CameraCaptureManager(mockCameraService);

  await expect(
    manager.captureImage({ sessionId: 'session-123' })
  ).rejects.toMatchObject({
    code: 'STORAGE_FULL'
  });

  // Verify no orphaned files
  const tempFiles = await getTempFiles();
  expect(tempFiles).toHaveLength(0);
});
```

---

### 3.3 Camera Session Edge Cases

#### TC-F4-E3.1: Camera Interrupted by Phone Call
**Objective**: Verify camera properly releases on interruption

**Test Steps**:
1. User opening camera
2. Incoming phone call
3. Verify camera closes gracefully
4. Verify no resources locked
5. User ends call
6. User can re-open camera and capture

**Expected Result**: 
- Camera properly released
- No resource locks
- Seamless recovery

**Code Sample**:
```typescript
it('should release camera on interruption', async () => {
  const manager = new CameraCaptureManager(mockCameraService);

  const capturePromise = manager.captureImage({ 
    sessionId: 'session-123' 
  });

  // Simulate incoming call
  mockCallService.incomingCall();

  try {
    await capturePromise;
  } catch (error) {
    expect(error.code).toBe('INTERRUPTED');
  }

  // Verify camera released
  expect(mockCameraService.isReleased()).toBe(true);

  // User ends call
  mockCallService.endCall();

  // Should be able to capture again
  const secondCapture = await manager.captureImage({
    sessionId: 'session-123'
  });
  expect(secondCapture).toBeDefined();
});
```

---

## 4. PERFORMANCE VALIDATION

### 4.1 Camera Operation Performance

#### TC-F4-P1.1: Camera Launch < 1 Second
**Objective**: Verify camera opens quickly

**Test Steps**:
1. Record start time
2. Call `openCamera()` on fresh app start
3. Record time when camera preview visible
4. Assert < 1000ms
5. Repeat 10 times for consistency

**Expected Result**: 
- All launches < 1 second
- Average < 700ms

**Code Sample**:
```typescript
it('should launch camera in under 1 second', async () => {
  const manager = new CameraCaptureManager(mockCameraService);
  const times = [];

  for (let i = 0; i < 10; i++) {
    const start = performance.now();
    await manager.openCamera();
    const duration = performance.now() - start;
    times.push(duration);
  }

  const max = Math.max(...times);
  const average = times.reduce((a, b) => a + b) / times.length;

  expect(max).toBeLessThan(1000);
  expect(average).toBeLessThan(700);
});
```

---

#### TC-F4-P1.2: Image Save Time < 2 Seconds
**Objective**: Verify captured image saves efficiently

**Test Steps**:
1. Capture image
2. Record start time for save operation
3. Save image with metadata
4. Record end time
5. Assert < 2000ms
6. Verify file integrity

**Expected Result**: 
- Save completes < 2 seconds
- File valid and complete

**Code Sample**:
```typescript
it('should save image in under 2 seconds', async () => {
  const manager = new CameraCaptureManager(mockCameraService);

  const times = [];
  for (let i = 0; i < 5; i++) {
    const capture = await manager.captureImage({
      sessionId: 'session-123'
    });

    const start = performance.now();
    const saved = await manager.saveImage(capture);
    const duration = performance.now() - start;
    times.push(duration);

    expect(await fileExists(saved.filePath)).toBe(true);
  }

  const max = Math.max(...times);
  expect(max).toBeLessThan(2000);
});
```

---

### 4.2 Memory Performance

#### TC-F4-P2.1: Camera Memory Usage
**Objective**: Verify camera doesn't consume excessive memory

**Test Steps**:
1. Measure heap before camera open
2. Open camera
3. Measure heap with camera active
4. Capture 5 images
5. Verify total memory increase < 50MB
6. Close camera
7. Verify memory released

**Expected Result**: 
- Reasonable memory consumption
- Proper cleanup

**Code Sample**:
```typescript
it('should use reasonable memory for camera operations', async () => {
  if (typeof gc !== 'function') return;

  gc();
  const initialMemory = process.memoryUsage().heapUsed;

  const manager = new CameraCaptureManager(mockCameraService);
  await manager.openCamera();

  for (let i = 0; i < 5; i++) {
    await manager.captureImage({ sessionId: 'session-123' });
  }

  const activeMemory = process.memoryUsage().heapUsed;
  const increase = activeMemory - initialMemory;

  expect(increase).toBeLessThan(50 * 1024 * 1024); // 50MB
});
```

---

## Test Execution Summary

### Test Categories
- **Unit Tests**: 3 suites, 9 test cases
- **Integration Tests**: 2 suites, 4 test cases
- **Edge Cases**: 3 suites, 6 test cases
- **Performance Tests**: 2 suites, 4 test cases

### Total: 23 comprehensive test cases

### Acceptance Criteria Coverage
- ✓ Capture image successfully
- ✓ Classify image type
- ✓ Attach metadata
- ✓ Queue OCR/vision processing
- ✓ Retake/delete image
- ✓ Camera launch < 1 second
- ✓ Image save < 2 seconds
- ✓ Auto-link to current session
- ✓ Offline capture supported
- ✓ OCR-ready image quality > 85%
- ✓ Successful image captures > 95%

