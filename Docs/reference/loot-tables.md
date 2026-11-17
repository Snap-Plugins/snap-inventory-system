# Loot tables

What a container may contain, as a shared asset — so fifty chests share one table and a designer rebalances all fifty by
editing one row.

**On this page:**

- [The asset](#the-asset)
- [A row](#a-row)
- [Nesting](#nesting)
- [Seeded rolls](#seeded-rolls)
- [Context tags](#context-tags)
- [Replacing the algorithm](#replacing-the-algorithm)
- [Why a container's table is a soft reference](#why-a-containers-table-is-a-soft-reference)
- [What the two shipped containers do differently](#what-the-two-shipped-containers-do-differently)
- [See also](#see-also)

## The asset

`USnapLootTable` is a data asset. Point a container at one and it rolls from it.

| Field | Means |
|---|---|
| `Rows` | The rows this table may yield |
| `NestedTables` | Other tables whose rows count as part of this one |
| `bUniquePicks` | No row is yielded twice by one roll |

A container supplies the rest, because it is the container's business rather than the table's:

- How many drops.
- What seed.
- What context it stands in.

That split is what lets one table serve a small crate that yields 1–2 things and a large chest that yields 1–5.

## A row

```cpp
FSnapLootRow
```

| Field | Default | Means |
|---|---|---|
| `ItemDef` | — | What the row yields (a soft reference) |
| `Count` | 1 | How many, or the LOW end of a range |
| `MaxCount` | -1 | The HIGH end, or -1 for "exactly `Count`" |
| `PickWeight` | 1.0 | Relative weight; **0 retires the row** |
| `MinDurabilityFraction` / `MaxDurabilityFraction` | 0.2 / 0.8 | The wear band, as a fraction of the item's effective max |
| `RarityOverride` | — | Pin the tier instead of rolling one |
| `bGuaranteed` | False | Always yielded, before any random picking |
| `bUnique` | False | At most once per roll, even when the table allows repeats |
| `RequiredContextTags` | — | Only eligible when the roll's context carries **all** of these |

Three rules are worth stating outright, because each is the kind of thing that is otherwise learned by being surprised.

**Weights are relative, not probabilities.**

- A table using 3/2/1 and one using 30/20/10 behave identically.
- Adding a row never requires rebalancing the others.

**A weight of zero retires a row completely — including a guaranteed one.**

- There is exactly one way to switch a row off, and it works the same way everywhere.
- The alternative — a guaranteed row that ignores its own weight — would mean a designer who zeroed a weight to retire a
  drop still saw it fall out of every container. That is the worse surprise by a wide margin.
- The combination is flagged when the asset is saved rather than left to be discovered in play.

**Guaranteed rows raise the drop budget rather than losing to it.**

- Three guaranteed rows in a container rolling 1–2 yield all three.
- A guarantee that silently lost to a budget would be the plugin telling you your quest key drops and then not dropping
  it.
- Whatever budget is left over is filled by weighted picks as usual.
- A guaranteed row stays eligible for those picks unless uniqueness says otherwise.

### Counts, and which opinion wins

A count can be stated in two places, so the precedence is fixed and announced:

1. **The row's own range** (`MaxCount` set) — The table stating a range outranks everything.
2. **The item's `USnapItemFragment_PickupAmount`** — The item saying what it is worth.
3. **`Count`** — The fixed fallback.

A row must be *explicit* to outrank the item.

- `Count = 3` on a coin that carries a pickup-amount fragment is ambiguous — it may be a deliberate "exactly three" or a
  leftover default.
  - So the fragment wins and nothing is guessed.
- Write `Count = 3, MaxCount = 3` to mean it.
- The log names the override either way.

> **Note:** A loot table is one of the places that fragment is *designed* to be rolled — along with a quest reward and a
> chest.
>
> - `ASnapPickupActor` still refuses it, and must keep refusing it.
> - A pickup carries a count that has to be conserved, or dropping three coins and picking them back up would launder
>   them into two hundred and fifty.

## Nesting

`NestedTables` flattens: a roll sees the table's own rows followed by every nested table's, with the weights relative
across the whole set. Nest one "common materials" table into five region tables and the materials are defined once.

- **A nested table's `bUniquePicks` and its `RollLoot` override are not consulted.** Both describe a *roll*, and there
  is only one roll — the root table's.
  - A nested table contributes rows and nothing else.
- **A cycle is survivable and is reported.** Each table is visited at most once per roll.
  - A diamond (a master table nesting five regions that each nest one shared table) is the same mechanism and is
    perfectly legitimate.

The shipped example uses it:

- `DA_ExampleChestLoot` holds the chest's own consumables and valuables.
- `DA_ExampleCrateLoot` holds the crate's.
- Both nest `DA_ExampleArmorLoot`, so the armor pool is defined once and both draw from it.
- The chest does not nest the crate's table.
  - The crate's coins, silver and gems land in the world as pickups, but a chest or a body is a container, and a
    container refuses a currency item.
  - Each one it rolled would be a warning in the log and a row it never holds.

## Seeded rolls

```cpp
USnapLootTableLibrary::RollLootTable(Table, Request);
```

The same table and the same seed produce the same results. That is what lets a project:

- Reproduce a dungeon.
- Replay a bug report.
- Hand every player on a server the identical chest without replicating its contents.

Set `LootSeed` on a container to fix its roll.

- **Zero — the default — means "take a fresh random seed"**, i.e. roll freshly.
- A constant default would make every container in a project roll identical loot forever, silently, and you would have
  to already know the field existed to find out why.

**What the seed covers.** Which rows, how many units, which tier, and how worn — everything `FSnapLootRollResult`
carries.

- It does **not** cover what happens to the item afterwards: where a pickup lands, what a stack merges into, or a tier
  some other system re-rolls later.
- Scatter physics is deliberately left on global RNG, because two breaks of a seeded crate producing a pixel-identical
  rosette of pickups reads as a bug rather than as a feature.

**The project's rarity table is an input to the roll**, not part of the seed.

- Two rolls of the same seed against two different rarity tables can differ in tier.

## Context tags

One shared table can still pay out differently in different places.

- A row's `RequiredContextTags` is tested against the container's `LootContextTags` with `HasAll`.
  - So the ice-cave chests also yield ice shards and the armory chests also yield blades — out of the same asset.
- An empty requirement means "always eligible", which is what every row written without context tags in mind already is.
- There is no "forbidden" list: a row that must be absent in one place is a row that is required everywhere else.
  - One rule that always reads the same way is worth more than two.

> ⚠ A gameplay tag your project has not declared imports as **empty**, so a requirement naming one is not a requirement
> at all — the row becomes eligible everywhere.
>
> - That is engine behavior for `FGameplayTagContainer` and it is silent.
> - Author these through the tag picker, which can only offer tags that exist.

## Replacing the algorithm

`RollLoot` is a `BlueprintNativeEvent`.

- Subclass `USnapLootTable` as a Blueprint and override it for pity timers, per-player luck, or a table that reads
  the time of day.
- Call `USnapLootTableLibrary::RollTableDefault` from the override to extend the default rather than replace it.

It **returns** its contribution.

- A UFUNCTION that contributes to a collection must return it.
- A non-const array parameter is an out parameter, and an override would receive an empty array and silently destroy
  everything gathered before it.

## Why a container's table is a soft reference

The reference is **soft** on purpose.

- A table is loaded only when a container rolls it (`USnapLootTableLibrary::RollLootTable`, once, on the server).
- So a Blueprint that names a table does not load it along with the Blueprint.
- Neither C++ container class names a table: the example table is set by the example Blueprint
  (`BP_Example_DestructibleCrate`), so a class every consuming project loads never names demo content.

## What the two shipped containers do differently

- `ASnapDestructibleContainer` names no table in C++. Its example, `BP_Example_DestructibleCrate`, ships **pointed
  at** `DA_ExampleCrateLoot`.
  - Because the example's whole purpose is to demonstrate the durability system.
  - A crate with no table drops nothing when it breaks, and logs a warning.
- `ASnapLootContainer` ships **empty**.
  - Because it is a general container a project places and fills.
  - Seeding it would mean every chest in your game silently contained the plugin's example armor.

## See also

- [Loot and interaction](loot-and-interaction.md) — pickups, drops, containers, bodies, the interact key
- [Durability, spoilage, weight](condition-and-weight.md) — what the wear band on a row is a fraction of
- [Items and fragments](items-and-fragments.md) — `USnapItemFragment_PickupAmount` and the rest
