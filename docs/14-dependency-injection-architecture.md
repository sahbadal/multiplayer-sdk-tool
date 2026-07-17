# Dependency Injection Architecture

## Purpose

This document defines how Dependency Injection (DI) using **VContainer** will be integrated into the Unity Multiplayer SDK.

The objective is to build a modular, maintainable and testable architecture by decoupling SDK modules from each other and from Photon-specific implementations.

The SDK should rely on interfaces rather than concrete implementations wherever possible.

---

# Design Goals

The dependency injection architecture should provide:

- Loose coupling
- Modular services
- Easy testing
- Service replacement
- Centralized initialization
- Automatic dependency resolution
- Clear service ownership
- Predictable object lifetimes
- Easy future expansion

---

# Why VContainer?

The SDK contains multiple independent modules:

- Session
- Matchmaking
- Player
- Voice
- RPC
- Events
- Validation
- Diagnostics
- Logging
- Configuration

Without DI, every service would manually create and manage its own dependencies, resulting in tightly coupled code.

VContainer allows these services to receive their dependencies automatically, making the SDK easier to maintain and extend.

---

# High-Level Architecture

```

Developer Code
│
▼

SDK Public API
│
▼

SDK Services
│
▼

Interfaces
│
▼

VContainer
│
▼

Photon Fusion / Photon Voice

```

---

# Root LifetimeScope

The SDK owns a single root LifetimeScope.

```

SdkLifetimeScope

```

Responsibilities:

- Register all SDK services
- Register configuration assets
- Register logging
- Register diagnostics
- Register Photon adapters
- Resolve dependencies
- Initialize SDK

Only one root LifetimeScope should exist.

---

# Service Registration

Each SDK module registers its own services.

```

CoreInstaller
SessionInstaller
PlayerInstaller
MatchmakingInstaller
VoiceInstaller
RpcInstaller
ValidationInstaller

```

Each installer registers only the services belonging to its module.

This keeps registrations modular and maintainable.

---

# Configuration Registration

Configuration assets are registered as instances.

Example:

```

SdkConfig
SessionConfig
PlayerConfig
MatchmakingConfig
VoiceConfig
RpcConfig

```

Services receive these configurations through constructor injection.

Services should never search for configuration assets manually.

---

# Service Lifetimes

## Singleton

Used for services that exist for the entire SDK lifetime.

Examples:

- Logger
- Config Provider
- Diagnostics
- Validation
- NetworkRunner Manager

---

## Scoped

Used for services that belong to an active multiplayer session.

Examples:

- Session Service
- Matchmaking State
- Player Registry
- Voice Session

---

## Transient

Used for lightweight temporary objects.

Examples:

- Validators
- Command Objects
- Request Builders
- Result Builders

---

# Constructor Injection

Constructor injection is the preferred injection method.

Example:

```

SessionService
├── Logger
├── Runner Service
├── Config
└── Matchmaking

```

Dependencies should always be explicit.

Field injection should be avoided unless required for Unity components.

---

# MonoBehaviour Injection

MonoBehaviours requiring SDK services use method injection.

Example:

```

MultiplayerBootstrap

```

receives:

- Session Service
- Runner Service
- Voice Service

through VContainer.

Gameplay scripts should not resolve services manually.

---

# Service Dependencies

Example hierarchy:

```

Session Service
│
├── Runner Service
├── Logger
├── Config
└── Diagnostics

```

Voice Service

```

Voice Service
│
├── Runner Service
├── Config
└── Logger

```

Validation Service

```

Validation Service
│
├── Config
├── Logger
└── Diagnostics

```

---

# Initialization Order

The SDK initializes services in the following order:

1. Load configuration
2. Initialize logger
3. Register services
4. Resolve dependencies
5. Validate project
6. Create NetworkRunner
7. Initialize Session
8. Initialize Matchmaking
9. Initialize Voice
10. SDK Ready

This order guarantees that all required services are available before gameplay begins.

---

# Runtime Ownership

The SDK owns:

- Dependency Container
- NetworkRunner
- Session lifecycle
- Voice lifecycle
- Diagnostics
- Validation

The game owns:

- Gameplay
- Character controllers
- UI
- Animations
- Combat
- Missions

---

# Interfaces

Every major service should expose an interface.

Examples:

- ISessionService
- IMatchmakingService
- IPlayerService
- IVoiceService
- IRpcService
- IEventService
- ILogger
- IValidationService

The public SDK should depend only on these interfaces.

---

# Photon Adapters

Photon-specific implementations remain internal.

Example:

```

ISessionService
        │
        ▼
PhotonSessionService

```

The rest of the SDK communicates only through interfaces.

This allows future replacement without affecting other modules.

---

# Optional Modules

Modules such as Voice should support optional registration.

If Voice is disabled:

```

IVoiceService

```

resolves to a lightweight no-op implementation rather than null.

This removes unnecessary null checks.

---

# Dependency Rules

Services may depend on:

- Interfaces
- Configuration
- Logger
- Diagnostics

Services should NOT depend directly on:

- Scene objects
- MonoBehaviour singletons
- Static classes
- Other concrete services

---

# Lifetime Rules

Singleton services should never depend on scoped services.

Scoped services may depend on singleton services.

Transient services may depend on singleton or scoped services.

---

# Service Resolution

Services should never manually resolve dependencies from the container.

All dependencies should be injected automatically.

Avoid:

```
Container.Resolve<T>()
```

outside composition/setup code.

---

# Testing Strategy

Because services depend on interfaces rather than implementations:

- Mock services can replace production services.
- Unit tests do not require Photon.
- Business logic can be tested independently.

---

# Extensibility

Developers should be able to replace:

- Matchmaking
- Logging
- Player Spawner
- Validation
- Event Serialization

without modifying SDK source code.

---

# Folder Structure

```
Runtime/
    Core/
    Session/
    Player/
    Matchmaking/
    Voice/
    Rpc/
    Events/
    Validation/

Editor/
    Setup/
    Validation/
    Diagnostics/

Installers/
    CoreInstaller.cs
    SessionInstaller.cs
    PlayerInstaller.cs
    MatchmakingInstaller.cs
    VoiceInstaller.cs

Configs/
    SdkConfig.asset
```

---

# Final Design Rule

The Dependency Injection container is responsible for **creating, configuring, connecting and disposing SDK services**.

Individual services should never create or own their dependencies directly.

The result is a modular, testable and scalable architecture that remains independent of gameplay implementation while providing a clean foundation for multiplayer infrastructure.
