# EPIC01 Feature 8 - Quick Interaction Tagging - Comprehensive Test Cases

## Feature Overview
Feature-08 enables users to efficiently tag conference interactions (meetings, panels, booths, investor leads, customers, partners) with minimal friction using preset tags, custom tags, and voice notes.

---

## Test Execution Environment

### Setup & Dependencies
- **Platform**: React Native (iOS/Android)
- **Test Framework**: Jest, Detox (E2E), React Testing Library
- **Mock Services**: 
  - AsyncStorage for local cache
  - MSW (Mock Service Worker) for API mocking
  - React Native Testing Library for UI components
- **Dependencies**:
  - `@react-native-async-storage/async-storage`
  - `@testing-library/react-native`
  - `jest`
  - `detox`

### Environment Variables
```
TEST_ENV=development
API_BASE_URL=http://localhost:3000
MOCK_API=true
STORAGE_TYPE=memory
```

---

## Unit Test Scenarios

### 1.1 Tag Application - Preset Tags

**Test Case ID**: F8-UNIT-001  
**Acceptance Criteria**: AC-F8-001 (Functional - Feature launches successfully)

**Description**: Verify preset tag selection and application to active interaction

**Test Steps**:
```typescript
// Test: Apply preset tag to active interaction
describe('Quick Interaction Tagging - Preset Tags', () => {
  beforeEach(() => {
    jest.clearAllMocks();
  });

  test('should successfully apply preset tag to interaction', async () => {
    const mockInteractionId = 'interaction-123';
    const mockPresetTag = { id: 'tag-investor', label: 'Investor', type: 'preset' };
    
    const { getByTestId } = render(
      <TaggingComponent interactionId={mockInteractionId} />
    );
    
    const tagButton = getByTestId('tag-preset-investor');
    fireEvent.press(tagButton);
    
    // Assert tag was applied
    expect(getByTestId('tag-badge-investor')).toBeTruthy();
    
    // Verify API call
    await waitFor(() => {
      expect(mockTagAPI.applyTag).toHaveBeenCalledWith({
        interactionId: mockInteractionId,
        tagId: 'tag-investor',
        tagType: 'preset'
      });
    });
  });

  test('should handle preset tag application when offline', async () => {
    const mockInteractionId = 'interaction-456';
    const { getByTestId } = render(
      <TaggingComponent interactionId={mockInteractionId} isOffline={true} />
    );
    
    fireEvent.press(getByTestId('tag-preset-customer'));
    
    // Verify local storage was updated
    const offlineQueue = await AsyncStorage.getItem('tag-queue');
    expect(offlineQueue).toContain('interaction-456');
  });

  test('should show success feedback within 1 second (UX-AC)', async () => {
    const { getByTestId, getByText } = render(
      <TaggingComponent interactionId="interaction-789" />
    );
    
    const startTime = Date.now();
    fireEvent.press(getByTestId('tag-preset-meeting'));
    
    await waitFor(() => {
      expect(getByText('Tag applied')).toBeTruthy();
    }, { timeout: 1000 });
    
    const duration = Date.now() - startTime;
    expect(duration).toBeLessThan(1000);
  });

  test('should handle API rate limiting gracefully', async () => {
    mockTagAPI.applyTag.mockRejectedValueOnce({
      status: 429,
      message: 'Rate limited'
    });
    
    const { getByTestId, getByText } = render(
      <TaggingComponent interactionId="interaction-001" />
    );
    
    fireEvent.press(getByTestId('tag-preset-investor'));
    
    await waitFor(() => {
      expect(getByText(/retry|queue/i)).toBeTruthy();
    });
  });
});
```

**Expected Result**: 
- Tag applied successfully to interaction
- API called with correct parameters
- Success feedback shown within 1 second
- Offline behavior queued correctly
- Rate limiting handled gracefully

**Failure Criteria**:
- UI freezes during tag application
- API call fails without retry
- No feedback to user

---

### 1.2 Custom Tag Creation

**Test Case ID**: F8-UNIT-002  
**Acceptance Criteria**: AC-F8-001, AC-F8-003 (Technical - API responses handled)

**Description**: Verify custom tag creation and storage

