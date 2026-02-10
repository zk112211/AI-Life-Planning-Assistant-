# AI Life Planning Assistant

An AI-powered trip planning app that generates personalized day plans from natural language. Built with a multi-agent backend (FastAPI + Google Gemini) and a cross-platform Flutter frontend with real-time WebSocket alerts.

## Demo

> _Add a screenshot or GIF here showing the app in action_

## Architecture

```
┌──────────────────────────────────────────────────┐
│                  Flutter App                      │
│  ┌──────────┐  ┌──────────┐  ┌────────────────┐  │
│  │ Planner  │  │ Plan     │  │ Real-time      │  │
│  │ Screen   │  │ Detail   │  │ Alert Banner   │  │
│  └────┬─────┘  └────┬─────┘  └───────┬────────┘  │
│       │              │                │           │
│       └──────────────┼────────────────┘           │
│                      │                            │
│              REST / WebSocket                     │
└──────────────────────┼────────────────────────────┘
                       │
┌──────────────────────┼────────────────────────────┐
│              FastAPI Backend                       │
│                      │                            │
│         ┌────────────┼────────────────┐           │
│         │      Orchestrator           │           │
│         │                             │           │
│    ┌────▼────┐  ┌──────────┐  ┌──────▼─────┐     │
│    │  Scout  │→ │Simulator │→ │ Validator  │     │
│    │ (context│  │(generate │  │ (score &   │     │
│    │  gather)│  │ plans)   │  │  validate) │     │
│    └────┬────┘  └────┬─────┘  └──────┬─────┘     │
│         └────────────┼───────────────┘            │
│                      │                            │
│              Google Gemini API                    │
└───────────────────────────────────────────────────┘
```

## Key Features

**Multi-Agent Planning Pipeline**
- **Scout** — Gathers context (weather, location, budget constraints)
- **Simulator** — Generates candidate day plans from natural language intent
- **Validator** — Scores and validates plans, triggers re-planning if needed

**Real-Time Monitoring**
- WebSocket-based alerts for weather changes, traffic, or constraint violations
- Automatic re-planning when conditions change

**Cross-Platform Frontend**
- Flutter app with Provider state management
- Plan creation, browsing, and detail views
- Live alert banners and notification system

## Tech Stack

| Layer | Technology |
|-------|------------|
| Frontend | Flutter (Dart 3.x), Provider, WebSocket |
| Backend | FastAPI, Python 3.11, Uvicorn, Pydantic |
| AI | Google Gemini API (`google-genai`) |
| Communication | REST API + WebSocket |

## Project Structure

```
AI-Life-Planning-Assistant/
├── backend/
│   ├── app/
│   │   ├── agents/          # Scout, Simulator, Validator agents
│   │   ├── api/             # REST endpoints & WebSocket routes
│   │   ├── schemas/         # Pydantic models
│   │   └── services/        # LLM client, orchestrator, tools
│   ├── main.py
│   └── requirements.txt
├── frontend/
│   ├── lib/
│   │   ├── models/          # Data models (trip plan, alerts)
│   │   ├── providers/       # State management
│   │   ├── screens/         # UI screens
│   │   ├── services/        # API client, WebSocket service
│   │   └── widgets/         # Reusable UI components
│   └── pubspec.yaml
└── README.md
```

## Getting Started

### Prerequisites

- Python 3.11+
- Flutter SDK 3.0+ — [Install](https://docs.flutter.dev/get-started/install)
- Google Gemini API key — [Get key](https://ai.google.dev/)

### Backend

```bash
cd backend
python3.11 -m venv venv
source venv/bin/activate
pip install -r requirements.txt
```

Create `.env` in `backend/`:
```env
GEMINI_API_KEY=your_api_key_here
```

Run:
```bash
uvicorn main:app --reload --host 0.0.0.0 --port 8000
```

API docs available at `http://localhost:8000/docs`

### Frontend

```bash
cd frontend
flutter pub get
flutter run
```

Make sure the API base URL in `frontend/lib/config/app_config.dart` points to your backend (default: `http://localhost:8000`).

## API Endpoints

| Method | Endpoint | Description |
|--------|----------|-------------|
| POST | `/api/plan/generate` | Generate plan from natural language |
| GET | `/api/plan/{plan_id}` | Retrieve plan by ID |
| WebSocket | `/api/ws/alerts/{plan_id}` | Real-time alerts stream |
| GET | `/health` | Health check |

### Example Request

```bash
curl -X POST http://localhost:8000/api/plan/generate \
  -H "Content-Type: application/json" \
  -d '{"intent": "romantic date in SF, budget $200"}'
```

## Design Highlights

- **Structured Output** — All Gemini responses are validated as JSON through Pydantic schemas, achieving 100% parsing success rate
- **Separation of Concerns** — Each agent (Scout/Simulator/Validator) has a single responsibility, making the pipeline easy to extend
- **Reactive Frontend** — Provider-based state management with WebSocket listeners for real-time UI updates
- **Error Resilience** — Graceful fallbacks when API calls fail or plans need re-generation
