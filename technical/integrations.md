# RESQ-ZONE — Integration and External Technology Boundaries

## 1. Integration map

| From | To | Mechanism | Purpose |
|---|---|---|---|
| Citizen app | Backend | HTTPS REST | Auth, reports, SOS, risk and alerts |
| Government dashboard | Backend | HTTPS REST | Operational management |
| Dashboard | Backend | Socket.IO | Real-time authorized updates |
| Backend | MongoDB Atlas | Mongoose | Operational persistence |
| Backend | AI/GIS | Internal HTTP | Risk, priority, safe-site and relocation analysis |
| Backend | Firebase | Firebase Admin SDK | Online push notifications |
| Android BLE device | Nearby Android BLE device | BLE GATT | Local message transfer |
| Gateway device | Backend | HTTPS POST | Upload queued messages |
| GIS pipeline | Public/official datasets | Download/import as supported | Hazard and context layers |

## 2. External data sources

Potential source families include:
- IMD for meteorological information.
- ISRO/Bhuvan/NDEM for geospatial and disaster-related layers.
- DEM providers for elevation.
- OpenStreetMap or other permitted sources for road and facility context.
- Official population/habitation datasets where access and licence permit.

Before integrating a source, record its exact product, access method, terms/licence, coverage, update frequency and limitations. This document does not assert that every named organization provides a usable public API for every required dataset.

## 3. Firebase / push notifications

Use FCM for online notification delivery if configured:
- Backend owns server credentials.
- Mobile app registers/refreshes device token.
- Backend associates token with the authenticated user/device.
- Backend sends notifications after alert publication.
- Delivery and acknowledgement states must be tracked separately.

FCM does not provide a substitute for offline BLE transfer.

## 4. Socket.IO

Use authenticated Socket.IO connections for operational updates:
- Report created/verified.
- SOS created.
- Incident updated.
- Risk updated.
- Safe site updated.
- Relocation updated.
- Alert published.
- Mesh message received.

Authorize each event by role and relevant scope. Avoid broadcasting personal data globally.

## 5. API integration rules

- The backend is the only client-facing API.
- The frontend must not call AI/GIS directly.
- The AI/GIS service must not access user credentials.
- BLE gateway uses the dedicated sync endpoint.
- Every module uses shared IDs and field names.
- API errors and validation behavior must be documented.
- Mock integrations must be clearly labelled and replaceable.

## 6. Integration readiness checklist

- [ ] Shared IDs and coordinate conventions agreed.
- [ ] API request/response schemas agreed.
- [ ] Backend health endpoint works.
- [ ] AI/GIS health endpoint works.
- [ ] Backend can call AI/GIS.
- [ ] Citizen report is visible on dashboard.
- [ ] Authority verification updates the report/incident.
- [ ] Risk output is visible with timestamp and version.
- [ ] Relocation proposals respect site capacity.
- [ ] Alert publication is role-restricted.
- [ ] BLE packet can transfer between two physical devices.
- [ ] Gateway sync deduplicates and persists the packet.
