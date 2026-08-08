# OpenGame Modules

## Purpose

A module groups one functional area and keeps its services, GameObjects, configuration, events, shared code, and client pieces together.

OpenGame provides the module registration mechanism, while concrete game modules live in the game implementation.

## Current Training module

The first implemented game module is `Training` inside TreadmillGame.

```text
TreadmillGame
└── Modules
    └── Training
        ├── Module
        ├── Services
        │   ├── TrainingService
        │   ├── TreadmillService
        │   └── RunningTrackService
        ├── GameObjects
        │   ├── Treadmill
        │   └── RunningTrack
        └── Config
            ├── Treadmills
            ├── RunningTracks
            └── Progression
```

Not every module needs every possible folder. Only create folders when they contain real responsibilities.

## Module entry point

Each functional module exposes a `Module` ModuleScript.

The Training module currently declares:

```lua
TrainingModule.Name = "Training"
```

and registers its own services through:

```lua
function TrainingModule:Register(engine)
    -- registers TrainingService
    -- registers TreadmillService
    -- registers RunningTrackService
end
```

This prevents the top-level server Bootstrap from knowing the internal services of the Training module.

## Game entry point and module loading

`TreadmillGame/Game` acts as the game-level manifest and entry point.

It is responsible for:

- identifying the game name and version;
- applying game-specific player configuration;
- locating game modules;
- registering the Training module through OpenGame.

The dependency chain is:

```text
Bootstrap
   ↓
OpenGame
   ↓
TreadmillGame/Game
   ↓
Training/Module
   ↓
Training services
```

Bootstrap does not register `TrainingService`, `TreadmillService`, or `RunningTrackService` directly.

## Training configuration

### Treadmills

`Training/Config/Treadmills` defines treadmill types and balance values such as:

- reward Coins;
- reward Strength;
- reward XP;
- belt Speed.

Runtime treadmill models select their configuration through a `Type` attribute such as:

```text
Type = Basic
```

### RunningTracks

`Training/Config/RunningTracks` defines values such as:

- CoinsPerSecond;
- DistancePerSecond;
- XPPerSecond.

Running-track models also select a type through a `Type` attribute.

### Progression

`Training/Config/Progression` owns the XP progression rule.

The current implementation exposes:

```lua
Progression:GetRequiredXP(level)
```

with a configurable base XP value.

This keeps progression balance outside `TrainingService`.

## Current player configuration boundary

Player stat definitions do not live in OpenGame.

They currently belong to:

```text
TreadmillGame
└── Config
    └── Player
```

The current game defines:

```text
Coins
Strength
Distance
XP
Level
```

`PlayerService` in OpenGame only knows how to create values described by this configuration.

## Dependency rules

- A game module may depend on OpenGame.
- OpenGame must not depend on a game module.
- A module owns its feature-specific services, GameObjects, and balance configuration.
- Modules should use public APIs or injected dependencies when practical.
- Circular dependencies are not allowed.
- Generic behavior discovered through real reuse should move to OpenGame.

An example of extracted reusable behavior is `OpenGame.World.PlayerZone`, which is used by both `Treadmill` and `RunningTrack`.

## Ownership rules

Training currently owns:

- treadmill behavior;
- running-track behavior;
- training rewards;
- XP earned through training;
- level progression resulting from training;
- treadmill and running-track balance configuration.

Future features such as shops, racing, pets, or inventory should become separate modules when their responsibilities are distinct enough to justify their own boundary.

## Configuration rule

Game balance values should live in Config modules rather than being duplicated inside behavior code.

Examples include:

- speed;
- rewards;
- XP rates;
- progression thresholds;
- prices;
- level requirements;
- cooldowns.

This allows game tuning without rewriting service or GameObject logic.
