# Restrictions and marks

Whether an item may leave a player's keeping, who decided, and how the game says so.

Two mechanisms share one answer:

- A **restriction** is authored by a designer on the item definition, and the player can never change it.
- A **mark** is set by the player on one instance, and they can clear it whenever they like.
- Both are read by a single predicate, so no door can obey one and forget the other.

**On this page:**

- [The one oracle](#the-one-oracle)
- [Restrictions — what a designer authors](#restrictions--what-a-designer-authors)
- [Marks — what a player sets](#marks--what-a-player-sets)
- [In the game](#in-the-game)
- [Bulk verbs](#bulk-verbs)
- [See also](#see-also)

## The one oracle

```cpp
ESnapRefusal USnapInventoryComponent::DescribeRemoval(const USnapItemInstance* Instance,
                                                         ESnapItemRemovalKind Kind);
bool         USnapInventoryComponent::CanRemove(const USnapItemInstance* Instance,
                                                   ESnapItemRemovalKind Kind);
```

- `None` means the removal is permitted.
- The oracle is **static**: it needs no session state, so the equipment component can ask it about a worn item that no
  inventory holds.

`ESnapItemRemovalKind` names the door: `Drop`, `Sell`, `Dismantle`, `Trade`, `DeathDrop`.

- A death drop is a separate value rather than a convenience.
- "I can throw this away but I do not lose it when I die" is how a starter weapon is normally authored, and one shared
  value could not express it.

### The seven doors

Every way an item can leave asks the oracle first.

| Door | Function |
|---|---|
| A deliberate drop, from the grid or a pouch | `USnapInventoryComponent::DropItemStacksToWorldAs` |
| A worn item dropped straight out of its slot | `USnapEquipmentComponent::DropEquippedToWorld` |
| A sale | `USnapVendorClientComponent::CalculateSellTotalInternal` |
| A dismantle | `USnapCraftingComponent::EvaluateDismantle` |
| A trade offer | `USnapInventoryComponent::DescribeTradeOfferCandidate` |
| The corpse adopting what you were wearing | `USnapDeathDropComponent::SpawnLootableBodyAtDeathFor`, worn loop |
| The corpse adopting what you were carrying | `USnapDeathDropComponent::SpawnLootableBodyAtDeathFor`, carried loop |

The death path is the one worth knowing about, and it is why the count is seven rather than six:

- One function carries two doors, because worn items and carried items are adopted by separate loops, and a rule added
  to one of them is half a rule.
- `RunDeathDrop` — which the death tag, `NotifyOwnerDied` and `TriggerDeathDrop` all reach — spawns the lootable body
  **before** it scatters anything.
  - So the corpse adopts the items and the scatter never sees them.
- A rule enforced only in the scatter would be enforced only when nobody dies properly.

### What is deliberately not a door

`ConsumeItem` and spending an item as a crafting ingredient are **uses**, not disposals.

- A lock stops you *losing* an item, not *using* it — so a locked potion can still be drunk and a locked ore can still
  be spent by a recipe.
- The internal custody transfers (storing, equipping, committing a trade) are not doors either: gating those would
  refuse an item *moving* rather than an item *leaving*.

## Restrictions — what a designer authors

Add `USnapItemFragment_Restrictions` to an item definition.

| Field | Refusal when false |
|---|---|
| `bCanDrop` | `CannotDrop` |
| `bCanSell` | `CannotSell` |
| `bCanTrade` | `CannotTrade` |
| `bCanDismantle` | `CannotDismantle` |
| `bDropOnDeath` | `KeptOnDeath` |
| `BindPolicy` | `ItemBound`, once bound |

- Every field defaults to the permissive value, so an item carrying the fragment with nothing changed behaves exactly
  like an item without it.
  - That is what makes the fragment's **absence** mean "unrestricted" by construction rather than by a table somebody
    has to maintain.
- `BindPolicy` is `Never`, `OnPickup` or `OnEquip`.
  - A bound item cannot be **traded**; it can still be dropped, sold and dismantled.
  - Nothing clears a bind.

## Marks — what a player sets

```cpp
enum class ESnapItemMark : uint8 { Locked, Favorite, Junk };

bool USnapInventoryComponent::Server_SetInstanceMark(USnapItemInstance* Instance,
                                                        ESnapItemMark Mark, bool bSet);
bool USnapInventoryComponent::IsInstanceMarked(const USnapItemInstance* Instance,
                                                  ESnapItemMark Mark);
```

| Mark | Means |
|---|---|
| `Locked` | Refuses **every** door, including a deliberate single drop or sell |
| `Favorite` | Restricts nothing; the bulk verbs pass it over |
| `Junk` | Restricts nothing; it is what `SellAllJunk` selects on |

- `Locked` and `Junk` are mutually exclusive — setting either clears the other.
- `Favorite` is independent of both, so an item can be Favorite and Junk at once.
- Marks are stored as a typed **memory** on the instance (`FSnapItemMarkMemory`). That is why they:
  - Replicate.
  - Survive being dropped into the world and picked back up.
  - Persist through a save with no extra plumbing.

### Switching marks off

The `bEnableItemMarks` setting (**Project Settings > Snap Inventory System - Items > Item Marks**, on by default) switches the
whole feature. Off, it is inert end to end, not merely hidden:

- `GetInstanceMarks` — the one oracle every reader asks — answers all-clear:
  - No door refuses a locked item.
  - Sell All Junk finds nothing.
  - A bulk verb passes no favorite over.
  - No badge or tooltip line shows a mark.
- The item menu offers no Lock, Favorite or Junk row.
- `Server_SetInstanceMark` writes nothing and returns false.
- `Server_SetItemMarks` changes nothing and reports `MarksTurnedOff` ("Item marks are turned off.") once — see
  [Refusals](refusals.md).
- The shop's **SELL JUNK** button is hidden.
  - With nothing markable its press could only say "mark something Junk first" from a menu that offers no such row.
  - It is decided each time the shop opens.

The marks themselves stay in each item's memory, and a save writes them as they are.

- So switching the feature back on brings every mark back.
- It is read on every call, so it can change during play: the demo's **ITEM MARKS** pad does (see the
  [demo map](demo-map.md#the-features-wing-switch-a-plugin-feature-for-this-session)).

### From the client

`Server_SetInstanceMark` is authority-only. A client marks through the RPC, which re-validates custody per instance:

```cpp
void USnapInventoryComponent::Server_SetItemMarks(const TArray<USnapItemInstance*>& Instances,
                                                     ESnapItemMark Mark, bool bSet);
```

It carries an absolute `bSet`, never "toggle".

- A toggle evaluated per item would make a mixed selection alternate rather than converge.
- Press *Lock* on three items of which one is already locked and that one would come back unlocked, with the row having
  promised to lock all three.

`USnapInventoryComponent::IsInstanceInOwnerCustody` is the custody test, and the same one the context menu asks
before offering a row.

- The item must be in this component's grid, inside one of its pouches at any depth, or worn on the sibling equipment
  component.

## In the game

### The context menu

Right-click offers **Lock / Unlock**, **Mark Favorite / Clear Favorite** and **Mark Junk / Clear Junk**.

- Each row's label states the direction pressing it will go, resolved from the selection, so it cannot promise one
  outcome and perform another.
- A mixed selection resolves toward the **protective** state: it locks rather than unlocks, favorites rather than
  clears, and *clears* junk rather than setting it.

A row blocked by a lock or a restriction is drawn **grayed with its reason**, not hidden.

- A row for something the item could never do — *Sell* on an item with no price, *Dismantle* on something with no recipe
  — is still omitted.
- That distinction is the point: a vanished row tells a player nothing, while a grayed one tells them they can fix it
  themselves.

### On the slot

Three optional widget names on `USnapInventorySlotWidget`: `LockGlyphWidgetName`, `FavoriteGlyphWidgetName` and
`JunkGlyphWidgetName`.

- Each defaults to `None`, so a slot WBP that names none of them is unchanged.
- Three names rather than one because Favorite is independent of the other two, and a single swapping glyph could only
  ever show one mark.
- The glyphs repaint on the ordinary panel refresh, so a mark set from the menu appears immediately without reopening
  the screen.

## Bulk verbs

The two sweeps are on the player's `USnapVendorClientComponent`, a component beside the main bag; the dismantle door
stays on the bag:

```cpp
FSnapInventoryBulkActionReport USnapVendorClientComponent::SellAllJunk();
FSnapInventoryBulkActionReport USnapVendorClientComponent::SalvageByRarity(const FGameplayTagContainer& RarityTags,
                                                                          const USnapCraftingRecipeSet* RecipeSet);
void USnapInventoryComponent::Server_DismantleItems(const TArray<USnapItemInstance*>& Instances,
                                                       const USnapCraftingRecipeSet* RecipeSet,
                                                       FGameplayTagContainer StationTags);
```

Each chooses a set and hands it to a door that already asks the oracle, so a locked item is held back by construction
rather than by a second copy of the rule.

`SalvageByRarity` takes an explicit **set** of tiers, never a threshold.

- A "this tier and below" form would have to order the tiers, and an unmigrated rarity table has no order to read.
- An empty container matches nothing and is a no-op — never "all".

### In the game

The shop screen builds a **SELL JUNK** button at the left end of the bag's currency line, apart from the money at the
right.

- That is the line the base screen deliberately leaves empty there, because a shop must never offer *drop everything on
  the floor*.
- The verb needs a vendor session, so that is the only place it can live.
- A WBP that names its own `SellJunkButton` gets that one bound instead, unrestyled.
- Either is shown only while marks are on.

There is no confirmation, and that is a decision:

- Every subject was marked Junk by hand.
- Locked and favorited items are held back by the verb itself.
- The outcome is currency rather than loss.
- So a modal would contradict the one thing the feature is for.
- What happens is reported afterwards as a notice, with the skip reasons stated apart.

`SalvageByRarity` and `Server_DismantleItems` ship without a button of their own.

- The dismantle verb is reached by the context menu's own **Dismantle** row on a multi-selection.
- The rarity sweep is API-only, for a project to bind where it makes sense.

### ⚠ They are authority-only, and a client must route

`SellAllJunk` and `SalvageByRarity` both refuse on a non-authority owner and return an **empty** report.

- That is the trap worth knowing about, because an empty report does not read as an error: it reads as
  *"the player had nothing marked"*.
- The shipped SELL JUNK button called `SellAllJunk()` directly for one build.
  - On every client that was not the listen-server host, it changed nothing while telling the player
    *"nothing in your bag is marked as junk"* immediately after they marked it.
  - It passed every test, because ordinary PIE runs the UI on the authority.

So a UI driving one of these needs both halves:

```cpp
if (Inventory->GetOwner() && Inventory->GetOwner()->HasAuthority())
{
    ReportJunkSweep(USnapVendorClientComponent::SellAllJunk(*Inventory));   // run it, and say what happened
}
else
{
    Inventory->Server_SellAllJunk();             // ...which sends Client_ReportJunkSweep back
}
```

Routing the mutation without routing the *report* is the same defect wearing a different hat — a button that works
silently.

- `USnapInventoryVendorScreenWidget::ReportJunkSweep` is the one place the notice is built, and both routes reach it.
  - It shows as the white notice line (`NoticeStyle` **Prompt**, the default), held a second longer for each line of the
    report after the first, or as the OK window (**Window**).

⚠ Report the authority branch **inline** rather than through the `Client_` RPC.

- A `Client_` RPC invoked on a listen-server host executes locally as well, so routing both would raise the notice twice
  on the host.

⚠ `SalvageByRarity` has a `Server_SalvageByRarity` but **no report channel**.

- A project binding it to a client-side button has to add one, or say nothing at all.
- `Server_DismantleItems` returns nothing by design; the context menu reports dismantles through the ordinary refusal
  path instead.

### Favorite is a filter, not a refusal

A Favorite item is passed over by the bulk verbs **and sells normally through a deliberate single sell**. That pair is
the whole rule.

- Favorite deliberately does *not* appear in `DescribeRemoval`.
- The oracle is asked by single deliberate actions too, so a Favorite term in it would refuse an ordinary right-click
  *Drop* on a favorited sword, which is not what a favorite means.
- An item marked both Favorite and Junk is **skipped** — "I flagged this to keep" beats "I flagged this as junk".

### The report

`FSnapInventoryBulkActionReport` says what happened, and says the reasons apart:

| Field | Means |
|---|---|
| `Considered` | Matched the selection rule, before any filtering |
| `Acted` | Actually sold or salvaged |
| `SkippedFavorite` | Passed over because the player favorited it |
| `SkippedProtected` | Refused by a lock or an authored restriction |
| `SkippedIneligible` | No price, no recipe, a pouch with something in it |
| `CurrencyGained` | Credited, read from the ledger rather than from the quote |

The four buckets sum to `Considered`.

- "3 sold, 1 kept" is not enough when one was kept because the player chose to keep it and another because the designer
  made it unsellable.
- The first is two clicks to undo and the second is not.

## See also

- [Refusals](refusals.md) — the shared vocabulary these verdicts are spoken in
- [Inventory and containers](inventory-and-containers.md) — the grid, pouches and custody
- [Crafting](crafting.md) — dismantle, recipes and yields
- [Vendor](vendor.md) — sessions, pricing and the sell bundle
- [Trade](trade.md) — why a trade offer is all-or-nothing where a sale skips
