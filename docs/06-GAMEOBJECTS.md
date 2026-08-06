# OpenGame GameObjects

## Purpose

A GameObject wraps one active object in the Roblox world and owns its object-specific behavior.

Examples in TreadmillGame include Treadmill and RunningTrack.

## Service and GameObject separation

A service discovers, creates and coordinates objects.

A GameObject controls one concrete object instance.

```text
TreadmillService
├── discovers treadmill models
├── validates configuration
└── creates Treadmill objects

Treadmill
├── controls one Belt
├── tracks active players
├── applies movement
└── releases its connections
```

## Recommended constructor

```lua
local object = Treadmill.new(model, config, dependencies)
object:Start()
```

Dependencies should be passed explicitly where practical instead of being searched globally inside every object.

## Required lifecycle

### Start

Connect events and begin runtime behavior.

### Stop

Pause behavior without necessarily destroying the model.

### Destroy

Stop loops, disconnect events, clear tables and release references.

## Runtime model versus code

The physical model used by players belongs in Workspace.

The reusable ModuleScript that controls that model belongs in the game module under GameObjects.

```text
Workspace/Map/Treadmill
    = active physical model

ReplicatedStorage/TreadmillGame/Modules/Training/GameObjects/Treadmill
    = reusable behavior code
```

## Future base contract

OpenGame may later provide a common GameObject base with connection management, state and cleanup helpers. This should be introduced only when multiple concrete objects demonstrate the same repeated behavior.
