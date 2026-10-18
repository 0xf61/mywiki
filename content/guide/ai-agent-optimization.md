---
title: "Free Models at Full Quality: An OmniRoute Setup for AI Coding Agents"
draft: false
date: 2026-10-18
tags:
  - ai
  - ai-agents
  - opencode
  - omniroute
  - token-optimization
description: "Free coding models got good. The catch is no longer quality. It is quota and context window. A free tier dies mid-session. Tool spam fills the context. The model hallucinates APIs it barely knows."
---

Free coding models got good. The catch is no longer quality. It is **quota and context window**. A free tier dies mid-session. Tool spam fills the context. The model hallucinates APIs it barely knows.

None of these are model problems. They are plumbing problems. This guide is the plumbing:

- **OmniRoute** — combine providers, fall back automatically, compress tool output in-flight.
- **ccusage** — measure what you actually burn.
- **RTK** — cut shell output before it enters the session.
- **Context7 + Serena** — the real quality levers.

## TL;DR — full setup

```bash
# 1. Router (any OS with Node) — serves on http://localhost:20128
npm install -g omniroute
omniroute setup                 # first run: prompts for the dashboard admin password
#   → dashboard at http://localhost:20128 — log in, then Keys → create an API key
#     and point agents at http://localhost:20128/v1 with that key as Bearer token
# 2. Measurement
npm install -g ccusage
ccusage opencode daily        # OpenCode; plain `ccusage daily` rolls up every agent it detects
# 3. Input compression (single Rust binary, no deps)
curl -fsSL https://raw.githubusercontent.com/rtk-ai/rtk/refs/heads/master/install.sh | sh
rtk init -g                   # global hooks for most CLI agents
# per-agent (default target is Claude Code)
rtk init --opencode          # or --agent cursor|windsurf|cline|kimi|pi|hermes|omp
# 4. Capability MCPs (keyless — the endpoint needs no API key)
opencode mcp add context7 --global --url https://mcp.context7.com/mcp   # OpenCode
npx ctx7 setup --oauth        # other agents: Claude Code, Cursor, VS Code, Codex, …
# pi — same URL pattern for any agent that supports remote MCP servers
pi mcp add context7 --url https://mcp.context7.com/mcp
# serena (symbol-level code tools):
uv tool install -p 3.13 serena-agent && serena init
opencode mcp add serena --global -- serena start-mcp-server --context=ide --project-from-cwd
pi mcp add serena -- serena start-mcp-server --context=ide --project-from-cwd
```

## 1. OmniRoute: the router is the strategy

A local LLM router sits between your agent and every provider. It gives you three things free tiers cannot:

1. **Model combination** — one OpenAI-compatible endpoint, hundreds of providers behind it.
2. **Fallback** — a free tier throws "limit exceeded" mid-task, the router moves to the next healthy provider. Quota-aware by design: connections that report exhaustion are weighted out of the pool until they recover.
3. **Quota orchestration** — per-connection quota snapshots and budget schedules in the dashboard, instead of guessing which free tier is still alive.

Free providers misbehave in ways that make this matter: soft rate-limit bursts, 429 storms on shared keys, per-model daily caps. With a fallback chain you stop caring which one is throttled right now.

OmniRoute starts safer than most local routers: loopback bind by default, a random storage-encryption key in `~/.omniroute/.env` from first boot, dashboard login enforced. Two things to do before anything else:

- **Run `omniroute setup`, log in, mint an API key** (Dashboard → Keys), point agents at `http://localhost:20128/v1` with the key as Bearer token. Setup prompts for the admin password; the `INITIAL_PASSWORD` env fallback ships as `CHANGEME`, and `REQUIRE_API_KEY` ships as `false` — fine on loopback, but set it to `true` before any wider bind.
- **Know what the installer starts.** The npm package installs a background service (on macOS a LaunchAgent, `com.omniroute.gateway`) that owns port 20128. If another router already listens there, one of them loses the port silently. One router per port. Stop it with `launchctl bootout gui/$(id -u)/com.omniroute.gateway`.

Until traffic actually flows through the router, none of its compression or fallback does anything. This is the step people skip — then wonder why nothing improved.

