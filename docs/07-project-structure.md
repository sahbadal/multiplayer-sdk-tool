# Project structure

## Recommended folder layout

```
Assets/
└── SDK/
    ├── Runtime/
    │   ├── Core/
    │   ├── Configuration/
    │   ├── Session/
    │   ├── Player/
    │   ├── Matchmaking/
    │   ├── RPC/
    │   ├── Voice/
    │   ├── Validation/
    │   ├── Logging/
    │   └── PublicAPI/
    │
    ├── Editor/
    │   ├── Windows/
    │   ├── ConfigEditors/
    │   ├── PrefabSetup/
    │   ├── SceneSetup/
    │   ├── Validators/
    │   ├── Diagnostics/
    │   └── CodeGeneration/
    │
    ├── Resources/
    │   └── SDKConfig/
    │       ├── SdkConfig.asset
    │       ├── SessionConfig.asset
    │       ├── PlayerConfig.asset
    │       ├── MatchmakingConfig.asset
    │       ├── RpcConfig.asset
    │       └── VoiceConfig.asset
    │
    ├── NetworkEvents/
    ├── Documentation/     
    ├── Samples/
    └── Tests/
```

`Runtime/` and `Editor/` mirror the module list from
`02-module-reference.md` and `03-editor-tooling.md` one-to-one — a
developer looking for the Matchmaking service code goes straight to
`Runtime/Matchmaking/`.

## Note on `Resources/SDKConfig/`

`Resources.Load` auto-discovery is the convenient default, but it isn't
the only supported layout — see production rule 4 in `01-architecture.md`
("recommended, not mandatory"). The preferred long-term approach is a
direct serialized `SdkConfig` reference on `MultiplayerBootstrap`, with
`Resources.Load` used only as a fallback when no explicit reference is
set. This keeps `Resources/` from becoming a hard dependency for projects
with stricter asset-bundling or addressables setups.
