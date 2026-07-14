# Validation and error handling

## Validation categories

Validation is split into four categories, checked at different times:

| Category | Checked when | Examples |
|---|---|---|
| **Dependency** | Setup Window open, SDK init | Fusion/Voice packages installed, supported versions |
| **Configuration** | Setup Window, SDK init | Config assets exist, App IDs set, timeouts valid, RPC IDs unique |
| **Player prefab** | Prefab assigned, Validation tab | Required components present, no duplicates, registered for spawning |
| **Scene** | Scene setup, Validation tab | One bootstrap, no duplicate runner, no duplicate voice connection |

## Validation result structure

Every issue the SDK reports carries the same fields, so tooling and logs
stay consistent:

```
Severity:      Error
Code:          PLAYER_PREFAB_NETWORK_OBJECT_MISSING
Module:        Player
Message:       The configured player prefab does not contain NetworkObject.
Suggested fix: Add NetworkObject to the selected prefab.
Can auto-fix:  Yes
Destructive:   No
Related:       CustomPlayerPrefab.prefab
```

Same shape for a runtime error:

```
Code:        SESSION_JOIN_TIMEOUT
Module:      Session
Operation:   JoinSession
Message:     The session could not be joined within 30 seconds.
Suggestion:  Verify network connection or try another session.
Timestamp:   2026-07-14T12:41:03Z
```

## Logging levels

`Off → Error → Warning → Info → Verbose`, configurable per-project in
`SdkConfig.asset`. Verbose is intended for SDK development, not for
shipping builds.

## Error message design

Every error follows one shape: **[what happened]. [how to fix it].** No
stack trace as the primary message, no `"Error:"` prefix, no vague
wording. Each also carries a code (`SDK-<MODULE>-<NUMBER>`), a severity
(**Block** stops progress, **Warn** logs and continues, **Auto-fix**
corrects safely and logs what changed), and a surface — edit-time issues
show inline in the Setup Window and the console; runtime issues fire
through `MultiplayerSDK.Error` as an `SdkError`, never as a thrown
exception, since something like a dropped session is an expected event in
a networked game, not a crash.

## Error catalog

### Core

| Code | Trigger | Severity | Message |
|---|---|---|---|
| `SDK-CORE-001` | Photon Fusion package not found | Block | Photon Fusion package was not found in this project. Install Photon Fusion, then reopen the setup window. |
| `SDK-CORE-002` | Photon Voice package not found, voice enabled | Block | Photon Voice package was not found, but voice is enabled in VoiceConfig. Install Photon Voice or disable voice. |
| `SDK-CORE-003` | Fusion App ID empty | Block | Fusion App ID is empty. Paste your App ID from the Photon dashboard. |
| `SDK-CORE-004` | Voice App ID empty, voice enabled | Block | Voice App ID is empty, but voice is enabled. Add an App ID or disable voice. |
| `SDK-CORE-005` | Multiple `SdkConfig` assets found | Block | Multiple SdkConfig assets were found. Only one is allowed — delete the extra copy. |
| `SDK-CORE-006` | Default config folder missing | Auto-fix | Default config folder was not found. Config assets were created at Assets/SDK/Resources/SDKConfig/. |
| `SDK-CORE-007` | API called before init finished | Block | An SDK method was called before initialization finished. Wait for Ready before calling SDK methods. |
| `SDK-CORE-008` | Multiple bootstraps in one scene | Block | Multiple MultiplayerBootstrap components were found in this scene. Only one is allowed. |
| `SDK-CORE-009` | `InitializeAsync` called twice | Warn | The SDK is already initialized — this call was ignored. |

### Session

| Code | Trigger | Severity | Message |
|---|---|---|---|
| `SDK-SESSION-001` | `ConnectAsync` called while already connected | Warn | Already connected — this call was ignored. |
| `SDK-SESSION-002` | Session name empty on create, no default set | Auto-fix | Session name was empty. A random name was generated: "session-4f2a". |
| `SDK-SESSION-003` | Joined session no longer exists | Runtime error | The session you tried to join is no longer available. |
| `SDK-SESSION-004` | `LeaveAsync` called with no active session | Warn | No active session to leave — this call was ignored. |
| `SDK-SESSION-005` | `CreateAsync` called while already in a session | Block | Cannot create a session while already in one. Leave the current session first. |

### Player

| Code | Trigger | Severity | Message |
|---|---|---|---|
| `SDK-PLAYER-001` | Player prefab not assigned | Block | Player prefab is not assigned. Assign one in the Setup Window before starting the game. |
| `SDK-PLAYER-002` | Prefab missing a required component | Block, pending approval | Player prefab is missing a NetworkObject component. Approve the fix to add it automatically. |
| `SDK-PLAYER-003` | Fixed-points strategy with no spawn points | Block | Spawn strategy is set to Fixed Points, but no spawn points are assigned. |
| `SDK-PLAYER-004` | Spawn requested for an already-registered player | Warn | This player is already spawned — spawn request ignored. |
| `SDK-PLAYER-005` | Assigned object is a scene instance, not a prefab asset | Block | The assigned player object must be a prefab asset, not a scene instance. |

### Matchmaking

| Code | Trigger | Severity | Message |
|---|---|---|---|
| `SDK-MATCH-001` | `QuickMatchAsync` called while a search is in progress | Warn | Matchmaking is already in progress — this call was ignored. |
| `SDK-MATCH-002` | Search exceeds configured timeout | Runtime error | Matchmaking timed out after 30 seconds. No matching session was found. |
| `SDK-MATCH-003` | Malformed filter string | Block | Session filter "skillTier=" is invalid — missing a value. |
| `SDK-MATCH-004` | Matchmaking started while already in a session | Block | Cannot start matchmaking while already in a session. |

### RPC / network events

| Code | Trigger | Severity | Message |
|---|---|---|---|
| `SDK-RPC-001` | Duplicate event name registered | Block | Event name "PlayerReady" is already registered. Names must be unique. |
| `SDK-RPC-002` | Duplicate event ID registered | Block | Event ID 1001 is already registered to a different event. |
| `SDK-RPC-003` | Event sent before joining a session | Block | Cannot send an event before joining a session. |
| `SDK-RPC-004` | Sender lacks required authority | Warn, call ignored | This event requires state authority — call was ignored on this client. |
| `SDK-RPC-005` | Payload exceeds configured size | Block | Event payload exceeds the configured maximum size. |
| `SDK-RPC-006` | Rate limit exceeded | Warn, call dropped | This event was sent too frequently and was dropped by the rate limiter. |

### Voice

| Code | Trigger | Severity | Message |
|---|---|---|---|
| `SDK-VOICE-001` | Voice enabled but Voice App ID missing | Block | Voice is enabled but Voice App ID is empty. Add an App ID or disable voice. |
| `SDK-VOICE-002` | Voice enabled but Recorder reference missing | Block | Voice is enabled but no Recorder is assigned. Assign one or disable voice. |
| `SDK-VOICE-003` | Mute/unmute called for an unregistered player | Warn | Tried to mute a player that isn't currently connected — call was ignored. |
| `SDK-VOICE-004` | Duplicate local Recorder detected | Block | More than one local Recorder was found. Only one is allowed. |