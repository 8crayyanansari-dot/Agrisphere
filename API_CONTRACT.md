# AgriSphere API contract

Interactive OpenAPI documentation: `http://127.0.0.1:8000/docs`.

## Confirm farm and retrieve weather

`POST /api/farm/location`

```json
{
  "latitude": 19.076,
  "longitude": 72.8777,
  "boundary": [{ "latitude": 19.075, "longitude": 72.877 }],
  "area_square_meters": 2500,
  "area_hectares": 0.25
}
```

Boundary is optional; when supplied it must contain at least three points. The response contains `farm.id` and the complete Member 3 weather response.

## Weather

`GET /api/weather?latitude=19.076&longitude=72.8777&farm_id=1`

Returns source, selected coordinates, current measurements, seven-day forecast, exact Member 3 risk features, three model risks and Member 1 environmental features.

## Crop recommendation

`POST /api/crop/predict`

```json
{
  "farm_id": 1,
  "soil": { "N": 90, "P": 42, "K": 43, "ph": 6.5 }
}
```

The backend loads weather for `farm_id`, combines it with soil data, runs Member 1's original preprocessing/model, and returns the top five crops plus SHAP explanation.

`POST /api/crop/can-grow` adds `crop_name` to the same body.

`POST /api/crop/what-if` adds, for example:

```json
{ "changes": { "rainfall": { "percent": 20 } } }
```

## Disease prediction

`POST /api/disease/predict`

Content type: `multipart/form-data`. Field: `image`. Accepted formats: JPG, PNG and WebP. Maximum size: 10 MB.

The response contains the actual MobileNetV2 class, crop, health status, disease, confidence, top three model outputs, and only the advice present in Member 2's response JSON.
