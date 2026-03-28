# Pi-Coding-Agent SDK Upgrade Plan: 0.58.1 → 0.63.1

## Current State

| Package | Current | Latest | Gap |
|---------|---------|--------|-----|
| `@mariozechner/pi-coding-agent` | 0.58.1 (`^0.58.0`) | 0.63.1 | 12 releases |
| `@mariozechner/pi-ai` | 0.58.1 (`^0.58.0`) | 0.63.1 | 12 releases |
| `@mariozechner/pi-agent-core` | 0.58.1 (`^0.58.0`) | 0.63.1 | 12 releases |

New required dependency in 0.63.0: `ajv@^8.17.1` (JSON Schema validation, now a direct dep).

---

## Release Timeline (0.58.1 → 0.63.1)

| Version | Date | Type | Key Change |
|---------|------|------|------------|
| 0.58.1 | 2026-03-14 | Patch | Current resolved version |
| 0.58.2 | 2026-03-15 | Patch | Bug fixes |
| 0.58.3 | 2026-03-15 | Patch | Bug fixes |
| 0.58.4 | 2026-03-16 | Patch | Steering messages fix (wait for batch completion) |
| **0.59.0** | 2026-03-17 | **Minor (breaking)** | Tool system prompt behavior change |
| **0.60.0** | 2026-03-18 | **Minor (breaking)** | `createLocalBashOperations()` export, `--fork` flag |
| **0.61.0** | 2026-03-20 | **Minor (breaking)** | Namespaced keybindings, JSONL export/import |
| 0.61.1 | 2026-03-20 | Patch | `ToolCallEventResult` typed exports |
| **0.62.0** | 2026-03-23 | **Minor (breaking)** | `sourceInfo` provenance, tool rendering changes |
| **0.63.0** | 2026-03-27 | **Minor (breaking)** | `getApiKey()` → `getApiKeyAndHeaders()`, multi-edit |
| 0.63.1 | 2026-03-27 | Patch | Fix repeated compaction message loss |

---

## Part 1: Breaking Changes Impact Assessment

### 0.59.0 — Tool system prompt behavior

**Change**: Extension/SDK tools are included in the `Available tools` system prompt section only when they provide `promptSnippet`. Omitting `promptSnippet` now excludes the tool from the prompt (previously fell back to `description`).

**Impact on Akmatori**: **LOW-MEDIUM** — Our custom gateway tools in `gateway-tools.ts` use `description` and `promptGuidelines`. We do NOT currently set `promptSnippet` on our custom tools.

**Action Required**: Verify that our custom tools (gateway_call, list_tool_types, list_tools_for_tool_type, get_tool_detail, execute_script) still appear in the system prompt. If not, add `promptSnippet` to each tool definition.

### 0.60.0 — Package startup & new exports

**Change**: Installed unpinned packages no longer auto-updated during startup. New `createLocalBashOperations()` export and `--fork` flag.

**Impact on Akmatori**: **NONE** — We don't use packages/extensions. `createLocalBashOperations()` is a new export we could optionally use.

**Action Required**: None.

### 0.61.0 — Namespaced keybindings

**Change**: Keybinding IDs are now namespaced (e.g., `"expandTools"` → `"app.tools.expand"`).

**Impact on Akmatori**: **NONE** — We run headless, no keybindings used.

**Action Required**: None.

### 0.62.0 — Source provenance & tool rendering

**Changes**:
- `ToolDefinition.renderCall` and `renderResult` semantics changed: fallback rendering only happens when renderer is not defined. If defined, it must return a `Component`.
- Removed `source` fields from `Skill` and `PromptTemplate` (use `sourceInfo.source`).
- Removed `ResourceLoader.getPathMetadata()`.
- Removed `extensionPath` from `RegisteredCommand` and `RegisteredTool`.

**Impact on Akmatori**: **NONE** — We don't use renderCall/renderResult, don't access `source` fields, don't use `getPathMetadata()`, and don't use `extensionPath`.

**Action Required**: None.

