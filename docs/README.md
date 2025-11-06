# CaveDroid Developer Documentation

Welcome to the CaveDroid developer documentation! This guide will help you understand the codebase and start contributing to the project.

## What is CaveDroid?

CaveDroid is a 2D, Minecraft-inspired exploration and building game written in Kotlin using the LibGDX framework. The game features:

- Procedurally generated worlds with horizontal looping
- Mining, crafting, and block placement
- Survival and Creative game modes
- Mob spawning and AI
- Day/night cycles with dynamic lighting
- Cross-platform support (Android, Desktop, iOS)

## Documentation Index

### Getting Started
- **[Quick Start Guide](QUICK_START.md)** - Set up your development environment and run the game
- **[Architecture Overview](ARCHITECTURE.md)** - Understand the code structure and game flow

### Developer Guides
- **[Adding New Blocks](guides/ADDING_BLOCKS.md)** - Step-by-step guide for creating new blocks
- **[Adding New Items](guides/ADDING_ITEMS.md)** - How to create new items and tools
- **[Adding New Mobs](guides/ADDING_MOBS.md)** - Create NPCs with custom behaviors
- **[Modifying Game Mechanics](guides/MODIFYING_MECHANICS.md)** - Change core gameplay systems

## Technology Stack

- **Language:** Kotlin (with some Java legacy code)
- **Game Framework:** LibGDX 1.12.0
- **Physics Engine:** Box2D with Box2DLights
- **Dependency Injection:** Dagger 2
- **Build System:** Gradle with Kotlin DSL
- **Serialization:** kotlinx-serialization (JSON & Protobuf)
- **Target Platform:** Java 17

## Project Structure

```
cavedroid/
├── core/                      # Shared game logic (multi-platform)
│   ├── common/                # Global utilities and interfaces
│   ├── data/                  # Data layer (serialization, repositories)
│   ├── domain/                # Domain models and interfaces
│   ├── entity/                # Game entity definitions
│   ├── game/                  # Game logic controllers
│   ├── gameplay/              # Core gameplay systems
│   └── gdx/                   # LibGDX integration
├── android/                   # Android platform implementation
├── desktop/                   # Desktop (LWJGL3) platform implementation
├── ios/                       # iOS platform implementation
├── assets/                    # Game assets (textures, sounds, configs)
└── buildSrc/                  # Build configuration
```

## Quick Links

### Core Files to Know
- `CaveDroidApplication.kt` - Application entry point
- `GameProc.java` - Main game loop
- `GameWorld.kt` - World management and physics
- `GameRenderer.kt` - Rendering pipeline
- `GameInputProcessor.kt` - Input handling

### Asset Locations
- **Textures:** `/assets/textures/`
- **Block/Item Definitions:** `/assets/json/`
- **Sounds:** `/assets/sfx/`
- **Localization:** `/assets/i18n/`

## Contributing

Before making changes:

1. Read the [Architecture Overview](ARCHITECTURE.md) to understand the system design
2. Follow the relevant guide for the type of change you're making
3. Test your changes on at least one platform (desktop recommended for development)
4. Run `./gradlew ktlintFormat` before committing
5. Ensure the project builds: `./gradlew desktop:dist`

## Getting Help

- Check the documentation in this `/docs` directory
- Look at existing implementations for reference
- Review comments in the code for implementation details

## License

See the LICENSE file in the root directory.
