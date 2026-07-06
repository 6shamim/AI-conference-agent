# EPIC01 Feature 10 - Conference Session Switching - Comprehensive Test Cases

## Feature Overview
Feature-10 enables users to quickly switch between conference agenda sessions, create ad hoc meetings, and manage media reassignment across sessions during a conference day.

---

## Test Execution Environment

### Setup & Dependencies
- **Platform**: React Native (iOS/Android)
- **Test Framework**: Jest, Detox (E2E), React Testing Library
- **Mock Services**: Calendar API, Session API, Media API
- **Environment Variables**:
```
TEST_ENV=development
API_BASE_URL=http://localhost:3000
AUTO_SUGGEST_SESSIONS=true
```

---

## Unit Test Scenarios

### 1.1 Active Session Selection

**Test Case ID**: F10-UNIT-001  
**Description**: Verify session selector displays and updates active session

**Test Steps**:
```typescript
describe('Session Switching - Active Session Selection', () => {
  test('should display current active session', async () => {
    const mockSession = {
      id: 'session-123',
      title: 'Keynote',
      startTime: Date.now()
    };
    
    const { getByTestId, getByText } = render(
      <SessionSwitcherComponent currentSession={mockSession} />
    );
    
    expect(getByText('Keynote')).toBeTruthy();
    expect(getByTestId('session-indicator')).toHaveStyle({ color: 'green' });
  });

  test('should switch to selected session', async () => {
    const sessions = [
      { id: 'session-123', title: 'Keynote' },
      { id: 'session-456', title: 'Breakout A' }
    ];
    
    const { getByTestId, getByText } = render(
      <SessionSwitcherComponent sessions={sessions} currentSession={sessions[0]} />
    );
    
    fireEvent.press(getByTestId('btn-session-selector'));
    fireEvent.press(getByText('Breakout A'));
    
    await waitFor(() => {
      expect(mockSessionAPI.setActive).toHaveBeenCalledWith('session-456');
    });
  });

  test('should update active session within 1 second (Performance AC)', async () => {
    const { getByTestId, getByText } = render(
      <SessionSwitcherComponent sessions={mockSessions} />
    );
    
    const startTime = Date.now();
    fireEvent.press(getByTestId('btn-session-selector'));
    fireEvent.press(getByText('Breakout B'));
    
    await waitFor(() => {
      expect(mockSessionAPI.setActive).toHaveBeenCalled();
    }, { timeout: 1000 });
    
    const duration = Date.now() - startTime;
    expect(duration).toBeLessThan(1000);
  });

  test('should persist session selection across app restart', async () => {
    const { getByTestId, getByText, unmount } = render(
      <SessionSwitcherComponent sessions={mockSessions} />
    );
    
    fireEvent.press(getByTestId('btn-session-selector'));
    fireEvent.press(getByText('Breakout C'));
    
    // Simulate app restart
    unmount();
    
    const { getByText: getByText2 } = render(
      <SessionSwitcherComponent sessions={mockSessions} />
    );
    
    // Should show previously selected session
    expect(getByText2('Breakout C')).toBeTruthy();
  });
});
```

**Expected Result**: 
- Current session displayed
- Session switch works correctly
- Performance target met
- Selection persists

---

### 1.2 Calendar/Agenda Import

**Test Case ID**: F10-UNIT-002  
**Description**: Verify agenda import and session listing

