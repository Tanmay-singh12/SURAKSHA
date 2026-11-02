# RESQ-ZONE — Coding Agent 2: Backend

## 1. Your ownership

You own the central backend and integration layer.

Folder: `idea/backend/`  
Branch: `feature/backend`

The backend is the system coordinator and source of truth for users, reports, SOS messages, incidents, habitations, hazard results, safe sites, relocation plans and alerts.

Do not implement frontend UI, AI/GIS algorithms or Android BLE internals. Coordinate their interfaces and own the APIs that connect them.

## 2. Product context and boundaries

RESQ-ZONE supports the workflow:

**Detect → Assess → Prioritize → Relocate → Communicate**

It combines hazard/risk decision support with emergency reporting, safe-site capacity assessment, proposed relocation and offline message synchronization.

Important:
- This is an MVP decision-support system, not an autonomous evacuation system.
- Authorities retain control over verification, alert publication and relocation approval.
- AI scores are not exact disaster predictions.
- BLE-originated reports remain unverified until reviewed.
- Clearly label demo/simulated data.
- Do not claim a notification was delivered merely because an alert was published or sent to an FCM provider.

## 3. Technology stack

- Node.js
- Express
- TypeScript
- MongoDB Atlas
- Mongoose
- JWT authentication
- bcrypt password hashing
- Zod or Joi validation
- Socket.IO
- Firebase Admin SDK / FCM where feasible
- Multer for optional media uploads
- Rate limiting and security middleware

## 4. Suggested project structure

```text
idea/backend/
  src/
    config/
    models/
    routes/
    controllers/
    services/
    middleware/
    validators/
    sockets/
    integrations/
      ai/
      firebase/
      mesh/
    utils/
    app.ts
    server.ts
  scripts/
    seed.ts
  tests/
  .env.example
  package.json
  README.md
```

Keep controllers thin. Put business logic in services and validate all external input.

## 5. Authentication and authorization

Implement:
- Citizen registration and login.
- Government user login.
- JWT access authentication.
- bcrypt password hashing.
- `/api/auth/me`.
- Role-based middleware.

Rules:
- Public registration can create only `CITIZEN` accounts.
- Government roles must be provisioned through a controlled seed/admin process.
- Never accept a role supplied by a public registration request.
- Never store plaintext passwords.
- Protect all authority-only routes.
- Record sensitive actions in audit logs.

Roles:
- `CITIZEN`
- `FIELD_OFFICER`
- `DISASTER_AUTHORITY`
- `ADMIN`

## 6. MongoDB models / collections

Create appropriate Mongoose schemas, validation and indexes.

### User
- `name`
- `email`
- `phone`
- `passwordHash`
- `role`
- `status`
- `createdAt`

### CitizenReport
- `reporterId`
- `hazardType`
- `description`
- `severity`
- `location` as GeoJSON Point
- `media`
- `status`
- `verifiedBy`
- `verifiedAt`
- `createdAt`

### Incident
- `reportId`
- `hazardType`
- `description`
- `severity`
- `location`
- `status`
- `source`
- `verifiedBy`
- `createdAt`

### SOS
- `reporterId` or `originDeviceId`
- `location`
- `peopleCount`
- `helpRequired`
- `description`
- `source` (`APP` or `BLE`)
- `status`
- `receivedAt`
- `synchronizedAt`

### Habitation
- `name`
- `location` as GeoJSON Point
- `population`
- `riskScore`
- `classification`
- `priority`
- `riskFactors`
- `updatedAt`

### HazardZone
- `hazardType`
- `geometry` as Polygon or MultiPolygon
- `riskScore`
- `classification`
- `factors`
- `modelVersion`
- `sourceTimestamp`
- `validUntil`

### SafeSite
- `name`
- `location`
- `totalCapacity`
- `currentOccupancy`
- `services` (water, sanitation, medical, food, power/logistics as applicable)
- `safetyScore`
- `suitabilityScore`
- `status`

### RelocationPlan
- `habitationId`
- `safeSiteId`
- `peopleToRelocate`
- `recommendedDistance`
- `explanation`
- `status` (`PROPOSED`, `APPROVED`, `REJECTED`)
- `approvedBy`
- `createdAt`

### Alert
- `title`
- `message`
- `severity`
- `targetArea` geometry or habitation IDs
- `issuedBy`
- `status` (`DRAFT`, `PUBLISHED`, `EXPIRED`)
- `expiresAt`
- `createdAt`

### MeshMessage
- `messageId`
- `type`
- `senderDeviceId`
- `originDeviceId`
- `location`
- `payload`
- `hopCount`
- `maxHops`
- `createdAt`
- `expiresAt`
- `gatewayStatus`

### AuditLog
- `actorId`
- `action`
- `resourceType`
- `resourceId`
- `metadata`
- `createdAt`

Create `2dsphere` indexes for GeoJSON location/geometry fields and ordinary indexes for frequently queried status, timestamps and IDs. Ensure `messageId` is unique for mesh deduplication.

## 7. API routes

Use the standard response format below.

Success:

```json
{
  "success": true,
  "data": {},
  "message": "Operation successful"
}
```

Error:

```json
{
  "success": false,
  "message": "Something went wrong",
  "error": { "code": "VALIDATION_ERROR" }
}
```

