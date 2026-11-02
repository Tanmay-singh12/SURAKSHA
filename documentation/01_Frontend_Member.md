# RESQ-ZONE — Coding Agent 1: Frontend

## 1. Your ownership

You own the frontend applications only:

- `idea/citizen-app/` — citizen-facing Android application.
- `idea/government-dashboard/` — web dashboard for government authorities.
- `idea/shared/` — coordinate with the team before editing shared contracts.

Work on branch: `feature/frontend`

Do not modify backend, AI/GIS, or BLE implementation files. If an API or shared model needs to change, raise it with the relevant member and update the shared contract together.

## 2. Product context

**RESQ-ZONE** is an AI-powered multi-hazard risk and offline emergency relocation decision-support platform.

Its workflow is:

**Detect → Assess → Prioritize → Relocate → Communicate**

The frontend must help users answer:

1. Where is unsafe?
2. Which habitations need attention first?
3. Where can people potentially be relocated safely?
4. How can citizens and authorities communicate during connectivity failures?

This is a hackathon MVP. Build a reliable, demonstrable end-to-end experience rather than a large collection of incomplete features.

Important boundaries:
- Risk classifications are decision-support information, not legally binding evacuation orders.
- Authorities remain responsible for approving evacuation and relocation decisions.
- Do not present AI scores as exact disaster predictions.
- BLE is a short-range, best-effort fallback. Never display a message as delivered until the relevant acknowledgement is received.
- Clearly label simulated/demo data.

## 3. Technology stack

### Citizen application
- React Native
- TypeScript
- React Navigation
- Axios
- React Native Maps
- AsyncStorage or SQLite for local persistence
- Firebase Cloud Messaging integration where feasible
- Native Android BLE adapter interface

Do not assume Expo Go supports custom native BLE modules. Choose a workflow that permits Android native modules.

### Government dashboard
- React
- Vite
- TypeScript
- Tailwind CSS
- Axios
- React Router
- Leaflet / React-Leaflet
- Socket.IO client

## 4. Required citizen screens

1. **Login / Registration**
   - Name, email or phone, password.
   - Backend assigns the role; never allow a public user to select an authority role.

2. **Home**
   - Current location and location-permission state.
   - Nearby risk classification and map.
   - Active alerts.
   - Safe-site summary.
   - Clear SOS and Report Hazard actions.

3. **Report Hazard**
   - Hazard type, description, severity, GPS location and optional photo.
   - MVP focus: flood / heavy rainfall.
   - Show submission state and resulting report ID where appropriate.
   - New reports begin as `PENDING`.

4. **SOS**
   - Current location, people count, help required and optional description.
   - Show whether the message is queued locally, relayed to a nearby device, synchronized to the backend, or acknowledged by the server.
   - Do not equate “sent to peer” with “received by authorities”.

5. **Risk Details**
   - Score, category, contributing factors, update timestamp, source and model/rule version.
   - Show a clear note that the score is an indicative decision-support assessment.

6. **Safe Sites**
   - Site name, distance, total capacity, occupancy, remaining capacity and available services.
   - Show safety/suitability information and any exclusion warning supplied by the backend.

7. **Alerts**
   - Title, message, severity, issuing authority, target area, issue time and expiry.
   - Allow acknowledgement where supported.
   - Distinguish issued, received and acknowledged states.

8. **Offline Center**
   - Internet and Bluetooth state.
   - Pending queue count.
   - Nearby peer status.
   - Last synchronization time.
   - Per-message states: `QUEUED`, `SENT_TO_PEER`, `RECEIVED`, `RELAY_PENDING`, `SYNC_PENDING`, `SYNCED`, `EXPIRED`, `FAILED`.

## 5. Required government dashboard screens

1. **Login**
   - Role-protected access.
   - Hide/deny authority features for unauthorized roles.

2. **Command Overview**
   - Incident count.
   - High/critical habitations.
   - Pending citizen reports.
   - Active SOS messages.
   - Safe-site capacity summary.
   - Active alerts.

3. **Live Map**
   - Hazard zones, habitations, reports, SOS messages, safe sites and alerts.
   - Filters by layer, risk category, status and hazard type.
   - Use GeoJSON coordinates in `[longitude, latitude]` order.

4. **Incident Management**
   - Inspect report details and location.
   - Verify, reject or resolve with confirmation.
   - Display reporter-submitted reports as unverified until reviewed.

5. **Habitation Risk / Priority**
   - Sort by priority and risk category.
   - Show population, vulnerability indicators, risk factors and data freshness.

6. **Safe-Site Management**
   - List and inspect sites.
   - Add/update demo sites if permitted by the backend API.
   - Display capacity, occupancy, services and suitability.

7. **Relocation Recommendations**
   - Show proposed habitation-to-site assignments.
   - Explain why a site was selected and any constraints.
   - Provide approve/reject controls only for authorized roles.
   - Make it clear that recommendations are proposals, not automatic evacuation orders.

