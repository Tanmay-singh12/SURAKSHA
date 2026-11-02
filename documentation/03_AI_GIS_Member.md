# RESQ-ZONE — Coding Agent 3: AI + GIS

## 1. Your ownership

You own geospatial data preparation, risk assessment, habitation prioritization, safe-site suitability and relocation recommendation logic.

Folder: `idea/ai-gis/`  
Branch: `feature/ai-gis`

Expose your functionality through a FastAPI service. The Node.js backend is the only application layer that should call this service.

Do not implement user authentication, citizen UI, government dashboard or BLE networking.

## 2. Product context

RESQ-ZONE turns hazard and geographic information into decision-support outputs:

**Detect → Assess → Prioritize → Relocate → Communicate**

Your module answers:
- What is the relative risk for each habitation?
- Which habitations should be reviewed first?
- Which candidate safe sites are unsuitable or have insufficient capacity?
- Which feasible relocation assignments can be proposed?

Important boundaries:
- This is not an exact disaster prediction system.
- Risk categories are prototype decision-support bands, not official standards.
- Do not fabricate data sources, model accuracy or feature importance.
- Authorities must validate hazard information and approve relocation decisions.
- A nearest site is not automatically a safe site.
- Clearly distinguish real data, derived values and simulated demo data.

## 3. Technology stack

- Python
- Pandas
- NumPy
- GeoPandas
- Shapely
- Rasterio / GDAL where required
- scikit-learn
- FastAPI
- Pydantic
- Joblib
- Uvicorn
- QGIS for visual inspection where useful

## 4. Suggested project structure

```text
idea/ai-gis/
  app/
    main.py
    schemas.py
    risk_engine.py
    priority_engine.py
    safe_site_engine.py
    relocation_engine.py
    data_loader.py
    feature_engineering.py
  data/
    raw/
    processed/
    demo/
  models/
  notebooks/
  scripts/
  tests/
  requirements.txt
  README.md
```

Keep processing logic modular and testable. Heavy raster processing should happen in preprocessing scripts, not on every API request.

## 5. Data acquisition — begin with one region

Build one consistent flood/heavy-rainfall demo scenario first. Potential data categories:

- Rainfall/weather warnings: IMD sources where accessible.
- Satellite and hazard layers: ISRO/Bhuvan/NDEM products where accessible.
- Elevation and terrain: DEM.
- Flood susceptibility or flood extent layers.
- Water bodies and drainage.
- Roads and transport accessibility.
- Population and habitation information.
- Hospitals, shelters and essential facilities.
- Verified or simulated citizen reports from the backend seed dataset.

For every dataset, document:
- Source and publisher.
- Download/access method.
- Licence or access conditions if known.
- Date/time of source data.
- Geographic coverage and CRS.
- Whether data is observed, derived or simulated.
- Any known limitations.

Do not assume that a source has a public API. If live access is unavailable, use a curated GeoJSON/CSV dataset and document that limitation. Never invent provenance.

## 6. Geospatial preprocessing

Implement reusable scripts to:
1. Load GeoJSON, shapefiles, CSVs and rasters.
2. Validate geometry and coordinate reference systems.
3. Reproject layers into suitable coordinate systems.
4. Repair or flag invalid geometries.
5. Spatially join habitations with hazard layers.
6. Calculate proximity to roads, water, hospitals and candidate safe sites.
7. Derive slope/elevation indicators if DEM data is available.
8. Aggregate hazard and exposure features at habitation level.
9. Export processed data in JSON/GeoJSON formats for the backend.

Important:
- GeoJSON coordinates use `[longitude, latitude]`.
- Use a projected CRS for metric distance calculations.
- Do not calculate metre/kilometre distances directly from raw latitude/longitude degrees.
- Preserve source metadata and timestamps.
- Represent missing data explicitly; do not silently treat it as a measured zero.

## 7. Habitation feature vector

Use normalized values between 0 and 1 where appropriate:

```json
{
  "hazardExposure": 0.0,
  "rainfallTrigger": 0.0,
  "populationExposure": 0.0,
  "vulnerability": 0.0,
  "evacuationDifficulty": 0.0,
  "infrastructureGap": 0.0
}
```

Document how each feature is derived. Explain normalization and missing-data handling.

Possible interpretations:
- `hazardExposure`: overlap/proximity to a relevant hazard layer.
- `rainfallTrigger`: normalized rainfall or warning indicator.
- `populationExposure`: normalized exposed population.
- `vulnerability`: available vulnerability indicators.
- `evacuationDifficulty`: road access, terrain and evacuation constraints.
- `infrastructureGap`: access to essential services and infrastructure.

Do not invent sensitive demographic information. Use defensible public/aggregated data or clearly labelled demo values.

## 8. Transparent MVP risk engine

For the initial MVP, implement a transparent weighted baseline:

```text
riskScore = 100 × (
  0.30 × hazardExposure +
  0.20 × rainfallTrigger +
  0.15 × populationExposure +
  0.15 × vulnerability +
  0.10 × evacuationDifficulty +
  0.10 × infrastructureGap
)
```

Clamp the final score to 0–100.

Prototype classifications:
- 0–19: `SAFE`
- 20–39: `LOW`
- 40–59: `MODERATE`
- 60–79: `HIGH`
- 80–100: `CRITICAL`

These weights and thresholds are illustrative prototype settings, not validated or official disaster standards. Document this prominently.

