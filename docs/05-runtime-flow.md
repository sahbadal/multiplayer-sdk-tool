# Runtime flow (play-time)

## Initialization flow

```
Unity application starts
        ↓
MultiplayerBootstrap starts
        ↓
SdkConfig is loaded
        ↓
Core configuration is validated
        ↓
Photon dependencies are checked
        ↓
Session service is initialized
        ↓
Player service is initialized
        ↓
Matchmaking service is initialized
        ↓
RPC service is initialized
        ↓
Voice service is initialized if enabled
        ↓
SDK state becomes Ready
```

## SDK states

`NotInitialized → Initializing → Ready → Connecting → Connected →
InSession → Disconnecting → ShuttingDown`, with `Error` reachable from
any state. Game code can read `MultiplayerSDK.State` at any point and
react safely rather than guessing.

## Typical gameplay flow

```
Game starts
    ↓
MultiplayerBootstrap loads SdkConfig
    ↓
Core validates configuration
    ↓
SDK services initialize
    ↓
Game calls QuickMatchAsync
    ↓
Matchmaking searches or creates a session
    ↓
Session service joins the session
    ↓
Player service spawns the configured player prefab
    ↓
RPC service activates registered events
    ↓
Voice service connects if enabled
    ↓
SDK reports Ready / InSession
```

## What the developer actually writes

```csharp
await MultiplayerSDK.InitializeAsync();

MatchResult result = await MultiplayerSDK.Matchmaking.QuickMatchAsync();

MultiplayerSDK.Events.Broadcast(NetworkEventId.PlayerReady);

MultiplayerSDK.Voice.SetMicrophoneMuted(true);

await MultiplayerSDK.Session.LeaveAsync();
```

Everything above this — connection handling, retries, spawn timing,
authority setup, voice lifecycle — happened inside the services layer.
The developer still fully controls UI, button behavior, player visuals,
game rules, game-specific state, and can still drop down to native Photon
`[Rpc]` methods or `[Networked]` properties whenever the simplified API
isn't enough.