### Auth
- `POST /api/auth/register`
- `POST /api/auth/login`
- `GET /api/auth/me`

### Reports
- `POST /api/reports`
- `GET /api/reports`
- `GET /api/reports/:id`
- `PATCH /api/reports/:id/status`

Only authorized officers can verify/reject reports. Verification should create or update the corresponding incident, add an audit log and emit a real-time event.

### SOS
- `POST /api/sos`
- `GET /api/sos`
- `PATCH /api/sos/:id/status`

Persist the SOS before attempting external integrations. Add rate limiting and duplicate protection.

### Hazards and habitations
- `GET /api/hazards`
- `GET /api/hazards/nearby?lat=&lng=`
- `GET /api/habitations`
- `GET /api/habitations/priority`

### Safe sites
- `GET /api/safe-sites`
- `GET /api/safe-sites/nearby?lat=&lng=`
- `POST /api/safe-sites`
- `PATCH /api/safe-sites/:id`

Restrict create/update operations to authorized roles.

### Relocation
- `GET /api/relocation/recommendations`
- `POST /api/relocation/generate`
- `PATCH /api/relocation/:id/approve`
- `PATCH /api/relocation/:id/reject`

Every generated plan begins as `PROPOSED`. Only authorized authorities may approve or reject.

### Alerts
- `POST /api/alerts`
- `GET /api/alerts`
- `GET /api/alerts/:id`
- `POST /api/alerts/:id/publish`
- `POST /api/alerts/:id/acknowledge`

Only `DISASTER_AUTHORITY` and `ADMIN` can publish alerts. Keep acknowledgement separate from delivery status.

### BLE gateway
- `POST /api/mesh/sync`
- `GET /api/mesh/status`

The sync endpoint should validate protocol version and packet structure, enforce expiry/hop limits, deduplicate by `messageId`, persist accepted packets and return accepted/duplicate/rejected IDs. Convert relevant SOS packets into SOS records with source `BLE`. Treat them as unverified citizen-originated messages.

## 8. AI/GIS integration

The AI/GIS member owns a separate FastAPI service. The Node backend calls it; frontend clients must not call it directly.

Initial request to AI service:

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

Expected response:

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

Coordinate the final schema in `idea/shared/api-contract.md`.

Store successful analysis results in MongoDB. If the AI service is unavailable, return the last known result with its timestamp and clearly indicate that it may be stale. Never silently present stale results as current.

## 9. Government communication

Implement an authority-controlled alert lifecycle:

1. Create draft.
2. Validate message, target and expiry.
3. Check publishing role.
4. Publish and record audit log.
5. Attempt FCM delivery.
6. Emit Socket.IO event.
7. Track acknowledgements separately where supported.

A published alert is not proof that every citizen received it. Save the alert even if FCM fails; record notification errors separately.

## 10. Socket.IO events

Implement authenticated sockets and role-restricted event delivery.

Suggested events:
- `report.created`
- `report.verified`
- `sos.created`
- `incident.updated`
- `risk.updated`
- `safeSite.updated`
- `relocation.updated`
- `alert.published`
- `mesh.message.received`

Do not broadcast sensitive citizen information to every connected client.

## 11. Seed data

Create a repeatable seed script with:
- One admin.
- One disaster authority.
- One field officer.
- Several citizen accounts.
- 5–10 demo habitations.
- Demo hazard polygons.
- 3–5 safe sites.
- Sample reports and alerts.

Clearly mark simulated records as `DEMO` / `SIMULATED`. Share stable IDs and coordinates with the frontend and AI/GIS members so all modules use the same scenario.

## 12. Security and reliability

- Validate request bodies, query parameters and route parameters.
- Centralized error handling.
- JWT authentication and RBAC.
- Rate-limit login and SOS/report endpoints.
- Configure CORS from environment variables.
- Keep secrets in environment variables.
- Add audit logs for verification, publication and relocation approval.
- Deduplicate BLE packets.
- Validate uploaded files and restrict size/type.
- Use safe defaults for missing optional data.
- Do not expose stack traces or secrets in production responses.

## 13. MVP scope

Prioritize:
1. Auth and role protection.
2. Report and SOS persistence.
3. Government verification workflow.
4. Habitation/hazard/safe-site APIs.
5. AI service integration.
6. Relocation proposal and approval.
7. Alert publishing and acknowledgements.
8. BLE gateway sync.
9. Seed data and API tests.

Avoid premature microservices, complex event infrastructure or production-scale multi-region architecture.

## 14. Git and deliverables

Work in `idea/backend/` on `feature/backend`.

```bash
git add idea/backend/
git commit -m "feat(backend): implement RESQ-ZONE API and integrations"
git push origin feature/backend
```

Deliver:
- Express + TypeScript backend.
- MongoDB schemas and indexes.
- Auth/RBAC middleware.
- API documentation and shared contract updates.
- Seed script.
- AI, FCM, Socket.IO and BLE gateway integrations.
- `.env.example`.
- README with setup and run instructions.
- Postman collection or curl examples.
- Tests for critical routes and authorization.

## 15. First end-to-end milestone

**Citizen submits a hazard report → backend stores it → government dashboard receives it → officer verifies it → verified incident is visible.**

Coordinate with the frontend member early. This is the first integration checkpoint for the entire team.
