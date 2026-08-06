# OpenGame Vision

## Purpose

OpenGame is a reusable framework for Roblox games.

Its purpose is to provide a stable foundation for common game systems without coupling the framework to one specific game.

## Core idea

OpenGame is the engine layer.

Each game is a separate implementation built on top of OpenGame.

Examples:

- OpenGame: reusable framework.
- TreadmillGame: training and running game.
- RacingGame: vehicle and racing game.
- ZombieGame: survival game.

## Principles

1. OpenGame must not depend on a specific game.
2. Game-specific code may depend on OpenGame.
3. Features are grouped into functional modules.
4. Services coordinate systems.
5. GameObjects represent active objects in the world.
6. Configuration is separated from behavior.
7. Server authority is preferred for progression and rewards.
8. The framework should remain simple enough for small games and scalable enough for larger projects.

## Initial scope

The first version focuses on:

- Bootstrap and lifecycle.
- Service registration and startup.
- Modular organization.
- World objects.
- Player lifecycle.
- Shared configuration.
- Server and client separation.

The first reference game is TreadmillGame.