**Test Steps**:
```typescript
test('should import calendar events as sessions', async () => {
  const mockCalendarEvents = [
    {
      id: 'event-1',
      title: 'Keynote',
      startTime: '09:00',
      endTime: '10:00'
    },
    {
      id: 'event-2',
      title: 'Breakout A',
      startTime: '10:15',
      endTime: '11:15'
    }
  ];
  
  mockCalendarAPI.getEvents.mockResolvedValueOnce(mockCalendarEvents);
  
  const { getByTestId, getByText } = render(
    <SessionSwitcherComponent />
  );
  
  fireEvent.press(getByTestId('btn-import-calendar'));
  
  await waitFor(() => {
    expect(getByText('Keynote')).toBeTruthy();
    expect(getByText('Breakout A')).toBeTruthy();
  });
});

test('should display sessions in chronological order', async () => {
  const { getByTestId, getAllByTestId } = render(
    <SessionSwitcherComponent sessions={mockSessions} />
  );
  
  fireEvent.press(getByTestId('btn-session-list'));
  
  const sessionItems = getAllByTestId('session-list-item');
  
  for (let i = 0; i < sessionItems.length - 1; i++) {
    const time1 = parseInt(sessionItems[i].getAttribute('data-start-time'));
    const time2 = parseInt(sessionItems[i + 1].getAttribute('data-start-time'));
    expect(time1).toBeLessThanOrEqual(time2);
  }
});

test('should handle calendar sync failure', async () => {
  mockCalendarAPI.getEvents.mockRejectedValueOnce({
    status: 401,
    message: 'Unauthorized'
  });
  
  const { getByTestId, getByText } = render(
    <SessionSwitcherComponent />
  );
  
  fireEvent.press(getByTestId('btn-import-calendar'));
  
  await waitFor(() => {
    expect(getByText(/import.*failed/i)).toBeTruthy();
  });
});
```

**Expected Result**:
- Calendar events imported successfully
- Sessions ordered chronologically
- Failures handled gracefully

---

### 1.3 Ad Hoc Session Creation

**Test Case ID**: F10-UNIT-003  
**Description**: Verify ad hoc session creation workflow

**Test Steps**:
```typescript
test('should create ad hoc session in less than 5 seconds', async () => {
  const { getByTestId, getByText, getByPlaceholderText } = render(
    <SessionSwitcherComponent />
  );
  
  const startTime = Date.now();
  
  fireEvent.press(getByTestId('btn-create-session'));
  
  const titleInput = getByPlaceholderText('Session title');
  fireEvent.changeText(titleInput, 'Hallway Meeting');
  
  fireEvent.press(getByText('Create'));
  
  await waitFor(() => {
    expect(mockSessionAPI.create).toHaveBeenCalledWith(
      expect.objectContaining({
        title: 'Hallway Meeting',
        type: 'adhoc'
      })
    );
  });
  
  const duration = Date.now() - startTime;
  expect(duration).toBeLessThan(5000);
});

test('should prevent duplicate session names', async () => {
  const { getByTestId, getByText, getByPlaceholderText, getByDisplayValue } = render(
    <SessionSwitcherComponent sessions={[
      { id: 'session-1', title: 'Meeting A' }
    ]} />
  );
  
  fireEvent.press(getByTestId('btn-create-session'));
  fireEvent.changeText(getByPlaceholderText('Session title'), 'Meeting A');
  fireEvent.press(getByText('Create'));
  
  await waitFor(() => {
    expect(getByText(/already exists/i)).toBeTruthy();
  });
});

test('should validate session title length', async () => {
  const { getByTestId, getByText, getByPlaceholderText } = render(
    <SessionSwitcherComponent />
  );
  
  fireEvent.press(getByTestId('btn-create-session'));
  fireEvent.changeText(
    getByPlaceholderText('Session title'),
    'a'.repeat(256)
  );
  
  expect(getByText(/too long/i)).toBeTruthy();
});
```

**Expected Result**:
- Ad hoc session created quickly
- Duplicates prevented
- Validation enforced

---

### 1.4 Media Reassignment

**Test Case ID**: F10-UNIT-004  
**Description**: Verify media reassignment to different sessions

