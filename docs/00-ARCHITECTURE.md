# OpenGame Architecture

## 1. Purpose

OpenGame is a reusable framework for building Roblox games with a modular, maintainable, and scalable architecture.

Its main objective is to separate:

- reusable framework infrastructure;
- game-specific features;
- Roblox engine objects and assets;
- server and client responsibilities.

OpenGame must not contain logic tied to a single game. A treadmill, a gym, a race, or a training system belongs to the game that uses the framework, not to the framework itself.

---

## 2. Core Principle

The architecture is divided into two main layers:

```text
OpenGame Framework
        ↓
Game Implementation
```

### OpenGame Framework

Contains reusable technical capabilities that can be used by any Roblox project.

Examples:

- service lifecycle;
- dependency registration;
- event system;
- logging;
- configuration;
- persistence abstractions;
- networking abstractions;
- shared utilities.

### Game Implementation

Contains the rules, systems, entities, configuration, and assets that belong to a specific game.

Examples for a treadmill game:

- training;
- treadmills;
- gyms;
- races;
- shops;
- player progression;
- game-specific statistics.

---

## 3. Proposed Roblox Structure

```text
ReplicatedStorage
├── OpenGame
│   ├── Core
│   │   ├── ServiceManager
│   │   ├── EventBus
│   │   ├── Logger
│   │   └── Configuration
│   │
│   ├── Services
│   │   ├── PlayerLifecycleService
│   │   ├── DataService
│   │   └── NetworkService
│   │
│   ├── Shared
│   │   ├── Types
│   │   ├── Constants
│   │   └── Utils
│   │
│   └── Remotes
│
└── TreadmillGame
    ├── Modules
    │   ├── Player
    │   │   ├── Services
    │   │   ├── Config
    │   │   └── Shared
    │   │
    │   ├── Training
    │   │   ├── Services
    │   │   ├── GameObjects
    │   │   ├── Config
    │   │   └── Events
    │   │
    │   ├── Shop
    │   │   ├── Services
    │   │   ├── Config
    │   │   └── Events
    │   │
    │   └── Racing
    │       ├── Services
    │       ├── GameObjects
    │       ├── Config
    │       └── Events
    │
    ├── Shared
    └── Remotes

ServerScriptService
└── Server
    ├── Bootstrap
    └── ServerConfig

StarterPlayer
└── StarterPlayerScripts
    └── ClientBootstrap

Workspace
└── Map
    ├── Treadmills
    ├── Gyms
    ├── RaceTracks
    └── SpawnLocations
```

---

## 4. Framework Responsibilities

OpenGame should provide infrastructure, not game rules.

### Core

The `Core` layer contains the fundamental building blocks of the framework.

#### ServiceManager

Responsible for:

- registering services;
- resolving services;
- controlling initialization order;
- starting and stopping services;
- preventing duplicate registrations;
- reporting startup errors.

A service lifecycle may evolve toward:

```lua
service:Init(context)
service:Start()
service:Stop()
```

#### EventBus

Provides decoupled communication between systems.

Example:

```lua
EventBus:Publish("PlayerTrainingStarted", player, treadmillId)
```

Consumers subscribe without direct dependency on the publisher.

#### Logger

Provides consistent logging levels:

- Debug;
- Info;
- Warning;
- Error;
- Critical.

#### Configuration

Provides centralized framework configuration and validation.

---

## 5. Game Module Responsibilities

Each game-specific feature should be organized as a functional module.

Example:

```text
Training
├── Services
│   ├── TrainingService
│   └── TreadmillService
├── GameObjects
│   └── Treadmill
├── Config
│   └── Treadmills
├── Events
│   └── TrainingEvents
└── Shared
    └── TrainingTypes
```

A module owns everything related to its business capability.

This prevents a global `Services` folder from becoming overcrowded and makes each feature easier to understand, move, test, and maintain.

---

## 6. Services and GameObjects

### Services

Services coordinate systems and manage shared state.

Examples:

- `TrainingService` calculates rewards;
- `TreadmillService` discovers and manages treadmill instances;
- `ShopService` handles purchases;
- `RaceService` coordinates races.

A service should not directly contain every detail of every physical object.

### GameObjects

GameObjects wrap Roblox models and give them behavior.

Example:

```lua
local treadmill = Treadmill.new(model, config)
treadmill:Start()
```

A `Treadmill` GameObject may be responsible for:

- validating the model structure;
- locating the belt;
- controlling belt movement;
- detecting players;
- exposing treadmill state;
- cleaning up connections.