### 0.63.0 — ModelRegistry API change ⚠️

**Change**: **`ModelRegistry.getApiKey(model)` replaced by `getApiKeyAndHeaders(model)`**. Returns `{ apiKey, headers }` instead of just the API key string.

**Impact on Akmatori**: **MUST VERIFY** — We use `ModelRegistry` but set keys via `AuthStorage.setRuntimeApiKey()`. We may not call `getApiKey()` directly. If we do, this is a compile-time error that must be fixed.

**Action Required**: Check if `agent-runner.ts` or any other file calls `modelRegistry.getApiKey()`. If yes, update to `getApiKeyAndHeaders()` and destructure `{ apiKey }`.

### Summary

| Breaking Change | Version | Impact | Action |
|---|---|---|---|
| Tool `promptSnippet` required for system prompt | 0.59.0 | **LOW-MEDIUM** | Verify tools appear; add `promptSnippet` if needed |
| Package auto-update removed | 0.60.0 | None | — |
| Namespaced keybindings | 0.61.0 | None | — |
| `renderCall`/`renderResult` semantics | 0.62.0 | None | — |
| `sourceInfo` replaces `source` | 0.62.0 | None | — |
| Removed `getPathMetadata()` | 0.62.0 | None | — |
| Removed `extensionPath` | 0.62.0 | None | — |
| **`getApiKey()` → `getApiKeyAndHeaders()`** | 0.63.0 | **VERIFY** | Check usage, update if needed |
| Removed `minimax` model IDs | 0.63.0 | None | — |

---

## Part 2: New Features & Opportunities

### Tier 1: Free with Upgrade (Zero Effort)

#### 1.1 — Repeated Compaction Message Loss Fix (0.63.1)
Messages were being dropped during re-compaction. Critical for our long-running investigations that hit context limits multiple times.

#### 1.2 — Concurrent Edit/Write Serialization (0.61.0)
Prevents interleaved file writes when the agent makes parallel tool calls. Eliminates potential file corruption in workspace.

#### 1.3 — `session.prompt()` Retry Loop Fix (0.61.0)
`session.prompt()` now waits for the full retry loop including tool execution before returning. Previously could return prematurely.

#### 1.4 — Steering Messages Batch Fix (0.58.4)
Fixed steering messages skipping tool calls — now waits for batch completion. Better agent reliability.

#### 1.5 — `session_shutdown` Event (0.63.0)
Extensions/SDK can release resources cleanly on session shutdown.

#### 1.6 — RPC `contextUsage` Exposure (0.63.0)
Headless clients can now read context usage — useful for monitoring/alerting on context utilization per incident.

### Tier 2: Low Effort, High Value

#### 2.1 — Multi-Edit Tool Support (0.63.0)
One tool call can update multiple disjoint regions in the same file. Agents can make more efficient edits, reducing tool call count and token usage.

**Benefit**: Faster incident resolution, lower LLM costs.
**Effort**: Automatic — the agent will use it when the model supports it.

#### 2.2 — `promptSnippet` for Custom Tools (0.59.0+)
Our gateway tools should provide `promptSnippet` to ensure they appear in the system prompt's "Available tools" section.

**Implementation**:
```typescript
// In gateway-tools.ts, for each tool:
{
  name: "gateway_call",
  promptSnippet: "Call MCP Gateway tools (SSH, Zabbix, etc.) via gateway_call(toolName, params, instance)",
  // ... rest of definition
}
```

**Effort**: Small — add one field per tool.
**Benefit**: Ensures tools remain visible to the agent after upgrade.

#### 2.3 — `ToolCallEventResult` Typed Exports (0.61.1)
Typed return values for `tool_call` handler. Better type safety in our event handling code.

#### 2.4 — `validateToolArguments()` Graceful Fallback (0.61.0)
Works in restricted runtimes — relevant if our agent container has limited Node.js APIs.

### Tier 3: Medium Effort, High Value

