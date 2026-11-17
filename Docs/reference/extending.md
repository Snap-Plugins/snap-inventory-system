# Extending the plugin

Eleven seams, roughly in order of how much you have to know to use them. This page covers each one, from settings to your
own HUD.

Every public name on this page is covered by the plugin's [deprecation policy](../../CHANGELOG.md#deprecation-policy).
From 1.0.0 on, a name is marked and kept for at least one minor release before it is removed.

**On this page:**

- [1. Settings](#1-settings)
- [2. Admission — what a container takes](#2-admission--what-a-container-takes)
- [3. Subclassing the components](#3-subclassing-the-components)
- [4. Fragments](#4-fragments)
- [5. Persistence](#5-persistence)
- [6. Raising your own refusals](#6-raising-your-own-refusals)
- [7. Context menu actions](#7-context-menu-actions)
- [8. Your own drop actor](#8-your-own-drop-actor)
- [9. Your own interaction options](#9-your-own-interaction-options)
- [10. Where the inventory lives](#10-where-the-inventory-lives)
- [11. Your own HUD](#11-your-own-hud)
- [Two things to check before you extend](#two-things-to-check-before-you-extend)

---

## 1. Settings

- There are 380 of them.
- Colors, sizes, class pointers, tags, thresholds and feature switches are all config.
- They are one settings class per core, except Quests, which has none (`USnap<Core>Settings`, each with its own Project
  Settings page), each `BlueprintType` — so Blueprint reads them through its core's getter:

```
USnapFoundationLibrary::GetSnapFoundationSettings()
USnapUICoreLibrary::GetSnapUICoreSettings()
USnapItemLibrary::GetSnapItemsSettings()
USnapInteractionSettings::GetSnapInteractionSettings()
USnapInventoryLibrary::GetSnapInventoryCoreSettings()
USnapEquipmentLibrary::GetSnapEquipmentSettings()
USnapCraftingSettings::GetSnapCraftingSettings()
USnapEconomySettings::GetSnapEconomySettings()
USnapWorldSettings::GetSnapWorldSettings()
USnapInventoryUISettings::GetSnapInventoryUISettings()
USnapInventoryDemoSettings::GetSnapInventoryDemoSettings()
```

See [appendix: settings](appendix-settings.md).

---

## 2. Admission — what a container takes

### By mode

Each container has an `AdmissionMode`:

| Mode | Takes |
|---|---|
| `ExactTag` | Items whose routing tag matches this container's exactly |
| `MatchesTag` | That tag or any child of it |
| `TagQuery` | Whatever `AdmissionQuery` matches |
| `AdmitAll` | Anything that is not a counter |

They are both `EditAnywhere` and `config`, so a project can change them without touching an asset.

⚠ `MatchesTag` admits everything `ExactTag` does: equal tags match too, empty ones included.

- So the untagged main bag still takes ordinary items under it, and refuses routed ones, exactly as under `ExactTag`.
- Only a tagged container gains anything: its tag's children.

### By policy

For a decision the modes cannot express, attach a `USnapInventoryAdmissionPolicy` — `Blueprintable`, `EditInlineNew`, held
`Instanced`, null by default:

```cpp
UCLASS(Blueprintable, EditInlineNew)
class UQuestItemsOnly : public USnapInventoryAdmissionPolicy
{
    GENERATED_BODY()
    // override the admission hooks here
};
```

```
SetAdmissionPolicy(NewPolicy)
```

- It refuses a class-default object or archetype, and a policy another container already owns.
  - It renames only an unowned one into itself.
- A policy is an `Instanced` subobject, so its outer must be the component that holds it.

**The policy has four hooks:**

- `CanAcceptItem(Container, ItemDef)`: may this definition go in at all, ignoring capacity.
  - It is asked only about what `AdmissionMode` already admits, so a policy narrows admission and never widens it.
  - It is asked in routing scans, and more than once for one refusal (four times for a refused adoption). Keep it cheap
    and pure.
  - It takes no count. Put a per-container unit limit in the Stackable fragment's `MaxUnitsPerContainer`.
- `CanAcceptInstance(Container, Instance)`: the same question for an existing item moving in (an adoption, a pickup, an
  unequip).
  - It is for rules that need the item's state: durability, spoilage, a rolled tier.
  - It is not asked on a fresh `AddItem`.
- `PreAddItem(Container, ItemDef, Count)`: a notice just before `AddItem` commits new units. It is not a gate.
- `PostRemoveItem(Container, ItemDef, Count, SlotIndex)`: a notice after `RemoveItem`, with the container already
  consistent.

⚠ **The two notices fire for created and destroyed units only.**

- An item that moves between containers, is unequipped, looted or traded fires neither.
- So a tally built on them drifts.

⚠ **A `PreAddItem` that adds items itself can use up the room the add was counting on.**

- The add then stops at capacity and places fewer units than asked.
- `AddItemWithRemainder` reports how many did not land; `AddItem` does not.
- Do such work on the next tick.

⚠⚠ **The policy is consulted only on authority**, and every gesture site that asks admission is a client widget.

- A local pre-check can report a mode or tag refusal, and reports **nothing** for a policy refusal — the case the feature
  exists for.
- That has to come back through `Client_ReportRefusal`.
- Ordinary Play-in-Editor is a listen server and passes either way.

---

## 3. Subclassing the components

Every component is `Blueprintable`, so `USnapInventoryComponent` and friends can be subclassed in Blueprint or C++.

⚠⚠ **If you subclass the main bag, leave `bIsMainInventory` ticked.**

- That marker is how the hotbar and the saving path find your bag.
- It is also which grid the pad goes to on a screen with more than one
  ([input](input.md#a-screen-with-more-than-one-inventory-grid)).
- There is deliberately no "the exact class, otherwise the first subclass found" fallback.
  - That rule silently points the consumable bar at whichever container happens to be first on the actor.
- A container that does not claim the role is not the main bag, and null is the honest answer.

The plugin's own non-main containers (`USnapCustomContainerComponent`, so every custom container, and
`USnapLootInventoryComponent`) clear the marker in their constructors.

⚠ **Do not give a Blueprint variable a name an inherited `UPROPERTY` already uses.**

- A Blueprint variable and an inherited native property share one scope on the generated class.
- The collision puts the Blueprint into a permanent compile error.

Three functions are `virtual` for a C++ subclass: `AcceptsItem`, `CanAddItem`, `ComputeWorldDropLocation`.

`IsOwnerDead` is a `BlueprintNativeEvent` for either kind of subclass.

- Every "not while dead" rule in the plugin asks it on the main bag.
- So overriding it is how a project with no death tag says its player is dead
  ([Death and loss](death-and-loss.md#saying-the-player-is-dead)).

### A new container, without C++

A Blueprint of `USnapCustomContainerComponent` ([custom containers](custom-containers.md) walks through one) — or of
`USnapInventoryComponent` itself — with `ContainerTagOverride` set in its class defaults **is** a distinct container.

- `GetContainerTag()` answers that tag, and every routing and lookup path follows.
- That is because they all ask that function and none of them reads a class.

**To set one up:**

1. Clear `bIsMainInventory` unless it really is the main bag.
2. Set `MaxNumSlots` (or `SlotCountOverride`) for its size.
3. Add a row for it to the container registry so it is attached, after the base `USnapInventoryComponent` row, with
   `bAddUnique` off.
   - The registry is the asset `ContainerRegistry` names (the plugin ships `DA_SnapContainerRegistry`).
   - The `ComponentAttachSpecs` rows on **Snap Inventory System - Foundation → Attachment** are read only when that setting is
    cleared, or when the asset it names does not load (a warning says so).
4. Give a `USnapInventoryPanel` the same tag in `ContainerTag`, so the UI finds it.

⚠ One row per container CLASS.

- `UGameFrameworkComponentManager::AddComponentRequest` takes no per-request data.
- So two rows naming the same component class produce two identical components with no way to tell them apart.
- That is why the tag lives on the component rather than on the row.

---

## 4. Fragments

`USnapItemFragment` is `Blueprintable` and carries `BlueprintNativeEvent` hooks:

```
OnInstanceCreated(Instance)
OnEquipped(Instance, Owner, Slot)      OnUnequipped(...)
CanEquip(Instance, Owner) -> bool
DeclaresConsumeGrant(ItemDef) -> bool  CanApplyConsumeGrantTo(ItemDef, GrantTarget) -> bool
ApplyConsumeGrantTo(OwningActor, ItemDef, GrantTarget)
GetConsumeCooldownFor(Instance, Asker, Remaining, Duration)   (optional: the cooldown a refusal waits on, for the hotbar)
GetTooltipLines(Instance) -> lines    GetActiveEffectLines(ItemDef) -> lines   (what its card says; what its running effect's card says)
```

- The three consume hooks go together: they make a fragment of your own the thing that happens when its item is consumed
  ([items and fragments](items-and-fragments.md#a-fragment-that-makes-its-item-a-consumable)).
- Plus the admission hooks the policy object also implements.

⚠ A hook runs **inside** the mutator that called it, so adding or removing items from one re-enters the component.
Iterate a copy of the fragment array if you iterate at all.

⚠ Overriding a `BlueprintNativeEvent` in Blueprint **replaces** the native implementation, and there is no parent call
from Blueprint. Read what `_Implementation` does before overriding it.

---

## 5. Persistence

**To plug in your own provider:**

1. Implement `ISnapPersistenceProvider`.
2. Name your class in `PersistenceProviderClass` (**Snap Inventory System - Foundation → Persistence**).

- There are five calls, and payloads are `FInstancedStruct`, so your provider never has to know what is inside them.
- See [persistence](persistence.md).

Per-instance state of your own goes in `Memories` — a typed struct saved and restored with the instance, which the plugin
never inspects.

---

## 6. Raising your own refusals

```cpp
Component->ReportRefusal(FSnapRefusal{ Domain, Code, SubjectItemDefinition });   // or no subject
```

- **From Blueprint,** call `USnapInventoryComponent::ReportActionRefusal(Domain, Code, Subject)` — the same entry
  point, so the player hears about it exactly once in every net mode.
- `ESnapRefusal::RefusedByPolicy` is the code for "the project's own rule said no".
- Bind `OnActionRefused` in Blueprint to consume it.
- The vocabulary has spare codes for exactly this. See [refusals](refusals.md).

---

## 7. Context menu actions

The right-click menu's own rows are fixed, but a project can add rows of its own and can remove or reorder any row.

- Both are done from Blueprint, and both are named in **Project Settings > Snap Inventory System - Inventory UI > Context Menu**.

### Your own row

1. Subclass `USnapInventoryContextAction` (it is `Abstract`; list a subclass, never the base).
2. Override three functions:

| Function | Answers |
|---|---|
| `CanPerform(Member)` | May this row act on this one item? The default is **no**, so a row that does not say is never offered. |
| `GetLabel(Subject)` | The row's noun ("Mark for Trade"). The plugin appends the selection count itself. The default is the class's display name — in a packaged build without its `DisplayName` metadata, which only the editor has. |
| `Perform(Subject)` | Do it. `GetSubjectItems(Subject)` lists the items. |

3. Add the class to `ContextMenuActions` (**Snap Inventory System - Inventory UI → Context Menu**) with `bEnabled` ticked.

Four class defaults on the action decide the rest:

- **`ActionTag`:** The row's identity to a menu policy.
  - Required, and it must not be another row's tag.
  - An action without a usable tag is not offered, and the menu logs one warning naming it.
- **The `bCostsTheItem` flag:** Where the row goes.
  - The menu is ordered least destructive first, and its rule for any entry is "does pressing it cost the player the
    item, or any part of it?".
  - False places the row right after `Unpack`; true places it right before `Drop`, which stays last.
  - Several rows in one group keep their order from the settings list.
- **The `bSingleSubjectOnly` flag:** Withhold the row from a selection of more than one item.
  - Use it for anything that opens a window about one thing.
- **The `bPerformsOnDoubleClick` flag:** A double-click performs this row.
  - Double-clicking an item uses it (while `bConsumeOnDoubleClick`, on **Snap Inventory System - UI Core → Input → Gestures**, is on): a consumable is consumed exactly as the menu's
    Consume would.
  - On an item Consume does not apply to, the double-click performs the **first** listed row that sets this and that the
    menu would draw live for that one item.
  - The project's list, each row's set rule and the menu policy all apply, as for a right-click.
  - Off by default, so listing a row never changes what a double-click does unless the row asks.
  - The example demo's Learn row on a spell scroll is the worked example.

**The plugin does the set arithmetic every built-in row gets:**

- `CanPerform` is asked once **per selected item**, and the row is offered only when every one says yes (an item that has
  gone away is dropped first).
- With more than one item, the label reads "Mark for Trade (3)".
- When the row is pressed, `Perform` is called **once**, after the menu has closed, with only the items that still pass
  `CanPerform` at that moment.

⚠⚠ **`Perform` runs on the owning client.**

- An action that changes game state must do it through its **own server RPC**, on an actor or component the player owns.
- It must check again on the server — a client's `CanPerform` is a prediction.
- The plugin's junk sweep is the worked example of the shape: `USnapInventoryComponent::Server_SellAllJunk` re-derives
  everything on the server and answers through `Client_ReportJunkSweep`.
- To tell the player why the server refused, call `ReportActionRefusal` there (section 6).
- There is deliberately no generic "run this action on the server" RPC — it would be a door any client could knock on
  with any class.

⚠ One instance of each action class exists per player, owned by that player's `USnapInventoryHUDComponent` (which is also its
outer, so world-context Blueprint nodes work).

- A player without that component gets the built-in rows only.
- `CanPerform` and `GetLabel` run on every menu open — twice — so keep them cheap and free of side effects.

⚠ If `Perform` raises a **modal**, it is yours to close on every path (death, a screen closing under it). The plugin
cannot know it exists.

### Removing and reordering rows

1. Set `ContextMenuPolicyClass` to a subclass of `USnapInventoryContextMenuPolicy`.
2. Override `FilterEntries`.
   - It receives every row the menu would draw — built-in and project rows alike — as `FSnapInventoryContextMenuEntry`
     (`ActionTag`, `Label`, `bDimmed`).
   - It returns the rows to draw, in order.

Built-in rows are named `SnapInventorySystem.ContextAction.<Row>` (`...Drop`, `...Sell`, `...ToggleLock`).

A policy can **remove** and **reorder** and nothing else.

- Only `ActionTag` is read back, and only tags it was offered are accepted, each once.
- An unknown tag, a repeat, or a row it invented is dropped with one warning per policy class.
- So whether a row is offered, whether it is dimmed, and what it does are never the policy's to change.

⚠ **The policy runs twice per menu open** — once to decide whether a menu opens at all, once to build it.

- So it must be free of side effects and answer the same way both times.
- A policy that removes every row leaves nothing to open, so no menu opens.
- One that leaves only dimmed rows opens with those.

**Starting points:**

- `USnapInventoryContextMenuPolicy_KeepAll` is the concrete pass-through.
- `USnapInventoryContextAction_Default` is the concrete action that offers nothing.
- They are a starting point for a Blueprint subclass, and a control: listing either changes no menu.

### The Inspect example

`USnapInventoryContextAction_Inspect` ships in `ContextMenuActions`, **disabled**.

1. Tick it.
2. Name a widget in its `InspectWidgetClass`:

```ini
[/Script/SnapInventorySystem.SnapInventoryContextAction_Inspect]
InspectWidgetClass=/Game/UI/WBP_MyInspectWindow.WBP_MyInspectWindow_C
```

- It is a no-cost, single-item row labeled "Inspect".
- Pressing it creates that widget for the player who right-clicked and adds it to the viewport.
- If the widget implements `ISnapInventoryInspectWidget`, `ReceiveInspectSubject` hands it the item **before** it is added,
  so its Construct can read it.
- Closing the widget is the widget's own business.
- Enabled with no widget class, Inspect is never offered and the menu logs a warning once.

### The menu's own widget classes

- The rows are built in code on every open: each row is a `UButton` subclass carrying its action.
- The rows have no class knob.
- **The popup around them is a Widget Blueprint you can own.**
  - `ContextMenuWidgetClass` (Project Settings, **Snap Inventory System - Inventory UI → Context Menu**) ships naming
    `WBP_ExampleContextMenu`.
  - A Blueprint with a designer tree of its own works — its rows go into the panel named by **Row Container Widget
    Name**.
- Actions and a policy cover what to show; how the menu looks follows the theme. See
  [UI and theming](ui-and-theming.md#replacing-a-native-surfaces-class).

---

## 8. Your own drop actor

A dropped item becomes the plugin's own `ASnapPickupActor` unless you name another class. Four links decide, in
order, and every miss falls to the next:

1. A `DropActorClassByItemDef` row on the dropping component (config), matched by exact item definition.
2. The item definition's own drop actor — `DropActorClass` on its `USnapItemFragment_WorldRepresentation`.
3. The component's `DroppedPickupClass`.
4. `ASnapPickupActor`.

**Good to know:**

- A row or a fragment that names a class that does not load, or is not an actor, is logged once and falls through.
- A row still wins over the item's own class, so a project can re-skin any item without editing it.
- An item carrying other items inside it always drops as the plugin's own pickup, whatever the rows say, because only
  that actor can hold the contents.

### Taking the item as one struct

Implement `ISnapDroppedPickup` on a class a row names, and the drop hands it an `FSnapDropSeed` through `SeedDrop`. The
seed holds:

- The definition and the count.
- The rarity tier.
- Durability.
- The spoilage reading and the server time it was taken at.
- The item's identity.
- Its memories (its marks and its binding among them).
- A launch velocity.
- Why it left (`Drop` or `DeathDrop`).

**Rules:**

- **`SeedDrop` runs before `FinishSpawning`.**
  - The construction script and `BeginPlay` have not run.
  - So store the seed and do nothing else; read it in `BeginPlay`.
- **An actor that implements the interface gets no by-name write at all.**
  - Nothing below is written to it, not even `CanCollect`: it arms itself.
  - Make it collectable in `BeginPlay` or later, never inside `SeedDrop`.
  - Otherwise a pawn already standing on the spot can collect it during its own spawn.
- **Collect it with `USnapCollectionLibrary::GrantDropSeedToActor`**, passing the stored seed back.
  - It returns how many units landed — all, some or none — and reduces the seed to match.
  - Destroy the actor when its `Count` reaches 0; keep it carrying the reduced seed otherwise.
  - The item comes back as it went down: the same identity, the same wear, the same marks, and a spoilage clock that kept
    running while it lay there.
  - The identity goes with the first units that land, so whoever collects a remainder gets a new item, as a split stack
    would.

A subclass of the plugin's own pickup is never handed a seed; it carries the item in its own fields.

### An item's own look

When a drop lands as the plugin's own pickup — links 3 and 4 — the pickup draws the item's
`USnapItemFragment_WorldRepresentation` `Mesh` at its `Scale`, keeping the mesh's own materials, instead of a fallback
shape.

- It resolves the mesh from the item it carries on every machine, so clients draw it with nothing extra replicated.
- So do the pickups a destructible container's loot, a despawn scatter and a restored world spawn.
- A pickup that already has a mesh keeps it.
- A project actor draws itself and ignores both values.

### A class that cannot implement the interface

A class that does not implement it — a Blueprint in a module that must not depend on this plugin, say — is written by
name instead, before `FinishSpawning`. Each name it does not declare is skipped:

| Name | Type | Holds |
|---|---|---|
| `PickupCount` | `int32` | The count (a class without it logs a Warning: the drop is not conserving) |
| `CurrencyAmount` | `int32` | The count again |
| `PluginItemDef` | `FSoftObjectPath` | The definition (a class without it logs a Warning) |
| `Rarity` / `RarityColor` | `FGameplayTag` / `FLinearColor` or `FColor` | The tier and its color, only when the item has one |
| `Durability` / `DurabilityMaxWear` | `int32` | Wear, only when the item has any |
| `SpoilRemainingSeconds` / `SpoilCaptureServerTime` | `int32` / a Float (double) or C++ `float` | The spoilage reading and when it was taken, only when the item spoils |
| `InstanceId` / `Memories` | `FGuid` / `TArray<FInstancedStruct>` | Identity and memories, only for a whole stack |
| `SpawnImpulse` | `FVector` | The launch velocity, in cm/s |
| `CanCollect` | `bool` | Set true AFTER `FinishSpawning` |

- `USnapCollectionLibrary::MakeDropSeedFromActor` reads those names back into a seed, so such an actor collects
  through `GrantDropSeedToActor` too.
- **Give `Durability` and `SpoilRemainingSeconds` a default of -1.**
  - They are left unwritten for an item without that data, and whatever default they carry is what comes back.
- **Anything not in the table is never written**, for example a game's own type or category tag.
  - The dropped actor is built from the mapped class's defaults, so set such fields as class defaults on every mapped
    class.
  - A mapped class whose own type tag is empty may never be picked up by game logic that filters on it, and nothing logs
    it.

### Hearing about drops

`USnapInventoryComponent::OnItemDropped(Actor, ItemDef, Count, Kind)` fires on the server once for every actor a drop
puts into the world, whichever class it is.

- It fires on the tick AFTER the drop — by then the item has left the inventory, so a handler never sees it in two places.
- `Actor` is null when the actor was collected or destroyed during its own spawn; the drop still happened.
- It is not raised on clients.

### Where drops land

`DropTraceChannel` (**Project Settings > Snap Inventory System - Inventory > World Drops**, default `ECC_WorldStatic`) is the channel
every drop traces on:

- The ground under each candidate spot.
- The check that the item fits there.
- The line from the player to it.
- A destructible container's loot.
- A death's lootable body.

Where nothing answers it, a drop falls back to a spot fanned out beside the player at the height of the player's feet.

### How a drop moves

`DropMotion` (**Project Settings > Snap Inventory System - Inventory > World Drops**) decides whether a drop may physically eject what
it spawns:

- **`Auto`:** The default, and the behavior before the setting existed.
  - The plugin's own pickup plays its throw arc and lands on the spot the placement chose.
  - A project actor is handed the dropping component's `DropImpulseStrength` as its launch velocity (`SpawnImpulse`).
  - One whose root simulates physics lands wherever physics takes it.
- **`Cosmetic`:** The arc still plays.
  - A project actor's launch velocity is zero whatever `DropImpulseStrength` says, so it is not launched.
  - It is seated at the spot the placement chose — checked for ground, room, reach and spacing.
  - One whose root simulates physics settles there under its own weight.

The plugin's own pickup never simulates physics in either mode: only its mesh flies, along an arc every machine replays
identically.

Either way, an actor that comes to rest is seated on the ground by the extent of what it DRAWS — its `UMeshComponent`s.

- A particle effect's bounds reach as far as its particles do, and measuring them used to lift the actor off the floor by
  that much.
- An actor with no mesh component to measure falls back to its whole-actor bounds, which is what the seat always used, so
  its placement is unchanged.

---

## 9. Your own interaction options

An actor that carries a `USnapInteractableComponent` can implement `ISnapInteractable` to add options of its own — "pull
the lever", "read the notice" — without subclassing the component.

It answers four questions:

- Which options it offers a player (`GetSuppliedInteractionOptions`).
- Whether one is available (`CanPerformSuppliedOption`).
- What the prompt says (`GetSuppliedOptionPromptText`).
- What pressing it does (`PerformSuppliedOption`).

**Good to know:**

- Its options follow the component's authored ones, use the `None` verb, and are counted, cycled, held and broadcast like
  any other.
- The component stays required: it is what makes the actor focusable.
- See [Loot and interaction](loot-and-interaction.md#an-actors-own-options).

## 10. Where the inventory lives

The plugin asks the actor its components hang off three questions:

- Which **player controller** should hear about something (a refusal notice, a screen opening).
- Which **pawn** represents it in the world (what reach is measured from, what a drop happens in front of).
- Which **player state** it belongs to (what the owner is named by).

It answers all three by walking the actor it is attached to. For a player, that walk is enough:

- A PlayerState's `Owner` is the controller.
- Its `GetPawn()` is the pawn.
- It is its own PlayerState.

An owner that is **none of those three** — a chest, a corpse, a vendor's back room, an NPC's pack, a GameState holding a
shared stash — has no such walk.

- It can hold items perfectly well, because `USnapInventoryComponent` never cared what it was attached to.
- But every one of those three questions answers null for it, so a refusal raised against it reaches nobody.

**`ISnapInventoryHost` is how such an owner answers for itself.** Implement it on the actor (C++ or Blueprint) and override
only the questions you have an answer for:

```cpp
UCLASS()
class AMyChest : public AActor, public ISnapInventoryHost
{
    GENERATED_BODY()
public:
    UPROPERTY(Transient) TObjectPtr<APlayerController> ViewingController;

    // The one question a chest can answer that the walk cannot.
    virtual APlayerController* ResolveHostController_Implementation() override { return ViewingController; }
};
```

`ASnapDemoContainerHost` in the demo classes is exactly this, and it is the shape to copy.

- **A project that implements nothing is unaffected.**
  - Every question has a default that reproduces the walk the plugin already performed.
  - So declaring nothing — or declaring the interface and overriding nothing — resolves exactly as it always did.
  - The interface is a seam, not a requirement.
- **Override only what you can answer.**
  - A chest has no pawn and belongs to no PlayerState; the honest answer to both is null, which is what the defaults
    already say.
  - Implementing an interface does not mean answering all of it.
- ⚠ **A Blueprint override REPLACES the default; there is no parent call.**
  - Overriding `ResolveHostController` and returning nothing means *"this host has no controller"*, not *"fall back to
    the walk"*.
  - A host that says null is taken at its word.
  - In C++ you can call the matching `SnapInventoryHost::DefaultResolveController` (or `DefaultResolveOwnerPawn`,
    `DefaultResolveOwnerPlayerState`) yourself to extend the walk instead of replacing it.
- ⚠ **Ask through the free functions, never a cast to the interface.**
  - `SnapInventoryHost::ResolveController`, `SnapInventoryHost::ResolveOwnerPawn` and
    `SnapInventoryHost::ResolveOwnerPlayerState` check `Implements<>` and go through `Execute_`, so they reach a Blueprint
    implementer.
  - A cast finds only native C++ ones and would make a Blueprint host silently invisible.

### Hosting an inventory on the GameState

A GameState can hold a shared stash, and it holds and moves items like any other owner.

⚠ **It cannot drop to the world on its own**, and that is deliberate.

- `AGameStateBase` is an `AInfo`, the engine's base for an actor with no physical representation.
  - Its only root component is a billboard created under `WITH_EDITORONLY_DATA`.
- Dropping from one would read a location of (0,0,0) in a packaged build and pile every item at the world origin,
  unreachable.
- The plugin refuses instead and logs it, exactly as it does for a PlayerState whose player is dead.

To make a GameState-hosted drop work, tell it which avatar the drop should come out in front of:

```cpp
virtual APawn* ResolveHostPawn_Implementation() override { return PawnThatAskedForThis; }
```

- The same applies to any `AInfo` subclass, including a GameMode.
- An ordinary actor — a chest standing somewhere — needs none of this: it has a transform, so it drops where it is.

## 11. Your own HUD

The plugin's gameplay code — the inventory component, the hotbar, interaction, persistence and the Blueprint library —
does not find `USnapInventoryHUDComponent` by class.

- When it needs a screen, it asks the player controller for one of six small interfaces, each covering one job:
  - The controller itself, if it implements that interface.
  - Otherwise the first of its components that does (`SnapScreenHost::FindForController` and its siblings, one per
    interface).
- The shipped `USnapInventoryHUDComponent` implements all six, so a project that keeps the plugin's HUD changes nothing.

A project that replaces the plugin's screens implements the same twenty-two events, in C++ or Blueprint, on its own
controller or on a component of it, each on the interface that holds it:

| Interface | Its job | Events |
|---|---|---|
| `ISnapFeatureScreenOpener` | The screens an interaction opens | `SnapOpenVendorScreen`, `SnapOpenLootScreen`, `SnapRequestTrade` |
| `ISnapCraftStationHost` | A crafting station's window | `SnapEnsureCraftScreen`, `SnapOpenCraftScreen`, `SnapCloseCraftScreen`, `SnapIsCraftScreenOpenForStation`, `SnapWatchStationRange` |
| `ISnapScreenHost` | The bag screen, persistence, death, a container open beside the bag | `SnapSetInventoryScreenVisible`, `SnapIsInventoryScreenVisible`, `SnapIsAnyScreenOpen`, `SnapSaveScreenLayoutNow`, `SnapCloseScreensForDeath`, `SnapSetTransferContainer`, `SnapClearTransferContainer` |
| `ISnapConsumeRouter` | Consuming through an ability | `SnapCanRouteConsumeToAbility`, `SnapRouteConsumeToAbility` |
| `ISnapRefusalReporter` | Telling the player | `SnapShowVendorRefusal`, `SnapShowJunkSweep`, `SnapShowTradeRefusal` |
| `ISnapReadoutHost` | The counter readout and the hotbars | `SnapShowCounterValue`, `SnapShowHotbarBars` |

- **Each, except `SnapIsAnyScreenOpen` and `SnapShowHotbarBars`, stands for a `USnapInventoryHUDComponent` function
  under another name.**
  - A Blueprint interface event may not share a name with a function on the class implementing it.
  - The shipped HUD already has `OpenVendor`, `IsInventoryVisible` and the rest.
  - Those functions are unchanged and still callable.
- **Implement only the interfaces whose jobs your HUD takes over.** The plugin asks each one on its own.
- **A parameter that would name a plugin class from a part above the interface's own takes its engine base**
  (`UActorComponent*`, `UPrimaryDataAsset*`, or `UObject*` for an item instance), so these interfaces do not reach into the
  parts of the plugin above them: cast to the class you expect, as the shipped HUD does (it refuses another class with a
  warning). A plugin type from the interface's own part or below keeps its type: `SnapShowVendorRefusal`'s `Subject` is a
  `const USnapItemDefinition*`.
- **Anything you leave alone does nothing.**
  - Every default answers "no screen, nothing done" — false, zero or no effect.
  - That is what the plugin already got from a controller with no HUD at all.
  - The one exception is `SnapEnsureCraftScreen`, whose default answers true.
    - With `bCraftingStationOpensInventory` off, a crafting station asks it before `SnapOpenCraftScreen`, and a HUD
      that does not implement it must not have every station refuse on its behalf.
- ⚠ **What a station press opens depends on `bCraftingStationOpensInventory`** (**Snap Inventory System - Crafting → Crafting**,
  on by default).
  - On, it shows the bag first and asks `SnapOpenCraftScreen` (never `SnapEnsureCraftScreen`).
  - Off, it asks `SnapEnsureCraftScreen`, then `SnapOpenCraftScreen` with no bag up.
  - Either way, a window that opened is then handed to `SnapWatchStationRange`.
- ⚠ **`SnapIsAnyScreenOpen` is the one to get right.**
  - The interaction detector asks it to keep the world prompt and the interact key out from under an open screen.
  - Answer true for every screen AND every modal your HUD can raise.
  - A modal left out lets the interact key act underneath it.
- ⚠ **One implementer per controller.**
  - The controller is asked first and then its components in order.
  - So a replacement HUD beside the shipped `USnapInventoryHUDComponent` is found or not depending on that order.
  - Remove the one you are replacing.
- **"Is this player dead?" is not on the interface.**
  - The player's main bag answers that for every refusal in the plugin.
  - Override `USnapInventoryComponent::IsOwnerDead` (see [Death and loss](death-and-loss.md)).

## Two things to check before you extend

- **Does a predicate you are adding read replicated state?**
  - If a client can call it, it must not read `NotReplicated` fields.
  - See [networking](networking.md).
- **Are two paths now answering the same question?**
  - They must not be able to disagree.
  - Where the plugin has a `Describe*` and a mutator, both ask one planner; keep that shape.
