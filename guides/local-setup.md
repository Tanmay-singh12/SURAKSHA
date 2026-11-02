# RESQ-ZONE — Local Development Setup

This guide describes the intended setup sequence. Exact commands depend on the package scripts and dependency versions committed by each module.

## 1. Prerequisites

- Git.
- Node.js LTS and npm.
- Python 3.10+ (confirm version against `requirements.txt`).
- MongoDB Atlas account or local MongoDB for development.
- Android Studio and Android SDK.
- Two physical Android phones for BLE testing.
- QGIS if inspecting geospatial layers.
- Access credentials for any external service actually integrated.

## 2. Clone repository

```bash
git clone <YOUR_GITHUB_REPOSITORY_URL>
cd RESQ-ZONE
```

## 3. Backend

```bash
cd idea/backend
npm install
```

Create `.env` from `.env.example`, fill in local values, then run the script defined in `package.json`, for example:

```bash
npm run dev
```

Verify the backend health route and database connection. Do not commit `.env`.

## 4. AI/GIS service

```bash
cd idea/ai-gis
python -m venv .venv
```

Activate the virtual environment, then:

```bash
pip install -r requirements.txt
uvicorn app.main:app --reload --port 8000
```

Verify `GET /health`. Confirm that geospatial system dependencies are installed if Rasterio/GDAL are used.

## 5. Government dashboard

```bash
cd idea/government-dashboard
npm install
```

Create its `.env` from `.env.example`, configure the backend API base URL, then run:

```bash
npm run dev
```

## 6. Citizen application

```bash
cd idea/citizen-app
npm install
```

Follow the React Native/Android instructions in the module README. Configure the backend base URL and build/run on an Android device or emulator.

For BLE, use physical Android devices. An emulator may not reproduce the required BLE behavior.

## 7. Offline communication

Follow `idea/offline-communication/README.md` for native Android build steps, permissions and device setup. Enable Bluetooth on both devices and test a minimal message before testing multi-hop relay.

## 8. Seed data

Run the backend seed script using the actual package command documented by the backend member. Use the same habitation IDs, site IDs and coordinates in AI/GIS and frontend demo configuration.

## 9. Local service map

| Service | Example local address |
|---|---|
| Backend | `http://localhost:5000` |
| AI/GIS | `http://localhost:8000` |
| Dashboard | Vite's printed local URL |
| MongoDB | Atlas URI or local connection |
| Citizen app | Android device/emulator |

For a physical phone, `localhost` refers to the phone itself. Use a reachable development machine address or configured tunnel for backend access, and do not expose development services publicly without appropriate safeguards.

## 10. Troubleshooting

- Check `.env` values and API base URLs.
- Confirm MongoDB network access and credentials.
- Confirm backend can reach AI/GIS service.
- Check CORS and Socket.IO origin settings.
- Confirm Android permissions and Bluetooth state.
- Confirm both BLE devices are using the same service/protocol version.
- Check that all modules use the same demo IDs and coordinate order.
