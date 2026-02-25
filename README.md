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
