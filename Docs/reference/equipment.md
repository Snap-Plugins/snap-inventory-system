# Equipment

`USnapEquipmentComponent` holds worn items in tag-named slots and grants whatever they carry. This page covers slots,
matching an item to a slot, slots a worn item blocks, equipping, what equipping grants, how a worn item looks,
requirements and refusals.

**On this page:**

- [Slots](#slots)
- [Matching an item to a slot](#matching-an-item-to-a-slot)
- [Slots a worn item takes out of use](#slots-a-worn-item-takes-out-of-use)
- [Equipping](#equipping)
- [What equipping grants](#what-equipping-grants)
- [A weapon that changes what a click does](#a-weapon-that-changes-what-a-click-does)
- [How a worn item looks](#how-a-worn-item-looks)
- [Ownership](#ownership)
- [One instance, one slot](#one-instance-one-slot)
- [Durability and wear](#durability-and-wear)
- [Requirements](#requirements)
- [Refusals](#refusals)
- [Settings](#settings)

## Slots

There are two ways to say which equipment slots your project has, and **only one of them is in force at a time**.

### The slots on the component

By default, slots are `FGameplayTag`s listed in `AvailableSlots` on `USnapEquipmentComponent`. The plugin ships ten,
from its own `Config/Game.ini`:

```
Equipment.Slot.MainHand     Equipment.Slot.Amulet
Equipment.Slot.Chest        Equipment.Slot.Ring.1
Equipment.Slot.Gloves       Equipment.Slot.Ring.2
Equipment.Slot.Greaves      Equipment.Slot.Consumable.1
Equipment.Slot.Boots        Equipment.Slot.Consumable.2
```

**Slot names:**

- The two ring slots read **Left Ring** and **Right Ring** wherever a slot is named.
  - Those are the plugin's default names for its own `Ring.1` and `Ring.2` (keyed, so they translate).
  - A slot set's row still wins.
- Every other numbered slot takes the name split out of its tag (`Consumable.1` reads "Consumable 1").
- `Ring.1` is the left hand's: the first ring a player puts on goes there.

⚠ `AvailableSlots` is the **one** key the plugin's own `Config/Game.ini` assigns, and that file is a **dynamic layer
above** your project's hierarchy, not a low-priority default.

- So a `+AvailableSlots=` line in your `DefaultGame.ini` **adds** to the ten (yours first).
- Nothing you write there removes one.

**To choose the slots outright:**

- Use a slot set asset (below), which replaces the list entirely.
- Or use the config route: `<YourProject>/Config/PluginOverrideGame.ini`, which sits above the plugin's layer. Write
  `!AvailableSlots=ClearArray` followed by the slots you want.

### Or a slot set asset

Point **Project Settings → Snap Inventory System - Equipment → Equipment → Equipment Slot Set** at a `USnapEquipmentSlotSet`, and
**the asset becomes the whole truth**:

- Its rows decide which slots exist and in what order.
- Each row also carries that slot's display name, the ghost an empty slot shows, where a worn visual attaches, and how
  fast items wear there.

**The example set:** `DA_ExampleEquipmentSlotSet` is the ten shipped slots written out that way, without the two
consumable slots, plus an **Off Hand** row (below) and a **Head** row.

- It ships unreferenced, so nothing changes until you point at it.
- The demo map points at it for each play session.

The setting ships **empty**, and those are the only two states:

| Equipment Slot Set | What decides |
|---|---|
| Empty | `AvailableSlots` on the component |
| Set, and the asset loads | The asset's rows — **an empty list included** |
| Set, and the asset will not load | `AvailableSlots`, plus one warning naming the path |

⚠ **THERE IS NO PER-FIELD MERGE.**

- A set in force does not fill gaps in the component's list.
- Its wear values replace `DamageWearMultiplierBySlot` outright rather than topping it up.
- A merge would make "why does this slot exist?" unanswerable without reading both sources and knowing which fields each
  one contributed.
- The one exception is *inside* a row: a row whose display name is left blank falls back to the name split out of its
  tag.
  - So adopting a set does not mean naming every slot before any of them read correctly.

⚠ An **empty** set in force really does mean no equipment slots. That is honored rather than treated as "unset".

- `IsDataValid` is what catches an accidentally-blank asset — on save, on Validate Assets and at cook.
- It also catches a duplicated tag, a row with no tag, and a row whose tag is a **parent** of another listed row.
  - Membership is a tag match, so the parent would match the child.

⭐ **ROW ORDER IS THE AUTO-EQUIP TIE-BREAK.**

- When an item fits more than one free slot, the first slot in the effective order wins.
- It is not a drawing order — the panels keep their own authored columns.

**Where a pad goes from each slot** is the equipment panel's, not the slot set's.

- `USnapEquipmentPanel::SlotNavigation`, in the panel's class defaults beside `SlotColumns`, maps each slot's Up / Down / Left /
  Right to another slot, the inventory grid, a hotbar bar or Stop.
- Automatic leaves it to the plugin's own choice.
- See [Input](input.md#the-bag-screen-on-a-pad).

### Locked slots

A row can start **locked**.

- A locked slot refuses anything put **into** it — equipping, transferring, splitting or merging in, and either direction
  of a swap.
- It refuses nothing else: unequip, drop, consume, break and every transfer **out** behave exactly as on an open slot.

⭐ **Re-locking never takes an item off.**

- A slot that is locked while something is worn keeps wearing it, and still gives it up on request.
- That is the only rule that cannot cost a player an item they already had.
- It means a worn item in a locked slot is a legal state a save can hold.

Two things open a locked slot, and they are independent:

| Opener | How it opens | Saved? |
|---|---|---|
| `UnlockSlot(Slot)` | Explicitly, and permanently | **Yes**, with the player |
| The row's `Unlock Tag` | While the owner has the tag | No — it re-locks when the tag goes |

There is no `RelockSlot`. An unlock the player earned is not taken back; a slot meant to come and go should use a tag
instead.

⚠ **GRANT AN UNLOCK TAG FROM A GAMEPLAY EFFECT, NOT AS A LOOSE TAG.**

- A loose tag does not replicate, so the slot would open on the server and stay shut on every client.
- The client would keep refusing a drag the server would have accepted, with nothing logged on either side.

**Reading the lock:**

- Ask `IsSlotUnlocked(Slot)` for the answer, on a client as readily as on the server.
- `OnSlotLockChanged` fires when a slot becomes admissible or stops being, so UI does not have to poll.
- Name a widget in `LockedGlyphWidgetName` on your slot widget, and it is shown exactly while that slot is locked. An
  Image is dimmed by the palette's `UnboughtSquareTint`, as the bag's locked squares are.

**In the example:**

- The example slot square `WBP_ExampleEquipmentSlot` names one, `LockedGlyph`: a padlock over a darkened square
  (`T_Example_SlotLocked`, in `/UI/Textures`).
- A locked, empty slot's tooltip reads its name and then "Locked" — followed by the row's `UnlockHint` when it has one
  ("Locked -- Complete a quest to unlock it.", the example set's Right Ring).
  - `USnapInventoryTooltipWidget::SetFromLockedSlot` draws it.
- A drop onto it shows the drag picture's red X and the notice "This slot is locked." (`SlotLocked`).

⚠ With **no slot set in force** there are no rows, so nothing is locked — `AvailableSlots` is a bare tag list with nowhere
to carry the flag.

⭐ **Loading a save bypasses the lock**, deliberately.

- A worn item in a locked slot is legal.
- A tag-unlocked slot's effect may not be back yet when the save is read.
- Refusing there would discard the item rather than restore it.
- Explicit unlocks are applied before the first worn line.

⚠ **A slot set is named by config, so the cooker cannot see it.**

- Like the container registry and the theme, a path in an `.ini` is not a reference.
- So add its folder to `DirectoriesToAlwaysCook` (or reference the asset from something that is cooked), or it will be
  missing from a packaged build.
- The editor's **Tools → Snap Inventory Setup Check** reports exactly this.

Everything that needs the list calls `USnapEquipmentComponent::GetEffectiveSlots()`, so the two sources can never be
half-applied — one caller seeing a slot another does not.

## Matching an item to a slot

- `Equippable`'s `SlotTag` names a slot **type**, and matching is by tag hierarchy.
  - An item whose `SlotTag` is `Equipment.Slot.Ring` fits both `Ring.1` and `Ring.2`.
- `AcceptedSlots` names **further** slot types the same item fits, for the case one tag cannot express.
  - A dagger that goes in either hand, where the two slots share no useful parent.
  - It is matched in exactly `SlotTag`'s direction, so an entry naming a category admits every leaf beneath it.
- `SlotTag` is still required, and `AcceptedSlots` does not replace it.
  - An item that declares no slot type is worn nowhere, whatever the container holds.
  - Both are empty by default, so an item definition saved before they existed answers exactly as it did before.
- **Consumable slots:** `bAllowedInConsumableSlot` (on by default) lets an item into the slots listed in `ConsumeSlots`
  when its `SlotTag` fits them. Clear it to keep the item out of those slots.
  - A consumable slot is one named in `ConsumeSlots` (exact tag match), not a tag family, so the flag does nothing while
    `ConsumeSlots` is empty.
  - It is checked in `USnapEquipmentComponent::DoesItemFitSlotType`, so the item menu, a drag, auto-equip and
    `EquipItem` agree.

**The shipped case:** The example iron sword and iron axe have `SlotTag` Main Hand and `AcceptedSlots` Off Hand.

- With one of them in the Main Hand, the other picked up equips itself into the Off Hand.
- It is drawn in the left fist, through its `SlotAttachOverrides` row ([Where a visual hangs](#where-a-visual-hangs)).

```
CanEquipItemFromInventory(Instance)          static, cheap
CanEquipInstanceInSlot(Instance, SlotTag)
USnapEquipmentSlotWidget::IsCompatibleWithItem(Instance)    the slot widget's own pre-reject, for its slot
```

⚠ `EquipFromInventory` needs the **leaf** slot tag. Passing `Equipment.Slot.Consumable` when the component's slots are
`Consumable.1` and `Consumable.2` returns false with nothing logged.

⚠ Widening the fit does not order it.

- Where several free slots now match, auto-equip takes the **first in the effective slot order** — which is config you
  can see and change.
- There is deliberately no priority field on the item.
  - An item-side ordering and a project-side ordering would disagree, and a player can see neither.

## Slots a worn item takes out of use

- `BlocksSlots` names slots the item **spans** while it is worn — a two-handed weapon and the off hand.
- It is matched in the same direction again, so one entry naming a category covers every leaf beneath it.

```
IsSlotBlocked(SlotTag)                       is anything spanning this slot right now?
GetSlotBlocker(SlotTag)                      which worn item, or null
DoesItemSpanSlot(Instance, SlotTag)          static -- would WEARING this take that slot out of use?
```

- The first two ask about the wearer and answer on a client, so the UI pre-reject and the server's doors read one answer.
- The third asks about the **item** and reads nothing about any wearer.
  - So it answers for an item that is nowhere near the actor — one in a vendor's stock, one under the cursor.

These rules decide what happens:

- **An occupant never blocks its own slot.**
  - Without that, an item listing the slot it is worn in would make itself impossible to replace.
  - The only escape would be an unequip nothing asks for.
- **Equipping it displaces** whatever occupies the slots it spans, into the bag.
  - If the bag cannot take **all** of it, the whole equip is refused and nothing moves — `NoRoomForDisplaced`.
- **A player's own gesture into a spanned slot takes the spanning item off.**
  - A shield dragged onto the off hand — or double-clicked, Shift+clicked, or equipped from the item menu — while a
    two-handed sword is worn goes on.
  - The sword goes to the bag, into the square the shield left.
  - It is refused with one notice, `NoRoomForDisplaced`, only when the bag cannot take the sword.
  - The `bEquipIntoBlockedSlotUnequipsBlocker` setting (Project Settings, Equipment; on by default) switches it off. The
    gesture is then refused with `SlotBlocked` as before.
- **Nothing automatic does that.**
  - Auto-equip on pickup, a rarity upgrade and Take All all treat a spanned slot as blocked.
  - Loading a save is not gated by `BlocksSlots`, nor by a locked slot: every worn line comes back worn, even two items
    that now span each other, because refusing would lose an item.
  - A worn item only comes off because the player asked for that slot.
  - The gesture doors are `EquipFromInventoryForGesture` / `EquipFromStorageForGesture`, and their oracle is
    `DescribeGestureEquip`.
  - `EquipItem`, `EquipFromInventory` and `DescribeEquip` keep refusing.

A **double-click** on an equippable item with no use — a sword, a ring, a piece of armor, in the bag or a pouch — equips
it exactly as Shift+click and the menu's Equip do:

- Into a free slot it fits.
- Or in place of the piece worn there, which takes the new one's square (`bEquipOnDoubleClick`, on by default).
- It is decided by the item: anything with a use (a potion, food) is used by a double-click and never equipped by it,
  even when its use is refused.

**While the sword is worn:**

- The off hand's square draws the sword's picture **dimmed** (`USnapEquipmentSlotWidget::BlockedByWornItemTint`).
- Its tooltip is the sword's own, which carries the item's `BlocksSlotsTooltipText` line.
  - That is "Two-handed: uses both hands" by default, on every tooltip of an item whose `BlocksSlots` is not empty.
- An item that takes some other slot out of use should say what it takes.

⚠ **Stack portions never displace.**

- `TransferInventoryToEquipped` and `SplitEquippedStack` refuse rather than turn anything out.
  - That is because neither has the all-or-nothing machinery the whole-item equip path has.
- In practice that is unreachable without a stackable item that also spans something.

⚠ **A swap is judged on the state it would create**, not the one it starts in.

- Two of the ways it can be illegal are invisible to `IsSlotBlocked`.
- An item that spans its own slot is legal exactly where it is and illegal one slot over.
- And an item that spans its neighbor's slot makes that slot read blocked right now, yet the exchange is legal because the
  blocker is the item leaving.

**The example set shows the shape:**

- `DA_ExampleEquipmentSlotSet` has an **Off Hand** row beside Main Hand.
- `ID_Example_Shield` goes in it, and `ID_Example_Greatsword` spans it.
- Wear the greatsword and the shield goes to the bag; put the shield on yourself while the greatsword is on and the
  greatsword goes to the bag.
  - That is refused only when the bag cannot take the greatsword, or with `bEquipIntoBlockedSlotUnequipsBlocker` off.
- Both example panels draw the off hand next to the main hand.
- The shipped `AvailableSlots` default is untouched — an off hand exists only for a project that points at the example set
  or names the slot itself.

## Equipping

```
EquipFromInventory(Instance, SlotTag)
UnequipToInventory(SlotTag, TargetSlotIndex)      pass -1 to let it find a slot
SwapEquipmentSlots(SlotA, SlotB)
GetEquippedItem(SlotTag)
```

- Note the asymmetry with the inventory component: it is `GetEquippedItem` here, not `GetItemInSlot`.
- `OnEquipmentChanged` fires on every equip, replace and unequip.
- **A drag of several selected items onto a worn slot is refused**, as on a hotbar square: a slot holds one item.
  - The slot shows the red X while you hover, and nothing changes on release.
  - A drag out of a loot window is the exception. It takes everything into the bag, then equips the item you pressed,
    if it fits the slot.

### What a pickup equips by itself

An equippable item collected through `USnapCollectionLibrary::GrantItemToActor` — what a pickup calls when it is
taken — can be worn at once.

Two switches decide, one per branch (**Project Settings → Snap Inventory System - Inventory → Pickups** and **Snap Inventory System - World → Pickups**, both on by default):

| Setting | Applies when | Off |
|---|---|---|
| `bAutoEquipOnPickup` | A slot the item fits is free | The item stays in the bag |
| `bSwapToBetterRarityOnPickup` | Every slot the item fits is taken, and one of them holds the SAME item at a strictly lower rarity | The item stays in the bag and the worn one stays worn |

- **The swap is for the same item only.** A better tier of another item (a Rare Iron Sword walked over while a Greatsword
  is worn) never takes off what you wear.
- It is still considered only when no slot is free, so either switch can be tried without the other.
- **A full bag changes nothing.** An item a switch keeps off the body then stays on the ground, and the pickup's prompt
  answers the same way (`USnapCollectionLibrary::GetGrantRefusalForActor`).
- Both are read on every collect.
- Neither is the vendor's paid upgrade (`bEnableEquipmentUpgrades`).

## What equipping grants

`USnapItemFragment_GrantedAbilitySet` lists what an item hands to the owning ability system, split by **when** it happens
rather than by what kind of thing it is. It has six arrays, with three lifetimes:

| Array | Lifetime |
|---|---|
| `GrantedEffectClasses` | Applied on equip, removed **by handle** on unequip |
| `GrantedAbilityClasses` | Granted on equip, cleared by handle on unequip |
| `ScaledEquipEffects` | Equip-time too, but applied through a **spec** so a `SetByCaller` magnitude can be assigned — `BaseMagnitude + MagnitudePerRarityRank * rank`, so the same item is stronger at a higher rarity. Each row names its effect in `ScaledEffectClass` |
| `ConsumeEffectClasses` | Applied **once** when a unit is spent, and their handles are discarded on purpose: a drunk potion's heal must outlive the potion |
| `ConsumeAbilities` | Activated once when a unit is spent. The engine owns the removal (`GiveAbilityAndActivateOnce` grants, activates and clears), so there is never a handle to own |
| `ContainerAbilities` | Granted while the item is **held in a container**, at a level equal to the entry's stack count — an ability scroll is never equipped. Each row names its ability in `ContainerAbilityClass` |

Every class the fragment names is a **soft** reference (`TSoftClassPtr`):

- That covers the four class arrays above, plus `ScaledEffectClass` in each `ScaledEquipEffects` row and
  `ContainerAbilityClass` in each `ContainerAbilities` row.
- The item's package names each class without importing it, so a project without `GameplayAbilities` loads the item with
  no warning about them.
- Each entry is loaded synchronously where it is used, so it still applies or grants inside that call:
  - On equip (and when a broken item worn in its slot is repaired).
  - When a unit is spent.
  - When a custom container grants a held item's ability.
  - In the tooltip, the stat validator and the hotbar.
- The class is no longer loaded along with the item definition, so its first use pays for loading it.
- An entry that does not resolve to a class of the right type is skipped with a `Warning` naming it at the equip, the
  spend or the container's grant.
  - The rest of the array still applies.
- An entry counts as declared when it names a class, loaded or not.
  - So a consume entry naming a missing asset is offered and, when spent, logs that `Warning` and applies nothing.

⚠ An equip-time effect must be `Has Duration` or `Infinite`.

- An `Instant` effect writes the attribute's *base* value and returns no handle.
  - So unequipping cannot remove it, and every re-apply adds another.
  - The component logs a warning naming the asset when one does.
- An effect that should fire once on use belongs in `ConsumeEffectClasses`.

⚠ `ScaledEquipEffects` is a separate array rather than another `GrantedEffectClasses` entry.

- That is because `ApplyGameplayEffectToSelf(CDO, Level)` cannot reach a `SetByCaller` modifier at all: it produces a live
  handle and a modifier of **zero**.
- Verify one of these by reading the attribute, never by counting effects.

⚠ **A rarity tier's `GrantedEffect` is applied that same way** — from its class default, at the tier's `EffectLevel`.

- So its modifiers need a magnitude of their own: a Scalable Float, or a curve keyed on the level so one effect serves
  every tier.
- The rarity table's validation warns about a `SetByCaller` modifier there, as an item set's validation does for a set
  tier.
- It also warns about an **Instant** effect.
  - Unequipping removes a tier's effect by the handle its apply returned, and an Instant effect returns none.
  - So it would change the attribute once per equip and never be undone.
- Make a tier's effect Has Duration or Infinite.

**Finding the ability system:**

- The component resolves an ability system component from its owner through
  `UAbilitySystemGlobals::GetAbilitySystemComponentFromActor`, which checks `IAbilitySystemInterface` and falls back to a
  component search.
- With no ASC (ability system component) anywhere, equipping still works and the grants are skipped with a warning — the
  plugin never casts to a game-specific type.

⚠ **Owner convention.** The component is designed to sit on the actor that owns the ability system component, commonly
the PlayerState.

- A PlayerState has no world transform.
- So a spawned equipment visual attaches to the **controlled pawn's** mesh instead, resolved through
  `APlayerState::GetPawn()`.

**Item sets:**

- An item may also belong to a **set**, and wearing several pieces of one grants further effects on top of whatever each
  piece grants by itself.
- Those are computed from the whole worn set rather than from any one piece — see [Item sets](item-sets.md).

### Stats an item declares itself

`USnapItemFragment_StatContributions` says what an item adds while worn, as plain data. Each row has:

- A stat's bare name (`Armor`, `MaxHealth`).
- A unit (`Flat` for "+5", `Percent` for "+15%").
- An `Amount`.
- An optional `AmountPerRarityRank`.

It works with or without `GameplayAbilities`, and the rows are **applied**, not decoration:

| Who | Reads the rows |
|---|---|
| The tooltip | As stat lines and in the swap-compare block — see [Tooltips and stats](tooltips-and-stats.md#stats-an-item-declares-itself) |
| `GetEquippedStatTotal(AttributeName, Unit)` | Summed over everything worn, in every build — what your own stat system reads |
| With `GameplayAbilities` | Each row is applied through the `GameplayEffect` the **Stat Effect Map** binds to it |

All three call one function (`ResolveContributions`), so none of them can disagree with the others.

`GetEquippedStatTotal` is computed from the replicated equipment list whenever it is asked — nothing is cached.

- So a client works it out from what it has received, the same way the server does.
- The network suite checks that on the owning client of a listen server
  (`SnapInventoryNet.ListenServerReplication.EquippedStatTotalReachesTheClient`).
  - After an equip and after an unequip, the client's total is the server's.
  - A listener that re-reads it when `OnEquippedStatsChanged` fires reads the new value.
- Flat and Percent are separate totals.
- A broken or spoiled worn item contributes nothing, exactly as it grants nothing else.

`OnEquippedStatsChanged` fires when the answer may have changed:

- Wherever `OnEquipmentChanged` fires (an equip, swap or unequip).
- On the server, when a worn item's rarity or durability changes.

It carries nothing — ask the total again — and it can fire more than once for one change.

⚠ It does **not** fire when a worn item spoils, although the item then drops out of the total. Read the total again after
spoilage if you cache it.

The total also counts the stat rows of every **item-set tier** the worn pieces reach (`FSnapItemSetTier::Stats` —
see [Item sets](item-sets.md#stat-rows)), from the same piece counts.

⚠ The total sums **stat rows only**. A stat moved by an effect in a `GrantedAbilitySet` is not in it; with
`GameplayAbilities`, read the attribute for the whole answer.

⭐ **The example items carry their base stats as stat rows:**

- The iron sword's Attack Power and Attack Cooldown.
- Every armor piece's Armor (with its rarity slope).
- The ring's and the Warrior's Signet's Max Health percent.

So the demo's numbers are the same with and without `GameplayAbilities`.

- Their `GrantedAbilitySet` keeps only what is an ability: the attacks and the shield's block.
- The ring's and the amulet's passives are demo passive fragments, which work without `GameplayAbilities` too.

**The Stat Effect Map** (`StatEffectMap` in **Project Settings → Snap Inventory System - Items → Equipment → Stats**, a
`USnapStatEffectMap`) binds a stat and a unit to an authored `GameplayEffect` and the `SetByCaller` tag its magnitude is
read from.

- Equipping applies the effect with the row's amount — as is for `Flat`, as the factor `1 + amount/100` for `Percent`.
- The plugin ships `DA_ExampleStatEffectMap` as the default.
  - It binds the demo attribute set's `Armor`, `AttackPower` and `AttackCooldown` (`Flat`) and `MaxHealth` (`Percent`) —
    every stat an example item's rows name.
- The network suite checks that what equipping applies through it reaches the owning client of a listen server as those
  authored effects, and comes off it again
  (`SnapInventoryNet.ListenServerGameplayAbilities.EquippedStatEffectsReachTheClient`).
  - It checks the effects' definitions before the attribute values.
  - That is because an attribute value replicates on its own and would read correctly even from an effect the client
    could not use.

⚠ A binding's effect must do **exactly** what the row says, and that is checked rather than trusted:

- One modifier, on the attribute the binding names.
- With a `SetByCaller` magnitude under its tag.
- An `Add` op for `Flat`, or a `Multiply` op for `Percent`.
- Not `Instant`, and not stacking.

**Checks and fallbacks:**

- Asset validation reports a binding that breaks any of these.
- Equipping refuses it, rather than apply something the tooltip does not describe.
- A row with no usable binding still counts toward the tooltip and the total.
  - It moves no attribute, and equipping says so once per stat.

Asset validation also **warns** when an item's stat row and one of its `GrantedAbilitySet` effects move the same
attribute. Both apply and both show, so they add together — usually a leftover.

See [GAS and Lyra](gas-and-lyra.md) for the ability side in detail.

## A weapon that changes what a click does

- The plugin decides what a worn item GRANTS; what a button does with it is your game's.
- The demo shows the two usual ways to let a picked-up weapon take over an attack, side by side and on the same items, so
  you can copy whichever fits.
- [The demo map](demo-map.md#attacks-the-shield-and-the-training-dummies) has the controls and the numbers.

| | With GameplayAbilities | Without |
|---|---|---|
| The item carries | Its attack ability in its Granted Ability Set (`USnapDemoAttack_OneHanded` on the iron sword) | A fragment of your own (`USnapDemoWeaponFragment`) with its style, damage and cooldown |
| Wearing it | Grants the ability; taking it off clears it, so the always-granted punch answers again | The fragment's `OnEquipped` tells your attack component what that hand holds; `OnUnequipped` takes it back |
| Which hand | The ability asks `USnapEquipmentComponent::FindSlotForInstance` for the slot its item (the spec's source object) is worn in | The hook is handed the slot |
| A click | Your input finds the ability for that hand and activates it | Your input asks the component, which decides on the server |
| The hit | Applies a GameplayEffect to each target's ability system | `UGameplayStatics::ApplyDamage`; each target's `TakeDamage` decides |

- Both hooks are `BlueprintNativeEvent`s every fragment has.
- They run on the server on every path that puts an item on or takes it off — a drag, a swap, a death drop.
- A fragment whose wearer has no attack component simply finds nothing.

## How a worn item looks

An item may spawn a visual actor, change how its wearer stands, both, or neither. The two are independent, and both live
on `USnapItemFragment_Equippable`.

### The three visual modes

`VisualMode` says how the item presents itself. It is on the item DEFINITION rather than the instance: it is what the item
*is*, not per-copy state.

| `ESnapEquipmentVisualMode` | What it does |
|---|---|
| `Attached` | Shown at its socket. **The default**, and exactly what every earlier version did. |
| `Hidden` | Worn, and not shown at all. No visual actor is spawned. |
| `Holstered` | Shown at a secondary *stowed* socket — the sword-on-the-back case. |

`Attached` is the zero value, so an item definition authored before this field existed loads as `Attached` and nothing
about it changes.

`Hidden` skips the **spawn**, not an attach.

- Nothing is created to hide, so it costs nothing and there is no actor for other code to stumble over.
- The item is equipped in every other respect: it holds its slot, grants its abilities and effects, wears out, saves and
  loads.

### Where a visual hangs

Four fields decide it — a primary pair and a stowed pair, each on the item and on its slot row:

| | Item (`USnapItemFragment_Equippable`) | Slot (`FSnapEquipmentSlotDefinition`) |
|---|---|---|
| Primary | `AttachSocketName`, `AttachOffset` | `AttachSocketName`, `AttachOffset` |
| Stowed | `HolsterSocketName`, `HolsterOffset` | `HolsterSocketName`, `HolsterOffset` |

**The item wins, the slot is the default, and neither is a merge.**

- An item that names a socket knows how it is held, and says so on every rig.
- A slot row says where that *place on the body* is — the right answer for every item worn there with no opinion of its
  own.
- Whichever side supplies the socket supplies the offset with it.
  - So an item can never end up at its own socket wearing the row's nudge.

⭐ **An item that fits several slots can hang somewhere different in each.**

- `SlotAttachOverrides` on the item is a list of rows — a slot, a socket, an offset.
- While the item is worn in a slot a row matches (an exact row first, then a category row such as `Equipment.Slot.Ring`),
  that row stands in for the item's own primary pair.
  - Everything above still applies, the slot row's default included when the row names no socket.
- The stowed pair is not affected.
- The example iron sword carries one: in the Off Hand it hangs from `hand_l`, at the exact mirror of its `hand_r` offset in
  the mannequin's reference pose.
- The rows live on the item rather than the slot row because the offset is the item's: where a grip sits depends on the
  mesh.

**Socket names:**

- A socket name is matched with `DoesSocketExist`, which answers for a **bone** as well as a socket.
  - So `hand_r` works on a rig that declares no explicit weapon sockets, which is most of them.
- A name that does not resolve is not an error:
  - The visual attaches at the mesh origin, and the offset is applied from there.
  - One Verbose line names which of the two paths it took.
  - On that fallback, the offset keeps its **authored length** — it is not scaled by the mesh, whose scale may belong to
    a stretched stand-in rather than to the rig the offset was written for.
- A socket that resolves carries its offset in its own space, scaled with the rig.

⚠ **`Holstered` falls back to the primary pair when no stowed socket is authored anywhere.**

- Flipping an item to `Holstered` and authoring nothing else leaves it looking exactly as it did, rather than dropping it
  at the wearer's origin.

⚠ **A stowed SOCKET, not a stowed offset, is what selects the stowed pair.**

- An offset with no socket beside it is not a place on the body.
- Honoring one would mean reading the stowed offset while taking the primary socket.
- Such a value is never applied, and `IsDataValid` says so on the item and on the slot row — at save, at *Validate Assets*
  and at cook.

⛔ There is deliberately **no per-state table** keyed on gameplay tags (combat / sheathed / mounted). A project that wants
one subclasses `USnapEquipmentComponent` and overrides where the visual is placed.

### An animation layer

`AnimLayerClass` names a `UAnimInstance` subclass linked onto the wearer's skeletal mesh while the item is worn
(`USkeletalMeshComponent::LinkAnimClassLayers`), and unlinked when it comes off.

- So a two-handed weapon can change how its wearer stands without owning a mesh at all.

The linked set is **re-derived from what is worn** rather than toggled item by item, and that has two consequences worth
relying on:

- **Two worn items may name the same layer class.**
  - Two two-handers sharing a stance, or a dual-wield layer on both weapons, is fine.
  - The class is linked once, and it stays linked until the **last** item naming it comes off.
  - Unreal's own `UnlinkAnimClassLayers` does not reference-count — it resets any layer node whose class matches.
  - So a plain unlink-on-unequip would have taken the stance off while the other item was still worn.
- **It works on a client.**
  - The set is derived wherever the worn list arrives, not only where equipping happens, so a remote player sees the
    stance too.
  - A spawned visual reaches clients by being an actor that replicates itself.
  - A layer has no such mechanism, and this is the equivalent.

**It is re-applied on a possession change:** The layer comes off the old pawn's mesh and goes onto the new one, exactly as
a spawned visual does.

- The mesh is the controlled pawn's where there is one, and the owning actor's otherwise.
- So a bare rigged actor (a character preview, a mannequin) wears layers too.

⚠ It is **not** gated on durability or spoilage, unlike the visual actor. A broken sword is still worn, and its stance is a
property of carrying it rather than of it being intact.

⚠ A path that will not load, and a class that is not a `UAnimInstance`, are named apart in one Warning per path per
session. Either way nothing is linked and the item is worn exactly as it would be with the field empty.

### Taking over the attachment yourself

A project's own visual actor can implement **`ISnapEquipmentVisual`** — a `Blueprintable` interface, so a Blueprint actor
may implement it too. It has one function:

```cpp
UFUNCTION(BlueprintNativeEvent)
bool OnEquipmentVisualSpawned(AActor* Wearer, USceneComponent* AttachTarget, FGameplayTag Slot, ESnapEquipmentVisualMode Mode);
```

- Return **true** to mean *"I have attached myself; do not attach me"*.
  - The plugin then leaves the actor exactly as you left it — no attach, no relative location, no relative rotation.
  - It still records it as the slot's visual, so unequip destroys it as usual.
- Return **false** (the default) and the ordinary path runs.

**It is one decision rather than a merge:** Either the plugin attaches the actor or you do.

- An override supplying *part* of a transform would make four authored sources for one placement.
- No reader could say where a visual ended up without knowing all four.

⚠ `Wearer` is **not** always a pawn. The inventory screen's character preview is a bare rig holding a copy of the pawn's
mesh, and it uses this same path.

⚠ The hook is not called for a `Hidden` item, because nothing is spawned to call it on. An actor that needs to know it was
equipped-but-not-shown reads the item's `VisualMode`.

⛔ There is no detach hook.

- A visual is destroyed on unequip, so `AActor::EndPlay` and `AActor::Destroyed` already fire on exactly that event.
- They fire for all three ways a visual goes away: unequip, a break, and the possession change that re-spawns everything
  worn onto a new pawn.

### Armor made of pieces

A sword hangs from one socket; a suit of armor does not.

**`ASnapEquipmentPartsVisual`** is a visual actor made of rigid pieces, each pinned to its own bone of the wearer's mesh:

- A breastplate on the upper spine.
- A greave on each calf.
- A glove on each hand.
- A boot on each foot.

Each piece is an ordinary static mesh that follows its bone as the wearer moves, which suits hard armor. For a piece that
must bend (a glove's fingers, a boot at the ankle), it can instead be a skinned mesh that follows the whole pose (below).

**To use it:**

1. Make a Blueprint subclass per item.
2. Fill its `Parts` array in the class defaults.
   - Each entry (`FSnapEquipmentVisualPart`) names a `Mesh`, the `Bone` (or socket) it follows, and an `Offset` in that
     bone's space, scale included.
3. Name the Blueprint in the item's `VisualActorClass`.

The item's own `AttachSocketName` and `AttachOffset` are not used: the actor implements `ISnapEquipmentVisual` and places
itself.

**Good to know:**

- **A piece whose bone the wearer's mesh does not have is skipped**, not drawn at the mesh origin.
  - A breastplate at a stand-in rig's feet is worse than none.
  - One Verbose line names it.
  - A piece with no mesh is skipped the same way.
- **Measure each side's bones on your own rig.**
  - A bone's frame decides what X, Y and Z mean in `Offset`.
  - On the UE5 mannequin, the two sides do not mirror each other the same way on the arms as on the legs.
  - So the left piece's offset is not the right piece's with one axis negated.
  - A negative scale on one axis mirrors the model itself (a left glove made from a right one).
- **Other players see it.**
  - The actor replicates; its pieces do not need to.
  - The server and the inventory screen's character preview draw them when the plugin hands over the wearer.
  - Every other machine draws them when the actor's attachment arrives — the same replicated attachment the engine uses to
    place any attached actor.
  - `GetPartComponents()` returns the rigid pieces drawn on the calling machine, `GetSkinnedPartComponents()` the skinned
    ones and `GetNumPartComponents()` how many of both.
- The pieces are this actor's own components attached to the wearer's mesh, so they leave with it on unequip, a break or a
  possession change.
- **A piece can bend.**
  - Set the entry's `SkinnedMesh` — a skeletal mesh made on the wearer's own skeleton, sitting where it should in that
    skeleton's reference pose.
  - It is then drawn as a skinned mesh that follows the wearer's pose bone by bone
    (`USkinnedMeshComponent::SetLeaderPoseComponent`), attached at the wearer mesh's origin.
  - `Offset` is not used; `Bone` is still the test that the wearer has that body part.
  - It is drawn only when the wearer's mesh is on the same skeleton asset. Otherwise the entry's rigid `Mesh` is drawn on
    its bone instead, or nothing (one Verbose line).
  - A skinned mesh cannot be mirrored by a negative scale, so model a pair as one mesh: entries that name the same skinned
    mesh draw it once.
  - An art pack can make the same swap without touching the Blueprint: `USnapArtPack::SkinnedParts` maps the rigid `Mesh`
    to the skinned one, read when the pieces are drawn (README, "The demo's art pack").

**The example armor uses it** — see the [demo map](demo-map.md#what-the-character-wears):

- `BP_Example_CuirassIronVisual` and `BP_Example_CuirassGildedVisual` (one piece on `spine_03`).
- `BP_Example_GreavesVisual`, `BP_Example_GauntletsVisual` and `BP_Example_SabatonsVisual` (one piece per side).
- `BP_Example_ShieldVisual` (the left forearm).

## Ownership

Equipping is refused for an instance the component's own actor does not hold.

- That check is about **ownership**, and ownership is the outer chain.
- That is why a traded item is renamed into its new owner at the handover, rather than the check being widened.

**If you move an instance between actors yourself,** do the same:

```cpp
Instance->Rename(nullptr, ReceivingActor, REN_DontCreateRedirectors | REN_DoNotDirty);
```

Widening the ownership test instead makes the symptom go away and leaves the instance's network GUID pointing at the old
owner. That fails only on a real client, after that owner disconnects.

**Custody, for extenders:**

- **An item instance is held by exactly one component at a time**: The owner's `USnapInventoryComponent` or its
  `USnapEquipmentComponent`.
  - It is rooted by a property there, registered as that component's replicated subobject, and changed only by that
    component's mutators.
  - A worn item is not in the bag: `GetAllItems`, `GetSlotIndexForItem` and `GetItemInSlot` answer "not here".
- **A transfer releases on the source before it acquires on the destination**:
  `USnapInventoryComponent::ReleaseInstance` then `AdoptInstance`, or the equipment component's `TakeCustody` /
  `ReleaseCustody`.
  - Instances are outered to the owning actor, so a transfer between the two components needs no `Rename`.
- **The equipment component's `OwnedInstances` is the custody ledger.** It roots worn items, including one between slots
  mid-swap that no entry references.
  - `ReconcileCustody` (also run on every inventory change) force-unequips any entry whose item is not in custody.
- **A worn item is never in the bag**, so `Server_RemoveItem`, `Server_DropItemsToWorld`, `MergeStacks`, `SplitStack` and
  `MoveItem` cannot touch it. "Equip it, then remove it so its grants outlive it" has no path.
- **`UnequipSlot` alone leaves the item in custody with no entry**; it is an internal step. To take an item off into the
  bag, call `UnequipToInventory` (`Server_UnequipSlot` does).
- The inventory-change hook is bound by `EnsureRuntimeBindings`, from `BeginPlay` and again before an equip.

## One instance, one slot

- The same `USnapItemInstance` must never occupy two slots at once.
- `IsInstanceEquippedElsewhere` is the check.
  - It is deliberately an exact-equality test rather than a definition comparison — two identical rings are two instances
    and may both be worn.

## Durability and wear

- If durability is enabled, worn equipment takes **damage wear** when its owner is hurt, scaled per slot and per rarity
  tier.
- A broken item stays equipped but stops granting.
- See [durability, spoilage, weight](condition-and-weight.md).

## Requirements

`USnapItemFragment_Requirements` says who may **equip** an item. Its terms:

- `RequiredOwnerTags`.
- `BlockedOwnerTags`.
- `AttributeRequirements` (an attribute NAME and a `MinimumValue`, compared with the owner's current value).
- `MinLevel`.

The owner is this component's owner — the PlayerState in Lyra. Every term is read from its ability system, unless a
**stat source** answers first (below).

**The rules:**

- **Only becoming worn is gated.**
  - Equipping from the bag, from a pouch or from nowhere, moving units off the grid onto the body, auto-move, and a
    pickup's auto-equip, upgrade-swap and worn-stack top-up all refuse while the owner does not meet the item.
  - A refused equip reports `RequirementsNotMet`.
- **What is worn stays worn.**
  - A worn item whose requirement stops being met keeps everything it grants, and can still be swapped between slots,
    split and merged.
  - Loading a save never asks.
- **An unmet pickup goes to the bag.**
  - Set `bAlsoGatesPickup` to leave the item where it lies instead.
  - That gates collection only — `USnapCollectionLibrary::GrantItemToActor` and
    `USnapCollectionLibrary::CanGrantItemToActor` — never loot, trades, purchases or crafting.
- **`MinLevel` needs a setting.**
  - `RequirementLevelAttributeName` names the attribute that holds a level, by its property name.
  - Left empty, `MinLevel` is ignored, and validation warns about each item that uses it.
- **Attributes are named by their property name**, the same way `RequirementLevelAttributeName` names the level attribute
  — `MaxHealth`, not `SnapDemoAttributeSet.MaxHealth`.
  - The editor shows a text field rather than an attribute picker.
  - That is what lets an item with requirements exist in a project built without `GameplayAbilities`.
- **A term that cannot be read is unmet**, and logs one Warning per owner.
  - It cannot be read with no ability system on this owner, a name that matches no attribute on the owner (a misspelling
    and a missing attribute set look the same from here), or a level name that matches nothing.
  - A typo never hands out gear; it withholds it.
- **A stat source is asked first.**
  - Implement `ISnapStatSource` (`GetSnapStatValue(StatName, OutValue)`) on the owner, its PlayerState or its pawn — or on
    a component of one.
  - An attribute minimum and the `MinLevel` row then ask it before the ability system.
  - A name it does not answer reads the ability system as before.
  - With no implementer, nothing changes.
  - It is asked on clients too (the screens predict whether an item can be worn), so answer from replicated state.
  - Required and blocked tags still read the ability system.
  - The demo's stats layer is one: it answers from whichever route is active, so a native buff counts toward the Warrior's
    Signet.
- **Without `GameplayAbilities`, only what a stat source answers is in force.**
  - With no stat source, a project built without it has nothing to read anywhere, so the gate could never be met.
  - Rather than make every gated item unwearable, the item equips freely, and one Warning per owner and item says that
    nothing is in force.
  - With a stat source, every term it answers is in force, met or not.
  - A term it does not answer, and every tag term, is not, with its own Warning.

**For a UI:**

- `USnapItemFragment_Requirements::EvaluateRequirements` returns every term as a row with its required and current value,
  for a UI to show.
- `USnapItemFragment_Requirements::DescribeEquipRequirements` returns the refusal alone.

**What the player sees.**

- The tooltip lists every term with the player's own value (see [Tooltips and stats](tooltips-and-stats.md)).
- The context menu keeps an **Equip** row that only the requirements stop, drawn grayed with the first unmet term —
  "Equip -- Requires Attack Power 12".
  - An item that fits no slot on this body has no Equip row at all.
- A multi-item selection counts and equips only the members that meet their requirements, with no notice for the rest.
  - A selection where none do is grayed.
- A drop onto a worn slot — a mouse drag or a gamepad carry — names `RequirementsNotMet` and raises one notice; see
  [Refusals](refusals.md).
- `USnapItemStatReader::DescribeRequirementTerm` is the short sentence the menu uses, for a UI of your own.

⚠ Grant required and blocked tags with gameplay effects. A loose tag added on the server does not replicate, so the owning
client's equipment UI would disagree with the server about it. See [GAS and Lyra](gas-and-lyra.md).

⚠ `USnapEquipmentComponent::ReseatWornInstance` is the C++ entry for putting back units that were already worn. It asks
slot admission but not requirements; the spoil transform and a death drop's put-back use it.

## Refusals

Refusals reach the player through `OnActionRefused` on this component. Two families report:

- **Putting something in.** Every `Server_` wrapper whose verb can place an item or units in a slot asks
  `DescribeEquip(Instance, Slot)` before the verb and reports once.
- **Taking something out.**
  - `Server_UnequipToInventory` and `Server_UnequipToStorage` ask whether the bag will accept what is coming off.
  - A worn slot's shift+click sends `Server_AutoMoveWornItemToInventory` or `Server_UnequipWornItemToStorage` instead.
  - Those are the same requests naming the item the player pressed. They move nothing (and report nothing; one Log line on
    the server) when the slot no longer holds it.

⚠ Not every path reports.

- The non-`Server_` verbs are also reached by recovery legs — a swap re-equipping what it just displaced — where a second
  notice for one gesture would be wrong.
- A refusal that reaches only the log is deliberate there. See [refusals](refusals.md).

⚠ `DescribeEquip` answers about **admission**, and deliberately not about room:

- Is the slot unlocked?
- Is it unspanned?
- Does the owner meet the item's requirements (asked only of an item not already worn here)?

**Room depends on the gesture.** Whether the bag can take what an equip would turn out depends on which gesture is being
performed:

- A merge into a worn stack displaces nothing.
- A rarity upgrade drops what it replaces to the world rather than into the bag.

The three whole-item equip wrappers ask that question for themselves. That is why `NoRoomForDisplaced` can reach a player
from them and not from a stack door.

⚠ A client cannot pre-reject `NoRoomForDisplaced`, by design.

- A predictive UI may over-offer; it must never under-offer, because that removes an action the player is entitled to with
  no explanation.
- The server refuses and reports the reason.

## Settings

Everything configurable is on **Snap Inventory System - Equipment**, with attachment on **Snap Inventory System - Foundation → Attachment**,
`ConsumeSlots` on **Snap Inventory System - Foundation → Consume**, `bAutoEquipOnPickup` on **Snap Inventory System - Inventory → Pickups**,
`bSwapToBetterRarityOnPickup` on **Snap Inventory System - World → Pickups**, durability, stats and requirements on **Snap Inventory System - Items**,
and `bEquipOnDoubleClick` on **Snap Inventory System - UI Core**; see
[appendix: settings](appendix-settings.md).
