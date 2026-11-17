# Items and fragments

An item is described by a **definition** asset and carried as an **instance** object.

- The definition is shared and immutable.
- The instance is what the player owns, and it is where per-item state such as a rarity roll or a durability value
  lives.

**On this page:**

- [USnapItemDefinition](#usnapitemdefinition)
- [USnapItemInstance](#usnapiteminstance)
- [The twenty-six shipped fragments](#the-twenty-six-shipped-fragments)
- [The rarity table](#the-rarity-table)
- [Writing your own fragment](#writing-your-own-fragment)
- [Reading item data from Blueprint](#reading-item-data-from-blueprint)
- [Validation](#validation)
- [See also](#see-also)

## `USnapItemDefinition`

A `UPrimaryDataAsset` with four fields of its own — `DisplayName`, `Description`, `ItemTags`, `Fragments` — and
everything else composed in.

```
Content/MyItems/ID_IronSword       (a USnapItemDefinition)
  DisplayName   "Iron Sword"
  Description   "Plain, heavy, reliable."
  ItemTags      Item.Type.Weapon
  Fragments
    UIData          icon, flavor text, fixed color
    Equippable      SlotTag = Equipment.Slot.MainHand
    Durability      MaxDurability = 200
    Weight          3.5 kg
```

Create one with **right-click > Snap Inventory > Item Definition**, or copy one of the 24 under
`/SnapInventorySystem/Examples/Items/`.

### `ItemTags`

A `FGameplayTagContainer` on the definition, for describing **what an item is** when no fragment already says so.

- It is the fourth and last thing the filter bar, the vendor's type column, the craft panel's category headings and the
  `Category` sort all read.
- The other three are an Equippable fragment's slot tag, a Container Routing tag and a fixed rarity tier.

Nine names ship, declared in the plugin's own `Config/Tags`:

```
Item.Type.Weapon      Item.Type.Armor       Item.Type.Accessory
Item.Type.Consumable  Item.Type.Material    Item.Type.Container
Item.Type.Currency    Item.Type.Quest       Item.Type.Tool
```

- They are a starting vocabulary, not a fixed one — a project is free to use its own names, and a filter row matches
  whatever tags you give it.
- Matching is **hierarchical**, because the tags are added through `FGameplayTagContainer::AddTag` and tested with
  `HasAny`.
  - A row naming the parent `Item.Type` therefore claims every item carrying any child of it — one row for
    "everything with a type at all".

Two Blueprint nodes read it:

| Node | Returns |
|---|---|
| `Get Item Tags` | The definition's whole container, empty for a null definition |
| `Has Item Tag` | Whether it has the tag, **including through a parent** |

⚠ An item that **no filter row can claim** is invisible the moment any filter is selected, prints no type word in a
vendor list and has no rank under the `Category` sort.

- The definition's validator warns about that on save, so it cannot reach a build unnoticed — but only in a project
  whose rows have no catch-all among them.
- The four rows the plugin ships include one, so by default every item is classified and the warning is silent.

## `USnapItemInstance`

The runtime object. One instance can hold several **units** of a stackable item, so "three potions" is one instance with
a count of three, not three objects.

Per-instance state that a definition cannot carry:

| State | Set by |
|---|---|
| Unit count | Stacking, splitting, merging |
| Rarity variant | Rolled at mint from the rarity table, or pinned by `FixedRarity` |
| Durability wear | Damage-wear, use-wear, repair |
| Spoilage clock | The spoilage sweep |
| `InstanceId` | Minted once, stable across a save and a trade |

`InstanceId` is an `FGuid` and is the identity everything outside the component uses — a saved hotbar binding, a world
pickup, a trade escrow entry.

## The twenty-six shipped fragments

Every one is optional. A definition with no fragments at all is a valid, inert item.

| Fragment | Declares |
|---|---|
| `UIData` | Icon (and an optional Learned Icon), flavor text, a fixed color |
| `Stackable` | Max stack size, and an optional per-definition unit cap for a container |
| `Equippable` | Which slot **type** the item accepts, matched by tag hierarchy |
| `Requirements` | Who may equip the item: owner tags it needs or forbids, attribute minimums, a level |
| `Rarity` | Roll a tier per instance at mint, or pin one with `FixedRarity` |
| `GrantedAbilitySet` | Abilities and effects applied while equipped; plus the one-shot consume payload |
| `ContainerRouting` | Which container the item belongs in, or that it is a slotless counter |
| `PickupAmount` | How many units a world pickup of this item is worth |
| `Weight` | What one unit weighs, in kilograms |
| `ItemStorage` | Makes the item a container — a pouch — with its own capacity, and optionally a grid width, a weight limit for its contents and how fast they spoil |
| `Durability` | Wear points, a break threshold, and what a broken item may still do |
| `RepairKit` | Marks the item as a repair consumable, and what it restores |
| `ConsumeCondition` | "Not now": refuses using the item while a stat is full — a health potion at full health says "Your health is full" (`RefuseWhenFullStat`, `FullAtStat`, `RefusalText`); nothing is spent and no cooldown starts |
| `Spoilage` | A shelf life, and what the item becomes when it spoils |
| `Upgradeable` | The definition this item upgrades into, and the cost |
| `CapacityUpgrade` | How many extra grid slots consuming this item grants |
| `RecipeUnlock` | Which crafting recipes consuming this item teaches |
| `Hotbar` | Which hotbar the item may be bound to — consumable, ability, or neither |
| `Footprint` | How many cells the item covers in a container laid out as a spatial grid |
| `WorldRepresentation` | What the item looks like dropped on the floor, and which actor a drop spawns |
| `Affixes` | Which affix table the item rolls from, and how many — see [Affixes](affixes.md) |
| `StatContributions` | The stats the item adds while worn, as data — see [Equipment](equipment.md#stats-an-item-declares-itself) |
| `Value` | What one unit is worth (`BaseValue`) and the counter it is priced in (`CurrencyTag`; empty means the project's vendor currency) — see [Currency and prices](currency-and-prices.md) |
| `Restrictions` | Whether the item may be dropped, sold, traded or dismantled, whether a death drops it, and whether it binds to its holder — see [Restrictions and marks](restrictions-and-marks.md) |
| `ItemSet` | The item set it is a piece of, whose tiers apply while enough pieces are worn — see [Item sets](item-sets.md) |
| `Presentation` | Its own lift, place, pickup, equip, consume, break and drop sounds (overriding its type's — see [Sounds](sounds.md)), and an accent color — see [Pickup modes and the collect feed](pickup-modes-and-the-collect-feed.md) |

`Hotbar` is worth calling out: **absence is the refusal.**

- An item with no `Hotbar` fragment can never be bound to a bar.
- So nothing has to be listed anywhere to exclude recipes, currency or materials.

`Footprint` works the same way round: **an item without one is a single cell**, and it is a single cell because nothing
asks it for a shape, not because a default was chosen for it. Add the fragment only to the items that should be bigger:

| Property | Means |
|---|---|
| `Width`, `Height` | The bounding box, in cells |
| `CellMask` | Which cells of that box the item covers. Leave it empty for a plain rectangle |
| `bCanRotate` | Whether the item may be turned in quarter steps. Off by default |

- An L-shape or a T-shape is a `CellMask` with some entries unticked, read row by row.
- The box has to be tight (no empty row or column).
- The covered cells have to form one piece — cells that meet only at a corner do not count as joined.
- Asset validation says so at save and at cook if they do not.

In the editor the mask is painted rather than typed:

- The fragment shows a grid of its box, the item's icon behind it and an outline of each turn it may take.
- A click covers or uncovers a cell. See [Editor tools](editor-tools.md#painting-a-footprint).
- However the mask is edited — the grid, a typed or dragged `Width` or `Height`, a paste, Python — changing the box
  keeps every painted cell where it was (a shrink crops, a grow adds uncovered cells).
- A mask that covers the whole box is stored empty, which is what a rectangle is.

⚠ **Cover the top-left cell, or it is held but unused.**

- An item is stored by the top-left cell of its box, so a spatial grid reserves that cell for the item even when the
  shape leaves it empty.
  - Nothing else can be placed there, and a packing that needs that exact cell is refused.
- Asset validation **warns** — the item stays valid — when the top-left cell is empty as authored or in any turn the
  item may take.
- An L with its corner cut away, and a T that may turn, are the shapes to check.
- This is listed under **Known limitations** in the plugin README.

The fragment does nothing at all in a container laid out in `Slots`, so footprints can be authored before any container
is switched over.

`WorldRepresentation` is the item's own answer to how it looks on the floor. Without it, a dropped item is drawn as one
of four engine shapes picked by its name and tinted by its rarity:

| Property | Means |
|---|---|
| `Mesh` | What the plugin's own pickup draws instead of a fallback shape, with its own materials (no rarity tint) |
| `Scale` | A multiplier on `Mesh`'s modeled size; 1 draws it as modeled. Read only together with `Mesh` |
| `DropActorClass` | The actor a drop spawns instead of the component's `DroppedPickupClass`. A configured row still wins |

- `Mesh` never replaces a mesh a pickup already has — a Blueprint pickup subclass with its own art keeps it.
- A project actor chosen by `DropActorClass` draws itself, so it ignores `Mesh` and `Scale`.
- A mesh that does not load falls back to the shape and says so in the log.
- Asset validation warns about a fragment with nothing set, and about a `Scale` with no `Mesh`.
- Every example item carries one, naming its own model. See [Your own drop actor](extending.md#8-your-own-drop-actor).

## The rarity table

`USnapItemRarityTable` maps each tier's tag to a `FSnapItemRarityEntry`. One table serves every item.

- Point at yours with `RarityTable` under `[/Script/SnapInventorySystem.SnapItemFragment_Rarity]` in your
  `DefaultGame.ini` (see the README's [Where the settings live](../../README.md#where-the-settings-live)).

| Column | Means |
|---|---|
| `Color` | The tier's tint. Linear, not sRGB |
| `Weight` | Relative pick weight; 0 retires the tier from rolls but keeps its look |
| `Rank` | The order, lower is worse. Gaps are fine, but ranks must be unique; -1 means unset, and validation fails the table |
| `DamageWearMultiplier` | How fast the tier wears from hits. Negative means unset: read as 1.0, and validation fails the table |
| `DisplayName` | Shown in the tooltip above the item's name |
| `GrantedEffect`, `EffectLevel` | An effect applied while an item of the tier is worn. Use a Has Duration or Infinite effect with Scalable Float magnitudes |

- `RollRarityWeighted` rolls with per-call weights, so a crate can be more generous than a barrel through the same
  roller.

**Stacks never mix tiers.**

- A pickup or grant whose units would join a stack of another tier gets a stack of its own instead.
- Dragging one stack onto a stack of another tier is refused.
- Units that have not rolled their tier yet cannot be proven to match, so a stackable item that rolls its rarity starts
  a new stack on every pickup. Drag stacks of the same tier together to combine them.
- An item with no Rarity fragment, or one that pins `FixedRarity`, stacks normally.

## Writing your own fragment

`USnapItemFragment` is `Blueprintable`, so a fragment can be authored entirely in Blueprint. In C++:

```cpp
UCLASS(DisplayName = "Soulbound")
class USoulboundFragment : public USnapItemFragment
{
    GENERATED_BODY()
public:
    virtual void OnInstanceCreated_Implementation(USnapItemInstance* Instance) const override;
    virtual bool CanEquip_Implementation(const USnapItemInstance* Instance, const AActor* Owner) const override;
};
```

**Note:** In `const AActor* Owner` the parameter is const, and an override that drops the `const` will not match the
base declaration.

The lifecycle hooks are `BlueprintNativeEvent`s: `OnInstanceCreated`, `OnEquipped`, `OnUnequipped`, `CanEquip`, and the
admission hooks described in [extending](extending.md).

### A fragment that makes its item a consumable

Three more hooks let a fragment of your own be what happens when its item is consumed — with no ability and no effect,
which is how a project without GameplayAbilities has consumables at all. Override all three together:

| Hook | Asked | Base |
|---|---|---|
| `DeclaresConsumeGrant(ItemDef)` | "Does consuming this do something I am responsible for?" True makes the item offered **Consume** in the bag and on the hotbar. Asked on clients — answer from authored data. | False |
| `CanApplyConsumeGrantTo(ItemDef, GrantTarget)` | "Could it land right now?" Before anything is applied: false refuses the whole consume and the item keeps its unit, and the bag menu does not draw a Consume that could only refuse. Client-reachable — read only replicated state. | True |
| `ApplyConsumeGrantTo(OwningActor, ItemDef, GrantTarget)` | The consume itself, on the **server**, after any ability or effect payload and before the unit is spent. It cannot refuse. | Nothing |
| `GetConsumeCooldownFor(Instance, Asker, Remaining, Duration)` | *Optional.* The cooldown a refusal in `CanApplyConsumeGrantTo` is waiting on, so the item's hotbar square counts it down. Asked on the owning client, about ten times a second while it runs — answer from replicated state. | None |

- `GrantTarget` is the player's main bag and is never null when the last two are asked.
- A worn item in one of the `ConsumeSlots` (**Snap Inventory System - Foundation → Consume**) is consumed through its slot, and the same three hooks run there.
- ⚠ A lone `ApplyConsumeGrantTo` never runs: the hook runs only on a fragment whose `DeclaresConsumeGrant` answers true,
  even beside an ability or effect; with no other payload, Consume is not offered at all.
- ⚠ `ApplyConsumeGrantTo` is server-only, so presentation done in it is invisible to clients.
- The older `CanApplyConsumeGrant` and `OnConsumed` still work: the new hooks' bases call them (both are deprecated).

⚠ **A fragment hook runs inside the mutator that called it.**

- Adding or removing items from inside one re-enters the component.
- The equipment path iterates a copy of the fragment array for exactly this reason; do the same if you iterate.

### A use that takes time

By default every use is instant. **Project Settings > Snap Inventory System - Inventory > Consume > Item Use Seconds** gives uses a
time, by item tag:

- Each row is a tag and a number of seconds.
- An item takes the time of the most specific row its `ItemTags` match (`Item.Type.Consumable` beats `Item.Type`).
- An item's Presentation fragment can give it its own time instead (`bOverrideUseSeconds` and `UseSeconds`; 0 there
  keeps that item instant).

```ini
[/Script/SnapInventorySystem.SnapInventorySettings]
+ItemUseSeconds=(("Item.Type.Consumable", 0.6))
```

A use with a time **starts at the press and lands at its moment**:

| When | What happens |
|---|---|
| The press | `OnItemUseStarted(ItemDef, Action, Seconds)` on the player's **main container**, on every machine — play your animation from it. Nothing is applied or spent yet. |
| The moment | On the server, the door an instant use runs (`ConsumeItem`, or `ConsumeSlot` for a worn consumable slot) runs now: it re-checks everything and applies the effect and spends the item **together**. The cooldown starts now, and the collect feed's line appears now. |
| The end | `OnItemUseEnded(ItemDef, Action, bLanded)` on every machine. `bLanded` false means nothing was spent. |

- **An interrupted use costs nothing.** A death (the main container's `IsOwnerDead`, asked every tenth of a second and
  again at the moment), a new pawn, or the item leaving the bag or its worn slot ends the use unlanded.
  - Walking does not, and neither does opening a screen — most uses start from the bag.
- **A second use waits for the first.** A press made while a use is under way is kept — the latest one — and run from
  the top when the first one ends.
  - A death, a new pawn or `CancelItemUse` forgets it.
- **A press that would be refused now is refused at once,** exactly as an instant use would be — a cooldown still
  running, a broken or a spoiled item.
  - No use starts.
- **What takes time:**
  - `Server_ConsumeItem` (the bag, the hotbar, the item menu, a double-click, the selectable hotbar's use).
  - `Server_ConsumeSlot` (the consume key's worn slots).
  - `Server_MixItems` (the dragged item's time).
  - `Server_ApplyRepairKit` (the kit's time).
  - `ConsumeItem`, `ConsumeSlot`, `MixItems` and `ApplyRepairKit` called directly on the server stay instant.
- **A use path of your own:** `USnapItemUseComponent::TryStartTimedItemUseFor` (C++) takes a pre-check, the function
  that lands the use, a still-valid test and the press to re-run if it has to wait.
  - `IsUsingItem`, `GetItemUse`, `CancelItemUse` and `GetItemUseSeconds` are Blueprint-callable.
- **The item is in the hand while it is used** (`bShowItemInHandDuringUse`, on; it, `ItemInHandLingerSeconds` and
  `ConsumeAbilityTag` are on **Snap Inventory System - Inventory → Consume**, the `HeldItem*` settings on **Snap Inventory System - Equipment →
  Hotbar → Selectable**):
  - The selectable hotbar's held layer draws a potion or a scroll as its world model at `HeldItemSocketName` /
    `HeldItemAttachOffset`, for every player.
  - It puts the worn look in `HeldItemSlot` away for the use.
  - It keeps the item for `ItemInHandLingerSeconds` after the use lands so the hand lowers with it.
  - Nothing is drawn while the selectable hotbar already holds something — unless what it holds is the weapon already
    worn in the hand (`IsSelectionDrawnInHand` false), which the use puts away like any worn look.

⚠ A project that plays its use through `ConsumeAbilityTag` already waits for its own animation before it calls the
consume door: leave **Item Use Seconds** empty there, or every use waits twice.

## Reading item data from Blueprint

`USnapItemLibrary` answers both the definition and the instance form of each question (the instance display name and
icon are on `USnapInventoryLibrary`). The instance form is the one you usually want, because it accounts for the rarity
roll:

```
GetItemDisplayName(Definition)      GetInstanceDisplayName(Instance)
GetItemIcon(Definition)             GetInstanceIcon(Instance)
GetItemRarityColor(Definition)      GetInstanceRarityColor(Instance)
IsItemStackable(Definition)         IsInstanceStackable(Instance)
GetItemEquipSlotTag(Definition)     GetInstanceEquipSlotTag(Instance)
```

`GetInstanceIcon` also accounts for where the item is:

- An item held by a container that keeps what enters it (a learned spell in a spellbook) draws its `UIData`
  **Learned Icon** when it has one, and its `Icon` everywhere else.
- `GetItemLearnedIcon(Definition)` is the definition form (the `Icon` when there is no Learned Icon).
- A container panel's squares (the bag, a custom container such as a spellbook), the equipment squares, the hotbar, the
  drag picture, the collect feed's "Learned" line and the buff row ask these.
- A pouch window's squares and the loot, vendor and craft rows draw the `Icon`.

The name follows the same rule:

- `GetInstanceDisplayName` answers the `UIData` **Learned Name** while the item is kept that way.
  - The example "Scroll of Fireball" is called "Fireball" in the Spells panel, on its hotbar square's card and in its
    item menu.
- Everywhere else it answers the item's own name (with its affixes).
- `GetItemLearnedName(Definition)` is the definition form (the `DisplayName` when there is no Learned Name).
  - The collect feed's "Learned" line uses it ("Learned Fireball").

`FindFragment(Definition, Class)` and `FindInstanceFragment(Instance, Class)` reach anything the library does not wrap.

## Validation

`USnapItemDefinition::IsDataValid` runs on save, on **Validate Assets**, and at cook. It refuses the combinations
that would fail silently at runtime:

- A `Rarity` fragment that rolls (no `FixedRarity`) with no rarity table, or with one whose path does not load.
- A storage item with `StorageSlots` below 1.
- A storage item that is also `Equippable`.
- A stackable storage, `Durability` or `Affixes` item (`MaxStackSize` above 1).
- A `Spoilage` chain (`SpoiledItemDef`) that loops back to the item.

It also **warns**, without failing the item, about:

- A project item (outside `/SnapInventorySystem/Examples`) that rolls from the plugin's own example rarity table.
- An `Equippable` fragment with a `HolsterOffset` but no `HolsterSocketName`, which is never applied.
- An item no filter row can claim, in a project whose rows have no catch-all.
- A `Stat Contributions` row whose stat the configured stat display table has no row for.
  - The row still counts, but no tooltip draws its line.
  - With no table set it says nothing, because that is how stat lines are turned off.

Then each fragment's own validator runs: Affixes, CapacityUpgrade, Footprint, Hotbar, ItemSet, ItemStorage, Requirements,
Restrictions, StatContributions, Upgradeable, Value and WorldRepresentation.

**The rarity table validates itself as well**, and fails a tier with no rank. That matters more than it looks:

- A new field on a struct stored inside an already-authored asset keeps its C++ initializer.
- So an unmigrated tier would compare equal to every other and the upgrade feature would simply never fire.
- The validator is the layer that catches it.
- With `GameplayAbilities` it warns about a tier whose `GrantedEffect` takes a magnitude from `SetByCaller`, which
  contributes nothing there — see [Equipment](equipment.md).

## See also

- [Inventory and containers](inventory-and-containers.md) — where instances live
- [Durability, spoilage, weight](condition-and-weight.md) — the condition fragments in depth
- [Extending the plugin](extending.md) — admission hooks and policies
