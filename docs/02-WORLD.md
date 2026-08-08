# OpenGame World

## Purpose

World defines reusable concepts and runtime components for objects that exist or operate inside the Roblox world.

It does not define the concrete objects of a particular game.

## Current structure

```text
OpenGame
└── World
    └── PlayerZone
```

`PlayerZone` is the first concrete reusable World component implemented in OpenGame.

The broader World model may evolve toward concepts such as `GameObject`, `Machine`, `Zone`, `Item`, and `Actor`, but those abstractions are not considered implemented until they exist in code.

## PlayerZone

`PlayerZone` encapsulates detection of players interacting with a Roblox `BasePart` used as a zone.

It exists to prevent game-specific GameObjects from duplicating `Touched` and `TouchEnded` handling.

### Construction

```lua
local zone = PlayerZone.new(part)
```

The supplied object must be a Roblox `BasePart`.

### Public API

```lua
zone:Start()
zone:IsPlayerInside(player)
zone:GetPlayers()
zone:RemovePlayer(player)
zone:Destroy()
```

### Start

`Start()` connects the zone to the `Touched` and `TouchEnded` events of the underlying part.

Calling `Start()` more than once does not create duplicate listeners.

### Player detection

A Roblox character consists of multiple physical parts. A simple boolean based directly on `Touched` and `TouchEnded` can therefore report that a player left a zone when only one body part stopped touching it.

`PlayerZone` maintains a touch count per player.

When a character part touches the zone, the count increases. When a character part stops touching it, the count decreases. The player is removed from the zone only when the count reaches zero.

Conceptually:

```text
Character part touches
        ↓
Resolve Player
        ↓
Increment touch count
        ↓
Player considered inside

Character part leaves
        ↓
Decrement touch count
        ↓
Count == 0 ?
   ├── No  → player remains inside
   └── Yes → player leaves zone
```

### IsPlayerInside

```lua
zone:IsPlayerInside(player)
```

Returns whether the supplied player is currently considered inside the zone.

### GetPlayers

```lua
local players = zone:GetPlayers()
```

Returns the players currently detected inside the zone.

The returned table is a new list and does not expose the internal player table directly.

### RemovePlayer

```lua
zone:RemovePlayer(player)
```

Explicitly removes a player and clears the player's touch count.

This is useful when a player leaves the server or when an owning GameObject needs to clean up its state.

### Destroy

```lua
zone:Destroy()
```

Stops the component, disconnects its Roblox event connections, and clears its internal state.

A World component that owns event connections must release those connections when destroyed.

## Current use in TreadmillGame

`PlayerZone` belongs to OpenGame because player-in-zone detection is reusable and does not contain treadmill-specific rules.

The concrete training objects remain inside TreadmillGame:

```text
TreadmillGame
└── Modules
    └── Training
        └── GameObjects
            ├── Treadmill
            └── RunningTrack
```

Both objects use the same framework component:

```text
Treadmill
    ↓
PlayerZone(Belt)

RunningTrack
    ↓
PlayerZone(Track)
```

`Treadmill` remains responsible for treadmill behavior such as belt movement and requesting training rewards.

`RunningTrack` remains responsible for running-track behavior such as checking whether a detected player is moving and requesting running rewards.

`PlayerZone` knows nothing about coins, strength, distance, treadmills, tracks, or training.

## Framework boundary

The dependency direction is:

```text
TreadmillGame GameObjects
          ↓
OpenGame.World.PlayerZone
          ↓
Roblox BasePart events
```

OpenGame must not depend on `TreadmillGame`.

This allows the same `PlayerZone` component to be reused later by systems such as shops, race starts, safe zones, teleport areas, gyms, capture areas, or other game-specific features.

## Future World concepts

The architecture reserves room for reusable concepts such as:

```text
World
├── GameObject
├── Machine
├── Zone
├── PlayerZone     ← implemented
├── Item
└── Actor
```

These names describe architectural directions, not current implementation claims.

Reusable abstractions should be added only when concrete game requirements justify them.

## Design rules

1. World contains reusable world-level behavior, not game rules.
2. Concrete game objects remain in their owning game module.
3. PlayerZone operates on a generic `BasePart`.
4. PlayerZone does not grant rewards or modify progression.
5. GameObjects compose PlayerZone rather than duplicating player-contact detection.
6. Runtime event connections must be released through `Destroy()`.
7. OpenGame never depends on a concrete game implementation.
