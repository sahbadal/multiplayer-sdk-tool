# Architecture Design Decisions

## Purpose

This document captures all architectural decisions, design questions, implementation rules and future considerations before development begins.

The goal is to ensure the SDK remains generic, reusable, maintainable and suitable for multiple multiplayer game genres.

---

# 1. SDK Philosophy

## Primary Goal

Provide a production-ready multiplayer infrastructure that minimizes repetitive Photon setup while remaining flexible for different game genres.

The SDK should never force a specific gameplay implementation.

---

## Core Principles

- Convention over configuration
- Single Source of Truth
- Editor-first workflow
- Validation before runtime
- Safe Auto-Fix
- Modular architecture
- Game-agnostic design
- Extensible APIs
- Backward compatibility
- Minimal developer setup

---

# 2. Project Installation

Questions

- How is the SDK imported?
- What folders are created?
- Which assets are automatically generated?
- Which assets remain templates?
- How are updates handled?

Decision

- Unity Package Manager package
- Config assets generated automatically
- Editor tooling installed automatically
- Runtime assets isolated from package assets

---

# 3. Configuration System

Questions

- Where does configuration live?
- How many config assets exist?
- Which config owns which settings?
- Can developers replace configs?
- How are configs loaded?

Decision

- SdkConfig is the root configuration
- Module-specific configs remain separate
- Single Source of Truth
- No duplicated configuration

---

# 4. Scene Bootstrap

Questions

- How is MultiplayerBootstrap installed?
- Can multiple bootstraps exist?
- Should Bootstrap survive scene loads?

Decision

- Exactly one bootstrap
- Persistent across scenes
- Validated before Play Mode

---

# 5. NetworkRunner

Questions

- Who creates the runner?
- Who owns the runner?
- Can developers manually create one?
- How are duplicate runners detected?

Decision

- SDK owns the runner lifecycle
- Single runner policy
- Validation prevents duplicates

---

# 6. Player Prefab

Questions

- How is the player prefab selected?
- Can multiple prefabs exist?
- Which components are required?
- Which components are optional?
- How are prefabs validated?

Decision

- Explicit assignment
- Validation before runtime
- Auto-Fix supported

---

# 7. Player Lifecycle

Questions

- Spawn
- Despawn
- Respawn
- Reconnect
- Ownership
- Player mapping

Decision

- SDK manages lifecycle
- Gameplay manages behavior

---

# 8. Player Movement

Questions

- Should SDK implement movement?
- Which movement systems are supported?
- How is input enabled?
- How are remote proxies handled?

Decision

SDK only manages:

- ownership
- authority
- synchronization
- lifecycle

Game implements movement.

---

# 9. Network Synchronization

Questions

- Position sync
- Rotation sync
- Scale sync
- Rigidbody sync
- Character sync
- Custom state sync
- Interpolation
- Prediction
- Tick rate
- Interest Management
- Bandwidth optimization

Decision

SDK validates synchronization.

Game defines gameplay state.

---

# 10. Authority

Questions

- Input Authority
- State Authority
- Ownership transfer
- Spawn authority
- Destroy authority

Decision

SDK validates authority.

Gameplay decides ownership rules.

---

# 11. RPC System

Questions

- Generic events
- Custom RPC
- Reliable
- Unreliable
- Rate limiting
- Serialization
- Payload validation

Decision

SDK provides wrappers.

Game provides gameplay RPCs.

---

# 12. Event System

Questions

Should these exist?

- Emoji
- Ping
- Reactions
- Ready
- Custom Events
- Notifications

Decision

SDK provides a lightweight event framework.

---

# 13. Voice

Questions

- Recorder location
- Speaker location
- Push to Talk
- Auto Connect
- Mute
- Spatial Audio
- Voice Channels
- Voice Reconnect

Decision

SDK manages lifecycle.

Game manages UI.

---

# 14. Session System

Questions

- Create
- Join
- Leave
- Reconnect
- Join by Code
- Public
- Private

Decision

SDK owns session lifecycle.

---

# 15. Matchmaking

Questions

- Quick Match
- Manual Match
- Skill
- Region
- Party
- Filters
- Timeout
- Retry
- Auto Create
- Cancel

Decision

SDK provides generic matchmaking.

Game supplies match criteria.

---

# 16. Scene Management

Questions

- Single Scene
- Multi Scene
- Additive
- Scene Loading
- Scene Synchronization

Decision

SDK manages networking lifecycle only.

---

# 17. Validation

Questions

What should be validated?

- Configs
- Prefabs
- Scenes
- Voice
- RPC
- Network
- Runner
- Matchmaking

Decision

Everything required before runtime.

---

# 18. Auto Fix

Questions

Which issues are safe?

Decision

Auto Fix only for deterministic changes.

Gameplay changes never automatic.

---

# 19. Diagnostics

Questions

How are problems reported?

Decision

Severity

- Info
- Warning
- Error
- Critical

Every issue includes:

- Description
- Cause
- Fix
- Auto Fix availability

---

# 20. Error Handling

Questions

Exceptions?

Error Codes?

Decision

SDKResult pattern.

No exceptions for expected runtime failures.

---

# 21. Logging

Questions

Should logging exist?

Decision

Central logging service.

Log Levels

- Debug
- Info
- Warning
- Error

---

# 22. Extensibility

Questions

Can developers replace services?

Decision

Interfaces everywhere.

Dependency Injection supported.

---

# 23. Testing

Required

- Unit Tests
- Validation Tests
- Multiplayer Tests
- Voice Tests
- Session Tests
- Matchmaking Tests

---

# 24. Package Structure

Questions

Runtime

Editor

Samples

Documentation

Tests

Decision

UPM compliant package.

---

# 25. Versioning

Questions

How are updates handled?

Decision

Semantic Versioning.

---

# 26. Performance

Questions

- Memory
- GC
- RPC Count
- Sync Count
- Tick Cost
- Voice Cost

Decision

Performance diagnostics included.

---

# 27. Security

Questions

Authority validation

RPC validation

Cheat prevention

Decision

SDK validates infrastructure.

Gameplay validates rules.

---

# 28. Future Expansion

Potential Modules

- Dedicated Servers
- PlayFab
- Steam
- EOS
- Vivox
- NGO Adapter
- Mirror Adapter
- Replay System
- Spectator Mode
- Analytics
- Metrics
- Cloud Save

---

# 29. Final Design Rule

The SDK owns infrastructure.

The game owns gameplay.

Never mix the two.