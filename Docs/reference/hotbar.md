# Hotbar

`USnapHotbarComponent` holds bars of slots that **reference** items rather than holding them. This page covers how bars are
defined, what may be bound, binding and moving squares, pressing a key, the selectable hotbar, cooldowns and charges,
pages, events, saving, and placing and restyling a bar.

- **Binding an item to a square does not move it.** The item stays in the bag and the square points at it.

**On this page:**

- [Bars are data](#bars-are-data)
- [What may be bound](#what-may-be-bound)
- [Binding](#binding)
- [Moving a square on the bag screen](#moving-a-square-on-the-bag-screen)
- [Pressing a key](#pressing-a-key)
- [The selectable hotbar](#the-selectable-hotbar)
- [Cooldown and charges](#cooldown-and-charges)
- [Pages](#pages)
- [Events](#events)
- [Which container the bar reads](#which-container-the-bar-reads)
- [Persistence](#persistence)
- [The consumable bar in the bag](#the-consumable-bar-in-the-bag)
- [Placing a bar on the HUD](#placing-a-bar-on-the-hud)
- [Visibility](#visibility)
- [Restyling a bar](#restyling-a-bar)
- [See also](#see-also)

## Bars are data

Every bar is named by a gameplay tag and described by one `FSnapHotbarBarSpec` row in `HotbarBarSpecs`
(**Project Settings > Snap Inventory System - Equipment > Hotbar > Bars**):

| Field | Means |
|---|---|
| `BarTag` | The bar's identity; every function, placement and saved binding names a bar by it |
| `SlotCount` | How many squares it has |
| `PanelSlotCount` | How many of them the equipment panel draws (the two built-in bars only) |
| `AdmissionQuery` | Which items it takes (below) |
| `Actions` | The Enhanced Input actions that fire its squares, in square order; empty is a keyless bar |

**`HotbarBarSpecs` ships empty, and empty means the two built-in bars**, served from the older per-bar settings exactly
as before:

| Bar | Served from | For | Keys |
|---|---|---|---|
| `SnapInventorySystem.Hotbar.Bar.Ability` | `HotbarAbilitySlotCount`, `HotbarSlotInputActions`, `HotbarPanelAbilitySlotCount` | Abilities, and consumables unless narrowed | `1`–`9`, `0` |
| `SnapInventorySystem.Hotbar.Bar.Consumable` | `HotbarConsumableSlotCount`, `HotbarConsumableInputActions`, `HotbarPanelConsumableSlotCount` | Consumables | Ship unmapped |

**Name any row and the list becomes the whole truth.**

- A bar it does not name does not exist, so list the two built-in bars too if you still want them.
- A row whose tag is invalid or already used by an earlier row is skipped.
- An empty `AdmissionQuery` is a bar that admits nothing.
- Each problem is logged once as a Warning.
- `GetBarSpecs()` answers the resolved list, and `GetBarSlotCount(BarTag)` one bar's count.

A project's own bar, keyless, taking every item whose hotbar activation is `Event`:

```ini
[/Script/SnapInventorySystem.SnapEquipmentSettings]
+HotbarBarSpecs=(BarTag=(TagName="SnapInventorySystem.Hotbar.Bar.Ability"),SlotCount=10,PanelSlotCount=10,AdmissionQuery=...)
+HotbarBarSpecs=(BarTag=(TagName="SnapInventorySystem.Hotbar.Bar.Consumable"),SlotCount=2,PanelSlotCount=2,AdmissionQuery=...)
+HotbarBarSpecs=(BarTag=(TagName="MyGame.Hotbar.Tools"),SlotCount=4,AdmissionQuery=...)
```

- **Author the queries in the Project Settings editor** rather than by hand — a tag query's text form is not meant to be
  typed.
- ⚠ A mistyped `TagName` in an `.ini` imports as an **empty** tag rather than failing, and that row is then skipped with
  a Warning.
- The equipment panel draws the two built-in bars only.
- A project's own bar appears on the HUD through a placement (below).

## What may be bound

One predicate decides bar admission, and absence is the refusal:

```
CanBarAdmitItem(BarTag, Instance)
CanAssignToBarSlot(BarTag, Index, Instance)
```

An item needs a `Hotbar` fragment whose `Activation` is set. After that:

- A built-in bar from an **empty** list admits by the older rule:
  - The Ability bar takes `Ability` items, and `Consumable` items too unless narrowed.
  - The Consumable bar takes `Consumable` items.
  - Neither takes an `Event` item — unless it ticks its Hotbar fragment's `bAllowOnAbilityBar`, which lets the Ability bar
    take it (never in abilities-only mode).
- A **listed** bar admits by its `AdmissionQuery`, matched against the item definition's `ItemTags` plus one implicit tag
  for its activation.
  - The implicit tag is `SnapInventorySystem.Hotbar.Activation.Ability`, `.Consumable` or `.Event`.
  - So no item needs re-authoring to be admitted by what it does.

**`CanAssignToBarSlot` additionally needs:**

- The index in range.
- The item **held** by the main bag or by a custom container whose held-item abilities are active (Spells).

**What "held" means:**

- "Held" here means the grid **or** a pouch. Moving a bound item into a pouch to tidy up does not unbind it.
- On a **selectable** bar, a weapon the player **wears** counts as held too (`IsHeldForBar` — see
  [The selectable hotbar](#the-selectable-hotbar)).

Recipes, bag upgrades, materials and currency carry no `Hotbar` fragment, so nothing has to be listed anywhere to
exclude them.

**Two settings narrow the Ability bar:**

- The `bHotbarAbilityBarAcceptsAbilitiesOnly` setting makes the bar refuse consumables (a bar from an empty list).
- The `bHotbarAbilitiesOnlyMode` setting fills the ability bar from `HotbarAbilityAllowList` (item definitions, in square
  order); its squares can't be cleared, only reordered by swapping.
  - This **implies** the narrowing above, on a listed Ability bar as well.
  - That is because an auto-populated unremovable bar that still accepted a potion would have to evict an ability the
    player has no way to put back.

## Binding

```
Server_AssignBarSlot(BarTag, Index, Instance, ExpectedPageIndex)
Server_ClearBarSlot(BarTag, Index, ExpectedPageIndex)
Server_SwapBarSlots(BarTag, IndexA, IndexB, ExpectedPageIndex)
ResolveBarSlotItem(BarTag, Index)
FindBarSlotForItem(BarTag, Instance)
FindFirstEmptyBarSlot(BarTag)
BindToFirstEmptyBarSlot(BarTag, Instance)      // authority only
IsBarSlotLocked(BarTag, Index)
```

`ExpectedPageIndex` is the page the caller was looking at.

- The page is server-authoritative and deliberately not predicted, so a client's index lags by a round trip.
- Flip the page and act immediately and, without the check, the change lands on the page the server is on while the
  squares still show the one you aimed at.
- Pass `-1` to skip the check.

### Consumables that fill the bar by themselves

The `bAutoBindConsumablesToHotbar` setting (**Snap Inventory System - Inventory → Hotbar → Bars**, off by default) is the consumable bar's answer to auto-equip. With it on, an
item the Consumable bar admits — a Hotbar fragment with Activation Consumable — is bound to the first empty consumable
square (`FindFirstEmptyBarSlot`) at two moments:

- A pickup grants it into the bag and no equipment slot took it (`BindToFirstEmptyBarSlot`, on the server).
- The player shift-clicks it in the bag and no equipment slot would take it.
  - The quick-equip gesture found no target and would otherwise end in a refusal.
  - With every consumable square taken, that Shift+click puts it in the FIRST square in place of the one there
    (`bShiftClickSwapsFirstConsumableSquare`, on by default; off refuses as before).
  - Nothing in the bag moves; the old potion simply stops being on a square.

**Good to know:**

- The item stays in the bag; the square only refers to it.
- An item already on the consumable bar is left where it is, and a full bar binds nothing.
- A project that keeps its consumables in consumable equipment slots sees no change either way.
  - A worn item is not in the bag, so neither moment reaches it.
- The demo turns it on — see [the demo map](demo-map.md#the-hud-the-hotbar-keys-and-the-menu).

## Moving a square on the bag screen

On the bars drawn in the bag screen (the equipment panel's — `ESnapInventoryHotbarSurface::Panel`), you move a square with
two presses:

1. **Click a filled square**, or press a pad's **A** on it. That **picks** it: nothing is used.
   - You should see the square stay framed and lightly tinted in the theme's `SelectionGlow`.
2. **Click another square of the same bar**, or press **A** on it. That puts it there — a swap with a filled square, a
   move into an empty one.

**How it works:**

- The move goes through that square's own release, the very one a drag of the picked square runs there
  (`AcceptDroppedOperation`).
  - So a click and a drag cannot disagree about what a square takes.
- While a square is picked, the square under the pointer — or the one a pad has focused — shows the green / red outline
  with the action's symbol (Swap, Place, the red X) at its top-right corner.
  - The game HUD's own bar never does, since a press there uses its square.

**Rules of a pick:**

- **Another bar's square** refuses it, with the refused sound, and the pick stays — a swap is within one bar of one page.
- **The pick ends** on the picked square again, B / Escape, a right-click on it (which clears the square), a page change,
  the square being emptied, closing the screen, dying.
  - It also ends when an item carry or a drag of a square begins: one "the next press moves it" at a time.
- **Dragging** a square still swaps it, and **right-click** still takes a square off the bar.
- **The `bBagScreenHotbarClickPicks` setting** (**Project Settings → Snap Inventory System - Equipment → Hotbar | Input**, on by
  default) decides the CLICK.
  - Off, a click on the bag screen's bars uses the square.
  - A pad's A picks whatever it says — a pad has no drag, so the pick is its only way to reorder the bar.
  - The game HUD's own bar always uses on a click.
- `USnapInventoryHUDComponent::TryPickOrPlaceBarSquare(Square)` is the one entry point (Blueprint-callable), with
  `HasBarPick`, `GetPickedBarSquare`, `GetPickedBarOperation` and `ClearBarPick`.
  - The pick holds a `USnapInventoryHotbarDragDropOperation` — the square's reorder, never an item.
  - So nothing that catches an item release can answer it.

## Pressing a key

`ActivateBarSlot(BarTag, Index)` is the one entry point — the key and a click on a square of the game HUD's bar both call
it.

- On the bag screen's bars, a click picks the square instead — see
  [Moving a square on the bag screen](#moving-a-square-on-the-bag-screen).
- A **Shift- or Ctrl-click** on a square does nothing (the bag's equip gesture is a habit, and on a square it used to use
  the item).
- **Right-click** clears the square.
- The square under the cursor shows its hover: a part named by `HoverGlowWidgetName` (`HoverGlow`) on a Widget Blueprint
  square that has one, else its frame in `HotbarSlotHoverColor`.

The **item's** `Activation` decides the route, never the bar:

| `ESnapHotbarActivation` | Pressing it |
|---|---|
| `Ability` | Activates the ability its Granted Ability Set grants (`ContainerAbilities`) |
| `Consumable` | Spends one unit through the bag's consume door, exactly as right-click > Consume does |
| `Event` | Calls the item's `USnapItemFragment_Hotbar::OnHotbarEventActivated` and reports what it returns |

⚠ Without `GameplayAbilities` an `Ability` item does nothing when pressed: there is no ability system and no Granted
Ability Set. `Consumable` and `Event` items work the same in both builds.

**Each square shows the key mapped to its action right now**, read from the player's live Enhanced Input mappings.

- It repaints after every rebuild of them, so a key changed in a key-binding screen shows on the square at once.
- A controller button shows by its short name — an arrow for a D-pad direction, LB, RT.
- A project that binds `FSnapInventorySystemModule::OnResolveInputGlyph` draws its own glyph there instead
  ([Input](input.md)).

**To make an `Event` item:**

1. Subclass `USnapItemFragment_Hotbar` in a Blueprint.
2. Override `OnHotbarEventActivated`.
3. Put that fragment on the item.

**Good to know:**

- It runs on the owning client, like every activation, so a server-side effect is your own RPC.
- The fragment is shared by every instance of the item: keep per-item state on the instance, never on the fragment.
- `Unset` is refused when the item is saved.
- An `Event` item whose key acts like an ability — a spell you cast your own way — ticks `bAllowOnAbilityBar` to sit on
  the default Ability bar.
  - And it overrides `GetEventCooldown` to show its cooldown on the square (the demo's spell scrolls do both).

## The selectable hotbar

Off by default. Turn on `bSelectableHotbar` (**Project Settings > Snap Inventory System - Equipment > Hotbar | Selectable**), and a
key of a **selectable** bar no longer uses its square: it **selects** it.

- **The square rises a little** (`SelectedHotbarSquareRise`, 6 units, eased like a hover grow — one step with
  `bReduceItemMotion`).
- Its item is **held in the hand**.
  - It is drawn in `HeldItemSlot` (unset means `Equipment.Slot.MainHand`).
  - The worn item's own look is put away there while something is held.
- Nothing is used until your **use input** calls `USnapHotbarSelectionComponent::UseSelectedItem` — the demo's left click and RT do.
  - The same activation the key ran with the switch off then runs: a potion is drunk, a scroll cast, a consumable eaten.

**The details:**

- **Which bars select:** `SelectableHotbarBars`; empty means the Ability bar (the 1-0 keys).
  - Every other bar — the demo's Q / R potion squares — uses its square at once, as always.
- **Put away by:**
  - The same key again.
  - The key of an **empty** square.
  - Opening the bag, a vendor, a loot window, the craft window or the trade screen.
  - A page turn that leaves another item on the square.
  - The item leaving the player (the last unit used, sold, dropped).
  - A death (`USnapInventoryHUDComponent::DismissScreensForDeath`).
  - A new pawn.
- **Escape:** Yours. Call `ClearBarSelection` from your own Escape or "put away" input.
  - The plugin binds no Escape, because in your game Escape belongs to your menu.
  - The demo's Escape and Start put the item away before opening its menu.
- **While a screen is open:** A selecting key does nothing (pointing at a bag item and pressing a key still puts that item
  on the square).
  - A **click** on a square of the game HUD's bar uses it at once in either mode — a square can only be clicked while a
    screen is open.
  - On the bag screen's bars, a click picks the square
    ([Moving a square on the bag screen](#moving-a-square-on-the-bag-screen)).
- **Weapons:** While the switch is on, a selectable bar also takes an item that is only ever **held**: one with no Hotbar
  activation whose Equippable fragment fits `HeldItemSlot` (a sword).
  - Using one is your attack, not the plugin's: `UseSelectedItem` answers `HeldOnly` for it and runs nothing.
  - The demo swings it with its own style.
  - Abilities-only mode still keeps the Ability bar for Ability items.
- **Worn weapons:** A weapon the player **wears**, in either hand, goes on a selectable square too.
  - Drag it from the equipment column (or carry it there on a pad), and it **stays worn** — the square is a quick swap
    back to it.
  - Selecting the square of the weapon already worn in `HeldItemSlot` raises the square and swaps nothing: the worn look
    stays and nothing else is drawn.
  - A one-hander worn in another slot — the off hand — is drawn in the hand while it is held, and its own worn look is
    hidden meanwhile.
  - A bound weapon keeps its square when it is equipped or taken off (it is held either way).
  - Dropped, sold or traded, it leaves the square like any item.
  - A shield (it does not fit `HeldItemSlot`) and a worn potion are not bound — a worn potion keeps its own consume key.
- **What is drawn:**
  - A weapon is drawn exactly as it would be worn in `HeldItemSlot`, in the hand whatever its own visual mode says.
  - Anything else — a potion, a scroll — is its world model (`ASnapHeldItemVisual`) at `HeldItemSocketName` (`hand_r`).
    - Its center is at `HeldItemAttachOffset` (the default is the UE5 mannequin's right palm).
    - It is at its own modeled size — the World Representation fragment's `Scale` is its size on the ground and is not
      used in the hand.
    - It is shrunk to at most `HeldItemMaxSize` (18 cm).
  - Both are spawned by the server and replicate, so every player sees what another holds.

The selection is the **server's** (`Server_ToggleBarSlotSelection`).

- So on a client, the square rises, and the item appears in the hand, one round trip after the key.
- It replicates to every machine as `FSnapHotbarSelection`: the bar, the square, the held item's definition and
  `WornSlot`, the equipment slot it is worn in.
  - Never the instance, which only its owner can see.

**Reading it:**

- Ask the player's `USnapHotbarSelectionComponent` (the server adds it beside the hotbar): `GetSelection`,
  `HasSelection`, `IsBarSlotSelected` or, on the owning machine, `GetSelectedItem`.
- `IsBarSelectable` (on the hotbar component) says whether a bar selects right now.
- `IsSelectionDrawnInHand` (on the selection component) says whether what is held takes the hand.
  - It is false for the weapon already worn there.
  - An attack that reads the selection should treat that one as not held, as the demo's do.
- `PressBarSlot` (on the hotbar component) is what a key does to its square in either mode — call it from your own
  hotbar input.
- The worn look in the hand is hidden through `USnapEquipmentComponent::SetSlotVisualSuppressed`.
  - A look spawned later for that slot (a new equip, a re-possession, a repair) obeys it from its first frame.
- `SpawnHeldItemVisual` is the one door that draws a held item.

**Off, nothing here changes anything:**

- Every key uses its square, no weapon goes on a bar, and nothing is replicated.
- A worn weapon bound while it was on reads as an empty square once it is off.
- The binding is dropped at the next change to the bag.

## Cooldown and charges

Two questions a square asks about its item, each a hook on `USnapHotbarComponent` you can override in a Blueprint or C++
subclass of the component:

| Hook | Answers | Shipped answer |
|---|---|---|
| `GetSlotCooldown(BarTag, SlotIndex, Remaining, Duration)` | Seconds left and the whole cooldown; a `Duration` of 0 means none | `GetDefaultSlotCooldown` |
| `GetSlotCharges(BarTag, SlotIndex, Current, Max)` | The item's charges; `Max` 0 means it has none, -1 means no limit | `GetDefaultSlotCharges` |

**Cooldown, by default,** comes from the Gameplay Ability System, by the same query the engine's own
`UGameplayAbility::GetCooldownTimeRemainingAndDuration` makes.

- The cooldown tags of the abilities the square's key runs are matched against the owner's active effects. The longest
  remaining wins.
- **An `Ability` item's** abilities are its Granted Ability Set's `ContainerAbilities`, read off their class defaults
  whether granted or not.
- **A `Consumable` item's** is the granted ability matching `ConsumeAbilityTag` (**Snap Inventory System - Inventory → Consume**),
  when that setting names one.
  - Every fragment of a `Consumable` item answers too, in every build, through
    `USnapItemFragment::GetConsumeCooldownFor` (the base says none; the longest wins).
  - So a consumable whose fragment refuses while a cooldown of its own runs shows the wait.
- **An `Event` item** answers for itself, in every build, through its Hotbar fragment's `GetEventCooldown` (the base says
  none).

**Charges, by default,** are the stacks of the Gameplay Effect named in the item's Hotbar fragment `ChargesEffect`,
active on the owner from any instigator.

- `Max` is that effect's Stack Limit Count, or -1 when it has no limit.
- Leave `ChargesEffect` unset and the item has no charges.
- The count is only what the square draws: whether the item can be used at zero charges is its ability's own business,
  as with a cooldown.

**Overriding them:**

- Both are asked on the owning client, several times a second while a cooldown runs.
  - So an override must be cheap and change nothing.
- An override can call the default and adjust it.

**Without `GameplayAbilities`,** both defaults answer zero.

- So a square draws no overlay unless the hook is overridden.
- A bar, with no ability system to listen to, repaints on its own activations and changes only.

### What a square draws over its item

| Overlay | Shown | Theme role |
|---|---|---|
| A shade over the icon, draining downward | While `GetSlotCooldown` reports time left; its height is the fraction left | `HotbarSlotCooldownFill` |
| The whole seconds left, centered, rounded up | With the shade | `HotbarSlotCooldownText` |
| The charges, top right: `2/3`, or `2` when they have no limit | While `GetSlotCharges` reports a `Max` other than 0 | `HotbarSlotCharges` |
| `HotbarSlotLockedMarker`, bottom left | On a square `IsBarSlotLocked` holds (abilities-only mode); empty turns it off | `HotbarSlotLocked` |

### What an empty square shows

**Nothing, by default.**

- `HotbarEmptySquareGhosts` names a picture per bar — keyed by the bar's tag, the value a texture's path.
- An empty square of that bar draws it in its own icon image, tinted `HotbarEmptySquareGhostTint` (white at 22% by
  default), until something is put on the square.
- A flat white silhouette on transparent reads best; the tint does the dimming.
- A Widget Blueprint square gets its icon's own tint back when its item shows.

**The HUD and the equipment panel can differ.**

- The HUD bars read `HotbarEmptySquareGhosts` only.
- The equipment panel's squares read their own map first, `HotbarPanelEmptySquareGhosts`:
  - A bar listed there draws that picture tinted `HotbarPanelEmptySquareGhostTint`.
  - An entry with no picture draws NO ghost on the panel.
  - A bar it does not list keeps `HotbarEmptySquareGhosts` and its tint.
  - So a project that sets only the shared map sees the same ghost on both surfaces.
- The demo draws none on the HUD (its shared map is empty), and a faint plain plus on the panel's hotbar and potion
  squares (`T_Ghost_Plus` in the panel map, white at 16%).

**Nothing ticks for them.**

- A bar listens to the owner's ability system — an effect added (on the server and on the owning client), removed, or
  re-stacked — and to its own activations and changes.
- It repaints every square's overlays when one arrives.
- A cooldown found running starts a timer that repaints ten times a second, and stops itself once no square on the bar is
  cooling down.
- Every overlay is hit-test invisible, so a press and a drop still land on the square.

### Tooltip and drag ghost

- An occupied square shows the **item tooltip** — the same card the inventory screen's grid slots show.
- A drag from it carries the grid's **drag ghost**, from the first frame, before the bag has ever been opened.
- **Where both classes come from:**
  - The square's own `TooltipWidgetClass` and `DragVisualClass`, when set.
  - Otherwise the grid slot class of the first inventory panel in `InventoryScreenClass`'s widget tree, read off the class
    rather than a live screen.
  - With neither, a square shows a plain tooltip with the item's name, and a drag carries no ghost.
- The ghost is only a picture: the drag still carries no item, so letting go over the world changes nothing.
- An empty square clears only a tooltip it set itself.

## Pages

**Functions:** `StepActivePage(Delta)`, `Server_SetActivePage(Index)`, `GetActivePageIndex()`.

- A page holds every bar.
- Page next and previous (`IA_Hotbar_PageNext` / `PagePrev`) ship unmapped.
- A raw modifier-plus-wheel binding sits behind `bBindHotbarPageKeysRaw`, which ships off.

**The page indicator:**

- A bar whose spec sets `bShowPageIndicator` shows which page is live (`2/3`) at the end of its row while
  `HotbarPageCount` is above 1, and follows `OnActivePageChanged`.
- The Ability bar an empty list serves shows it.
- The Consumable bar shows it only with `bShowConsumableBarPageIndicator` on (off by default; the demo turns it on).
  - It pages with the Ability bar either way.
- The indicator a bar builds itself draws the house text shadow (offset (1, 1), the palette's `TextShadow`), so it reads
  over a bright scene.
- A TextBlock named by `PageIndicatorWidgetName` keeps its own style.

## Events

| Event | Raised |
|---|---|
| `OnSlotActivated(BarTag, SlotIndex, Result)` | Once per activation attempt — a key, a click or a call to `ActivateBarSlot` — with its `ESnapRefusal`. An empty square reports `SlotEmpty`, an index past the bar `NoSuchSlot`. Assigning, clearing and swapping raise nothing. |
| `OnActivePageChanged(NewPageIndex, OldPageIndex)` | When the live page changes: on authority from the switch, the step and the restore; on a client when the replicated page arrives. Setting the page it is already on raises nothing. |
| `OnHotbarChanged(Change)` | Whenever the bars may need repainting (see Persistence). |
| `OnSelectionChanged(NewSelection, OldSelection)` | When what is held from a selectable bar changes: on the server when it decides, on every client when the replicated selection arrives. Using the held item raises `OnSlotActivated` as a key does. |

`OnSlotActivated` is raised where the activation runs — the owning client, or a listen-server host for itself — because
activation stays local for ability prediction.

## Which container the bar reads

The bar reads, resolved from the owning actor's components:

- The **main inventory**.
- **Every custom container whose held-item abilities are active** (the default setup has one, Spells, which exists only
  with `GameplayAbilities`).

The main bag is the one whose `bIsMainInventory` is true — an explicit marker, not a class test.

- So a Blueprint subclass of `USnapInventoryComponent` keeps its hotbar.

⚠ There is no fallback.

- A container that does not claim the role is not the main bag, and the honest answer is "no bag".
- An "exact class, else the first subclass found" rule would silently point the consumable bar at another container
  depending on component order.

**Containers it does not search:**

- A container whose held-item abilities are off — Shards — is deliberately not searched: nothing in it carries a `Hotbar`
  fragment.
- A project that wants shards on a bar adds the fragment and a line to `ResolveHolderOf`.

With `bSelectableHotbar` on, a selectable bar also reads the owning actor's **equipment component**, for the worn weapons
its squares may refer to (`IsWornForBar`).

## Persistence

- Bindings are saved per profile and restored on login, keyed by item **definition** and re-resolved to a currently-held
  instance.
- `SaveHotbarPages()` and `LoadHotbarPages()` are callable directly.

**The restore waits** until the main bag or a container whose held-item abilities are active holds an item in its grid.

- That is because resolving a saved definition against an empty bag resolves nothing.
- An item inside a pouch does not count.
- A worn item counts only while `bSelectableHotbar` is on.
  - A selectable bar can restore to a worn weapon — after any item of the same definition that is held in the bag.
- It checks once a second and stops waiting after twenty tries, about twenty seconds, then proceeds with whatever is
  there.

⚠ Nothing is saved before the restore has run, not even when the player leaves.

- Until then the hotbar cannot tell whether the profile on disk holds bindings a save would overwrite.
- So a session shorter than that, in which everything the player picked up was equipped, leaves the profile's hotbar
  store as it was — a new profile gets none.
- That is expected, not a lost save.

`OnHotbarChanged` carries an `ESnapHotbarChange` saying what changed, so a listener that only repaints does not also
trigger a save.

**Record versions:** Each saved binding names its bar by `BarTag` (`FSnapHotbarSavedSlot`), and the record is version 2.

- A **version 1** record — bars named by the enum — is discarded when it loads, not migrated:
  - The store is deleted, and one Log line says so.
  - The hotbar starts empty, and saving carries on normally.
- A record from a **newer** build, or one that cannot be read, is still refused and left on disk.
  - Nothing is saved over it that session.
- A binding on a bar the project has since removed is kept and written back, and restores if the bar returns.

## The consumable bar in the bag

The bag's equipment panel draws the Consumable bar in one of two places:

- **In place of the consumable equipment slots it supersedes.**
  - This needs `bHotbarReplacesConsumableEquipmentSlots` on (the default) and those slots listed in `ConsumeSlots`.
  - The bar is drawn where the first of them would have been, and the slots themselves are not drawn.
- **In a container the panel names**, when nothing was superseded: `USnapEquipmentPanel::ConsumableHotbarContainerName` (None
  by default).
  - This is for a slot set with no consumable slots at all.
  - Its potions stay in the bag, where the bar can refer to them, so the bar needs a place of its own.
  - The example panels name theirs: `ConsumableRow` on the Character screen's, `ColumnConsumables` on the plain one.

A panel never draws the bar twice: the named container is used only when supersession placed nothing.

**Titles over the panel's two bars.** With `bShowEquipmentPanelHotbarTitles` on (off by default), an equipment panel shows
the widgets `USnapEquipmentPanel::AbilityHotbarTitleName` (`HotbarTitle`) and `ConsumableHotbarTitleName` (`PotionsTitle`)
name.

- Each shows only while its bar is drawn on that panel:
  - The ability bar placed and not hidden by its host.
  - The consumable bar parented with at least one square.
- It keeps them collapsed otherwise, so a title never labels the shard row the ability bar gives back or an empty potion
  row.
- **Put a title BESIDE its bar's container, never inside it.** The consumable bar's container is cleared on every build.
- The Character screen's panel authors HOTBAR and POTIONS, each in a box with its row (`HotbarGroup`, `PotionsGroup`) that
  takes the row's old place.
  - So with the setting off, it lays out exactly as it did without them.
  - The demo turns the setting on.
- The example authors both titles COLLAPSED, so the code is the only thing that shows them.
  - A panel that is never bound, or a frame before its first build, shows neither.

## Placing a bar on the HUD

`HotbarHUDPlacements` puts a bar where a widget your HUD already owns sits. Each entry names its bar by `BarTag`:

```ini
+HotbarHUDPlacements=(BarTag=(TagName="SnapInventorySystem.Hotbar.Bar.Ability"),HostWidgetClass=...,AnchorWidgetName="...")
```

An entry with no `BarTag` is inert.

**The bar is placed once its host widget exists, however late that is.**

- The HUD component keeps looking for about thirty seconds after it starts, and says so once in the log if a host never
  appears.
- A native host class that does not exist is not waited for. A Blueprint host is, until it loads.
- The demo map's HUD is the example — see the [demo map](demo-map.md#the-hud-the-hotbar-keys-and-the-menu).

**A placed bar is built once, from the settings in force then.**

- After a change a bar reads only when it is built (a theme, say), `USnapInventoryHUDComponent::RebuildHUDHotbars` takes every
  bar the component placed off its host and places it again from the settings now.
- It is the partner of `USnapInventoryHUDComponent::DiscardCachedScreens`, which rebuilds the screens and leaves the HUD's bars
  alone.
- A placement whose host is not on screen at that moment is left unplaced, and the log says so.

## Visibility

```
USnapUICoreLibrary::SetHotbarBarVisibility(WorldContext, BarTag, bVisible)
```

An empty `BarTag` means every bar. It returns how many bars it changed.

## Restyling a bar

The **Hotbar | Appearance** settings are on **Snap Inventory System - Inventory UI**; the bars, keys, pages, placement and
selection are on **Snap Inventory System - Equipment** (`bAutoBindConsumablesToHotbar` on **Snap Inventory System - Inventory**,
`GlyphDevicePreference` on **Snap Inventory System - UI Core**).

A bar and its squares are native widgets that need no asset, and both can be replaced by a **Widget Blueprint**:

- The bar through `HotbarWidgetClass`.
- Each square through `HotbarSlotWidgetClass`.

The two are separate choices, so a new frame can keep the plugin's squares and a new square can sit in the plugin's
frame.

- Every bar is built from these classes: every bar on the HUD and the two on the equipment panel.
- A `HotbarWidgetClass` that will not load, is not a `USnapInventoryHotbarWidget` or is abstract builds the plugin's own bar
  instead, with one Warning naming the path.

**Both settings ship naming one:** `WBP_ExampleDefaultHotbar` and `WBP_ExampleDefaultHotbarSlot`.

- They are the plugin's own bar and square copied part for part, with **Apply Hotbar Appearance Settings** (below) on.
- So they draw exactly as the C++ widgets, and every **Hotbar | Appearance** setting still reaches them.
- Open them to restyle. Set a setting back to empty for the C++ widget.

A Blueprint keeps its tree exactly as you authored it. The plugin finds the parts it needs **by name**:

| Class | Name property | Default | Must be | The plugin drives |
|---|---|---|---|---|
| `USnapInventoryHotbarWidget` | `RowWidgetName` | `HotbarRow` | A panel that holds several children | Its children: it fills it with squares |
| `USnapInventoryHotbarSlotWidget` | `FrameWidgetName` | `HotbarSlotFrame` | A Border | Its Brush Color, while a drag hovers the square |
| | `IconWidgetName` | `HotbarSlotIcon` | An Image | The brush's texture and the visibility |
| | `KeyTextWidgetName` | `HotbarSlotKey` | A TextBlock | The text and the visibility |
| | `CountTextWidgetName` | `HotbarSlotCount` | A TextBlock | The text and the visibility |
| | `CooldownWidgetName` | *None* | An Image | The visibility, and its vertical render scale from a bottom-center pivot |
| | `CooldownTextWidgetName` | *None* | A TextBlock | The text and the visibility |
| | `ChargesWidgetName` | *None* | A TextBlock | The text and the visibility |
| | `LockedWidgetName` | *None* | A TextBlock | The text and the visibility |
| `USnapInventoryHotbarWidget` | `PageIndicatorWidgetName` | *None* | A TextBlock | The text and the visibility |

- The overlay parts and the page indicator ship as *None*, so a Blueprint authored before they existed stays exactly as it
  was and logs nothing.
- Name a part to have it driven.
- The native square and bar build all five.

With **Apply Hotbar Appearance Settings** off (below), everything else — fonts, colors, sizes, brushes, layout — stays
yours:

- At rest, a square's frame paints the Brush Color you authored.
- The drag-over cue is the only fill the plugin writes.

⚠ The shipped default square, `WBP_ExampleDefaultHotbarSlot`, has that switch **on**.

- So in a copy of it, a frame color, key color, font or size you change in the designer is repainted from the settings.
- Turn the switch off in your copy, or set that part's Themed Parts row to *Authored*.

Set a name to *None* to keep that part out of the plugin's hands: a square with no count text simply shows no counts.

**Apply Hotbar Appearance Settings.** A Blueprint square or bar can ask the plugin to style it exactly as it styles its
own. Turn on `bApplyHotbarAppearanceSettings` in its class defaults (it is off unless you turn it on), and the
**Hotbar | Appearance** settings reach it as they reach the C++ widgets:

- Hotbar Slot Size on the square's root Size Box.
- Corner Radius and Outline Width on the frame's outline and on the focus ring.
- Outline Width as the frame's padding.
- Icon Padding around the icon and the cooldown shade.
- The key, outline, background and panel background colors, and the cooldown, charges and locked colors.
- The key and count fonts (sized by Key / Count Font Size with no theme; a picked theme's fonts and colors otherwise).

**What else changes:**

- The frame then rests on the background of the surface it is drawn on, instead of its authored Brush Color.
- On a bar, it styles the page indicator (its font, color and shadow).
- A **Themed Parts** row whose Source is *Authored* keeps exactly what it names — a Font row the text's whole font, a color
  row that color.
  - So one part can stay yours while the settings reach the rest.

⚠ Two things the plugin does decide on a Blueprint square:

1. The square itself is set *Visible*, as the native one is, because a drop and a press must land on it even while it is
   empty.
2. With the switch off, the frame's resting fill is the Brush Color it has when the square is **initialized** — the value
   in the designer.
   - So author it there: a color written from the Blueprint's *Construct* event is repainted with the authored one.

A bar's row, unlike a square's parts, cannot be set to *None*: without a row there are no squares.

`/SnapInventorySystem/Core/UI/WBP_ExampleHotbar` (a plate around a Horizontal Box) and `WBP_ExampleHotbarSlot` (a rounder,
rimmed square) are restyled examples of both, with the switch off.

- **Put decoration around the row, never inside it.**
  - The bar clears the row's children every time it rebuilds its squares.
  - The one exception is the page indicator: a TextBlock named by `PageIndicatorWidgetName` inside the row is put back
    after the squares on every rebuild.
  - That is where `WBP_ExampleHotbar` keeps it.
- **The row can be a Horizontal Box or a Vertical Box.**
  - A Vertical Box makes a vertical bar, and keyboard and gamepad navigation then walks it up and down.
  - Any other panel that holds several children receives the squares and lays them out itself, and navigation treats the
    bar as one row.
- **The bar and its row are set Self Hit Test Invisible**, as the native row is, so the bar does not eat hover meant for
  whatever is beneath it.
  - A plate you put around the row needs the same setting in your tree — a Border left Visible is a hit target across its
    whole rect.
- **A missing or mistyped part is a Warning**, once per class and name, saying what the squares will not show.
  - A bar with no row draws no squares and says so.
  - Before this contract, a Blueprint bar placed successfully and drew nothing, with nothing in the log.

**Typography:**

- A theme's typography reaches the **native** squares' key and count labels (`HotbarKey`, `HotbarCount`).
- A Blueprint square's text keeps the fonts you authored unless its Themed Parts list a **Font** row for it.
  - The default copies list one for every text; the restyled `WBP_ExampleHotbarSlot` lists none.
- See [UI and theming](ui-and-theming.md#typography).

## See also

- [Input](input.md) — the keys
- [Persistence](persistence.md) — how bindings survive a session
- [UI and theming](ui-and-theming.md) — themes, and the widget-name contract every widget follows
- [Appendix: settings](appendix-settings.md) — the 60 **Hotbar** settings (most on Snap Inventory System - Equipment, the look on
  Snap Inventory System - Inventory UI)
