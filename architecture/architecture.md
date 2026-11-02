# RESQ-ZONE — System Architecture

## 1. Architecture goals

The architecture is designed around five goals:

1. **Modularity:** each major capability has a clear owner and interface.
2. **Central coordination:** the Node.js backend is the operational source of truth.
3. **Explainability:** risk and relocation outputs expose factors, timestamps and versions.
4. **Resilience:** the citizen app can queue emergency messages locally and use localized BLE relaying.
5. **Human authority:** verification, alert publication and relocation approval remain controlled by authorized personnel.

The architecture is an MVP design. It should not be described as a certified or production-ready emergency-management system.

## 2. High-level architecture

```mermaid
flowchart TB
  subgraph UserLayer["User-facing applications"]
    CIT[Citizen App<br/>React Native + TypeScript]
    GOV[Government Dashboard<br/>React + Vite + TypeScript]
  end

  subgraph BackendLayer["Central backend"]
    API[Express REST API]
    AUTH[Authentication + RBAC]
    SERVICES[Application Services]
    SOCKET[Socket.IO]
    NOTIFY[FCM Integration]
    MESHAPI[Mesh Gateway API]
  end

  DB[(MongoDB Atlas)]

  subgraph AI["AI and geospatial intelligence"]
    FAST[FastAPI Service]
    PRE[Data ingestion and preprocessing]
    GIS[GIS feature engineering]
    RISK[Risk and priority engines]
    SAFE[Safe-site and relocation engines]
    DATA[(Raw / Processed / Demo data)]
    DATA --> PRE --> GIS
    GIS --> RISK
    GIS --> SAFE
    RISK --> FAST
    SAFE --> FAST
  end

  subgraph Offline["Offline communication"]
    RNADAPTER[React Native BLE Adapter]
    NATIVE[Kotlin Android BLE]
    LOCAL[(SQLite message queue)]
    GATEWAY[Internet-connected gateway]
    RNADAPTER --> NATIVE
    NATIVE <--> LOCAL
    LOCAL --> GATEWAY
  end

  CIT <-->|HTTPS REST| API
  GOV <-->|HTTPS REST| API
  CIT <-->|Authenticated events| SOCKET
  GOV <-->|Authenticated events| SOCKET
  API --> AUTH
  API --> SERVICES
  SERVICES <--> DB
  SERVICES <-->|Internal HTTP| FAST
  API --> SOCKET
  API --> NOTIFY
  GATEWAY -->|HTTPS /api/mesh/sync| MESHAPI
  MESHAPI --> SERVICES
```

## 3. Component responsibilities

### 3.1 Citizen app

Responsibilities:
- Citizen authentication and profile.
- Location and map presentation.
- Hazard report and SOS forms.
- Display risk information and safe sites.
- Display official alerts.
- Local status and pending-message interface.
- Invoke the BLE adapter for offline message operations.

It must not:
- Connect directly to MongoDB.
- Call the AI service directly.
- decide that a citizen must evacuate.
- Claim a BLE message has reached authorities without backend acknowledgement.

### 3.2 Government dashboard

Responsibilities:
- Authority authentication and role-based UI.
- Incident/report review.
- Operational map and lists.
- Habitation priority and risk visualization.
- Safe-site and capacity overview.
- Relocation proposal review.
- Alert creation and publication.
- Real-time updates from authorized backend events.

It must not:
- Bypass backend authorization.
- Publish alerts using client-only logic.
- Treat citizen-submitted data as verified automatically.

### 3.3 Node.js backend

The backend coordinates all application workflows.

Responsibilities:
- Authentication and RBAC.
- Input validation and API responses.
- Persistence and data access.
- Citizen reports, incidents and SOS lifecycle.
- Hazard, habitation and safe-site APIs.
- Calls to FastAPI AI/GIS service.
- Relocation proposal storage and approval workflow.
- Alert lifecycle and audit logs.
- Socket.IO events and push notification integration.
- BLE gateway ingestion and deduplication.

### 3.4 MongoDB Atlas

Stores operational application data:
- Users.
- Citizen reports and incidents.
- SOS messages.
- Habitations and hazard zones.
- Safe sites.
- Relocation plans.
- Alerts and acknowledgements.
- Mesh messages and audit logs.

Geospatial fields should use GeoJSON and appropriate `2dsphere` indexes.

### 3.5 FastAPI AI/GIS service

Responsibilities:
- Preprocessed geospatial features.
- Risk scoring and explainability.
- Habitation priority calculations.
- Safe-site suitability and capacity checks.
- Relocation recommendations.

The service does not manage users, passwords, authority roles, alerts or citizen communications.

### 3.6 Android BLE module

Responsibilities:
- Device capability and permission handling.
- BLE discovery and GATT transfer.
- Packet validation.
- Persistent local message storage.
- Deduplication, TTL and hop-limit rules.
- Best-effort store-and-forward.
- Gateway synchronization.

## 4. Trust boundaries

