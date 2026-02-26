# AI Match Maker — Installation & Quick Start

This document shows simple, safe ways to preview and run the AI Match Maker demo and how to configure provider settings. It assumes the repository is a mostly static demo (index.html + JS) and that provider credentials should never be committed.

---

## Prerequisites

- Git (to clone the repo)
- Node.js (v16+) for the optional backend proxy
- A modern web browser (Chrome, Firefox, Edge, Safari)
- Docker & Docker Compose (optional, recommended for a reproducible environment)

Note: This repo does not contain cloud API keys. Do not commit secrets to this repository.

---

## 1) Clone the repo

```bash
git clone https://github.com/Octobuddy3827/AI_MatchMaker.git
cd AI_MatchMaker
```

---

## 2) Quick local preview (static)

If the app is a static site (index.html), the easiest way to preview is to serve the directory over HTTP.

Option A — Python 3:

```bash
python3 -m http.server 8000
# Then open http://localhost:8000 in your browser
```

Option B — Node (http-server):

```bash
npm install -g http-server
http-server -p 8000
# then open http://localhost:8000
```

Option C — Use the included backend proxy (recommended if you want to test provider routing):
- Start the proxy (see section 5) and open http://localhost:8080

---

## 3) Provider configuration (ai-providers.json)

Create `ai-providers.json` in the repo root (or the path your frontend expects). This example contains public metadata only — do not put credentials here. The backend should look up credentials in environment variables/secret store and perform the authenticated provider calls.

Example `ai-providers.json` (see sample file in repo):

```json
{
  "weights": {
    "capability": 0.5,
    "latency": 0.2,
    "cost": 0.15,
    "privacy": 0.15
  },
  "providers": [
    {
      "id": "openai-gpt-4o",
      "type": "cloud",
      "display_name": "OpenAI GPT-4o",
      "capability_score": 9.5,
      "latency_ms": 300,
      "cost_per_1k": 2.50,
      "privacy_score": 2,
      "availability": true,
      "endpoint": "https://api.openai.com/v1/..."
    },
    {
      "id": "anthropic-claude-2",
      "type": "cloud",
      "display_name": "Anthropic Claude 2",
      "capability_score": 9.0,
      "latency_ms": 350,
      "cost_per_1k": 2.00,
      "privacy_score": 2,
      "availability": true,
      "endpoint": "https://api.anthropic.com/v1/..."
    },
    {
      "id": "local-llama",
      "type": "local",
      "display_name": "On-device LLaMA",
      "capability_score": 7.0,
      "latency_ms": 150,
      "cost_per_1k": 0.00,
      "privacy_score": 9,
      "availability": false,
      "exec": "/usr/local/bin/llama-server"
    }
  ]
}
```

Notes:
- Replace `endpoint` values with your real endpoints and implement a secure server-side proxy to attach API keys.
- If a provider requires credentials, keep them server-side (environment variables or a secrets manager). The frontend should call your backend, and your backend should call providers.

---

## 4) Running locally with the included Node proxy (recommended for testing)

1. Install dependencies:

```bash
# from repo root
npm install
```

2. Start the server:

```bash
npm start
```

The server will serve the static site and expose a POST endpoint `/api/match` that the frontend can call. The server reads `ai-providers.json` and demonstrates routing logic. Replace the mock provider call code in the server with real provider calls and environment-based credentials for production.

---

## 5) Docker (optional)

Build and run with Docker Compose (recommended for local dev):

```bash
docker compose up --build
# open http://localhost:8080
```

To stop:

```bash
docker compose down
```

---

## 6) Securing API keys and provider calls

- Never embed API keys in client-side JS.
- Add a backend endpoint (e.g., `/api/match`) that:
  - Receives the matching request from the frontend
  - Looks up provider credentials from environment variables or a secret store
  - Calls the selected provider(s)
  - Returns results to the frontend
- Use per-provider rate limiting and error handling.
- Consider auditing and logging provider usage for cost control.

---

## 7) Troubleshooting

- Blank page: check browser console for JS errors and confirm the path to `ai-providers.json`.
- CORS issues: ensure the frontend calls your backend, not provider endpoints directly, or configure the backend to proxy requests.

---

## Next steps I can take for you

- Commit these files to a new branch and open a pull request with the changes.
- Create a minimal server-side example calling one provider securely (requires which provider + how to deliver credentials).
- Extend the selection logic to use a normalized scoring function or ML-based router.

If you want me to push these files into the repository and open a PR, say “Yes — push and open PR” and I will do that for you.
