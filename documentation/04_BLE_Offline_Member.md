# RESQ-ZONE — Coding Agent 4: BLE + Offline Communication

## 1. Your ownership

You own the Android Bluetooth Low Energy (BLE) proof of concept and offline store-and-forward communication module.

Folder: `idea/offline-communication/`  
Branch: `feature/ble-offline`

Do not modify the citizen app, government dashboard or backend implementation directly. Coordinate the native adapter interface with the frontend member and the gateway sync contract with the backend member.

## 2. Product context

RESQ-ZONE must provide a localized communication fallback when internet or cellular connectivity is unavailable.

The intended workflow:

**Create SOS → Save locally → Discover nearby device → Transfer/relay → Store on receiving device → Reach internet gateway → Sync to backend → Display on authority dashboard**

This is a best-effort, short-range system. It is not unlimited-range communication and cannot guarantee message delivery. Do not claim to implement the full Bluetooth Mesh standard.

## 3. Technology stack

- Android
- Kotlin
- Android BLE APIs
- BLE GATT
- React Native native-module integration
- SQLite or another persistent local store
- JSON packet format
- Node backend gateway endpoint: `POST /api/mesh/sync`

Start Android-first. The hackathon proof of concept should work on physical Android devices.

## 4. Suggested project structure

```text
idea/offline-communication/
  android/
    app/
      src/main/java/
  protocol/
    message-schema.md
    gatt-service.md
  src/
    storage/
    relay/
    gateway/
    adapter/
  tests/
  README.md
```

Adjust the exact structure to the chosen native-module setup, but keep protocol documentation separate from implementation.

## 5. Implementation order

Build incrementally:

1. Check BLE hardware and OS capability.
2. Handle runtime permissions.
3. Implement BLE discovery.
4. Define and advertise a custom RESQ-ZONE service.
5. Establish GATT connections.
6. Transfer one small test message between two phones.
7. Persist received messages locally.
8. Add message IDs and deduplication.
9. Add store-and-forward queue.
10. Add gateway upload and backend acknowledgement.
11. Integrate with React Native.
12. Test three-device foreground relay.

Start with two physical Android phones. Do not begin with a complex multi-hop system before a basic two-device transfer works.

## 6. BLE service and GATT design

Define a custom RESQ-ZONE BLE service and document:
- Service UUID.
- Characteristic UUIDs.
- Characteristic properties: read, write, notify/indicate.
- Device/protocol information exchange.
- Emergency message transfer.
- Transfer acknowledgement.
- Sync status where applicable.
- Connection and retry behavior.
- Payload framing and maximum sizes.

Do not assume a complete JSON message will fit in one BLE write. Implement chunking and reassembly if needed, with message ID, chunk index, total chunks and integrity checks.

Do not include personal or emergency details in BLE advertisements. Use advertisements for discovery/service identification; transfer message content through the GATT connection.

## 7. Shared emergency message packet

Use a stable packet schema and coordinate it with the team:

```json
{
  "messageId": "UUID",
  "type": "SOS",
  "originDeviceId": "DEVICE-001",
  "createdAt": "ISO_DATE",
  "expiresAt": "ISO_DATE",
  "location": {
    "latitude": 21.1458,
    "longitude": 79.0882,
    "accuracyMeters": 15
  },
  "priority": "CRITICAL",
  "payload": {
    "peopleCount": 4,
    "helpRequired": "EVACUATION",
    "description": "Need assistance"
  },
  "hopCount": 0,
  "maxHops": 6
}
```

Add a protocol version. Agree with backend/frontend members before changing field names. Store the schema in `idea/shared/data-models.md` or link to `idea/offline-communication/protocol/message-schema.md`.

## 8. Store-and-forward behavior

### Sender
1. Generate a unique `messageId` (UUID).
2. Create packet and timestamps.
3. Persist packet locally before attempting transfer.
4. Add it to the outgoing queue.
5. Scan for participating devices.
6. Attempt transfer and record the acknowledgement state.

### Receiver
1. Receive and reconstruct the packet.
2. Validate schema and protocol version.
3. Check expiry and hop count.
4. Check persistent deduplication store.
5. Persist a new valid message.
6. Acknowledge receipt to the sender.
7. Queue it for relay if eligible.

### Relay
- Do not send a message to a peer known to have already received it, where that state is available.
- Increment `hopCount` on forwarding.
- Stop forwarding if expired or `hopCount >= maxHops`.
- Use bounded retry/backoff.
- Avoid repeatedly forwarding the same packet in a loop.

### Gateway
1. Detect internet connectivity.
2. Upload queued messages to the backend.
3. Process accepted, duplicate and rejected IDs.
4. Mark accepted/known messages as synchronized.
5. Retry transient failures.
6. Do not delete an undelivered message just because an upload failed.

## 9. Local persistence

Persist messages across app restarts.

Suggested fields:
- `messageId`
- `type`
- `packetJson`
- `status`
- `hopCount`
- `createdAt`
- `expiresAt`
- `lastAttemptAt`
- `gatewaySyncedAt`

Suggested states:
- `QUEUED`
- `SENT_TO_PEER`
- `RECEIVED`
- `RELAY_PENDING`
- `SYNC_PENDING`
- `SYNCED`
- `EXPIRED`
- `FAILED`

