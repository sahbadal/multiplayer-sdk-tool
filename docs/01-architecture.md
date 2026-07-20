>  **[Click here for the full detailed architecture diagram →](https://sahbadal.github.io/multiplayer-sdk-tool)**

# Architecture

## Layered Architecture

```text
┌──────────────────────────────────────┐
│      Unity Game UI / Gameplay        │
└──────────────────┬───────────────────┘
                    │ uses
                    ▼
┌──────────────────────────────────────┐
│            SDK Public API            │
│ Session | Player | Matchmaking       │
│ RPC / Events | Voice                 │
└──────────────────┬───────────────────┘
                    │ calls
                    ▼
┌──────────────────────────────────────┐
│             SDK Services             │
│ Core | Session | Player              │
│ Matchmaking | RPC | Voice            │
└──────────────────┬───────────────────┘
                    │ reads
                    ▼
┌──────────────────────────────────────┐
│     ScriptableObject Configuration   │
│ SDK | Session | Player               │
│ Matchmaking | RPC | Voice            │
└──────────────────┬───────────────────┘
                    │ used by
                    ▼
┌──────────────────────────────────────┐
│         Photon Adapter Layer         │
│ Fusion Adapter | Voice Adapter       │
└──────────────────┬───────────────────┘
                    │ wraps
                    ▼
┌──────────────────────────────────────┐
│       Photon Fusion & Voice SDK      │
└──────────────────────────────────────┘
```

Each layer only communicates with the layer directly below it.

Game code never depends directly on Photon Fusion or Photon Voice APIs.

Only the Adapter Layer imports Photon namespaces.

If Photon changes in a future version, only the adapter layer should require modification.

---

# Configuration Architecture

The SDK uses a centralized configuration model based on ScriptableObjects.

```
SdkConfig.asset
│
├── SessionConfig.asset
├── PlayerConfig.asset
├── MatchmakingConfig.asset
├── RpcConfig.asset
└── VoiceConfig.asset
```

`SdkConfig.asset` acts as the root configuration and owns references to every module configuration.

Individual modules only read the configuration they own.

---

## Developer-facing Configuration

Every module exposes only the settings required by the developer.

| Configuration | Typical Settings |
|---------------|------------------|
| **SdkConfig.asset** | Fusion App ID, Voice App ID, Default Region, Enable Debug Logs |
| **SessionConfig.asset** | Session Name, Network Topology, Max Players, Is Visible, Is Open, Custom Session Properties |
| **PlayerConfig.asset** | Player Prefab, Auto Spawn, Auto Despawn |
| **MatchmakingConfig.asset** | Create Session If Not Found, Matchmaking Timeout |
| **RpcConfig.asset** | Enable RPC Validation, Enable RPC Logging, Max Payload Size |
| **VoiceConfig.asset** | Enable Voice, Auto Connect, Mute On Start, Enable Voice Detection |

> The Setup Window edits these assets. It is **not** another configuration source.

---

# Module Scope

```text
Unity Photon Multiplayer SDK
│
├── Core
│   ├── Bootstrap
│   ├── Configuration
│   ├── SDK State
│   ├── Validation
│   └── Logging
│
├── Session
│   ├── Connect
│   ├── Create Session
│   ├── Join Session
│   ├── Leave Session
│   └── Session State
│
├── Player
│   ├── Prefab Setup
│   ├── Spawn
│   ├── Despawn
│   ├── Registration
│   ├── Transform Synchronization
│   └── Authority
│
├── Matchmaking
│   ├── Quick Match
│   ├── Session Search
│   ├── Session Filters
│   ├── Create If Not Found
│   └── Cancel
│
├── RPC
│   ├── Broadcast Events
│   ├── Targeted Events
│   ├── Authority Requests
│   ├── Event Registration
│   ├── Payload Validation
│   └── Event Dispatch
│
├── Voice
│   ├── Voice Connection
│   ├── Local Recorder
│   ├── Remote Speakers
│   ├── Microphone Mute
│   ├── Remote Player Mute
│   ├── Speaking Status
│   └── Voice Lifecycle
│
└── Editor Tooling
    ├── Setup Window
    ├── Config Editors
    ├── Prefab Setup
    ├── Scene Setup
    ├── Validation Dashboard
    ├── Diagnostics
    └── Safe Auto Fixes
```

See **02-module-reference.md** for detailed responsibilities of every module.

---

# Source of Truth Rules

Every setting exists in exactly one location.

No duplicated configuration is allowed.

| Setting | Source of Truth |
|----------|-----------------|
| Photon App IDs | Photon Settings Asset |
| Photon Region List | Photon Settings Asset |
| SDK-wide Settings | `SdkConfig.asset` |
| Session Defaults | `SessionConfig.asset` |
| Player Setup | `PlayerConfig.asset` |
| Matchmaking Defaults | `MatchmakingConfig.asset` |
| RPC Rules | `RpcConfig.asset` |
| Voice Behaviour | `VoiceConfig.asset` |
| Scene Bootstrap | `MultiplayerBootstrap` |

The Setup Window edits these assets but never becomes another storage location.

---

# Runtime Configuration Flow

```text
Developer edits configuration
            │
            ▼
SDK Setup Window
            │
            ▼
ScriptableObject Assets
            │
            ▼
MultiplayerBootstrap
            │
            ▼
SDK Services
            │
            ▼
Photon Adapter Layer
            │
            ▼
Photon Fusion / Photon Voice
```

---

# Production Rules

1. Important references must always use explicit serialized references.
2. Runtime code must never rely on `GameObject.Find`, tags or naming conventions.
3. Runtime must not depend on `Resources.Load` as the primary loading mechanism.
4. Tags are optional and never required.
5. Folder conventions improve organization but are never mandatory.
6. ScriptableObjects are the single source of configuration.
7. Every setting has exactly one owner.
8. The SDK Setup Window is only an editor, not a storage layer.
9. The SDK must never modify a developer prefab without confirmation.
10. Automatic fixes must clearly distinguish safe changes from destructive changes.
11. Generated files must never overwrite developer-owned assets.
12. Only one `MultiplayerBootstrap` may exist per active scene.
13. Voice remains completely optional.
14. RPC/Event identifiers must remain unique and stable.
15. Matchmaking cancellation must be safe from every state.
16. Common Photon boilerplate should be hidden from the developer.
17. Advanced Photon APIs must remain accessible.
18. The SDK manages networking only and never game UI or gameplay.
19. Configuration assets should be reusable across scenes.
20. Runtime modules must never directly modify configuration assets.
