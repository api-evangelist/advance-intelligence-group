---
name: Verify an identity document (eKYC)
description: >-
  Run an ADVANCE.AI global document-verification session end to end — authenticate,
  authorize the client SDK, then retrieve the verification result including OCR
  output and ID-forgery detection.
api: openapi/advance-intelligence-group-advance-ai-openapi.yml
operations: [generateToken, getDocumentVerificationAuthLicense, queryDocumentVerification]
---

# Verify an identity document (eKYC)

Use the ADVANCE.AI Open API (`https://api.advance.ai`) to verify a government ID.
All calls return HTTP 200 with a business status in the envelope `code` field;
treat `code == "SUCCESS"` as success, anything else as an error (see
`errors/advance-intelligence-group-error-codes.yml`).

## Steps

1. **Authenticate** — `generateToken`
   `POST /openapi/auth/ticket/v1/generate-token` with `accessKey`, a 13-digit
   `timestamp`, and `signature = SHA256(accessKey + secretKey + timestamp)`.
   Read `data.token` and send it as the `X-ACCESS-TOKEN` header on every
   subsequent call. Re-request when `data.expiredTime` passes.

2. **Authorize the client SDK** — `getDocumentVerificationAuthLicense`
   `POST /intl/openapi/face-identity/document-verification/v1/auth-license`.
   Optionally scope with `applicationId` and set `licenseEffectiveSeconds`.
   The client SDK uses the returned license to run the capture session and
   yields an `IDVID`.

3. **Fetch the result** — `queryDocumentVerification`
   `POST /intl/openapi/face-identity/document-verification/v1/query` with the
   `IDVID`. Set `resultType` to `IMAGE_URL` (24-hour links) or `IMAGE_BASE64`.
   The `data` payload carries OCR fields, forgery flags and document images.

## Rules

- Correlate and log every call by the response `transactionId` (<=64 chars).
- `pricingStrategy` tells you whether a call was `FREE` or `PAY`.
- `IDVID_NOT_EXISTS` means the session id is invalid/expired — restart at step 2.
- No idempotency key is supported; do not blindly retry a `PAY` submission.