**Home-server variant (Raspberry Pi 5, arm64).** The official image runs on small machines — a 446 MB download, ~1.4 GB on disk, and a few hundred MB of RAM at idle (it is a full Next.js app). The redis sidecar exists only as the rate-limiter backend; npm installs run without any redis. Everything in one file — the secrets are generated the moment you create it, and `REQUIRE_API_KEY=true` is set on purpose (the shipped default is `false`):

```bash
cat > docker-compose.yml <<EOF
services:
  omniroute:
    image: ghcr.io/diegosouzapw/omniroute:latest
    restart: unless-stopped
    environment:
      - JWT_SECRET=$(openssl rand -base64 48)
      - API_KEY_SECRET=$(openssl rand -hex 32)
      - INITIAL_PASSWORD=$(openssl rand -hex 12)
      - REQUIRE_API_KEY=true
      - DATA_DIR=/app/data
      - REDIS_URL=redis://redis:6379
    ports: ["127.0.0.1:20128:20128"]
    volumes: [./data:/app/data]
    depends_on: [redis]
  redis:
    image: docker.io/library/redis:8.6.5-alpine
    restart: unless-stopped
    command: redis-server --save 60 1 --loglevel warning
    volumes: [redis-data:/data]
volumes:
  redis-data:
EOF
# dashboard login password: grep INITIAL_PASSWORD docker-compose.yml
docker compose up -d
```

Agents on other machines reach it through an SSH tunnel or your reverse proxy. Open the bind to the LAN only while `REQUIRE_API_KEY=true` stays on.

One honest caveat on "keyless": a fresh install answers `/v1/models` with hundreds of entries and can route through keyless providers — but some keyless routes are gated. OpenCode's free tier, for one, only answers requests that originate inside OpenCode, and the router stops on that refusal instead of trying the next pool member. Add one real provider key before you depend on it.

## 2. Measure first: ccusage

Every decision below is "does this save quota/context?" Unmeasurable opinions are worthless. ccusage reads local agent session data. Yes, OmniRoute's dashboard has its own usage view — but it only sees traffic that flowed through `/v1`; ccusage counts everything, including sessions that bypassed the router:

```bash
ccusage opencode daily
ccusage opencode session
```

One caveat on free models: ccusage prices tokens with its **paid** rate table. The dollars on screen are fiction — your real spend is $0. Read the **token counts and per-model breakdown**. That is what you are optimizing.

Run it before and after each change, on comparable work.

## 3. Input side: RTK cuts tool output at the source

Most agent context is tool output: `ls` walls, full git logs, 400-line test runs. RTK is a CLI proxy that rewrites these to token-optimal equivalents **before they enter the session history**:

```bash
curl -fsSL https://raw.githubusercontent.com/rtk-ai/rtk/refs/heads/master/install.sh | sh
# macOS alternative: brew install rtk   (mind the name collision: you want rtk-ai/rtk)
rtk init -g
```

It wraps the usual suspects — `git`, `ls`, `find`, `diff`, `log`, `docker`, `test` (failures only), `err` (errors only) — and the hook rewrites your agent's shell commands transparently. `rtk bypass <cmd>` runs something unfiltered when you need raw output. Agent hooks are built in for OpenCode, Claude Code, Cursor, Windsurf, Cline, Kimi, pi, hermes, omp and more (see the TL;DR for per-agent syntax).

Know exactly what it covers:

- RTK sees **only shell/Bash tool calls**. Built-in Read/Grep/Glob tools bypass it. The next section covers that gap.
- What it saves, it saves **at the source** — the shrunken output lands in session history, so the agent's context stays small.

**Honest numbers:** a JetBrains experiment measured no significant savings (cost even went *up* at low effort); on my real workload it was ~0.5% of spend. Marketing says "90%"; treat that as a ceiling nobody hits. Verify on your machine:

```bash
rtk gain --graph --daily   # measured savings
rtk discover               # commands RTK could have wrapped but didn't
```

If `rtk gain` says ~0 after a week, uninstall it. It earns its place only with receipts.

## 4. Proxy side + output side: what OmniRoute does for free

Once traffic flows through OmniRoute, the compression pipeline covers what RTK never sees. Configure in **Dashboard → Compression**, or with `omniroute compression` on the CLI:

