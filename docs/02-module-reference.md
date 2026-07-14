# Module reference

Entry point for all public calls is a static `MultiplayerSDK` class. Every
module below is reached through it — `MultiplayerSDK.Session`,
`MultiplayerSDK.Players`, `MultiplayerSDK.Matchmaking`,
`MultiplayerSDK.Events`, `MultiplayerSDK.Voice`.

---

## Core

**Problem it solves.** Without a central module, every service (session,
player, matchmaking, RPC, voice) could initialize independently and in
the wrong order — for example, matchmaking starting before the session
service exists. Core owns initialization order, shared state, and a
common error format.

**Developer provides**
- Auto-initialize on scene load — on/off
- Debug logging — on/off, plus log level
- Persist SDK across scene loads — on/off
- References to each module's config asset (usually pre-wired by the
  Setup Window)

**Configuration storage — `SdkConfig.asset`**
SDK version, auto-initialize, persist-between-scenes, enable debug logs,
log level, plus references to `SessionConfig`, `PlayerConfig`,
`MatchmakingConfig`, `RpcConfig`, `VoiceConfig`. Photon App IDs are *not*
duplicated here — they stay in Photon's own settings assets; the Setup
Window edits them in place.

**SDK handles internally**
Loading `SdkConfig`, validating required config and Photon dependencies,
initializing every service in dependency order, tracking current SDK
state, converting internal exceptions into the common `SdkError` shape,
and shutting services down cleanly.

**Public API**
```csharp
await MultiplayerSDK.InitializeAsync();
MultiplayerSDK.State;      // NotInitialized, Initializing, Ready, Error, ...
MultiplayerSDK.IsReady;
await MultiplayerSDK.ShutdownAsync();
```

**Validation rules**
`SdkConfig` must exist · all required module config references must be
assigned · only one active bootstrap may exist · Photon Fusion must be
installed · Fusion App ID must be set · Voice config is required only if
voice is enabled · the SDK must not initialize twice · calls made before
`Ready` are rejected with `SDK-CORE-007`.

**Stays outside the SDK.** Nothing — this module has no gameplay-facing
surface, it's purely internal plumbing.

---

## Session

**Problem it solves.** Removes the need to hand-configure
`NetworkRunner`, `StartGameArgs`, and Fusion's connection callbacks for
every project.

**Developer provides**
Default session name, network topology (Fusion's authority model — e.g.
Shared), gameplay mode (a game-specific label — e.g. Deathmatch — kept
*separate* from topology), max players, region, visibility, openness,
custom session properties, connection/join timeouts, reconnect behavior.

**Configuration storage — `SessionConfig.asset`**
All of the above as defaults. Individual calls can override any of them
per-request (e.g. a different session name at runtime) without touching
the asset.

**SDK handles internally**
Creating and owning the `NetworkRunner`, registering its callbacks,
connecting to Photon, creating/joining/leaving sessions, tracking current
session info and connected players, converting Photon errors into
`SdkError`, and preventing duplicate connection attempts.

**Public API**
```csharp
await MultiplayerSDK.Session.ConnectAsync();
await MultiplayerSDK.Session.CreateAsync(new CreateSessionRequest { SessionName = "room-101", MaxPlayers = 8 });
await MultiplayerSDK.Session.JoinAsync("room-101");
await MultiplayerSDK.Session.LeaveAsync();
MultiplayerSDK.Session.CurrentSession;
MultiplayerSDK.Session.State;

MultiplayerSDK.Session.Connected += OnConnected;
MultiplayerSDK.Session.Joined    += OnSessionJoined;
MultiplayerSDK.Session.Left      += OnSessionLeft;
MultiplayerSDK.Session.Failed    += OnSessionFailed;
```

**Validation rules**
Session name required for an explicit join · max players > 0 and within
Photon's supported limit · can't create while already in a session · a
second join can't start while one is in flight · property keys must be
unique and of a supported type · region must be a valid Photon region ·
`LeaveAsync` must be safe to call even after a partial connection
failure.

**Stays outside the SDK.** Lobby UI, room browser UI, and any
game-specific rule for what makes a session "valid" beyond player count.

---

## Player

**Problem it solves.** A developer normally hand-builds a networked
player prefab, adds `NetworkObject` and transform sync, registers it,
spawns it, assigns authority, and tracks local vs. remote players
manually. The SDK does not ship a default visual player — the developer
brings their own prefab; the SDK wires the networking parts onto it.

**Developer provides**
Their own player prefab, auto-spawn on/off, auto-despawn on/off, spawn
strategy, transform sync settings (position/rotation/interpolation),
authority mode, an optional custom spawn point provider.

