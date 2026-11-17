# Persistence

Five things can be saved: an actor's inventory and equipment, hotbar bindings, saved loadouts, window layout and world
state. All five go through one seam, so a project that already has a save system plugs into it rather than
working around it.

**On this page:**

- [The seam](#the-seam)
- [What is stored](#what-is-stored)
- [Versioning](#versioning)
- [Automatic saving](#automatic-saving)
- [A container that keeps its own save](#a-container-that-keeps-its-own-save)
- [World state](#world-state)
- [Window layout](#window-layout)
- [See also](#see-also)

## The seam

`ISnapPersistenceProvider` is a Blueprint-implementable interface with five calls:

```
SaveStore(ProfileKey, StoreId, Payload)      -> bool
LoadStore(ProfileKey, StoreId, OutPayload)   -> bool
DeleteStore(ProfileKey, StoreId)             -> bool
DoesStoreExist(ProfileKey, StoreId)          -> bool
ResolveProfileKey(PlayerController)          -> FString
```

- A payload is an `FInstancedStruct`, so the provider never has to know what is in it.
- Name your implementation in `PersistenceProviderClass` and everything routes through it.
- The shipped default is `USnapSaveGamePersistenceProvider`, a plain `USaveGame` writer.
- A project that wants no persistence at all clears the class.

⚠ `ResolveProfileKey` never returns an empty string. An empty key would collapse every player's saves onto one record.

## What is stored

`FSnapActorInventoryRecord` (version 3) carries:

- The containers and the equipment.
- One section per component beside the bag (`ComponentSections`), among them the wallet's counters
  (`FSnapWalletRecord`), crafting's unlocked recipes, craft queue and finished crafts waiting at stations or for room in
  the bag (`FSnapCraftingRecord` —
  [Finished crafts wait at the station](crafting-stations.md#finished-crafts-wait-at-the-station) and
  [Timed queues](crafting.md#timed-queues); restored behind the craft queue's own duplicate guard), and the quest log's
  finished and active quests (`FSnapQuestLogRecord` — [Quest givers](quest-givers.md)).
- An older record (version 1 or 2) is brought up to date when it is read.

The item record:

```
FSnapItemRecord
  ItemDef               soft path
  Count
  SlotIndex
  ParentLineIndex       which pouch this item is inside, or INDEX_NONE
  Variant               the per-instance roll -- today the rarity tier
  Durability / DurabilityMaxWear
  SpoilRemainingSeconds
  InstanceId            FGuid, stable across the round trip
  Memories              TArray<FInstancedStruct> -- your own per-instance state
```

`ParentLineIndex` is what makes nested pouches restore in the right shape.

A container laid out as a spatial grid also writes a sparse side table beside its items:

```
FSnapInventoryRecord
  Placements            TArray<FSnapItemPlacementRecord> -- { LineIndex, Rotation }
```

- Only items that are actually turned get an entry.
  - So a container laid out in `Slots` writes an empty array and a record it produces carries no rotation data at all.
- A record written before the two layouts existed simply has no table, which reads the same way: nothing is turned.

`Memories` is the extension point: a typed struct you attach to an instance, saved and restored with it, that the plugin
never inspects.

⚠ `MaxNumSlotsAtSave` is recorded alongside the items. Loading a save made against a larger bag needs a decision, not a
silent truncation.

The equipment record also carries `UnlockedSlots` — the slots this character has opened with `UnlockSlot`.

- A slot opened by an **unlock tag** is never written: it is open exactly as long as its owner holds the tag.
  - Saving one would resurrect an unlock whose reason had since been lost.
- On load, unlocks are applied before the first worn line.
- Worn lines are restored whether or not their slot is open — see [equipment](equipment.md).

⭐⭐ **LOADING A SAVE NEVER COSTS THE PLAYER SOMETHING THEY WERE WEARING.** That is one rule with three parts, and each
exists because a record is written by an EARLIER build of a project's content:

- **Admission is not applied.** A worn line goes back into its slot whether or not the slot is locked, and whether or
  not something else worn now takes it out of use.
  - An item that gained a spanning entry after the player saved would otherwise silently shed whatever it now conflicts
    with.
- **Neither is displacement.** The bypass reaches the *consequences* as well as the gates.
  - So a record holding two items that span each other comes back with both worn rather than one of them turned out into
    the bag.
  - Nothing would have been lost either way, and that is not the claim — the claim is that what was worn is worn.
- **A line that genuinely cannot be worn lands in the bag**, over capacity if it has to.
  - Slot not in the set any more, no longer a fit, a fragment veto: the item comes back to the player rather than being
    reported and dropped.
  - The container then reports itself over capacity, exactly as it does when a save made against a larger bag is read,
    and the entry is drawn and draggable.
  - `FSnapImportReport` counts these in `WornLinesDivertedToBag`, which is deliberately **not** a refusal — nothing was
    denied, the item simply arrived somewhere else.

The state a load is allowed to create is therefore one no ordinary door will create, and it resolves itself the next
time the player equips anything into either slot.

⭐ **Adding `UnlockedSlots` needed no `SaveVersion` bump, and that does not generalize.**

- A record round-trips through UE tagged properties, which are keyed by name.
- So a file written before the field existed simply does not mention it and the array keeps its initializer.
- Here the empty initializer *is* the right answer, "nothing was explicitly unlocked".
- Add a field whose unwritten default is indistinguishable from a meaningful value and it needs a version and a
  migration instead.

## Versioning

Each record carries a `SaveVersion`, and `USnapPersistenceMigrator` upgrades older ones. A record from a **newer** build
than the one reading it is refused rather than half-read.

The one exception is the `Hotbar` store:

- A version 1 `FSnapHotbarRecord` is discarded rather than migrated, and the hotbar is saved normally from then on.
- See [Hotbar](hotbar.md#persistence).

⚠ Test the gate. A version check that is never exercised is indistinguishable from one that is missing — this plugin
shipped an inert one twice before it was measured.

## Automatic saving

```
EnableAutoSaveForActor(Actor, ProfileKey)
SaveActorInventory(Actor, ProfileKey)
HasSavedInventory(ProfileKey)     DeleteSavedInventory(ProfileKey)
```

- The auto-save binds to the component delegates and coalesces a burst of changes into one write.
- What is still waiting is written when the opted-in actor, the actor carrying its components, or the first of those
  components begins to end play, whichever comes first.
- `SaveActorInventory` writes nothing — and returns false with a Warning — when the actor carries no inventory or
  equipment component: it never replaces a profile with an empty record.

A typed change enum means a listener can tell a repaint from a real change: a hotbar repaint does not schedule a save,
while a binding change does.

## A container that keeps its own save

A container can keep its own record instead of riding the whole-inventory one.

1. Turn off `bSaveWithActorInventory` in its class defaults (a Blueprint subclass of `USnapInventoryComponent` or
   `USnapCustomContainerComponent`).
   - Then `ExportActorInventory` leaves it out.
   - `ImportActorInventory` skips any line that names it (with a Log line and a message in the report).
   - Its changes do not queue an auto-save.
2. Save it on your own schedule: `ExportRecord` into a store of its own through `SaveStore`.
3. Load it with `ImportRecord` after `LoadVersionedStore`.

⚠ One container, one record. Import is additive, so a container restored from two saves comes back twice.

- Every item the older copy still lists — sold, dropped or moved out since — returns as a second one.

The example demo's personal bank is the worked example (`SnapDemoBank` in `Demo/SnapDemoBank.h`):

- `BP_Example_BankContainer` keeps its own save, and admits every item (`AdmissionMode` AdmitAll).
- The bank is read once, when the player arrives (the demo game mode's `PostLogin`), through `LoadVersionedStore`. The
  four outcomes are taken literally:
  - Nothing stored means a fresh bank that may be written.
  - A store that cannot be read, or that a newer build wrote, is left alone and nothing is written over it that session.
- It is written when its window closes, after a deposit, and when the player's state ends play — before its components
  do, so the bank is still whole.
- Its record wraps the container's own record in a struct whose `SaveVersion` defaults to 0, with a separate
  `CurrentVersion` stamped on every save (see [Versioning](#versioning)).

## World state

`USnapWorldPersistenceSubsystem` saves what is lying around: dropped pickups, loot containers and bodies, so a reload
finds the world as it was left.

- Actors are keyed by an identity that combines the level instance with a `WorldId`, so two `ALevelInstance`s of one
  level do not share a record.
- ⚠ `AActor::ActorGuid` is editor-only and cannot be used for this.

**Observed regions** decide what a restore may remove:

- A save records which levels were loaded when it was taken.
- A live actor the save does not mention is removed only in one of those.
- A saved object whose level is not loaded now is skipped and counted (`RecordsSkippedRegionNotLoaded`).
- The plugin README's world persistence section has the whole rule.

## Window layout

Floating window positions are saved per profile, keyed by widget name, in the `WindowLayout` store
(`bPersistWindowLayout`, on by default). A project can keep them elsewhere: give
`USnapWindowManager::SetWindowLayoutStore` (C++) its own `ISnapWindowLayoutStore`.

- The craft window is keyed by a fixed name instead: `SnapCraftWindow`.
  - Each local player's craft window is its own widget, so split-screen players each keep their own window and position.
  - In split screen each player's bag screen, and every window drawn over it, is laid out in that player's own part of the
    screen.
  - There is one layout for every screen size and split. While the screen is split, a remembered place is shown inside the
    player's part (the whole window, as far as it fits) without changing what is saved; with one player it opens where it was
    left.
  - A window dragged while the screen is split is saved where it was put, so that is also where it opens with one player.

- A player added during play (split screen) keeps a layout of its own: the load waits until the engine has attached that
  player's local player, which comes after its controller's BeginPlay.
- The closed state is not saved: every window comes back when the screen opens.
- Pouch windows are not covered.
- See the README's [Optional: floating windows](../../README.md#optional-floating-windows) for when it is written.

## See also

- [Networking](networking.md) — what is authoritative
- [Extending the plugin](extending.md) — writing a provider
- [Appendix: settings](appendix-settings.md) — the **Persistence** category
