# AgriSphere — Integrated Frontend and Backend

AgriSphere connects the uploaded work of all four members without changing their responsibilities or retraining their models.

## Architecture

### Crop recommendation

```text
Member 4 map/GPS/boundary
  → latitude + longitude
  → FastAPI
  → Member 3 Open-Meteo weather + risk models
  → weather/environmental features
  + farmer-entered N, P, K and pH
  → Member 1 tuned Random Forest
  → top five crops + confidence + SHAP
```

### Disease detection

```text
Farmer crop image
  → FastAPI
  → Member 2 MobileNetV2
  → class_names.json
  → agrisphere_disease_responses.json
  → disease/healthy result + supplied advice
```

Disease detection is independent of farm location, weather, soil and crop recommendation.

## Included uploaded artifacts

- Member 1: `backend/vendor/member1/models/crop_recommendation_model.pkl`, its original `src` preprocessing/prediction package, label encoder, feature list and component models.
- Member 2: `backend/vendor/member2/agrisphere_mobilenetv2.keras`, `class_names.json`, and `agrisphere_disease_responses.json`.
- Member 3: `drought_model.pkl`, `rainfall_model.pkl`, and `extreme_cold_model.pkl`.
- Member 4: GPS, manual farm selection, polygon boundary, farm centre and area behavior integrated into the existing React map.

The models are loaded once during FastAPI startup and reused for requests.

## Backend setup (macOS/Linux)

Python 3.12 is recommended.

```bash
cd AgriSphere_Integrated
python3.12 -m venv .venv
source .venv/bin/activate
pip install -r backend/requirements.txt
uvicorn backend.main:app --reload --host 127.0.0.1 --port 8000
```

The API documentation is available at `http://127.0.0.1:8000/docs`.

## Frontend setup

In a second terminal:

```bash
cd AgriSphere_Integrated
cp .env.example .env
npm install
npm run dev
```

Open the local URL printed by the frontend, normally `http://localhost:3000`.

## Environment variables

Frontend `.env`:

```env
VITE_API_BASE_URL=http://127.0.0.1:8000
VITE_USE_DEMO_MODE=false
VITE_GOOGLE_MAPS_API_KEY=replace_with_your_restricted_browser_key
```

Backend variables (optional):

```env
AGRISPHERE_CORS_ORIGINS=http://localhost:3000,http://127.0.0.1:3000
AGRISPHERE_DB_PATH=backend/agrisphere.db
```

Production/default behavior uses the real API. Demo mode runs only when explicitly set to `true`.

## Maps and Google Maps security

The integrated frontend uses Member 4's Google Maps satellite implementation. It supports map selection, GPS recentering, polygon drawing, editable polygon vertices, farm-centre calculation and area calculation through the Google Maps geometry library.

Put a browser-restricted Google Maps JavaScript API key in `VITE_GOOGLE_MAPS_API_KEY`. The key found in the uploaded Member 4 source is used only in the local ignored `.env`; it is not committed or included in the downloadable ZIP. Because that key was exposed in source, rotate it and restrict the replacement to the Maps JavaScript API and your permitted website origins.

## Weather setup

Member 3's existing Open-Meteo approach is used and does not require an API key. FastAPI calls Open-Meteo from the selected farm coordinates. The service prepares:

- current temperature and humidity;
- current and accumulated rainfall;
- 2 m, 10 m and 50 m wind;
- surface pressure, daily solar radiation and elevation;
- seven-day forecast;
- the exact 12 features expected by Member 3's three risk models.

For Member 1, `rainfall` is the observed 30-day accumulated rainfall. The 2 m wind follows Member 3's power-law conversion from 10 m wind. The 50 m value uses Open-Meteo CMA when available and clearly identifies the fallback estimate when necessary.

## Member 1 inputs

The saved feature list contains 28 model features. Four raw soil values are entered by the farmer:

- `N` — 0–200 kg/ha
- `P` — 0–150 kg/ha
- `K` — 0–250 kg/ha
- `ph` — 0–14

Ten raw weather/location values come from Member 3 and the selected coordinates. Member 1's original `engineer_features` function creates the remaining 14 features. Missing soil values are not guessed.

## API endpoints

- `GET /api/health`
- `POST /api/farm/location` — saves location/boundary/area and automatically retrieves weather
- `GET /api/weather?latitude=...&longitude=...&farm_id=...`
- `POST /api/crop/predict`
- `POST /api/crop/can-grow`
- `POST /api/crop/what-if`
- `POST /api/disease/predict` — multipart field name: `image`

Crop requests use a saved `farm_id` plus farmer soil values. The frontend does not copy weather values into crop requests; FastAPI retrieves the stored Member 3 weather record and combines it with soil data.

## Persistence

The backend uses Python's built-in SQLite support and creates `backend/agrisphere.db`. It stores confirmed farms, weather records, crop predictions and disease prediction metadata. Uploaded image bytes are not stored.

## Error handling

The API validates coordinates, boundaries, soil ranges, crop names, image MIME type, image size and image decoding. Farmer-facing responses do not expose stack traces. A weather failure can be retried without inventing fallback weather values.

## Known limitations

- Open-Meteo requires internet access.
- The Member 1 dataset contains only 20 repeated city locations and lacks external farm outcome validation.
- Member 2 reported 89.75% held-out accuracy; a prediction is not a confirmed diagnosis.
- Member 3 risk models return their trained `Low`, `Medium` or `High` classes; the uploaded helper incorrectly compared these string outputs to integer `1`, so the integration uses the models' actual classes.
- TensorFlow and SHAP make initial backend startup slower than subsequent requests.
- The hosted React site needs a publicly deployed HTTPS FastAPI URL configured at build time. The complete real-model system works locally; deploying the Python/TensorFlow backend requires a separate model-capable hosting service.

## Responsible use

AgriSphere provides model-based decision support. SHAP explains model behavior, not agricultural causation. Confirm important decisions with soil testing and qualified local agricultural advice.
