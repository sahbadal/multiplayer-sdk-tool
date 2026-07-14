# Architecture

## Layered architecture

```
┌──────────────────────────────────────┐
│      Unity Game UI / Gameplay        │
└──────────────────┬───────────────────┘
                    │ uses
                    ▼
┌──────────────────────────────────────┐
│            SDK Public API            │
│  Session | Player | Matchmaking      │
│  RPC / Events | Voice                │
└──────────────────┬───────────────────┘
                    │ calls
                    ▼
┌──────────────────────────────────────┐
│             SDK Services             │
└──────────────────┬───────────────────┘
                    │ reads
                    ▼
┌──────────────────────────────────────┐
│        ScriptableObject Config       │
└──────────────────┬───────────────────┘
                    │ used by
                    ▼
┌──────────────────────────────────────┐
│         Photon Adapter Layer         │
└──────────────────┬───────────────────┘
                    │ wraps
                    ▼
┌──────────────────────────────────────┐
│       Photon Fusion & Voice SDK      │
└──────────────────────────────────────┘
```

Each layer only talks to the layer directly below it — the game never
sees Fusion types, and only the adapter layer imports Fusion/Voice
namespaces. If Fusion's API changes in a future SDK version, only the
adapter layer changes; everything above it is untouched.

## Module scope

```
Unity Photon Multiplayer SDK
│
├── Core            — Bootstrap, Configuration, SDK state, Validation, Logging
├── Session         — Connect, Create, Join, Leave, Session state
├── Player          — Prefab setup, Spawn, Despawn, Transform sync, Registration
├── Matchmaking     — Quick match, Session search, Filters, Create-if-not-found, Cancel
├── RPC             — Broadcast events, Targeted events, Authority requests, Event registration, Validation
├── Voice           — Voice connection, Microphone mute, Remote mute, Speaking status, Lifecycle
└── Editor Tooling  — Setup window, Config editors, Prefab setup, Scene setup, Validation dashboard
```

Full detail on every item is in `02-module-reference.md`.

## Source-of-truth rules

Every setting lives in exactly one place. Nothing is duplicated across
two inspectors or two files.

| Setting | Lives in |
|---|---|
| Photon-owned values (App IDs, region list) | Photon's own settings assets |
| SDK-wide settings | `SdkConfig.asset` |
| Session defaults | `SessionConfig.asset` |
| Player setup | `PlayerConfig.asset` |
| Matchmaking defaults | `MatchmakingConfig.asset` |
| RPC/event rules | `RpcConfig.asset` |
| Voice behavior | `VoiceConfig.asset` |
| Scene bootstrap reference | `MultiplayerBootstrap` component in the scene |

The Setup Window (`03-editor-tooling.md`) is only the *editing surface*
for these files — it never becomes a second storage location. When the
developer types an App ID into the Setup Window, it writes to Photon's
settings asset, not to a separate SDK-owned copy.

## Production rules

These are hard constraints the implementation must follow, independent of
which module is being written:

1. Important references are always explicit, serialized fields — never
   resolved by searching the scene at runtime.
2. Runtime code never uses `GameObject.Find`, tag lookups, or
   `Resources.Load` by convention as its primary mechanism — see the
   detection priority order in `03-editor-tooling.md`.
3. Tags are never required for core SDK behavior.
4. Recommended folder structure is a convenience, not a hard requirement.
5. ScriptableObject assets are the single configuration source of truth.
6. The SDK never modifies a developer's prefab without explicit
   confirmation.
7. Safe automatic fixes are always visibly separated from destructive
   changes — never the same code path.
8. Generated files are stored separately from developer-owned files.
9. Only one active SDK bootstrap is allowed per scene.
10. Voice always remains optional — no other module depends on it being
    connected.
11. RPC/event identifiers must stay unique and stable.
12. Matchmaking cancellation must be safe at every stage of the flow.
13. Common Photon boilerplate is hidden by default.
14. Advanced Photon APIs stay reachable for cases the simplified surface
    doesn't cover.
15. The SDK never controls game UI or gameplay design — it manages
    networking, not the game.