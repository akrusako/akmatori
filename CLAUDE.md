# Claude Code Instructions for Akmatori

## Project Overview

Akmatori is an AI-powered AIOps platform that receives alerts from monitoring systems (Zabbix, Alertmanager, PagerDuty, Grafana, Datadog), analyzes them using multi-provider LLM agents (via the pi-mono coding-agent SDK), and executes automated remediation.

## Architecture

- **5-container Docker architecture**: API, Agent Worker, MCP Gateway, PostgreSQL, QMD (runbook search)
- **Backend**: Go 1.24+ (API server, MCP gateway)
- **Agent Worker**: Node.js 22+ / TypeScript using `@mariozechner/pi-coding-agent` SDK (v0.67.6)
- **Frontend**: React 19 + TypeScript + Vite + Tailwind
- **Database**: PostgreSQL 16 with GORM
- **LLM Providers**: Anthropic, OpenAI, Google, OpenRouter, Custom (configured via web UI)

## Key Directories

```
/opt/akmatori/
├── cmd/akmatori/           # Main API server entry point
├── internal/
│   ├── alerts/adapters/    # Alert source adapters (Zabbix, Alertmanager, etc.)
│   ├── alerts/extraction/  # AI-powered alert extraction from free-form text
│   ├── api/                # Request/response helpers, pagination
│   ├── database/           # GORM models and database logic
│   ├── handlers/           # HTTP/WebSocket handlers
│   ├── middleware/         # Auth, CORS middleware
│   ├── output/             # Agent output parsing (structured blocks)
│   ├── logging/           # Structured logging (slog) initialization
│   ├── services/           # Business logic layer (+ interfaces.go for testability)
│   ├── setup/              # Zero-config first-run setup
│   ├── slack/              # Slack integration (Socket Mode, hot-reload)
│   ├── testhelpers/        # Test utilities, builders, mocks
│   └── utils/              # Utility functions
├── agent-worker/           # Node.js/TypeScript agent worker
│   └── src/                # TypeScript source (gateway-client, gateway-tools, script-executor)
├── mcp-gateway/            # MCP protocol gateway (separate Go module)
│   └── internal/
│       ├── auth/           # Per-incident tool authorization (allowlist enforcement)
│       ├── cache/          # Generic TTL cache
│       ├── mcpproxy/       # MCP proxy: connection pool + handler for external MCP servers
│       ├── ratelimit/      # Token bucket rate limiter
│       └── tools/          # SSH, Zabbix, VictoriaMetrics, PostgreSQL, ClickHouse, Grafana, Catchpoint, PagerDuty, NetBox, Kubernetes, and HTTP connector implementations
├── web/                    # React frontend
├── qmd/                    # QMD search sidecar (Dockerfile, config, entrypoint)
├── docs/                   # OpenAPI specs (swagger at /api/docs)
└── tests/fixtures/         # Test payloads and mock data
```

## CRITICAL: Always Verify Changes with Tests

**After ANY code change, run the appropriate test command:**

| After changing... | Run command |
|-------------------|-------------|
| Alert adapters (`internal/alerts/adapters/`) | `make test-adapters` |
| MCP Gateway (`mcp-gateway/`) | `make t

## Personal Notes (Fork)

- I'm primarily using this with Anthropic (Claude) as the LLM provider — OpenRouter is a good fallback for cost reasons
- For local dev, the QMD container can be skipped if you don't need runbook search; saves memory
- TODO: explore adding a Prometheus adapter alongside the existing Alertmanager one
