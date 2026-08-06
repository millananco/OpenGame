# OpenGame World

## Purpose

World defines reusable concepts for objects that exist or operate inside the Roblox world.

It does not define the concrete objects of a particular game.

## Base concepts

```text
World
├── GameObject
├── Machine
├── Zone
├── Item
└── Actor
```

### GameObject

Base runtime object associated with a Roblox Instance or Model.

Typical responsibilities:

- Store the underlying model.
- Manage lifecycle state.
- Own event connections.
- Release resources through Destroy.

### Machine

A GameObject with operational behavior and state.

Examples in specific games include a treadmill, generator, elevator or production machine.

### Zone

A bounded area that detects players or objects entering, remaining inside and leaving.

Examples include a training track, safe area or capture area.

### Item

A collectible, usable or transferable object.

Examples include a coin, key or consumable.

### Actor

An active world participant with behavior.

Examples include an NPC, pet or companion.

## Framework boundary

OpenGame defines reusable contracts and common behavior.

Concrete implementations remain inside each game:

```text
TreadmillGame
└── Modules
    └── Training
        └── GameObjects
            ├── Treadmill
            └── RunningTrack
```

## Lifecycle

World objects should support Start, Stop and Destroy.

Destroy must disconnect events, stop loops and clear references.