**Test Steps**:
```typescript
test('should create and store custom tag', async () => {
  const customTagName = 'VIP Prospect';
  const { getByTestId, getByPlaceholderText, getByText } = render(
    <TaggingComponent interactionId="interaction-111" />
  );
  
  // Tap custom tag button
  fireEvent.press(getByTestId('btn-custom-tag'));
  
  // Enter custom tag name
  const input = getByPlaceholderText('Tag name');
  fireEvent.changeText(input, customTagName);
  
  // Submit
  fireEvent.press(getByText('Create'));
  
  // Verify custom tag was stored
  const storedTags = await AsyncStorage.getItem('custom-tags');
  expect(JSON.parse(storedTags)).toContainEqual(
    expect.objectContaining({ name: customTagName })
  );
  
  // Verify API was called
  await waitFor(() => {
    expect(mockTagAPI.createCustomTag).toHaveBeenCalledWith({
      name: customTagName,
      userId: mockUserId
    });
  });
});

test('should prevent duplicate custom tag names', async () => {
  const tagName = 'Duplicate Tag';
  const { getByTestId, getByText, getByPlaceholderText } = render(
    <TaggingComponent interactionId="interaction-222" />
  );
  
  // First tag creation
  fireEvent.press(getByTestId('btn-custom-tag'));
  fireEvent.changeText(getByPlaceholderText('Tag name'), tagName);
  fireEvent.press(getByText('Create'));
  
  await waitFor(() => {
    expect(mockTagAPI.createCustomTag).toHaveBeenCalledTimes(1);
  });
  
  // Attempt duplicate
  fireEvent.press(getByTestId('btn-custom-tag'));
  fireEvent.changeText(getByPlaceholderText('Tag name'), tagName);
  fireEvent.press(getByText('Create'));
  
  await waitFor(() => {
    expect(getByText(/already exists/i)).toBeTruthy();
  });
});

test('should validate custom tag length constraints', async () => {
  const tooLongTag = 'a'.repeat(101); // Assuming 100 char limit
  const { getByTestId, getByText, getByPlaceholderText } = render(
    <TaggingComponent interactionId="interaction-333" />
  );
  
  fireEvent.press(getByTestId('btn-custom-tag'));
  fireEvent.changeText(getByPlaceholderText('Tag name'), tooLongTag);
  fireEvent.press(getByText('Create'));
  
  expect(getByText(/too long/i)).toBeTruthy();
  expect(mockTagAPI.createCustomTag).not.toHaveBeenCalled();
});
```

**Expected Result**:
- Custom tag created and stored locally
- API call made with correct tag data
- Duplicates prevented
- Length validation enforced

---

### 1.3 Voice Note Tagging

**Test Case ID**: F8-UNIT-003  
**Acceptance Criteria**: AC-F8-002 (UX - Workflow minimal interaction)

**Description**: Verify voice note recording and attachment as tag

**Test Steps**:
```typescript
test('should record and attach voice tag', async () => {
  const { getByTestId } = render(
    <TaggingComponent interactionId="interaction-444" />
  );
  
  // Start voice recording
  fireEvent.press(getByTestId('btn-voice-tag'));
  
  // Mock voice data
  await new Promise(resolve => setTimeout(resolve, 2000)); // Simulate 2s recording
  
  fireEvent.press(getByTestId('btn-stop-voice'));
  
  // Verify API call with voice data
  await waitFor(() => {
    expect(mockMediaAPI.uploadVoiceTag).toHaveBeenCalledWith(
      expect.objectContaining({
        interactionId: 'interaction-444',
        mediaType: 'voice'
      })
    );
  });
});

test('should handle voice tag permission denial', async () => {
  mockPermissions.checkMicrophonePermission.mockResolvedValueOnce(false);
  
  const { getByTestId, getByText } = render(
    <TaggingComponent interactionId="interaction-555" />
  );
  
  fireEvent.press(getByTestId('btn-voice-tag'));
  
  await waitFor(() => {
    expect(getByText(/permission.*microphone/i)).toBeTruthy();
  });
});
```

**Expected Result**:
- Voice recording captured
- Voice tag uploaded successfully
- Permission handling correct

---

### 1.4 Edit and Remove Tags

**Test Case ID**: F8-UNIT-004  
**Acceptance Criteria**: AC-F8-004 (Functional - User interaction completes)

**Description**: Verify tag editing and removal functionality

