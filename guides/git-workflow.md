# RESQ-ZONE — Git and Team Collaboration Workflow

## 1. Branch ownership

| Member | Branch | Owned folder |
|---|---|---|
| Frontend | `feature/frontend` | `idea/citizen-app/`, `idea/government-dashboard/` |
| Backend | `feature/backend` | `idea/backend/` |
| AI/GIS | `feature/ai-gis` | `idea/ai-gis/` |
| BLE/Offline | `feature/ble-offline` | `idea/offline-communication/` |

Shared contract changes require communication and review.

## 2. Start work

```bash
git checkout main
git pull origin main
git checkout -b feature/your-module
```

If the branch already exists:

```bash
git checkout feature/your-module
git pull origin feature/your-module
```

## 3. Commit only owned files

Example:

```bash
git add idea/backend/
git commit -m "feat(backend): implement report APIs"
git push origin feature/backend
```

Use descriptive commit messages.

## 4. Pull request process

1. Push the feature branch.
2. Open a PR into `main`.
3. Describe completed work and tests.
4. Identify API/schema changes.
5. Include screenshots or sample requests where relevant.
6. Request review from the integration owner and affected module members.
7. Resolve conflicts before merging.
8. Delete merged branches only when no longer needed.

## 5. Shared contract rule

Before changing a shared field:
1. Explain the change in the team channel.
2. Identify affected modules.
3. Update `contracts/` and `idea/shared/`.
4. Update implementation and tests.
5. Merge in a coordinated order.

## 6. Avoid

- Committing `.env` or credentials.
- Force-pushing shared branches without agreement.
- Editing another member's module directly.
- Replacing the whole `idea/` folder with an outdated local copy.
- Merging code that uses different IDs or status values.
- Claiming an integration works when only a mock is connected.