**Configuration storage — `PlayerConfig.asset`**
Player prefab reference, auto spawn/despawn, spawn strategy, spawn point
selection, sync position/rotation, interpolation, authority mode.

**Prefab setup flow**
```
Developer creates their own player prefab
        ↓
Developer assigns it in the Setup Window
        ↓
SDK validates it and lists missing required components
        ↓
Developer reviews the proposed changes
        ↓
Developer clicks "Apply required components"
        ↓
SDK adds only what's missing — never removes developer content
        ↓
Reference saved to PlayerConfig.asset
```
Required additions are limited to `NetworkObject`, `NetworkTransform`, and
an internal `SDKNetworkPlayer` adapter. The SDK never touches the
developer's model, animator, character controller, gameplay scripts,
colliders, or visual children.

**SDK handles internally**
Validating the prefab, registering it for network spawning, spawning the
local player, assigning input authority, tracking local and remote player
references, despawning safely, and exposing player lookup.

**Spawn strategies**
First available point · random point · round-robin · custom provider ·
scene-origin fallback. Spawn points are identified by a
`MultiplayerSpawnPoint` component on a GameObject, never by tag or name.

**Public API**
```csharp
NetworkPlayer localPlayer = MultiplayerSDK.Players.LocalPlayer;
IReadOnlyList<NetworkPlayer> players = MultiplayerSDK.Players.AllPlayers;
NetworkPlayer player = MultiplayerSDK.Players.GetPlayer(playerId);

await MultiplayerSDK.Players.SpawnLocalPlayerAsync();
await MultiplayerSDK.Players.DespawnLocalPlayerAsync();

MultiplayerSDK.Players.PlayerJoined        += OnPlayerJoined;
MultiplayerSDK.Players.PlayerLeft          += OnPlayerLeft;
MultiplayerSDK.Players.LocalPlayerSpawned  += OnLocalPlayerSpawned;
```

**Validation rules**
Prefab must be assigned and must be a prefab asset (not a scene
instance) · must contain `NetworkObject` and the SDK player adapter ·
sync components must not be duplicated · only one local player may
auto-spawn · spawn points must not contain invalid duplicates · the SDK
must show a change preview and ask for confirmation before touching a
prefab.

**Stays outside the SDK.** The player's model, animation, movement
controller, health/inventory/gameplay scripts — the SDK only owns the
networking components it explicitly adds.

---

## Matchmaking

**Problem it solves.** Search-then-create-if-none-found is a flow every
project rebuilds by hand. This uses Photon's own session matchmaking, not
a separate backend queue.

**Developer provides**
Matchmaking profile/mode name, region, search timeout, session property
filters, create-if-not-found on/off, max players, retry count/delay.

**Configuration storage — `MatchmakingConfig.asset`**
```
Profile: ranked-1v1
Region: Asia
Search timeout: 30 seconds
Session filter: skillTier=Gold
Create if not found: true
```

**SDK handles internally**
Starting a request, searching compatible sessions, applying filters,
selecting and joining a valid one, creating a fallback session if
enabled, tracking matchmaking state, supporting cancellation, and
preventing more than one simultaneous request.

**Flow**
```
QuickMatchAsync called
        ↓
Request + config validated
        ↓
Search compatible sessions
        ↓
Match found? ── Yes → Join session
        │
        No
        ↓
Create-if-not-found enabled? ── Yes → Create session
                               └ No  → Return NoMatchFound
```

**Public API**
```csharp
MatchResult result = await MultiplayerSDK.Matchmaking.QuickMatchAsync();
MatchResult custom = await MultiplayerSDK.Matchmaking.QuickMatchAsync(
    new MatchRequest { Mode = "ranked-1v1", Region = "asia", MaxPlayers = 2 });
await MultiplayerSDK.Matchmaking.CancelAsync();

MultiplayerSDK.Matchmaking.Started    += OnStarted;
MultiplayerSDK.Matchmaking.MatchFound += OnMatchFound;
MultiplayerSDK.Matchmaking.Cancelled  += OnCancelled;
MultiplayerSDK.Matchmaking.Failed     += OnFailed;
```

**Validation rules**
Only one matchmaking operation at a time · timeout > 0 · filter keys
unique and correctly typed · max players valid · can't start while
already in a session · cancellation must be safe at every stage ·
create-if-not-found must reuse the same properties used in the search.

**Stays outside the SDK.** Skill-rating calculation, party/friend-based
matching, and any custom backend queue — this is session-level
matchmaking only.

---

## RPC / network events

**Problem it solves.** Native Photon RPC is powerful but repetitive to
wire consistently — authority rules, payloads, reliability, and rate
limits get decided ad hoc per project. The SDK offers a simplified event
layer for common cases while leaving native `[Rpc]` fully usable for
anything advanced.

