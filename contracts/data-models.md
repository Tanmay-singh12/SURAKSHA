# RESQ-ZONE — Shared Data Models

## 1. Coordinate conventions

- GeoJSON Point coordinates: `[longitude, latitude]`.
- BLE location object: separate `latitude` and `longitude` numeric fields.
- Use projected CRS for metric GIS calculations.
- Do not interchange GeoJSON coordinate order and BLE latitude/longitude fields.

## 2. User

```json
{
  "id": "USER-ID",
  "name": "Example Citizen",
  "email": "citizen@example.com",
  "phone": "optional",
  "role": "CITIZEN",
  "status": "ACTIVE",
  "createdAt": "ISO_DATE"
}
```

Password hash is internal and must never be returned to clients.

## 3. Citizen report

```json
{
  "id": "REPORT-ID",
  "reporterId": "USER-ID",
  "hazardType": "FLOOD",
  "description": "Water level rising near the road",
  "severity": "HIGH",
  "location": {
    "type": "Point",
    "coordinates": [79.0882, 21.1458]
  },
  "media": [],
  "status": "PENDING",
  "createdAt": "ISO_DATE"
}
```

## 4. SOS

```json
{
  "id": "SOS-ID",
  "reporterId": "USER-ID",
  "originDeviceId": "DEVICE-001",
  "location": {
    "type": "Point",
    "coordinates": [79.0882, 21.1458]
  },
  "peopleCount": 4,
  "helpRequired": "EVACUATION",
  "description": "Need assistance",
  "source": "BLE",
  "status": "RECEIVED_UNVERIFIED",
  "receivedAt": "ISO_DATE"
}
```

The exact SOS status enum should be finalized by the backend team. Do not treat a BLE-originated SOS as a verified incident.

## 5. Habitation

```json
{
  "id": "HAB-001",
  "name": "Example Habitation",
  "location": {
    "type": "Point",
    "coordinates": [79.0882, 21.1458]
  },
  "population": 1200,
  "riskScore": 72,
  "classification": "HIGH",
  "priority": 1,
  "riskFactors": {},
  "updatedAt": "ISO_DATE"
}
```

## 6. Hazard zone

```json
{
  "id": "HZ-001",
  "hazardType": "FLOOD",
  "geometry": {
    "type": "Polygon",
    "coordinates": []
  },
  "riskScore": 82,
  "classification": "CRITICAL",
  "factors": {},
  "modelVersion": "risk-v1",
  "sourceTimestamp": "ISO_DATE",
  "validUntil": "ISO_DATE"
}
```

## 7. Safe site

```json
{
  "id": "SITE-001",
  "name": "Example Relief Centre",
  "location": {
    "type": "Point",
    "coordinates": [79.09, 21.15]
  },
  "totalCapacity": 500,
  "currentOccupancy": 120,
  "remainingCapacity": 380,
  "services": {
    "water": true,
    "sanitation": true,
    "medical": true,
    "food": false
  },
  "safetyScore": 0.9,
  "suitabilityScore": 0.86,
  "status": "AVAILABLE"
}
```

`remainingCapacity` is derived from total capacity and current occupancy and should be validated.

## 8. Relocation plan

```json
{
  "id": "PLAN-001",
  "habitationId": "HAB-001",
  "safeSiteId": "SITE-001",
  "peopleToRelocate": 300,
  "explanation": "Candidate site passed safety screening and has sufficient capacity",
  "status": "PROPOSED",
  "approvedBy": null,
  "createdAt": "ISO_DATE"
}
```

Every AI-generated plan begins as `PROPOSED`.

## 9. Alert

```json
{
  "id": "ALERT-001",
  "title": "Flood warning",
  "message": "Follow instructions issued by local authorities.",
  "severity": "HIGH",
  "targetArea": {},
  "issuedBy": "USER-ID",
  "status": "DRAFT",
  "expiresAt": "ISO_DATE",
  "createdAt": "ISO_DATE"
}
```

## 10. BLE emergency packet

```json
{
  "protocolVersion": 1,
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

## 11. Mesh message persistence

Suggested local and backend fields:
- `messageId`
- `type`
- `packetJson` or normalized packet fields
- `senderDeviceId`
- `originDeviceId`
- `hopCount`
- `maxHops`
- `createdAt`
- `expiresAt`
- `gatewayStatus`
- `lastAttemptAt`
- `gatewaySyncedAt`

Use a unique index on `messageId` for backend deduplication.