The corresponding service manages multiple treadmill objects.

---

## 7. Configuration-Driven Design

Game behavior should be configured rather than duplicated in scripts.

Example:

```lua
return {
    Basic = {
        DisplayName = "Basic",
        CoinsPerSecond = 1,
        StrengthPerSecond = 2,
        Speed = 8,
        RequiredLevel = 1,
        Price = 0,
    },

    Professional = {
        DisplayName = "Professional",
        CoinsPerSecond = 5,
        StrengthPerSecond = 8,
        Speed = 15,
        RequiredLevel = 10,
        Price = 500,
    },
}
```

A treadmill model only needs an attribute such as:

```text
Type = Basic
```

The code reads the matching configuration automatically.

---

## 8. Server and Client Separation

### Server

The server is authoritative for:

- player data;
- rewards;
- purchases;
- progression;
- game rules;
- anti-cheat validation;
- persistence.

### Client

The client is responsible for:

- user interface;
- local visual effects;
- local sounds;
- input handling;
- camera behavior;
- presentation of server state.

The client must never be trusted to directly grant currency, strength, experience, items, or purchases.

---

## 9. Dependency Direction

The dependency direction must remain clear:

```text
Game Modules
     ↓
OpenGame Framework
     ↓
Roblox Services
```

OpenGame must never depend on `TreadmillGame`.

A game may depend on OpenGame, but the framework must remain unaware of game-specific modules.

This rule is essential for reusability.

---

## 10. Bootstrap Process

The server bootstrap should initialize the framework first and the game second.

Example sequence:

```text
1. Load OpenGame Core
2. Register framework services
3. Register game services
4. Initialize services
5. Start services
6. Mark server as ready
```

Conceptual example:

```lua
ServiceManager:Register("PlayerLifecycleService", PlayerLifecycleService)
ServiceManager:Register("TrainingService", TrainingService)
ServiceManager:Register("TreadmillService", TreadmillService)

ServiceManager:Init()
ServiceManager:Start()
```

The bootstrap should remain small and should not contain game logic.

---

## 11. Initial Module Classification

### Belongs to OpenGame

- `ServiceManager`;
- `EventBus`;
- `Logger`;
- generic configuration utilities;
- generic player lifecycle handling;
- generic persistence abstractions;
- generic networking abstractions;
- shared validation and utility functions.

### Belongs to TreadmillGame

- `TrainingService`;
- `TreadmillService`;
- `Treadmill` GameObject;
- treadmill configuration;
- Coins, Strength, Level, and game progression rules;
- gyms;
- races;
- treadmill shops;
- treadmill-specific UI.

---

## 12. Design Rules

1. OpenGame contains only reusable functionality.
2. Game rules remain outside the framework.
3. Functional modules own their services, configuration, events, and GameObjects.
4. Server code is authoritative for gameplay state.
5. Configuration replaces repeated hard-coded values.
6. Services coordinate; GameObjects represent and control instances.
7. Modules communicate through explicit APIs or events.
8. Bootstrap remains minimal.
9. Dependencies point from the game toward the framework, never the opposite.
10. Each component must have one clear responsibility.

---

## 13. Initial Development Roadmap

### Phase 1 — Framework foundation

- create repository structure;
- implement `ServiceManager`;
- define service lifecycle;
- implement basic logging;
- create server bootstrap.

### Phase 2 — First game integration

- create `TreadmillGame` structure;
- move `TrainingService` into the game module;
- move `TreadmillService` into the game module;
- create `Treadmill` GameObject;
- validate configuration-driven treadmill creation.

### Phase 3 — Player progression

- define game statistics;
- add experience and level progression;
- add treadmill unlock requirements;
- add purchase validation.

### Phase 4 — Persistence and networking

- add `DataService` abstraction;
- save player progression;
- define RemoteEvents and RemoteFunctions;
- add server-side validation.

### Phase 5 — Reuse validation

- create a second small game module;
- verify that OpenGame can be reused without treadmill dependencies;
- extract any newly discovered generic components into the framework.

---

## 14. Architectural Goal

OpenGame should grow through real game requirements.

The framework should not attempt to implement every possible system in advance. New reusable components should be extracted only when they solve a real need shared by more than one game or module.

The intended result is:

```text
OpenGame = reusable game framework
TreadmillGame = first implementation using OpenGame
Future games = new implementations using the same framework
```

This separation is the foundation of the project.