**RTK engine** (in the default pipeline). Compresses *every* `tool_result` in-flight to the model — including built-in Read/Grep/Glob output the shell-side RTK cannot see. Complement, not duplicate: RTK shrinks shell output at the source and your session history; the proxy engine shrinks everything else, in the request only. Per-request opt-out header: `x-omniroute-compression: off`.

**Caveman engine** — the output shaper, built in (not to be confused with the Lite and Ultra engines: those are separate context-pruning stages in the 12-engine stack):

- Terse answers, fewer words per reply.
- **Never standalone.** A standalone copy next to the built-in engine means two conflicting "be brief" system prompts.
- **At most one style engine at a time.** Two shapers buy nothing twice.

On a *free* model these save $0 by definition. What they actually buy: context and quota relief, plus output style. Independent tests come out an order of magnitude below vendor claims — one benchmark measured a terse shaper *increasing* total tokens by 7%. Enable one only if verbose output genuinely bothers you, then compare with ccusage.

**LLMLingua** — local ML compression (a small ONNX model). The only engine that rewrites meaning instead of formatting; costs RAM and latency. Off unless you measure a win.

**MCP accessibility guard** — compression skips MCP tool payloads by default, so capability MCPs (next section) keep working. If an MCP tool misbehaves, check this before blaming the tool.

## 5. The actual quality lever: capability MCPs

Token savers move single-digit percentages. **Capability tools move answer quality by whole grades** — especially on small free models.

**Context7** — up-to-date library docs in context. Kills the classic free-model failure of calling an API that changed two versions ago:

```bash
# OpenCode — one command, connects keyless immediately:
opencode mcp add context7 --global --url https://mcp.context7.com/mcp
# Other agents — interactive wizard (Claude Code, Cursor, VS Code, Codex, Gemini CLI, …):
npx ctx7 setup --oauth
```

No API key anywhere: the remote endpoint answers without one, and `--oauth` (one-time browser login) skips the key prompt that plain `ctx7 setup` shows. The wizard also installs a usage rule + skill that teach the agent to actually call Context7; the one-liner doesn't — if you go manual, add one line to your agent instructions: *prefer Context7 for library/framework docs*.

Any other agent that speaks remote (streamable-HTTP) MCP servers takes the same URL. pi, for instance:

```bash
pi mcp add context7 --url https://mcp.context7.com/mcp
pi mcp list     # → context7: connected, 2 tools (resolve-library-id, query-docs)
```

**Serena** — symbol-level code reading/editing via LSP (40+ languages). Breaks the "grep, then read the whole file anyway" loop that burns most of a free tier's context:

```bash
uv tool install -p 3.13 serena-agent && serena init
# OpenCode:
opencode mcp add serena --global -- serena start-mcp-server --context=ide --project-from-cwd
# pi:
pi mcp add serena -- serena start-mcp-server --context=ide --project-from-cwd
pi mcp list      # → serena: connected, 3 tools
```

Serena starts with only three tools (`activate_project`, `initial_instructions`, `serena_repl`) — the rest appear after your agent activates the project, which it will ask to do on first use. Any other MCP-capable agent can use the same `serena start-mcp-server` command over stdio. If `serena` isn't on PATH, give the full path.

## 6. The workflow, in order

| Problem | Lever | Check |
|---|---|---|
| Free tier dies mid-task | OmniRoute quota-aware fallback | provider switch visible in dashboard |
| "How much am I burning?" | `ccusage opencode daily` | token counts, not dollars |
| Context full of tool spam | RTK (shell) + OmniRoute compression (rest) | `rtk gain`, ccusage before/after |
| Verbose / over-engineered output | Caveman engine (one, measured) | ccusage before/after |
| Wrong APIs, hallucinated docs | Context7 | — |
| Reading whole files for one symbol | Serena | — |

Why each rule exists:

- **One copy of a saver.** Two copies inject conflicting prompts and save nothing twice.
- **One change at a time, measured.** Change three things and ccusage can't tell you which one worked.
- **Distrust vendor percentages.** Independent tests come out an order of magnitude lower.
- **Check the router first when nothing improves.** Savers and fallback only exist for traffic that actually passes through `localhost:20128`.

The order that worked for me: **router up and secured → agent pointed at `/v1` with a key → ccusage baseline → RTK → verify with `rtk gain` → Context7 + Serena → consider one style engine, measured.** The savers are trimming. The router plus the MCPs are the actual upgrade.
