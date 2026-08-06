# OpenGame Bootstrap

## Purpose

Bootstrap is the composition root of the application.

It knows which framework services and game services must be registered. Individual services should not be responsible for starting the entire application.

## Startup flow

```text
ServerScriptService/Bootstrap
        |
        v
OpenGame Core
        |
        v
Register framework services
        |
        v
Register game services
        |
        v
Init all services
        |
        v
Start all services
```

## Current prototype

The current game registers PlayerService, TrainingService and TreadmillService through ServiceManager.

## Target design

Bootstrap should eventually provide a context containing shared dependencies:

```lua
local context = {
    Framework = OpenGame,
    Game = TreadmillGame,
    Services = ServiceManager,
}
```

Each service receives this context during Init.

## Rules

- Keep Bootstrap small.
- Do not place gameplay logic in Bootstrap.
- Make startup order deterministic.
- Fail clearly when a required module is missing.
- Start the server before enabling client-dependent interactions.
- Avoid silently waiting forever for incorrectly named folders.

## Server and client

The server uses a server Bootstrap in ServerScriptService.

The client should use a separate ClientBootstrap in StarterPlayerScripts for controllers, interfaces and local effects.
