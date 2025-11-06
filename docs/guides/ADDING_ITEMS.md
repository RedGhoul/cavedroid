# Adding New Items

This guide explains how to add new items to CaveDroid, including tools, food, and special items.

## Table of Contents

1. [Overview](#overview)
2. [Prerequisites](#prerequisites)
3. [Step 1: Create Item Texture](#step-1-create-item-texture)
4. [Step 2: Define Item in JSON](#step-2-define-item-in-json)
5. [Step 3: Add Crafting Recipe (Optional)](#step-3-add-crafting-recipe-optional)
6. [Step 4: Add Special Behavior (Optional)](#step-4-add-special-behavior-optional)
7. [Step 5: Test Your Item](#step-5-test-your-item)
8. [Item Type Reference](#item-type-reference)
9. [Examples](#examples)

---

## Overview

Items in CaveDroid are defined through JSON configuration. Different item types have different properties and behaviors:

- **Tools:** Pickaxes, axes, shovels, swords, shears
- **Food:** Consumables that restore health
- **Usable Items:** Buckets, spawn eggs, throwables
- **Placeable Items:** Items that place blocks (covered in [ADDING_BLOCKS.md](ADDING_BLOCKS.md))
- **Simple Items:** Crafting materials, resources

**Key File:** `/assets/json/game_items.json` - All item definitions

---

## Prerequisites

1. Development environment set up (see [QUICK_START.md](../QUICK_START.md))
2. Image editing software for 16x16 textures
3. Understanding of JSON format

---

## Step 1: Create Item Texture

Items use 16x16 pixel PNG textures.

### Create Your Texture

1. Create a 16x16 pixel PNG image
2. Match the existing art style
3. Save as `/assets/textures/items/your_item_name.png`

**Example:** For a "ruby" gem, create `/assets/textures/items/ruby.png`

### Texture Guidelines

- **Size:** Exactly 16x16 pixels
- **Format:** PNG with transparency
- **Style:** Match existing items
- **Naming:** Lowercase with underscores (e.g., `ruby_pickaxe`, `golden_apple`)

### Tool Texture Positioning

For held items (tools, weapons), consider the `origin_x` and `origin_y` properties to position the sprite correctly in the player's hand.

---

## Step 2: Define Item in JSON

Open `/assets/json/game_items.json` and add your item to the `"items"` section.

### Simple Item (Crafting Material)

The most basic item:

```json
{
  "items": {
    "ruby": {
      "name": "Ruby",
      "texture": "ruby"
    }
  }
}
```

This creates a stackable item with no special behavior.

### Common Properties

| Property | Description | Default |
|----------|-------------|---------|
| `name` | Display name in inventory | Required |
| `texture` | Texture filename (without .png) | Required |
| `type` | Item type (see Item Type Reference) | `"item"` |
| `max_stack` | Maximum stack size | 64 |
| `origin_x` | Horizontal sprite offset when held | 0.5 |
| `origin_y` | Vertical sprite offset when held | 0.5 |
| `tint` | Color tint as hex (e.g., `"#FF0000"`) | None |
| `burning_time` | Fuel value for furnaces (ticks) | None |
| `smelt_product` | Item produced when smelted | None |

---

## Step 3: Define Item Type

Items behave differently based on their `type` property. See the reference below for each type.

### Tool Items

**Pickaxe Example:**
```json
{
  "ruby_pickaxe": {
    "name": "Ruby Pickaxe",
    "type": "pickaxe",
    "texture": "ruby_pickaxe",
    "origin_x": 0.125,
    "tool_level": 4,
    "max_stack": 1,
    "durability": 2000,
    "block_damage_multiplier": 1.2
  }
}
```

**Tool Properties:**

| Property | Description | Required |
|----------|-------------|----------|
| `type` | Tool type: `"pickaxe"`, `"axe"`, `"shovel"`, `"sword"`, `"shears"` | Yes |
| `tool_level` | Mining tier (0-4) | Yes |
| `durability` | Number of uses before breaking | Yes |
| `max_stack` | Set to 1 for tools | Yes |
| `origin_x` | Usually `0.125` for tools | Recommended |
| `block_damage_multiplier` | Mining speed multiplier | No (default: 1.0) |
| `mob_damage_multiplier` | Melee damage multiplier (swords) | No (default: 1.0) |

**Tool Levels:**
- 0 = Hand/Wood tier
- 1 = Stone tier
- 2 = Iron tier
- 3 = Diamond tier
- 4 = Netherite/Custom tier

### Food Items

```json
{
  "golden_apple": {
    "name": "Golden Apple",
    "type": "food",
    "texture": "golden_apple",
    "heal": 10,
    "origin_x": 0,
    "origin_y": 1
  }
}
```

**Food Properties:**

| Property | Description |
|----------|-------------|
| `type` | Set to `"food"` |
| `heal` | Health restored (can be negative for poison) |
| `smelt_product` | Cooked version (e.g., raw beef → cooked beef) |

### Usable Items

Usable items trigger actions when right-clicked:

```json
{
  "ruby_bucket": {
    "name": "Ruby Bucket",
    "type": "usable",
    "texture": "ruby_bucket",
    "origin_x": 0.25,
    "action_key": "use_custom_bucket",
    "max_stack": 1
  }
}
```

**Usable Item Properties:**

| Property | Description |
|----------|-------------|
| `type` | Set to `"usable"` |
| `action_key` | Action identifier (requires code) |

**Built-in action_key values:**
- `"use_empty_bucket"` - Pick up water/lava
- `"use_water_bucket"` - Place water
- `"use_lava_bucket"` - Place lava
- `"use_milk_bucket"` - Drink milk
- `"use_spawn_egg"` - Spawn mob (requires `mob_key`)

### Armor Items

```json
{
  "ruby_helmet": {
    "name": "Ruby Helmet",
    "type": "helmet",
    "texture": "armor_helmet_ruby",
    "protection": 3,
    "max_stack": 1,
    "durability": 500,
    "armor_material": "ruby"
  }
}
```

**Armor Types:** `"helmet"`, `"chestplate"`, `"leggings"`, `"boots"`

**Armor Properties:**

| Property | Description |
|----------|-------------|
| `type` | Armor slot |
| `protection` | Defense points |
| `durability` | Uses before breaking |
| `armor_material` | Material identifier |
| `tint` | Color tint (for leather armor) |

---

## Step 3: Add Crafting Recipe (Optional)

Define how to craft your item in `/assets/json/crafting.json`.

### Shaped Recipe (Position Matters)

```json
{
  "crafting": {
    "ruby_pickaxe": {
      "input": [
        ["ruby", "ruby", "ruby"],
        ["", "stick", ""],
        ["", "stick", ""]
      ],
      "output": {
        "item": "ruby_pickaxe",
        "count": 1
      }
    }
  }
}
```

### Shapeless Recipe

For recipes where item positions don't matter:

```json
{
  "ruby_block_to_gems": {
    "input": [
      ["ruby_block", "", ""],
      ["", "", ""],
      ["", "", ""]
    ],
    "output": {
      "item": "ruby",
      "count": 9
    }
  }
}
```

**Note:** Empty strings (`""`) represent empty slots in the 3x3 crafting grid.

---

## Step 4: Add Special Behavior (Optional)

Most items work with just JSON. For custom behaviors, you need code.

### When Code is Required

- **Custom use actions** (new bucket types, magic items)
- **Special effects** (potions, enchanted items)
- **Unique mechanics** (grappling hooks, teleportation)

### Creating a Use Action

For usable items with custom `action_key`:

1. **Create Action Class:**

```kotlin
// In: core/gameplay/controls/src/main/kotlin/ru/fredboy/cavedroid/gameplay/controls/action/useitem/

@GameScope
@BindUseItemAction(stringKey = "use_custom_bucket")
class UseCustomBucketAction @Inject constructor(
    private val gameWorld: GameWorld,
    private val mobController: MobController
) : IUseItemAction {

    override fun perform(item: Item.Usable, x: Int, y: Int) {
        // Your custom logic here
        // Example: Place a special block, spawn particles, etc.
    }
}
```

2. **Register via Annotation:**
The `@BindUseItemAction` annotation automatically registers the action through Dagger multi-binding.

### Example: Creating a Throwable Item

```kotlin
@GameScope
@BindUseItemAction(stringKey = "throw_ruby_grenade")
class ThrowRubyGrenadeAction @Inject constructor(
    private val projectileController: ProjectileController,
    private val mobController: MobController
) : IUseItemAction {

    override fun perform(item: Item.Usable, x: Int, y: Int) {
        val player = mobController.player
        val projectile = Projectile.Ruby(
            position = player.position.cpy(),
            velocity = calculateVelocity(player, x, y)
        )
        projectileController.addProjectile(projectile)
        player.decreaseCurrentItemCount()
    }
}
```

---

## Step 5: Test Your Item

### 1. Launch Game

```bash
./gradlew desktop:run
```

### 2. Enable Creative Mode

Press `G` (if debug mode enabled) or start a Creative world.

### 3. Test Features

- **Find item in inventory** (press `E`)
- **Stack behavior** (test max_stack)
- **For tools:**
  - Mine appropriate blocks
  - Check durability decreases
  - Verify mining speed
- **For food:**
  - Consume item (right-click)
  - Check health restoration
- **For usable items:**
  - Right-click to activate
  - Verify custom behavior

### 4. Test Crafting

- Open crafting table
- Place ingredients
- Verify recipe works
- Check output item

---

## Item Type Reference

### Complete Type List

| Type | Description | Example |
|------|-------------|---------|
| _(default)_ | Simple item (crafting material) | Stick, coal, ruby |
| `"block"` | Placeable block item | Dirt, stone |
| `"slab"` | Placeable slab item | Stone slab |
| `"pickaxe"` | Mining tool | Diamond pickaxe |
| `"axe"` | Wood-cutting tool | Iron axe |
| `"shovel"` | Digging tool | Stone shovel |
| `"sword"` | Melee weapon | Diamond sword |
| `"shears"` | Special tool for leaves/wool | Shears |
| `"food"` | Consumable | Apple, bread |
| `"usable"` | Right-click action | Bucket, spawn egg |
| `"helmet"` | Head armor | Diamond helmet |
| `"chestplate"` | Torso armor | Iron chestplate |
| `"leggings"` | Leg armor | Chainmail leggings |
| `"boots"` | Foot armor | Leather boots |

### Property Summary by Type

**Simple Items:**
```json
{
  "name": "Display Name",
  "texture": "texture_name"
}
```

**Tools (pickaxe, axe, shovel, sword, shears):**
```json
{
  "name": "Tool Name",
  "type": "pickaxe",
  "texture": "texture_name",
  "origin_x": 0.125,
  "tool_level": 3,
  "max_stack": 1,
  "durability": 1562,
  "block_damage_multiplier": 1.0,
  "mob_damage_multiplier": 7.0
}
```

**Food:**
```json
{
  "name": "Food Name",
  "type": "food",
  "texture": "texture_name",
  "heal": 4,
  "smelt_product": "cooked_version"
}
```

**Usable:**
```json
{
  "name": "Usable Name",
  "type": "usable",
  "texture": "texture_name",
  "origin_x": 0.25,
  "action_key": "use_action_identifier",
  "max_stack": 1
}
```

**Armor:**
```json
{
  "name": "Armor Name",
  "type": "helmet",
  "texture": "armor_texture",
  "protection": 3,
  "max_stack": 1,
  "durability": 406,
  "armor_material": "diamond",
  "tint": "#AA00FF"
}
```

---

## Examples

### Example 1: Ruby Gem (Simple Item)

```json
{
  "items": {
    "ruby": {
      "name": "Ruby",
      "texture": "ruby"
    }
  }
}
```

Add crafting recipe:
```json
{
  "crafting": {
    "ruby": {
      "input": [
        ["ruby_block", "", ""],
        ["", "", ""],
        ["", "", ""]
      ],
      "output": {
        "item": "ruby",
        "count": 9
      }
    }
  }
}
```

### Example 2: Ruby Pickaxe (Tool)

```json
{
  "items": {
    "ruby_pickaxe": {
      "name": "Ruby Pickaxe",
      "type": "pickaxe",
      "texture": "ruby_pickaxe",
      "origin_x": 0.125,
      "tool_level": 4,
      "max_stack": 1,
      "durability": 2000,
      "block_damage_multiplier": 1.5
    }
  }
}
```

Crafting recipe:
```json
{
  "crafting": {
    "ruby_pickaxe": {
      "input": [
        ["ruby", "ruby", "ruby"],
        ["", "stick", ""],
        ["", "stick", ""]
      ],
      "output": {
        "item": "ruby_pickaxe",
        "count": 1
      }
    }
  }
}
```

### Example 3: Golden Carrot (Food)

```json
{
  "items": {
    "golden_carrot": {
      "name": "Golden Carrot",
      "type": "food",
      "texture": "golden_carrot",
      "origin_x": 0,
      "origin_y": 1,
      "heal": 15
    }
  }
}
```

### Example 4: Enchanted Sword

```json
{
  "items": {
    "enchanted_sword": {
      "name": "Enchanted Sword",
      "type": "sword",
      "texture": "enchanted_sword",
      "origin_x": 0.125,
      "tool_level": 4,
      "max_stack": 1,
      "durability": 3000,
      "mob_damage_multiplier": 10
    }
  }
}
```

### Example 5: Spawn Egg

```json
{
  "items": {
    "spawn_egg_dragon": {
      "name": "Dragon Spawn Egg",
      "type": "usable",
      "texture": "spawn_egg",
      "tint": "#FF00FF",
      "action_key": "use_spawn_egg",
      "mob_key": "dragon",
      "origin_x": 0.5
    }
  }
}
```

**Note:** Requires the "dragon" mob to be defined in `/assets/json/mobs.json`.

### Example 6: Ruby Helmet (Armor)

```json
{
  "items": {
    "ruby_helmet": {
      "name": "Ruby Helmet",
      "type": "helmet",
      "texture": "armor_helmet_ruby",
      "protection": 4,
      "max_stack": 1,
      "durability": 600,
      "armor_material": "ruby"
    }
  }
}
```

---

## Tips and Best Practices

1. **Consistent Naming:** Use `material_type` pattern (e.g., `ruby_pickaxe`, `iron_sword`)
2. **Balance Stats:** Compare with similar items for appropriate durability and damage
3. **Tool Levels:** Ensure `tool_level` makes sense in progression
4. **Stack Size:** Tools/armor should have `max_stack: 1`
5. **Origin Points:** For held items, use `origin_x: 0.125` (tools) or `origin_x: 0.25` (buckets)
6. **Durability Guidelines:**
   - Wood: ~60
   - Stone: ~130
   - Iron: ~250
   - Diamond: ~1560
   - Custom/Legendary: 2000+
7. **Test Thoroughly:** Check all interactions (crafting, using, breaking)

---

## Common Issues

### Item Doesn't Appear

- Verify JSON syntax is valid
- Check texture file exists at correct path
- Ensure `name` and `texture` are defined

### Item Texture Missing

- Texture must be in `/assets/textures/items/` or `/assets/textures/blocks/`
- Filename must match texture property (without `.png`)
- File must be exactly 16x16 pixels

### Tool Not Mining Properly

- Check `tool_level` is appropriate
- Verify `type` matches required tool (pickaxe, axe, shovel)
- Ensure blocks require the correct tool type

### Food Not Healing

- Verify `type: "food"` is set
- Check `heal` value is defined
- Positive values heal, negative values damage

### Crafting Recipe Not Working

- Verify all input items exist
- Check recipe structure (3x3 grid)
- Empty slots must be `""`
- Output item must exist

---

## Next Steps

- **[Adding New Blocks](ADDING_BLOCKS.md)** - Create placeable blocks
- **[Adding New Mobs](ADDING_MOBS.md)** - Create creatures
- **[Modifying Game Mechanics](MODIFYING_MECHANICS.md)** - Change core systems

---

## Advanced Topics

### Custom Tool Behavior

To create tools with unique behaviors (e.g., a pickaxe that auto-smelts):

1. Extend tool damage logic in block break handlers
2. Add custom actions triggered on block break
3. Use `OnBlockDestroyedListener` to hook into break events

### Item Metadata

For items that store data (enchantments, NBT-like data):

1. Extend `Item` sealed class with new type
2. Add serialization support in data layer
3. Update inventory rendering for special display

---

Need help? Check [ARCHITECTURE.md](../ARCHITECTURE.md) for how items work internally!