**Test Steps**:
```typescript
test('should remove tag from interaction', async () => {
  const { getByTestId, queryByTestId } = render(
    <TaggingComponent 
      interactionId="interaction-666" 
      initialTags={['tag-investor']}
    />
  );
  
  const removeButton = getByTestId('btn-remove-tag-investor');
  fireEvent.press(removeButton);
  
  await waitFor(() => {
    expect(queryByTestId('tag-badge-investor')).toBeFalsy();
  });
  
  // Verify API call
  await waitFor(() => {
    expect(mockTagAPI.removeTag).toHaveBeenCalledWith({
      interactionId: 'interaction-666',
      tagId: 'tag-investor'
    });
  });
});

test('should prevent removal of tag from active interaction', async () => {
  mockInteractionAPI.getActive.mockResolvedValueOnce({
    id: 'interaction-777',
    status: 'active'
  });
  
  const { getByTestId, getByText } = render(
    <TaggingComponent 
      interactionId="interaction-777"
      initialTags={['tag-customer']}
      isActive={true}
    />
  );
  
  fireEvent.press(getByTestId('btn-remove-tag-customer'));
  
  await waitFor(() => {
    expect(getByText(/cannot remove.*active/i)).toBeTruthy();
  });
});
```

**Expected Result**:
- Tags removed successfully
- API called correctly
- Constraints enforced

---

## Integration Test Scenarios

### 2.1 Tag Sync Across Devices

**Test Case ID**: F8-INT-001  
**Acceptance Criteria**: AC-F8-003 (Technical - Retry logic implemented)

**Description**: Verify tags sync correctly when moving between devices

**Test Steps**:
```typescript
describe('Tag Sync - Integration Tests', () => {
  test('should sync applied tags to backend after network restoration', async () => {
    const { getByTestId } = render(<ConferenceApp />);
    
    // Start offline
    mockNetworkAPI.setOnline(false);
    
    // Apply tag offline
    fireEvent.press(getByTestId('tag-preset-investor'));
    
    // Verify offline queue
    let queue = await AsyncStorage.getItem('tag-queue');
    expect(JSON.parse(queue)).toHaveLength(1);
    
    // Restore network
    mockNetworkAPI.setOnline(true);
    
    // Wait for sync
    await waitFor(() => {
      expect(mockTagAPI.applyTag).toHaveBeenCalled();
    }, { timeout: 5000 });
    
    // Verify queue cleared
    queue = await AsyncStorage.getItem('tag-queue');
    expect(JSON.parse(queue)).toHaveLength(0);
  });

  test('should handle tag sync conflicts', async () => {
    mockTagAPI.applyTag.mockRejectedValueOnce({
      status: 409,
      message: 'Tag already exists'
    });
    
    const { getByTestId, getByText } = render(<ConferenceApp />);
    
    fireEvent.press(getByTestId('tag-preset-customer'));
    
    await waitFor(() => {
      expect(getByText(/already tagged/i)).toBeTruthy();
    });
  });

  test('should implement retry logic for failed tag operations', async () => {
    let callCount = 0;
    mockTagAPI.applyTag.mockImplementation(() => {
      callCount++;
      if (callCount < 3) {
        return Promise.reject({ status: 500 });
      }
      return Promise.resolve({ success: true });
    });
    
    const { getByTestId } = render(<ConferenceApp />);
    
    fireEvent.press(getByTestId('tag-preset-meeting'));
    
    // Should retry and eventually succeed
    await waitFor(() => {
      expect(mockTagAPI.applyTag).toHaveBeenCalledTimes(3);
    }, { timeout: 5000 });
  });
});
```

**Expected Result**:
- Tags queued offline
- Successfully synced after network restoration
- Conflicts handled gracefully
- Retry logic implemented

---

### 2.2 Tag Metadata and Context

**Test Case ID**: F8-INT-002  
**Acceptance Criteria**: AC-F8-005 (Technical - Local persistence supported)

**Description**: Verify tag metadata stored with interaction context

**Test Steps**:
```typescript
test('should store tag metadata with interaction context', async () => {
  const { getByTestId } = render(
    <TaggingComponent interactionId="interaction-888" />
  );
  
  fireEvent.press(getByTestId('tag-preset-investor'));
  
  // Verify metadata stored
  await waitFor(() => {
    const storedData = AsyncStorage.getItem('interaction-888');
    expect(storedData).toMatchObject({
      interactionId: 'interaction-888',
      tags: [
        expect.objectContaining({
          id: 'tag-investor',
          appliedAt: expect.any(Number),
          confidence: expect.any(Number)
        })
      ]
    });
  });
});

test('should track tag source and timestamp', async () => {
  const { getByTestId } = render(
    <TaggingComponent interactionId="interaction-999" />
  );
  
  const beforeTime = Date.now();
  fireEvent.press(getByTestId('tag-preset-booth'));
  const afterTime = Date.now();
  
  await waitFor(() => {
    const stored = JSON.parse(
      AsyncStorage.getItem('interaction-999')
    );
    const tag = stored.tags[0];
    
    expect(tag.source).toBe('user');
    expect(tag.appliedAt).toBeGreaterThanOrEqual(beforeTime);
    expect(tag.appliedAt).toBeLessThanOrEqual(afterTime);
  });
});
```

