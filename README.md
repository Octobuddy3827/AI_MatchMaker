# 🤖 AI MatchMaker

> **Not sure which AI to use?** AI MatchMaker picks the best model for your task — automatically.

AI MatchMaker intelligently routes your requests to the optimal AI provider based on what actually matters: accuracy, speed, cost, privacy, and whether you need to run offline. Stop guessing — let the algorithm decide.

---

## ✨ What It Does

- **Profiles your task** — understands what you are trying to accomplish
- **Scores every provider** across capability, latency, cost, and privacy
- **Routes to the winner** — sends your request to the best-fit model
- **Falls back gracefully** — if a provider is down, it picks the next best option
- **Explains its choice** — you always know *why* a model was selected

## 🧠 Selection Criteria

| Factor | What It Measures |
|---|---|
| `capability_score` | Model quality for your task type |
| `latency_ms` | Expected response time |
| `cost_per_1k` | Price per 1,000 tokens |
| `privacy_score` | 0 = cloud only → 10 = fully on-device |
| `availability` | Whether the provider is currently reachable |

## 🚀 Getting Started

```bash
git clone https://github.com/Octobuddy3827/AI_MatchMaker.git
cd AI_MatchMaker
# Open index.html in your browser
```

## ⚙️ Configuration

Define your providers in `ai-providers.json`:

```json
{
  "providers": [
    { "name": "OpenAI GPT-4", "capability_score": 9, "latency_ms": 800, "cost_per_1k": 0.03, "privacy_score": 2 },
    { "name": "Local LLaMA", "capability_score": 7, "latency_ms": 200, "cost_per_1k": 0.00, "privacy_score": 10 }
  ],
  "weights": { "capability": 0.4, "latency": 0.2, "cost": 0.2, "privacy": 0.2 }
}
```

## 🛠️ Built With

- HTML / JavaScript
- Configurable JSON-based provider definitions
- Extensible scoring engine

## 📄 License

MIT

