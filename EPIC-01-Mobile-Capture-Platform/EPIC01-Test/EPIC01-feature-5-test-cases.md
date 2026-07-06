# EPIC01 Feature 5: Offline Buffering - Comprehensive Test Cases

## Feature Overview
Offline Buffering ensures capture works without reliable network connectivity through local encrypted queue, retry sync, and conflict-safe upload mechanisms.

---

## Part 1: Unit Test Scenarios

### Test Suite: OfflineQueueManager

#### UT-5.1: Queue Item Creation
**Objective**: Verify queue items are created with proper encryption
**Prerequisites**: App initialized, encryption service available
**Test Steps**:
1. Create offline queue item with audio payload
2. Verify item encrypted locally
3. Confirm metadata stored (id, type, status, created_at)
4. Validate SQLite record created

**Expected Result**: Item stored in encrypted queue with "pending" status

**Code Sample**:
```typescript
describe('OfflineQueueManager', () => {
  let queueManager: OfflineQueueManager;
  let encryptionService: EncryptionService;

  beforeEach(() => {
    encryptionService = new EncryptionService();
    queueManager = new OfflineQueueManager(encryptionService);
  });

  test('should create and encrypt queue item', async () => {
    const payload = { audio: 'base64encodeddata', metadata: { duration: 120 } };
    const queueItem = await queueManager.createQueueItem('audio', payload);

    expect(queueItem).toBeDefined();
    expect(queueItem.status).toBe('pending');
    expect(queueItem.encrypted).toBe(true);
    expect(queueItem.created_at).toBeLessThanOrEqual(Date.now());
  });
});
```

#### UT-5.2: Queue Persistence
**Objective**: Verify queue survives app termination
**Test Steps**:
1. Create queue item
2. Terminate app
3. Restart app
4. Query queue
5. Verify item still present

**Expected Result**: Queue item recovered with same id and status

#### UT-5.3: Retry Logic with Exponential Backoff
**Objective**: Verify retry intervals increase exponentially
**Test Steps**:
1. Create queue item
2. Simulate sync failure (retry_count=0)
3. Calculate next retry time: 2^0 * 30s = 30s
4. Simulate second failure (retry_count=1)
5. Calculate: 2^1 * 30s = 60s
6. Verify exponential increase up to max 3600s

**Expected Result**: Retry intervals: 30s, 60s, 120s, 240s, 480s, 960s, 1800s, 3600s (capped)

**Code Sample**:
```typescript
test('should apply exponential backoff for retries', async () => {
  const baseInterval = 30000; // 30 seconds in ms
  const maxInterval = 3600000; // 1 hour cap

  for (let retryCount = 0; retryCount <= 8; retryCount++) {
    const nextRetryTime = Math.min(
      baseInterval * Math.pow(2, retryCount),
      maxInterval
    );
    
    expect(nextRetryTime).toBeGreaterThan(0);
    if (retryCount < 8) {
      expect(nextRetryTime).toBeLessThan(maxInterval);
    }
  }
});
```

#### UT-5.4: Encryption/Decryption
**Objective**: Verify data encrypted at rest and decrypted for sync
**Test Steps**:
1. Create payload with sensitive data
2. Encrypt using symmetric key
3. Verify ciphertext differs from plaintext
4. Decrypt
5. Verify plaintext matches original

**Expected Result**: Roundtrip encryption/decryption successful

#### UT-5.5: Queue Status Transitions
**Objective**: Verify valid state transitions
**Test Steps**:
1. Create item (status: pending)
2. Start sync (status: syncing)
3. Confirm sync (status: synced)
4. Attempt invalid transition (synced -> pending)

**Expected Result**: Valid transitions allowed, invalid transitions rejected

---

## Part 2: Integration Test Scenarios

### Test Suite: Offline Sync Pipeline

#### IT-5.1: End-to-End Offline Capture and Sync
**Objective**: Verify complete offline workflow
**Prerequisites**: 
- Network disabled (use airplane mode or mock)
- Auth token available
- Conference session active

**Test Steps**:
1. Disable network connectivity
2. Record audio for 10 seconds
3. Capture image metadata
4. Verify items in offline queue (count=2)
5. Enable network connectivity
6. Trigger manual sync
7. Verify sync success telemetry
8. Confirm items marked as synced

**Expected Result**: All offline items synced successfully, queue cleared

