# OpenGame Architecture

## 1. Purpose

OpenGame is a reusable framework for building Roblox games with a modular, maintainable, and scalable architecture.

The framework must remain independent from the rules of any specific game. Game-specific behavior lives outside OpenGame and is loaded through the framework's public API.

The first integration used to validate this architecture is `TreadmillGame`.

---

## 2. Architectural Principle

The dependency direction is:

```text
Roblox Engine
     ↑
OpenGame Framework
     ↑
Game Implementation
```

A game may depend on OpenGame. OpenGame must never depend on a specific game.

This allows the same framework to support future projects such as racing, combat, tycoon, survival, or simulation games without changing the framework for each project.

---

## 3. OpenGame 0.1.0 Baseline

The current validated baseline introduces a public engine object with the following responsibilities:

```lua
OpenGame.new()
OpenGame:RegisterService(name, service)
OpenGame:GetService(name)
OpenGame:RegisterModule(module)
OpenGame:GetModule(name)
OpenGame:LoadGame(gameModule)
OpenGame:Initialize()
OpenGame:Start()
OpenGame:Shutdown()
```

The engine keeps deterministic registration order for services and modules.

The lifecycle is:

```text
Register
   ↓
Initialize
   ↓
Start
   ↓
Shutdown
```

Shutdown occurs in reverse order so dependent systems can release resources before the systems they depend on are closed.

---

## 4. Current Roblox Structure

```text
ReplicatedStorage
├── OpenGame
│   ├── Core
│   │   └── OpenGame
│   └── Services
│       └── PlayerService
│
└── TreadmillGame
    ├── Game
    ├── Config
    │   └── Player
    └── Modules
        └── Training
            ├── Module
            ├── Services
            │   ├── TrainingService
            │   └── TreadmillService
            ├── GameObjects
            │   └── Treadmill
            └── Config
                └── Treadmills

ServerScriptService
└── Server
    └── Bootstrap

Workspace
└── Map
    └── Treadmill
        ├── Belt
        ├── TreadmillBase
        ├── Handle
        ├── SupportLeft
        └── SupportRight
```

The physical treadmill stays in `Workspace` because it is an active world object.

The `Treadmill` ModuleScript in `GameObjects` contains reusable behavior for one treadmill instance.

---

## 5. Bootstrap Responsibilities

`Bootstrap` must remain small.

Its current responsibilities are:

1. load the OpenGame engine class;
2. create the engine instance;
3. register framework-level services;
4. load the selected game;
5. start the engine;
6. forward Roblox server shutdown to `engine:Shutdown()`.

Conceptually:

```lua
local engine = OpenGame.new()

engine:RegisterService("PlayerService", PlayerService)
engine:LoadGame(TreadmillGame)
engine:Start()
```

Bootstrap does not register `TrainingService` or `TreadmillService` directly.

That responsibility belongs to `TreadmillGame` and its modules.

---

## 6. Game Entry Point

Every game integrated with OpenGame should expose one entry ModuleScript.

For the first game:

```text
TreadmillGame
└── Game
```

The game entry point declares metadata such as:

```lua
Game.Name = "TreadmillGame"
Game.Version = "0.1.0"
```

It also owns registration of the game's modules.

Example responsibility:

```text
TreadmillGame.Game
        ↓
Training Module
        ↓
TrainingService
TreadmillService
```

OpenGame therefore knows that a game has been loaded, but it does not need to know the internal feature list of that game.

---

## 7. Functional Modules

Game-specific systems are grouped by business capability rather than by global technical type.

Example:

```text
Training
├── Module
├── Services
│   ├── TrainingService
│   └── TreadmillService
├── GameObjects
│   └── Treadmill
└── Config
    └── Treadmills
```

The module entry point is responsible for registering its own services with the engine.

This prevents a global `Services` directory from growing indefinitely as the game gains more features.

Future modules may include:

```text
Shop
Racing
Gym
Pets
Quests
```

Each module should own its services, configuration, GameObjects, events, and other feature-specific resources.

---

## 8. Services

Services coordinate systems and shared gameplay state.

### Framework services

Framework services must be generic.

The current example is `PlayerService`.

`PlayerService` is responsible for generic player initialization but does not define treadmill-specific statistics.

### Game services

Game-specific services belong to their functional module.

Current examples:

- `TrainingService` applies training rewards;
- `TreadmillService` discovers and manages treadmill instances.

Services should coordinate behavior rather than contain all implementation details of every world object.

---

## 9. Generic Player Configuration

Player statistics are defined by the game, not by OpenGame.

OpenGame's `PlayerService` accepts a configuration supplied by the loaded game.

Current `TreadmillGame` configuration defines:

```text
Coins
Strength
Level
```

Conceptually:

```lua
PlayerConfig.Stats = {
    {
        Name = "Coins",
        Type = "IntValue",
        Default = 0,
        Leaderstat = true,
    },
}
```

