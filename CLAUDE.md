# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What This Is

`kerygma_social` — the POSSE distribution and social platform automation package for ORGAN-VII (Kerygma). Provides platform clients for Mastodon, Discord, Bluesky, and Ghost, a `PosseDistributor` orchestrator, and a resilience stack (retry, circuit breaker, rate limiter).

## Package Structure

Source is in `kerygma_social/`, installed as the `kerygma_social` package:

### Platform Clients
| Module | Class | Protocol |
|--------|-------|----------|
| `mastodon.py` | `MastodonClient` | REST API via `urllib`. `MastodonConfig` dataclass. `Toot` dataclass for posts. `format_for_mastodon()` helper. |
| `discord.py` | `DiscordWebhook` | Webhook POST. `DiscordEmbed` dataclass for rich embeds. |
| `bluesky.py` | `BlueskyClient` | AT Protocol. `BlueskyConfig`/`BlueskyPost` dataclasses. Session-based auth. |
| `ghost.py` | `GhostClient` | Admin API with JWT auth (`HS256`, `{id}:{secret}` key format). `GhostConfig`/`GhostPost` dataclasses. Optional newsletter targeting. |

### Orchestration & Resilience
| Module | Purpose |
|--------|---------|
| `posse.py` | `PosseDistributor` — central dispatcher. `create_post()` → `syndicate()` flow. `_with_resilience()` wraps calls: rate limiter (outermost) → circuit breaker → retry (innermost). Deduplicates via `DeliveryLog`. Enums: `Platform`, `SyndicationStatus`. Dataclasses: `ContentPost`, `SyndicationRecord`. |
| `circuit_breaker.py` | Three-state machine: CLOSED → OPEN → HALF_OPEN. `CircuitBreakerConfig` (failure_threshold=5, reset_timeout=60s). Raises `CircuitOpenError` when open. |
| `retry.py` | Exponential backoff retry. `RetryConfig` dataclass. |
| `rate_limiter.py` | Token bucket rate limiter. `RateLimiterConfig` dataclass. `acquire(block=True)` blocks until token available. |
| `delivery_log.py` | `DeliveryLog` — persistent JSON log of dispatch attempts. `has_been_delivered(post_id, platform)` for deduplication. `DeliveryRecord` dataclass. |
| `rss_poller.py` | `RssPoller` — polls RSS/Atom feeds via stdlib `xml.etree` and `urllib`. Tracks seen entry IDs in JSON. `parse_feed()` handles both Atom and RSS 2.0. |

### Configuration
| Module | Purpose |
|--------|---------|
| `config.py` | `load_config(path)` → `SocialConfig` dataclass. YAML file + env var overrides (prefix `KERYGMA_`). All platform credentials, `live_mode`, `delivery_log_path`, `rss_feed_url`. |
| `cli.py` | CLI entry point (`social-dispatch`). |

## Development Commands

```bash
# Install (from superproject root or this directory)
pip install -e .[dev]

# Tests
pytest tests/ -v
pytest tests/test_posse.py::TestPosseDistributor::test_syndicate_mastodon -v

# Lint
ruff check kerygma_social/
```

## Key Design Details

- **All clients accept `live=False`** (default). In dry-run mode, API calls return mock responses. Never commit config with `live_mode: true`.
- **Resilience ordering matters**: rate limiter prevents burst → circuit breaker fails fast if service is down → retry handles transient errors. `CircuitOpenError` propagates immediately (not retried).
- **RSS poller is stdlib-only** — uses `urllib.request` and `xml.etree.ElementTree`. No `feedparser` dependency at the package level (feedparser is only needed by the pipeline orchestrator).
- **Delivery log uses atomic writes** — writes to `.tmp` then `os.replace()`.
- **Runtime dependency**: only `pyyaml>=6.0` (for config loading).

## Test Structure

Tests in `tests/` with `fixtures/` directory for test config files:
- `test_mastodon.py`, `test_discord.py`, `test_bluesky.py`, `test_ghost.py` — per-client formatting and dispatch
- `test_posse.py` — PosseDistributor syndication, deduplication, resilience
- `test_circuit_breaker.py` — state transitions, threshold, recovery
- `test_retry.py` — backoff behavior, max retries
- `test_rate_limiter.py` — token bucket, blocking acquire
- `test_delivery_log.py` — persistence, dedup checks
- `test_rss_poller.py` — Atom/RSS parsing, seen tracking
- `test_config.py` — YAML loading, env var overrides

<!-- ORGANVM:AUTO:START -->
## System Context (auto-generated — do not edit)

**Organ:** ORGAN-VII (Marketing) | **Tier:** standard | **Status:** GRADUATED
**Org:** `organvm-vii-kerygma` | **Repo:** `social-automation`

### Edges
- **Produces** → `ORGAN-IV`: delivery_log

### Siblings in Marketing
`announcement-templates`, `distribution-strategy`, `.github`, `kerygma-pipeline`, `kerygma-profiles`

