# Implementation checklist

## How to use this doc

This is the build order — do the phases top to bottom, in sequence. Every
phase ends with a **Complete when** list. Those conditions are not
suggestions — they're the gate. If a "Complete when" item isn't true yet,
the next phase does not start, even if it looks tempting to jump ahead.
This is what keeps a half-finished Session module from getting buried
under three more phases of code built on top of it, only to break
everything when you go back to fix it.

> **Rule: do not start the next phase until every "Complete when" box in
> the current phase is genuinely true — not "mostly done," not "works on
> my machine once."**

---

## Final implementation order

```text
 1. Project Foundation
 2. Configuration System
 3. Dependency Injection (VContainer)
 4. Core Runtime Bootstrap
 5. Photon Adapter Layer
 6. Fusion Session Runtime
 7. Player Runtime
 8. RPC Module
 9. Photon Voice Module
10. Matchmaking Module
11. Validation Framework
12. Player Prefab Validation
13. Scene Validation
14. SDK Setup Window
15. Public SDK API
16. Testing
17. Sample Scene and Documentation
18. Package and Release
```

---

## Phase 1: Project Foundation

- [ ] Create the final SDK folder structure (see `07-project-structure.md`)
- [ ] Create Runtime, Editor, and Tests assembly definitions
- [ ] Define SDK namespaces
- [ ] Add SDK version information
- [ ] Add common `SdkResult`, `SdkError`, and logging models
- [ ] Confirm Editor code cannot enter runtime builds

**Complete when**
- SDK compiles without errors
- Runtime and Editor code are separated into their own assemblies
- Basic SDK types (`SdkResult`, `SdkError`) are available to every module

---

## Phase 2: Configuration System

- [ ] Implement `SdkConfig`
- [ ] Implement `SessionConfig`
- [ ] Implement `PlayerConfig`
- [ ] Implement `MatchmakingConfig`
- [ ] Implement `RpcConfig`
- [ ] Implement `VoiceConfig`
- [ ] Make `SdkConfig` reference all module configs
- [ ] Add configuration validation
- [ ] Decide on one runtime loading method (see the `Resources.Load`
      vs. direct-reference note in `07-project-structure.md`)
- [ ] Create default configuration assets

**Complete when**
- Every configuration value has exactly one source of truth (see
  `01-architecture.md`)
- SDK can load and validate configuration
- Missing config produces a clear error, not a silent null

---

## Phase 3: Dependency Injection (VContainer)

- [ ] Install VContainer
- [ ] Create `SdkLifetimeScope`
- [ ] Register currently available core services and configuration assets
- [ ] Define registration methods for services and adapters that will be
      added in later phases
- [ ] Use constructor injection throughout — no field injection, no
      service locator pattern
- [ ] Define service lifetimes (Singleton vs. Scoped) deliberately per
      service, not by default

**Complete when**
- Core configuration, logger, and SDK state services resolve through
  VContainer
- Lifetime rules and registration structure are defined
- Future services can be registered without changing the composition
  design

> **Note on ordering:** `SdkLifetimeScope` is defined here, but it isn't
> *built* yet — that needs `MultiplayerBootstrap`, which doesn't exist
> until Phase 4. Building the container is Phase 4's job. Phase 3 only
> defines what the container looks like and registers what's already
> available (config, logging, SDK state); Session, Player, and the
> Photon adapters get registered in their own later phases, once those
> services actually exist. Don't try to fully wire the container here —
> it isn't possible yet, and forcing it creates the exact circular
> dependency this note is warning against.

---

## Phase 4: Core Runtime Bootstrap

- [ ] Implement `MultiplayerBootstrap`
- [ ] Load and validate `SdkConfig`
- [ ] Connect `MultiplayerBootstrap` with `SdkLifetimeScope`
- [ ] Build and initialize the VContainer scope during SDK startup
- [ ] Initialize SDK services through the built container
- [ ] Initialize Photon adapters through the same container
- [ ] Prevent duplicate SDK initialization
- [ ] Add SDK startup and shutdown lifecycle
- [ ] Expose SDK initialization state (`NotInitialized → Initializing →
      Ready → Error`, see `01-architecture.md`)
