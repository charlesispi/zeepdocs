
# ZEEP - Item Mod Strings

**Mod strings** define how an item changes gameplay stats when picked up. They’re short expressions that tell the game what to modify and how to calculate the effect. Each item in *Zeep* uses a mod string to apply stat changes automatically—to the player, weapon, or world—every time the item is added to the inventory.

These changes are defined through **modifier segments**, which make up the full mod string.

> **Note:** A mod string is applied once for each copy of the item in the player’s inventory. To control the **maximum number of times** a modifier can be applied, set the item’s `max_quantity` field accordingly. For example, if your item is only meant to apply its effect once, set `max_quantity` to 1—this ensures the player can never carry more than one, and thus the modifier won't stack beyond that.

## 1. Stat Overview

Before you can write mod strings, you must understand what a stat is in the context of Zeep.

**Stats** represent numerical values that define various aspects of gameplay. Every stat has a **stat name**, which is always lowercase and always includes dots (e.g., `game.player.speed`, `game.weapon.damage`, `game.player.kills`).

**There are two categories of stats:**

- **Tracked stats** are read-only values that reflect gameplay activity or current conditions. Examples include `stat.player.kills`, `stat.player.distance_traveled`, or `stat.player.in_elite_room`.  *These **cannot** be modified by a mod string but **can be read** in expressions.*

- **Gameplay stats** are values that represent permanent or semi-permanent attributes of the player, weapon, or world. Examples include `game.player.speed`, `game.player.max_health`, or `game.weapon.damage`.  *These **can** be modified by a mod string and **can be read** in expressions.*

Some stats — both tracked and gameplay — are **boolean-style**, meaning they represent true or false using `1` and `0`, respectively. For example, a boolean gameplay stat might control whether a weapon is automatic, while a boolean tracked stat might indicate whether the player is in an elite room.

## 2. Basic Mod String Structure

A full mod string is made up of one or more **modifier segments**, separated by commas. Each segment modifies a stat, either by multiplying it, adding to it, subtracting from it, or setting its value.

**Example:**
```
game.player.speed*1.2,game.weapon.damage+10
```
This mod string has two **modifier segments**:
- `game.player.speed*1.2`
- `game.weapon.damage+10`

> NOTE: Whitespace is irrelevant— it’s automatically stripped out and has no effect on parsing.

**Breakdown of each line:**

| Modifier Segment        | Description                        |
|------------------------|------------------------------------|
| `game.player.speed*1.2`     | Multiply `game.player.speed` by 1.2     |
| `game.weapon.damage+10`    | Add 10 to `game.weapon.damage`         |

This mod string, when attached to an item in a players inventory, would increase the player's speed by 20% and add 10 to their weapon's damage.

>The modifier segment expressions in this case are simple numeric values: `1.2` and `10`.

## 3. Modifier Segment Format

Each modifier segment follows this structure:

```
<target stat><operator><modifier segment expression>
```

### 3.1 Target Stat

The **target stat** is the stat name being modified.

> A complete table of valid stat names can be found at the end of this documentation.

### 3.2 Operator

| Symbol | Description                                  |
|--------|----------------------------------------------|
| `+`    | Add the expression result to the stat        |
| `-`    | Subtract the expression result from the stat |
| `*`    | Multiply the stat by the expression result   |
| `=`    | Set the stat to the expression result        |

The first occurrence of `+`, `-`, or `*` in the modifier segment determines the operator. Everything before it is the target stat; everything after it is the modifier segment expression.

> **Note:** The equals sign (`=`) is not valid syntactically inside a modifier segment expression.

### 3.3 Modifier Segment Expression Basics

The **modifier segment expression** (or just **expression**) is the value or expression that tells the game how much to add, subtract, or multiply. This can be:

-   A single number (such as in the prior example)
    
-   A full mathematical expression (e.g. `1+0.5*3`)

The latter is explained in further detail in the next section.
    

## 4. Modifier Segment Expression Format

