# OpenGame Services

## Purpose

Services coordinate systems that live for most or all of the game session.

A service should represent a clear capability, not merely a folder for unrelated functions.

## Framework services

Framework-level services are reusable across games.

Examples:

- Player lifecycle.
- Logging.
- Configuration.
- Save abstraction.
- Networking abstraction.

## Game services

Game-specific services belong to the game module that owns the feature.

```text
TreadmillGame
└── Modules
    └── Training
        └── Services
            ├── TrainingService
            ├── TreadmillService
            └── RunningTrackService
```

## Responsibilities

A service may:

- Coordinate multiple GameObjects.
- Connect Roblox events.
- Apply business rules.
- Expose a stable API to other modules.
- Manage runtime collections.

A service should not contain all behavior of every object it manages. Object-specific behavior belongs in GameObjects.

## Lifecycle

Recommended contract:

```lua
function Service:Init(context)
end

function Service:Start()
end

function Service:Stop()
end
```

## Rules

- One clear responsibility per service.
- No hidden dependency lookup when explicit injection is possible.
- Server services own authoritative rewards and progression.
- Services must guard against duplicate startup.
- Event connections and loops must be cleaned up when stopped.