**Code Sample**:
```typescript
describe('Offline Sync Pipeline', () => {
  let captureService: CaptureService;
  let networkService: NetworkService;
  let syncManager: SyncManager;
  let storageService: StorageService;

  beforeEach(async () => {
    captureService = new CaptureService();
    networkService = new NetworkService();
    syncManager = new SyncManager();
    storageService = new StorageService();
    await storageService.initialize();
  });

  test('end-to-end offline capture and sync', async () => {
    // Simulate offline
    networkService.setConnectivity(false);
    expect(networkService.isOnline()).toBe(false);

    // Capture offline
    const audioData = await captureService.recordAudio({ duration: 10 });
    const queuedItems = await storageService.getQueueItems();
    expect(queuedItems.length).toBe(1);
    expect(queuedItems[0].status).toBe('pending');

    // Go online and sync
    networkService.setConnectivity(true);
    const syncResult = await syncManager.syncQueue();
    
    expect(syncResult.success).toBe(true);
    expect(syncResult.itemsSynced).toBe(1);
    
    const remainingItems = await storageService.getQueueItems();
    expect(remainingItems.length).toBe(0);
  });
});
```

#### IT-5.2: Conflict Resolution - Duplicate Detection
**Objective**: Prevent duplicate uploads
**Test Steps**:
1. Create queue item with unique ID
2. Simulate network timeout during sync
3. Retry sync (item already on server)
4. Verify duplicate detection on server
5. Server returns conflict status
6. Verify item not duplicated

**Expected Result**: Duplicate rejected gracefully, item status updated to synced

#### IT-5.3: Partial Sync Recovery
**Objective**: Handle interruptions during multi-item sync
**Test Steps**:
1. Queue 5 items for sync
2. Successfully sync items 1-3
3. Network fails on item 4
4. Reconnect
5. Verify item 4 retried
6. Verify item 5 still queued

**Expected Result**: Partial sync recorded, remaining items retry from failure point

#### IT-5.4: Storage Quota Enforcement
**Objective**: Prevent queue from exceeding storage limits
**Prerequisites**: Storage quota = 500MB
**Test Steps**:
1. Queue items totaling 450MB
2. Attempt to queue 100MB item
3. Verify operation succeeds (under quota)
4. Attempt to queue 60MB item
5. Verify operation rejected (quota exceeded)
6. User receives notification

**Expected Result**: User warned, oldest pending items auto-deleted if necessary

#### IT-5.5: Authentication Token Refresh
**Objective**: Handle token expiration during sync
**Test Steps**:
1. Queue item
2. Start sync
3. Auth token expires mid-sync
4. Verify token refresh triggered
5. Retry sync with new token
6. Verify sync succeeds

**Expected Result**: Token refreshed transparently, sync continues

---

## Part 3: Resource Monitoring Tests

### Battery Impact Tests

#### BT-5.1: Queue Processing Battery Drain
**Objective**: Measure battery impact of queue operations
**Prerequisites**: Battery monitoring enabled, test device with known battery capacity
**Test Steps**:
1. Log initial battery percentage
2. Add 50 items to queue over 5 minutes
3. Monitor battery drain
4. Calculate mAh per operation

**Expected Result**: Battery drain <5% for 50 queue operations over 5 minutes

**Code Sample**:
```typescript
test('battery monitoring during queue operations', async () => {
  const batteryMonitor = new BatteryMonitor();
  const initialBattery = await batteryMonitor.getBatteryPercentage();

  // Simulate 50 queue operations
  for (let i = 0; i < 50; i++) {
    await queueManager.createQueueItem('audio', { sample: i });
    if (i % 10 === 0) {
      await new Promise(resolve => setTimeout(resolve, 100)); // Realistic spacing
    }
  }

  const finalBattery = await batteryMonitor.getBatteryPercentage();
  const drain = initialBattery - finalBattery;
  
  expect(drain).toBeLessThan(5); // Less than 5% drain
});
```

#### BT-5.2: Idle Battery Consumption
**Objective**: Measure battery drain when queue is idle
**Test Steps**:
1. Queue 10 items
2. Disable automatic sync
3. Monitor battery for 1 hour
4. Measure drain

**Expected Result**: <0.5% drain per hour (negligible)

### Memory Impact Tests

#### MT-5.1: Queue Memory Footprint
**Objective**: Verify queue doesn't cause memory leaks
**Test Steps**:
1. Get baseline memory (heap size)
2. Add 1000 items to queue
3. Measure heap growth
4. Clear queue
5. Measure heap after garbage collection

**Expected Result**: <50MB growth for 1000 items, recovered after clearing

**Code Sample**:
```typescript
test('queue memory footprint with 1000 items', async () => {
  if (global.gc) {
    global.gc(); // Force garbage collection
  }

  const initialMemory = (performance.memory?.usedJSHeapSize || 0) / 1024 / 1024;

  // Add 1000 items
  const items = [];
  for (let i = 0; i < 1000; i++) {
    items.push(await queueManager.createQueueItem('audio', { 
      index: i,
      data: new Array(1000).fill('x')
    }));
  }

  const peakMemory = (performance.memory?.usedJSHeapSize || 0) / 1024 / 1024;
  const growth = peakMemory - initialMemory;

  expect(growth).toBeLessThan(50); // Less than 50MB growth

  // Cleanup
  await queueManager.clearQueue();
  if (global.gc) {
    global.gc();
  }

  const finalMemory = (performance.memory?.usedJSHeapSize || 0) / 1024 / 1024;
  expect(finalMemory).toBeLessThan(initialMemory + 10); // ~Initial + 10MB overhead
});
```

