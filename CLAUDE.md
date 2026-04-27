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

## Personal Notes (fork)

- I'm primarily using this with Alertmanager + VictoriaMetrics; Zabbix-specific paths are less relevant for my setup.
- My preferred LLM provider is Anthropic (claude-sonnet). OpenRouter is a useful fallback for cost control.
- TODO: explore the `qmd/` runbook search — specifically whether it can index our internal Confluence runbooks via the HTTP connector.
- The VictoriaMetrics tool in `mcp-gateway/internal/tools/` is the most relevant for my setup; worth reading through its query-building logic.
- When testing locally, use `docker compose up api agent-worker mcp-gateway postgres` — skipping `qmd` saves startup time if runbook search isn't needed.
