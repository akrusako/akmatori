# Akmatori

Akmatori is an AI-powered AIOps agent that integrates with monitoring systems and Slack to provide intelligent incident response and automated remediation.

<img width="1436" height="659" alt="image" src="https://github.com/user-attachments/assets/b2c78bf5-9e20-47da-8ec6-b841c6a0a3de" />

## Key Features

- **Multi-LLM Support**: Use OpenAI, Anthropic, Google, OpenRouter, or on-premise models (GLM, Kimi, Minimax, Mistral, LLaMA)
- **Multi-Source Alert Ingestion**: Receive alerts from Alertmanager, PagerDuty, Grafana, Datadog, Zabbix, and Slack channels
- **Slack Integration**: Post incidents to channels, receive commands, and provide real-time updates
- **AI-Powered Automation**: Analyze incidents and execute remediation skills using your preferred LLM
- **[Agent Skills](https://github.com/agentskills/agentskills) Format**: Skills follow the open Agent Skills specification for portability across AI agents
- **Tools Management**: Configure reusable tools (SSH, Python scripts, API clients) for skills
- **Web Dashboard**: Manage incidents, skills, tools, and settings through a modern UI
- **Context Files**: Upload reference documents for the AI to use during incident analysis
- **Self-Hosted**: Your data never leaves your infrastructure

## Supported LLM Providers

| Provider | Models |
|----------|--------|
| **OpenAI** | GPT-5.4, GPT-5.4 Mini, GPT-5.3 Codex, o4-mini |
| **Anthropic** | Claude Opus 4.6, Sonnet 4.6, Haiku 4.5 |
| **Google** | Gemini 2.5 Pro, 2.5 Flash, 2.0 Flash |
| **OpenRouter** | Access to 100+ models |
| **Custom/On-Prem** | GLM, Kimi, Minimax, Mistral, LLaMA, etc. |

## Quick Start

### Prerequisites

- Docker and Docker Compose v2+
- LLM API key (OpenAI, Anthropic, Google, or OpenRouter)
- Slack App (optional, for Slack integration)

### Installation

1. Clone the repository:
   ```bash
   git clone https://github.com/akmatori/akmatori.git
   cd akmatori
   ```

2. Create and configure environment file:
   ```bash
   cp .env.example .env
   ```

   Edit `.env` and set secure passwords:
   ```bash
   ADMIN_PASSWORD=your-secure-password
   POSTGRES_PASSWORD=your-db-password
   ```

3. Start the services (first run builds containers, takes 3-5 minutes):
   ```bash
   docker-compose up -d
   ```

4. Verify all containers are running:
   ```bash
   docker-compose ps
   ```
   All 5 services should show "Up" status.

5. Access the web dashboard at `http://localhost:8080`
   - **Username:** `admin`
   - **Password:** the `ADMIN_PASSWORD` you set in `.env`

6. Configure your LLM provider in **Settings → LLM Provider**

> **Personal note:** I've been using Gemini 2.5 Flash as my default provider — it's fast, cheap, and handles most incident analysis well. Claude Sonnet 4.6 is worth it for complex runbooks.

> **My setup note:** Running this alongside a homelab Prometheus stack. I mapped the dashboard to port 9090 instead of 8080 to avoid conflicts — just change `ports: - "9090:8080"` for the frontend service in `docker-compose.yml`.

## Architecture

Akmatori uses a secure 4-container architecture:

```
┌─────────────────┐     ┌─────────────────┐     ┌─────────────────┐
│  Alert Sources  │────▶│    API Server   │◀───▶│   PostgreSQL    │
│  (Prometheu
```
