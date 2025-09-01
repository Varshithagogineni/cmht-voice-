# CMHT Voice Kiosk

A production-lean voice kiosk that answers CMHT desk questions using Deepgram (STT) → Gemini (LLM) → Supabase pgvector (RAG) → ElevenLabs (TTS).

## Architecture

- **Gateway**: FastAPI backend handling real-time audio and orchestration
- **Kiosk UI**: React frontend with push-to-talk interface
- **RAG**: Supabase with pgvector for document retrieval
- **Orchestration**: n8n webhook integration

## Quick Start

### Prerequisites

- Python 3.11+
- Node.js 18+
- Docker & Docker Compose (optional)
- API keys for:
  - Deepgram (STT)
  - ElevenLabs (TTS)
  - Google Gemini (LLM & Embeddings)
  - Supabase (Database)

### 1. Setup Environment

```bash
# Clone and setup
git clone <repository-url>
cd cmht-voice-kiosk

# Copy environment template
cp env.example .env

# Edit .env with your API keys
# See env.example for all required variables
```

### 2. Database Setup

```bash
# Create Supabase project and enable pgvector extension
# Run the schema.sql in your Supabase SQL editor
cat rag/schema.sql

# Or use Supabase CLI if available
supabase db reset
```

### 3. Document Ingestion

```bash
# Add your PDF documents to rag/input/
# - directions.pdf
# - professor_classroom.pdf  
# - faqs.pdf

# Install dependencies and ingest
cd rag
pip install -r requirements.txt
python ingest.py
```

### 4. Start Services

#### Option A: Local Development

```bash
# Terminal 1: Start gateway
cd gateway
pip install -r requirements.txt
uvicorn app:app --reload --host 0.0.0.0 --port 8000

# Terminal 2: Start kiosk UI
cd kiosk-ui
npm install
npm run dev
```

#### Option B: Docker Compose

```bash
# Start all services
docker-compose up --build

# Or run in background
docker-compose up -d
```

### 5. Configure n8n Webhook

1. Set up n8n instance (cloud or self-hosted)
2. Create webhook workflow using examples in `n8n/webhook_examples.md`
3. Set `N8N_WEBHOOK_URL` in `.env`

## Project Structure

```
cmht-voice-kiosk/
├── gateway/              # FastAPI backend
│   ├── app.py           # Main application
│   ├── models.py        # Pydantic models
│   ├── deps.py          # Dependencies & clients
│   ├── settings.py      # Configuration
│   ├── requirements.txt # Python dependencies
│   ├── Dockerfile       # Container definition
│   └── test_app.py      # Unit tests
├── kiosk-ui/            # React frontend
│   ├── src/
│   │   ├── App.tsx      # Main component
│   │   ├── api.ts       # API client
│   │   └── main.tsx     # Entry point
│   ├── package.json     # Node dependencies
│   ├── vite.config.ts   # Build configuration
│   └── Dockerfile       # Container definition
├── rag/                 # Document ingestion
│   ├── ingest.py        # PDF processing script
│   ├── schema.sql       # Database schema
│   └── requirements.txt # Python dependencies
├── n8n/                 # Webhook examples
│   └── webhook_examples.md
├── docker-compose.yml   # Container orchestration
├── env.example         # Environment template
└── README.md           # This file
```

## API Endpoints

### Gateway API

- `GET /healthz` - Health check
- `POST /api/transcript` - Process voice transcript
- `POST /api/stream-stt` - Real-time STT (optional)

### Example Usage

```bash
# Health check
curl http://localhost:8000/healthz

# Process transcript
curl -X POST http://localhost:8000/api/transcript \
  -H "Content-Type: application/json" \
  -d '{
    "session_id": "test_123",
    "text": "Where is Professor Smith's classroom?",
    "locale": "en-US"
  }'

# Stream STT
curl -X POST http://localhost:8000/api/stream-stt \
  -H "Content-Type: application/json" \
  -d '{
    "session_id": "test_123",
    "audio_data": "base64_encoded_audio",
    "format": "wav"
  }'
```

## Testing

### Gateway Tests

```bash
cd gateway
pip install -r test-requirements.txt
pytest test_app.py -v
```

### Manual Testing

1. Open `http://localhost:5173` in browser
2. Allow microphone permissions
3. Click "Press to Talk" and ask a question
4. Verify response is displayed and audio plays

## Environment Variables

| Variable | Description | Required |
|----------|-------------|----------|
| `N8N_WEBHOOK_URL` | n8n webhook endpoint | Yes |
| `DEEPGRAM_API_KEY` | Deepgram API key | Yes |
| `ELEVENLABS_API_KEY` | ElevenLabs API key | Yes |
| `GEMINI_API_KEY` | Google Gemini API key | Yes |
| `SUPABASE_URL` | Supabase project URL | Yes |
| `SUPABASE_ANON_KEY` | Supabase anonymous key | Yes |
| `SUPABASE_SERVICE_KEY` | Supabase service key (for ingest) | Yes |
| `DEFAULT_VOICE_ID` | ElevenLabs voice ID | No |
| `VITE_API_BASE` | Frontend API base URL | No |

## Development

### Local Development

- Gateway: `http://localhost:8000`
- Kiosk UI: `http://localhost:5173`
- Health check: `http://localhost:8000/healthz`
- API docs: `http://localhost:8000/docs`

### Docker Development

- Gateway: `http://localhost:8000`
- Kiosk UI: `http://localhost:5173`

### Logging

Gateway uses structured logging with JSON format. Log levels:
- `DEBUG`: Detailed debugging information
- `INFO`: General information
- `WARNING`: Warning messages
- `ERROR`: Error messages

### Performance

- STT timeout: 5 seconds
- n8n timeout: 6 seconds  
- TTS timeout: 5 seconds
- Request latency tracking enabled

## Troubleshooting

### Common Issues

1. **Microphone not working**
   - Check browser permissions
   - Ensure HTTPS in production
   - Test with browser dev tools

2. **Gateway not responding**
   - Check environment variables
   - Verify API keys are valid
   - Check logs for errors

3. **n8n webhook failing**
   - Verify webhook URL is correct
   - Check n8n workflow is active
   - Test webhook with curl

4. **Document ingestion failing**
   - Ensure Supabase keys are correct
   - Check PDF files exist in `rag/input/`
   - Verify pgvector extension is enabled

### Debug Mode

```bash
# Enable debug logging
export LOG_LEVEL=DEBUG

# Run gateway with debug
uvicorn app:app --reload --log-level debug
```

## Production Deployment

### Docker Production

```bash
# Build and run production containers
docker-compose -f docker-compose.yml up -d

# Check logs
docker-compose logs -f gateway
docker-compose logs -f kiosk-ui
```

### Environment Setup

1. Set production environment variables
2. Configure reverse proxy (nginx/traefik)
3. Set up SSL certificates
4. Configure monitoring and logging
5. Set up backup for Supabase data

## Contributing

1. Fork the repository
2. Create feature branch
3. Make changes and add tests
4. Run tests: `pytest test_app.py -v`
5. Submit pull request

## License

This project is licensed under the MIT License. 