- [ ] Add cleanup for scene unload and application shutdown

**Complete when**
- Adding one bootstrap initializes the whole SDK
- A duplicate bootstrap or a second initialization call is handled
  safely, not silently ignored and not a crash
- SDK shuts down without leaving runtime objects behind

---

## Phase 5: Photon Adapter Layer

Full design detail for this phase lives in
`09-photon-integration-adapter-layer.md` — this checklist is the
build-order version of that doc.

- [ ] Create `IFusionAdapter`
- [ ] Create `IVoiceAdapter`
- [ ] Implement `PhotonFusionAdapter`
- [ ] Implement `PhotonVoiceAdapter`
- [ ] Keep Photon types strictly inside adapter implementations — never
      let a Photon type leak into a service constructor signature
- [ ] Convert Photon callbacks into SDK events
- [ ] Convert Photon errors into `SdkError`
- [ ] Add mock adapters for testing (registered as an alternate
      VContainer binding, not a hand-wired substitute)

**Complete when**
- SDK services do not directly depend on any Photon class
- Public APIs do not expose `NetworkRunner`, `PlayerRef`, `Recorder`, or
  any other Photon type
- Adapters can be replaced with mocks through the container, with no
  code change in the services that use them

---

## Phase 6: Fusion Session Runtime

- [ ] Implement `SessionService`
- [ ] Detect or create one `NetworkRunner`
- [ ] Enforce a single active runner
- [ ] Implement create session
- [ ] Implement join session
- [ ] Implement leave session
- [ ] Implement shutdown
- [ ] Handle `INetworkRunnerCallbacks`
- [ ] Track session state
- [ ] Handle connection failure and disconnect
- [ ] Expose clean SDK session events
- [ ] Destroy and recreate the `NetworkRunner` after it disconnects or
      fails to connect; do not attempt to restart the same runner
      instance

**Complete when**
- A developer can create, join, and leave using only SDK APIs
- Duplicate session starts are blocked, not silently allowed
- Failed connections return clear SDK errors, not raw Photon exceptions
- Runner cleanup works correctly on every exit path (normal leave,
  disconnect, and app shutdown)
- A fresh `NetworkRunner` is created for reconnect or the next session —
  a Photon `NetworkRunner` is not designed to be reused after it
  disconnects or fails, so reconnect always means a new instance, never
  a restart of the old one

---

## Phase 7: Player Runtime

- [ ] Implement `PlayerService`
- [ ] Load the player prefab from `PlayerConfig`
- [ ] Validate that the prefab contains required network components
- [ ] Handle player joined
- [ ] Spawn the local player
- [ ] Track local and remote players
- [ ] Handle player left
- [ ] Despawn and remove player mappings
- [ ] Add a spawn-position provider interface
- [ ] Expose SDK player events

**Complete when**
- Players spawn and despawn correctly
- Player mappings do not remain after disconnect — no stale references
- SDK does not control game-specific movement
- Games can provide their own spawn-location logic through the provider
  interface, without touching `PlayerService`

---

## Phase 8: RPC Module

**Scope.** This module covers SDK-owned RPCs, common RPC helpers,
validation, logging, and routing. It does not attempt to replace
game-specific gameplay RPCs — Fusion RPC methods are normally defined
directly on a developer's own `NetworkBehaviour`, and that stays true
here. Game-specific RPCs remain inside developer-owned
`NetworkBehaviour`s, written with native Fusion `[Rpc]` methods exactly
as they would be without this SDK; only the common, cross-cutting cases
(the events listed in `02-module-reference.md` — `PlayerReady`,
`MatchStarted`, and similar) go through the SDK's RPC layer.

- [ ] Define the public RPC API for SDK-owned events
- [ ] Define allowed RPC targets
- [ ] Define reliable and unreliable RPC options
- [ ] Add RPC validation rules (unique IDs/names — see
      `06-validation-and-errors.md`)
