# Public API Specification

## Purpose

This document defines the public API exposed by the Unity Multiplayer SDK.

The primary goal is to provide a simple, consistent and production-ready developer experience while hiding repetitive networking infrastructure.

The SDK should feel intuitive, predictable and easy to integrate into any Unity multiplayer project.

---

# Design Principles

The public API should be:

- Simple
- Consistent
- Discoverable
- Async-first
- Modular
- Extensible
- Strongly typed
- Game-agnostic
- Safe by default

---

# Root SDK Entry Point

The SDK exposes a single entry point.

```csharp
MultiplayerSDK.Instance
```

or

```csharp
Sdk.Instance
```

The root object provides access to all SDK modules.

Example:

```csharp
Sdk.Instance.Session
Sdk.Instance.Matchmaking
Sdk.Instance.Player
Sdk.Instance.Voice
Sdk.Instance.Events
Sdk.Instance.Rpc
Sdk.Instance.Validation
Sdk.Instance.Diagnostics
Sdk.Instance.Config
```

---

# Initialization

## Initialize SDK

```csharp
await Sdk.InitializeAsync();
```

---

## Shutdown SDK

```csharp
await Sdk.ShutdownAsync();
```

---

# Session Module

Responsibilities

- Create Session
- Join Session
- Join By Code
- Leave Session
- Reconnect
- Current Session
- Session State

Methods

```csharp
CreateAsync()

JoinAsync()

JoinByCodeAsync()

LeaveAsync()

ReconnectAsync()

GetCurrentSession()

IsConnected()
```

Events

```csharp
OnConnected

OnDisconnected

OnSessionCreated

OnSessionJoined

OnSessionLeft
```

---

# Matchmaking Module

Responsibilities

- Quick Match
- Create Match
- Search Match
- Cancel Search

Methods

```csharp
QuickMatchAsync()

FindMatchAsync()

CreateMatchAsync()

CancelAsync()
```

Events

```csharp
OnSearchStarted

OnMatchFound

OnSearchCancelled

OnMatchFailed
```

---

# Player Module

Responsibilities

- Spawn
- Despawn
- Local Player
- Remote Players
- Ownership
- Player Registry

Methods

```csharp
SpawnAsync()

DespawnAsync()

GetLocalPlayer()

GetPlayers()

FindPlayer()
```

Events

```csharp
OnPlayerSpawned

OnPlayerDespawned

OnPlayerJoined

OnPlayerLeft
```

---

# Voice Module

Responsibilities

- Connect
- Disconnect
- Mute
- Unmute
- Push To Talk
- Speaker State

Methods

```csharp
ConnectAsync()

DisconnectAsync()

Mute()

Unmute()

EnablePushToTalk()

DisablePushToTalk()

IsMuted()
```

Events

```csharp
OnVoiceConnected

OnVoiceDisconnected

OnPlayerStartedSpeaking

OnPlayerStoppedSpeaking
```

---

# RPC Module

Responsibilities

- Reliable RPC
- Unreliable RPC
- Target Player
- Broadcast
- Authority Validation

Methods

```csharp
Broadcast()

Send()

SendReliable()

SendUnreliable()
```

---

# Network Events Module

Responsibilities

Generic lightweight events.

Methods

```csharp
Raise()

Register()

Unregister()
```

Example Events

- Emoji
- Ping
- Ready
- Reaction
- Custom

---

# Validation Module

Responsibilities

Project validation.

Methods

```csharp
ValidateProject()

ValidateScene()

ValidatePrefab()

ValidateVoice()

ValidateNetwork()

ValidateRunner()
```

---

# Diagnostics Module

Responsibilities

Problem reporting.

Methods

```csharp
RunDiagnostics()

GetIssues()

Clear()

ExportReport()
```

---

# Auto Fix Module

Responsibilities

Automatic repair.

Methods

```csharp
FixIssue()

FixSelected()

FixAllSafeIssues()
```

---

# Configuration Module

Responsibilities

SDK configuration.

Methods

```csharp
Load()

Save()

Reload()

Reset()
```

---

# Logging Module

Methods

```csharp
Debug()

Info()

Warning()

Error()
```

---

# Network Sync Module

Responsibilities

Player synchronization.

Methods

```csharp
EnableSync()

DisableSync()

RegisterNetworkObject()

UnregisterNetworkObject()
```

---

# Spawn Module

Methods

```csharp
SpawnPlayer()

DespawnPlayer()

RespawnPlayer()
```

---

# Scene Module

Responsibilities

Scene lifecycle.

Methods

```csharp
LoadSceneAsync()

UnloadSceneAsync()

SynchronizeScene()
```

---

# Result Pattern

Every async method returns

```csharp
SdkResult<T>
```

Example

```csharp
SdkResult<SessionInfo>

SdkResult<PlayerInfo>

SdkResult<MatchResult>
```

---

# Error Categories

- Configuration
- Validation
- Session
- Matchmaking
- Player
- Voice
- RPC
- Network
- Scene
- Package
- Unknown

---

# Event Naming Rules

Use past tense.

Examples

```text
OnPlayerJoined

OnPlayerLeft

OnMatchFound

OnSessionCreated

OnVoiceConnected
```

---

# Async Rules

All network operations must be asynchronous.

Never block the Unity main thread.

---

# Thread Safety

Only Unity objects may execute on the Unity main thread.

Background operations must never directly modify Unity objects.

---

# Extensibility

Developers should be able to replace:

- Matchmaking
- Logging
- Player Spawner
- Event Serializer
- Voice Provider (future)
- Backend Adapter (future)

without modifying SDK source code.

---

# API Design Rules

- One responsibility per module.
- No duplicate APIs.
- No hidden side effects.
- Explicit method names.
- Strongly typed parameters.
- Async for all network operations.
- Consistent naming across modules.
- Backward compatible whenever possible.

---

# Final Rule

The SDK should allow a developer to build a multiplayer game without repeatedly writing Photon infrastructure code, while remaining flexible enough to support different gameplay architectures.