8. **Alert Composer**
   - Draft, preview and publish alerts.
   - Publishing is restricted to `DISASTER_AUTHORITY` and `ADMIN`.
   - Show expiry and target-area information.

## 6. Shared API contract

Use the backend as the only source of truth. The frontend must never connect directly to MongoDB or call the AI service directly.

Standard success response:

```json
{
  "success": true,
  "data": {},
  "message": "Operation successful"
}
```

Standard error response:

```json
{
  "success": false,
  "message": "Something went wrong",
  "error": { "code": "VALIDATION_ERROR" }
}
```

Initial routes to integrate:

| Method | Route | Purpose |
|---|---|---|
| POST | `/api/auth/register` | Citizen registration |
| POST | `/api/auth/login` | Login |
| GET | `/api/auth/me` | Current user |
| POST | `/api/reports` | Submit hazard report |
| GET | `/api/reports` | List reports |
| PATCH | `/api/reports/:id/status` | Update report status |
| POST | `/api/sos` | Submit SOS |
| GET | `/api/hazards/nearby?lat=&lng=` | Nearby hazard data |
| GET | `/api/habitations/priority` | Priority habitations |
| GET | `/api/safe-sites/nearby?lat=&lng=` | Nearby safe sites |
| GET | `/api/relocation/recommendations` | Proposed plans |
| POST | `/api/relocation/generate` | Request recommendations |
| PATCH | `/api/relocation/:id/approve` | Approve proposal |
| POST | `/api/alerts` | Create draft alert |
| POST | `/api/alerts/:id/publish` | Publish alert |
| GET | `/api/alerts` | List alerts |
| POST | `/api/alerts/:id/acknowledge` | Acknowledge alert |

Coordinate exact request/response fields with the backend member and record them in `idea/shared/api-contract.md`.

## 7. BLE integration

Create a frontend-facing adapter so screens do not depend directly on native BLE implementation details.

Expected operations:
- `initializeBle()`
- `startAdvertising()`
- `startScanning()`
- `stopScanning()`
- `sendEmergencyMessage(packet)`
- `getPendingMessages()`
- `getBleStatus()`
- `syncWithGateway()`

Expected events:
- `onPeerDiscovered`
- `onMessageReceived`
- `onMessageRelayed`
- `onMessageSynced`
- `onBleStateChanged`
- `onTransferFailed`

The BLE member owns the native implementation. Agree on exact types, return values and error states. If native integration is not ready, create a clearly labelled mock adapter with the same interface.

## 8. Shared conventions

- Roles: `CITIZEN`, `FIELD_OFFICER`, `DISASTER_AUTHORITY`, `ADMIN`.
- Report/incident statuses: `PENDING`, `VERIFIED`, `REJECTED`, `RESOLVED`.
- Alert statuses: `DRAFT`, `PUBLISHED`, `EXPIRED`.
- Risk categories: `SAFE`, `LOW`, `MODERATE`, `HIGH`, `CRITICAL`.
- Prototype score bands: 0–19 SAFE, 20–39 LOW, 40–59 MODERATE, 60–79 HIGH, 80–100 CRITICAL. These are prototype thresholds, not official standards.
- GeoJSON coordinate order: `[longitude, latitude]`.
- Keep IDs and field names consistent with `idea/shared/data-models.md`.

## 9. UI quality requirements

- Responsive, clean and accessible interface.
- Consistent typography, spacing, colors and reusable components.
- Loading, empty, error, permission-denied and offline states.
- Avoid fake “live” indicators when data is static.
- Use a visible `DEMO DATA` label for simulated records.
- Keep maps usable on smaller screens.
- Handle API failures gracefully and show retry actions.
- Do not hardcode production credentials or API URLs.

## 10. MVP scope — do not overbuild

Prioritize:
- Authentication.
- Citizen hazard report and SOS.
- Government report review.
- Risk and safe-site map/list.
- Relocation proposal view.
- Alert creation and receipt.
- Offline status and BLE adapter integration.

Defer:
- Complex route navigation.
- Full multilingual content system.
- Advanced analytics and forecasting.
- Continuous tracking of every citizen.
- Production-grade mapping/navigation.
- Complex role/organization administration.

## 11. Git and deliverables

Work only in:
- `idea/citizen-app/`
- `idea/government-dashboard/`
- agreed shared contract files.

Commit example:

```bash
git add idea/citizen-app/ idea/government-dashboard/
git commit -m "feat(frontend): build citizen app and authority dashboard"
git push origin feature/frontend
```

Deliver:
- Working citizen app and dashboard.
- Reusable UI components and API client.
- Map layers and offline states.
- BLE adapter interface or mock.
- `.env.example` files.
- README with setup/run instructions.
- Screenshots or a short demo recording.
- List of API fields/routes consumed.
- List of unfinished features and known limitations.

## 12. First end-to-end milestone

Build the following flow with the backend member:

**Citizen submits a flood report → backend stores it → government dashboard displays it → officer verifies it → verified incident appears on the dashboard.**

Do not wait until all screens are complete before integrating.
