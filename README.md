<p align="center">
  <img src="assets/Qless-Banner.svg" alt="Qless Banner" width="100%" />
</p>

<p align="center">
  <strong>AI-assisted self-checkout & conversational shopping assistant</strong>
</p>

<p align="center">
  <img alt="Flutter" src="https://img.shields.io/badge/Flutter-^3.12.2-02569B?logo=flutter" />
  <img alt="Python" src="https://img.shields.io/badge/Python-3.10+-3776AB?logo=python" />
  <img alt="FastAPI" src="https://img.shields.io/badge/FastAPI-0.111.0-009688?logo=fastapi" />
  <img alt="License" src="https://img.shields.io/badge/license-proprietary-blue" />
</p>

---

## Overview

Qless is a prototype for seamless, AI-assisted self-checkout in retail. Shoppers scan products with their mobile camera — computer vision (CLIP) matches the item against a vector catalog and resolves metadata locally in milliseconds. A conversational agent powered by Groq (Mixtral 8x7B) handles natural-language queries, cart management, recipe lookups, and nutritional info.

## Architecture

```
┌─────────────────────┐     ┌─────────────────────────────────────┐
│   Flutter App        │     │   FastAPI Backend (hf_server/)      │
│                     │────▶│                                     │
│  Dashboard Screen   │     │  /detect   → CLIP → ChromaDB       │
│  Cart Service       │     │  /chat     → Groq Agent            │
│  Inventory Service  │◀────│  /embed    → Raw embeddings          │
│  Chat Agent Service │     │  /health   → Liveness               │
└─────────────────────┘     └─────────────────────────────────────┘
        │                            │
        ▼                            ▼
┌─────────────────┐       ┌──────────────────────────┐
│  SharedPrefs    │       │  Supabase (auth, carts,  │
│  (offline cart)  │       │  inventory, tenants)     │
└─────────────────┘       └──────────────────────────┘
```

## Tech Stack

| Layer | Technology |
|-------|-----------|
| **Mobile** | Flutter / Dart, Provider, camera, speech_to_text |
| **Backend** | Python, FastAPI, Uvicorn |
| **Vision** | CLIP (ONNX), HuggingFace Transformers, Pillow |
| **Vector DB** | ChromaDB Cloud |
| **LLM** | Groq (Mixtral 8x7B) |
| **Database** | Supabase (PostgreSQL, auth, storage) |
| **Payments** | Razorpay |
| **Infrastructure** | Docker, ngrok, Oracle VM, HuggingFace Spaces |

## Getting Started

### Prerequisites

- Flutter SDK ^3.12.2
- Python 3.10+
- ngrok (for mobile-to-local testing)
- ChromaDB and Supabase API keys

### Backend

```bash
python -m venv .venv
source .venv/bin/activate   # or .\.venv\Scripts\Activate.ps1 on Windows
pip install -r hf_server/requirements.txt
uvicorn hf_server.app.main:app --host 0.0.0.0 --port 8000
```

### Mobile

```bash
flutter pub get
flutter run -d chrome --disable-web-security   # web
# or
flutter run                                    # physical device / emulator
```

### Environment

Copy `.env.example` to `.env` and configure:

| Variable | Purpose |
|----------|---------|
| `CHROMA_API_KEY` | ChromaDB Cloud auth |
| `PRIMARY_DETECTION_URL` | Self-hosted VM URL |
| `BACKUP_DETECTION_URL` | HuggingFace Space URL |
| `SUPABASE_URL` | Supabase project URL |
| `SUPABASE_ANON_KEY` | Supabase anonymous key |
| `RAZORPAY_KEY_ID` | Razorpay key |

## API Endpoints

| Endpoint | Method | Description |
|----------|--------|-------------|
| `/` | GET | Server status & model info |
| `/health` | GET, HEAD | Liveness probe |
| `/embed` | POST | Generate CLIP embeddings |
| `/detect` | POST | Full detection pipeline |
| `/chat/message` | POST | Conversational assistant |
| `/cart-analysis/missing-regulars` | POST | Missing items analysis |

## Multi-Tenant

Qless supports white-label deployments for multiple retailers via `build.sh`:

```bash
./build.sh --tenant lulu apk --release
./build.sh --tenant carrefour web
```

The build script fetches brand theming (colors, fonts, app name) from Supabase and applies it at compile time.

## Project Structure

```
├── android/               # Android platform files
├── assets/                # Fonts, images, SVGs
├── hf_server/             # Python FastAPI backend
│   ├── app/
│   │   ├── agents/        # AI agents (ShoppingAssistant, Recipe, etc.)
│   │   ├── models/        # Pydantic schemas
│   │   ├── routes/        # API route handlers
│   │   ├── services/      # CLIP, Chroma, Supabase, Nutrition services
│   │   ├── utils/         # InMemoryCartStateManager
│   │   ├── config.py      # Model & server config
│   │   └── main.py        # App entrypoint
│   └── tests/             # Backend test suite
├── lib/                   # Flutter app source
│   ├── config/            # Brand/theming config
│   ├── models/            # Data models
│   ├── screens/           # Auth, Chatbot, Dashboard, Inventory, Profile
│   ├── services/          # Cart, Detection, Inventory, Chat, Payment services
│   └── main.dart          # App entrypoint
├── scripts/               # Seed generation, thumbnail uploads, tenant config
├── web/                   # PWA manifest, icons, HTML entry
├── inventory.json         # Local product catalog (56 items)
└── pubspec.yaml           # Flutter dependencies
```

## Deployment

CI/CD via GitHub Actions pushes `hf_server/` to a HuggingFace Space, then SSH-deploys to an Oracle VM. See `.github/workflows/deploy.yml`.

## License

Proprietary — all rights reserved.
