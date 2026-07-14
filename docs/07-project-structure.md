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

## Where to save these docs

Put this whole doc set at:

```
Assets/SDK/Documentation/
    00-overview.md
    01-architecture.md
    02-module-reference.md
    03-editor-tooling.md
    04-editor-workflow.md
    05-runtime-flow.md
    06-validation-and-errors.md
    07-project-structure.md
    08-roadmap.md
```

Keeping them inside `Assets/SDK/` means they ship with the SDK package
itself — anyone who imports the SDK into a new project gets the docs
alongside the code, instead of the docs living in a separate repo that
can drift out of sync.

## Note on `Resources/SDKConfig/`

`Resources.Load` auto-discovery is the convenient default, but it isn't
the only supported layout — see production rule 4 in `01-architecture.md`
("recommended, not mandatory"). The preferred long-term approach is a
direct serialized `SdkConfig` reference on `MultiplayerBootstrap`, with
`Resources.Load` used only as a fallback when no explicit reference is
set. This keeps `Resources/` from becoming a hard dependency for projects
with stricter asset-bundling or addressables setups.