**Test Steps**:
```typescript
test('should reassign single media item to session', async () => {
  const mockMedia = { id: 'media-1', title: 'Booth Photo' };
  const mockSessions = [
    { id: 'session-1', title: 'Session A' },
    { id: 'session-2', title: 'Session B' }
  ];
  
  const { getByTestId, getByText } = render(
    <MediaReassignmentComponent 
      media={mockMedia}
      sessions={mockSessions}
      currentSession={mockSessions[0]}
    />
  );
  
  fireEvent.press(getByTestId('btn-reassign'));
  fireEvent.press(getByText('Session B'));
  
  await waitFor(() => {
    expect(mockMediaAPI.reassign).toHaveBeenCalledWith(
      'media-1',
      'session-2'
    );
  });
});

test('should support bulk media reassignment', async () => {
  const mockMedia = [
    { id: 'media-1' },
    { id: 'media-2' },
    { id: 'media-3' }
  ];
  
  const { getByTestId, getByText } = render(
    <MediaReassignmentComponent 
      media={mockMedia}
      sessions={mockSessions}
      bulkMode={true}
    />
  );
  
  // Select all media
  fireEvent.press(getByTestId('btn-select-all'));
  
  fireEvent.press(getByTestId('btn-reassign'));
  fireEvent.press(getByText('New Session'));
  
  await waitFor(() => {
    expect(mockMediaAPI.reassign).toHaveBeenCalledTimes(3);
  });
});

test('should handle reassignment failure with rollback', async () => {
  mockMediaAPI.reassign.mockRejectedValueOnce({
    status: 500,
    message: 'Server error'
  });
  
  const { getByTestId, getByText } = render(
    <MediaReassignmentComponent media={mockMedia} sessions={mockSessions} />
  );
  
  fireEvent.press(getByTestId('btn-reassign'));
  fireEvent.press(getByText('New Session'));
  
  await waitFor(() => {
    expect(getByText(/reassignment.*failed/i)).toBeTruthy();
  });
});
```

**Expected Result**:
- Media reassigned successfully
- Bulk operations supported
- Failures handled with rollback

---

### 1.5 Auto-Suggest Current Session

**Test Case ID**: F10-UNIT-005  
**Description**: Verify auto-suggestion of current session based on time

**Test Steps**:
```typescript
test('should auto-suggest session based on current time', async () => {
  const now = new Date();
  const mockSessions = [
    {
      id: 'session-1',
      title: 'Keynote',
      startTime: new Date(now.getTime() - 10 * 60000), // 10 min ago
      endTime: new Date(now.getTime() + 50 * 60000) // 50 min from now
    },
    {
      id: 'session-2',
      title: 'Breakout',
      startTime: new Date(now.getTime() + 60 * 60000), // 60 min from now
      endTime: new Date(now.getTime() + 120 * 60000)
    }
  ];
  
  const { getByTestId, getByText } = render(
    <SessionSwitcherComponent sessions={mockSessions} />
  );
  
  await waitFor(() => {
    expect(getByText('Keynote (current)')).toBeTruthy();
  });
});

test('should allow user to dismiss auto-suggestion', async () => {
  const { getByTestId, getByText } = render(
    <SessionSwitcherComponent sessions={mockSessions} autoSuggest={true} />
  );
  
  fireEvent.press(getByTestId('btn-dismiss-suggestion'));
  
  const suggestion = getByTestId('auto-suggestion');
  expect(suggestion).toHaveStyle({ display: 'none' });
});
```

**Expected Result**:
- Correct session auto-suggested
- User can dismiss suggestion

---

## Integration Test Scenarios

### 2.1 Session Context Persistence

**Test Case ID**: F10-INT-001  
**Description**: Verify session context maintained across operations

**Test Steps**:
```typescript
describe('Session Context Persistence', () => {
  test('should maintain session context during media capture', async () => {
    const { getByTestId } = render(
      <ConferenceApp currentSession={{ id: 'session-123', title: 'Breakout' }} />
    );
    
    // Capture media
    fireEvent.press(getByTestId('btn-capture'));
    
    // Switch session
    fireEvent.press(getByTestId('btn-session-selector'));
    fireEvent.press(getByText('New Session'));
    
    // Verify captured media linked to original session
    const media = await mockMediaAPI.getRecent();
    expect(media.sessionId).toBe('session-123');
  });

  test('should update new captures to new session', async () => {
    const { getByTestId, getByText } = render(
      <ConferenceApp />
    );
    
    fireEvent.press(getByTestId('btn-session-selector'));
    fireEvent.press(getByText('Session B'));
    
    // Capture media
    fireEvent.press(getByTestId('btn-capture'));
    
    const recentMedia = await mockMediaAPI.getRecent();
    expect(recentMedia.sessionId).toBe('session-b-id');
  });
});
```

