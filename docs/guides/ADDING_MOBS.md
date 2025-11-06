# Adding New Mobs

This guide explains how to add new mobs (creatures, NPCs, enemies) to CaveDroid.

## Table of Contents

1. [Overview](#overview)
2. [Prerequisites](#prerequisites)
3. [Step 1: Create Mob Sprites](#step-1-create-mob-sprites)
4. [Step 2: Define Mob in JSON](#step-2-define-mob-in-json)
5. [Step 3: Add Custom Behavior (Optional)](#step-3-add-custom-behavior-optional)
6. [Step 4: Create Spawn Egg (Optional)](#step-4-create-spawn-egg-optional)
7. [Step 5: Test Your Mob](#step-5-test-your-mob)
8. [Mob Property Reference](#mob-property-reference)
9. [Examples](#examples)

---

## Overview

Mobs in CaveDroid are defined through JSON configuration with sprite-based animations. Each mob consists of:

- **Sprites:** Multiple body parts (head, body, legs, arms)
- **Behavior:** Passive, aggressive, or custom AI
- **Properties:** Health, speed, drops, etc.

**Key Files:**
- `/assets/json/mobs.json` - Mob definitions
- `/assets/textures/mobs/[mob_name]/` - Mob sprites
- `/assets/json/game_items.json` - Spawn eggs (optional)

---

## Prerequisites

1. Development environment set up (see [QUICK_START.md](../QUICK_START.md))
2. Image editing software for creating sprites
3. Understanding of JSON format
4. Familiarity with basic animation concepts

---

## Step 1: Create Mob Sprites

Mobs are built from multiple sprite parts that animate together.

### Sprite Structure

Create a directory: `/assets/textures/mobs/your_mob_name/`

**Common sprite parts:**
- `head.png` - Head/face
- `body.png` - Torso
- `leg.png` - Legs (animated when walking)
- `hand.png` - Arms (optional, for humanoid mobs)

### Sprite Guidelines

- **Size:** Variable (based on mob size)
- **Format:** PNG with transparency
- **Style:** Match existing game art
- **Naming:** Lowercase, simple names

### Example: Simple Mob (4-legged animal)

```
/assets/textures/mobs/fox/
├── body.png    (main body)
├── head.png    (head/face)
└── leg.png     (legs, will be duplicated and animated)
```

### Example: Humanoid Mob

```
/assets/textures/mobs/villager/
├── head.png    (head)
├── body.png    (torso)
├── hand.png    (arms)
└── leg.png     (legs)
```

### Sprite Tips

- **Background/Foreground:** Create sprite layers for depth (some sprites render behind the body)
- **Animation:** Legs and arms are typically animated by rotation
- **Static Parts:** Body and head can be static or animated
- **Overlay Sprites:** Used for special effects (e.g., sheep wool)

---

## Step 2: Define Mob in JSON

Open `/assets/json/mobs.json` and add your mob definition.

### Basic Passive Mob Example

```json
{
  "fox": {
    "name": "Fox",
    "width": 20,
    "height": 16,
    "speed": 4,
    "hp": 8,
    "breath": 20,
    "behavior": "passive",
    "drop_info": [
      {
        "key": "leather",
        "amount": {
          "type": "random_range",
          "min": 1,
          "max": 2,
          "chance": 0.66
        }
      }
    ],
    "sprites": [
      {
        "file": "leg",
        "offset_x": 6,
        "offset_y": 10,
        "is_background": true,
        "is_static": false
      },
      {
        "file": "leg",
        "offset_x": 14,
        "offset_y": 10,
        "is_background": true,
        "is_static": false
      },
      {
        "file": "body",
        "is_background": false,
        "is_static": true
      },
      {
        "file": "head",
        "is_background": false,
        "is_static": true
      },
      {
        "file": "leg",
        "offset_x": 6,
        "offset_y": 10,
        "is_background": false,
        "is_static": false
      },
      {
        "file": "leg",
        "offset_x": 14,
        "offset_y": 10,
        "is_background": false,
        "is_static": false
      }
    ]
  }
}
```

### Required Properties

| Property | Description | Example |
|----------|-------------|---------|
| `name` | Display name | `"Fox"` |
| `width` | Mob width in pixels | `20` |
| `height` | Mob height in pixels | `16` |
| `speed` | Movement speed | `3.0` - `5.0` |
| `hp` | Health points | `10` |
| `breath` | Breath capacity (for drowning) | `20` (use `-1` for undead) |
| `behavior` | AI behavior type | `"passive"`, `"aggressive"`, `"archer"`, `"sheep"` |
| `sprites` | Array of sprite definitions | See sprite structure below |

### Optional Properties

| Property | Description | Default |
|----------|-------------|---------|
| `animation_range` | Animation speed multiplier | `30` |
| `damage_to_player` | Damage dealt on attack | `0` |
| `takes_sun_damage` | Burns in sunlight (undead) | `false` |
| `drop_info` | Items dropped on death | `[]` |

### Sprite Structure

Each sprite in the `sprites` array:

```json
{
  "file": "body",           // Sprite filename (without .png)
  "offset_x": 2,           // X position offset from mob center (optional)
  "offset_y": 8,           // Y position offset from mob center (optional)
  "is_background": false,  // Render behind body (for depth)
  "is_static": true,       // Don't animate (no rotation)
  "is_head": false,        // Is this the head sprite? (optional)
  "is_hand": false,        // Is this a hand sprite? (optional)
  "is_overlay": false,     // Render as overlay (sheep wool, etc.) (optional)
  "origin_x": 0.5,         // Rotation origin X (optional, default 0.5)
  "origin_y": 0.5          // Rotation origin Y (optional, default 0.5)
}
```

### Sprite Rendering Order

Sprites are rendered in the order they appear in the array:
1. Background sprites first (`is_background: true`)
2. Foreground sprites second (`is_background: false`)

**Example ordering for depth:**
```
1. Back legs (is_background: true)
2. Body (is_background: false)
3. Front legs (is_background: false)
4. Head (is_background: false)
```

---

## Step 3: Add Custom Behavior (Optional)

Built-in behaviors work for most mobs, but you can create custom AI.

### Built-in Behaviors

| Behavior | Description | Example Mobs |
|----------|-------------|--------------|
| `"passive"` | Wanders randomly, doesn't attack | Pig, Cow, Chicken |
| `"aggressive"` | Chases and attacks player | Zombie |
| `"archer"` | Ranged attacks with projectiles | Skeleton |
| `"sheep"` | Special sheep behavior (shearing, wool) | Sheep |
| `"player"` | Player-controlled | Player |

### When to Create Custom Behavior

- **Special AI patterns** (flying, teleporting, fleeing)
- **Unique interactions** (trading, taming)
- **Complex attack patterns** (boss mechanics)
- **Environmental reactions** (attacking specific blocks)

### Creating Custom Behavior

1. **Create Behavior Class:**

```kotlin
// In: core/entity/mob/src/main/kotlin/ru/fredboy/cavedroid/entity/mob/behavior/

class CustomMobBehavior(
    private val mobWorldAdapter: MobWorldAdapter,
    private val playerAdapter: PlayerAdapter
) : MobBehavior {

    override fun update(mob: Mob, delta: Float) {
        // Your custom AI logic here

        // Example: Run away from player
        val player = playerAdapter.getPlayer()
        val distance = mob.position.dst(player.position)

        if (distance < 64f) {
            // Move away from player
            if (mob.position.x < player.position.x) {
                mob.velocity.x = -mob.speed
                mob.direction = Mob.Direction.LEFT
            } else {
                mob.velocity.x = mob.speed
                mob.direction = Mob.Direction.RIGHT
            }
        } else {
            // Wander randomly
            // ... implement wandering logic
        }
    }
}
```

2. **Register Behavior in MobFactory:**

```kotlin
// In mob factory, map behavior string to behavior instance
when (mobParams.behavior) {
    "custom_mob" -> CustomMobBehavior(mobWorldAdapter, playerAdapter)
    // ... other behaviors
}
```

3. **Use in JSON:**

```json
{
  "your_mob": {
    "behavior": "custom_mob",
    ...
  }
}
```

---

## Step 4: Create Spawn Egg (Optional)

Allow players to spawn your mob with a spawn egg item.

### Add Spawn Egg to game_items.json

```json
{
  "items": {
    "spawn_egg_fox": {
      "name": "Fox Spawn Egg",
      "type": "usable",
      "texture": "spawn_egg",
      "tint": "#FF6600",
      "action_key": "use_spawn_egg",
      "mob_key": "fox",
      "origin_x": 0.5
    }
  }
}
```

**Properties:**
- `type`: Must be `"usable"`
- `texture`: Use `"spawn_egg"` (standard egg texture)
- `tint`: Color hex code (makes egg unique)
- `action_key`: Must be `"use_spawn_egg"`
- `mob_key`: Your mob's key from mobs.json

### Add Crafting Recipe (Optional)

```json
{
  "crafting": {
    "spawn_egg_fox": {
      "input": [
        ["", "fox_fur", ""],
        ["fox_fur", "egg", "fox_fur"],
        ["", "fox_fur", ""]
      ],
      "output": {
        "item": "spawn_egg_fox",
        "count": 1
      }
    }
  }
}
```

---

## Step 5: Test Your Mob

### 1. Launch Game

```bash
./gradlew desktop:run
```

### 2. Spawn Your Mob

**Method 1:** Use spawn egg in Creative mode
- Open inventory (`E`)
- Find your spawn egg
- Right-click to place mob

**Method 2:** Modify spawn rate temporarily
- Edit mob spawn logic to force your mob to spawn
- Wait for natural spawning

### 3. Test Behaviors

- **Movement:** Does it walk/run correctly?
- **AI:** Does behavior work as expected?
- **Sprites:** Do sprites render correctly?
- **Animation:** Do legs/arms animate when moving?
- **Drops:** Kill mob and check drops
- **Combat:** For aggressive mobs, test attacking

### 4. Common Issues

**Mob doesn't appear:**
- Check JSON syntax
- Verify sprite files exist
- Check mob key matches spawn egg

**Sprites look wrong:**
- Verify sprite offsets (`offset_x`, `offset_y`)
- Check `is_background` ordering
- Adjust sprite origins

**Animation issues:**
- Check `is_static` flags
- Verify `animation_range`
- Ensure animated sprites aren't marked static

**Mob doesn't move:**
- Check `speed` value
- Verify behavior is correct
- Check for physics issues (mob stuck in blocks)

---

## Mob Property Reference

### Complete Property List

```json
{
  "your_mob": {
    // === REQUIRED ===
    "name": "Display Name",
    "width": 20,                    // Collision width in pixels
    "height": 16,                   // Collision height in pixels
    "speed": 3.0,                   // Movement speed
    "hp": 10,                       // Health points
    "breath": 20,                   // Breath capacity (-1 for undead)
    "behavior": "passive",          // AI behavior type

    // === OPTIONAL ===
    "animation_range": 30,          // Animation speed multiplier (default: 30)
    "damage_to_player": 2,          // Damage per hit (default: 0)
    "takes_sun_damage": true,       // Burns in sunlight (default: false)

    // === DROP INFO ===
    "drop_info": [
      {
        "key": "item_name",         // Item to drop
        "amount": {
          "type": "exact_amount",   // or "random_range"
          "amount": 1               // or "min", "max", "chance"
        }
      }
    ],

    // === SPRITES ===
    "sprites": [
      {
        "file": "sprite_name",      // Filename without .png
        "offset_x": 0,              // X offset from center (default: 0)
        "offset_y": 0,              // Y offset from center (default: 0)
        "is_background": false,     // Render behind body (default: false)
        "is_static": true,          // Don't animate (default: true)
        "is_head": false,           // Is head sprite (default: false)
        "is_hand": false,           // Is hand sprite (default: false)
        "is_overlay": false,        // Overlay rendering (default: false)
        "origin_x": 0.5,            // Rotation origin X (default: 0.5)
        "origin_y": 0.5             // Rotation origin Y (default: 0.5)
      }
    ]
  }
}
```

### Drop Amount Types

**Exact Amount:**
```json
{
  "key": "item_name",
  "amount": {
    "type": "exact_amount",
    "amount": 1
  }
}
```

**Random Range with Chance:**
```json
{
  "key": "rare_item",
  "amount": {
    "type": "random_range",
    "min": 1,
    "max": 3,
    "chance": 0.25          // 25% chance to drop
  }
}
```

---

## Examples

### Example 1: Simple Passive Mob (Rabbit)

```json
{
  "rabbit": {
    "name": "Rabbit",
    "width": 14,
    "height": 12,
    "speed": 4.5,
    "hp": 3,
    "breath": 20,
    "behavior": "passive",
    "drop_info": [
      {
        "key": "rabbit_hide",
        "amount": {
          "type": "random_range",
          "min": 0,
          "max": 1,
          "chance": 1
        }
      }
    ],
    "sprites": [
      {
        "file": "leg",
        "offset_x": 4,
        "offset_y": 8,
        "is_background": true,
        "is_static": false
      },
      {
        "file": "leg",
        "offset_x": 10,
        "offset_y": 8,
        "is_background": true,
        "is_static": false
      },
      {
        "file": "body",
        "is_background": false,
        "is_static": true
      },
      {
        "file": "head",
        "is_background": false,
        "is_static": true
      },
      {
        "file": "leg",
        "offset_x": 4,
        "offset_y": 8,
        "is_background": false,
        "is_static": false
      },
      {
        "file": "leg",
        "offset_x": 10,
        "offset_y": 8,
        "is_background": false,
        "is_static": false
      }
    ]
  }
}
```

### Example 2: Aggressive Mob (Wolf)

```json
{
  "wolf": {
    "name": "Wolf",
    "width": 24,
    "height": 18,
    "speed": 5.0,
    "hp": 12,
    "breath": 20,
    "behavior": "aggressive",
    "damage_to_player": 3,
    "animation_range": 20,
    "drop_info": [],
    "sprites": [
      {
        "file": "leg",
        "offset_x": 6,
        "offset_y": 12,
        "is_background": true,
        "is_static": false
      },
      {
        "file": "leg",
        "offset_x": 18,
        "offset_y": 12,
        "is_background": true,
        "is_static": false
      },
      {
        "file": "body",
        "is_background": false,
        "is_static": true
      },
      {
        "file": "head",
        "offset_x": -2,
        "is_background": false,
        "is_static": true
      },
      {
        "file": "leg",
        "offset_x": 6,
        "offset_y": 12,
        "is_background": false,
        "is_static": false
      },
      {
        "file": "leg",
        "offset_x": 18,
        "offset_y": 12,
        "is_background": false,
        "is_static": false
      }
    ]
  }
}
```

Add spawn egg:
```json
{
  "spawn_egg_wolf": {
    "name": "Wolf Spawn Egg",
    "type": "usable",
    "texture": "spawn_egg",
    "tint": "#8B7355",
    "action_key": "use_spawn_egg",
    "mob_key": "wolf",
    "origin_x": 0.5
  }
}
```

### Example 3: Flying Mob (Bat)

**Note:** Requires custom behavior for flying AI.

```json
{
  "bat": {
    "name": "Bat",
    "width": 14,
    "height": 10,
    "speed": 3.0,
    "hp": 4,
    "breath": -1,
    "behavior": "flying_passive",
    "drop_info": [],
    "sprites": [
      {
        "file": "body",
        "is_background": false,
        "is_static": true
      },
      {
        "file": "wing",
        "offset_x": -4,
        "is_background": false,
        "is_static": false
      },
      {
        "file": "wing",
        "offset_x": 4,
        "is_background": false,
        "is_static": false
      }
    ]
  }
}
```

### Example 4: Undead Mob (Skeleton Warrior)

```json
{
  "skeleton_warrior": {
    "name": "Skeleton Warrior",
    "width": 8,
    "height": 30,
    "speed": 2.5,
    "hp": 25,
    "breath": -1,
    "behavior": "aggressive",
    "damage_to_player": 4,
    "takes_sun_damage": true,
    "animation_range": 15,
    "drop_info": [
      {
        "key": "bone",
        "amount": {
          "type": "random_range",
          "min": 2,
          "max": 4,
          "chance": 1
        }
      },
      {
        "key": "iron_sword",
        "amount": {
          "type": "exact_amount",
          "amount": 1,
          "chance": 0.05
        }
      }
    ],
    "sprites": [
      {
        "file": "hand",
        "offset_x": -6,
        "offset_y": 8,
        "is_background": true,
        "is_hand": true,
        "is_static": false,
        "origin_x": 1,
        "origin_y": 0.5
      },
      {
        "file": "leg",
        "offset_x": 2,
        "offset_y": 20,
        "is_background": true,
        "is_static": false
      },
      {
        "file": "head",
        "is_background": false,
        "is_head": true,
        "is_static": true,
        "origin_y": 1
      },
      {
        "file": "body",
        "offset_x": 2,
        "offset_y": 8,
        "is_background": false
      },
      {
        "file": "leg",
        "offset_x": 2,
        "offset_y": 20,
        "is_background": false,
        "is_static": false
      },
      {
        "file": "hand",
        "offset_x": -6,
        "offset_y": 8,
        "is_background": false,
        "is_hand": true,
        "is_static": false,
        "origin_x": 1,
        "origin_y": 0.5
      }
    ]
  }
}
```

---

## Tips and Best Practices

1. **Start Simple:** Begin with a basic passive mob before adding complex behaviors
2. **Copy Similar Mobs:** Find an existing mob similar to yours and modify it
3. **Sprite Layers:** Use background/foreground for depth (back legs behind body)
4. **Animation:** Legs should be `is_static: false`, body usually `is_static: true`
5. **Size Balance:** Compare with existing mobs for appropriate width/height
6. **Speed Values:** Typical range is 2.0 (slow) to 5.0 (fast)
7. **Drop Chances:** Use `chance` field for rare drops (0.0 - 1.0)
8. **Undead Mobs:** Set `breath: -1` and `takes_sun_damage: true`
9. **Test Thoroughly:** Check movement, animation, spawning, drops, and combat

---

## Advanced Topics

### Complex Sprite Animations

For mobs with multiple animation states:
- Create multiple sprite files
- Use sprite switching in behavior code
- Example: Different textures for attacking vs. idle

### Boss Mobs

For special boss enemies:
- Increase HP significantly
- Add unique behavior class
- Create custom attack patterns
- Add special drops

### Tameable/Rideable Mobs

Requires code modifications:
- Extend `Mob` class with taming state
- Add interaction logic
- Modify behavior based on tamed state

---

## Next Steps

- **[Adding New Blocks](ADDING_BLOCKS.md)** - Create blocks for your mobs to interact with
- **[Adding New Items](ADDING_ITEMS.md)** - Create items your mobs can drop
- **[Modifying Game Mechanics](MODIFYING_MECHANICS.md)** - Change mob spawning and world generation

---

Need help? Check [ARCHITECTURE.md](../ARCHITECTURE.md) to understand how the mob system works internally!
