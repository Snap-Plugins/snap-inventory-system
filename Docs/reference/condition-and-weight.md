# Durability, spoilage and weight

Three optional systems, each switched on by a setting and each driven by a fragment. This page covers durability (wear,
breaking and repair), spoilage and carry weight.

**On this page:**

- [Durability](#durability)
- [Spoilage](#spoilage)
- [Carry weight](#carry-weight)
- [See also](#see-also)

---

## Durability

The `bEnableDurability` setting turns it on. The durability settings are on **Project Settings → Plugins →
Snap Inventory System - Items → Durability** and its sub-categories (`DamageWearBoneToSlot` and `bUseDefaultMannequinBoneMap` are on **Snap Inventory System - Equipment**).

- An item with a `Durability` fragment carries **wear points**.
- It breaks when they run out.

```
MaxDurability          points a new instance starts with (100), before any rarity bonus -- not a fraction
ConsumeDurabilityCost  above 0, consuming costs this many points instead of a unit (a multi-use tool)
BreakBehaviorOverride  what a broken item may still do, overriding the project-wide setting
```

Wear is stored as **points** rather than as a fraction, on purpose.

- Points survive a rarity change, where a fraction would silently re-scale an item's remaining life.

**A worn multi-use tool** (`ConsumeDurabilityCost` above 0, with `bEnableDurability` on), consumed from its equipment
slot:

- It spends durability instead of a unit, and applies its ability and effect payload.
- Its consume grants (a recipe unlock, a capacity upgrade) are skipped, silently. Don't put a durability cost and a
  consume-grant fragment on one item.
- The durability is spent before the payload. A payload that is then refused (no ability system, or no consume ability
  activated) has still cost the durability.
  - That is logged, and it is the deliberate trade against free repeated effects.

### Two ways to wear

- **Use wear** — spending or using the item.
- **Damage wear** — a worn item takes wear when its owner's health attribute (`DamageWearHealthAttributeName`) falls,
  through the `USnapDamageWearComponent` the server makes beside the equipment. A project whose damage is not that
  attribute calls `USnapDamageWearComponent::ApplyDamageWear(HealthLost, HitBoneName)` from its own damage code.
  - It is scaled per slot and per rarity tier (`DamageWearMultiplier` on the rarity table).

**The per-slot scale** comes from **one** of two places, never both:

- `DamageWearMultiplierBySlot` (**Snap Inventory System - Items → Durability → Damage Wear**), while no equipment slot set is in
  force.
- Or the in-force set's own row for that slot.

⚠ A set **replaces** the map rather than filling gaps in it.

- So a row left at the default `1.0` means "this slot wears at the normal rate" — not "look the multiplier up in the map".
- See [Equipment](equipment.md#slots).

**How much a hit wears:**

- A hit's wear is the health it took times `DamageWearPointsPerHealthPoint` (0.1 by default) times those two multipliers.
- It is **rounded to the nearest point, with no per-hit minimum**.
- So at the defaults a hit has to take 5 health before it wears a point. A smaller one wears nothing.
- That is on purpose: a floor of one point per hit would let a fast damage-over-time effect shred a full set of armor in
  seconds.
- Raise the rate if your game's hits are small.

**The reading never goes over the cap:**

- `GetCurrentDurability` is always clamped to the current effective maximum and never returned raw.
- So a rarity change cannot leave an item reading above its own cap.

### Breaking

- A broken item stays equipped and stops granting.
- `OnItemBroke` fires.

### Repair

There are three routes: a repair kit, a shop and repair in the field. The shop answers through two describe/mutate pairs
on the player's `USnapVendorClientComponent`, by item and by worn slot (below; their `Server_` forms need an open vendor
session at a shop whose stock has `bOffersRepair` on). The kit and field repair have their own pairs on the bag, in the
list after it:

```
DescribeItemRepair(Instance, ...)     RepairItem(Instance)
DescribeEquipmentRepair(Slot, ...)    RepairEquippedSlot(Slot)
```

- **A repair kit** — an item with the `RepairKit` fragment, dropped on the item: in the bag, in a pouch, or worn (on its
  equipment slot), from the bag or from a pouch window.
  - `DescribeRepairKitUse` is the one verdict the drop plan previews and `ApplyRepairKit` re-asks.
  - Over an item with durability that has nothing to repair, the drop is refused (`NotDamaged`).
  - A kit that names the kinds it repairs (`RepairableItemTags`, empty = all) is refused over any other
    (`CannotBeRepaired`).
  - Over an item with no durability, it swaps as any item does.
  - Or use the kit from its own item menu: see [Repairing with Use](#repairing-with-use) below.
- **A shop** — paid from the currency counter. See [vendor](vendor.md).
- **In the field** — no shop open and no kit carried.
  - Units of `FieldRepairMaterialItem` from the bag, through `DescribeItemFieldRepair` and `FieldRepairItem`.
  - It ships off, because that setting (`FieldRepairMaterialItem`) ships empty.

#### Repairing with Use

1. Choose **Use** on the kit's own item menu.
   - The pointer carries the kit's drag picture (sized to its square), centered on it.
   - On a pad, the picture sits on the square that has focus instead.
   - The arrow is hidden over the bag and its windows (`HeldItemPointer`, `Centered` by default; `Beside` puts it below
     and to the right of a visible arrow).
   - You should see the green Repair badge over an item the kit would repair, and the red X over anything else
     (`USnapInventoryHUDComponent::BeginRepairTargeting`, `CanRepairTargetItem`).
2. **Left-click a repairable item.** That spends one use through `Server_ApplyRepairKit`.

**To cancel:** A right click, Escape, or a click on anything else ends it, and nothing is spent.

- **On a gamepad:** Accept on the focused item is the click and Close cancels, as for a carry.
- **Closing the bag** (or a pouch window, a shop or a trade) ends it too.

### Where durability is shown

- On the **item tooltip**, and only there.
- The grid badge that used to carry a number was removed — a badge is the most width-starved text in the UI.
- `USnapInventoryTooltipWidget::StatusTextWidgetName` names the TextBlock.
- If your tooltip has none, no item shows durability anywhere, and the plugin warns.
- The line reads "Durability 92 / 150", with "(Broken)" at zero, and shows at every level, full included.
- Its color is the durability band (`USnapInventoryComponent::GetDurabilityBandTint`). A broken item's is
  `DurabilityBrokenTextTint`, a red.
- With `bEnableDurability` off there is no line.

---

## Spoilage

The `Spoilage` fragment gives an item a shelf life and something it becomes when it expires. The spoilage settings are on
**Project Settings → Plugins → Snap Inventory System - Items → Spoilage**.

- **A sweep runs periodically** and converts expired items.
- The clock is **custody-rate based**. An item in a cooler ages more slowly than one in a bag.
  - The reconciler is self-healing, so an item that changes custody mid-life does not lose or gain time at the boundary.

**Example:** `/SnapInventorySystem/Examples/Items/ID_Example_Meat` spoils into `ID_Example_RottenMush` after five minutes.

- `ID_Example_Cooler` is a pouch that slows it.

**An item on the ground ages too:**

- A dropped item carries its own reading.
- A pickup that arrives with none — placed in a level, from a loot table, out of a crate, from a spawner — is given the
  item's full life when it appears.
  - So food lying in the world is already spoiling when it is picked up.
- A placed pickup that respawns comes back fresh.

**A spoiled item** is marked with a terminal tag rather than being re-checked forever.

**On the tooltip:**

- The status line reads "Fresh 72% (4m 12s)", or "Spoiled". There is no line with `bEnableSpoilage` off.
- The countdown is real seconds under the item's current custody (`GetWallClockRemainingSeconds`). In a cooler at rate
  0.5 it shows the longer time the item really has, and falls one per second.
- The percent is the item's freshness, how much of its life is left. A cooler slows it but never reverses it, so it
  doesn't jump when the item moves.
- In a container that stops the clock (rate 0) the line shows the percent and no countdown.

**With `bEnableSpoilage` off,** the whole feature is switched off:

- No clock is written.
- The sweep does nothing.
- Merges do not blend.
- Prices are not scaled.
- No slot draws a spoilage bar.

---

## Carry weight

The `bEnableCarryWeight` setting turns it on. The weight settings are on **Project Settings → Plugins →
Snap Inventory System - Inventory → Carry Weight** and its sub-categories (`DefaultItemWeightKilograms` is on **Snap Inventory System - Items →
Carry Weight**).

- Each unit's `Weight` fragment contributes kilograms to a carried total.
- Exceeding the limit slows the carrier down.
- Dropping back under undoes it.

**What the carried total counts:** The grid, every pouch at every depth, items offered in a trade (escrow is still
yours, so offering ore does not lighten you), and worn items when `bCarryWeightIncludesEquipment` is on (the default).

- It does not count ingredients a queued timed craft has consumed: queueing removes them.
  - So a player can park heavy ingredients in the queue, travel light, and cancel to get them back.
  - If that matters, gate crafting on a station.
- Only the main bag's `MaxCarryWeightKilograms` is read. Setting it on another container does nothing.

**Two ways to slow the player down.** They are independent — use either, both or neither:

- A **direct** speed scale, on by default (`bEnableOverweightSlowdown`).
  - `MaxWalkSpeed` is multiplied by `OverweightSpeedMultiplier`, which covers **walking and falling only**.
  - Name your movement component's own scale function in `OverweightSpeedScaleFunctionName`, and the plugin drives that
    instead.
  - A multiplier of 1.0 turns this route off.
  - `OverweightSlowdownMode` chooses its shape:
    - **Flat** (the default) applies `OverweightSpeedMultiplier` however far over the carrier is.
    - **Proportional** slows a carrier more the further over it is: 1.0 just over capacity, falling in a straight line to
      `OverweightProportionalFloor` at `OverweightProportionalFullSlowdownAt` over capacity (0.25 at 50 percent over, as
      shipped). It is re-applied whenever the load moves it.
  - The rule is `USnapEncumbranceComponent::ComputeOverweightSpeedMultiplier`.
- A **gameplay effect**, opt-in.
  - Name one in `OverweightGameplayEffect` (empty by default), and it is applied to the owner's ability system while over
    the limit.
  - It needs `GameplayAbilities`. Without it, the setting does nothing, silently.

**For code that asks every frame** — a sprint that refuses an overloaded carrier, say:

- `USnapEncumbranceComponent::IsOverCarryCapacityCached` is the last sweep's "over capacity" with no inventory walk.
- It reports the load whatever the slowdown switch says.
- It does so on the server and the owning client (the two machines that sweep).

⚠ With `OverweightSpeedScaleFunctionName` set, the direct route calls a function **your project** provides, resolved
reflectively by name.

- The name the apply used is remembered.
- So a later change to the setting cannot leave a scale applied through one name and removed through another.

**Capacity can be bought:** An item with the `CapacityUpgrade` fragment grants bonus slots and bonus carry weight when
consumed.

```
GetEffectiveMaxNumSlots()
ApplyCapacityUpgrade(Tier, BonusSlots, BonusCarryWeightKilograms)
WouldAcceptCapacityUpgrade(Tier)
```

- Upgrades are **tiered**, so that consuming a lower tier after a higher one does not downgrade the bag.
- **A shop does not sell an upgrade the bag already has, at that tier or a better one.** Its row is dimmed and BUY is
  grayed, and the server refuses the sale too (`USnapInventoryComponent::DescribeCapacityUpgrade`).
- **Using such an upgrade says why it is refused:** "Your inventory already has this upgrade or a better one." The item
  menu's Consume row is drawn dimmed with that sentence, and a double-click on it in the bag says it, naming the item.
- **The bag can show the room the upgrades could still add**, as locked squares (`bShowUnboughtBagSquares`, off by
  default; see [the README](../../README.md#carrying-more--purchasable-capacity-upgrades)).

**The weight readout** is shown by the inventory panel.

- `WeightUnit` chooses kilograms, pounds or an icon.

---

## See also

- [Items and fragments](items-and-fragments.md)
- [Vendor](vendor.md) — paying for a repair
- [Appendix: settings](appendix-settings.md) — the **Durability**, **Spoilage** and **Carry Weight** categories, 59 settings
  between them
