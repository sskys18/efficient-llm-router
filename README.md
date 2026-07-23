# Efficient LLM Router

Open-source LLM router for the **2026 오픈소스 개발자대회** SK텔레콤 지정과제 (Efficient LLM Routing Challenge).

Routes each prompt to the most cost-effective model within a given budget tier, optimizing quality-per-cost with quality-prediction-based escalation.

## Challenge

[SK텔레콤 지정과제](https://www.kossa.kr/materials/2026/ossp/tasks-skt.html)

- 3 budget tiers: Fast / Balanced / Premium (lower tiers weighted more heavily)
- Offline code router: no external API, network, or model calls
- Final answer must be one of the called candidate model's outputs
- Public data for training/design, private simulator for evaluation

## Status

Planning phase. See [docs/direction.md](docs/direction.md) for the full direction document.

## License

Apache-2.0