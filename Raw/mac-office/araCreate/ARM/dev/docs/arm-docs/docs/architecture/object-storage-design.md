# Object Storage Design — Hetzner S3-Compatible Media Storage

> Spec for adding centralized media storage to the ARM platform using Hetzner Object Storage. Core-be owns all upload/delete operations. Files served directly from Object Storage via public HTTPS URLs.

**Status:** Proposed  
**Date:** 2026-08-21

---

## 1. Problem

Profile pictures are stored as base64 data URLs in PostgreSQL (`users.picture` column, up to 256KB each). This works at current scale but:

- Bloats the database with binary data
- Every query that includes `picture` transfers the full base64 payload
- No path for larger media (org logos, general uploads)
- No CDN-friendly URLs for browser caching

---

## 2. Decision

Use **Hetzner Object Storage** (S3-compatible) with a single public-read bucket. Core-be is the sole service that reads/writes to the bucket. All other services and frontends consume public HTTPS URLs.

---

## 3. Architecture

### Upload flow

```
Browser (multipart/form-data)
    → session-service proxy (passthrough)
    → core-be /v1/media/upload/*
    → validates type/size
    → resizes (avatars only, via sharp)
    → S3 PutObject to Hetzner
    → stores public URL in database
    → returns { url } to frontend
```

### Read flow

```
Browser
    → <img src="https://fsn1.your-object-storage.hetzner.com/arm-media/profiles/{userId}/avatar.jpg">
    → direct from Hetzner (core-be not involved)
```

### Delete flow

```
core-be
    → S3 DeleteObject
    → nulls or updates database column
```

---

## 4. Bucket Structure

Single bucket: **`arm-media`** (public-read ACL)

```
arm-media/
├── profiles/{userId}/avatar.jpg       ← user profile pictures
├── orgs/{orgId}/logo.jpg              ← organization logos
└── media/{userId}/{uuid}.{ext}        ← general platform media
```

**Why single bucket:** Hetzner caps at 100 buckets per account. Prefixes give logical separation without consuming that quota. Access policy is uniform (public-read) for all current media types.

---

## 5. Core-be: New `media` Module

### Files

```
src/media/
├── media.module.ts            ← wires S3 client, controller, service
├── media.controller.ts        ← upload endpoints
├── media.service.ts           ← S3 operations, validation, URL generation
└── dto/
    └── upload-response.dto.ts ← { url: string }
```

### Endpoints

| Method | Path | Guard | Purpose |
|---|---|---|---|
| `POST` | `/v1/media/upload/avatar` | KongJwtGuard | Upload/replace user profile picture |
| `POST` | `/v1/media/upload/org-logo` | KongJwtGuard | Upload/replace organization logo |
| `POST` | `/v1/media/upload` | KongJwtGuard | Upload general media file |
| `DELETE` | `/v1/media/avatar` | KongJwtGuard | Delete the caller's own avatar and clear `users.picture` |

There is deliberately no delete-by-key endpoint. The wildcard `DELETE /v1/media/*key` route this table used to describe had no ownership check, and an Express 5 wildcard param arrives as an array of path segments — so the key the caller meant was mangled (`profiles,<id>,avatar.jpg`), while percent-encoding the slashes reconstructed it exactly and let any authenticated user name any object in the bucket. Keys are now derived server-side from the session user. Adding org-logo or general-media deletion needs a media-ownership record first, not another key parameter.

All endpoints accept `multipart/form-data` with a single `file` field.

### Upload logic per type

**Profile picture (`/upload/avatar`):**
1. Accept JPEG, PNG, or WebP (validate MIME + magic bytes)
2. Reject if raw size > 2 MB
3. Resize to 256×256 centre-crop JPEG 85% quality (sharp)
4. Upload to `profiles/{userId}/avatar.jpg` (overwrites previous)
5. Update `users.picture` with the public URL
6. Delete old object if the key changed (provider URL → Object Storage)
7. Return `{ url }`

**Org logo (`/upload/org-logo`):**
1. Accept JPEG, PNG, WebP, or SVG
2. Reject if raw size > 1 MB
3. No resize (logos need to retain original aspect ratio)
4. Upload to `orgs/{orgId}/logo.{ext}`
5. Return `{ url }`

**General media (`/upload`):**
1. Accept JPEG, PNG, WebP, GIF, or PDF
2. Reject if raw size > 10 MB
3. No resize
4. Generate UUID filename: `media/{userId}/{uuid}.{ext}`
5. Return `{ url }`

### S3 Client Configuration

```typescript
new S3Client({
  endpoint: process.env.S3_ENDPOINT,     // https://fsn1.your-object-storage.hetzner.com
  region: process.env.S3_REGION,         // fsn1
  credentials: {
    accessKeyId: process.env.S3_ACCESS_KEY_ID,
    secretAccessKey: (secret removed),
  },
  forcePathStyle: true,                  // Hetzner uses path-style, not virtual-hosted
});
```

---

## 6. Database Changes

### `users` table

The `picture` column changes from storing `data:image/jpeg;base64,...` to storing `https://fsn1.your-object-storage.hetzner.com/arm-media/profiles/{userId}/avatar.jpg`.

- Column type stays `text` (nullable) — no schema migration needed
- The `IsAvatarDataUrl` validator on `UpdateUserDto` is replaced with a validator that accepts HTTPS URLs
- The `POST /v1/media/upload/avatar` endpoint handles the upload and updates the column — the `PATCH /v1/users/:id` endpoint no longer accepts `picture` as a writable field