- [ ] Implement RPC sending through Fusion
- [ ] Convert incoming RPC calls into SDK events or handlers
- [ ] Prevent unsupported payloads
- [ ] Add useful RPC errors and logs

**Complete when**
- SDK-owned RPC flows use consistent validation and error handling
- Developers can still define game-specific RPCs in their own
  `NetworkBehaviour`s, with no interference from the SDK
- SDK does not attempt to serialize arbitrary unsupported payloads

---

## Phase 9: Photon Voice Module

- [ ] Implement `VoiceService`
- [ ] Start Voice only after Fusion connects (see the ordering rule in
      `09-photon-integration-adapter-layer.md`)
- [ ] Initialize `VoiceConnection`
- [ ] Detect or create one local `Recorder`
- [ ] Handle remote `Speaker` components
- [ ] Map Fusion players to Voice participants
- [ ] Implement microphone mute
- [ ] Implement speaker mute
- [ ] Expose speaking-state events
- [ ] Handle Voice failure separately from Fusion failure
- [ ] Disconnect and clean up Voice during session shutdown

**Complete when**
- Voice connects after the multiplayer session, never before or in
  parallel
- Mute and speaker controls work through SDK APIs
- A Voice failure does not unnecessarily stop gameplay — Session and
  Player keep working with Voice down
- Recorder and Speaker components are cleaned up correctly on leave

---

## Phase 10: Matchmaking Module

- [ ] Define `MatchmakingService`
- [ ] Implement quick-match request
- [ ] Implement cancel matchmaking
- [ ] Track matchmaking state
- [ ] Handle match-found result
- [ ] Convert match result into a session join request
- [ ] Handle timeout and failure
- [ ] Prevent duplicate matchmaking requests
- [ ] Expose matchmaking events

**Complete when**
- Quick Match can find a match and join its Fusion session end to end
- Cancellation and failure paths both work, from every matchmaking state
- Matchmaking code stays separate from `SessionService` — it calls
  Session, it isn't merged into it

---

## Phase 11: Validation Framework

- [ ] Implement `ValidationResult`
- [ ] Define severity levels: Info, Warning, Error (maps to the
      Auto-fix / Warn / Block severities in `06-validation-and-errors.md`)
- [ ] Define validator interfaces
- [ ] Implement a central validation runner
- [ ] Support project, scene, config, and prefab validation
- [ ] Add safe auto-fix support
- [ ] Require confirmation before modifying developer assets
- [ ] Integrate Unity Undo for editor fixes

**Complete when**
- All validators return results in the same consistent shape
- Auto-fixes can be undone through normal Unity Undo
- Unsafe (destructive) changes are never applied automatically, under
  any setting

---

## Phase 12: Player Prefab Validation

- [ ] Check that a player prefab is assigned
- [ ] Check required Fusion components
- [ ] Check duplicate components
- [ ] Check required SDK bridge components
- [ ] Validate Voice components according to the selected Voice setup:
      player-prefab voice, separate voice prefab, or runtime-managed
      voice — do not assume every player prefab must carry a `Recorder`
      and `Speaker`; Photon Voice's Fusion integration can attach voice
      to the spawned player object, but voice sources are just as
      commonly separate objects from the avatar itself
- [ ] Add safe missing components only after confirmation
- [ ] Preserve developer-owned scripts and settings
- [ ] Save prefab changes correctly

**Complete when**
- An invalid player prefab is detected before runtime, not discovered
  as a crash mid-game
- Safe fixes work without damaging anything else on the prefab
- Manual-fix instructions are clear when a fix can't be automatic

---

## Phase 13: Scene Validation

- [ ] Check for `MultiplayerBootstrap`
- [ ] Check for duplicate bootstrap objects
- [ ] Check for duplicate `NetworkRunner` objects
- [ ] Validate the bootstrap's configuration reference
- [ ] Check required scene manager configuration
- [ ] Detect duplicate Voice runtime objects
- [ ] Add safe scene fixes using Undo

**Complete when**
- Scene problems are visible before Play Mode, not after
- Duplicate runner and bootstrap issues are actively prevented, not just
  logged