#### 3.1 — `getApiKeyAndHeaders()` for Custom Headers (0.63.0)
The new API returns both `apiKey` and `headers` per request. This enables:
- Custom headers per provider (e.g., org-id, tracking headers)
- Better integration with enterprise proxy/gateway setups
- AWS Bedrock cost allocation tagging (0.62.0)

**Implementation**: When we upgrade, update any `getApiKey()` calls to `getApiKeyAndHeaders()`. Even if we don't call it directly, this signals a shift toward header-based auth that we should leverage.

#### 3.2 — `sessionDir` Setting (0.63.0)
Configurable session directory in `settings.json` without CLI flag. Combined with our existing workspace-per-incident pattern, this lets us control session storage location programmatically via `SettingsManager`.

**Implementation**:
```typescript
const settings = SettingsManager.inMemory();
settings.set("sessionDir", path.join(WORKSPACE_DIR, incidentId));
```

**Benefit**: Aligns session files with incident workspace. Simpler than the extension hook approach from 0.57.1.

#### 3.3 — AWS Bedrock Cost Allocation Tags (0.62.0)
`requestMetadata` option for cost tracking. Enterprise customers using Bedrock can tag LLM costs by incident/tenant.

#### 3.4 — `./hooks` Sub-path Export (0.63.0+)
New entry point `@mariozechner/pi-coding-agent/hooks` — provides hook utilities that could simplify our event handling.

### Tier 4: High Effort, Future Consideration

#### 4.1 — JSONL Session Export/Import (0.61.0)
Export/import sessions via `/export` and `/import`. Could enable:
- Incident investigation audit exports
- Session migration between environments
- Debug replay of investigations

**Effort**: Medium — need to build UI and API endpoints for export/import.

#### 4.2 — `createLocalBashOperations()` (0.60.0)
Reusable bash backend for extensions. Could simplify our `createBashTool()` usage if we need multiple bash tool variants.

#### 4.3 — Session Forking (0.60.0)
`--fork <path|id>` for creating investigation branches. Could enable "what-if" analysis — fork an investigation to try a different remediation approach.

---

## Part 3: Implementation Plan

### Phase 1: Core Upgrade (1-2 hours)

**Goal**: Get to 0.63.1 with all tests passing.

1. **Update dependencies**:
   ```bash
   cd agent-worker
   npm install @mariozechner/pi-coding-agent@^0.63.1 \
               @mariozechner/pi-ai@^0.63.1 \
               @mariozechner/pi-agent-core@^0.63.1
   ```

2. **Check for `getApiKey()` usage** in `agent-runner.ts` and other files:
   - If found: update to `getApiKeyAndHeaders()` and destructure
   - If not found: no action needed

3. **Add `promptSnippet` to all custom tools** in `gateway-tools.ts`:
   - `gateway_call`: "Call MCP Gateway tools by name with optional instance hint"
   - `list_tool_types`: "List all available MCP tool types"
   - `list_tools_for_tool_type`: "List tools of a given type"
   - `get_tool_detail`: "Get JSON schema for a specific tool"
   - `execute_script`: "Run JavaScript in isolated sandbox"

4. **TypeScript compilation check**:
   ```bash
   npx tsc --noEmit
   ```

5. **Run tests**:
   ```bash
   make test-agent
   ```

6. **Docker build verification**:
   ```bash
   docker-compose build akmatori-agent
   ```

### Phase 2: Observability & Session Improvements (2-4 hours)

**Goal**: Leverage new features for better agent monitoring.

1. **Add context usage monitoring** — Use the new `contextUsage` RPC exposure to track context utilization per incident. Send metrics back via WebSocket.

2. **Handle `session_shutdown` event** — Clean up resources (temp files, gateway connections) when sessions end.

3. **Configure `sessionDir` setting** — Use `SettingsManager.inMemory()` to set session directory per incident:
   ```typescript
   settings.set("sessionDir", path.join(WORKSPACE_DIR, incidentId));
   ```

4. **Update event handler** — Add cases for any new event types not already handled.

### Phase 3: Enterprise Features (1-2 days, future)

