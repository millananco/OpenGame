# OpenGame Modules

## Purpose

A module groups one functional area and keeps its code, configuration and runtime objects together.

## Recommended structure

```text
Modules
└── Training
    ├── Services
    ├── GameObjects
    ├── Config
    ├── Events
    ├── Shared
    └── Client
```

Not every module needs every folder. Only create folders that contain real responsibilities.

## Examples

```text
TreadmillGame
└── Modules
    ├── Training
    ├── Shop
    ├── Progression
    ├── Racing
    └── UI
```

## Dependency rules

- A game module may depend on OpenGame.
- OpenGame must not depend on a game module.
- Modules should depend on public APIs instead of internal files when possible.
- Circular dependencies are not allowed.
- Shared contracts should move to a neutral Shared or Core location.

## Module ownership

Training owns treadmill and running-track behavior.

Shop owns catalog, prices and purchases.

Progression owns levels, experience and unlock rules.

UI owns presentation and client interaction, but not authoritative rewards.

## Configuration

Game balance values belong in Config modules rather than being repeated inside behavior code.

Examples include speed, rewards, prices, level requirements and cooldowns.
