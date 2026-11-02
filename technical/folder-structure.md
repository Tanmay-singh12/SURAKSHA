# RESQ-ZONE — Repository and Folder Structure

## 1. Proposed repository layout

```text
RESQ-ZONE/
│
├── README.md
│
├── idea/
│   ├── README.md
│   │
│   ├── citizen-app/
│   │   ├── android/
│   │   ├── ios/                         # Optional / future
│   │   ├── src/
│   │   │   ├── assets/
│   │   │   ├── components/
│   │   │   ├── navigation/
│   │   │   ├── screens/
│   │   │   ├── services/
│   │   │   ├── hooks/
│   │   │   ├── storage/
│   │   │   ├── ble/
│   │   │   ├── types/
│   │   │   └── utils/
│   │   ├── .env.example
│   │   └── README.md
│   │
│   ├── government-dashboard/
│   │   ├── public/
│   │   ├── src/
│   │   │   ├── assets/
│   │   │   ├── components/
│   │   │   ├── pages/
│   │   │   ├── layouts/
│   │   │   ├── routes/
│   │   │   ├── services/
│   │   │   ├── hooks/
│   │   │   ├── context/
│   │   │   ├── map/
│   │   │   ├── types/
│   │   │   └── utils/
│   │   ├── .env.example
│   │   └── README.md
│   │
│   ├── backend/
│   │   ├── src/
│   │   │   ├── config/
│   │   │   ├── models/
│   │   │   ├── routes/
│   │   │   ├── controllers/
│   │   │   ├── services/
│   │   │   ├── middleware/
│   │   │   ├── validators/
│   │   │   ├── sockets/
│   │   │   ├── integrations/
│   │   │   │   ├── ai/
│   │   │   │   ├── firebase/
│   │   │   │   └── mesh/
│   │   │   ├── utils/
│   │   │   ├── app.ts
│   │   │   └── server.ts
│   │   ├── scripts/
│   │   ├── tests/
│   │   ├── .env.example
│   │   └── README.md
│   │
│   ├── ai-gis/
│   │   ├── app/
│   │   │   ├── main.py
│   │   │   ├── schemas.py
│   │   │   ├── data_loader.py
│   │   │   ├── feature_engineering.py
│   │   │   ├── risk_engine.py
│   │   │   ├── priority_engine.py
│   │   │   ├── safe_site_engine.py
│   │   │   └── relocation_engine.py
│   │   ├── data/
│   │   │   ├── raw/
│   │   │   ├── processed/
│   │   │   └── demo/
│   │   ├── models/
│   │   ├── notebooks/
│   │   ├── scripts/
│   │   ├── tests/
│   │   ├── requirements.txt
│   │   └── README.md
│   │
│   ├── offline-communication/
│   │   ├── android/
│   │   ├── protocol/
│   │   │   ├── message-schema.md
│   │   │   └── gatt-service.md
│   │   ├── src/
│   │   │   ├── storage/
│   │   │   ├── relay/
│   │   │   ├── gateway/
│   │   │   └── adapter/
│   │   ├── tests/
│   │   └── README.md
│   │
│   └── shared/
│       ├── README.md
│       ├── api-contract.md
│       ├── data-models.md
│       └── status-enums.md
│
├── architecture/
│   ├── architecture.md
│   ├── data-flow.md
│   ├── deployment-architecture.md
│   └── diagrams/
│
├── technical/
│   ├── tech-stack.md
│   ├── folder-structure.md
│   ├── ai-gis-design.md
│   ├── ble-offline-protocol.md
│   └── integrations.md
│
├── contracts/
│   ├── api-contract.md
│   ├── data-models.md
│   └── status-enums.md
│
└── guides/
    ├── local-setup.md
    ├── git-workflow.md
    ├── testing.md
    └── demo-runbook.md
```

## 2. Folder ownership

| Folder | Owner | Responsibility |
|---|---|---|
| `idea/citizen-app/` | Frontend member | Citizen mobile application |
| `idea/government-dashboard/` | Frontend member | Authority web dashboard |
| `idea/backend/` | Backend member | APIs, database and integrations |
| `idea/ai-gis/` | AI/GIS member | Data processing and decision engines |
| `idea/offline-communication/` | BLE member | Android BLE and offline queue |
| `idea/shared/` | All members by agreement | Shared API and data contracts |
| `architecture/` | Team / integration owner | System-level diagrams and architecture |
| `technical/` | Team / integration owner | Technology and implementation documentation |
| `contracts/` | Team / integration owner | Canonical shared schemas |
| `guides/` | Team / integration owner | Setup, testing and demo instructions |

## 3. Folder structure rules

- Treat this as a proposed layout; adapt to actual framework-generated structures.
- Do not create empty folders only to make the tree look complete.
- Each module should have a README with setup, environment variables, commands and current status.
- Keep raw data separate from processed data.
- Do not commit large private datasets, credentials or generated build artifacts.
- Keep shared API contracts outside individual module internals.
- If a shared field changes, update the contract and notify all members before merging.

## 4. Root-level files recommended

- `README.md` — project overview and documentation index.
- `.gitignore` — ignore secrets, dependencies, build outputs and local data.
- `LICENSE` — only after the team chooses a licence.
- `CONTRIBUTING.md` — contribution and PR rules if useful.
- `SECURITY.md` — responsible vulnerability reporting if the project becomes public.