**Expected Result**:
- Session context maintained
- Media linked to correct session
- Captures use active session

---

### 2.2 Offline Session Management

**Test Case ID**: F10-INT-002  
**Description**: Verify offline session switching support

**Test Steps**:
```typescript
test('should support offline session switching', async () => {
  mockNetworkAPI.setOnline(false);
  
  const { getByTestId, getByText } = render(
    <SessionSwitcherComponent sessions={mockSessions} />
  );
  
  fireEvent.press(getByTestId('btn-session-selector'));
  fireEvent.press(getByText('New Session'));
  
  // Should update locally
  const activeSession = await AsyncStorage.getItem('active-session');
  expect(JSON.parse(activeSession).id).toContain('new-session');
});

test('should sync session changes when online', async () => {
  mockNetworkAPI.setOnline(false);
  
  const { getByTestId, getByText, rerender } = render(
    <SessionSwitcherComponent sessions={mockSessions} />
  );
  
  fireEvent.press(getByTestId('btn-session-selector'));
  fireEvent.press(getByText('New Session'));
  
  mockNetworkAPI.setOnline(true);
  rerender(<SessionSwitcherComponent sessions={mockSessions} />);
  
  await waitFor(() => {
    expect(mockSessionAPI.setActive).toHaveBeenCalled();
  });
});
```

**Expected Result**:
- Offline switching works
- Changes sync when online

---

## User Interaction Tests

### 3.1 UI Flow - Session Switching

**Test Case ID**: F10-UX-001  
**Description**: End-to-end session switching workflow

**Test Steps**:
```typescript
describe('UI Flow - Session Switching', () => {
  test('should display current session in header', async () => {
    const { getByTestId, getByText } = render(
      <ConferenceApp currentSession={{ id: 'session-1', title: 'Keynote' }} />
    );
    
    expect(getByText('Keynote')).toBeTruthy();
    expect(getByTestId('session-status')).toHaveStyle({ color: 'green' });
  });

  test('should show session list modal', async () => {
    const { getByTestId, getByText } = render(
      <SessionSwitcherComponent sessions={mockSessions} />
    );
    
    fireEvent.press(getByTestId('btn-session-selector'));
    
    await waitFor(() => {
      expect(getByText('Select Session')).toBeTruthy();
      mockSessions.forEach(session => {
        expect(getByText(session.title)).toBeTruthy();
      });
    });
  });

  test('should highlight current session in list', async () => {
    const { getByTestId, getByText } = render(
      <SessionSwitcherComponent 
        sessions={mockSessions}
        currentSession={mockSessions[1]}
      />
    );
    
    fireEvent.press(getByTestId('btn-session-selector'));
    
    const currentItem = getByText(mockSessions[1].title).parentElement;
    expect(currentItem).toHaveStyle({ backgroundColor: expect.any(String) });
  });

  test('should enable quick session switch from lock screen control', async () => {
    const { getByTestId } = render(<ConferenceApp />);
    
    fireEvent.press(getByTestId('lockscreen-session-quick-switch'));
    
    // Should present session selector
    await waitFor(() => {
      expect(getByTestId('session-selector-modal')).toBeTruthy();
    });
  });
});
```

**Expected Result**:
- Current session visible
- Session list displays correctly
- Quick switching available

---

### 3.2 Accessibility

**Test Case ID**: F10-ACC-001  
**Description**: Verify accessibility features

