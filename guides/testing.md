# RESQ-ZONE — Testing Strategy and Acceptance Criteria

## 1. Backend tests

- Citizen registration cannot assign authority roles.
- Passwords are hashed.
- Protected routes reject missing/invalid tokens.
- Role restrictions are enforced.
- Report validation and persistence work.
- Report verification records actor and timestamp.
- SOS persistence and rate limiting work.
- BLE message IDs are deduplicated.
- Expired or invalid BLE packets are rejected.
- Alert publishing is restricted.
- Relocation approval is restricted.
- Audit logs are created for sensitive actions.

## 2. AI/GIS tests

- Input feature values are validated.
- Risk score stays within 0–100.
- All classification boundaries are tested.
- Missing features are handled explicitly.
- Geometry and CRS validation works.
- Hard hazard exclusions are respected.
- Remaining capacity is calculated correctly.
- No assignment exceeds safe-site capacity.
- Outputs include explanation, version and timestamp.

## 3. Frontend tests

- Login and logout states.
- Form validation and API error states.
- Report submission and status display.
- SOS state labels accurately reflect backend acknowledgement.
- Map layers and coordinate order.
- Role-based route protection.
- Alert display and acknowledgement.
- Offline state and queue display.
- Empty/loading/error states.

## 4. BLE physical-device tests

- BLE disabled.
- Permission denied.
- Two phones discover each other.
- Two phones transfer a packet.
- Duplicate packet is ignored.
- Expired packet is rejected.
- Hop limit is enforced.
- Packet survives app restart.
- Gateway sync succeeds.
- Failed sync retries.
- Three-phone foreground relay is tested if implemented.

Record phone models, Android versions and actual outcomes. Distinguish physical-device tests from mocks.

## 5. End-to-end acceptance tests

### Test A: Report verification

Citizen report → backend persistence → dashboard visibility → officer verification → updated incident state.

### Test B: Risk assessment

Shared habitation dataset → AI/GIS score → backend persistence → dashboard explanation.

### Test C: Relocation

Priority habitations + safe sites → safety and capacity screening → proposal → authority approval.

### Test D: Offline SOS

Phone A creates SOS → Phone B receives and persists → gateway syncs → backend stores → dashboard displays unverified SOS.

### Test E: Alert

Authority drafts → publishes → backend records → client receives when online → acknowledgement is recorded separately.

## 6. Demo integrity

- Label simulated records.
- Do not claim live data if using static files.
- Do not claim BLE delivery if using a mock.
- Do not claim AI accuracy without evaluation.
- Do not claim an alert reached a device without evidence.
- Do not present a proposed relocation as an executed evacuation.
