# CaveDroid Architecture Guide

This document explains how CaveDroid is structured, how code flows through the system, and how different components interact.

## Table of Contents

1. [Architecture Overview](#architecture-overview)
2. [Application Lifecycle](#application-lifecycle)
3. [Game Loop Flow](#game-loop-flow)
4. [Module Organization](#module-organization)
5. [Core Systems](#core-systems)
6. [Design Patterns](#design-patterns)
7. [Data Flow](#data-flow)

---

## Architecture Overview

CaveDroid follows a **Clean Architecture** pattern with clear separation between layers:

```
┌─────────────────────────────────────────┐
│     Presentation Layer (UI/Screens)     │  ← LibGDX screens, renderers
├─────────────────────────────────────────┤
│    Game Logic Layer (Controllers)       │  ← Entity controllers, world management
├─────────────────────────────────────────┤
│    Domain Layer (Models & Interfaces)   │  ← Entities, items, blocks, repositories
├─────────────────────────────────────────┤
│    Data Layer (Persistence & Assets)    │  ← Save files, JSON loading, caching
└─────────────────────────────────────────┘
```

### Module Dependencies

```
android/desktop/ios  →  core/gdx  →  core/gameplay  →  core/game  →  core/domain  →  core/common
                                  →  core/entity   →               →  core/data   →
```

**Dependency Rule:** Inner layers never depend on outer layers. Domain and entity modules are pure Kotlin with minimal external dependencies.

---

## Application Lifecycle

### Startup Flow

```
Platform Entry Point (AndroidLauncher / DesktopLauncher)
    ↓
Creates CaveDroidApplication (extends LibGDX Game)
    ↓
create() → Initializes Dependency Injection
    ↓
ApplicationComponent.build()
    ├─ Loads assets (textures, sounds, fonts)
    ├─ Reads JSON configs (blocks.json, items.json)
    ├─ Builds ItemsRepository
    ├─ Initializes AssetRepositories
    └─ Creates PreferencesStore
    ↓
Sets MenuScreen as active screen
    ↓
User navigates menu and starts game
    ↓
createGameComponent(StartGameConfig)
    ├─ Creates GameComponent (new DI scope)
    ├─ Loads or generates GameWorld
    ├─ Initializes entity controllers
    ├─ Sets up input processor
    └─ Creates GameProc
    ↓
Sets GameScreen as active screen
    ↓
Game loop begins (GameProc.update())
```

### Key Entry Points

**Android:** `/android/src/main/kotlin/ru/fredboy/cavedroid/AndroidLauncher.kt`
```kotlin
class AndroidLauncher : AndroidApplication() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        val config = AndroidApplicationConfiguration().apply {
            useAccelerometer = false
            useCompass = false
        }
        initialize(CaveDroidApplication(), config)
    }
}
```

**Desktop:** `/desktop/src/main/kotlin/ru/fredboy/cavedroid/desktop/DesktopLauncher.kt`
```kotlin
fun main(args: Array<String>) {
    // Parse command-line args (--touch, --debug, --verbose)
    val config = Lwjgl3ApplicationConfiguration()
    config.setWindowedMode(960, 540)
    config.setTitle("CaveDroid")
    Lwjgl3Application(CaveDroidApplication(), config)
}
```

**Common Core:** `/core/gdx/src/main/kotlin/ru/fredboy/cavedroid/gdx/CaveDroidApplication.kt`
```kotlin
class CaveDroidApplication : Game() {
    override fun create() {
        component = DaggerApplicationComponent.builder()
            .applicationModule(ApplicationModule(this))
            .build()
        // Set initial screen
        screen = component.menuScreen
    }
}
```

---

## Game Loop Flow

The game loop is managed by `GameProc` (`/core/gdx/src/main/java/ru/fredboy/cavedroid/gdx/game/GameProc.java`).

### Frame Update Order

Every frame (~60 FPS), the following happens in order:

```
GameScreen.render(delta)
    ↓
GameProc.update(delta)
    ├─ 1. GameWorld.update(delta)
    │       ├─ Update world time (day/night cycle)
    │       ├─ Box2D physics step (1/60s fixed timestep)
    │       └─ Update lighting (Box2DLights RayHandler)
    │
    ├─ 2. MobController.update(delta)
    │       ├─ Update player physics & state
    │       ├─ Update all NPC mobs
    │       ├─ Handle mob AI behaviors
    │       └─ Process mob health/damage
    │
    ├─ 3. DropController.update(delta)
    │       ├─ Update item drop physics
    │       ├─ Check player pickup radius
    │       └─ Add items to player inventory
    │
    ├─ 4. ProjectileController.update(delta)
    │       ├─ Update arrow/snowball trajectories
    │       └─ Handle collision damage
    │
    ├─ 5. GameInputProcessor.update(delta)
    │       ├─ Process keyboard/mouse/touch input
    │       ├─ Update cursor position
    │       ├─ Handle block breaking progress
    │       └─ Execute actions (place, use, attack)
    │
    ├─ 6. GameRenderer.render(delta)
    │       ├─ Update camera position
    │       ├─ Render world layers (background → foreground)
    │       ├─ Render entities (drops → mobs → projectiles)
    │       ├─ Render lighting and weather
    │       └─ Render HUD (hotbar, cursor, windows)
    │
    └─ 7. ContainerController.update(delta)
            └─ Update furnace smelting progress
```

### Scheduled Tasks (Timer-based)

In addition to frame updates, certain tasks run on fixed intervals:

| Task | Interval | Purpose |
|------|----------|---------|
| `GameWorldFluidsLogicControllerTask` | 100ms | Water/lava spreading |
| `GameWorldBlocksLogicControllerTask` | 100ms | Grass growth, sapling growth, crop growth |
| `GameWorldMobDamageControllerTask` | 500ms | Drowning, fire damage, fall damage |
| `GameWorldMobSpawnControllerTask` | 10s | Spawn new mobs if below limit |

These tasks are managed by `MutableTimer` and run independently of frame rate.

---

## Module Organization

### Core Modules (Platform-Independent)

#### **core/common**
- **Purpose:** Shared constants, interfaces, and utilities
- **Key Files:**
  - `Assets.kt` - Asset path constants
  - `Direction.kt` - Cardinal directions enum
  - `CommonLogger.kt` - Logging wrapper
- **No dependencies** on other core modules

#### **core/domain**
- **Purpose:** Business logic, models, and repository interfaces
- **Submodules:**
  - `domain/items` - Item and Block models (sealed classes)
  - `domain/assets` - Asset repository interfaces
  - `domain/world` - World constants (gravity, chunk size)
  - `domain/save` - Save data models
- **Dependencies:** Only `core/common`

#### **core/entity**
- **Purpose:** Game entity definitions (player, mobs, drops, projectiles)
- **Key Classes:**
  - `Player` - Player entity with inventory
  - `Mob` - Base mob class with physics
  - `Drop` - Item drops in world
  - `Projectile` - Arrows, snowballs
- **Dependencies:** `core/domain`, `core/common`

#### **core/data**
- **Purpose:** Data persistence, asset loading, repositories
- **Submodules:**
  - `data/assets` - Texture, sound, font loading
  - `data/items` - JSON deserialization for items/blocks
  - `data/save` - Save file I/O (world data)
  - `data/configuration` - Preferences storage
- **Dependencies:** `core/domain`, `core/common`

#### **core/game**
- **Purpose:** Game logic layer - controllers and world management
- **Submodules:**
  - `game/controller` - Entity lifecycle management (MobController, DropController)
  - `game/world` - GameWorld, world generation
  - `game/window` - UI window configuration
- **Dependencies:** `core/entity`, `core/domain`, `core/data`

#### **core/gameplay**
- **Purpose:** Gameplay systems (rendering, physics, input)
- **Submodules:**
  - `gameplay/rendering` - Renderer implementations
  - `gameplay/physics` - Physics tasks, block logic
  - `gameplay/controls` - Input handling and actions
- **Dependencies:** `core/game`, `core/entity`, `core/domain`

#### **core/gdx**
- **Purpose:** LibGDX integration (screens, main loop, DI)
- **Key Files:**
  - `CaveDroidApplication.kt` - App lifecycle
  - `GameProc.java` - Main game loop
  - `GameScreen.kt`, `MenuScreen.kt` - Screen implementations
  - `di/ApplicationComponent.kt` - Dagger root component
- **Dependencies:** All other core modules

---

## Core Systems

### 1. World System

**Location:** `/core/game/world/src/main/kotlin/ru/fredboy/cavedroid/game/world/GameWorld.kt`

```kotlin
class GameWorld {
    private val foreMap: Array<Array<Block?>>  // Foreground blocks (solid)
    private val backMap: Array<Array<Block?>>   // Background blocks (decorative)
    private val world: World                     // Box2D physics world
    private val rayHandler: RayHandler           // Lighting system

    fun getBlock(x: Int, y: Int, layer: Layer): Block?
    fun setBlock(x: Int, y: Int, layer: Layer, block: Block?)
    fun update(delta: Float)  // Physics step + lighting update
}
```

**Key Features:**
- **Horizontal Looping:** X-coordinates wrap using modulo
- **Two Layers:** Foreground (collision) and Background (decorative)
- **Block Storage:** 2D arrays indexed by [x][y]
- **Physics:** Box2D World with gravity (32 m/s²)
- **Lighting:** Dynamic lights via Box2DLights RayHandler
- **Time System:** Day/night cycle (1200 seconds per cycle)

**World Generation:**
- `GameWorldGenerator` creates terrain using noise/randomness
- Saves to disk using `ChunkedGameWorldSave` (Protobuf)
- Can load existing worlds or generate new ones

### 2. Entity System

**Controllers manage entity lifecycle:**

| Controller | Entities | Responsibilities |
|------------|----------|------------------|
| `MobController` | Player, NPCs | Movement, AI, health, inventory |
| `DropController` | Item drops | Physics, pickup detection |
| `ProjectileController` | Arrows, snowballs | Trajectory, collision damage |
| `ContainerController` | Chests, furnaces | Storage, smelting logic |

**Entity Update Pattern:**
```
Controller.update(delta)
    ↓
For each entity:
    ├─ Update physics body
    ├─ Update state (health, animation, etc.)
    ├─ Execute behavior (AI, movement)
    └─ Remove if dead/invalid
```

**Mob Behaviors:**
- `WalkingMob` - Simple left/right movement with random direction changes
- `ArcherMob` - Ranged attacks when player in range
- `Sheep`, `Cow`, `Pig` - Passive with special interactions (shearing, milking)

### 3. Item & Block System

**Location:** `/core/domain/items/model/`

**Item Hierarchy:**
```kotlin
sealed class Item {
    object None : Item()  // Empty slot

    data class Placeable : Item()  // Can be placed as block
    data class Slab : Item()       // Half-height block
    data class Tool : Item()       // Durable tools
    data class Shears : Item()     // Special tool
    data class Usable : Item()     // Right-click action
    data class Food : Item()       // Consumable
}
```

**Block Properties:**
```kotlin
sealed class Block {
    abstract val params: CommonBlockParams  // Collision, texture, etc.
    abstract val meta: BlockMeta            // Hardness, drop info

    fun getDrop(): Item?  // What item to drop when mined
    fun isTransparent(): Boolean
    fun isBackground(): Boolean
}
```

**Item/Block Loading:**
1. JSON files define items/blocks (`/assets/json/items.json`, `blocks.json`)
2. `ItemsAssets` deserializes JSON on startup
3. `ItemsRepository` provides access to all items/blocks by key

### 4. Rendering System

**Location:** `/core/gameplay/rendering/src/main/kotlin/.../rendering/GameRenderer.kt`

**Render Pipeline:**
```
GameRenderer.render(delta)
    ↓
Update camera position (follow player)
    ↓
Render world layers:
    ├─ BackgroundBlocksRenderer (layer 0)
    ├─ ForegroundBlocksRenderer (layer 10)
    ├─ MobsRenderer (layer 20)
    ├─ DropsRenderer (layer 21)
    ├─ ProjectilesRenderer (layer 22)
    ├─ PlayerCursorRenderer (layer 30)
    └─ WeatherRenderer (layer 40)
    ↓
Render lighting (Box2DLights)
    ↓
Render HUD:
    ├─ TouchControlsRenderer (mobile)
    ├─ CrosshairRenderer
    ├─ HotbarRenderer
    ├─ Window renderers (inventory, chest, etc.)
    └─ DebugRenderer (if enabled)
```

**Camera System:**
- Orthographic projection (world units = pixels)
- Two modes:
  - **Static:** Locked to player center
  - **Dynamic:** Follows midpoint between player and cursor
- Smooth interpolation for movement

### 5. Input System

**Location:** `/core/gameplay/controls/src/main/kotlin/.../controls/GameInputProcessor.kt`

**Input Handling:**
```
User Input (keyboard/mouse/touch)
    ↓
InputProcessor methods (keyDown, touchDown, etc.)
    ↓
GameInputProcessor delegates to handlers:
    ├─ GameKeyboardInputHandler
    ├─ GameMouseInputHandler
    └─ GameTouchInputHandler
    ↓
Handlers invoke Actions:
    ├─ PlaceBlockActions (different for each block type)
    ├─ UseBlockActions (doors, chests, furnaces)
    ├─ UseItemActions (buckets, eggs)
    └─ UseMobActions (shearing, milking)
```

**Action System:**
- Actions are injectable via Dagger multi-binding
- Each action has a `perform()` method
- Actions can be conditional (e.g., only in Creative mode)

**Touch Controls (Mobile):**
- Left joystick: Movement
- Right area: Cursor control + tap to interact
- Dual-touch: Inventory operations

### 6. Physics System

**Location:** Box2D integration in `GameWorld`

**Physics Configuration:**
```kotlin
val world = World(Vector2(0f, 32f), true)  // Gravity = 32 m/s² down
world.setContactListener(BlockDamageContactListener())
```

**Update Pattern (Fixed Timestep):**
```kotlin
private val timeStep = 1f / 60f  // 60 FPS physics
private var accumulator = 0f

fun update(delta: Float) {
    accumulator += delta
    while (accumulator >= timeStep) {
        world.step(timeStep, 6, 2)  // velocity iterations, position iterations
        accumulator -= timeStep
    }
}
```

**Physics Bodies:**
- **Mobs:** Dynamic bodies with capsule shapes
- **Drops:** Dynamic bodies with small box shapes
- **Blocks:** Static bodies for collision (not all blocks have bodies)
- **Projectiles:** Dynamic bodies with bullet flag (CCD)

---

## Design Patterns

### 1. Dependency Injection (Dagger 2)

**Two DI Scopes:**
- **ApplicationScope:** Singleton, lives for entire app lifetime
  - AssetRepositories, ItemsRepository, SoundPlayer
- **GameScope:** Lives for one game session
  - GameWorld, Controllers, GameRenderer, GameProc

**Component Structure:**
```
ApplicationComponent (Singleton)
    ↓
GameComponent (created per game session)
    ↓
Injects dependencies into GameProc, Controllers, etc.
```

**Example Module:**
```kotlin
@Module
interface GameModule {
    @Binds
    @GameScope
    fun bindGameWorld(impl: GameWorld): GameWorld

    @Multibinds
    fun bindPlaceBlockActions(): Set<PlaceBlockAction>
}
```

### 2. Repository Pattern

**Asset Repositories:**
```kotlin
interface TexturesRepository {
    fun getBlockTexture(name: String): TextureRegion
    fun getItemTexture(name: String): TextureRegion
}

interface ItemsRepository {
    fun getBlock(key: String): Block
    fun getItem(key: String): Item
    fun getAllBlocks(): Collection<Block>
}
```

**Benefits:**
- Decouples data access from business logic
- Allows caching and lazy loading
- Testable (can mock repositories)

### 3. Adapter Pattern

**Entity Adapters:**
```kotlin
interface MobWorldAdapter {
    fun getPlayer(): Player
    fun getMobs(): MutableList<Mob>
}

class MobWorldAdapterImpl @Inject constructor(
    private val mobController: MobController
) : MobWorldAdapter {
    override fun getPlayer() = mobController.player
    override fun getMobs() = mobController.mobs
}
```

**Purpose:** Provide simplified access to entities for different systems without exposing full controller API.

### 4. Observer Pattern

**World Events:**
```kotlin
interface OnBlockPlacedListener {
    fun onBlockPlaced(x: Int, y: Int, layer: GameWorld.Layer)
}

class GameWorld {
    private val onBlockPlacedListeners = mutableListOf<WeakReference<OnBlockPlacedListener>>()

    fun addBlockPlacedListener(listener: OnBlockPlacedListener) {
        onBlockPlacedListeners.add(WeakReference(listener))
    }

    private fun notifyBlockPlaced(x: Int, y: Int, layer: Layer) {
        onBlockPlacedListeners.forEach { it.get()?.onBlockPlaced(x, y, layer) }
    }
}
```

**Uses weak references** to prevent memory leaks.

### 5. Factory Pattern

**Mob Factory:**
```kotlin
interface MobFactory {
    fun create(type: String, x: Float, y: Float): Mob
}

class MobFactoryImpl @Inject constructor(
    private val mobsRepository: MobsRepository,
    private val mobPhysicsFactory: MobPhysicsFactory
) : MobFactory {
    override fun create(type: String, x: Float, y: Float): Mob {
        val mobConfig = mobsRepository.getMobConfig(type)
        val body = mobPhysicsFactory.createBody(x, y, mobConfig)
        return mobConfig.toMob(body)
    }
}
```

---

## Data Flow

### Player Places Block

```
1. User Input (mouse click on world)
    ↓
2. GameMouseInputHandler.touchUp()
    ↓
3. Determines clicked world coordinates
    ↓
4. Checks if block can be placed (air, replaceable, etc.)
    ↓
5. Gets PlaceBlockAction for current item
    ↓
6. PlaceBlockAction.place(x, y, layer)
    ↓
7. GameWorld.setBlock(x, y, layer, block)
    ├─ Updates block array
    ├─ Creates physics body if needed
    ├─ Notifies OnBlockPlacedListeners
    └─ Decrements item stack in inventory
    ↓
8. On next render frame:
    ├─ ForegroundBlocksRenderer draws new block
    └─ HotbarRenderer shows updated inventory
```

### Mob Takes Damage

```
1. Damage source (fall, combat, environment)
    ↓
2. Mob.damage(amount)
    ↓
3. Reduces mob health
    ↓
4. If health <= 0:
    ↓
5. Mob.kill()
    ├─ Sets dead flag
    └─ Calls onDeath()
    ↓
6. MobController.update() detects dead mob
    ├─ Spawns item drops (loot)
    ├─ Destroys physics body
    └─ Removes from mob list
```

### Loading a Save File

```
1. User selects world from menu
    ↓
2. MenuScreen creates StartGameConfig
    ↓
3. CaveDroidApplication.createGameComponent(config)
    ↓
4. GameComponent.Builder reads config.saveData
    ↓
5. If saveData exists:
    ├─ ChunkedGameWorldSave.load()
    ├─ Deserializes Protobuf data
    ├─ Reconstructs block arrays
    ├─ Loads player position/inventory
    └─ Loads mob/container states
    ↓
6. GameWorld initialized with loaded data
    ↓
7. Physics bodies recreated for all blocks/entities
    ↓
8. Game begins with restored state
```

---

## Summary

CaveDroid is architected with:

- **Clean separation of concerns** across modules
- **Dependency injection** for testability and modularity
- **Fixed timestep physics** for deterministic simulation
- **Event-driven updates** for block/mob changes
- **Layered rendering** with priority-based ordering
- **Cross-platform core** with platform-specific launchers

Understanding this architecture will help you navigate the codebase and make modifications confidently. For specific changes, refer to the guides in `/docs/guides/`.