**Test Steps**:
```typescript
describe('Accessibility - Session Switching', () => {
  test('should announce session change to screen reader', async () => {
    const { getByTestId, getByText } = render(
      <SessionSwitcherComponent sessions={mockSessions} />
    );
    
    fireEvent.press(getByTestId('btn-session-selector'));
    fireEvent.press(getByText('New Session'));
    
    await waitFor(() => {
      expect(getByTestId('aria-live-region')).toHaveTextContent('Session switched to New Session');
    });
  });

  test('should support keyboard navigation', async () => {
    const { getByTestId } = render(
      <SessionSwitcherComponent sessions={mockSessions} />
    );
    
    fireEvent.press(getByTestId('btn-session-selector'));
    
    // Tab through sessions
    fireEvent.keyDown(getByTestId('session-list'), { key: 'ArrowDown' });
    expect(document.activeElement).toHaveAttribute('data-session-index', '1');
  });
});
```

**Expected Result**:
- Screen reader announcements work
- Keyboard navigation supported

---

## Edge Case Validation

### 4.1 Error Scenarios

**Test Case ID**: F10-EDGE-001  
**Description**: Verify error handling

**Test Steps**:
```typescript
describe('Edge Cases - Session Switching Error Scenarios', () => {
  test('should handle overlapping sessions', async () => {
    const overlappingSessions = [
      {
        id: 'session-1',
        title: 'Keynote',
        startTime: new Date('2024-01-01T09:00'),
        endTime: new Date('2024-01-01T10:30')
      },
      {
        id: 'session-2',
        title: 'Breakout',
        startTime: new Date('2024-01-01T10:00'),
        endTime: new Date('2024-01-01T11:00')
      }
    ];
    
    const { getByTestId } = render(
      <SessionSwitcherComponent sessions={overlappingSessions} />
    );
    
    fireEvent.press(getByTestId('btn-session-selector'));
    
    // Should show warning for overlapping sessions
    expect(getByText(/overlaps/i)).toBeTruthy();
  });

  test('should handle empty session list', async () => {
    const { getByTestId, getByText } = render(
      <SessionSwitcherComponent sessions={[]} />
    );
    
    fireEvent.press(getByTestId('btn-session-selector'));
    
    expect(getByText(/no sessions/i)).toBeTruthy();
  });

  test('should handle failed session reassignment', async () => {
    mockMediaAPI.reassign.mockRejectedValueOnce({
      status: 409,
      message: 'Media already reassigned'
    });
    
    const { getByTestId, getByText } = render(
      <MediaReassignmentComponent media={mockMedia} sessions={mockSessions} />
    );
    
    fireEvent.press(getByTestId('btn-reassign'));
    fireEvent.press(getByText('Target Session'));
    
    await waitFor(() => {
      expect(getByText(/reassignment.*failed/i)).toBeTruthy();
    });
  });

  test('should handle session deletion while active', async () => {
    const { getByTestId } = render(
      <SessionSwitcherComponent currentSession={mockSessions[0]} />
    );
    
    // Simulate deletion
    mockSessionAPI.delete.mockResolvedValueOnce({ success: true });
    
    fireEvent.press(getByTestId('btn-delete-session'));
    
    await waitFor(() => {
      expect(mockSessionAPI.setActive).toHaveBeenCalledWith(mockSessions[1].id);
    });
  });
});
```

**Expected Result**:
- Overlapping sessions handled
- Empty states managed
- Failures handled gracefully
- Active session fallback works

---

## Test Execution Guide

```bash
npm run test:unit -- EPIC01-feature-10-test-cases.spec.ts
npm run test:integration -- EPIC01-feature-10-test-cases.spec.ts
detox test e2e/session-switching.e2e.ts --configuration ios.sim.debug
```

### Coverage Target
- **Line Coverage**: >85%
- **Branch Coverage**: >80%
- **Function Coverage**: >90%

---

## Sign-Off Checklist

- [ ] Unit tests pass (>85% coverage)
- [ ] Integration tests pass
- [ ] E2E tests pass (iOS & Android)
- [ ] Accessibility tests pass
- [ ] Edge cases validated
- [ ] Performance targets met
- [ ] Telemetry verified
