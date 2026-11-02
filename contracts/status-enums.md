# RESQ-ZONE — Shared Roles, Statuses and Enums

These are proposed shared MVP values. If implementation needs a change, agree across modules and update this document before integration.

## 1. Roles

- `CITIZEN`
- `FIELD_OFFICER`
- `DISASTER_AUTHORITY`
- `ADMIN`

Public registration may create only `CITIZEN`.

## 2. Risk classification

| Score range | Value |
|---:|---|
| 0–19 | `SAFE` |
| 20–39 | `LOW` |
| 40–59 | `MODERATE` |
| 60–79 | `HIGH` |
| 80–100 | `CRITICAL` |

Prototype-only thresholds. Not official standards.

## 3. Report / incident status

- `PENDING`
- `VERIFIED`
- `REJECTED`
- `RESOLVED`

## 4. Alert status

- `DRAFT`
- `PUBLISHED`
- `EXPIRED`

Acknowledgement and delivery should be tracked separately from publication status.

## 5. Relocation status

- `PROPOSED`
- `APPROVED`
- `REJECTED`

Only authorized authority roles may approve or reject.

## 6. BLE local message status

- `QUEUED`
- `SENT_TO_PEER`
- `RECEIVED`
- `RELAY_PENDING`
- `SYNC_PENDING`
- `SYNCED`
- `EXPIRED`
- `FAILED`

## 7. Message type and priority

Initial message type:
- `SOS`

Possible priority values:
- `LOW`
- `MEDIUM`
- `HIGH`
- `CRITICAL`

Keep enum values consistent across mobile, backend and dashboard.

## 8. Data provenance marker

Recommended marker:
- `OBSERVED`
- `DERIVED`
- `SIMULATED`
- `UNKNOWN`

Use these markers or an equivalent documented field to distinguish actual source data from demo values.
