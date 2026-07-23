---
name: OCR an ID and match the holder's face
description: >-
  Extract the fields from an identity document via ADVANCE.AI OCR, then confirm
  the presenter is the document holder with a face-comparison check.
api: openapi/advance-intelligence-group-advance-ai-openapi.yml
operations: [generateToken, ocrKtpCheck, faceComparison]
---

# OCR an ID and match the holder's face

Combine ADVANCE.AI OCR and face comparison to onboard a user from a photo of
their ID plus a selfie. Base URL `https://api.advance.ai`; success is
`code == "SUCCESS"` in the response envelope.

## Steps

1. **Authenticate** — `generateToken`
   `POST /openapi/auth/ticket/v1/generate-token`
   (`accessKey` + `timestamp` + `signature`). Use `data.token` as the
   `X-ACCESS-TOKEN` header below.

2. **Extract ID fields** — `ocrKtpCheck`
   `POST /openapi/face-recognition/v3/ocr-ktp-check` as `multipart/form-data`
   with the `ocrImage` file (max 15 MB). Read the structured `data` fields
   (`idNumber`, `name`, `birthday`, `address`, …). Handle `OCR_NO_RESULT`,
   `IMAGE_INVALID_FORMAT`, `IMAGE_INVALID_SIZE`.

3. **Match the face** — `faceComparison`
   `POST /openapi/face-recognition/v4/check` as `multipart/form-data` with
   `firstImage` (the ID portrait) and `secondImage` (the live selfie). Read
   `data.similarity` (0-100) and apply your own accept threshold.
   Handle `NO_FACE_DETECTED_FROM_FIRST_IMAGE` / `..._SECOND_IMAGE` and the
   `*_LOW_QUALITY_FACE` codes by re-prompting for a clearer image.

## Rules

- Both OCR and face endpoints use `multipart/form-data`, not JSON.
- Log the `transactionId`; note `pricingStrategy` (`FREE`/`PAY`) per call.
- Quality-rejection codes are billable (`PAY`) — validate image quality client
  side before submitting to reduce cost.