- Scene auto-fixes are safe and fully reversible

---

## Phase 14: SDK Setup Window

Full design for this window lives in `03-editor-tooling.md` — this is
the implementation checklist for it.

- [ ] Create the SDK Editor Window
- [ ] Add General tab
- [ ] Add Session tab
- [ ] Add Player tab
- [ ] Add Matchmaking tab
- [ ] Add RPC tab
- [ ] Add Voice tab
- [ ] Add Validation tab
- [ ] Edit configs using `SerializedObject` and `SerializedProperty` —
      never by writing to a second copy of the data
- [ ] Add "Create Default Config" action
- [ ] Add "Validate Project" action
- [ ] Add prefab and scene fix actions
- [ ] Display clear success, warning, and error messages

**Complete when**
- The SDK can be fully configured without manually opening every asset
  one by one
- Setup Window and the raw ScriptableObjects edit the exact same source
  of truth — never two copies that can drift apart
- Validation results and fixes are all available from one place

---

## Phase 15: Public SDK API

- [ ] Create simple public entry points
- [ ] Expose Session API
- [ ] Expose Player API
- [ ] Expose Matchmaking API
- [ ] Expose RPC API
- [ ] Expose Voice API
- [ ] Expose SDK-owned events and models only
- [ ] Hide internal services and Photon implementations completely
- [ ] Add clear initialization checks (calls before `Ready` fail with a
      specific error, see `SDK-CORE-007` in `06-validation-and-errors.md`)

**Example**
```csharp
await MultiplayerSdk.Session.JoinAsync(request);

MultiplayerSdk.Voice.SetMicrophoneMuted(true);

MultiplayerSdk.Matchmaking.StartQuickMatch(request);
```

**Complete when**
- Game code uses only the SDK public API — nothing else is reachable
  from outside the SDK assembly
- Photon-specific references are never required in game code
- Common operations need minimal setup and minimal code, matching the
  original goal in `00-overview.md`

---

## Phase 16: Testing

- [ ] Add EditMode configuration tests
- [ ] Add validation tests
- [ ] Add adapter-mapping tests
- [ ] Add Session Service tests using a mock adapter (from the binding
      added in Phase 5)
- [ ] Add Player Service tests
- [ ] Add Voice Service tests
- [ ] Add Matchmaking state tests
- [ ] Add PlayMode connection tests
- [ ] Test connect, disconnect, and reconnect
- [ ] Test two-player spawn and despawn
- [ ] Test Voice startup and cleanup
- [ ] Test duplicate bootstrap and runner cases

**Complete when**
- Core logic can be tested without Photon Cloud, using the mock adapters
- Critical runtime flows pass PlayMode testing
- Failure and cleanup paths are covered, not just the happy path

---

## Phase 17: Sample Scene and Documentation

- [ ] Create a minimal sample scene
- [ ] Add create, join, and leave buttons
- [ ] Add a developer-owned sample player prefab
- [ ] Add an RPC example
- [ ] Add Voice controls
- [ ] Add a Quick Match example
- [ ] Write an installation guide
- [ ] Write a quick-start guide
- [ ] Document configuration
- [ ] Document public APIs
- [ ] Document common errors
- [ ] Document what developers should never modify manually

**Complete when**
- A new developer can install and run the sample without help
- The sample demonstrates every module shipped in this version
- Basic integration can be completed by following documentation alone —
  no tribal knowledge required

---

## Phase 18: Package and Release

- [ ] Convert the SDK into a valid UPM package structure
- [ ] Finalize assembly definitions
- [ ] Declare Photon dependencies clearly
- [ ] Add package metadata
- [ ] Add a changelog
- [ ] Add a semantic version
- [ ] Remove test-only and temporary files
- [ ] Test installation in a clean Unity project
- [ ] Test upgrade from the previous package version
- [ ] Create the initial release

**Complete when**
- SDK installs cleanly in a fresh project
- Sample and documentation both work immediately after installation
- No hidden, project-specific dependency remains anywhere in the package