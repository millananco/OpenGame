# OpenGame Core

## Responsibility

Core contains the minimum infrastructure required to start and coordinate the framework.

Core must remain independent from game-specific concepts such as treadmills, shops, races, weapons or quests.

## Proposed structure

```text
OpenGame
└── Core
    ├── Bootstrap
    ├── ServiceManager
    ├── Logger
    ├── Configuration
    └── Lifecycle
```

## ServiceManager

The ServiceManager registers framework and game services, initializes them and starts them in a controlled order.

Recommended lifecycle:

```lua
service:Init(context)
service:Start()
service:Stop()
```

`Init` resolves dependencies and configuration. `Start` connects events and begins runtime work. `Stop` releases resources when supported.

## Rules

- Core cannot require modules from a specific game.
- Core should expose small, explicit APIs.
- Startup errors must identify the failing service.
- Duplicate service names must be rejected.
- Services should not silently start twice.
- Startup order should eventually be deterministic rather than relying on `pairs()`.

## Current implementation

The current prototype includes `ServiceManager:Register()` and `ServiceManager:Start()`.

The next evolution is to add deterministic ordering and an `Init` phase without breaking the current game.
