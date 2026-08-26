---
name: rhombus-mind
description: >
  Interact with Rhombus MIND — the natural-language AI assistant built into the
  Rhombus CLI via `rhombus chat` (text) and `rhombus voice` (spoken). Use when
  the user wants to "ask MIND", says "chat with Rhombus AI", wants to query the
  Rhombus platform with natural language, asks about voice-driven security
  operations, wants a hands-free way to check cameras or alerts, or asks about
  whisper-cpp, sox, or speech-to-text in a Rhombus context. Does NOT cover the
  auto-generated CLI (see rhombus-cli) or deployment-context commands (see
  rhombus-deployment-context).
argument-hint: "[chat|voice] [question]"
allowed-tools: Read, Bash
---

# Rhombus MIND (chat + voice)

Rhombus MIND is a natural-language interface to the platform. It's an interactive Claude-powered agent that has tool access to the entire `rhombus` CLI — so anything the CLI can do, MIND can do via conversation.

## `rhombus chat` — text interface

```bash
rhombus chat
```

Opens an interactive session. Type natural-language queries:

- "Show me all motion alerts from the last 30 minutes at the Warehouse."
- "Which doors are currently unlocked?"
- "Has anyone badged into the lab after hours this week?"
- "Download a clip of the latest parking lot alert."

MIND dispatches the appropriate CLI commands locally, interprets results, and produces a conversational summary. No setup required beyond an authenticated CLI.

## `rhombus voice` — spoken interface

```bash
rhombus voice
rhombus voice --model medium   # Options: tiny, base, small (default), medium, large
```

Records audio with `sox`, transcribes with `whisper-cpp`, sends to MIND, speaks the response back.

**Dependencies:**

- `sox` — audio recording (`brew install sox` on macOS)
- `whisper-cpp` — speech-to-text (`brew install whisper-cpp`)

Models auto-download to `~/.rhombus/models/` on first use. Smaller models are faster but less accurate on noisy audio; start with `small` and bump to `medium` if you see transcription errors.

## When to reach for MIND vs. direct CLI

| Situation | Prefer |
|---|---|
| Exploratory / "I don't know exactly what to ask" | `rhombus chat` |
| Hands-free or reviewing on the go | `rhombus voice` |
| Scripting or automation | Direct CLI |
| Known API operation | Direct CLI (`rhombus <service> <op>`) |
| Claude Code session already active | Just ask Claude — this plugin gives it CLI access |

## MIND vs. this Claude Code plugin

If the user is already in a Claude Code session with this plugin enabled, Claude itself can do everything MIND does — including running `rhombus` commands via Bash. Use `rhombus chat` / `voice` when:

- The user is on a machine without Claude Code installed.
- They want a standalone terminal/voice interface that doesn't require an IDE.
- They specifically want the MIND persona and workflow.

## Troubleshooting

| Issue | Fix |
|---|---|
| `chat` says "not authenticated" | Run `rhombus login` first |
| `voice` hangs at startup | Check microphone permissions for the terminal |
| Transcription quality poor | Try `--model medium` or `--model large` |
| MIND gives wrong answers for your org | Confirm `--profile` matches the org you care about |
