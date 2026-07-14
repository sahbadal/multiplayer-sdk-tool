# Editor workflow (edit-time)

## Full flow

```
Developer imports Photon Fusion and Photon Voice
        ↓
Developer imports the Multiplayer SDK
        ↓
Developer opens Tools → Multiplayer SDK → Setup
        ↓
SDK validates package dependencies
        ↓
Developer enters Photon App IDs
        ↓
SDK creates configuration assets
        ↓
Developer creates their own player prefab
        ↓
Developer assigns the player prefab
        ↓
SDK validates the prefab
        ↓
Developer reviews and applies required components
        ↓
Developer configures session defaults
        ↓
Developer configures matchmaking
        ↓
Developer registers common RPC events
        ↓
Developer configures voice
        ↓
Developer installs scene bootstrap
        ↓
SDK runs complete validation
        ↓
Developer uses simple SDK APIs from game code
```

## Step by step

1. **Dependencies first.** Fusion and Voice packages must already be in
   the project — the SDK checks for them rather than installing them.
2. **Import the SDK.** Package or UPM dependency added on top.
3. **Open the Setup Window.** `Tools → Multiplayer SDK → Setup` — the
   single entry point described in `03-editor-tooling.md`.
4. **Dependency validation.** The window confirms both Photon packages
   are present before letting the developer proceed.
5. **App IDs.** Pasted from the Photon dashboard, written directly into
   Photon's own settings assets (see the source-of-truth table).
6. **Config assets created.** `SdkConfig`, `SessionConfig`,
   `PlayerConfig`, `MatchmakingConfig`, `RpcConfig`, `VoiceConfig` are
   generated in the recommended folder if missing.
7. **Player prefab.** Developer brings their own prefab and assigns it.
8. **Prefab validation.** SDK inspects it and lists what's missing.
9. **Review and apply.** Developer sees the exact proposed change and
   approves it — nothing is added silently.
10. **Session, matchmaking, RPC, voice config.** Filled in through their
    respective tabs — each writes to its own config asset.
11. **Scene bootstrap installed.** One `MultiplayerBootstrap` added to
    the active scene.
12. **Full validation.** Every check across dependencies, config, prefab,
    and scene runs together, with a pass/fail summary.
13. **Ready.** From here on, the developer only writes calls against
    `MultiplayerSDK.*` — no further editor setup unless a value changes.