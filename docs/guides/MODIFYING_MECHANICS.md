# Modifying Game Mechanics

This guide explains how to modify core gameplay systems in CaveDroid.

## Table of Contents

1. [Overview](#overview)
2. [Prerequisites](#prerequisites)
3. [Understanding the Code Structure](#understanding-the-code-structure)
4. [Common Modifications](#common-modifications)
   - [World Generation](#world-generation)
   - [Mob Spawning](#mob-spawning)
   - [Block Behavior](#block-behavior)
   - [Physics Parameters](#physics-parameters)
   - [Day/Night Cycle](#daynight-cycle)
   - [Crafting System](#crafting-system)
5. [Advanced Modifications](#advanced-modifications)
6. [Testing Changes](#testing-changes)
7. [Best Practices](#best-practices)

---

## Overview

Unlike adding blocks, items, or mobs (which use JSON), modifying core game mechanics requires **code changes**. This guide covers common modifications that developers often want to make.

**Important:** Always read the [ARCHITECTURE.md](../ARCHITECTURE.md) guide first to understand how the codebase is organized.

---

## Prerequisites

1. Development environment set up (see [QUICK_START.md](../QUICK_START.md))
2. Understanding of Kotlin programming
3. Familiarity with the architecture (read [ARCHITECTURE.md](../ARCHITECTURE.md))
4. IDE configured (IntelliJ IDEA recommended)

---

## Understanding the Code Structure

Before modifying mechanics, understand where different systems live:

| System | Location |
|--------|----------|
| World Generation | `/core/game/world/src/main/kotlin/.../GameWorldGenerator.kt` |
| Mob Spawning | `/core/gameplay/physics/src/main/kotlin/.../GameWorldMobSpawnControllerTask.kt` |
| Block Updates | `/core/gameplay/physics/src/main/kotlin/.../GameWorldBlocksLogicControllerTask.kt` |
| Fluid Logic | `/core/gameplay/physics/src/main/kotlin/.../GameWorldFluidsLogicControllerTask.kt` |
| Physics Constants | `/core/domain/world/src/main/kotlin/.../PhysicsConstants.kt` |
| World Management | `/core/game/world/src/main/kotlin/.../GameWorld.kt` |
| Crafting Logic | (Handled via JSON in `/assets/json/crafting.json`) |

---

## Common Modifications

### World Generation

**File:** `/core/game/world/src/main/kotlin/ru/fredboy/cavedroid/game/world/GameWorldGenerator.kt`

#### Example: Change Terrain Height

```kotlin
class GameWorldGenerator @Inject constructor(...) {

    companion object {
        private const val SEA_LEVEL = 64  // Change this to raise/lower sea level
        private const val TERRAIN_ROUGHNESS = 0.5f  // 0.0 = flat, 1.0 = very rough
    }

    private fun generateTerrain(x: Int): Int {
        // Modify terrain generation algorithm here
        val baseHeight = SEA_LEVEL
        val variation = (noise(x) * TERRAIN_ROUGHNESS * 20).toInt()
        return baseHeight + variation
    }
}
```

#### Example: Add Custom Biomes

```kotlin
private fun placeBlock(x: Int, y: Int) {
    when (getBiome(x)) {
        Biome.DESERT -> placeDesertBlocks(x, y)
        Biome.FOREST -> placeForestBlocks(x, y)
        Biome.SNOWY -> placeSnowyBlocks(x, y)
        Biome.CUSTOM_BIOME -> placeCustomBlocks(x, y)
    }
}

private fun placeCustomBlocks(x: Int, y: Int) {
    // Your custom biome generation logic
    val surfaceBlock = itemsRepository.getBlock("custom_grass")
    gameWorld.setForeMap(x, y, surfaceBlock)
}
```

#### Example: Change Ore Generation

```kotlin
private fun generateOres() {
    // Modify ore spawn rates and depths
    placeOre("coal_ore", frequency = 20, minDepth = 0, maxDepth = 128)
    placeOre("iron_ore", frequency = 15, minDepth = 0, maxDepth = 64)
    placeOre("diamond_ore", frequency = 1, minDepth = 0, maxDepth = 16)
    placeOre("custom_ore", frequency = 5, minDepth = 20, maxDepth = 50)
}

private fun placeOre(oreKey: String, frequency: Int, minDepth: Int, maxDepth: Int) {
    val ore = itemsRepository.getBlock(oreKey)
    // ... ore placement logic
}
```

---

### Mob Spawning

**File:** `/core/gameplay/physics/src/main/kotlin/ru/fredboy/cavedroid/gameplay/physics/task/GameWorldMobSpawnControllerTask.kt`

#### Example: Change Spawn Rates

```kotlin
class GameWorldMobSpawnControllerTask @Inject constructor(...) {

    companion object {
        private const val MAX_MOBS = 50  // Maximum mobs in world
        private const val SPAWN_INTERVAL_MS = 10000L  // Time between spawns (10s)
    }

    override fun run() {
        if (mobController.mobs.size >= MAX_MOBS) {
            return
        }

        // Modify spawn logic
        if (shouldSpawnMob()) {
            spawnRandomMob()
        }
    }

    private fun shouldSpawnMob(): Boolean {
        // Custom spawn conditions
        val isNight = gameWorld.isNight()
        val nearbyMobCount = getNearbyMobCount()

        return isNight && nearbyMobCount < 10 && Random.nextFloat() < 0.3f
    }
}
```

#### Example: Biome-Specific Spawning

```kotlin
private fun spawnRandomMob() {
    val spawnX = getRandomSpawnX()
    val spawnY = getSpawnY(spawnX)
    val biome = getBiome(spawnX)

    val mobType = when (biome) {
        Biome.DESERT -> listOf("zombie", "skeleton").random()
        Biome.SNOWY -> listOf("polar_bear", "yeti").random()
        Biome.FOREST -> listOf("pig", "cow", "wolf").random()
        else -> listOf("zombie", "pig").random()
    }

    mobFactory.create(mobType, spawnX.toFloat(), spawnY.toFloat())
}
```

#### Example: Custom Spawn Conditions

```kotlin
private fun canSpawnAt(x: Int, y: Int): Boolean {
    // Must be on solid ground
    val groundBlock = gameWorld.getForeMap(x, y + 1)
    if (groundBlock == null || !groundBlock.params.hasCollision) {
        return false
    }

    // Must have air above
    val aboveBlock = gameWorld.getForeMap(x, y - 1)
    if (aboveBlock != null && aboveBlock.params.hasCollision) {
        return false
    }

    // Must have light level below threshold (for hostile mobs)
    val lightLevel = getLightLevel(x, y)
    if (lightLevel > 7) {
        return false
    }

    return true
}
```

---

### Block Behavior

**File:** `/core/gameplay/physics/src/main/kotlin/ru/fredboy/cavedroid/gameplay/physics/task/GameWorldBlocksLogicControllerTask.kt`

#### Example: Add Custom Block Updates

```kotlin
class GameWorldBlocksLogicControllerTask @Inject constructor(...) {

    @BindUpdateBlockAction(stringKey = "custom_growing_block")
    class CustomBlockUpdateAction @Inject constructor(
        private val gameWorld: GameWorld,
        private val itemsRepository: ItemsRepository
    ) : IUpdateBlockAction {

        override fun update(x: Int, y: Int, elapsed: Long) {
            // Check if enough time has passed
            if (elapsed < 100) return

            // Check if block has light
            if (!hasLight(x, y)) return

            // Transform to grown state
            val grownBlock = itemsRepository.getBlock("custom_grown_block")
            gameWorld.setForeMap(x, y, grownBlock)
        }

        private fun hasLight(x: Int, y: Int): Boolean {
            // Check for light level
            return gameWorld.getLightLevel(x, y) > 8
        }
    }
}
```

#### Example: Grass Spreading Mechanics

```kotlin
// Modify existing grass spreading behavior
@BindUpdateBlockAction(stringKey = "grass")
class GrassUpdateAction @Inject constructor(...) : IUpdateBlockAction {

    override fun update(x: Int, y: Int, elapsed: Long) {
        // Faster grass spreading
        if (elapsed < 50) return  // Was 100, now 50 = spreads faster

        // Try to spread to adjacent blocks
        val directions = listOf(
            Pair(x - 1, y),  // Left
            Pair(x + 1, y),  // Right
        )

        for ((adjX, adjY) in directions) {
            val block = gameWorld.getForeMap(adjX, adjY)
            if (block?.key == "dirt") {
                val grassBlock = itemsRepository.getBlock("grass")
                gameWorld.setForeMap(adjX, adjY, grassBlock)
                break
            }
        }
    }
}
```

---

### Physics Parameters

**File:** `/core/domain/world/src/main/kotlin/ru/fredboy/cavedroid/domain/world/model/PhysicsConstants.kt`

#### Example: Modify Gravity

```kotlin
object PhysicsConstants {
    const val GRAVITY = 32f  // Default gravity (increase for stronger, decrease for weaker)

    // Example: Moon gravity
    // const val GRAVITY = 8f

    // Example: Strong gravity
    // const val GRAVITY = 50f
}
```

#### Example: Player Movement

**File:** `/core/entity/mob/src/main/kotlin/ru/fredboy/cavedroid/entity/mob/model/Player.kt`

```kotlin
class Player(...) : Mob(...) {

    companion object {
        private const val WALK_SPEED = 4.317f  // Increase for faster movement
        private const val JUMP_VELOCITY = -11f  // More negative = higher jump
    }

    fun walk(direction: Direction) {
        velocity.x = when (direction) {
            Direction.LEFT -> -WALK_SPEED
            Direction.RIGHT -> WALK_SPEED
        }
    }

    fun jump() {
        if (canJump) {
            velocity.y = JUMP_VELOCITY
        }
    }
}
```

---

### Day/Night Cycle

**File:** `/core/game/world/src/main/kotlin/ru/fredboy/cavedroid/game/world/GameWorld.kt`

#### Example: Change Cycle Length

```kotlin
class GameWorld @Inject constructor(...) {

    companion object {
        private const val TICKS_IN_DAY = 1200f  // Seconds in a full day/night cycle
        // Decrease for faster day/night: 600f = 10 minutes
        // Increase for slower: 2400f = 40 minutes
    }

    private var worldTime = 0f

    fun update(delta: Float) {
        worldTime += delta
        if (worldTime >= TICKS_IN_DAY) {
            worldTime -= TICKS_IN_DAY
        }

        updateLighting()
    }

    fun isNight(): Boolean {
        // Night is when time is between 50% and 100% of day
        return worldTime > TICKS_IN_DAY * 0.5f
    }

    fun isDay(): Boolean = !isNight()
}
```

#### Example: Add Twilight Period

```kotlin
enum class TimeOfDay {
    MORNING,
    DAY,
    EVENING,
    NIGHT
}

fun getTimeOfDay(): TimeOfDay {
    val progress = worldTime / TICKS_IN_DAY
    return when {
        progress < 0.25f -> TimeOfDay.MORNING
        progress < 0.50f -> TimeOfDay.DAY
        progress < 0.75f -> TimeOfDay.EVENING
        else -> TimeOfDay.NIGHT
    }
}

// Adjust mob spawning based on time of day
private fun canSpawnHostileMob(): Boolean {
    return getTimeOfDay() in listOf(TimeOfDay.EVENING, TimeOfDay.NIGHT)
}
```

---

### Crafting System

**Crafting recipes are defined in JSON**, but you can modify crafting logic:

**File:** `/core/game/controller/src/main/kotlin/.../CraftingController.kt` (if it exists, or create)

#### Example: Add Crafting Bonuses

```kotlin
class CraftingController @Inject constructor(...) {

    fun craft(recipeKey: String): Boolean {
        val recipe = getRecipe(recipeKey)

        // Check if player has materials
        if (!hasIngredients(recipe)) {
            return false
        }

        // Remove ingredients
        consumeIngredients(recipe)

        // Add output
        val outputAmount = recipe.output.count * getCraftingMultiplier()
        player.inventory.add(recipe.output.item, outputAmount)

        return true
    }

    private fun getCraftingMultiplier(): Int {
        // Example: Crafting near a workbench gives bonus items
        if (isNearWorkbench()) {
            return 2  // Double output
        }
        return 1
    }

    private fun isNearWorkbench(): Boolean {
        val playerX = player.mapX
        val playerY = player.middleMapY

        for (x in playerX - 5..playerX + 5) {
            for (y in playerY - 5..playerY + 5) {
                val block = gameWorld.getForeMap(x, y)
                if (block?.key == "crafting_table") {
                    return true
                }
            }
        }
        return false
    }
}
```

---

## Advanced Modifications

### Adding New Game Modes

**Example: Add Hardcore Mode**

1. **Add to game configuration:**

```kotlin
// In GameContext
enum class GameMode {
    SURVIVAL,
    CREATIVE,
    HARDCORE  // New mode
}
```

2. **Modify player death:**

```kotlin
// In Player class
override fun onDeath() {
    when (gameContext.gameMode) {
        GameMode.HARDCORE -> {
            // Delete world on death
            deleteWorld()
            returnToMainMenu()
        }
        GameMode.SURVIVAL -> {
            // Normal respawn
            respawn()
        }
        GameMode.CREATIVE -> {
            // Do nothing
        }
    }
}
```

### Custom Inventory Systems

**Example: Add Tool Belt**

```kotlin
class Player(...) {
    val mainInventory = Inventory(size = 40)
    val toolBelt = Inventory(size = 10)  // New inventory

    fun getQuickAccessInventory(): Inventory {
        return when (quickAccessMode) {
            QuickAccessMode.MAIN -> mainInventory
            QuickAccessMode.TOOL_BELT -> toolBelt
        }
    }
}
```

### Weather System

**Example: Add Rain/Snow**

```kotlin
class GameWorld(...) {

    private var weatherType = WeatherType.CLEAR
    private var weatherDuration = 0f

    enum class WeatherType {
        CLEAR,
        RAIN,
        SNOW,
        STORM
    }

    fun update(delta: Float) {
        updateWeather(delta)
        // ... rest of update
    }

    private fun updateWeather(delta: Float) {
        weatherDuration -= delta

        if (weatherDuration <= 0) {
            // Choose new weather
            weatherType = WeatherType.values().random()
            weatherDuration = Random.nextFloat() * 600f + 300f  // 5-15 minutes
        }

        // Apply weather effects
        when (weatherType) {
            WeatherType.RAIN -> {
                // Fill empty spaces with water (slowly)
                if (Random.nextFloat() < 0.01f) {
                    fillRandomBlockWithWater()
                }
            }
            WeatherType.SNOW -> {
                // Place snow on top of blocks
                if (Random.nextFloat() < 0.01f) {
                    placeSnowOnSurface()
                }
            }
            else -> {}
        }
    }
}
```

---

## Testing Changes

### 1. Unit Testing

Create tests for your changes:

```kotlin
// In appropriate test directory
class GameWorldGeneratorTest {

    @Test
    fun `terrain generation stays within bounds`() {
        val generator = GameWorldGenerator(...)
        val world = generator.generate()

        for (x in 0 until world.width) {
            val surfaceY = world.getSurfaceY(x)
            assertTrue(surfaceY in 0..world.height)
        }
    }
}
```

### 2. In-Game Testing

```bash
# Run with debug mode
./gradlew desktop:run --args="--debug"
```

**Enable debug info** to see:
- FPS and performance
- Entity counts
- World coordinates
- Physics debug rendering

### 3. Performance Testing

Monitor performance after changes:

```kotlin
// Add timing logs
val startTime = System.currentTimeMillis()
yourExpensiveOperation()
val elapsed = System.currentTimeMillis() - startTime
CommonLogger.debug("Performance", "Operation took ${elapsed}ms")
```

---

## Best Practices

### 1. Make Small Changes

- Modify one system at a time
- Test after each change
- Commit frequently

### 2. Keep Configuration Separate

```kotlin
// Good: Use constants
object GameConfig {
    const val MAX_MOBS = 50
    const val SPAWN_RATE = 0.3f
}

// Bad: Hardcoded values everywhere
if (mobCount > 50) { ... }
```

### 3. Document Your Changes

```kotlin
/**
 * Generates terrain with custom biome support.
 *
 * @param seed World seed for deterministic generation
 * @return Generated world
 *
 * Modifications:
 * - Added CUSTOM_BIOME support
 * - Changed ore spawn rates
 * - Increased terrain roughness
 */
fun generate(seed: Long): GameWorld {
    // ...
}
```

### 4. Maintain Compatibility

```kotlin
// When modifying save format, maintain backward compatibility
fun loadWorld(saveData: SaveData): GameWorld {
    return when (saveData.version) {
        1 -> loadWorldV1(saveData)
        2 -> loadWorldV2(saveData)
        else -> throw UnsupportedVersionException()
    }
}
```

### 5. Use Dependency Injection

```kotlin
// Good: Injected dependencies
class MyNewSystem @Inject constructor(
    private val gameWorld: GameWorld,
    private val itemsRepository: ItemsRepository
) {
    // ...
}

// Bad: Direct instantiation
class MyNewSystem {
    private val gameWorld = GameWorld()  // Don't do this
}
```

### 6. Follow Existing Patterns

- Study similar existing code
- Use same coding style
- Follow module organization
- Respect architecture layers

---

## Common Pitfalls

### 1. Performance Issues

**Problem:** Checking every block every frame

```kotlin
// Bad: Very slow
fun update(delta: Float) {
    for (x in 0 until width) {
        for (y in 0 until height) {
            updateBlock(x, y)  // Runs millions of times per second!
        }
    }
}

// Good: Use scheduled tasks with delays
class BlockUpdateTask : MutableTask {
    override fun run() {
        updateRandomBlocks(count = 100)  // Only update 100 blocks per run
    }
}
```

### 2. Save File Corruption

**Problem:** Changing data structures without migration

```kotlin
// When modifying save data structures:
@Serializable
data class SaveData(
    val version: Int = 2,  // Increment version
    val blocks: List<Block>,
    val newField: String? = null  // Make new fields nullable with defaults
)
```

### 3. Breaking Changes

**Problem:** Modifying public APIs

```kotlin
// Bad: Breaking change
fun getBlock(x: Int, y: Int): Block  // Changed return type

// Good: Add new method, deprecate old
@Deprecated("Use getBlockOrNull", ReplaceWith("getBlockOrNull(x, y)"))
fun getBlock(x: Int, y: Int): Block

fun getBlockOrNull(x: Int, y: Int): Block?
```

---

## Next Steps

- **[ARCHITECTURE.md](../ARCHITECTURE.md)** - Deep dive into system internals
- **[ADDING_BLOCKS.md](ADDING_BLOCKS.md)** - Create content for your mechanics
- **[ADDING_ITEMS.md](ADDING_ITEMS.md)** - Add items for gameplay
- **[ADDING_MOBS.md](ADDING_MOBS.md)** - Create mobs for your systems

---

## Getting Help

- **Check Existing Code:** Look for similar implementations
- **Read Architecture Docs:** Understand the system first
- **Test Incrementally:** Small changes are easier to debug
- **Use Logging:** Add debug logs to track behavior

Happy modding!
