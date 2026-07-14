# Roadmap

## What's deliberately deferred

Not missing by accident — explicitly out of scope for this version, see
`00-overview.md` for the full non-goals list. The short version:
authentication, inventory/currency, friends/parties, skill-based backend
matchmaking, lobby UI, player visuals, game rules, VR-specific
head/hand sync, text chat, custom backend allocation, and a full
replacement of the Fusion API.

## Phases to go from this spec to shipped code

**1. Research**
- Unity Editor scripting patterns for the Setup Window
- ScriptableObject custom editor patterns for the config tabs
- How to safely read/write Photon's own settings assets from the Setup
  Window (for App ID fields)
- Safe prefab editing from editor code (add-component, save, without
  corrupting prefab overrides)
- Network prefab registration API in the current Fusion version
- RPC/event code generation approach (for stable `NetworkEventId` values)
- Photon Voice Recorder/Speaker mapping API

**2. Design**
- Concrete class interfaces for each service (`SessionService`,
  `PlayerService`, etc.)
- `SdkResult` / `SdkError` type design
- Event payload contract rules (versioning, size limits)
- Test strategy — what's unit-testable in isolation vs. what needs a
  Photon test environment

**3. Implementation**
- Core module
- Session module
- Player module
- Matchmaking module
- RPC module
- Voice module
- Editor tooling
- Validation system
- Samples
- Documentation pass against the shipped API (this doc set becomes the
  first draft, not the final word)

## How to use this roadmap

Each phase gates the next — design shouldn't start until research answers
the open Unity/Fusion API questions above, and implementation shouldn't
start until the class interfaces in phase 2 are agreed. Treat every other
doc in this set (`01` through `07`) as the target shape to build toward,
not as a description of code that already exists.