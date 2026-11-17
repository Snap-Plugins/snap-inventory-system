# Loot and interaction

Four ways an item gets from the world into a bag: a pickup actor, a world drop, a loot container, and a body.

**On this page:**

- [Pickups](#pickups)
- [Dropping](#dropping)
- [Loot containers](#loot-containers)
- [Bodies](#bodies)
- [The interaction system](#the-interaction-system)
- [The test spawner](#the-test-spawner)
- [See also](#see-also)

## Pickups

`ASnapPickupActor` holds a definition and a count (`Count`, 1 by default) and grants them on overlap. Code that creates
value (a loot table, the test spawner, the demo's **SPAWN EVERY ITEM** pad) rolls the `PickupAmount` fragment and sets the
count; a drop carries its exact count and never rolls.

A pickup carries the dropped instance's **identity** (`InstanceId`) and its rarity **variant**, so an item dropped and
picked up again is the same item, at the same tier, with the same wear — not a fresh roll.

⭐ **A placed pickup can choose its tier with the same field.** `Variant` is editable on any pickup in a level.

- Set it to one of your rarity table's tags and the item is collected at that tier, every time.
- Leave it empty for a fresh roll.
- A definition that pins its tier (`FixedRarity` on its Rarity fragment) outranks it, and a definition with no Rarity
  fragment ignores it.
  - Saving the level warns about both, because at runtime either is silent.
- An item with no model of its own shows the tier on its fallback shape; an item's own model is drawn as authored.
- The demo map's five helmets are one definition placed five times with five `Variant`s.

⭐ **The tier shows on the floor.** Every pickup whose tier is decided — a dropped item's own, a placed pickup's
`Variant`, a definition's `FixedRarity` — draws a flat glow under the item in the rarity table's color for it
(`GetGroundRarity`, the collect's own order).

- It is sized to the item's footprint, rides its toss and hover, and never covers it.
- ⭐ While the glow is on, a pickup whose tier is ROLLED (a Rarity fragment with no `FixedRarity`, no `Variant`) has it
  rolled by the server when it appears.
  - That is the rarity table's `RollRarity`, the very roll the collect made, so only the moment moves.
  - It is rolled again on a refill and after each collect of a pickup that is not destroyed when collected
    (`RollGroundTierIfUndecided`).
  - It glows, and you collect the tier you saw.
- A tier the pickup was placed with is kept, and so is one the world memory remembered.
- With the glow off it rolls at the collect as before and shows none; an item with no Rarity fragment shows none.
- One project switch, `bShowPickupRarityGlow` (on).
- **The look:** A soft light in the tier's color around the item and a faint rim on it (`RarityLightIntensity`,
  `RarityLightRadiusScale`, `RarityHaloMaterial`).
  - With `bPickupRarityGlowAsLight` off it is the flat disc under the item (`RarityGlowMesh` / `RarityGlowMaterial`).
- All of it is in `/SnapInventorySystem/Core/Effects` — the README's one `/SnapInventorySystem/Core` cook line covers it (its Required setup).

A capacity refusal leaves the item on the ground and **re-offers it where the player stands** as soon as their inventory
changes.

- Each inventory component re-tests the pickups its owner is overlapping whenever its own contents change, so freeing a
  slot takes the item without stepping off and back on.
- The re-test runs on the next tick rather than inside the change that triggered it.
- It goes through the same mode, filter and collect rules a walk-on does — there is no second collect path.

⚠ Only a pickup whose **last collect granted nothing** is re-offered.

- A pickup with `bDestroyOnPickup` cleared, which is collected over and over by design, is therefore not emptied into
  the player's inventory once per tick while they stand on it.
- A pile the bag could take only part of (it takes what fits — [Pickup modes](pickup-modes-and-the-collect-feed.md))
  granted something, so its rest is not re-offered either: stepping off and back on takes more.

⚠ Units are conserved on a drop-and-recollect; **slot occupancy is not**. A pile of six units of a `MaxStackSize = 2`
item comes back as several stacks rather than one.

`PickupLifespanSeconds` controls how long a dropped pickup survives.

⭐ **A pickup can be knocked across the floor**, by an explosion say: `KnockAwayFrom(Origin, Distance)` on the server.

- It flies along the drop's toss and lands where the drop placement's own tests pass: the open path from where it lay,
  walkable ground in reach under the spot, the spot clear of geometry, all on `DropTraceChannel`.
- A blocked path cuts it short in front of the obstacle, so it never goes through a wall.
- It stays the same pickup, collectable throughout, and every client plays the same flight.
- A refilling placed pickup comes back where it was placed.

A thrown pickup with `bCollectOnlyAfterLanding` set cannot be walked into until its toss has landed; whoever is standing
on it then is offered it.

- A breakable container's drops set it.
- The inventory's own world drop leaves it off, because its placement already keeps the dropper out of reach.

## Dropping

```
Server_DropItemsToWorld(...)                drop items (the DROP ALL gesture sends every stack here)
Server_DropCounterToWorld(CounterTag, N)    drop currency (and the DROP ALL gesture's currency)
DropEverythingToWorld()                     the scatter step of a death (server only; see Death and loss)
```

Dragging an item out of a window and releasing over the world drops it. A drop is refused with a named reason when it
cannot be placed.

⭐ **A player's drop of more than `DropAllLootBagThreshold` (20) whole stacks at once goes into ONE loot bag** just ahead
of their feet — DROP ALL, or a multi-item drop.

- It is opened and emptied like a death's body (a loot window with Take All), instead of scattering as separate pickups.
- Pouches go in whole.
- The bag is `DropAllLootBagClass`, or `LootableBodyClass` when that is unset.
  - It is widened for the drop when its class holds fewer stacks.
  - It never takes a death's ragdoll look.
  - It lives by the body's rules (`LootableBodyLifetimeSeconds`, `bRemoveEmptiedLootableBodies`).
- Whatever it cannot take scatters as before; 0 turns the bag off.
- A death's drop and a body's despawn scatter never make one.

A drop of more than `MaxThrownStacksPerDrop` (8) stacks at once — a DROP ALL that scatters, a death's scatter — is
PLACED.

- No physics impulse, so a hundred bodies do not fly into each other (the plugin pickup's drawn throw below still
  plays).
- Once the drop ring is full, each further item tries `CrowdedDropPlacementAttempts` (3) spots and keeps the best,
  instead of `DropPlacementAttempts`.

A dropped item is thrown, not placed: the pickup stands at its rest point from the start and only its mesh flies.

- It flies from the thrower's hands (`ThrowOriginHeight` above them) along a low arc (`ThrowArcHeight`, 45 cm) to a spot
  `DropMinDistance` to `DropMaxDistance` away (160-250 cm).
- It takes `ThrowMinFlightTime` to `ThrowMaxFlightTime` (0.5-0.7 s) by distance.
- The toss is eased: it leaves the hands and arrives with no sideways speed, so it lands exactly on its rest point.
- The hover starts there, blended in over `HoverBlendInSeconds` (0.5 s), so it rises out of the landing rather than
  jumping.
- `SettleSlideDistance` (0 as shipped) brings back a skid past the spot and back, with the hover waiting for it.
- A breakable container's drops and a death's currency are thrown the same way.
- `DropMinDistance` is the dropper's protection: a pickup's 100 cm sphere plus a character's capsule is about 142 cm, so
  the player who dropped an item does not stand in it.

⚠ Dropping a **pouch** drops its contents with it.

- The failure mode the code guards against is the pouch landing in the world while its contents stay in the bag, which
  is item loss.
- So that case is never silent.
- **Picking the pouch up puts its contents back in it.** Anything the pouch can no longer take back stays on the ground
  as a pickup of its own, as the rest of a pile does.
  - For example: an item that belongs in a different container, a nested pouch past `MaxStorageNestingDepth`, or more
    than fits.

## Loot containers

`ASnapLootContainer` is a chest or a crate: a container actor a player opens, sees the contents of, and takes from.
`USnapLootInventoryComponent` is the container behind it.

Two flags apply to a chest, since a crate holds no contents:

| Flag | Default | Effect |
|---|---|---|
| `bRepopulateWhenEmpty` | On | Skip the restock while anything is still inside |
| `bClearOnRespawn` | Off | Destroy everything in it first, *including what a player put there* |

`bClearOnRespawn` is an item-loss door and logs a warning naming the count when it fires.

A lootable body placed in the level restocks the same way when its `SpawnPolicy` is set; a body a death dropped never
does.

A pickup placed in the level can come back too:

- Its `RespawnSeconds` (0 = never, the default) hides it when the last unit is taken and brings it back holding what it
  was placed with.
- Only a placed pickup carrying nothing of its own comes back, so a dropped item is never duplicated.
- See the README's "Pickups can come back".

`USnapLootInventoryComponent` ships `AdmitAll`, so a container takes anything that is not a counter.

A container shows one of three looks, all driven by replicated state:

- **Closed:** Its own mesh.
- **Opened:** `OpenedMesh`, or a tint when none is set, while anything is left inside.
- **Emptied:** `EmptiedMesh`, once it holds nothing.

Putting something back shows the opened look again, and a respawn closes it. Leave `EmptiedMesh` empty to keep the
opened look. `IsEmptied()` reports the state.

**Hearing about it:**

- `USnapLootInventoryComponent::OnOpenedChanged` is a toggle, not a one-way event.
  - It fires `true` the first time anyone opens the container (`MarkOpened`, once however many players open it).
  - It fires `false` when a respawn closes it again (`ClearOpened`).
  - Read the bool in a handler that plays an opening sound or effect, or it also plays when the chest closes.
- `HasBeenOpenedSinceRestock()` is the flag the hold-to-open rule reads. A skipped refill closes the lid but does not
  clear it.
- `LootRange` 0 makes every container un-lootable, even with the pawn standing on it. A container is offered only within
  both `LootRange` and `InteractionScanRange`, so change them together.

A container can also **throw its contents on the ground** instead of opening a window.

- Set its loot component's `Presentation` to `SpillToWorld` (or the project's `DefaultLootPresentation`).
- Opening it then scatters every item as a pickup beside it, through the server, and shows the opened and emptied looks.

The example `BP_Example_ScatterChest` is that chest opened by a one-second HOLD of the interact key:

- Its option's `HoldSeconds` is 1.
- Its loot component's `DropAngleSpreadDegrees` is 180, so the items land in a ring all around it rather than in a fan
  in front.
- Its class defaults carry the chest's `SpawnPolicy` too (`RespawnSeconds` 60), so a minute after it is spilled it
  refills and closes again.
- Once it has spilled everything it **offers nothing** — no outline, no prompt, no hold — until it holds something
  again.
  - `USnapLootInventoryComponent::IsSpentSpill()` (it spills, it has been opened, it is empty) is refused by the
    interaction, never by `CanBeLootedBy`, so the takes and the range watch are unchanged.

A container that opens a window keeps prompting when empty, so the player can see it is empty.

### The loot window

Opening a chest or a body shows what is inside in the loot window (`LootWindowClass`; the example is
`WBP_ExampleLootWindow`) **and opens the bag screen with it** (`bLootWindowOpensInventory`, on).

- So the inventory and the equipment are on screen while the player takes things.

**Where it sits:**

- **With windows fixed**, the loot window joins the bag screen's row after the bag's column
  (`USnapInventoryScreenWidget::LootWindowDockRowName`, default `Row`; a screen without that row gets the window on the
  window layer).
  - The three re-center as one group.
  - On a screen too narrow for them — 16:10, 4:3 — the group is scaled down to fit.
- **With windows floating** (`bAllowWindowDragAndClose`), it opens beside the bag panel: to its right, else below it,
  else to its left, else above it.
  - It takes the first of those that covers no other window, else the one that covers least. It is centered when the
    bag isn't drawn.
  - Later opens keep where it was dragged.
- `IsLootWindowDocked()` says whether it sits in the row.

**Closing:** The two close together.

- Escape, the toggle key, the close box and walking out of range close both.
- A bag that was already open when the loot window opened (a project's own `OpenLootWindow` call — the interact key is
  not live while the bag is up) stays open.
- Taking the last item leaves the window open on its "Empty" line, Take All disabled, until it is closed or the player
  walks away. The interact key stays off while it is up, as with any open screen.
  - On a pad, focus moves to its close box, so A closes the two.
  - Another player emptying the container leaves your window open too.
  - Set `bCloseWhenEmptied` on the loot window class to close the window, and the bag it opened, when the last item is
    taken. A window opened on a container that is already empty stays open either way.
- A chest that refills while its emptied window is open lists the new items there. Its lid shows closed until the first
  take opens it again.

**What it draws:** One square per item still inside, in the order the container holds them, closing up as items are
taken.

- Six squares wide (`SlotsPerRow`), five rows showing (`MaxVisibleRows`); a longer container scrolls inside the window.
- Nothing can be put back from it, so it draws no empty squares.
- A container that holds nothing shows its "Empty" line (`EmptyLabelText`) instead.

**Taking things:**

- **TAKE ALL** takes what fits and leaves the rest — with a full bag, topping up the bag's stacks of an item with room,
  as Shift+click does.
  - It says so once: "Not everything could fit." when part of it was taken, "Your inventory is full." when none of it
    was.
  - With `bAutoEquipOnPickup` on (the shipped default) it first puts on whatever can be worn in an EMPTY slot it fits,
    so a player looting their own body gets their gear back on.
  - It never replaces a worn item and never takes one off (an item that would, a two-handed sword with something in the
    off hand, goes to the bag).
- **Shift+click** takes one item.
  - With a full bag it tops up the bag's stacks of the same item that have room, in square order, and the rest stays in
    the chest.
  - Nothing is said when part of it moved; "Your inventory is full." when none did.
  - X and A with the carry off on a pad are the same take.
- **A selection** works as it does in the bag.
- **A drag** carries items into the bag, a pouch or an equipment slot — or onto the ground when it is let go outside
  every window.

**Where a dragged item lands in the bag:** On the square or grid cell it is let go on
(`USnapInventoryComponent::TakeFromLootContainerToSlot`).

- A free square takes it.
- The same stackable item takes the units.
- A different item gives up its square and moves to the first free one.
- A full bag is painted red and nothing is sent — the take's own notice says "Your inventory is full."
  - Except over a stack of the same item with room, which takes the units that fit straight from the chest while the
    rest stays in it.
- A selection puts the item the drag began on there and the rest on the first free squares.
- An item that belongs in another of the player's containers goes to the first free square there.
- Nothing goes into the container: a chest item let go on a pouch, a damaged item or a mix partner in the bag changes
  places with it rather than going in, being repaired or mixed.
- Ctrl held over a bag square still takes the whole item: there is no counted take out of a container.
- On a gamepad, A picks a chest item up and puts it down the same way ([Input](input.md)).

**Other places a dragged item can land:**

- **A pouch window's square or an equipment slot** is two requests: the take into the bag, then the store or the equip.
  - Both are reliable and go through the same actor, so they arrive in order.
  - If the second step is refused, the item is simply in the bag (nothing is lost or duplicated), and the player
    finishes the move by hand.
- **A selection dropped on an equipment slot:** Every item is taken, and only the item the drag began on is worn; the
  rest land in the bag.
- **A window's frame** (the bag and equipment panels, a pouch window, the craft window) or a hotbar square leaves the
  item in the container. Only a release outside every window drops it on the floor.

A square's position is not the container slot its item sits in: square 0 is the first item still inside, wherever it
sits.

- `USnapInventoryLootSlotWidget::GetViewIndex` is the position and `GetSourceSlotIndex` the slot.
- The selection, a Shift-range's anchor and a drag follow their items, so another player taking an earlier item moves
  the squares without changing what this player picked.

`ASnapDestructibleContainer` is the breakable variant. When it breaks:

- Each drop is tossed from above it to where it lands (`DropFlightTime`, the pickup's own throw arc).
- The drops fan out on the side away from whoever broke it, at least `MinDropDistanceFromBreaker` from them.
- A drop in the air cannot be walked into.
- Set both to 0 and each drop appears where it lands, collectable at once.
- With `BreakPieces` set it also bursts into pieces of its own model that tumble and shrink away — on each machine that
  sees the break, touching nothing (see the README).

What either one may contain is a shared asset — see [Loot tables](loot-tables.md).

- Both roll through `USnapLootTableLibrary`, from a seed, so the same table and the same seed produce the same
  contents.

## Bodies

A dead player's inventory can become a lootable body.

- Because the loot component admits everything, a body inherits routed items that a stricter container would have
  refused.

`IsPlacedInLevel` tells a body placed in a level from one spawned while the game runs.

- A spawned body with no `LootTable` starts empty quietly — its contents come from the death — while a placed one
  without a table warns, like a chest.
- With `bRemoveEmptiedLootableBodies` on, a spawned body goes once it is emptied and no player is left in loot range (an
  emptied window stays open on it); a placed one never does.
- A placed one never times out either (`LootableBodyLifetimeSeconds` counts down only a body spawned while the game
  runs).
- See [Death and loss](death-and-loss.md).

`CanBeLootedBy` decides who may open one.

- It is asked at the PlayerState level as well as the pawn level, because a viewer between pawns still has an identity.

## The interaction system

`USnapInteractionDetectorComponent` on the PlayerController scans for the nearest interactable and raises a prompt;
`USnapInteractableComponent` on the actor declares what interacting does.

The prompt names the key that performs the interaction: **"(E) Forge"**.

- The key is read live from Enhanced Input for `InteractInputAction` (or the prompt widget's own `InteractAction`, when
  a themed subclass sets one), so a rebind shows the new key.
- A player whose last input came from a gamepad is shown the gamepad button — or the device `GlyphDevicePreference`
  forces, as on the hotbar squares.
- When nothing is mapped there is no prefix at all.
  - A project whose own input context takes the key from that action sees the bare label, because naming a key that
    would not reach the plugin would be wrong.
- The `bShowInteractKeyInPrompt` setting turns the prefix off.
- `InteractPromptKeyPattern` (`({Key}) {Label}`) changes how the two are put together; a pattern with no `{Label}` is
  not obeyed.
- The raw `InteractKey` is named only while `bBindInteractKeyRaw` is on and the action has no key.

**The prompt stays on screen over a tall object.** It is drawn just above the object's top.

- When that would leave the top of the player's view (at a market stall's counter, or looking at the middle of
  something tall), it slides down the object toward its middle, as high as it still fits, and stays on screen. In split
  screen it stays in that player's part of the screen.
- An object of ordinary height never moves: its prompt already fits.
- Only where it is drawn moves. Reach, the view cone and line of sight still use the top (`GetInteractionPromptAnchor`).
- `bKeepInteractionPromptOnScreen` (on) switches it, and `InteractionPromptScreenMargin` (24) is how far inside the edge
  it stays.
- A prompt class of your own gets it too: the detector calls `SetPromptSlideTarget` with the object's middle
  (`GetInteractionAimPoint`) after `SetPromptWorldLocation`.

**A project with its own interaction system.** When your project's input takes `E` for its own interact action, or
`bAddInventoryMappingContext` is off, the plugin's `IA_Interact` can be left with no key.

- The plugin's pickups, chests, bodies and stations still raise their prompt, but it names no key, and your key does
  nothing to them.

Do one of three things:

1. Map a key to the plugin's action in a context of your own.
2. Set `InteractInputAction` to your own interact action, so the detector binds it and the prompt names its key.
   - Its mapping must carry no trigger, or every hold ends as it starts — *Interact must be holdable* in
     [Input](input.md).
3. Call `TryInteract` on the player's `USnapInteractionDetectorComponent` from your own interact handler.
   - For an option that asks for a hold, call `BeginInteractHold` on the press and `EndInteractHold` on the release
     instead (see *Holding to interact* below).

```
Options                  everything this interactable offers, in order
OnInteracted             fires on the interactable
OnInteractionOptionPerformed   fires with the option that was performed
OnFocusChanged           fires on the detector when the focus moves
```

Each option is a verb, an enabled flag, a prompt override, an identity tag, a hold time and, for a hold, an optional tap
action (see [One key, tap or hold](#one-key-tap-or-hold)).

- An actor can carry several — a blacksmith is `[ { OpenVendor }, { OpenCraftingStation } ]`.
- An empty list offers nothing, which is a way to switch a component off without touching `bInteractionEnabled`.

### Picking an option

An option is **available** to a player when it is enabled and its own gate passes (a shop with stock, a station with
something to offer, a pickup with somewhere to land). Everything a player or a Blueprint sees counts available options
only, in authored order:

- With more than one available, the prompt reads **"Shop 1/3"**. One option shows no count.
- The key performs the **selected** option, which is the one the prompt names.
  - The selection starts on the first available option and resets every time the focus changes.
  - It survives the scan re-finding the same target.
- `InteractCycleInputAction` (`IA_InteractCycle`) steps the selection forward and wraps.
  - It ships **unmapped** — see [input](input.md).
  - Once a key is mapped to it, the prompt's count names that key: **"Shop 1/3 (Tab)"**.
  - `CycleInteractionOption(Step)` on the detector does the same from anything.
- If the selected option stops being available, the selection moves to the next one that is.

```
GetAvailableInteractionOptions()    detector: what the focused target offers this player
GetSelectedInteractionOptionIndex() detector: which of those is selected
TryInteractOption(Index)            detector: perform option Index now, without moving the selection
CycleInteractionOption(Step)        detector: move the selection
GetAvailableOptionsFor(Viewer)      interactable: the same list, for any viewer
PerformInteractionOption(Viewer, Index)  interactable: perform option Index for Viewer
```

⚠ `Index` is a position in the **available** list, never an index into `Options`.

- The two agree only while every option is available.
- `FindAvailableOptionIndexAt(Viewer, Index)` is the one place a position becomes a row — an `Options` row, or one of
  the actor's own options after them (see [An actor's own options](#an-actors-own-options)).
- An index that names no available option is refused with a log line and nothing happens.

⚠ `PerformInteraction(Viewer)` on the interactable performs position 0.

- The selection belongs to the viewer's detector, not to the interactable, so a button wired to `PerformInteraction`
  ignores what the player picked.
- Call the detector's `TryInteract` for the selected option.

### Holding to interact

An option with `HoldSeconds` above 0 needs the interact input **held** that long; 0 (the default) performs on the press.

- Its prompt says so — **"Hold (E) Open"**.
- While the key is held a **ring** floats over the target's middle and fills clockwise.
- The ring appears only once the key has been held `InteractHoldIndicatorDelaySeconds` (0.2 s), already filled to the
  elapsed fraction, so an ordinary press never flashes one.
- Let go before the hold completes and it winds back to empty, then hides.

The ring is its own widget, `USnapInteractionHoldWidget`, built by `InteractHoldWidgetClass`.

- `WBP_ExampleInteractionHoldRing` is the same ring as a Widget Blueprint.
- A themed copy needs a CanvasPanel root holding a `USnapHoldRing` named `InteractionHoldRing`.
- `InteractHoldMeter` set to `Bar` draws the thin bar under the prompt's words instead (a ProgressBar named
  `InteractionPromptProgress` in a themed prompt), after the same delay.
- Since 1.0.0 the prompt draws no ring of its own: a themed prompt's `InteractionPromptRing` part is kept hidden.
- `USnapHoldRing` is a native widget with no asset, usable in any widget of your own.

A hold performs nothing when any of these happens first:

- The input is released.
- The focus moves.
- The selection moves.
- The option stops being available.
- Interaction is suppressed — one of the plugin's screens opens, the player dies, or something pushes a suppression (see
  [Switching interaction off](#switching-interaction-off)).

When it completes, the option is performed exactly as a press would perform it.

The plugin's own input drives holds:

- `InteractInputAction` is bound on `Started` (begin) and `Completed` / `Canceled` (end), and the raw key binds press
  and release.
- A project driving interaction from its own input calls `BeginInteractHold()` and `EndInteractHold()` on the detector.
- `TryInteract`, `TryInteractOption` and `PerformInteraction` ignore `HoldSeconds` and perform at once — a Blueprint
  that calls them has already decided.

**A hold to open a loot container is asked once.** A loot option with a hold (verb Open Loot Container, no tap action)
is a plain press, for every player, once anyone has opened the container — until it is restocked.

- A refill from its loot table (`ASnapLootContainer::RespawnNow` restocking it) asks for the hold again.
- A refill that finds something still inside skips the restock and keeps it a press.
- A body, which never restocks, stays a press.
- A project that refills a container its own way calls `USnapLootInventoryComponent::MarkRestocked()` after.
- Switch it off per container with `bHoldOnlyUntilOpened` (a sealed vault that should always ask).
- The prompt and the hold read the same rule (`USnapInteractableComponent::ResolveHoldSeconds`);
  `HasBeenOpenedSinceRestock()`, `IsOpeningHoldWaived()` and the detector's `GetFocusedOptionHoldSeconds()` (0 = a
  press, -1 = nothing on offer) read it.
- It is not saved: after a world reload an opened container asks for its hold once more.

### One key, tap or hold

A hold option can also carry a **tap action**: set `bHasTapAction`.

- A short press — the input let go within `InteractTapMaxSeconds` (0.2 s) — performs the tap action.
- A full hold performs the option.
- A hold let go in between performs **neither**: the ring winds back and nothing happens.
- On such an option the ring appears only once the tap window has closed, so a release after it shows can never tap.

One option then offers both, and the prompt names both on one line with no count and no cycle key:
**"(E) Open your bank  ·  Hold (E) Deposit everything"**.

- The joining pattern is keyed text, so a translation may reorder it.
- Each half follows `InteractPromptKeyPattern`.

The tap action is the option with `TapOptionTag` and `TapPromptText` in place of its own `OptionTag` and `PromptText`
(`MakeTapVariant()`): the same verb, through the same door, raising the same delegates.

- So it is a `None` option's feature in practice — a built-in verb would do the same thing either way.
- `TapOptionTag` is how a handler of `OnInteractionOptionPerformed` tells a tap from the hold.
- An actor's own option with a tap action reaches `GetSuppliedOptionPromptText` and `PerformSuppliedOption` in that tap
  form.

A tap is performed only when nothing that would cancel a hold happened first (the focus, the selection or the option's
availability changed, or interaction was suppressed).

- Let go after the full time but before the hold's timer fired, and the option itself is performed.
- An option **without** a tap action still performs nothing on an early release.
- To have any release before the hold completes perform the tap, raise `InteractTapMaxSeconds` toward the hold's length.

```
TryInteractOptionTap(Index)                 detector: perform option Index's tap action now
GetTapPromptTextForOption(Viewer, Index)    interactable: the tap action's words; empty when it has none
PerformInteractionOptionTap(Viewer, Index)  interactable: perform option Index's tap action for Viewer
```

⚠ **A hold is timed on the player's machine and is not a security boundary.**

- Nothing on the server checks how long the key was held, so a modified client can skip the wait.
- Use holds for feel and to guard against an accidental press, never to make something cost time.

What the server does re-check, when a completed hold (or a press) performs an option:

| Option | Re-checked on the server by |
|---|---|
| `OpenVendor` | `Server_BeginVendorSessionAt` — an enabled `OpenVendor` option on that component, and reach |
| `OpenCraftingStation` | `Server_BeginStationSession` — an enabled `OpenCraftingStation` option, and reach |
| `PickUpItem` | `Server_TryPickUpItem` — the pickup's collect mode, and reach |
| `OpenLootContainer` | The loot requests — `CanBeLootedBy` |
| `TradeWithPlayer` | `Server_RequestTradeWith` |
| `None` | Nothing: `OnInteracted` runs on the player's machine, so a project's own server request must re-check |

### Switching interaction off

Interaction is **suppressed** while one of the plugin's own screens is open or the player is dead:

- The focus clears (the prompt hides, the outline comes off, `OnFocusChanged` fires with null).
- The key and the cycle action refuse.
- A hold in progress ends.

A project's own screens switch it off the same way:

```
PushInteractionSuppression(Source)   detector: suppress for as long as Source wants it
PopInteractionSuppression(Source)    detector: release Source's suppression
IsInteractionSuppressed()            detector: the plugin's own check, or any live source
```

- `Source` is any object — the pause menu widget, the cutscene actor. It is held **weak**: destroy it without popping
  and interaction comes back on its own, within one scan.
- A second push from the same source does not stack; one pop releases it.
- The plugin's own check keeps working alongside: a bag that is open still suppresses whatever is pushed.
  - A controller without a `USnapInventoryHUDComponent` has none of the plugin's screens, so there only pushed sources
    suppress.
- The refusal line the key logs at Verbose names what is suppressing it.

### An actor's own options

An actor that carries a `USnapInteractableComponent` can also implement `ISnapInteractable` to offer options of its own,
without subclassing the component. The component lists its authored `Options` first, then the options the actor returns:

```
GetSuppliedInteractionOptions(Viewer)         what the actor offers this player
CanPerformSuppliedOption(Option, Viewer)      whether one is available     (default: its bEnabled)
GetSuppliedOptionPromptText(Option, Viewer)   what the prompt reads        (default: its PromptText)
PerformSuppliedOption(Option, Viewer)         do it; true fires the events (default: true)
```

- ⚠ **The component is still required.** The detector finds candidates in a list the component joins, so an actor that
  implements the interface and carries no component is never focused.
- ⚠ **An actor's options use the `None` verb.** The built-in verbs read their payload from the component — the stock,
  the station tags — so an actor option naming one is dropped, with one warning per actor class.
  - Tell the actor's options apart with `OptionTag`.
- An actor's options are counted, cycled and held exactly like authored ones, and `OnInteracted` /
  `OnInteractionOptionPerformed` fire for them.
  - The component's re-entry guard covers them too: calling back into `PerformInteraction` from inside
    `PerformSuppliedOption` is refused.
- `PerformSuppliedOption` runs on the player's machine, like a `None` option: a server request of your own has to
  re-check.
- From C++, call through `ISnapInteractable::Execute_...` rather than the method: a Blueprint implementation has no C++
  body to call.
- **An actor's option may carry a tap action** (`bHasTapAction`; see [One key, tap or hold](#one-key-tap-or-hold)).
  - Its tap is worded and performed through the same two functions, `GetSuppliedOptionPromptText` and
    `PerformSuppliedOption`.
  - They are handed the option's tap form (`FSnapInteractionOption::MakeTapVariant`: `TapOptionTag` and `TapPromptText`
    in place of the option's own), so the actor tells a tap from a hold by the tag.

### A subclass's own options

A subclass of `USnapInteractableComponent` can offer options of its own, per player, without the actor implementing
anything. Override the extra-option seam:

- `GatherExtraOptions(Viewer, Out)` — the options for this player right now.
- `IsExtraOption` — tell yours apart by `OptionTag`.
- `CanPerformExtraOption`, `GetExtraOptionPromptText` and `PerformExtraOption`.
  - `PerformExtraOption` runs on the player's machine: send a server request of your own, and re-check there.

They come after the authored options and the actor's, use the `None` verb, and go through every gate an authored option
does — reach, the prompt, the press and its re-entry guard.

- `USnapQuestGiverComponent` is the shipped example: its "Turn in" exists only for a player who could turn a quest in
  ([Quest givers](quest-givers.md)).
- `USnapTradePartnerComponent` is the other: its "Trade with <name>" opens the trade window with the actor as the other
  side ([Trading with NPCs](npc-trade.md)).

The detector reads every candidate through the same interface — whether anything is on offer, the prompt's words and
where it goes, the reach and the view cone, and performing an option.

- So a Blueprint subclass of the component can answer those differently.
- The reach and the cone are asked only when the component's own `InteractionRangeOverride` / `MaxAngleDegreesOverride`
  is set, because the scan works out the defaults once for every candidate.
- The outline and the player's pick among the options stay on the component.

Reach and view cone are per-instance: `InteractionRangeOverride` and `MaxAngleDegreesOverride`, each `-1` to use the
plugin settings.

- Every reader goes through `GetEffectiveInteractionRange()` and `GetEffectiveMaxAngleDegrees()`.
- So the distance the prompt appears at is the distance the shop closes at and the distance the server will accept.

If the line of sight to the best-aimed interactable is blocked, the scan falls through to the next-best one that can be
seen rather than offering nothing.

The focused actor gets an outline, drawn by a post-process material named by `InteractionOutlineMaterial`.

⚠ It needs **Custom Depth-Stencil Pass** = **Enabled with Stencil**: `r.CustomDepth=3` under
`[/Script/Engine.RendererSettings]` in your `DefaultEngine.ini`.

- The outline reads the custom-depth stencil, and the renderer clears and writes that stencil only at 3.
- At any other value — the engine default is 1 — the detector installs no outline pass (nothing is loaded or spawned),
  turns an installed one off if the setting drops below 3, and logs one line naming the setting.
- The prompt still works.
- The editor's **Setup Check** warns about it at editor start (`OutlineStencilNotWritten`).
- The example project the generator writes sets it.

⚠ That material lives in `/SnapInventorySystem/Core/Effects` and is **not example content** — it is needed even by a project
that ships none of the examples.

- It has no referencer, so it needs a cook rule or it is simply absent from a package while working perfectly in the
  editor.
- Clearing `InteractionOutlineMaterial` is the deliberate "no outline" choice.

With `bInteractionOutlineEnabled` off, the material is not loaded and no post-process volume is spawned. Switching it on
mid-session installs the outline pass on the detector's next scan.

The interact key is `E` through `IA_Interact`. See [input](input.md).

### How the focus is chosen

The rules, for debugging why something does or doesn't take focus:

- **The candidates are every registered interactable in the world**, not a physics query's result.
  - So the scan's cost grows with the number of interactables. That is cheaper than a physics query at the tens to
    hundreds it is designed for; a project with tens of thousands should partition them.
- **Range is measured from the viewer's pawn**, not the camera, which in third person sits far behind it.
- **The view cone's apex is the pawn too**; only its direction is the camera's. From the camera, a target behind the
  player would score a near-perfect hit.
- **The cone is measured to the target's aim point** (its middle), not to the prompt anchor that floats above it.
- **A target at least as far below the viewer as beside them** (the height difference at least the horizontal
  distance) skips the cone, so the crate underfoot can focus.
  - It keeps its real score, so anything the player faces wins over it.
  - The bound is the scan range, so a larger `InteractionScanRange` lets "below" reach further.
- **The cone is widened by the angle the target subtends**: `atan(radius / distance)`, the radius from
  `GetInteractionBoundsRadius`.
  - The effective half-angle is capped at 90 degrees, and the widening never tightens a configured cone of 90 or more.
  - So up close, inside a standard character's bounds (about 62 units, against the shipped 35 degrees), even a sideways
    look focuses them.
- **An interactable at the viewer's own position** (one on the player's own pawn) is scored last.
- **Survivors are sorted by score** (stable, so two equal crates don't flicker), and the first with a clear line of sight
  wins. A blocked best target falls through to the next visible one.
- **The log says why, at Verbose:** For each candidate the plain cone rejected, whether the widening rescued it and with
  what numbers; when a target has no bounds to widen by; and, for a blocked trace, what blocked it.

### Your own outline material

The outline is drawn by `InteractionOutlineMaterial` on an unbound, transient post-process volume the detector spawns,
enabled only while something is focused.

- **It needs `r.CustomDepth=3`.** At any other value the detector installs no pass and logs one line.
- **The plugin sets three parameters**, matched by exact name. A name the material lacks is silently ignored, so a typo
  shows as a wrong color or no outline.

| Parameter | Type | Set from |
|---|---|---|
| `StencilValue` | Scalar | `InteractionOutlineStencilValue` (250, clamped 0 to 255) |
| `OutlineColor` | Vector | `InteractionOutlineColor`, through the palette's `InteractionOutline` role |
| `OutlineThickness` | Scalar | `InteractionOutlineThickness` (2) |

- **The outline covers the owner's own mesh components only.** Meshes on child actors (`UChildActorComponent`) are not
  outlined.
- **The focus outline borrows the custom-depth stencil**, which a project may already use (a team system). Each mesh's
  `bRenderCustomDepth` and `CustomDepthStencilValue` are captured once, before the first write, and restored exactly.
- **The material is told one stencil number**, `InteractionOutlineStencilValue`.
  - An actor that sets its own `OutlineStencilValue` is not outlined by the plugin's pass; only a pass of yours that
    reads that value outlines it.
- **A project whose own post-process already outlines that stencil** shows two outlines, composited (two full-screen
  passes). Clear `InteractionOutlineMaterial` to keep only yours.

## The test spawner

`USnapTestPickupSpawner` is a **development aid** and ships disabled.

1. Turn it on with **Project Settings > Plugins > Snap Inventory System - World > Examples > Enable Example Item Spawner**.
   - That binds a key which spawns a ring of every item definition in a folder.
2. Untick it before packaging — it is a cheat key.

A Shipping build never creates the spawner, whatever the tickbox says; a Development or Test build does.

**What a press does:** It clears the previous ring and spawns a fresh one around you.

- It also tops up a craft fixture in your bag (`GrantCraftingTestFixture`): a pouch holding a spoilable item, plus what
  the recipes `Recipe.GreavesWithCurrency` and `Recipe.ChestFromSwordGloves` need.
- The pre-filled containers are ground pickups that grant a pouch already holding items, on an inner ring.
- It runs on the server only. A client spawns nothing and binds no key.

**Its other values** are config on the class. Set them in your own `DefaultGame.ini` under
`[/Script/SnapInventorySystem.SnapTestPickupSpawner]`.

- They are read when the class loads, so an edit needs an editor restart. The tickbox is read live.
- Write `SpawnKey` bare (`SpawnKey=L`), as every key in config is written (see "A key that names nothing" in
  [The setup check](editor-tools.md#the-setup-check)).

| Value | Default | What it does |
|---|---|---|
| `bAutoSpawnOnStart` | `false` | Spawns a ring by itself, once, shortly after a pawn is possessed |
| `SpawnKey` | `L` | The key that spawns a ring; `None` turns it off |
| `CopiesPerDefinition` | 2 | Pickups per item definition |
| `CounterItemCopies` | 6 | Copies of a counter item (a currency) instead |
| `ItemFolder` | `/SnapInventorySystem/Examples/Items` | The folder it scans, not recursively |
| `RingRadius` | 500 | The ring's radius, in cm |
| `FilledContainerCopies` | 2 | Pre-filled containers (0 leaves them out) |
| `FilledContainerRingRadius` | 220 | The inner ring's radius, in cm |
| `FilledContainerItemCount` | 3 | Items in each, at most one less than its capacity |
| `SpawnHeightOffset` | 60 | How far above your footing a pickup starts before it is put on the ground |
| `GroundTraceDistance` | 2000 | How far down it looks for ground |
| `StackCountPerPickup` | 1 | Units per pickup; an item with `USnapItemFragment_PickupAmount` rolls its own amount |

## See also

- [Inventory and containers](inventory-and-containers.md) — admission modes
- [Persistence](persistence.md) — pickups and containers surviving a save
- [Appendix: settings](appendix-settings.md) — the **Loot** category (split over the **Snap Inventory System - Inventory** and **Snap Inventory System - World** pages),
  **Interaction** (**Snap Inventory System - Interaction**) and **World Drops** (**Snap Inventory System - Inventory**)