**Expected Result**:
- Tag metadata stored with timestamps
- Context preserved
- Source tracked

---

## User Interaction Tests

### 3.1 UI Flow - Tag Application

**Test Case ID**: F8-UX-001  
**Acceptance Criteria**: AC-UX-001, AC-UX-002 (UX - Minimal interaction, feedback within 1s)

**Description**: End-to-end UI flow for applying tags

**Test Steps**:
```typescript
describe('UI Flow - Quick Interaction Tagging', () => {
  test('should display tag picker after tapping tag button', async () => {
    const { getByTestId, getByText } = render(<ConferenceApp />);
    
    // Tap tag button
    fireEvent.press(getByTestId('btn-quick-tag'));
    
    // Verify tag picker visible
    await waitFor(() => {
      expect(getByText('Preset Tags')).toBeTruthy();
      expect(getByText('Investor')).toBeTruthy();
      expect(getByText('Customer')).toBeTruthy();
    });
  });

  test('should show recent tags at top of list', async () => {
    // Set up recent tag history
    await AsyncStorage.setItem('recent-tags', JSON.stringify([
      { id: 'tag-investor', label: 'Investor', lastUsed: Date.now() - 1000 },
      { id: 'tag-customer', label: 'Customer', lastUsed: Date.now() - 5000 }
    ]));
    
    const { getByTestId, getByText } = render(<ConferenceApp />);
    
    fireEvent.press(getByTestId('btn-quick-tag'));
    
    // Verify recent tags section
    const recents = getByTestId('section-recent-tags');
    expect(recents).toBeTruthy();
  });

  test('should enable tag search', async () => {
    const { getByTestId, getByPlaceholderText, getByText, queryByText } = render(
      <ConferenceApp />
    );
    
    fireEvent.press(getByTestId('btn-quick-tag'));
    
    const searchInput = getByPlaceholderText('Search tags');
    fireEvent.changeText(searchInput, 'inv');
    
    // Should filter to 'Investor'
    expect(getByText('Investor')).toBeTruthy();
    // Should hide 'Customer'
    expect(queryByText('Customer')).toBeFalsy();
  });

  test('should show tag confirmation with undo option', async () => {
    const { getByTestId, getByText } = render(<ConferenceApp />);
    
    fireEvent.press(getByTestId('btn-quick-tag'));
    fireEvent.press(getByTestId('tag-preset-investor'));
    
    await waitFor(() => {
      expect(getByText('Tag applied')).toBeTruthy();
      expect(getByTestId('btn-undo')).toBeTruthy();
    });
  });
});
```

**Expected Result**:
- Tag picker displays correctly
- Search functionality works
- Recent tags shown first
- Feedback and undo options displayed

---

### 3.2 Accessibility Testing

**Test Case ID**: F8-ACC-001  
**Acceptance Criteria**: AC-UX-003 (UX - Accessibility supported)

**Description**: Verify accessibility features for tagging interface

**Test Steps**:
```typescript
describe('Accessibility - Quick Interaction Tagging', () => {
  test('should have accessible labels for all tag buttons', async () => {
    const { getByTestId, getByLabelText } = render(
      <TaggingComponent interactionId="interaction-aaa" />
    );
    
    expect(getByLabelText('Apply Investor tag')).toBeTruthy();
    expect(getByLabelText('Apply Customer tag')).toBeTruthy();
    expect(getByLabelText('Create custom tag')).toBeTruthy();
  });

  test('should support screen reader announcements', async () => {
    const { getByTestId, getByText } = render(
      <TaggingComponent interactionId="interaction-bbb" />
    );
    
    fireEvent.press(getByTestId('tag-preset-investor'));
    
    // Verify aria-live region updated
    const liveRegion = getByTestId('aria-live-region');
    await waitFor(() => {
      expect(liveRegion.textContent).toContain('Investor tag applied');
    });
  });

  test('should support keyboard navigation', async () => {
    const { getByTestId, getByLabelText } = render(
      <TaggingComponent interactionId="interaction-ccc" />
    );
    
    // Tab to first tag
    fireEvent.keyDown(getByTestId('tag-list'), { key: 'Tab' });
    expect(document.activeElement).toBe(getByLabelText('Apply Investor tag'));
  });

  test('should have sufficient color contrast', async () => {
    const { getByTestId } = render(
      <TaggingComponent interactionId="interaction-ddd" />
    );
    
    const tagButton = getByTestId('tag-preset-investor');
    const styles = window.getComputedStyle(tagButton);
    
    // Verify WCAG AA contrast (4.5:1 for text)
    expect(getContrastRatio(styles.color, styles.backgroundColor)).toBeGreaterThan(4.5);
  });
});
```

