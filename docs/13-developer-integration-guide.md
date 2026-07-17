# Developer Integration Guide

## Purpose

This document explains how a developer integrates the Unity Multiplayer SDK into a new or existing Unity project.

The objective is to minimize setup time, eliminate repetitive networking configuration, and provide a consistent multiplayer workflow.

---

# Prerequisites

Before importing the SDK, ensure the following requirements are met:

- Unity version supported by the SDK
- Photon Fusion installed
- Photon Voice installed
- Valid Photon App IDs
- A player prefab
- Basic understanding of Unity scenes

---

# Installation

## Step 1 - Import Photon Packages

Import the required Photon packages:

- Photon Fusion
- Photon Voice

---

## Step 2 - Import Multiplayer SDK

Import the Unity Multiplayer SDK package.

After importing, the following menu becomes available:

```

Tools
└── Multiplayer SDK

```

---

# First Time Setup

Open:

```

Tools
→ Multiplayer SDK
→ Setup

```

The Setup Window will automatically detect whether the project has already been configured.

If no configuration exists, the SDK will create the required project assets automatically.

Generated assets:

```

Assets/
MultiplayerSDK/
Config/
SdkConfig.asset
SessionConfig.asset
PlayerConfig.asset
MatchmakingConfig.asset
RpcConfig.asset
VoiceConfig.asset

```

The developer does **not** need to manually create these assets.

---

# Configure Photon

Inside the Setup Window:

Enter:

- Fusion App ID
- Voice App ID
- Default Region

The SDK validates the configuration before saving.

---

# Configure Player

Assign the player prefab.

The SDK scans the prefab and checks:

- NetworkObject
- NetworkTransform
- SDK components
- Voice components
- Required references

If anything is missing:

The Setup Window displays a validation report.

Example:

```

Player Prefab Validation

✓ Animator

✓ Character Controller

✗ NetworkObject

✗ NetworkTransform

✗ SDKPlayer

```

Click:

```

Apply Fixes

```

The SDK adds supported components automatically.

Gameplay scripts are never modified.

---

# Configure Voice

Assign:

- Recorder
- Speaker Prefab (optional)
- Voice Settings

The SDK validates:

- Recorder
- Speaker
- AudioSource
- FusionVoiceClient

Missing components can be fixed automatically where supported.

---

# Configure Session

Configure default session options:

- Session Name
- Max Players
- Public / Private
- Join By Code
- Allow Reconnect

These values become the project defaults.

They can still be overridden in code.

---

# Configure Matchmaking

Configure:

- Default Region
- Match Type
- Max Players
- Auto Create Room
- Search Timeout
- Retry Policy

These act as default matchmaking settings.

---

# Configure RPC

Configure:

- Event IDs
- Reliability defaults
- Payload limits
- Validation rules

Game-specific RPCs remain the responsibility of the developer.

---

# Configure Validation

Select which validations run automatically.

Example:

- Project Validation
- Scene Validation
- Prefab Validation
- Voice Validation
- Runner Validation

Recommended:

Run all validations before entering Play Mode.

---

# Install Scene Bootstrap

Click:

```

Install Multiplayer Bootstrap

```

The SDK creates:

```

MultiplayerBootstrap

```

Only one Bootstrap exists in the project.

The Bootstrap owns:

- SDK initialization
- NetworkRunner lifecycle
- Session initialization
- Voice initialization

The developer should never manually duplicate this object.

---

# Run Diagnostics

Click:

```

Run Diagnostics

```

The SDK scans:

- Project
- Scene
- Player Prefab
- Voice
- NetworkRunner
- Configurations
- Photon setup

Issues are grouped by severity.

Example:

```

Critical

Error

Warning

Information

```

Each issue includes:

- Description
- Cause
- Recommended Fix
- Auto Fix availability

---

# Auto Fix

Click:

```

Fix All Safe Issues

```

The SDK repairs supported issues automatically.

Examples:

✓ Missing NetworkObject

✓ Missing SDK components

✓ Missing Config Assets

✓ Missing Bootstrap

✓ Missing Recorder

The SDK never modifies gameplay logic automatically.

---

# Enter Play Mode

If validation succeeds:

The SDK initializes automatically.

Initialization order:

1. Load Config
2. Initialize SDK
3. Create NetworkRunner
4. Initialize Voice
5. Register Services
6. Ready

---

# Using the SDK

Example:

Initialize

```csharp
await Sdk.InitializeAsync();
```

Create Session

```csharp
await Sdk.Session.CreateAsync();
```

Join Session

```csharp
await Sdk.Session.JoinAsync();
```

Quick Match

```csharp
await Sdk.Matchmaking.QuickMatchAsync();
```

Leave Session

```csharp
await Sdk.Session.LeaveAsync();
```

Mute Voice

```csharp
Sdk.Voice.Mute();
```

Send Network Event

```csharp
Sdk.Events.Raise("Emoji", emojiId);
```

Run Validation

```csharp
Sdk.Validation.ValidateProject();
```

Run Diagnostics

```csharp
Sdk.Diagnostics.RunDiagnostics();
```

---

# Existing Projects

The SDK supports integration into existing Unity projects.

Workflow:

1. Import SDK
2. Open Setup Window
3. Assign Player Prefab
4. Configure Photon
5. Run Diagnostics
6. Apply Safe Fixes
7. Resolve Manual Issues
8. Test Multiplayer

The SDK never overwrites gameplay scripts.

---

# Updating the SDK

When updating:

- Existing configs are preserved
- Validation runs automatically
- New optional settings are added
- Deprecated settings generate warnings

---

# Best Practices

✔ Use a single NetworkRunner.

✔ Keep one MultiplayerBootstrap.

✔ Always validate before Play Mode.

✔ Keep gameplay separate from networking.

✔ Use Session APIs instead of raw Photon calls where possible.

✔ Use SDK Event APIs for lightweight events.

✔ Keep configuration centralized.

✔ Fix validation warnings early.

✔ Use Auto Fix only for supported issues.

✔ Keep gameplay systems independent from SDK internals.

---

# What the SDK Handles

- SDK Initialization
- Project Configuration
- Session Lifecycle
- Matchmaking
- NetworkRunner Lifecycle
- Voice Integration
- Player Registration
- Player Spawn Workflow
- RPC Helpers
- Network Events
- Validation
- Diagnostics
- Auto Fix
- Logging
- Configuration Management

---

# What Developers Implement

The SDK does NOT create game-specific functionality.

Developers remain responsible for:

- Character Movement
- Gameplay Logic
- Combat
- Inventory
- UI
- Menus
- Animations
- AI
- Missions
- Backend
- Economy
- Leaderboards
- Custom RPC Logic
- Game Rules
- Visual Effects

---

# Final Workflow

```
Import SDK
        │
        ▼
Open Setup Window
        │
        ▼
SDK Creates Config Assets
        │
        ▼
Configure Photon
        │
        ▼
Assign Player Prefab
        │
        ▼
Configure Voice
        │
        ▼
Configure Session
        │
        ▼
Configure Matchmaking
        │
        ▼
Run Diagnostics
        │
        ▼
Apply Safe Fixes
        │
        ▼
Install Bootstrap
        │
        ▼
Press Play
        │
        ▼
Call SDK APIs
        │
        ▼
Build Multiplayer Game
```

---

# Design Goal

A developer should be able to integrate multiplayer into a Unity project in **under 10 minutes** without manually configuring Photon infrastructure, while still retaining full control over gameplay implementation.