# RESQ-ZONE — Shared API Contract (MVP)

> This is the planned shared contract. The implemented backend routes and schemas are authoritative; update this file whenever implementation changes.

## 1. Base conventions

- Backend API prefix: `/api`
- Transport: HTTPS in deployed environments.
- Authentication: `Authorization: Bearer <JWT>` for protected routes.
- JSON request/response unless multipart upload is explicitly used.
- GeoJSON coordinate order: `[longitude, latitude]`.

Success envelope:

```json
{
  "success": true,
  "data": {},
  "message": "Operation successful"
}
```

Error envelope:

```json
{
  "success": false,
  "message": "Something went wrong",
  "error": {
    "code": "VALIDATION_ERROR"
  }
}
```

## 2. Route inventory

| Method | Route | Access | Purpose |
|---|---|---|---|
| POST | `/auth/register` | Public; creates CITIZEN only | Register citizen |
| POST | `/auth/login` | Public | Login |
| GET | `/auth/me` | Authenticated | Current user |
| POST | `/reports` | Citizen/authenticated | Submit report |
| GET | `/reports` | Role-scoped | List reports |
| GET | `/reports/:id` | Role-scoped | Report detail |
| PATCH | `/reports/:id/status` | Authorized officer | Verify/reject/resolve |
| POST | `/sos` | Authenticated | Submit SOS |
| GET | `/sos` | Role-scoped | List SOS |
| PATCH | `/sos/:id/status` | Authorized role | Update SOS status |
| GET | `/hazards` | Authenticated/public policy to be finalized | List hazard zones |
| GET | `/hazards/nearby?lat=&lng=` | Authenticated/public policy to be finalized | Nearby hazards |
| GET | `/habitations` | Authorized | List habitations |
| GET | `/habitations/priority` | Authorized | Prioritized habitations |
| GET | `/safe-sites` | Authenticated/authorized | List sites |
| GET | `/safe-sites/nearby?lat=&lng=` | Authenticated | Nearby sites |
| POST | `/safe-sites` | Authorized | Add site |
| PATCH | `/safe-sites/:id` | Authorized | Update site |
| GET | `/relocation/recommendations` | Authorized | List proposals |
| POST | `/relocation/generate` | Authorized | Generate proposals |
| PATCH | `/relocation/:id/approve` | DISASTER_AUTHORITY/ADMIN | Approve proposal |
| PATCH | `/relocation/:id/reject` | DISASTER_AUTHORITY/ADMIN | Reject proposal |
| POST | `/alerts` | DISASTER_AUTHORITY/ADMIN | Create draft |
| GET | `/alerts` | Role-scoped | List alerts |
| GET | `/alerts/:id` | Role-scoped | Alert detail |
| POST | `/alerts/:id/publish` | DISASTER_AUTHORITY/ADMIN | Publish |
| POST | `/alerts/:id/acknowledge` | Citizen/authenticated | Acknowledge |
| POST | `/mesh/sync` | Authenticated gateway | Synchronize BLE packets |
| GET | `/mesh/status` | Authorized | Mesh status |

## 3. AI/GIS internal service

| Method | Route | Purpose |
|---|---|---|
| GET | `/health` | Service health |
| POST | `/predict-risk` | Score one habitation |
| POST | `/analyze-region` | Analyze region and prioritize habitations |
| POST | `/safe-sites/evaluate` | Evaluate safe-site suitability |
| POST | `/relocation/recommend` | Generate proposed assignments |

The frontend must never call this service directly.

## 4. Risk request example

```json
{
  "habitationId": "HAB-001",
  "features": {
    "hazardExposure": 0.8,
    "rainfallTrigger": 0.7,
    "populationExposure": 0.6,
    "vulnerability": 0.5,
    "evacuationDifficulty": 0.7,
    "infrastructureGap": 0.4
  }
}
```

## 5. Risk response example

```json
{
  "habitationId": "HAB-001",
  "riskScore": 72,
  "classification": "HIGH",
  "factors": [],
  "modelVersion": "risk-v1",
  "generatedAt": "ISO_DATE"
}
```

## 6. BLE gateway request example

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

Response must return accepted, duplicate and rejected message IDs, including rejection reasons.

## 7. Contract ownership

- Backend member owns REST route implementation and final server response schema.
- AI/GIS member owns internal FastAPI schemas; backend integration must agree with them.
- BLE member owns packet transport implementation; packet schema and sync envelope must be agreed with backend.
- Frontend member consumes documented backend APIs.
- Shared changes must be communicated before merging.
