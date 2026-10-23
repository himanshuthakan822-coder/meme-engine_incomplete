# Meme Engine

An AI-powered meme remixing and video-editing application. The product will let users
upload videos, detect on-screen text, rephrase it with AI, and render remixed video output.

> **Status: Foundation only.** This repository currently establishes the project
> architecture and directory layout. No application features are implemented yet.

## Repository Layout

```
meme-engine/
├── frontend/   Next.js + React + TypeScript + Tailwind CSS application
├── backend/    FastAPI (Python) application
├── projects/   Per-user-project working data (created by the application over time)
├── storage/    Local filesystem storage for media and generated artifacts
├── docs/       Project documentation
├── scripts/    Operational / development scripts
└── tests/      Repository-wide / integration tests
```

## Major Directories

### Frontend — `frontend/`
Next.js application (React, TypeScript, Tailwind CSS). UI lives here. Heavy video,
OCR, and AI processing must **not** run in the browser.

- `app/` — Next.js App Router pages and routes
- `components/` — Reusable UI components
- `features/` — Feature-scoped frontend modules
- `hooks/` — Shared React hooks
- `lib/` — Browser-side utilities and API client code
- `types/` — Shared TypeScript types
- `styles/` — Global styles and Tailwind configuration

### Backend — `backend/`
FastAPI (Python) application. All heavy processing happens here.

- `app/api/` — HTTP API endpoints (routers)
- `app/core/` — Core configuration, settings, logging
- `app/models/` — Core domain models / data contracts (see `docs/domain-model.md`)
- `app/schemas/` — Request/response schemas (validation)
- `app/services/video/` — `VideoService` interface: ingest, probing, frame extraction (FFmpeg/FFprobe, OpenCV)
- `app/services/ocr/` — `OCRService` interface: text detection and recognition (OCR)
- `app/services/ai/` — `AnalysisService` interface: AI semantic understanding (OpenAI API)
- `app/services/context/` — `ContextService` interface: meme-context assembly and retrieval
- `app/services/remix/` — `RemixService` interface: caption-candidate generation
- `app/services/text_processing/` — `ValidationService` interface: text cleaning and the six caption checks
- `app/services/rendering/` — `RenderingService` interface: render jobs and output delivery (FFmpeg)
- `app/services/projects/` — `ProjectService` interface: project persistence
- `app/storage/` — Access layer over the local `storage/` directory (`StorageService` interface + `LocalFileStorage` backend)
- `app/validation/` — Validation helpers and rules
- `tests/` — Backend unit and integration tests

### Storage — `storage/`
Local filesystem storage, kept fully separate from source code.

- `uploads/` — Uploaded source media
- `frames/` — Extracted video frames
- `renders/` — Generated / rendered output videos
- `temporary/` — Scratch space for intermediate processing

### Other
- `projects/` — Per-project working data (established later)
- `docs/` — Architecture and design documentation
- `scripts/` — Development and operational scripts
- `tests/` — Repository-wide tests spanning frontend/backend boundaries

## Current Development Stage

Repository foundation, the centralized configuration system, the core
domain models/data contracts, the versioned HTTP API layer, the service-boundary
interfaces, and the storage abstraction. The directory architecture,
documentation, ignoring/environment scaffolding, the backend configuration
module (`backend/app/core/config.py`), the domain models (`backend/app/models/`),
the API layer (`backend/app/api/`, `app/main.py`), the service interfaces
(`backend/app/services/*/interfaces.py`), and the storage layer
(`backend/app/storage/`) are in place. The API exposes liveness/operational
endpoints and the stable error/async-job contracts; processing routes are
explicit `501` placeholders.

## Intentionally NOT Implemented Yet

- AI text generation (OpenAI API)
- OCR / text detection
- Video upload and ingest pipeline
- Video frame extraction, processing, and rendering
- Real API processing endpoints (placeholder routes return `501`)
- The Next.js frontend application
- Database / persistence (no database is planned at this stage)

Do not mistake the `services/` directories for working functionality — each
contains only its capability interface (`interfaces.py`); the implementations
are placeholders that future tasks will fill incrementally.

## Environment Configuration

Configuration is **centralized**. Application code reads a single settings
object; it never calls into environment variables directly.

**Backend** — settings live in `backend/app/core/config.py` (built on
`pydantic-settings`). Values are resolved in this order:

1. process environment variables
2. the repository-root `.env` file (if present)
3. built-in development defaults

All values are validated at load time. Invalid configuration (for example
`APP_ENV=bogus` or `BACKEND_PORT=99999`) fails fast with a clear error instead
of surfacing later as an obscure runtime failure.

**Frontend** — follows the Next.js convention: `NEXT_PUBLIC_*`-prefixed
variables are the only values embedded in browser code. Private values go in
non-public environment files (`.env.local`). Never put secrets in
`NEXT_PUBLIC_*` variables.

### Creating a local environment file

Copy the template to the repository root:

    cp .env.example .env

`env .env.example .env` on Windows (or use a file manager).

### Variables

**Required now:** none — every value has a development-safe default.

**Reserved for future features** (safe to leave empty today):

- `OPENAI_API_KEY`, `OPENAI_MODEL` — consumed by the backend AI service later
- `FRONTEND_URL` — used for CORS once the API is wired to the frontend

## Development

Backend configuration tests (requires Python 3.11+; the repo uses `uv` when
available):

    cd backend
    python -m pip install -r requirements-dev.txt
    python -m pytest
