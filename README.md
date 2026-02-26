# AI Match Maker

AI Match Maker intelligently recommends which AI(s) to use for your project and can route matching requests to the best provider or model based on your goals (accuracy, latency, cost, privacy, and offline capability). This README explains how the selection/choice feature works, how to configure providers, and how to extend the selection logic.

---

## Quick summary of what I changed
I updated the README so that it is clear that it *chooses which AI you should use* rather than leaving that entirely to the integrator. The document now explains the selection criteria, shows a configuration example, and gives integration and deployment guidance. Next, I can scaffold a sample `ai-providers.json` and a small `index.html`/JS demo that implements the selection logic if you'd like.

---

## What AI Match Maker does

- Collects profile and preference data from users.
- Converts that data into a matching request.
- Chooses the most appropriate AI provider or model for the request using a configurable decision algorithm (rules + scoring).
- Sends the request to the selected AI and returns ranked matches with confidence and explanation.
- Supports multiple providers (cloud APIs, managed models, or on-device/local models) and can fall back if a provider is unavailable.

---

## Why automatic AI selection?

Different AI providers excel at different trade-offs. The selection feature lets your app:
- Reduce cost by routing inexpensive queries to lower-cost providers.
- Improve responsiveness by choosing low-latency endpoints.
- Preserve privacy by preferring on-device or self-hosted models when needed.
- Maintain high quality on sensitive or complex queries by routing to higher-capability models.
- Implement business rules (e.g., always use Provider X for financial matching).

---

## Selection criteria (default)

The decision algorithm scores providers on multiple axes:

- capability_score: model quality for semantic/complex matching (higher is better)
- latency_ms: expected request latency (lower is better)
- cost_per_1k: price per 1,000 tokens/requests (lower is better)
- privacy_score: 0 (least private, external cloud) — 10 (fully on-device)
- availability: boolean (if false, provider is skipped)
- custom_rule_weight: adjustable to prioritize business needs

The engine computes a weighted score and selects the provider with the best overall ranking. You can use simple rule overrides (e.g., "if query contains PII -> prefer on-device") or plug in ML-based routing later.

---

## Example provider configuration

Place provider definitions in a JSON or YAML file (example below). The app reads this and uses the weights you configure.

````json
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
AI Match Maker — Installation & Quick Start
This document shows simple, safe ways to preview and run the AI Match Maker demo and how to configure provider settings. It assumes the repository is a mostly static demo (index.html + JS) and that provider credentials should never be committed.

Prerequisites
Git (to clone the repo)
Node.js (v16+) for the optional backend proxy
A modern web browser (Chrome, Firefox, Edge, Safari)
Docker & Docker Compose (optional, recommended for a reproducible environment)
Note: This repo does not contain cloud API keys. Do not commit secrets to this repository.

1) Clone the repo
git clone https://github.com/Octobuddy3827/AI_MatchMaker.git
cd AI_MatchMaker
2) Quick local preview (static)
If the app is a static site (index.html), the easiest way to preview is to serve the directory over HTTP.

Option A — Python 3:

python3 -m http.server 8000
# Then open http://localhost:8000 in your browser
Option B — Node (http-server):

npm install -g http-server
http-server -p 8000
# then open http://localhost:8000
Option C — Use the included backend proxy (recommended if you want to test provider routing):

Start the proxy (see section 5) and open http://localhost:8080
3) Provider configuration (ai-providers.json)
Create ai-providers.json in the repo root (or the path your frontend expects). This example contains public metadata only — do not put credentials here. The backend should look up credentials in environment variables/secret store and perform the authenticated provider calls.

Example ai-providers.json (see sample file in repo):

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
Notes:

Replace endpoint values with your real endpoints and implement a secure server-side proxy to attach API keys.
If a provider requires credentials, keep them server-side (environment variables or a secrets manager). The frontend should call your backend, and your backend should call providers.
4) Running locally with the included Node proxy (recommended for testing)
Install dependencies:
# from repo root
npm install
Start the server:
npm start
The server will serve the static site and expose a POST endpoint /api/match that the frontend can call. The server reads ai-providers.json and demonstrates routing logic. Replace the mock provider call code in the server with real provider calls and environment-based credentials for production.

5) Docker (optional)
Build and run with Docker Compose (recommended for local dev):

docker compose up --build
# open http://localhost:8080
To stop:

docker compose down
6) Securing API keys and provider calls
Never embed API keys in client-side JS.
Add a backend endpoint (e.g., /api/match) that:
Receives the matching request from the frontend
Looks up provider credentials from environment variables or a secret store
Calls the selected provider(s)
Returns results to the frontend
Use per-provider rate limiting and error handling.
Consider auditing and logging provider usage for cost control.
7) Troubleshooting
Blank page: check browser console for JS errors and confirm the path to ai-providers.json.
CORS issues: ensure the frontend calls your backend, not provider endpoints directly, or configure the backend to proxy requests.
Next steps I can take for you
Commit these files to a new branch and open a pull request with the changes.
Create a minimal server-side example calling one provider securely (requires which provider + how to deliver credentials).
Extend the selection logic to use a normalized scoring function or ML-based router.
If you want me to push these files into the repository and open a PR, say “Yes — push and open PR” and I will do that for you.