Add a rule layer for official or manually verified critical hazard information. A hard safety rule must not be overridden by a low weighted score.

### Explainability output

Return:
- Habitation ID.
- Risk score and category.
- Input features.
- Weighted contribution of each feature.
- Main contributing factors.
- Data freshness/source metadata.
- Model/rule version.
- Generation timestamp.

Example response:

```json
{
  "habitationId": "HAB-001",
  "riskScore": 82,
  "classification": "CRITICAL",
  "factors": [
    {
      "name": "hazardExposure",
      "value": 0.9,
      "contribution": 0.27
    },
    {
      "name": "rainfallTrigger",
      "value": 0.85,
      "contribution": 0.17
    }
  ],
  "modelVersion": "risk-v1",
  "generatedAt": "ISO_DATE"
}
```

For the weighted baseline, report the actual weighted contributions. Do not call them learned feature importance.

## 9. ML extension — only if defensible

Keep the architecture ready for a future Random Forest or Gradient Boosting model.

Do not train a model merely to claim that the product uses AI. Train only if you have documented, defensible labels and enough data.

If training:
- Document label provenance.
- Separate train and test data appropriately.
- Check leakage and class imbalance.
- Report precision, recall, F1 and confusion matrix.
- Avoid unsupported accuracy claims.
- Version the model and preprocessing pipeline.

The weighted baseline is acceptable for the hackathon MVP.

## 10. Habitation prioritization engine

Create a transparent priority score using:
- Risk score.
- Exposed population.
- Vulnerability.
- Evacuation difficulty.

Return a sorted list and an explanation for each priority. Make the weights configurable and document them. Do not imply that the ordering alone is an evacuation order.

## 11. Safe-site suitability engine

Evaluate candidate sites using:
- Hazard safety.
- Available capacity.
- Current occupancy.
- Water.
- Sanitation.
- Healthcare.
- Road accessibility.
- Logistics and other available services.

Calculate:

```text
remainingCapacity = totalCapacity - currentOccupancy
```

Exclude or strongly penalize sites that fall inside critical hazard areas or violate hard safety constraints. Return:
- Suitability score.
- Remaining capacity.
- Services available/missing.
- Exclusion reasons.
- Ranking and explanation.

Never recommend a site solely because it is geographically closest.

## 12. Relocation recommendation engine

Implement a simple, explainable MVP process:

1. Sort habitations by priority.
2. Filter out unsafe candidate sites.
3. Filter out sites with insufficient remaining capacity.
4. Rank feasible sites by safety, capacity, accessibility, services and distance.
5. Assign people without exceeding site capacity.
6. Return alternative feasible sites where possible.
7. Explain the reasoning and constraints.

Only split a habitation's population across sites if the capacity accounting is correct and the response clearly describes the split.

Every generated assignment is a proposal. The backend should store it with status `PROPOSED`; an authorized government user must approve it.

## 13. FastAPI endpoints

Implement and document:

### `GET /health`
Returns service status and version.

### `POST /predict-risk`
Input: habitation ID and normalized feature vector.  
Output: risk score, classification, factor contributions, model version and timestamp.

### `POST /analyze-region`
Input: region or demo dataset identifier.  
Output: habitation risk results, priority list and category summary.

### `POST /safe-sites/evaluate`
Input: candidate sites, hazard context, capacity and service data.  
Output: suitability scores, remaining capacity, exclusions and ranking.

### `POST /relocation/recommend`
Input: priority habitations and candidate sites.  
Output: proposed assignments, people count, remaining capacity, alternatives and explanation.

Use Pydantic request/response schemas. Validate ranges, missing fields and geometry.

## 14. Backend integration contract

The Node backend calls FastAPI and stores results in MongoDB. AI/GIS must not manage accounts, roles, alerts or citizen messages.

Coordinate exact schemas with the backend member in:

- `idea/shared/api-contract.md`
- `idea/shared/data-models.md`

Keep IDs and field names stable. Share the same habitation IDs and coordinates used in the backend seed data.

## 15. Demo dataset

Prepare:
- 5–10 habitations.
- One or more flood/heavy-rainfall hazard areas.
- 3–5 candidate safe sites.
- Population and infrastructure attributes.
- Rainfall/hazard values with source metadata.
- A few reports if useful.

Use actual open data where possible. Mark simulated values explicitly. Ensure the map and backend use exactly the same coordinates and IDs.

## 16. Tests and deliverables

Tests should cover:
- Risk score boundaries and classification.
- Missing/invalid features.
- Hard hazard exclusion rules.
- Safe-site capacity calculation.
- Insufficient-capacity handling.
- Relocation assignment capacity constraints.
- API validation and error cases.

Deliver:
- Data acquisition and provenance documentation.
- Raw/processed/demo data as appropriate.
- Cleaning and preprocessing scripts.
- Feature engineering.
- Risk, priority, safe-site and relocation engines.
- FastAPI service.
- Tests.
- `requirements.txt`.
- `.env.example` if needed.
- README with setup and endpoint examples.

## 17. Git workflow

Work only in `idea/ai-gis/` and agreed shared contracts.

```bash
git add idea/ai-gis/
git commit -m "feat(ai-gis): implement risk and relocation engines"
git push origin feature/ai-gis
```

## 18. First integration milestone

Use the shared demo habitation data to return a risk score and classification through FastAPI. The backend should call your endpoint, store the result and expose it to the government dashboard.

Get this working before adding advanced ML.
