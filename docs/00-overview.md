# Unity Photon Multiplayer SDK — overview

## What this is

A reusable Unity toolkit built on top of **Photon Fusion** and **Photon
Voice**, targeted at PC VR projects but not limited to VR. It does not
replace Fusion or Voice — it's a thinner, more consistent layer on top of
them, so a project doesn't reimplement the same multiplayer boilerplate
every time.

## Tech stack

- **Unity** — target engine, editor tooling built with `EditorWindow` /
  `UnityEditor` APIs
- **Photon Fusion** — underlying networking (NetworkRunner, NetworkObject,
  RPC attributes, networked properties)
- **Photon Voice** — underlying voice (VoiceConnection, Recorder, Speaker)
- **ScriptableObjects** — all developer-facing configuration
- **C# async/await (UniTask or native `Task`)** — public API shape uses
  async methods (`await Sdk.Session.CreateAsync(...)`) instead of
  callback-only patterns

## Problem statement

Setting up Photon multiplayer manually requires a developer to configure
App IDs, create and manage a `NetworkRunner`, implement Photon's callback
interfaces, handle session create/join/leave and their error states,
prepare the player prefab with the right networking components, register
it for spawning, write matchmaking retry logic by hand, write RPC methods
per-project, and separately wire up Photon Voice's Recorder/Speaker
lifecycle and mute state. Every one of these steps is repeated, project
after project, and each one is easy to get subtly wrong.

## How it's solved

The SDK centralizes all of this behind five developer-facing systems —
**Session, Player, Matchmaking, RPC, Voice** — backed by a **Core** module
that handles initialization order and validation, and an **Editor
Tooling** module that gives the developer one setup surface instead of
scattered assets and manual component wiring. See `01-architecture.md`
for how these pieces are layered.

## Goals

- Reduce Photon multiplayer setup time
- Centralize configuration in one place, one source of truth per setting
- Validate project, scene, and prefab setup before problems reach runtime
- Apply safe automatic fixes where possible, never destructive ones
  silently
- Keep advanced Photon APIs reachable for cases the SDK doesn't cover
- Stay game-genre-agnostic — not tied to VR, PC, mobile, or a specific
  game type
- Keep game UI and gameplay logic strictly outside the SDK's
  responsibility

## Non-goals (out of scope for this SDK)

- Authentication (PlayFab or otherwise)
- Player inventory, currency, or progression systems
- Friends or party systems
- Skill-based backend matchmaking (this SDK uses Photon's own session
  matchmaking, not a custom queue)
- Game-specific lobby UI
- Player models, visual design, or animation
- Game rules, health, score, or weapon systems
- VR-specific head/hand tracking synchronization
- Text chat
- Custom backend server allocation
- A full replacement for the Photon Fusion API surface

## Document index

| Doc | Contents |
|---|---|
| `01-architecture.md` | Layered architecture, module scope, source-of-truth and production rules |
| `02-module-reference.md` | Every module — problem it solves, developer inputs, config, internal behavior, public API, validation |
| `03-editor-tooling.md` | Setup Window, tabs, prefab setup flow, validation dashboard, detection rules |
| `04-editor-workflow.md` | Full edit-time developer workflow, step by step |
| `05-runtime-flow.md` | Full play-time flow, step by step |
| `06-validation-and-errors.md` | Validation categories, result structure, and the full error message catalog |
| `07-project-structure.md` | Recommended folder layout and where every file type lives |
| `08-roadmap.md` | What's deliberately deferred, and the phases to get from this spec to shipped code |

Read them in order the first time — each one assumes the last.