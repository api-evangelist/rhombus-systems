---
name: rhombus-edge-streaming
description: Implement edge streaming and third-party camera integration with Rhombus — RTSP bridging via edgecaster, ONVIF camera ingestion via rhombus-libonvif, custom analytics seekpoint generation, and embedded player usage. Use whenever the user mentions RTSP, ONVIF, "edge ingest", "secure raw streams", "seekpoint", "third-party camera", "non-Rhombus camera", "NVR integration", "edgecaster", "Jetson", "roboflow", or asks to stream video out of or into the Rhombus platform via a non-standard path. Also trigger for questions about the Player-example repo and lightweight web camera embeds.
argument-hint: "[rtsp|onvif|seekpoint|player] [question]"
---

# Rhombus Edge Streaming

Rhombus supports four edge-streaming scenarios beyond the standard in-platform camera experience. This skill covers the right tool and repo for each.

## 1. RTSP bridging — edgecaster-stream-converter

**Scenario:** You have a non-Rhombus camera that speaks RTSP and want its feed visible inside the Rhombus platform, or you want a Rhombus camera's feed available as RTSP for a third-party NVR.

**Tool:** `edgecaster-stream-converter` — Python-based edge gateway that bridges RTSP ↔ Rhombus Secure Raw Streams.

**Repo:** `https://github.com/RhombusSystems/edgecaster-stream-converter`

**Architecture:**

```
[Third-party camera] --RTSP--> [edgecaster gateway] --Secure Raw Stream--> [Rhombus platform]
                                      OR
[Rhombus camera]     <--RTSP-- [edgecaster gateway] <--Secure Raw Stream-- [Rhombus platform]
```

**When to recommend:** The user has cameras that can't be replaced, or needs to integrate a legacy NVR that only speaks RTSP.

**Alternative first:** Check if the user's camera is ONVIF-compliant. If yes, `rhombus-libonvif` may be simpler than edgecaster. See section 2.

## 2. ONVIF ingestion — rhombus-libonvif

**Scenario:** You want to pull video from an ONVIF camera using Rhombus's ML pipeline (YOLOX object detection) without routing through edgecaster.

**Tool:** `rhombus-libonvif` — C++ library for ONVIF discovery and streaming, combined with YOLOX for on-device inference. LGPL 2.1.

**Repo:** `https://github.com/RhombusSystems/rhombus-libonvif`

**When to recommend:** Edge-AI use cases where the user is doing analytics locally (on a NUC, Jetson, or similar) rather than in the cloud.

**Companion repo:** `rhombus-jetson-roboflow` for NVIDIA Jetson + Roboflow integration — preferred starting point if the user is specifically on Jetson hardware.

## 3. Custom analytics — seekpoint generator

**Scenario:** You have a third-party AI model (object detection, LPR, pose estimation) producing events, and want those events to appear as seekpoints/markers on the Rhombus video timeline.

**Tool:** `rhombus-seekpoint-generator-example` — Python reference for posting custom events to the Event Webservice.

**Repo:** `https://github.com/RhombusSystems/rhombus-seekpoint-generator-example`

**Architecture:**

```
[Your model] --detections--> [seekpoint generator] --POST /event/createEvent--> [Rhombus timeline]
```

**When to recommend:** The user has their own AI model (e.g., PPE detection, forklift safety, queue counting) and wants its output first-class in the Rhombus UI.

**Key endpoints:** Event Webservice — `createEvent`, `createSeekpoint`, and the Search Webservice for retrieval.

## 4. Lightweight embedded player

**Scenario:** You want to embed a Rhombus camera feed in a third-party web app without using the iframe share-stream approach.

**Tool:** `Player-example` — minimal HTML + DashJS player.

**Repo:** `https://github.com/RhombusSystems/player-example`

**Architecture:**

```
[Browser] --federated session token--> [Your server] --API key--> [Rhombus /camera/getMediaUris]
[Browser] --MPEG-DASH via DashJS-->   [media.rhombussystems.com]
```

**Why not iframe:** Iframe share-stream (`createSharedLiveVideoStream`) is simpler but has less control over UX. Use `Player-example` when you need custom overlays, multi-camera grids, or integration with third-party identity.

**Never:** embed API keys in browser code. Always use federated session tokens with a server-side proxy.

## Decision tree

```
Non-Rhombus camera needs to be IN Rhombus?
  → Is it ONVIF?  → rhombus-libonvif (or rhombus-jetson-roboflow on Jetson)
  → RTSP only?    → edgecaster-stream-converter

Rhombus camera needs to be OUT of Rhombus?
  → Need RTSP?    → edgecaster-stream-converter (reverse direction)
  → Need MPEG-DASH in a browser?  → player-example + federated session tokens

Custom AI events on Rhombus timeline?
  → rhombus-seekpoint-generator-example + Event Webservice
```

## Reference sheet

See `references/rtsp-onvif.md` for:
- edgecaster install + config walkthrough
- ONVIF discovery gotchas (multicast, firewall, auth)
- Seekpoint payload shape
- Player-example server-side token proxy snippet