1. **AWS Bedrock cost allocation** — Add `requestMetadata` for per-incident cost tracking.
2. **Custom headers per provider** — Leverage `getApiKeyAndHeaders()` for tracing headers.
3. **JSONL session export** — Build API endpoint to export investigation sessions for audit.
4. **Session forking** — Evaluate `--fork` for investigation branching.

---

## Part 4: Risk Assessment

| Risk | Likelihood | Impact | Mitigation |
|------|-----------|--------|------------|
| `promptSnippet` omission hides tools from agent | **HIGH** | Agent can't discover tools | Add `promptSnippet` to all custom tools in Phase 1 |
| `getApiKey()` call site exists | Low | Compile error | `tsc --noEmit` catches this immediately |
| Behavioral changes in compaction | Low | Investigation quality | Monitor first 5 investigations post-upgrade |
| Multi-edit changes agent behavior | Low | Unexpected file modifications | Agent already has safe coding tools; multi-edit is additive |
| `ajv` dependency conflict | Low | Build failure | Check for existing `ajv` versions in lockfile |
| Compaction message loss fix changes history | Low | Different token counts | 0.63.1 is a fix — should only improve behavior |

---

## Part 5: Files Requiring Changes

### Phase 1 (Core Upgrade)

| File | Change |
|------|--------|
| `agent-worker/package.json` | Bump all 3 pi packages to `^0.63.1` |
| `agent-worker/package-lock.json` | Regenerated by `npm install` |
| `agent-worker/src/gateway-tools.ts` | Add `promptSnippet` to all 5 custom tool definitions |
| `agent-worker/src/agent-runner.ts` | Update `getApiKey()` if used; verify event handling |

### Phase 2 (Observability)

| File | Change |
|------|--------|
| `agent-worker/src/agent-runner.ts` | Add `session_shutdown` handler, context usage monitoring |
| `agent-worker/src/orchestrator.ts` | Forward new metrics to API |

### Phase 3 (Enterprise)

| File | Change |
|------|--------|
| `agent-worker/src/agent-runner.ts` | `requestMetadata` for Bedrock, custom headers |
| `internal/handlers/` | New endpoints for session export |

---

## Appendix: Detailed Changelog (0.58.1 → 0.63.1)

### 0.58.2 - 0.58.4 (Patches)
- Bug fixes for steering messages, tool call batching

### 0.59.0 (2026-03-17)
- **BREAKING**: Tools without `promptSnippet` excluded from system prompt
- Lazy provider SDK loading — faster startup

### 0.60.0 (2026-03-18)
- **BREAKING**: Installed packages no longer auto-updated on startup
- `createLocalBashOperations()` export
- `--fork <path|id>` CLI flag for session forking

### 0.61.0 (2026-03-20)
- **BREAKING**: Keybinding IDs namespaced (`expandTools` → `app.tools.expand`)
- JSONL session export/import (`/export`, `/import`)
- Concurrent edit/write serialization
- `session.prompt()` waits for full retry loop
- `validateToolArguments()` graceful fallback

### 0.61.1 (2026-03-20)
- `ToolCallEventResult` typed exports

### 0.62.0 (2026-03-23)
- **BREAKING**: `renderCall`/`renderResult` semantics changed
- **BREAKING**: `sourceInfo` replaces `source`, `location`, `extensionPath` fields
- **BREAKING**: Removed `ResourceLoader.getPathMetadata()`
- Built-in tools as extensible `ToolDefinition` objects
- AWS Bedrock cost allocation tagging (`requestMetadata`)

### 0.63.0 (2026-03-27)
- **BREAKING**: `ModelRegistry.getApiKey(model)` → `getApiKeyAndHeaders(model)`
- **BREAKING**: Removed `minimax` and `minimax-cn` model IDs
- Multi-edit support (one tool call, multiple regions)
- `sessionDir` setting (configurable without CLI flag)
- `ajv` added as direct dependency
- `session_shutdown` event for resource cleanup
- RPC `contextUsage` exposure

### 0.63.1 (2026-03-27)
- Fixed repeated compaction message loss
