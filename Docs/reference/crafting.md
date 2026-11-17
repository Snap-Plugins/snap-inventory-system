# Crafting

Recipes live in a `USnapCraftingRecipeSet` data asset. This page covers what a recipe holds, crafting, dismantling, discovery,
timed queues, stations, the UI and mixing items.

- **Crafting** spends ingredients and currency and mints an output.
- **Dismantling** reverses it at a fraction.

**On this page:**

- [A recipe](#a-recipe)
- [Crafting](#crafting-1)
- [Dismantling](#dismantling)
- [Discovery](#discovery)
- [Timed queues](#timed-queues)
- [Stations](#stations)
- [The UI](#the-ui)
- [Mixing items](#mixing-items)
- [See also](#see-also)

## A recipe

```
FSnapCraftingRecipe
  RecipeId            FName, the stable identity
  OutputDef           USnapItemDefinition*
  OutputCount         how many units one craft yields
  OutputRarityMode    how the output's tier is chosen; OutputRarity the tier it names
  Ingredients         TArray<FSnapCraftingIngredient>  (definition + count)
  CurrencyCost        paid from the counter
  CurrencyTag         which counter CurrencyCost is paid from (empty = inherit; see currency-and-prices)
  RequiredStationTag  which station is required, if any (empty = craftable anywhere)
  bAllowDismantle     may the output be taken apart again
  bUnlockedByDefault  known from the start, or taught by an item
  CraftDurationSeconds  0 crafts at once; above 0 the craft joins the timed queue
```

- `/SnapInventorySystem/Examples/Items/DA_ExampleRecipes` is the demo's set.
- The `DefaultRecipeSet` setting ships empty.
  - The example recipes are demo content, which the demo map's session sets for itself.

**An ingredient's tier** (`RequiredRarity` on the ingredient line):

- Empty accepts any tier. A named tier matches exactly: a Rare line takes Rare bars only, not Godly.
- A dismantle returns that tier.
- One definition may appear on two lines at two different tiers, but not on a tiered line and a tier-agnostic line
  together (validation refuses it).

**The output's tier:** `OutputRarityMode` Fixed with an empty `OutputRarity` gives the rarity table's lowest tier; Random
rolls.

- An output definition that pins its own `FixedRarity` always wins.

## Crafting

The crafting functions on this page are on `USnapCraftingComponent`, which sits beside the main bag on the PlayerState
(the plugin adds it there), and so are their client-reachable `Server_` forms (the main bag keeps same-named doors for
all but `Server_UnlockRecipe`). The crafting settings are on
**Project Settings → Plugins → Snap Inventory System - Crafting**, except `bCraftingConsumesFromSiblingContainers`
(**Snap Inventory System - Inventory → Crafting**) and `bFieldRepairDrawsFromPouches` (**Snap Inventory System - Items → Durability →
Repair**).

```
DescribeCraft(...)                 -> what a craft would cost and whether it can run
EvaluateCraftPlan(...)             the shared planner
CraftRecipe(...)                   Server_CraftRecipe(...)
```

`DescribeCraft` and `CraftRecipe` ask the same planner, so the button's enabled state and the server's answer come from one
place.

- **A craft the server refuses is told to the player who pressed CRAFT.** For example, the ingredients changed since the
  panel looked. The notice names what the recipe makes. A full bag refuses a craft only with `bCraftWhenBagFull` off, and
  then reads "Your inventory is full." (see [Timed queues](#timed-queues)).

⚠ `DescribeCraft` is reachable from a client, so none of its terms may read `NotReplicated` state.

- A client would otherwise compute "you cannot afford this" from a default-constructed snapshot and disable a legal craft.

⚠ `DescribeCraft`'s `OutMaxCraftable` is bounded by ingredients and currency only, not by room for the output or the carry
limit.

- A "craft max" button should also bound it with `GetRemainingUnitAllowance` (a carry limit still refuses a craft), and with
  `CanAddItem` only when `bCraftWhenBagFull` is off (with it on, what does not fit waits).

### Where ingredients come from

Three places, each behind its own switch:

| Source | Switch | Default |
|---|---|---|
| This component's own grid | Always | — |
| Storage items (pouches) inside it, recursively | `bCraftingConsumesFromPouches` | On |
| The owner's **other** inventory components | `bCraftingConsumesFromSiblingContainers` | **Off** |

**Why the third source exists:**

- `USnapItemFragment_ContainerRouting` can send an item to a container that is a *sibling component* rather than a pouch —
  a shard bag, an ability bag.
- Without it, an ingredient that routes itself out of the main bag is out of reach of the very scan that would spend it.
  - The recipe can never be satisfied.
  - The refusal is `MissingIngredients` for materials the player is visibly carrying.

⚠⚠ **It is off by default, and that is not timidity.**

- A second container is often a deliberate *safe* place.
  - A project that routes quest items or ability shards out of the main bag did so to stop them being spent by accident.
- Switching this on reaches into all of them at once.
- It is a decision about a project's own containers, so the project makes it.

⚠ **One level, never transitive.**

- A sibling is searched with its own pouches but is never asked about *its* siblings.
- Otherwise every component would count every other one repeatedly, double-count a shared unit, and never terminate.

⭐ **This component's own holdings are always spent first.**

- So turning the switch on cannot change what a craft takes from the main bag.
- It only adds a source for what the main bag could not cover.

⚠ The count and the spend both honor it.

- Each removal step records **which component owns the instance**, so the unit is removed from the right ledger.
- A count that could see units the spend cannot reach is the defect this whole path is written to avoid.

⚠ **Field repair is unaffected.**

- It has its own setting, `bFieldRepairDrawsFromPouches` (Durability), for the reason it always did — it does not borrow
  crafting's switches.
- It counts through the local-only form, so it can never report materials it would then fail to spend.

## Dismantling

```
EvaluateDismantle(...)
DismantleItem(...)                 Server_DismantleItem(...)
ResolveDismantleYield(Ingredient, OutputCount, ReturnFraction)   static
```

- **The yield** is a fraction of the recipe's ingredients, per ingredient.
  - It is rounded by one shared function, so the preview and the result agree.
- **What can be taken apart:** An item in the grid, or in a pouch the bag holds.
  - A worn item must be taken off first.
  - A pouch that still holds something is refused.
- **A dismantle takes one batch: the recipe's `OutputCount` units, from one stack.**
  - The recipe is the unit of salvage. If one craft makes 3 repair kits, 3 kits from the same stack come apart together.
  - A stack holding fewer is refused, and so are kits spread over two stacks.
  - A non-stackable output with `OutputCount` above 1 can never be dismantled: make it stackable, or set `OutputCount`
    to 1.
- **Each ingredient line returns, per batch,** `floor(Count x DismantleReturnFraction)`.
  - A one-unit line returns 1 while the fraction is above 0, rather than flooring to nothing.
  - A line's `DismantleReturnOverride` replaces that with the override times `OutputCount` (-1 = computed, 0 = nothing
    back).
  - It never returns more than the line's `Count`, so crafting and dismantling in a loop cannot make material.
    Validation refuses an override that would.
- **Each yield comes back at the tier its recipe line required** (`RequiredRarity`), not the tier of the item taken
  apart.
  - So a dismantle reverses the craft, and an upgraded item can't be salvaged into better materials.
  - A line that names no tier yields at the rarity table's lowest-ranked tier (by `Rank`, never by drop weight).
  - With no table, or no tier with a `Rank` set, the yield keeps its own roll, and the log warns once per session.
  - A yield definition with `FixedRarity` keeps that tier. One with no Rarity fragment is minted as authored (logged
    once per definition).
- **The salvage recipe's own gates apply.** It must be unlocked, and made at its station if it names one.
- **Room for every yield is checked, together, before the item is touched.**
- **Two dismantlable recipes with the same output:** Salvage always uses the first (validation warns).
- **An item whose dismantle would yield nothing is refused** rather than silently consumed.
  - The count and the gate use the same rule.
- **A recipe that lists one definition twice at the same tier** is refused at validation.
  - Two such lines would be spent twice and returned once.

## Discovery

- A recipe with `bUnlockedByDefault = false` is hidden until taught.
- The `RecipeUnlock` fragment on an item names the recipes consuming it teaches.

```
IsRecipeUnlocked(RecipeId)
UnlockRecipe(RecipeId)             Server_UnlockRecipe(RecipeId)
```

⚠ `Server_UnlockRecipe` is an RPC a client can send. It is guarded — the server checks the request is legitimate rather
than trusting the caller.

Locked recipes show as silhouettes in the craft panel, so a player can see there is something to find.

## Timed queues

A craft does not have to be instant. With a duration, it enters a **queue** that drains over time.

```
GetCraftQueue()
CancelQueuedCraft(EntryId)         Server_CancelQueuedCraft(EntryId)
```

**Each entry carries:**

- A stable `EntryId` that is never reused.
- The remaining units.
- A finish time.
- A **snapshot** of what it cost.
  - The snapshot is what a cancel refunds, so changing a recipe mid-queue cannot change what a pending entry pays back.
  - A refunded ingredient comes back at the tier its line asked for (`RequiredRarity`), on a cancel as on every other
    refund.

Two behaviors worth knowing:

- **Queueing the same recipe again coalesces** into the entry already doing that work rather than appending a second line.
- **A refund that cannot fit conserves.**
  - If only part of a refund fits in the bag, the entry shrinks by what actually landed, rather than clearing or staying
    whole.
  - The rest lands when there is room.

**Each crafting station, each crafting shop and the bag's own craft window keeps its own queue** (`bCraftQueuePerStation`, on
by default):

- A craft shows only in the window it was ordered from, and CLEAR there cancels only those crafts.
- Each queue crafts one entry at a time on its own clock (`bSequentialCraftQueue`) and holds `MaxCraftQueueLength` rows of its
  own, so a shop's craft and a forge's count down together.
- An entry's host is its `SnapshotStationKey`, replicated to the owner: the station's or shop's key
  (`SnapCraftDelivery::MakeStationKey`), or None for the bag's own. A station spawned at runtime has no lasting key, so its
  crafts are the bag's.
- Off, every window shows the player's one queue, with one slot and one row limit.

**A full bag never stops a craft** (`bCraftWhenBagFull`, on by default):

- CRAFT stays enabled with a full bag, the quantity is not capped by room, and `DescribeCraft` and `CraftRecipe` never answer
  `InventoryFull` for a craft. A Blueprint gate on that answer no longer sees a full bag.
- A unit that finishes with no room waits in the queue it was ordered from, drawn as a finished tile (a full bar, its count,
  a green check, no cancel; "Finished" in its tooltip), and the entry goes on to its next unit. An instant craft makes the
  units that fit, and the rest waits the same way.
- It moves into the bag once there is room, as many units as fit: the bag's own crafts from anywhere, a station's or shop's
  while its window is open there (with `CraftDeliveryMode` Immediate, from anywhere). A line on the player's collect feed says
  "Some crafted items are still waiting -- make room." when a craft starts waiting (once for a whole batch) and each time a
  station's or shop's window opens with crafts still waiting there. It is never a notice, so crafting does not interrupt
  play.
- Waiting crafts are kept with the player's record (`GetCraftsWaiting`; None is the bag's own). The ingredients were spent
  at the press, so nothing is lost.
- An item's carry limit (`MaxUnitsPerContainer`) counts what the bag holds, not what is queued or waiting. Each CRAFT
  press is checked against the bag, so more can be ordered than the limit allows; the hand-over then moves in only what
  the limit leaves room for, and the rest keeps waiting where it was ordered. Nothing is lost.
- Off, a craft needs room for its whole batch when it is ordered, and a unit that finds no room stalls its entry
  (WaitingForRoom) until room frees.

**A craft ordered at a crafting station** or at a shop that crafts is handed over AT that station.

- By default (`CraftDeliveryMode` AtStation), one that finishes while the player is away waits there, with its icons over
  the station.
- It waits until they open its window again.
- See [Finished crafts wait at the station](crafting-stations.md#finished-crafts-wait-at-the-station).

## Stations

- `RequiredStationTag` on a recipe gates it to a station.
- The server reads the station tags it holds for the session, rather than trusting a client-supplied list.

**Two things can supply them:**

- An open vendor session (`USnapVendorStockDataAsset::StationTags`).
- An open crafting station (`ESnapInteractionVerb::OpenCraftingStation`).

`USnapCraftingComponent::GetActiveStationTags` unions them and is the only place either is derived. See
[Crafting stations](crafting-stations.md).

**What each craft window lists** (see [What a station lists](crafting-stations.md#what-a-station-lists)):

- A station's craft window lists only that station's own recipes.
- A crafting shop that is no station lists the recipes that need no station.
- The bag's CRAFT window lists them all, saying "Made at: ..." on a recipe whose station is not active there.

**What each craft window's queue shows:** the crafts ordered there, then its finished crafts waiting for room (see
[Timed queues](#timed-queues)).

## The UI

**Crafting appears in two places,** driven by the same panel class:

- A CRAFT tab on the shop screen.
- A floating craft window on the inventory screen.
  - The window is hidden by default and remembers its position.

`USnapInventoryCraftPanelWidget` carries:

- The recipe list.
- A filter bar with collapsible categories.
- A detail pane.
- A quantity slider.
- The queue: the crafts ordered at this host, then its finished crafts waiting for room.

Recipe rows are styled by output rarity.

**The quantity** stays on screen at a ceiling of 1, shown but disabled, so the panel keeps one height.

- Over the stepper the mouse wheel changes the amount (Shift: ten). Over the recipe list the wheel scrolls the list.
- The CRAFT button reads "Craft x5" for a batch of five. `CraftQueueButtonFormat` can word a queued batch differently.

**The filter bar** sits below the recipe list.

- Category headers fold with the mouse only. They aren't gamepad focus stops (a header draws no focus state), so on a
  pad the filter bar narrows the list by the same categories.

**The craft time:** The detail pane shows a recipe's craft time as a duration ("Craft time: 1m 30s").

- It shows "Instant" for a recipe that crafts at once, including a timed recipe while `bEnableTimedCrafting` is off.
- Override `USnapInventoryCraftPanelWidget::ResolveCraftTimeLabel` to print words ("Short", "Long") instead.

**The DISMANTLE button** works on copies in the bag's grid only, not pouched or equipped ones.

- A press spends one whole batch (the recipe's `OutputCount`) from one stack, so two stacks of 2 hold no batch of 3.
- Its label counts what presses could salvage: "Dismantle (3)", or "Dismantle x3 (have 6)" for a recipe that makes more
  than one.
- It is disabled when the server would refuse: a station, a lock, no room for the salvage.
  - When the bag holds a copy, the status line says why (unless it is already saying why the craft is refused).
- A press asks for confirmation first, naming what comes back.

⚠ The recipe list is inside a height-bounded scroll box.

- An unbounded `UScrollBox` reports its content's full desired size and simply grows instead of scrolling.
- That pushes everything below it off screen.

**The panel's layout:**

- Its widths, gutters and padding are its `CraftLayout` class default, edited on a Widget Blueprint subclass such as
  `WBP_ExampleCraftPanel`.
- Its band heights are fixed.
- See [UI and theming](ui-and-theming.md#layout-on-the-widget-class), including the shop screen column that must match
  `DetailColumnWidth`.

## Mixing items

**Drop one item onto another** and, when the project's **mix rule set** names the pair, both are spent and a third is made.

- `USnapCraftingSettings::MixRuleSet` names the set and ships **empty**.
- A project that names none mixes nothing, and every drop means what it always did.

```
USnapMixRuleSet          Rules: FSnapMixRule[]
FSnapMixRule             ItemA, CountA, ItemB, CountB, Result, ResultCount, ResultRarityMode, ResultRarity
```

- **Either order.** A row taking A and B matches A dropped on B and B dropped on A.
  - Each input spends its count from the ONE stack in the gesture — the dragged stack and the stack it was dropped on —
    never from elsewhere in the bag.
  - A stack holding fewer is refused (`MissingIngredients`).
- **The result** lands in the bag, at the tier crafting would give it.
  - `Fixed` (empty = the lowest tier) or `Random`.
  - A definition that pins a tier keeps it.
- **What a release over an item means, in order:**
  1. A storage item under it **stores**.
  2. A mix row **mixes** (`ESnapInventoryDropAction::Mix`).
  3. A repair kit **repairs**.
  4. Anything else merges, stores a portion or swaps.

  `USnapInventorySlotWidget::ResolveItemOnItemPlan` states it once for the bag list, the bag grid, a pouch window and the
  gamepad carry. Its plan sets `FSnapInventoryDropPlan::bItemOnItem`, so a grid outlines the target item, not a landing.
- **Refused, never swapped.** A row that matches but cannot run says why:
  - An input not held by this container (`NotOwned` — a worn item never mixes).
  - A storage item still holding something (`StorageNotEmpty`).
  - An input the player locked (`ItemLocked`).
  - Too few units (`MissingIngredients`).
  - Or no room for the result (`InventoryFull`, `UnitCapReached`, `WrongContainer`).
- **The server decides.**
  - `USnapCraftingComponent::DescribeMix` is the verdict both sides ask.
  - `MixItems` / `Server_MixItems` re-find the row in the server's own loaded set before spending anything.
  - They report a refusal back to the player.
- ⚠ **Room is checked before anything is spent**, on the bag as it is — crafting's rule.
  - Spending can only add room, so a mix never destroys its inputs for want of space.
  - The cost: a full bag cannot mix two single items, even though the mix would free a square.
- **Validation:**
  - `IsDataValid` refuses a row with an empty input or result, a count below one, the same item twice, a result that is
    one of its inputs, or a pair an earlier row already takes.
  - It warns when an input is a storage item. A drop onto a storage item always stores, so that row is reached only with
    the pouch as the dragged item.
- ⚠ A set named only in config has no referencer for the cooker.
  - Cook its folder (`DirectoriesToAlwaysCook`) or reference it from content, as with `UITheme`.

## See also

- [Items and fragments](items-and-fragments.md) — `RecipeUnlock`, `Upgradeable`
- [Crafting stations](crafting-stations.md) — a forge that is not a shopkeeper
- [Vendor](vendor.md) — the shop's CRAFT tab
- [Appendix: settings](appendix-settings.md) — the **Crafting** category