```mermaid
flowchart LR
  subgraph Untrusted["Untrusted / user-originated inputs"]
    USER[Citizen input]
    BLEIN[BLE received packet]
    UPLOAD[Uploaded media]
  end

  subgraph AppBoundary["Backend validation boundary"]
    VALID[Schema and size validation]
    AUTHZ[Authentication and authorization]
    DEDUPE[Deduplication and expiry checks]
  end

  subgraph Trusted["Persisted application state"]
    DB[(Database)]
    AUDIT[Audit log]
  end

  USER --> VALID
  BLEIN --> VALID
  UPLOAD --> VALID
  VALID --> AUTHZ
  AUTHZ --> DEDUPE
  DEDUPE --> DB
  DB --> AUDIT
```

Citizen and BLE data must be treated as unverified input. Backend validation and authorized review are required before it becomes verified operational information.

## 5. Main workflows

### 5.1 Hazard report verification

```mermaid
sequenceDiagram
  participant Citizen
  participant App as Citizen App
  participant API as Backend
  participant DB as MongoDB
  participant Officer as Government Dashboard

  Citizen->>App: Submit hazard report
  App->>API: POST /api/reports
  API->>API: Validate token and payload
  API->>DB: Save report as PENDING
  API-->>App: Report created
  API-->>Officer: report.created event
  Officer->>API: PATCH report status
  API->>API: Check officer role
  API->>DB: Update report / incident and audit log
  API-->>Officer: report.verified event
  API-->>App: Updated status (where subscribed/queried)
```

### 5.2 Risk assessment

```mermaid
sequenceDiagram
  participant Dashboard
  participant API as Node Backend
  participant AI as FastAPI AI/GIS
  participant DB as MongoDB

  Dashboard->>API: Request habitation priority
  API->>DB: Load habitation/features
  API->>AI: POST /analyze-region or /predict-risk
  AI->>AI: Validate features and calculate outputs
  AI-->>API: Scores, categories, factors, version
  API->>DB: Store result and timestamp
  API-->>Dashboard: Risk and priority data
```

### 5.3 Relocation proposal

```mermaid
sequenceDiagram
  participant Authority
  participant Dashboard
  participant API as Backend
  participant AI as AI/GIS
  participant DB as MongoDB

  Authority->>Dashboard: Request relocation analysis
  Dashboard->>API: POST /api/relocation/generate
  API->>AI: POST /relocation/recommend
  AI-->>API: Proposed assignments and explanations
  API->>DB: Save plans as PROPOSED
  API-->>Dashboard: Return proposed plans
  Authority->>Dashboard: Review and approve/reject
  Dashboard->>API: PATCH approval/rejection route
  API->>API: Check authority role
  API->>DB: Update plan and audit log
```

### 5.4 Offline SOS and gateway synchronization

```mermaid
sequenceDiagram
  participant A as Phone A
  participant B as Phone B
  participant C as Phone C / Gateway
  participant API as Backend
  participant DB as MongoDB
  participant GOV as Dashboard

  A->>A: Create SOS and persist locally
  A->>B: BLE GATT transfer
  B->>B: Validate, deduplicate and persist
  B-->>A: Transfer acknowledgement
  B->>C: Optional BLE relay
  C->>C: Persist packet
  C->>API: POST /api/mesh/sync when online
  API->>API: Validate, expire-check, deduplicate
  API->>DB: Store mesh message and SOS
  API-->>C: Accepted / duplicate / rejected IDs
  API-->>GOV: mesh.message.received / sos.created
  GOV->>API: Review SOS
```

A BLE transfer acknowledgement means the receiving device accepted the packet. It does not mean that the backend or emergency authority has received it.

## 6. Data ownership and source of truth

| Data | Primary owner | Persistence |
|---|---|---|
| User accounts and roles | Backend | MongoDB |
| Citizen reports | Backend | MongoDB |
| Verified incidents | Backend | MongoDB |
| SOS records | Backend | MongoDB |
| Local unsynchronized SOS packet | BLE module | Device SQLite |
| Habitation base data | AI/GIS data preparation + backend seed/import | Files and MongoDB |
| Risk calculation | AI/GIS | Result stored by backend |
| Safe-site suitability | AI/GIS | Result/plan stored by backend |
| Relocation approval | Backend authority workflow | MongoDB |
| Official alerts | Backend | MongoDB |
| BLE protocol state | BLE module | Device SQLite |
| Mesh synchronization result | Backend | MongoDB |

## 7. Key design decisions

- Backend is the only client-facing application API.
- AI/GIS is an internal service called by the backend.
- GeoJSON uses `[longitude, latitude]`.
- Risk scoring is explainable and versioned.
- Relocation outputs are proposals, not automatic actions.
- BLE uses localized store-and-forward, not a claim of unlimited connectivity.
- Every important state transition should have a timestamp.
- Shared IDs and schemas must be agreed before parallel development.
- Mock services must be visibly identified as mocks.

## 8. Scalability path

For the MVP, a modular monolith backend plus a separate AI/GIS service is sufficient. Avoid premature microservices.

Future scale options, only when justified:
- Background job queue for large GIS processing.
- Object storage for media and large geospatial files.
- Dedicated geospatial database or optimized spatial storage.
- Event broker for high-volume event delivery.
- Caching for frequently accessed map layers.
- Regional data partitioning.
- Observability, metrics and alerting.

These are future options, not assumed current implementations.
