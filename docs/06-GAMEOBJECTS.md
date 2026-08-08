# OpenGame GameObjects

## Purpose

A GameObject wraps one active object in the Roblox world and owns its object-specific behavior.

The current concrete GameObjects are implemented by TreadmillGame, not OpenGame.

```text
TreadmillGame
└── Modules
    └── Training
        └── GameObjects
            ├── Treadmill
            └── RunningTrack
```

## Service and GameObject separation

A service discovers, validates, creates, tracks, and removes GameObjects.

A GameObject controls one concrete runtime object.

```text
TreadmillService
      ↓
Treadmill

RunningTrackService
      ↓
RunningTrack
```

This keeps services focused on coordination and GameObjects focused on per-instance behavior.

## Treadmill

The `Treadmill` GameObject wraps one treadmill model in `Workspace/Map`.

Its current responsibilities are:

- store the treadmill model and selected configuration;
- resolve the `Belt` part;
- create a reusable `OpenGame.World.PlayerZone` over the Belt;
- move the Belt using configured Speed;
- obtain players currently inside the Belt zone;
- request Coins, Strength, and XP rewards from `TrainingService`;
- remove a player when requested by the owning service;
- stop loops and destroy its PlayerZone during cleanup.

Conceptually:

```text
Workspace/Map/Treadmill
        ↓
Treadmill GameObject
        ├── Belt movement
        ├── PlayerZone(Belt)
        └── TrainingService:AddTraining(...)
```

The Treadmill does not directly own progression formulas. It requests rewards from `TrainingService`.

## RunningTrack

The `RunningTrack` GameObject wraps one running-track model in `Workspace/Map`.

Its current responsibilities are:

- store the track model and selected configuration;
- resolve the `Track` part;
- create a reusable `PlayerZone` over the Track;
- obtain players currently inside the zone;
- check whether a detected player is actually moving through the Humanoid `MoveDirection`;
- request Coins, Distance, and XP rewards from `TrainingService`;
- remove players when requested by the owning service;
- destroy its PlayerZone and stop runtime work during cleanup.

Conceptually:

```text
Workspace/Map/RunningTrack
        ↓
RunningTrack GameObject
        ├── PlayerZone(Track)
        ├── movement check
        └── TrainingService:AddRunningTraining(...)
```

## Shared PlayerZone composition

Both current GameObjects reuse the same framework component:

```text
OpenGame
└── World
    └── PlayerZone
```

This removes duplicated `Touched` and `TouchEnded` handling from game-specific objects.

```text
Treadmill     ─┐
               ├──> PlayerZone
RunningTrack  ─┘
```

`PlayerZone` only answers world-presence questions. It does not know about treadmill speed, running rewards, Coins, Strength, Distance, XP, or Level.

## Configuration-driven behavior

GameObjects receive configuration selected by their owning service.

The runtime model identifies its type through a Roblox attribute such as:

```text
Type = Basic
```

The service resolves the matching configuration and injects it into the GameObject constructor.

Typical construction is:

```lua
local object = Treadmill.new(model, config, trainingService)
object:Start()
```

or:

```lua
local object = RunningTrack.new(model, config, trainingService)
object:Start()
```

This means behavior code remains the same while balance changes through configuration.

## Runtime model versus behavior code

The physical model belongs in Workspace because it exists in the active Roblox world.

The reusable ModuleScript belongs in ReplicatedStorage under the owning game module.

```text
Workspace
└── Map
    ├── Treadmill
    └── RunningTrack

ReplicatedStorage
└── TreadmillGame
    └── Modules
        └── Training
            └── GameObjects
                ├── Treadmill
                └── RunningTrack
```

A `GameObjects` folder should not contain the live physical model merely because the object is called a GameObject.

## Lifecycle

The current GameObjects implement:

```lua
Start()
RemovePlayer(player)
Destroy()
```

### Start

Starts runtime behavior and the composed PlayerZone.

### RemovePlayer

Removes one player from internal zone state. This is used when a player leaves the server.

### Destroy

Stops runtime loops and delegates cleanup to the composed `PlayerZone`.

Future GameObjects may add an explicit `Stop()` lifecycle operation if pause/resume behavior becomes a real requirement.

## Dependencies

Current GameObjects receive game-specific dependencies such as `TrainingService` through construction.

They resolve the reusable `PlayerZone` from OpenGame because it is framework infrastructure.

The intended dependency direction is:

```text
GameObject
   ├── game configuration
   ├── game service
   └── OpenGame reusable World components
```

OpenGame never imports TreadmillGame GameObjects.

## Future base contract

OpenGame may later provide a common `GameObject` base if several concrete objects demonstrate repeated needs such as:

- connection tracking;
- lifecycle state;
- cancellation of tasks;
- common cleanup;
- model validation.

That base should only be introduced after the duplication is proven by real implementations.