**Modifier segment expressions**, or **expressions**, as mentioned, are the part of a modifier segment that comes **after the operator**. They are either a single predetermined value or a math expression.

### 4.1 Using Expressions Mathematically

You can tell the game to calculate, on the fly, the value to be used in a modifier segment, instead of a predetermined value. 
For example:
```
game.player.max_health+5*2+30
```
- The first `+` is the operator.
- The **modifier segment expression** is: `5*2+30`
- This expression is evaluated using standard order of operations:
  - First, `5 * 2 = 10`
  - Then, `10 + 30 = 40`

**Result:** This modifier segment will add 40 to `game.player.max_health`.

These operations are supported:

- `+` (addition)
- `-` (subtraction)
- `*` (multiplication)
- `/` (division)
- Parentheses `()` for grouping


## 5. Stat References in Expressions

Expressions can include **stat references** to read current, past, or changing values from the game. These make item effects dynamic and context-sensitive, allowing them to scale based on player progress or gameplay conditions. For example, a stat reference could increase an item’s effectiveness based on how many chests have been opened, how far the player has traveled, or how many enemies were killed this level.

### 5.1 Syntax

```
%[prefix:]<stat name>%
```

Stat references are always wrapped in percent signs `%`.

### 5.2 Prefix Options

| Prefix     | Description                                                  |
|------------|--------------------------------------------------------------|
| *(none)*   | Current value of the stat                                    |
| `initial:` | Value the stat had when the item was picked up               |
| `delta:`   | Difference since the item was picked up                      |
| `stage:`   | Difference since the beginning of the current stage          |
| `level:`   | Difference since the beginning of the current level          |

These can be used with both tracked and gameplay stats. Using prefixes allows item effects to scale based on specific moments in gameplay. For example, you can increase damage based on kills since entering the level, or apply a speed boost depending on rooms discovered before pickup.

### Example Modifier Segment:

```
game.weapon.damage*1+0.01*%delta:game.player.kills%
```

**Result:** Multiply `game.weapon.damage` by 1.12 if the player has 12 kills since pickup.

## 6. Example Mod Strings

Here's a few examples to really exemplify the utility and power of the mod string.

### Example 1: Make weapon automatic and boost damage per kill this level

```
game.weapon.automatic=1,game.weapon.damage*1+0.01*%level:stat.player.kills%
```

**Result:** Sets the weapon to automatic and increases damage based on level kills.

### Example 2: Bonus damage per room entered before pickup

```
stat.weapon.damage*1+0.1*%initial:stat.player.rooms_entered%
```

**Result:** Multiplies weapon damage based on the number of rooms entered before picking up the item.

# 7. Available Stats
>Not a complete list. To be updated.

| **Stat Name**                  | **Description**            | **Stat Type**             |
|-------------------------------|-----------------------------|---------------------------|
| stat.player.in_elite_room          | In elite room?              | Tracked, read only        |
| stat.player.kills                  | Enemies killed              | Tracked, read only        |
| stat.player.nearest_enemy_distance | Distance to nearest enemy   | Tracked, read only        |
| game.player.max_health             | Max health                  | Gameplay, read and write  |
| game.player.speed                  | Movement speed              | Gameplay, read and write  |
| stat.weapon.consecutive_hits       | Hits without missing        | Tracked, read only        |
| game.weapon.automatic              | Automatic fire enabled      | Gameplay, read and write  |
| game.weapon.bullet_bounces         | Bullet bounces allowed      | Gameplay, read and write  |
| game.weapon.damage                 | Base weapon damage          | Gameplay, read and write  |
| game.weapon.fire_cooldown          | Time between shots          | Gameplay, read and write  |
| game.weapon.knockback              | Force on hit                | Gameplay, read and write  |
| game.weapon.splash_damage          | Area damage                 | Gameplay, read and write  |
| game.weapon.spread                 | Bullet spread angle         | Gameplay, read and write  |