**Two levels**
- **SDK network events** — for common, simple events (`PlayerReady`,
  `MatchStarted`, `MatchEnded`, `PlayerActionRequested`,
  `CustomGameplayEvent`)
- **Native Photon RPC** — untouched, still available for
  highly game-specific networking

**Developer provides**
Enable/disable the RPC helper layer, enable RPC logging, reject-unknown
events on/off, default target, max payload size, rate limit, and the set
of registered event definitions.

**Configuration storage — `RpcConfig.asset`**
Enable RPC helpers, enable logging, reject unknown events, default
target, max payload size, rate limit on/off + events-per-second,
registered event definitions.

**Why stable IDs matter.** A string call like `Send("PlyerReddy")` only
fails at runtime, silently. `Send(NetworkEventId.PlayerReady)` fails to
compile instead. Each registered event carries a stable ID, name, source
rule, target rule, payload type, reliability, rate limit, and
description — which lets the SDK catch duplicate IDs, duplicate names,
unknown events, and payload mismatches before they ship.

**SDK handles internally**
Registering event definitions, sending broadcast/targeted/authority-gated
events, validating IDs/payloads/authority/rate limits on send, dispatching
incoming events to registered listeners, and converting transport errors
to `SdkError`.

**Public API**
```csharp
MultiplayerSDK.Events.Broadcast(NetworkEventId.PlayerReady);
MultiplayerSDK.Events.Broadcast(NetworkEventId.MatchStarted, payload);
MultiplayerSDK.Events.SendToPlayer(targetPlayerId, NetworkEventId.CustomGameplayEvent, payload);
MultiplayerSDK.Events.SendToStateAuthority(NetworkEventId.PlayerActionRequested, payload);
MultiplayerSDK.Events.Subscribe(NetworkEventId.MatchStarted, HandleMatchStarted);
```

**Validation rules**
Every event ID and name unique · payload must match the registered type
and stay under the size limit · sender must hold the required authority ·
unknown events rejected when strict mode is on · rate-limited events must
not exceed their configured limit · listeners must be removable · none of
this blocks a developer from also using native Photon `[Rpc]` methods
directly.

**Stays outside the SDK.** Game-specific event payload design beyond the
handful of common events listed above — anything bespoke goes through
native Photon RPC.

---

## Voice

**Problem it solves.** Photon Voice normally needs manual setup of the
voice connection, `Recorder`, `Speaker` mapping per remote player, mute
state, speaking detection, and cleanup on disconnect.

**Developer provides**
Enable voice, auto-connect voice, mute on start, voice mode (Global,
Spatial, or Custom), voice interest group, Recorder reference/prefab,
speaking-detection on/off + threshold, remote-mute support, voice
logging.

**Configuration storage — `VoiceConfig.asset`**
All of the above. Spatial voice stays optional — the SDK itself stays
Unity-generic, not VR-specific.

**SDK handles internally**
Initializing Photon Voice, connecting it to the current session,
configuring the local `Recorder`, mapping remote `Speaker`s, handling
microphone and per-remote-player mute, tracking speaking state,
reconnecting when required, and cleaning up voice resources on leave —
without ever blocking the rest of the session if voice fails or is
disabled.

**Flow**
```
Player joins session
        ↓
Voice enabled? ── No → skip voice entirely
                 └ Yes
                    ↓
             Connect voice
                    ↓
             Start local Recorder
                    ↓
             Map remote Speakers
                    ↓
             Speaking-state events available
```

**Public API**
```csharp
MultiplayerSDK.Voice.SetMicrophoneMuted(true);
bool isMuted = MultiplayerSDK.Voice.IsMicrophoneMuted;
MultiplayerSDK.Voice.SetRemotePlayerMuted(playerId, true);
bool isSpeaking = MultiplayerSDK.Voice.IsPlayerSpeaking(playerId);

MultiplayerSDK.Voice.Connected      += OnVoiceConnected;
MultiplayerSDK.Voice.Disconnected   += OnVoiceDisconnected;
MultiplayerSDK.Voice.SpeakingChanged += OnSpeakingChanged;
MultiplayerSDK.Voice.Error          += OnVoiceError;
```

**Validation rules**
Voice App ID required only when voice is enabled · Recorder reference
must be valid · no duplicate local Recorder · voice must not initialize
twice · must disconnect safely on session leave · remote mute must not
affect other players · a voice failure must never crash the session ·
voice-disabled projects have zero voice-config requirements.

**Stays outside the SDK.** Text chat, spatial audio tuning beyond basic
mode selection, and any custom push-to-talk UI.