# OpenGame Services

## Purpose

Services coordinate long-lived capabilities and authoritative game rules.

A service should represent one clear responsibility. Framework services belong to OpenGame; game-specific services remain inside the game module that owns them.

## Current framework service

```text
OpenGame
└── Services
    └── PlayerService
```

### PlayerService

`PlayerService` is generic. It creates player values from configuration supplied by the loaded game.

It does not know about treadmill-specific statistics such as Coins, Strength, Distance, XP, or Level.

The game applies its configuration through:

```lua
playerService:SetConfig(playerConfig)
```

The current TreadmillGame player configuration defines:

```text
Coins
Strength
Distance
XP
Level
```

This keeps the framework reusable for games with completely different player statistics.

## Training services

The current Training module contains:

```text
TreadmillGame
└── Modules
    └── Training
        └── Services
            ├── TrainingService
            ├── TreadmillService
            └── RunningTrackService
```

### TrainingService

`TrainingService` owns authoritative training rewards and progression updates.

Current public operations include:

```lua
TrainingService:AddTraining(player, coins, strength, xp)
TrainingService:AddRunningTraining(player, coins, distance, xp)
TrainingService:AddXP(player, amount)
```

Responsibilities include:

- adding Coins and Strength for treadmill training;
- adding Coins and Distance for running-track training;
- adding XP;
- checking level-up thresholds;
- carrying excess XP into subsequent levels;
- updating Level on the server.

The service does not hard-code the XP progression formula. It consumes the game configuration from `Training/Config/Progression`.

### TreadmillService

`TreadmillService` discovers treadmill models under `Workspace/Map`, validates each model and its configured type, creates one `Treadmill` GameObject per active model, and owns the runtime collection of those objects.

It also handles models added or removed while the game is running and removes leaving players from managed GameObjects.

### RunningTrackService

`RunningTrackService` performs the equivalent role for running tracks.

It:

- discovers models containing a `Track` part;
- reads the model `Type` attribute;
- resolves the matching `RunningTracks` configuration;
- creates a `RunningTrack` GameObject;
- manages runtime addition and removal;
- removes players when they leave the server.

## Service and GameObject boundary

Services coordinate multiple instances. Object-specific runtime behavior belongs in GameObjects.

```text
TreadmillService
      ↓
Treadmill instances

RunningTrackService
      ↓
RunningTrack instances
```

For example, `TreadmillService` does not directly implement belt movement. The `Treadmill` GameObject owns that behavior.

Likewise, `RunningTrackService` does not determine whether an individual player is physically inside a track; `RunningTrack` composes the reusable `OpenGame.World.PlayerZone` component.

## Lifecycle

OpenGame currently controls services through the lifecycle methods they implement:

```lua
Initialize(engine)
Start()
Shutdown()
```

Not every service currently implements every lifecycle method.

Services are started in deterministic registration order. This is important because framework services such as `PlayerService` must be available before dependent game services begin operating.

Shutdown proceeds in reverse service order so dependent systems can release resources before their dependencies are closed.

## Authority

Training rewards and progression are server-authoritative.

Client code must not directly grant:

- Coins;
- Strength;
- Distance;
- XP;
- Level.

Future networking must send player intent to the server, where services validate and apply changes.

## Design rules

1. One clear responsibility per service.
2. Framework services remain game-agnostic.
3. Game-specific services live inside their owning functional module.
4. Services coordinate GameObjects instead of absorbing all object behavior.
5. Progression and reward mutations remain server-authoritative.
6. Balance values belong in configuration modules rather than service code.
7. Registration order defines startup order.
8. Runtime collections and event connections must be cleaned up when their owning service or GameObject is destroyed.
