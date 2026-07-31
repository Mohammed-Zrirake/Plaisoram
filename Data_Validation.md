# Plaisoram Data Validation Architecture & Standards

This document defines the unified, dual-layer data validation strategy enforced across the **Plaisoram Web Dashboard (`plaisoram_web`)** and the **Plaisoram Symfony Server (`Plaisoram_Server`)**.

---

## 🏛️ 1. Architecture Overview: The Mirror Contract Rule

To ensure maximum performance, security, and developer productivity, every user input and API payload MUST follow the **Mirror Contract Design**:

```
 [ User Input on Web ]
          │
          ▼
   Frontend Validation  ──(Zod + React Hook Form)──▶ Instant Inline Errors (No Network Request)
          │ (Valid Data)
          ▼
   HTTP Request (JSON / Query)
          │
          ▼
   Backend Validation   ──(Symfony #[MapRequestPayload] / #[MapQueryString] + DTOs)──▶ Strict 422 JSON Violations
          │ (Valid Payload)
          ▼
  Domain Service Execution (100% Safe Execution)
```

---

## 🛠️ 2. Frontend Validation Standards (`plaisoram_web`)

- **Form Framework**: `react-hook-form`
- **Schema Validation**: `zod`
- **Bridge Resolver**: `@hookform/resolvers/zod`
- **Location**: All schemas reside in `@/schemas/` (e.g., `@/schemas/mediaValidationSchema.ts`).

### Rules:
1. **Pre-flight Check**: Before making any API request (including presigned URLs or file uploads), validate the payload against its Zod schema.
2. **Inline Error Display**: Render form errors under the inputs using RHF `formState.errors`.
3. **Toast Rejections**: For file uploads, reject invalid files (oversized or unsupported resolution) with an instant toast alert.

---

## 🔒 3. Backend Validation Standards (`Plaisoram_Server`)

- **DTO Mapping**: Use `#[MapRequestPayload]` for `POST/PUT/PATCH` and `#[MapQueryString]` for `GET` requests.
- **Validation Engine**: `Symfony\Component\Validator\Constraints` (`Assert`)
- **Location**: All DTOs reside in `App\Modules\<Module>\DTO\` (e.g., `App\Modules\Media\DTO\MediaPresignedUrlDTO`).

### Standard 422 Response Structure:
When validation fails, Symfony automatically returns HTTP `422 Unprocessable Entity`:
```json
{
  "error": "Validation failed",
  "violations": [
    {
      "property": "fileSize",
      "message": "File size must not exceed 500MB (524,288,000 bytes)."
    }
  ]
}
```

---

## 📁 4. Media Upload Constraints (Strict Rules)

| Metric | Constraint Limit | Enforced On | Error Message |
| :--- | :--- | :--- | :--- |
| **Max File Size** | **500 MB** (`524,288,000` bytes) | Frontend & Backend | `"File size exceeds maximum 500MB limit."` |
| **Max Video Resolution** | **4K** ($\le 4096 \times 2160$ px) | Frontend & Backend | `"Video resolution exceeds maximum 4K limit (4096x2160)."` |
| **Allowed Image Mimes** | `image/jpeg`, `image/png`, `image/webp`, `image/gif` | Frontend & Backend | `"Unsupported image format."` |
| **Allowed Video Mimes** | `video/mp4`, `video/webm`, `video/quicktime` | Frontend & Backend | `"Unsupported video format."` |

---

## 🔄 5. Future API Endpoint Mapping Roadmap

Every new endpoint added to Plaisoram MUST have both a Zod schema in `plaisoram_web/src/schemas/` and a corresponding PHP DTO in `Plaisoram_Server/src/Modules/<Module>/DTO/`.