**Expected Result**:
- All interactive elements have accessible labels
- Screen reader announcements work
- Keyboard navigation supported
- Color contrast meets WCAG AA

---

## Edge Case Validation

### 4.1 Error Scenarios

**Test Case ID**: F8-EDGE-001  
**Acceptance Criteria**: AC-F8-006 (Functional - Errors logged correctly)

**Description**: Verify error handling for edge cases

**Test Steps**:
```typescript
describe('Edge Cases - Error Scenarios', () => {
  test('should handle tagging when no active interaction exists', async () => {
    mockInteractionAPI.getActive.mockResolvedValueOnce(null);
    
    const { getByTestId, getByText } = render(<ConferenceApp />);
    
    fireEvent.press(getByTestId('btn-quick-tag'));
    
    await waitFor(() => {
      expect(getByText(/no active.*interaction/i)).toBeTruthy();
    });
  });

  test('should handle tag application with low battery', async () => {
    mockDeviceAPI.getBatteryLevel.mockResolvedValueOnce(5); // 5%
    
    const { getByTestId, getByText } = render(<ConferenceApp />);
    
    fireEvent.press(getByTestId('btn-quick-tag'));
    fireEvent.press(getByTestId('tag-preset-investor'));
    
    await waitFor(() => {
      expect(getByText(/low battery/i)).toBeTruthy();
      // Should still apply tag
      expect(mockTagAPI.applyTag).toHaveBeenCalled();
    });
  });

  test('should handle permission revocation during tagging', async () => {
    mockPermissions.checkStoragePermission.mockResolvedValueOnce(true);
    
    const { getByTestId, getByText } = render(<ConferenceApp />);
    
    fireEvent.press(getByTestId('tag-preset-customer'));
    
    // Simulate permission revoked mid-operation
    mockPermissions.checkStoragePermission.mockResolvedValueOnce(false);
    
    await waitFor(() => {
      expect(getByText(/permission denied/i)).toBeTruthy();
    });
  });

  test('should handle app backgrounding during tag operation', async () => {
    const { getByTestId } = render(<ConferenceApp />);
    
    fireEvent.press(getByTestId('btn-quick-tag'));
    fireEvent.press(getByTestId('tag-preset-meeting'));
    
    // Simulate app backgrounding
    fireEvent(document, 'appBackground');
    
    // Should save state
    const savedState = await AsyncStorage.getItem('tagging-state');
    expect(savedState).toContain('tag-preset-meeting');
  });

  test('should handle duplicate tag application', async () => {
    const { getByTestId, getByText } = render(
      <TaggingComponent 
        interactionId="interaction-eee"
        initialTags={['tag-investor']}
      />
    );
    
    fireEvent.press(getByTestId('tag-preset-investor'));
    
    await waitFor(() => {
      expect(getByText(/already tagged/i)).toBeTruthy();
    });
  });

  test('should handle tag conflict after sync', async () => {
    mockTagAPI.applyTag.mockRejectedValueOnce({
      status: 409,
      message: 'Tag already applied from another device'
    });
    
    const { getByTestId, getByText } = render(<ConferenceApp />);
    
    fireEvent.press(getByTestId('tag-preset-booth'));
    
    await waitFor(() => {
      expect(getByText(/already.*applied/i)).toBeTruthy();
    });
  });
});
```

**Expected Result**:
- All edge cases handled gracefully
- Error messages clear and actionable
- Data integrity maintained
- No crashes or undefined states

---

## Test Code Samples

### Sample Test Setup

