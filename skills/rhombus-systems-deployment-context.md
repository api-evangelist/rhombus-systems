---
name: rhombus-deployment-context
description: >
  Generate, query, and analyze Rhombus deployment context — a cached snapshot of
  every location, camera, and still image across an organization — and use that
  context to review footage. Covers the `rhombus context`, `rhombus analyze`,
  and `rhombus stitch` commands. Use when the user asks about deployment
  context, camera snapshots, "what are my cameras seeing", wants to generate
  or refresh a site overview (`index.md`), asks Claude to analyze alert
  footage or a time window on a camera, wants to stitch multi-camera clips
  into a single review video, mentions incident review, wants to "pull video
  from last night", or asks to understand motion/activity in a specific time
  range. Also trigger for `--lan` mode questions and anything involving
  `~/.rhombus/context/`.
argument-hint: "[context|analyze|stitch] [args]"
allowed-tools: Read, Grep, Glob, Bash
---

# Rhombus Deployment Context + Footage Analysis

Three CLI commands that work together to give Claude a working knowledge of a Rhombus deployment and let it reason over footage.

## `rhombus context` — build a snapshot

Generates a local cache of locations, cameras, hardware, and per-camera stills. Claude reads the cache to understand what each camera covers without re-calling the API.

```bash
# Generate full deployment context (stills + index)
rhombus context generate
rhombus context generate --lan            # Use LAN for faster still downloads
rhombus context generate --concurrency 8  # Parallel downloads

# Query cached context
rhombus context location "Main Office"   # Location details + camera list
rhombus context camera "Front Door"      # Fresh still + camera details + recent activity
```

**Output files** (in `~/.rhombus/context/<profile>/`):

- `index.md` — Compact deployment reference. Read this first when the user's question needs site-wide awareness.
- `manifest.json` — Full machine-readable manifest.
- `stills/<camera>.jpeg` — One still per camera.

**Usage pattern:** When the user asks about their deployment, `Read` `~/.rhombus/context/<profile>/index.md`. When they ask about a specific camera, `Read` the matching still and any notes in the index.

## `rhombus analyze` — describe what happened

Extracts and analyzes frames from alert clips or camera footage over a time window.

```bash
# Analyze an alert — extracts frames and describes what happened
rhombus analyze alert "ALERT_UUID"

# Analyze footage from a camera over a time window
rhombus analyze footage "front lobby" --period "yesterday between 8am and 9am"

# Analyze footage from all cameras at a location
rhombus analyze footage --location "Main Office" --start 1700000000000 --end 1700003600000

# LAN mode — download frames directly from cameras (faster on local network)
rhombus analyze footage --location "Office" --period "last hour" --lan

# Include motion seekpoints (default: only human/vehicle/object activity)
rhombus analyze footage "parking lot" --period "today" --include-motion

# Include evenly-spaced fill frames (not just activity frames)
rhombus analyze footage "parking lot" --period "today between 6am and 7am" --fill

# Raw mode — output frames + manifest for external analysis (skip visual analysis)
rhombus analyze alert "ALERT_UUID" --raw --output /tmp/frames
rhombus analyze footage "lobby" --period "last hour" --raw
```

**When to use:** User asks "what happened…", "show me the activity at…", "describe the alert…". Prefer this over raw clip download when the user wants an answer, not the file.

## `rhombus stitch` — chronological multi-camera review

Downloads clips for detected events and stitches them into a single chronological video. Concurrent events from multiple cameras are shown in a grid with timestamp overlays. Uses LAN streaming when available.

```bash
# Stitch events from specific cameras over a time period
rhombus stitch --camera "front lobby,parking lot" --period "yesterday between 6am and 7am"

# Stitch all events at a location
rhombus stitch --location "Main Office" --period "last night between 10pm and 6am"

# Stitch with start/end times and custom buffer
rhombus stitch --camera "entrance" --start 1700000000000 --end 1700003600000 --buffer 10

# Include motion seekpoints (default: only activity)
rhombus stitch --location "Office" --period "today" --include-motion

# Save to a specific file
rhombus stitch --location "Warehouse" --period "today between 8am and noon" --output incident-review.mp4
```

**When to use:** User wants a single video they can watch later — incident review, shift-handover summaries, insurance submissions. Prefer `analyze` when they just want an answer.

## Typical flow for an incident review

1. `rhombus context generate` (if stale) so Claude knows the deployment.
2. `rhombus analyze alert <UUID>` — first pass for the triggering event.
3. If the answer needs wider context: `rhombus analyze footage --location <site> --period "around <time>"` for co-occurring activity.
4. For a shareable artifact: `rhombus stitch --location <site> --period "<window>" --output review.mp4`.

## Slash commands

This plugin ships three slash commands that wrap the above:

- `/rhombus-analyze <alert-uuid | camera time-range>` — fires `analyze`.
- `/rhombus-context-refresh` — regenerates the full context (slash-only; writes to disk).
- `/rhombus-watch <camera>` — opens live footage in the browser (slash-only; browser side effect).

## Tips

- **First context generation is slow** (one still per camera, downloaded over the internet). Use `--lan` if you're on the same network as the cameras.
- **Keep the context fresh.** Regenerate after major deployment changes (new cameras, reassigned locations). The `index.md` footer includes the generation timestamp.
- **`--raw` mode** is for when you want to pipe frames to a separate tool (your own model, a labeling flow) rather than have Claude describe them.
