---
name: oauth-disguise
version: 1.2.0
description: Configure Anthropic OAuth tokens (sk-ant-oat01-*) to work with Hermes Agent as API keys via ANTHROPIC_API_KEY. Use when: (1) user has OAuth tokens that fail direct API calls with authentication_error, (2) switching Hermes from proxy/custom endpoint to the Anthropic native provider, (3) configuring Claude subscription tokens for API use in Hermes. NOT for: standard API keys (sk-ant-api03-*), working proxy services, or non-Anthropic providers.
---

# OAuth Token Disguise for Hermes

Configure Anthropic OAuth tokens (`sk-ant-oat01-*`) as working API keys through Hermes Agent's native Anthropic provider.

Hermes' Anthropic provider speaks the native Anthropic Messages protocol and reads its credential from `ANTHROPIC_API_KEY`. The token works when presented this way even though direct API calls with it fail elsewhere.

## Core Method

Put the token in `~/.hermes/.env` (secrets live here, never in `config.yaml`):

```bash
hermes config set ANTHROPIC_API_KEY sk-ant-oat01-YOUR_TOKEN
```

(The CLI routes secrets to `~/.hermes/.env` automatically. Or edit the file directly: add a line `ANTHROPIC_API_KEY=sk-ant-oat01-YOUR_TOKEN`.)

Then pin the provider and model in `~/.hermes/config.yaml`:

```yaml
model:
  provider: "anthropic"
  default: "claude-sonnet-4-20250514"
```

Exit the current session completely and start `hermes` again so the new environment is picked up.

Interactive alternative: run `hermes model` (in the terminal, outside any session) and walk through the Anthropic provider setup.

## Switching Models

- `hermes model` — full setup wizard, terminal only (use when adding a provider for the first time)
- `/model` — inside a chat session, quick switch between already-configured providers/models

## Verification

Start `hermes` and check the startup banner, or ask which model is active. Look for:

- Model shows `anthropic/claude-*` -> success
- Falls back to another provider, or an auth error on start -> token invalid or expired

Run `hermes doctor` for diagnostics — it reports exactly which provider config is missing or broken.

## Troubleshooting

| Symptom | Cause | Fix |
|---------|-------|-----|
| Falls back to another provider | `model.provider` not set to `anthropic` | Set `provider: "anthropic"` under `model:` in `~/.hermes/config.yaml` |
| Auth error on start | Token expired or revoked | Replace the token in `~/.hermes/.env`, restart the session fully |
| Still uses the old key | Session started before the env change | Exit completely (`/quit` or Ctrl+C) and start `hermes` again |
| `API key not set` | Secret never landed in `.env` | Re-run `hermes config set ANTHROPIC_API_KEY ...`, verify with `hermes doctor` |

**Critical**: Do not rapidly switch between tokens. Anthropic applies auth cooldowns after repeated failures — wait a few minutes before retrying.

## Dual Provider Setup

Keep a fallback provider (e.g. OpenRouter) while using the official API:

```yaml
model:
  provider: "anthropic"
  default: "claude-sonnet-4-20250514"

fallback_providers:
  - provider: openrouter
    model: anthropic/claude-sonnet
```

(Requires `OPENROUTER_API_KEY` in `~/.hermes/.env`. If the primary provider goes down, Hermes switches automatically.)

## Native OAuth Alternative

Hermes also offers official Anthropic OAuth via `hermes model` -> Anthropic OAuth. Heads-up on plan semantics, from the official docs:

- Only works on a Claude **Max** plan **with purchased extra usage credits** — Hermes usage bills as extra/overage credits, the base Max allowance is never consumed.
- Claude **Pro** subscribers cannot use this path.

If you don't have Max + extra credits, the token-injection method in this skill is the way to go.

## Security

- Secrets live in `~/.hermes/.env` — never put raw keys in `config.yaml`
- Never commit `.env` or any token to version control
- `hermes config get` redacts secrets in its output
- When a token expires, replace it in `.env` and restart the session; no config rewrite needed

## Changelog

- 1.2.0: Rewritten for Hermes Agent (was OpenClaw). Config via `~/.hermes/.env` + `config.yaml`; added native OAuth alternative notes and `fallback_providers` dual setup.
- 1.1.x: OpenClaw `env.vars` injection method.
