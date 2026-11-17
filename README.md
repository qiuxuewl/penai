<div align="center">

# PenAI

AI-powered autonomous penetration testing system (Go backend + Next.js frontend)

</div>

---

## Overview

PenAI is an AI agent system for autonomous penetration testing. It uses a dual-graph architecture — a shared asset graph and a per-task exploration graph — driven by a planner and multiple workers in an event-driven loop.

**Key capabilities:**
- Autonomous recon, vulnerability discovery, and exploitation chaining
- Nuclei-based CVE scanning with daily-updated templates
- Real-time asset graph (domains, IPs, services, endpoints)
- Built-in MITM traffic proxy and tool approval gate
- MCP and skill extensibility
- Notifications (DingTalk, Feishu, WeCom)
- Vulnerability retest and report generation

---

## Requirements

- PostgreSQL
- LLM API key: `ANTHROPIC_API_KEY` or `OPENAI_API_KEY` (configurable in UI)
- Default port: `8787`

---

## Installation

### Docker Compose (recommended)

```bash
git clone https://github.com/qiuxuewl/penai.git
cd penai
cp .env.example .env
# edit .env with your API keys and DB config
docker compose up -d
```

### Build from source

```bash
go build ./cmd/penai
./penai
```

---

## Skills

| Skill | Description |
|-------|-------------|
| `nuclei-scan` | CVE validation using Nuclei templates (updated daily) |
| `api-recon` | Frontend API surface collection |
| `playwright-cli` | Browser automation for runtime recon |
| `scopesentry` | Asset sync with ScopeSentry |

---

## Enhancements over upstream

- **Nuclei integration** (`skills/nuclei/`) — fingerprint-triggered CVE scanning
- **Extended attack surface** — added 8 modern techniques to planner prompt:
  - HTTP Request Smuggling (CL.TE / TE.CL)
  - GraphQL introspection + batch query abuse
  - CORS misconfiguration
  - Cross-Site WebSocket Hijacking (CSWSH)
  - OAuth 2.0 / OIDC misconfiguration
  - Prototype Pollution (frontend + Node.js)
  - Race condition / TOCTOU
  - Cloud SSRF to IMDS credential leak chain

---

## Versioning

| Tag | Description |
|-----|-------------|
| `v0.3.16-nuclei` | Nuclei skill + modern attack surface update |

---

## License

AGPL-3.0

---

> **For authorized security testing and research only.**