### Future: `organizations` table

When orgs get a `logo` column, it stores the Object Storage URL directly. No schema design needed now — this is forward-compatible.

---

## 7. OAuth Provider Pictures

Currently, Google and Microsoft OAuth callbacks store the provider's avatar URL (e.g. `https://lh3.googleusercontent.com/...`) directly in `users.picture`.

**New behaviour:** After OAuth login, core-be:
1. Downloads the provider avatar URL to a buffer
2. Resizes to 256×256 JPEG (same as user uploads)
3. Uploads to `profiles/{userId}/avatar.jpg`
4. Stores the Object Storage URL in `users.picture`

This ensures all profile pictures are served from the same origin (Hetzner), not a mix of Google/Microsoft/Hetzner URLs. Provider URLs can break when users change their Google/Microsoft profile picture — our copy doesn't.

---

## 8. Frontend Changes

### Core-fe settings page

- Replace the current flow (file → canvas downscale → base64 → JSON body) with: file → `FormData` → `POST /session/proxy/core/v1/media/upload/avatar`
- Remove `downscaleToAvatar()` — server handles resizing
- Image preview before upload: use `URL.createObjectURL(file)` (already works, no base64 needed)
- On success: update local user state with the returned `url`

### All frontends (core-fe, calendar-fe, admin-fe)

No display changes needed. `<img src={user.picture}>` already works with HTTPS URLs. The transition from `data:` URLs to `https:` URLs is transparent to rendering.

---

## 9. Environment Variables

| Variable | Example | Service | Required |
|---|---|---|---|
| `S3_ENDPOINT` | `https://fsn1.your-object-storage.hetzner.com` | core-be | Yes |
| `S3_REGION` | `fsn1` | core-be | Yes |
| `S3_BUCKET` | `arm-media` | core-be | Yes |
| `S3_ACCESS_KEY_ID` | (from Hetzner console) | core-be | Yes |
| `(secret removed)` | (from Hetzner console) | core-be | Yes |

**Descriptor change:** Add all five to core-be's `REQUIRED_ENV_VARS` in `core/arm-core-be/Makefile`.

**Deploy change:** Add all five to `deploy/arm-deploy-make/.env.example` and to `REQUIRED_OPTIONAL_VARS` in `config.mk` (platform boots without media storage — it's not a hard dependency like the database).

---

## 10. Migration: Base64 → Object Storage

A one-time TypeORM migration in core-be:

1. Query all rows from `users` where `picture LIKE 'data:%'`
2. For each row:
   - Decode the base64 payload to a buffer
   - Resize to 256×256 JPEG 85% via sharp (normalizes old formats)
   - Upload to `profiles/{userId}/avatar.jpg`
   - Update `picture` to the Object Storage URL
3. Log failures per-user but don't abort — set `picture = NULL` for any row that fails
4. Remove the `IsAvatarDataUrl` validator after migration runs

**Rollback:** The migration is one-way. If rolled back, `picture` would contain Object Storage URLs that still resolve — no data loss, just the column format doesn't match the old validator. A reverse migration could re-download and re-encode to base64 if needed, but this is unlikely.

---

## 11. New Dependencies (core-be)

| Package | Purpose |
|---|---|
| `@aws-sdk/client-s3` | S3 client — works with Hetzner's S3-compatible API |
| `sharp` | Server-side image resizing for avatars |

Both are well-maintained, widely used, and have no known security issues.

---

## 12. What Doesn't Change

| Component | Why |
|---|---|
| **session service** | Pure proxy — passes multipart through, no awareness of media |
| **Calendar-be** | Reads `picture` from account objects via core-be — URL format is transparent |
| **Admin-be** | Reads user data from core-be — same |
| **Notification service** | No media involvement |
| **Auth guards** | Upload endpoints use the existing `KongJwtGuard` |
| **Module Federation** | No change — frontends already render `<img src={url}>` |

---

## 13. Security Considerations

- **No public write access.** The bucket is public-read but write requires S3 credentials, which only core-be has.
- **File type validation.** Both MIME header and magic bytes checked server-side. No relying on `Content-Type` from the client.
- **Size limits enforced server-side.** session-service proxy has its own body size limit; core-be enforces per-type limits.
- **No path traversal.** Object keys are constructed server-side from `userId`/`orgId`/UUID — never from user input.
- **Credentials.** `(secret removed)` and `(secret removed)` are secrets — same handling as `JWT_ACCESS_SECRET` (env var, never logged, never in browser).

---

## 14. Hetzner Object Storage Constraints

| Limit | Value |
|---|---|
| Max buckets per account | 100 |
| Max per bucket | 100 TB |
| Max objects per bucket | 50 million |
| Max single PUT | 5 GB |
| Min billable object size | 64 KB |
| Requests per bucket | 750/s |
| Included storage | 1 TB |
| Included egress | 1 TB/mo |
| Base cost | €4.99/mo |

None of these are remotely close to being hit by ARM's current scale.

---

## 15. Bucket Setup (Manual, One-Time)

Done in Hetzner Cloud Console before deployment:

1. Create Object Storage bucket `arm-media` in `fsn1` (Falkenstein)
2. Set bucket ACL to public-read
3. Generate S3 credentials (access key + secret key)
4. Add credentials to ARM's `.env` on the server
