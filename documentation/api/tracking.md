# Tracking API v1

The [versioned generated OpenAPI file](v1/openapi.json) is the contract reference for the two intentionally public ingestion operations. The app serves Swagger at `/api/docs` and its generated specification at `/api/docs/openapi`. Documentation does not enable ingestion; deployments must explicitly configure encrypted storage and credentials.

Use a bearer key scoped to an organisation, provider and registered devices. No human dashboard role is accepted as an API credential. Expired/revoked keys and missing entitlement fail. Never put a token or tenant ID in a URL, log or documentation screenshot.

Submit up to 100 observations per JSON batch, within 256 KiB; binary Codec 8/8E frames are limited to 64 KiB and use the registered IMEI header. IMEI and CRC do not authenticate a request. Latitude/longitude appear together; a registered device or vehicle and position or odometer evidence are required. Observation times are UTC/offset timestamps, no earlier than 2000 and no more than five minutes ahead of server time.

The service permits 60 requests per key per minute. HTTP 429 includes Retry-After. A stable event ID makes an identical retry safe; conflicting content returns 409. Delayed/out-of-order observations remain historical evidence. HTTP 202 means durably accepted, not invoiced or charged. See generated request/response examples and error schemas for exact field and status definitions. Other modules are not public API-enabled.

## Sending a batch

Save the generated request example to `telemetry.json`. With the service URL and key supplied through your secure environment, send:

```sh
curl --request POST "$TRACKING_BASE_URL/api/v1/tracking/events" \
  --header "Authorization: Bearer $TRACKING_KEY" \
  --header 'Content-Type: application/json' \
  --data-binary @telemetry.json
```

For a genuine device frame stored in `frame.bin`, use the gateway operation and its registered device identifier:

```sh
curl --request POST "$TRACKING_BASE_URL/api/v1/tracking/teltonika" \
  --header "Authorization: Bearer $TRACKING_KEY" \
  --header "x-device-id: $REGISTERED_IMEI" \
  --header 'Content-Type: application/octet-stream' \
  --data-binary @frame.bin
```

Do not paste secret values into issue reports, shell history exports or shared screenshots. Use the JSON response to decide the next step: 202 accepted/duplicate counts, 400 invalid fields/frame, 401 credentials, 403 entitlement, 404 registered binding unavailable, 409 conflicting retry, 413 body size, 429 rate limit, or 503 service disabled. Codec failures may report INVALID_FRAME, INVALID_CRC or UNSUPPORTED_CODEC; JSON validation reports VALIDATION_FAILED. An unknown 500 outcome must be retried with the original event IDs, not new IDs.