### Governance
- *Standard ORGANVM governance applies*

*Last synced: 2026-05-23T00:26:31Z*

## Active Handoff Protocol

If `.conductor/active-handoff.md` exists, **READ IT FIRST** before doing any work.
It contains constraints, locked files, conventions, and completed work from the
originating agent. You MUST honor all constraints listed there.

If the handoff says "CROSS-VERIFICATION REQUIRED", your self-assessment will
NOT be trusted. A different agent will verify your output against these constraints.

## Session Review Protocol

At the end of each session that produces or modifies files:
1. Run `organvm session review --latest` to get a session summary
2. Check for unimplemented plans: `organvm session plans --project .`
3. Export significant sessions: `organvm session export <id> --slug <slug>`
4. Run `organvm prompts distill --dry-run` to detect uncovered operational patterns

Transcripts are on-demand (never committed):
- `organvm session transcript <id>` — conversation summary
- `organvm session transcript <id> --unabridged` — full audit trail
- `organvm session prompts <id>` — human prompts only


## System Library

Plans: 269 indexed | Chains: 5 available | SOPs: 8 active
Discover: `organvm plans search <query>` | `organvm chains list` | `organvm sop lifecycle`
Library: `/Users/4jp/Code/organvm/praxis-perpetua/library`


## Active Directives

| Scope | Phase | Name | Description |
|-------|-------|------|-------------|
| system | any | atomic-clock | The Atomic Clock |
| system | any | execution-sequence | Execution Sequence |
| system | any | multi-agent-dispatch | Multi-Agent Dispatch |
| system | any | session-handoff-avalanche | Session Handoff Avalanche |
| system | any | system-loops | System Loops |
| system | any | prompting-standards | Prompting Standards |
| system | any | background-task-resilience | background-task-resilience |
| system | any | context-window-conservation | context-window-conservation |
| system | any | session-self-critique | session-self-critique |
| system | any | the-descent-protocol | the-descent-protocol |
| system | any | the-membrane-protocol | the-membrane-protocol |
| system | any | theory-to-concrete-gate | theory-to-concrete-gate |
| system | any | triangulation-protocol | triangulation-protocol |

Linked skills: SOP-TRIADIC-REVIEW-PROTOCOL, cicd-resilience-and-recovery, continuous-learning-agent, evaluation-to-growth, genesis-dna, multi-agent-workforce-planner, promotion-and-state-transitions, quality-gate-baseline-calibration, repo-onboarding-and-habitat-creation, session-self-critique, structural-integrity-audit, the-membrane-protocol, triple-reference


**Prompting (Anthropic)**: context 200K tokens, format: XML tags, thinking: extended thinking (budget_tokens)


## Atomization Pipeline

Run `organvm atoms pipeline --write && organvm atoms fanout --write` to generate task queue.


## System Density (auto-generated)

AMMOI: 25% | Edges: 0 | Tensions: 0 | Clusters: 0 | Adv: 27 | Events(24h): 37975
Structure: 8 organs / 148 repos / 1654 components (depth 17) | Inference: 0% | Organs: META-ORGANVM:63%, ORGAN-I:53%, ORGAN-II:48%, ORGAN-III:54% +5 more
Last pulse: 2026-05-23T00:26:28 | Δ24h: n/a | Δ7d: n/a


## Dialect Identity (Trivium)

**Dialect:** SIGNAL_PROPAGATION | **Classical Parallel:** Astronomy | **Translation Role:** The Broadcast — structure-preserving projection to external

Strongest translations: III (structural), VI (analogical), I (analogical)

Scan: `organvm trivium scan VII <OTHER>` | Matrix: `organvm trivium matrix` | Synthesize: `organvm trivium synthesize`


## Logos Documentation Layer

**Status:** ACTIVE | **Symmetry:** 0.5 (DREAM)

Nature demands a documentation counterpart. This formation maintains its narrative record in `docs/logos/`.

### The Tetradic Counterpart
- **[Telos (Idealized Form)](../docs/logos/telos.md)** — The dream and theoretical grounding.
- **[Pragma (Concrete State)](../docs/logos/pragma.md)** — The honest account of what exists.
- **[Praxis (Remediation Plan)](../docs/logos/praxis.md)** — The attack vectors for evolution.
- **[Receptio (Reception)](../docs/logos/receptio.md)** — The account of the constructed polis.

### Alchemical I/O
- **[Source & Transmutation](../docs/logos/alchemical-io.md)** — Narrative of inputs, process, and returns.



*Compliance: Record exists without implementation.*

<!-- ORGANVM:AUTO:END -->












## ⚡ Conductor OS Integration
This repository is a managed component of the ORGANVM meta-workspace.
- **Orchestration:** Use `conductor patch` for system status and work queue.
- **Lifecycle:** Follow the `FRAME -> SHAPE -> BUILD -> PROVE` workflow.
- **Governance:** Promotions are managed via `conductor wip promote`.
- **Intelligence:** Conductor MCP tools are available for routing and mission synthesis.