### CPU Impact Tests

#### CT-5.1: Sync Operation CPU Usage
**Objective**: Verify sync doesn't max out CPU
**Test Steps**:
1. Start CPU monitor
2. Queue 20 items
3. Trigger sync
4. Monitor CPU usage (should not spike >70%)
5. Verify UI remains responsive

**Expected Result**: CPU usage <70%, UI responsive (60 FPS maintained)

#### CT-5.2: Encryption/Decryption CPU Load
**Objective**: Measure CPU overhead of encryption operations
**Test Steps**:
1. Monitor CPU
2. Encrypt 100 items sequentially
3. Measure average CPU time per item
4. Decrypt and verify

**Expected Result**: <50ms per item encryption (efficient crypto library)

---

## Part 4: Edge Case Validation

### EC-5.1: Storage Full Condition
**Objective**: Verify graceful handling when storage is full
**Test Steps**:
1. Fill device storage to 95%
2. Attempt to queue new item
3. Verify user receives warning
4. Offer options: clear oldest items or reduce quality
5. If cleared, verify sync not impacted

**Expected Result**: Graceful degradation with user notification

### EC-5.2: User Logout While Offline
**Objective**: Verify queue handling when user logs out offline
**Test Steps**:
1. Queue items while offline
2. User logs out
3. Logout completes successfully
4. Verify queue items deleted (privacy)
5. User logs back in
6. Verify empty queue

**Expected Result**: Queue cleared, no unauthorized sync

### EC-5.3: Sync with Deleted Item
**Objective**: Prevent syncing of deleted items
**Test Steps**:
1. Queue item (status: pending)
2. User deletes item locally (status: deleted)
3. Sync starts
4. Verify deleted item NOT synced to server
5. Item removed from queue

**Expected Result**: Deleted items never reach server

### EC-5.4: Multiple Retries Exhaustion
**Objective**: Verify handling after max retries exceeded
**Test Steps**:
1. Create queue item
2. Simulate network failures 8 times (max retries)
3. Verify retry_count = 8
4. Trigger sync again
5. Verify item moves to "failed" state
6. User receives notification to manually retry or delete

**Expected Result**: Item stuck in failed state, manual intervention option shown

### EC-5.5: Concurrent Offline and Online Sessions
**Objective**: Handle user in transit (switching networks)
**Test Steps**:
1. Start capturing offline
2. Queue items
3. Switch to cellular network (weak signal)
4. Attempt sync (may fail or succeed partially)
5. Switch back to offline
6. Queue more items
7. Eventually sync when on strong WiFi

**Expected Result**: All items eventually synced, no loss

---

## Part 5: Telemetry and Logging

### Telemetry Events
```
- offline_item_created { type, size, timestamp }
- offline_sync_started { item_count, total_size }
- offline_sync_completed { synced_count, failed_count, duration }
- offline_sync_failed { error_code, last_error, retry_count }
- storage_quota_exceeded { current_size, quota_limit }
- offline_item_deleted { id, reason }
```

### Log Levels
```
DEBUG: Queue operations (add/remove)
INFO: Sync started/completed
WARNING: Quota near limit, max retries exceeded
ERROR: Sync failed, storage full, decryption failed
```

---

## Part 6: Performance Benchmarks

| Metric | Target | Tolerance |
|--------|--------|-----------|
| Queue item creation | <50ms | ±10ms |
| Encrypt operation (1MB) | <100ms | ±20ms |
| Sync single item | <500ms | ±100ms |
| Sync 100 items | <30s | ±5s |
| Memory per 100 items | <10MB | ±2MB |
| Battery drain (1hr idle) | <0.5% | ±0.2% |

---

## Part 7: Test Environment Setup

### Mock Dependencies
```typescript
class MockNetworkService {
  private isConnected = true;
  setConnectivity(value: boolean) { this.isConnected = value; }
  isOnline() { return this.isConnected; }
}

class MockEncryptionService {
  async encrypt(data: any) { return Buffer.from(JSON.stringify(data)).toString('base64'); }
  async decrypt(encrypted: string) { return JSON.parse(Buffer.from(encrypted, 'base64').toString()); }
}

class MockStorageService {
  private queue: QueueItem[] = [];
  async addQueueItem(item: QueueItem) { this.queue.push(item); }
  async getQueueItems() { return this.queue; }
  async clearQueue() { this.queue = []; }
}
```

---

## Execution Notes
- Run on iOS 14+ and Android 10+ devices
- Test with various network conditions (2G, 3G, 4G, WiFi, offline)
- Verify on low-end devices (2GB RAM, 10% battery)
- Test concurrent sync and capture operations
