# InnerVoice (CESPF-NLP)

InnerVoice is a privacy-focused journaling application for recording reflections and receiving empathetic, AI-assisted insights. The project combines a cross-platform React Native client with a Django REST API that anonymizes journal text, detects crisis language, retrieves relevant past entries, and generates asynchronous reflections through OpenRouter.

> [!IMPORTANT]
> InnerVoice is a journaling and reflection tool, not a replacement for professional mental health care or emergency services.

## Features

- **Cross-platform app:** Runs on Android, iOS, and the web from a React Native and Expo codebase.
- **Private journaling:** Stores raw journal text in encrypted database fields and keeps an encrypted local journal cache on the client.
- **PII anonymization:** Uses spaCy named-entity recognition to replace people, locations, and organizations before text is sent to an AI provider.
- **Crisis-language interception:** Applies deterministic phrase matching and a local classifier before normal journal processing.
- **AI-assisted reflection:** Uses OpenRouter and a project-specific system prompt to generate empathetic replies, emotion labels, topics, cognitive patterns, and suggested prompts.
- **Context-aware insights:** Creates local text embeddings and, when PostgreSQL with pgvector is enabled, retrieves semantically related entries for additional context.
- **Weekly patterns:** Aggregates emotions, topics, activity, and AI-generated weekly reflections.
- **Asynchronous processing:** Uses Celery with Redis or Valkey for production-style background insight generation. Local development can run tasks eagerly without a worker.
- **JWT authentication:** Supports registration, login, access-token refresh, and user-scoped journal data.

## Tech Stack

### App

- React Native 0.81
- Expo 54
- Expo Router 6
- React 19
- TypeScript
- React Navigation
- AsyncStorage and Expo SecureStore

### API and data

- Python and Django
- Django REST Framework
- Simple JWT
- Celery
- SQLite for the default local setup
- PostgreSQL and pgvector for semantic retrieval
- Redis or Valkey for caching and queued Celery tasks
- Encrypted Django model fields

### NLP and AI

- OpenRouter API
- spaCy for named-entity anonymization
- FastEmbed with `sentence-transformers/all-MiniLM-L6-v2`
- scikit-learn for local crisis-language classification

## Repository Structure

```text
.
├── innervoice-api/   Django REST API, NLP pipeline, Celery tasks, and data models
└── innervoice-app/   React Native app built with Expo Router and TypeScript
```

More package-specific notes are available in [`innervoice-api/README.md`](innervoice-api/README.md) and [`innervoice-app/README.md`](innervoice-app/README.md).

## Architecture

1. The Expo app authenticates with the Django API and sends a journal entry.
2. The API checks the entry for crisis language. A detected risk stops the normal pipeline and instructs the client to show its crisis-support screen.
3. spaCy replaces recognized names, locations, and organizations with placeholders.
4. FastEmbed generates a 384-dimensional embedding locally.
5. With PostgreSQL and pgvector, the API retrieves up to three semantically similar entries belonging to the same user. SQLite development skips this step.
6. The raw entry is stored in an encrypted field; its anonymized form and embedding are stored for processing and retrieval.
7. A Celery task sends only the anonymized entry and anonymized historical context to OpenRouter.
8. The generated reply and structured emotional insight are saved for the app to retrieve.

## Getting Started

### Prerequisites

- Git
- Python 3
- Node.js LTS and npm
- An OpenRouter API key for generated AI insights

PostgreSQL, pgvector, and Redis or Valkey are optional for the basic local setup. By default, Django uses SQLite, an in-memory cache, and eager Celery tasks.

### 1. Clone the repository

```bash
git clone <repository-url>
cd innervoice-workplace
```

### 2. Set up the API

```bash
cd innervoice-api
python3 -m venv .venv
source .venv/bin/activate
python -m pip install -r requirements.txt
cp .env.example .env
python -m spacy download en_core_web_sm
python manage.py migrate
```

On Windows PowerShell, activate the environment with:

```powershell
.\.venv\Scripts\Activate.ps1
```

Edit `innervoice-api/.env` and set at least:

```env
OPENROUTER_API_KEY=your_openrouter_api_key
```

Then start Django:

```bash
python manage.py runserver
```

Use `0.0.0.0:8000` when connecting from a physical device or another machine:

```bash
python manage.py runserver 0.0.0.0:8000
```

The embedding model is cached in `innervoice-api/local_ai_models/` and may be downloaded the first time an entry is processed.

### 3. Set up the Expo app

In a second terminal:

```bash
cd innervoice-app
npm install
npm run start
```

From the Expo terminal, open the app on a supported Android or iOS device/emulator, or in a web browser. Dedicated scripts are also available:

```bash
npm run android
npm run ios
npm run web
```

The default API URL is `http://127.0.0.1:8000`. Override it without editing committed configuration by setting `EXPO_PUBLIC_API_BASE_URL`:

```bash
EXPO_PUBLIC_API_BASE_URL=http://192.168.1.20:8000 npm run start
```

Choose the URL for your environment:

- iOS simulator or web: `http://127.0.0.1:8000`
- Android emulator: `http://10.0.2.2:8000`
- Physical device: your computer's LAN address, with both devices on the same network

When using Expo web, add the Expo development-server origin to `CORS_ALLOWED_ORIGINS` in `innervoice-api/.env` if it differs from the provided defaults.

## Optional Production-Style Services

To enable pgvector-backed contextual retrieval, set `DATABASE_ENGINE=postgres` and configure the `POSTGRES_*` variables in `innervoice-api/.env`.

To process AI insights through a background worker, start Redis or Valkey, then set:

```env
CELERY_TASK_ALWAYS_EAGER=false
CELERY_BROKER_URL=redis://127.0.0.1:6379/0
```

The Redis protocol URL also works with Valkey. Start a worker from `innervoice-api`:

```bash
.venv/bin/celery -A InnerVoice worker -l info
```

To run the scheduled Sunday synthesis job, start Celery Beat separately:

```bash
.venv/bin/celery -A InnerVoice beat -l info
```

## API Endpoints

All journal endpoints require a JWT bearer token.

| Method | Endpoint | Purpose |
| --- | --- | --- |
| `POST` | `/api/v1/auth/register/` | Create an account |
| `POST` | `/api/v1/auth/login/` | Obtain access and refresh tokens |
| `POST` | `/api/v1/auth/refresh/` | Refresh an access token |
| `GET` | `/api/v1/journal/entries/` | List the authenticated user's entries |
| `POST` | `/api/v1/journal/submit/` | Submit and process a journal entry |
| `GET` | `/api/v1/journal/insights/latest/` | Fetch the latest generated entry insight |
| `GET` | `/api/v1/journal/insights/weekly/` | Fetch weekly insight data |

## Development Checks

API:

```bash
cd innervoice-api
.venv/bin/python manage.py check
.venv/bin/python manage.py test
.venv/bin/python -m pip check
```

App:

```bash
cd innervoice-app
npm run lint
npx tsc --noEmit
npm run format:check
```

## Contributing

Contributions, bug reports, and feature requests are welcome. Please run the relevant checks before opening a pull request.

## License

The app includes the [GNU General Public License v3.0](innervoice-app/LICENSE.md).