Maintain a persistent set/index of seen message IDs. A duplicate must not create a second SOS or a second queue item.

## 10. Gateway API contract

Coordinate this exact contract with the backend member.

Endpoint:

`POST /api/mesh/sync`

Example request:

```json
{
  "gatewayDeviceId": "DEVICE-GATEWAY-01",
  "protocolVersion": 1,
  "syncedAt": "ISO_DATE",
  "messages": [
    {
      "messageId": "UUID",
      "type": "SOS",
      "originDeviceId": "DEVICE-001",
      "createdAt": "ISO_DATE",
      "expiresAt": "ISO_DATE",
      "location": {
        "latitude": 21.1458,
        "longitude": 79.0882,
        "accuracyMeters": 15
      },
      "priority": "CRITICAL",
      "payload": {
        "peopleCount": 4,
        "helpRequired": "EVACUATION",
        "description": "Need assistance"
      },
      "hopCount": 2,
      "maxHops": 6
    }
  ]
}
```

Expected response should identify:
- Accepted message IDs.
- Duplicate message IDs.
- Rejected message IDs with reasons.

The backend owns and validates this endpoint. A BLE-originated SOS must appear as an unverified citizen-originated emergency report, not as a verified incident.

## 11. React Native adapter interface

Expose a small, stable interface for the citizen app:

- `initializeBle()`
- `startAdvertising()`
- `startScanning()`
- `stopScanning()`
- `sendEmergencyMessage(packet)`
- `getPendingMessages()`
- `getBleStatus()`
- `syncWithGateway()`

Events:
- `onPeerDiscovered`
- `onMessageReceived`
- `onMessageRelayed`
- `onMessageSynced`
- `onBleStateChanged`
- `onTransferFailed`

Document exact argument types, return values, event payloads and errors. If native integration is delayed, provide a mock adapter with the same interface and clearly label it as simulated.

## 12. Android permissions and platform constraints

Handle:
- `BLUETOOTH_SCAN`
- `BLUETOOTH_ADVERTISE`
- `BLUETOOTH_CONNECT`
- Location permission if required by the Android version and scan implementation.
- Bluetooth disabled.
- Permission denied or revoked.
- Device without BLE support.
- Connection timeout and disconnection.
- App background restrictions.
- Battery optimization and OS process suspension.

Do not promise always-on relay when the app is force-stopped or suspended by the operating system. For the hackathon demo, prioritize a foreground workflow with visible status.

## 13. Security and privacy

- Validate packet structure and payload size.
- Enforce TTL and hop limits.
- Deduplicate by message ID.
- Include protocol version.
- Avoid personal details in advertisements.
- Authenticate gateway uploads using the mechanism agreed with backend.
- Use HTTPS for gateway synchronization.
- Do not claim end-to-end encryption or verified device identity unless implemented and tested.
- Collect only the location and emergency details required for the SOS workflow.

## 14. Testing checklist

Test on physical devices and record actual results:

1. BLE disabled.
2. Permission denied.
3. Two phones discover each other.
4. Two phones exchange a message.
5. Duplicate message is ignored.
6. Expired packet is rejected.
7. Hop limit is enforced.
8. Message survives app restart.
9. Gateway uploads a queued message.
10. Failed upload retries.
11. Three-phone foreground relay.
12. Backend displays the synchronized SOS.

Clearly distinguish physical-device tests from mocked/simulated tests.

## 15. Hackathon demonstration

Demonstrate this exact sequence:

1. Phone A creates an SOS while internet is disabled.
2. Phone A stores the SOS locally.
3. Phone B discovers Phone A and receives the message.
4. Phone B persists it and can relay it.
5. Phone C receives the forwarded message (if three-device relay is implemented).
6. A gateway device regains internet connectivity.
7. Gateway uploads the queued SOS to the backend.
8. Government dashboard displays the SOS as an unverified emergency report.

Demonstrate local transfer before gateway synchronization. Do not fake a successful relay if the devices did not actually exchange the message.

## 16. MVP scope — do not overbuild

Prioritize:
- Two-device BLE discovery and transfer.
- Persistent local message queue.
- Deduplication.
- TTL and hop limit.
- Gateway sync.
- React Native adapter.
- Foreground physical-device demo.

Defer:
- Full Bluetooth Mesh standard.
- Guaranteed background relay.
- Unlimited multi-hop networking.
- Complex cryptographic identity infrastructure unless time and expertise permit.
- iOS implementation.
- Guaranteed delivery or long-range communication.

## 17. Git and deliverables

Work only in `idea/offline-communication/` and agreed shared contract files.

```bash
git add idea/offline-communication/
git commit -m "feat(ble): implement offline SOS relay and gateway sync"
git push origin feature/ble-offline
```

Deliver:
- Android BLE implementation.
- GATT service/characteristic documentation.
- Message protocol documentation.
- Persistent local database and queue.
- Store-and-forward logic.
- Gateway synchronization integration.
- React Native adapter and mock if needed.
- Permission/error handling.
- Physical-device test results.
- README with setup, device requirements and limitations.

## 18. First integration milestone

**Two Android phones discover each other and transfer one SOS packet; the receiver stores it persistently.**

After that works, add relay, deduplication, TTL/hop limits and gateway synchronization.
