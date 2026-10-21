# AgriSphere Integration Report

## Result

All four uploaded member systems are connected in one local React + FastAPI application. No model was retrained or replaced, and production/default frontend behavior uses the real backend.

## Member status

- **Member 1:** tuned Random Forest, original feature engineering, top-five prediction, “Can I grow?”, what-if analysis, technical SHAP, and added farmer-friendly SHAP are connected.
- **Member 2:** MobileNetV2, 30 class names, supplied disease responses, upload validation, and independent image prediction are connected.
- **Member 3:** selected coordinates feed Open-Meteo; exact 12 risk features are prepared; all three risk models return their trained `Low`/`Medium`/`High` classes; required weather values are stored for Member 1.
- **Member 4:** the provided Google Maps satellite view, GPS control, manual coordinates, map selection, editable polygon boundary, centre, geometry-based area, and backend handoff are connected.

## Backend

- FastAPI loads all models once during startup.
- Farm, weather, crop prediction, can-grow, what-if, disease, and health endpoints are implemented.
- SQLite stores farms, boundaries, weather records, and prediction metadata without retaining uploaded image bytes.
- CORS, range validation, upload validation, safe farmer-facing errors, and environment examples are included.

## Frontend

- Home, Farm Location, Crop Recommendation, Disease Detection, Weather, Dashboard, and About routes are connected.
- Crop and disease workflows remain independent.
- Loading, validation, success, and error states are implemented.
- Demo behavior is disabled by default and requires `VITE_USE_DEMO_MODE=true`.

## Tests performed

| Test | Result |
|---|---|
| Python module compilation | Passed |
| TypeScript `tsc --noEmit` | Passed |
| Production frontend build | Passed |
| Rendered HTML/source contract tests | 2/2 passed |
| `GET /api/health` and all model startup checks | Passed |
| Farm save + automatic live weather | Passed |
| Member 3 three-model risk inference | Passed |
| Member 1 crop/top-five/SHAP inference | Passed |
| Member 1 can-grow and what-if | Passed |
| Member 2 API inference | Passed |
| Browser crop workflow | Passed |
| Browser disease upload and inference | Passed |
| Browser weather and dashboard | Passed |
| Browser console warnings/errors | None |
| Invalid coordinates/soil/what-if requests | Correctly rejected with 422 |
| Unsupported disease file type | Correctly rejected with 415 |

The browser crop test used Bengaluru coordinates with N=90, P=42, K=43, and pH=6.5. The real model returned jute at 81.2%, followed by rice. A +20% rainfall what-if changed the top result to rice at 90.0%.

The disease upload smoke test used the website hero image only to verify the full upload/inference path; it is not a disease-accuracy validation image. The model returned Maize Leaf Spot at approximately 57.0%.

## Limitations and security

- Device GPS permission was not accepted during automated testing; the GPS control and error handling are present, while manual coordinate selection was tested end-to-end.
- Live weather needs internet access.
- SHAP explains model behavior, not agricultural causation.
- The uploaded Member 4 source exposed a Google Maps API key. It is kept only in the ignored local `.env`, not in source or the ZIP; rotate and restrict it before continued use.
- The hosted React frontend needs a public HTTPS FastAPI/TensorFlow service URL. The complete real-model application works locally using the included setup instructions.
