# Editor tooling

## Why this module exists

Even a well-designed runtime SDK stays hard to use if the developer still
has to open multiple assets, hunt for Photon's settings file, add
components by hand, register prefabs manually, and diagnose problems by
reading console stack traces. Editor Tooling gives the whole SDK one
setup surface.

## Setup window

```
Tools
└── Multiplayer SDK
    └── Setup
```

Tabs: **Setup · Session · Player · Matchmaking · RPC · Voice ·
Validation · Diagnostics**

### Setup tab
Fusion App ID, Voice App ID, auto-initialize, persist-between-scenes,
debug logging, module config references, dependency status. The
developer only ever sees this one window — internally, each field writes
to its correct source-of-truth file (see `01-architecture.md`).

### Player tab
Player prefab reference, validation result, list of missing required
components, a preview of the proposed changes, an "Apply required
components" action, network-prefab registration, spawn settings. The SDK
never edits a prefab without this explicit confirmation step.

### Validation tab
Runs every check across dependencies, config, prefab, and scene, and
shows a pass/fail list:

```
✓ Photon Fusion package detected
✓ Fusion App ID configured
✓ SDK configuration created
✓ Session configuration valid
✗ Player prefab missing NetworkObject
✗ Player prefab missing SDKNetworkPlayer
✓ RPC event IDs are unique
⚠ Voice Recorder not assigned
✓ Scene bootstrap installed
```

Actions available: **Validate all · Fix selected safe issues · Fix all
safe issues · Open related configuration**. "Fix" only ever applies to
issues marked auto-fixable — destructive changes are never offered here,
only surfaced with a suggested manual fix.

## Scene setup

"Install scene setup" does the following:

```
Check whether an SDK bootstrap exists
        ↓
No bootstrap found? ── Yes → Create MultiplayerSDKRoot
                       └ No  → Reuse the existing one
        ↓
Assign SdkConfig
        ↓
Configure required runtime components
        ↓
Validate the scene
```

```
MultiplayerSDKRoot
└── MultiplayerBootstrap
```

This root object holds networking logic only — no visual design, no
gameplay behavior lives on it.

## Detection priority — how the SDK finds things

Used during editor-time validation, never as the primary runtime lookup
mechanism:

1. Explicit serialized reference (preferred, used everywhere at runtime)
2. Required component or interface present on the object
3. A generated asset registry
4. The recommended folder convention (fallback only)
5. Tag or name — last resort, editor-time only

Runtime code specifically avoids `GameObject.Find("Player")`,
`GameObject.FindWithTag("NetworkPlayer")`, or a bare
`Resources.Load("Player")` as its main mechanism. Anything the SDK needs
at runtime was already handed to it explicitly during setup — see
production rule 1 and 2 in `01-architecture.md`.