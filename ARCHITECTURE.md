# Architecture

How omnisearch is structured. For dependencies and runtime versions see [STACK.md](STACK.md). Legal terms: [DISCLAIMER.md](DISCLAIMER.md), [LICENSE](LICENSE). Security reporting: [SECURITY.md](SECURITY.md).

## What it is

omnisearch is a **single Rust binary** that exposes multi-provider search (and extract/research) to AI clients over:

1. **MCP stdio** (default) — Claude Desktop, Cursor, and other MCP clients
2. **MCP streamable HTTP** (optional) — local/private bind with bearer `AUTH_TOKENS`
3. **Bench** — one-shot provider latency/error check

It does **not** host search indexes. It fans out to third-party APIs (and a few keyless public endpoints), merges results, and returns them to the client. You supply keys; vendors do the retrieval.

## High-level flow

```
MCP client (stdio or HTTP)
        │
        ▼
   OmniServer (src/tools.rs)     ← tool schemas / MCP surface
        │
        ▼
   AppState (src/orchestrator.rs)
        │
        ├─ Config / keys / named accounts / gateways
        ├─ Provider Registry
        ├─ SearchCache
        └─ HealthBoard
        │
        ▼
   search / extract / research / ground
        │
        ├─ intent (auto / ladder / all)
        ├─ parallel provider fan-out (budgets, timeouts, cooldowns)
        ├─ RRF merge + dedupe + domain caps (src/merge.rs)
        └─ optional ground_top fetch (SSRF-gated)
        │
        ▼
   SearchResponse / ExtractResponse / …
```

## Crate layout (`src/`)

| Module | Role |
| --- | --- |
| `main.rs` | CLI: `stdio` (default), `http`, `bench` |
| `lib.rs` | Module tree + public re-exports |
| `tools.rs` | MCP `OmniServer` tool router (`search`, `research`, provider tools, health, …) |
| `orchestrator.rs` | `AppState`, parallel fan-out, budgets, cache, research/extract entrypoints |
| `config.rs` | Env-only configuration; secret key rings; named accounts; gateways |
| `accounts.rs` | Named account pools, rotation, failover, redacted `AccountHealth` |
| `providers/` | One adapter per engine + `Registry` + `Provider` trait |
| `providers/omniroute.rs` | OmniRoute search gateways (gateway credentials only) |
| `providers/mcp_backend.rs` | Optional remote MCP search backends |
| `merge.rs` | Reciprocal Rank Fusion, URL/title dedupe, spam hosts, domain diversity |
| `intent.rs` | `auto` / `ladder` provider selection from query + health |
| `cache.rs` | In-process TTL cache (partial runs optional) |
| `health.rs` | Per-provider cooldown / latency / error snapshots |
| `http.rs` / `http_server.rs` | Shared HTTP client; Axum + rmcp streamable HTTP server |
| `ssrf.rs` | Block loopback, link-local, private, metadata hosts on fetch/extract |
| `ground.rs` | Top-hit snippet grounding via safe fetch |
| `keys.rs` | Dual/triple env key rings (`NAME`, `NAME_2`, `NAME_3`) |
| `types.rs` | Request/response and provider ID types |
| `error.rs` | Internal errors → MCP error mapping (no secret echo) |
| `urlutil.rs` | URL/title normalization for merge |
| `bench.rs` | Provider bench rows |

## Provider model

Every engine implements `providers::Provider`:

- `search` → `SearchPage` of `SearchHit`
- optional `extract` / `crawl` / `map_urls`
- `is_configured`, cost estimate, notes, `known_account` for pin-by-name

`Registry` builds the live set from `Config`. Unconfigured providers are skipped. Named `OMNISEARCH_ACCOUNTS` replace legacy env keys for that provider when present (see `docs/accounts-and-gateways.md`).

## Search modes

| Mode | Behavior |
| --- | --- |
| `all` | Fan out to every selected/configured engine (default unless config changes it) |
| `auto` | Rank engines by query intent + recent health; may skip cooled-down or expensive ones |
| `ladder` | Prefer free/cheap engines first; can stop early once `evidence_min` is met |

Budgets (`budget_usd`, `max_providers`, timeouts) cap spend and wall time — they are **best-effort**, not a billing guarantee ([DISCLAIMER.md](DISCLAIMER.md) §3.3).

## Merge

Provider hit lists are fused with **reciprocal rank fusion**:

`score += 1 / (k + rank)` (`OMNISEARCH_RRF_K`, default 60)

Then: tracking-param strip, URL/title dedupe, shortener drop, per-domain caps, optional freshness filter and quality report.

## Transports

### Stdio MCP

`omnisearch` / `omnisearch stdio` → `OmniServer` over `rmcp` stdio. Preferred for desktop clients.

### HTTP MCP

`omnisearch http` → Axum middleware (bearer tokens + RPM) + rmcp streamable HTTP. Intended for localhost / private networks. Empty `AUTH_TOKENS` is dangerous; see [SECURITY.md](SECURITY.md).

## Security boundaries (design)

- Secrets live in env / process memory only; health and tool errors must stay redacted.
- Extract / ground / direct-fetch go through `ssrf::assert_public_http_url`.
- Account health exposes categories and latency — not upstream bodies that might contain tokens.
- CI and releases are **not** a security certification ([DISCLAIMER.md](DISCLAIMER.md)).

## Tests

| Path | Focus |
| --- | --- |
| `tests/accounts.rs` | Named accounts / pin / rotation |
| `tests/omniroute.rs` | Gateway wiring |
| `tests/parallel_search.rs` | Fan-out / merge behavior |
| `tests/protocol_smoke.py` | Local MCP stdio smoke (CI + release) |

Unit tests also live next to modules where practical. Prefer `wiremock` over live keys in CI.

## Extension points

1. **New provider** — implement `Provider` under `src/providers/`, register in `Registry`, document env / accounts in README + `.env.example`.
2. **New MCP tool** — add to `tools.rs`; keep schemas and README in sync.
3. **New config** — env parsing in `config.rs`; never log raw secrets.

## Related docs

- [STACK.md](STACK.md) — language, crates, CI
- [README.md](README.md) — user install and tool table
- [AGENTS.md](AGENTS.md) — coding-agent rules
- [DISCLAIMER.md](DISCLAIMER.md) · [LICENSE](LICENSE) · [SECURITY.md](SECURITY.md)