This means another game can define completely different statistics without modifying `PlayerService`.

For example:

```text
RacingGame
├── Money
├── Wins
├── Cars
└── Reputation
```

This is an important boundary of the framework.

---

## 10. GameObjects

A GameObject represents behavior for one concrete world instance.

The current implemented example is:

```text
Training/GameObjects/Treadmill
```

A treadmill object receives:

- the Roblox model;
- treadmill configuration;
- the required training service.

It is responsible for behavior associated with that single treadmill instance, including:

- locating the belt;
- detecting players;
- moving the belt;
- applying configured training behavior through `TrainingService`;
- cleaning up event connections.

`TreadmillService` manages the collection of treadmill GameObjects.

The distinction is:

```text
TreadmillService
= manages many treadmill instances

Treadmill GameObject
= controls one treadmill instance
```

---

## 11. Configuration-Driven Game Objects

Treadmill behavior is driven by game configuration instead of duplicated scripts.

A physical treadmill model can expose an attribute such as:

```text
Type = Basic
```

The game then resolves the matching configuration from:

```text
Training/Config/Treadmills
```

Example categories may define values such as:

```text
Speed
Coins
Strength
```

This allows multiple treadmill variants to reuse the same behavior implementation.

---

## 12. Workspace vs ReplicatedStorage

The project follows a strict distinction between code and active world instances.

### Workspace

Contains active Roblox objects that exist in the running world.

Example:

```text
Workspace/Map/Treadmill
```

### ReplicatedStorage

Contains reusable code, configuration, module definitions, and shared resources.

Example:

```text
ReplicatedStorage/TreadmillGame/Modules/Training/GameObjects/Treadmill
```

The same name may therefore appear in both places, but the responsibilities are different:

```text
Workspace/Map/Treadmill
= physical model

GameObjects/Treadmill
= reusable ModuleScript behavior
```

---

## 13. Lifecycle Ordering

OpenGame preserves service registration order.

The current validated service order is:

```text
1. PlayerService
2. TrainingService
3. TreadmillService
```

This ensures framework-level player infrastructure starts before game systems that may depend on player state.

The engine uses explicit ordered arrays in addition to lookup tables so startup order does not depend on Lua table iteration order.

---

## 14. Current Startup Flow

The validated startup path is:

```text
Roblox Server
    ↓
Bootstrap
    ↓
OpenGame.new()
    ↓
Register PlayerService
    ↓
Load TreadmillGame
    ↓
TreadmillGame registers Training module
    ↓
Training registers TrainingService
    ↓
Training registers TreadmillService
    ↓
OpenGame.Initialize()
    ↓
OpenGame.Start()
    ↓
TreadmillService discovers Workspace treadmill
    ↓
Treadmill GameObject starts
```

This flow has been exercised successfully in Roblox Studio with the treadmill active and player statistics created from game configuration.

---

## 15. Design Rules

1. OpenGame contains only reusable framework behavior.
2. OpenGame must never depend on a specific game implementation.
3. A game exposes one entry module that OpenGame can load.
4. The game entry module owns registration of game modules.
5. Functional modules own their own services, GameObjects, configuration, and events.
6. Framework services must not hard-code game-specific rules or statistics.
7. Services coordinate systems; GameObjects control individual instances.
8. Active physical objects live in `Workspace`.
9. Reusable behavior and configuration live outside `Workspace`.
10. Startup and shutdown order must be deterministic.
11. Bootstrap remains minimal and contains no gameplay rules.
12. New framework features should be extracted from real reusable needs rather than designed speculatively.

---

## 16. Status of OpenGame 0.1.0

The following foundation is currently implemented and validated:

- public `OpenGame` engine object;
- service registration and lookup;
- module registration and lookup;
- game loading through `LoadGame`;
- deterministic service startup order;
- initialize/start/shutdown lifecycle;
- generic `PlayerService` configuration;
- `TreadmillGame` game entry point;
- `Training` functional module;
- `TrainingService`;
- `TreadmillService`;
- `Treadmill` GameObject;
- configuration-driven treadmill types;
- separation between framework and game-specific player statistics.

Components discussed but not yet considered part of the 0.1.0 implementation include systems such as EventBus, Logger, persistence, networking abstractions, generic zones, AI, UI infrastructure, and other future engine capabilities.

---

## 17. Next Direction

The next development should validate the current abstractions with real gameplay requirements instead of expanding the framework prematurely.

A good next sequence is:

```text
1. stabilize current Training/Treadmill implementation
2. add a second world interaction such as a running track
3. identify which behavior is truly reusable
4. extract generic world abstractions only when justified
5. add persistence after player progression rules are stable
```

The architectural target remains:

```text
OpenGame = reusable framework
TreadmillGame = first game using OpenGame
Future games = separate implementations using the same framework
```