```typescript
// setupTests.ts
import '@testing-library/jest-native/extend-expect';
import { server } from './mocks/server';

beforeAll(() => server.listen());
afterEach(() => server.resetHandlers());
afterAll(() => server.close());

// Mock AsyncStorage
jest.mock('@react-native-async-storage/async-storage', () => ({
  __esModule: true,
  default: {
    getItem: jest.fn(),
    setItem: jest.fn(),
    removeItem: jest.fn(),
    clear: jest.fn(),
  },
}));

// Mock Permissions
jest.mock('react-native-permissions', () => ({
  PERMISSIONS: {
    IOS: { MICROPHONE: 'ios.permission.MICROPHONE' },
    ANDROID: { RECORD_AUDIO: 'android.permission.RECORD_AUDIO' },
  },
  RESULTS: {
    GRANTED: 'granted',
    DENIED: 'denied',
  },
  check: jest.fn(),
  request: jest.fn(),
}));
```

### Sample Mock API

```typescript
// mocks/api.ts
export const mockTagAPI = {
  applyTag: jest.fn().mockResolvedValue({ success: true }),
  removeTag: jest.fn().mockResolvedValue({ success: true }),
  createCustomTag: jest.fn().mockResolvedValue({ 
    id: 'tag-custom-1',
    name: 'Custom Tag'
  }),
  getTags: jest.fn().mockResolvedValue([
    { id: 'tag-investor', label: 'Investor' },
    { id: 'tag-customer', label: 'Customer' },
  ]),
};

export const mockInteractionAPI = {
  getActive: jest.fn().mockResolvedValue({
    id: 'interaction-active',
    sessionId: 'session-123'
  }),
  update: jest.fn().mockResolvedValue({ success: true }),
};

export const mockMediaAPI = {
  uploadVoiceTag: jest.fn().mockResolvedValue({ success: true }),
};
```

### Sample E2E Test

```typescript
// e2e/tagging.e2e.ts
describe('Quick Interaction Tagging E2E', () => {
  beforeAll(async () => {
    await device.launchApp();
  });

  beforeEach(async () => {
    await device.reloadReactNative();
  });

  it('should complete full tagging workflow', async () => {
    // Navigate to conference mode
    await element(by.id('btn-conference-mode')).multiTap();
    
    // Tap tag button
    await element(by.id('btn-quick-tag')).tap();
    
    // Select preset tag
    await element(by.id('tag-preset-investor')).tap();
    
    // Verify tag applied
    await expect(element(by.text('Tag applied'))).toBeVisible();
    
    // Verify tag visible on interaction
    await expect(element(by.id('tag-badge-investor'))).toBeVisible();
  });
});
```

---

## Test Execution Guide

### Running Unit Tests
```bash
npm run test:unit -- EPIC01-feature-8-test-cases.spec.ts
npm run test:unit -- --coverage  # Generate coverage report
```

### Running Integration Tests
```bash
npm run test:integration -- EPIC01-feature-8-test-cases.spec.ts
```

### Running E2E Tests
```bash
detox build-framework-cache
detox build-app --configuration ios.sim.debug
detox test e2e/tagging.e2e.ts --configuration ios.sim.debug --cleanup
```

### Test Coverage Target
- **Line Coverage**: >85%
- **Branch Coverage**: >80%
- **Function Coverage**: >90%

---

## Telemetry & Logging

### Events Tracked
- `tag_applied` - Tag successfully applied
- `tag_removed` - Tag removed from interaction
- `custom_tag_created` - Custom tag created
- `voice_tag_created` - Voice note tagged
- `tag_sync_success` - Tag synced to backend
- `tag_sync_failed` - Tag sync failed

### Sample Event Structure
```typescript
{
  eventName: 'tag_applied',
  interactionId: 'interaction-123',
  tagId: 'tag-investor',
  tagType: 'preset',
  timestamp: 1704067200000,
  duration: 450, // ms
  offline: false,
  retryCount: 0
}
```

---

## Test Maintenance

### Review Schedule
- Weekly: Run full test suite
- Monthly: Review test coverage and update edge cases
- Quarterly: Refactor tests for maintainability

### Common Issues & Solutions

| Issue | Solution |
|-------|----------|
| Flaky async tests | Use longer timeouts in CI environment |
| Mock drift | Keep mocks synchronized with API changes |
| Storage conflicts | Clear AsyncStorage between tests |

---

## Sign-Off Checklist

- [ ] Unit tests pass (>85% coverage)
- [ ] Integration tests pass
- [ ] E2E tests pass on iOS and Android
- [ ] Accessibility tests pass
- [ ] Edge cases validated
- [ ] Error scenarios covered
- [ ] Performance targets met
- [ ] Telemetry verified
