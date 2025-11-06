# Adding New Blocks

This guide walks you through adding a new block to CaveDroid step by step.

## Table of Contents

1. [Overview](#overview)
2. [Prerequisites](#prerequisites)
3. [Step 1: Create Block Texture](#step-1-create-block-texture)
4. [Step 2: Define Block in JSON](#step-2-define-block-in-json)
5. [Step 3: Define Item in JSON](#step-3-define-item-in-json)
6. [Step 4: Add Crafting Recipe (Optional)](#step-4-add-crafting-recipe-optional)
7. [Step 5: Add Special Behavior (Optional)](#step-5-add-special-behavior-optional)
8. [Step 6: Test Your Block](#step-6-test-your-block)
9. [Block Property Reference](#block-property-reference)
10. [Examples](#examples)

---

## Overview

Blocks in CaveDroid are defined primarily through JSON configuration files. The game automatically loads these definitions on startup.

**Key Files:**
- `/assets/json/game_items.json` - Block and item definitions
- `/assets/json/crafting.json` - Crafting recipes
- `/assets/textures/blocks/` - Block textures

**No Code Required** for basic blocks! Only special behaviors require code changes.

---

## Prerequisites

1. Development environment set up (see [QUICK_START.md](../QUICK_START.md))
2. Image editing software for creating 16x16 textures
3. Basic understanding of JSON format

---

## Step 1: Create Block Texture

Blocks use 16x16 pixel PNG textures.

### Create Your Texture

1. Create a 16x16 pixel PNG image
2. Use appropriate style to match existing blocks
3. Save as `/assets/textures/blocks/your_block_name.png`

**Example:** For a "ruby_block", create `/assets/textures/blocks/ruby_block.png`

### Texture Guidelines

- **Size:** Exactly 16x16 pixels
- **Format:** PNG with transparency support
- **Style:** Match the existing game aesthetic
- **Naming:** Use lowercase with underscores (e.g., `ruby_block`, `smooth_stone`)

### Optional: Animated Textures

For animated blocks:
1. Create multiple frames: `block_name_0.png`, `block_name_1.png`, etc.
2. Define animation in JSON (see Block Property Reference)

---

## Step 2: Define Block in JSON

Open `/assets/json/game_items.json` and add your block definition to the `"blocks"` section.

### Basic Block Example

```json
{
  "blocks": {
    "ruby_block": {
      "hp": 1500,
      "texture": "ruby_block",
      "tool_level": 3,
      "tool_type": "pickaxe",
      "drop_info": [
        {
          "key": "ruby_block",
          "amount": {
            "type": "exact_amount",
            "amount": 1
          }
        }
      ],
      "material": "stone"
    }
  }
}
```

### Required Properties

| Property | Description | Example |
|----------|-------------|---------|
| `hp` | Block durability (higher = takes longer to break) | `1500` |
| `texture` | Name of texture file (without .png) | `"ruby_block"` |
| `tool_type` | Required tool to mine efficiently | `"pickaxe"`, `"axe"`, `"shovel"` |
| `tool_level` | Tool tier required (0-4) | `0` (wood), `3` (diamond) |
| `drop_info` | What item(s) drop when mined | See examples below |
| `material` | Material type for sounds | `"stone"`, `"wood"`, `"dirt"`, `"grass"` |

### Optional Properties

| Property | Description | Default |
|----------|-------------|---------|
| `collision` | Has collision physics | `true` |
| `transparent` | Light passes through | `false` |
| `casts_shadows` | Casts shadows in lighting | `true` |
| `replaceable` | Can be replaced by other blocks | `false` |
| `light_info` | Emits light (see Light Emitting Blocks) | None |
| `tint` | Color tint as hex | None |
| `burning_time` | Can be used as furnace fuel (ticks) | None |
| `density` | Fluid density (for water/lava) | None |

---

## Step 3: Define Item in JSON

Blocks need a corresponding item definition in the `"items"` section so they can be held in inventory.

### Basic Item Definition

```json
{
  "items": {
    "ruby_block": {
      "name": "Ruby Block",
      "type": "block",
      "texture": "ruby_block"
    }
  }
}
```

### Item Properties

| Property | Description | Required |
|----------|-------------|----------|
| `name` | Display name | Yes |
| `type` | Item type: `"block"`, `"placeable_item"`, etc. | Yes |
| `texture` | Texture name (can differ from block) | Yes |
| `max_stack` | Maximum stack size | No (default varies) |
| `burning_time` | Fuel value if used in furnace | No |

---

## Step 4: Add Crafting Recipe (Optional)

If your block should be craftable, add a recipe to `/assets/json/crafting.json`.

### Crafting Recipe Example

```json
{
  "crafting": {
    "ruby_block": {
      "input": [
        ["ruby", "ruby", "ruby"],
        ["ruby", "ruby", "ruby"],
        ["ruby", "ruby", "ruby"]
      ],
      "output": {
        "item": "ruby_block",
        "count": 1
      }
    }
  }
}
```

### Recipe Structure

- **`input`**: 3x3 grid of ingredient keys (use `""` for empty slots)
- **`output`**: Resulting item and count

### Shapeless Recipes

For recipes where position doesn't matter:
```json
{
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
```

---

## Step 5: Add Special Behavior (Optional)

Most blocks work automatically with just JSON. For special behaviors, you need to write code.

### When Code is Required

- **Interactive blocks** (chests, doors, furnaces)
- **Block updates** (grass spreading, crops growing)
- **Custom physics** (special collision rules)
- **Triggered actions** (pressure plates, buttons)

### Adding Block Update Logic

Block updates run periodically via `UpdateBlockAction`.

**Example:** Making a block grow over time

1. **Create Action Class:**

```kotlin
// In: core/gameplay/controls/src/main/kotlin/ru/fredboy/cavedroid/gameplay/controls/action/updateblock/

@GameScope
@BindUpdateBlockAction(stringKey = "your_block")
class YourBlockUpdateAction @Inject constructor(
    private val gameWorld: GameWorld
) : IUpdateBlockAction {

    override fun update(x: Int, y: Int, elapsed: Long) {
        // Your logic here
        // Example: Change block after certain time
        if (elapsed > 100) {
            gameWorld.setForeMap(x, y, gameWorld.itemsRepository.getBlock("grown_block"))
        }
    }
}
```

2. **Register via Dagger:**
The `@BindUpdateBlockAction` annotation automatically registers it via multi-binding.

### Adding Use/Interaction Logic

For blocks that do something when clicked:

1. **Create Action Class:**

```kotlin
// In: core/gameplay/controls/src/main/kotlin/ru/fredboy/cavedroid/gameplay/controls/action/useblock/

@GameScope
@BindUseBlockAction(stringKey = "your_block")
class YourBlockUseAction @Inject constructor(
    private val gameWorld: GameWorld,
    private val mobController: MobController
) : IUseBlockAction {

    override fun perform(x: Int, y: Int, player: Player) {
        // What happens when player right-clicks the block
        // Example: Open a custom window, trigger an effect, etc.
    }
}
```

---

## Step 6: Test Your Block

### 1. Launch the Game

```bash
./gradlew desktop:run
```

### 2. Enable Creative Mode

Press `G` (in debug mode) or create a new world in Creative mode.

### 3. Test Your Block

- Open inventory (press `E`)
- Find your block
- Place it in the world
- Mine it to test drops
- Test any special behaviors

### 4. Common Issues

**Block doesn't appear in inventory:**
- Check JSON syntax (valid JSON)
- Ensure item definition exists
- Check texture file exists

**Block texture is missing:**
- Verify texture path: `/assets/textures/blocks/your_texture.png`
- Check texture name in JSON matches filename
- Texture must be exactly 16x16 pixels

**Block can't be mined:**
- Verify `tool_type` and `tool_level` are set correctly
- Check `hp` is not too high for testing

**Block doesn't drop items:**
- Verify `drop_info` is correctly defined
- Check that the dropped item exists in items section

---

## Block Property Reference

### Complete Property List

```json
{
  "your_block": {
    // === REQUIRED ===
    "hp": 100,                         // Durability (mining time)
    "texture": "block_texture",        // Texture filename (no .png)
    "tool_type": "pickaxe",           // Required tool: "pickaxe", "axe", "shovel", "shears"
    "tool_level": 0,                  // Tool tier: 0=wood, 1=stone, 2=iron, 3=diamond, 4=netherite
    "material": "stone",              // Sound material: "stone", "wood", "dirt", "grass", "gravel", "sand"

    // === DROP INFO (required) ===
    "drop_info": [
      {
        "key": "dropped_item",        // Item to drop
        "amount": {
          "type": "exact_amount",     // or "random_range"
          "amount": 1                 // or "min" and "max" for random_range
        },
        "requires_tool": true         // Optional: only drops with correct tool (default: true)
      }
    ],

    // === OPTIONAL: PHYSICS ===
    "collision": false,               // Has collision (default: true)
    "transparent": true,              // Transparent to light (default: false)
    "casts_shadows": false,           // Casts shadows (default: true)
    "replaceable": true,              // Can be replaced (like tall grass) (default: false)

    // === OPTIONAL: RENDERING ===
    "tint": "#FF0000",                // Color tint (hex format)
    "animation_info": {               // Animated texture
      "animation_frames": 4,
      "frame_time": 100
    },

    // === OPTIONAL: LIGHTING ===
    "light_info": {
      "light_distance": 15            // Emits light (0-15, like torch = 13)
    },

    // === OPTIONAL: SLABS ===
    "meta": "slab",                   // Mark as slab
    "top": 8,                         // Top collision boundary (for bottom slab)
    "bottom": 8,                      // Bottom collision boundary (for top slab)
    "sprite_top": 8,                  // Sprite rendering offset
    "sprite_bottom": 8,
    "full_block": "double_stone_slab", // Full block when two slabs stacked
    "other_part": "stone_slab_top",   // Counterpart slab

    // === OPTIONAL: FUEL ===
    "burning_time": 5000              // Fuel value for furnaces (ticks)
  }
}
```

### Drop Info Types

**Exact Amount:**
```json
"drop_info": [
  {
    "key": "item_name",
    "amount": {
      "type": "exact_amount",
      "amount": 1
    }
  }
]
```

**Random Range:**
```json
"drop_info": [
  {
    "key": "wheat_seeds",
    "amount": {
      "type": "random_range",
      "min": 0,
      "max": 3
    }
  }
]
```

**Multiple Drops:**
```json
"drop_info": [
  {
    "key": "leaves_oak",
    "amount": { "type": "exact_amount", "amount": 1 },
    "requires_tool": true
  },
  {
    "key": "sapling_oak",
    "amount": { "type": "exact_amount", "amount": 1 },
    "requires_tool": false
  }
]
```

---

## Examples

### Example 1: Simple Stone Block

```json
{
  "blocks": {
    "smooth_stone": {
      "hp": 600,
      "texture": "smooth_stone",
      "tool_level": 1,
      "tool_type": "pickaxe",
      "drop_info": [
        {
          "key": "smooth_stone",
          "amount": {
            "type": "exact_amount",
            "amount": 1
          }
        }
      ],
      "material": "stone"
    }
  },
  "items": {
    "smooth_stone": {
      "name": "Smooth Stone",
      "type": "block",
      "texture": "smooth_stone"
    }
  }
}
```

### Example 2: Light-Emitting Block

```json
{
  "blocks": {
    "glowstone": {
      "hp": 30,
      "texture": "glowstone",
      "tool_level": 0,
      "tool_type": "pickaxe",
      "casts_shadows": false,
      "light_info": {
        "light_distance": 15
      },
      "drop_info": [
        {
          "key": "glowstone_dust",
          "amount": {
            "type": "random_range",
            "min": 2,
            "max": 4
          }
        }
      ],
      "material": "stone"
    }
  },
  "items": {
    "glowstone": {
      "name": "Glowstone",
      "type": "block",
      "texture": "glowstone"
    }
  }
}
```

### Example 3: Transparent Block (Glass)

```json
{
  "blocks": {
    "glass": {
      "hp": 30,
      "texture": "glass",
      "tool_level": 0,
      "tool_type": "pickaxe",
      "transparent": true,
      "casts_shadows": false,
      "drop_info": [],
      "material": "stone"
    }
  },
  "items": {
    "glass": {
      "name": "Glass",
      "type": "block",
      "texture": "glass"
    }
  }
}
```

Note: Empty `drop_info` means it drops nothing (glass breaks without dropping).

### Example 4: Slab Block

```json
{
  "blocks": {
    "oak_slab_bottom": {
      "top": 8,
      "sprite_top": 8,
      "hp": 180,
      "transparent": true,
      "meta": "slab",
      "texture": "planks_oak",
      "full_block": "planks_oak",
      "other_part": "oak_slab_top",
      "tool_level": 0,
      "tool_type": "axe",
      "drop_info": [
        {
          "key": "oak_slab",
          "amount": {
            "type": "exact_amount",
            "amount": 1
          }
        }
      ],
      "material": "wood"
    },
    "oak_slab_top": {
      "bottom": 8,
      "sprite_bottom": 8,
      "hp": 180,
      "transparent": true,
      "meta": "slab",
      "texture": "planks_oak",
      "full_block": "planks_oak",
      "other_part": "oak_slab_bottom",
      "tool_level": 0,
      "tool_type": "axe",
      "drop_info": [
        {
          "key": "oak_slab",
          "amount": {
            "type": "exact_amount",
            "amount": 1
          }
        }
      ],
      "material": "wood"
    }
  },
  "items": {
    "oak_slab": {
      "name": "Oak Slab",
      "type": "slab",
      "texture": "planks_oak"
    }
  }
}
```

---

## Tips and Best Practices

1. **Start Simple:** Begin with a basic block before adding special behaviors
2. **Copy Existing Blocks:** Find a similar block and modify it
3. **Test Frequently:** Run the game after each change to catch errors early
4. **Consistent Naming:** Use lowercase with underscores throughout
5. **Match Materials:** Use appropriate material for correct sounds
6. **Balance HP Values:** Consider existing blocks for reference
7. **Tool Levels:**
   - 0 = Wood tools
   - 1 = Stone tools
   - 2 = Iron tools
   - 3 = Diamond tools
   - 4 = Netherite tools (if implemented)

---

## Next Steps

- **[Adding New Items](ADDING_ITEMS.md)** - Create tools, food, and other items
- **[Adding New Mobs](ADDING_MOBS.md)** - Create creatures and NPCs
- **[Modifying Game Mechanics](MODIFYING_MECHANICS.md)** - Change core gameplay systems

---

## Troubleshooting

### JSON Syntax Errors

Use a JSON validator (online or in your IDE) to check for:
- Missing commas
- Unclosed brackets
- Invalid property names

### Game Won't Start After Adding Block

1. Check console/logs for error messages
2. Verify JSON is valid
3. Temporarily remove your changes to isolate the issue
4. Check all referenced items/textures exist

### Block Appears but Looks Wrong

- **Black/Pink texture:** Texture file not found
- **Wrong texture:** Check texture name in JSON
- **Wrong size:** Texture must be exactly 16x16

Need more help? Check [ARCHITECTURE.md](../ARCHITECTURE.md) for deeper understanding of how blocks work internally.
