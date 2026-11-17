# Inventory and containers

`USnapInventoryComponent` is the container. One actor can carry several of them.

- The plugin ships the main bag and five [custom containers](custom-containers.md): Spells and Shards in
  `Core/Attachment`, the Keyring and Bank examples, and the demo's spellbook (`BP_Example_DemoSpellsContainer`).
- Its default setup attaches the bag, and Spells in a build with GameplayAbilities; one line of configuration adds
  Shards.

**On this page:**

- [Two layouts, and the one switch between them](#two-layouts-and-the-one-switch-between-them)
- [The spatial panel](#the-spatial-panel)
- [The grid](#the-grid)
- [Adding and removing](#adding-and-removing)
- [Stacking](#stacking)
- [Moving](#moving)
- [Rearranging — sort, compact, stack all](#rearranging--sort-compact-stack-all)
- [Searching](#searching)
- [Pouches — containers inside the grid](#pouches--containers-inside-the-grid)
- [Counters — currency and anything like it](#counters--currency-and-anything-like-it)
- [Routing between containers](#routing-between-containers)
- [Admission — what a container takes](#admission--what-a-container-takes)
- [Weight](#weight)
- [Events](#events)

## Two layouts, and the one switch between them

A container arranges what it holds in one of two ways, set per container by `LayoutMode`:

| `ESnapLayoutMode` | What a square means |
|---|---|
| `Slots` (the default) | One item per numbered slot, whatever its size |
| `Spatial` | A `GridWidth` x `GridHeight` grid; an item covers the cells its shape needs |

Both are first-class and both are permanent. Pick whichever suits the game:

- A container in `Slots` is the classic one-item-per-square bag.
- A container in `Spatial` is the shaped grid where a rifle takes six cells and a ring takes one.
- Neither is a step on the way to the other, and a project can run both at once — a spatial backpack beside a
  slots-based quick bag is an ordinary configuration.

**A project that sets nothing gets `Slots`**, reaches the same outcomes it always did, and pays nothing for the spatial
model:

- No behavior changes.
- No extra bytes are replicated.
- A save written by such a container carries no spatial data at all.

To turn a container spatial, set three properties — in your project's `DefaultGame.ini`, or on the placed actor:

```ini
[/Script/SnapInventorySystem.SnapInventoryComponent]
LayoutMode=Spatial
GridWidth=10
GridHeight=6
```

Give the items that should take up more than one cell a `Footprint` fragment (see
[Items and fragments](items-and-fragments.md)); everything else stays one cell.

A container that already holds items changes layout through `SetLayoutMode`, which re-places every item or changes
nothing — see [Changing the layout while a container holds things](#changing-the-layout-while-a-container-holds-things).

## The spatial panel

The model above is only half of a spatial container: a project also needs a panel that draws items at the shape they
occupy.

- `USnapInventorySpatialPanel` is that panel, and it is a subclass of `USnapInventoryPanel`.
- The flowing one keeps working exactly as it did, and a project that does not opt in never constructs the spatial one.

Where `USnapInventoryPanel` flows its slot widgets through a wrap box in order, the spatial panel places one slot widget per
CELL on a canvas.

- By default each cell of a multi-cell item draws that item's own share of it: the whole icon at the size of the item's
  box, offset so that the cell's share lands in the cell, and clipped to the cell's bounds.
  - So no widget is stretched over a cell it does not own (`bDrawSpatialItemsPerCell`, **Snap Inventory System - Inventory → Spatial Grid**, on; turning it off is described
    below).
- Everything a slot widget already does — hover, tooltip, context menu, shift-click, selection, focus navigation — is
  inherited rather than rewritten.
  - Because they are still the same slot widgets in the same `SlotWidgets` array, and each works from any cell of an
    item rather than only its top-left one.

⭐ **One item, one target.** The cells of a multi-cell item light no selection or focus glow of their own: the grid draws
one of each round the item's whole box, with the slot's own glow brush (`USnapInventorySpatialPanel::UpdateItemGlows`).

- They share one tooltip — the anchor cell's — so it stays open while the cursor crosses the item.
- A plain press on an item that is not selected selects it on the release, so a press that becomes a drag selects
  nothing (`USnapInventorySlotWidget::SetPartOfMultiCellItem`).
- Every gesture on any cell acts on the item — Shift+click quick-move, a double-click that opens a container, the item
  menu, a sell — because each cell resolves to the item's anchor (`USnapInventorySlotWidget::GetOwningItem`).
  - A cell's own index is only for drawing (its count, its bars).
- Hovering a cell lights no square of its own: the grid outlines the whole item.
- A pouch window drawn as a grid follows the same rules (`USnapInventoryStorageSlotWidget::SetPartOfMultiCellItem`).

⭐ **Every cell of an item shows its rarity.** A multi-cell item's rarity background — the slot's `RarityGlowWidgetName`
part, at `RarityBackgroundOpacity` — fills every cell the item covers.

- So the item reads as one thing across its cells the way a Diablo grid colors an item's footprint.
- An item with no tier shows its own UI color.
- It used to show on the item's top-left cell only.
- The switch `bFillSpatialItemCells` (on) turns it off.
- A pouch window's grid does the same.

⭐ **Square cells with no gap.**

- `CellGap` is the space between neighboring cells (4 by default).
- Turning on `bSquareCellCorners` (off by default) draws every cell the grid builds with square corners.
  - Each rounded-box part of the slot's Widget Blueprint that rounds with fixed radii (its plate, its rarity background,
    its hover / selection / focus rings, its small badges) gets zero radii
    (`USnapSpatialGridRenderer::SquareRoundedCorners`).
  - A pill (half-height rounding) is left round.
- The shipped `WBP_ExampleSpatialPanel` sets both — `CellGap` 0, square corners — so its cells touch as one grid, each
  keeping its thin outline, and an item's picture runs across its cells with no band between them.
- A pouch window laid out as a grid has its own pair: `SpatialCellGap` and `bSquareCellCorners` on
  `USnapInventoryStorageWidget`.
  - `SpatialCellGap` below 0, the default, keeps `SlotSpacing` — which also spaces a flat pouch's list.
  - The shipped `WBP_ExampleStorageWindow` sets 0 and square corners, so a pouch's grid reads like the bag's; a flat
    pouch keeps its spacing.

⭐ **A shape that leaves cells of its box empty** — an L, a T — draws only in the cells it covers.

- Its empty cells are ordinary cells: hover and clicks on one reach that cell rather than the shape, and another item
  can stand in one.
- Two shaped items that each stand in the other's empty cells are both drawn whole.
- The shape's own top-left cell is different — it is reserved for the shape even when empty (see the Footprint fragment
  in [Items and fragments](items-and-fragments.md)).

⚠ Turning `bDrawSpatialItemsPerCell` off restores the older drawing: the widget at an item's anchor is stretched across
the item's whole box and the cells under it are collapsed.

- A shape's empty cells, and anything standing in them, are then drawn **above** it.
  - One pure rule decides the order for the bag's panel and for a pouch's window alike (`SnapGridDrawOrder::Resolve`),
    so they can still be seen and clicked.
- It reorders rather than repaints: an empty cell drawn over a shape shows its own background over that part of the
  shape's art.
- A grid of rectangles and single cells keeps exactly the two layers it has always had.
- Two shaped items that each stand in the other's empty cells cannot both be drawn whole that way: the one with the
  lower slot index is drawn on top and a cell of the other stays under its art.
- Such placements are allowed in either mode and the data is the same; see **Known limitations** in the plugin README.
- Turning the flag off does not change the width the pad walks.

`USnapSpatialGridRenderer` owns the arithmetic: cell to rectangle, rectangle back to cell, and the translucent overlay a
drag paints.

- It is a plain `UObject` a host widget creates and attaches to a named container.
- So a second surface can host a grid without a second implementation of where cell 14 is.

### What a drag shows

While an item is dragged over a spatial grid:

- The cells it would cover are washed GREEN when a release there would be accepted — it fits, or it would SWAP with the
  one item in the way.
- They are washed RED when it is blocked.
- A cell that is none of those is not painted at all.
- The drag picture's corner badge says which accepted release it is (two arrows for a swap); see
  "The drag picture, its badge and the outline" below.

The preview asks `ResolveGridPlacement` — the same function the server asks when the button is released — so the two
cannot disagree about whether a placement is legal. For a swap it asks `ResolveGridSwap`, which the move mutator asks
too (below).

### The drag picture, its badge and the outline

A drag carries the item's own picture — no square behind it:

- At about 70 % (`USnapInventoryDragVisualWidget::VisualOpacity`, applied to the picture only).
- At the size of the square it came from.
- Centered on the pointer, which is hidden for the whole drag, so the item itself is the cursor
  (`USnapUICoreSettings::HeldItemPointer`, `Centered` by default).
- `Beside` holds the picture where it was grabbed, with the arrow drawn.

A round **action badge** at its top-left says what letting go over the current target would do:

| Badge | Release |
|---|---|
| Down arrow (`Place`) | Into an empty square; onto a hotbar square; into a trade offer |
| Two arrows (`Swap`) | Trades places with the item there (an occupied square; an exchange of two worn items) |
| Stack (`Merge`) | Units join the stack there |
| Flask (`Mix`) | A mix rule makes a new item (`USnapMixRuleSet`) |
| Wrench (`Repair`) | A repair kit repairs the item |
| Bag (`Store`) | Into a pouch |
| Shield (`Equip`) | Worn in that slot |
| Arrow into a box (`MoveToContainer`) | To another container of the same owner (the bag and a bank) |
| Coin (`Sell`) | Reserved for a project's own drop target (no plugin surface sells on a drop) |
| X (`Refused`) | Refused — nothing happens |

- No badge shows over a panel's background, the screen, or the square the item came from.
- The square under the cursor is **outlined** in the theme's `DropAccepted` (green) for any accepted release and
  `DropRefused` (red) for the X.
  - One widget the player's HUD owns draws it (`USnapDropOutlineWidget`), drawn above every plugin window and never
    taking the pointer.
- The grids do not get the outline: their wash is their green and red.

Both come from the square's OWN release code, run check-only on every drag-over move:

- `ResolveDropBadge` on the bag square, the equipment slot, the hotbar square, the trade offer square and a pouch
  window's square.
- `GetLastPreviewBadge` on the bag grid and the pouch window's grid.
- They reach the picture and the outline through `USnapInventoryDragDropOperation::ReportDropFeedback` — the same call a
  project's own drop target makes from Blueprint.

Details:

- Slate sends drag-over only when the pointer moves, so a modifier pressed mid-drag shows on the next move.
- A gamepad carry outlines the focused square the same way (it draws no picture).
- The pouch window's grid washes and badges exactly like the bag grid; in its list layout its squares show the badge and
  the outline like the bag's squares.
- A view-only window shows the X on its squares and paints its grid's cell red: it takes no drop.

### Dropping an item onto another item: the swap

Release a dragged item onto exactly ONE other item and the two change places the way they do in Diablo or Tarkov:
**your item lands where you dropped it, and the other item moves out of the way.** The other item goes:

1. Back to the square your item came from, when it fits there at its own turn — the plain exchange.
2. Else into the space your item left — at its own turn first, then turned, when its `Footprint` sets `bCanRotate`.
3. Else to the first free spot in the grid, in reading order.

When it fits nowhere, nothing moves and the preview shows the cells red.

- So a three-cell sword dropped one cell up onto the potion above it swaps — the potion takes the cell the sword
  uncovered at its other end.
- It used to be refused because the sword's new cells covered its own old square.

`ResolveGridSwap(Mover, Target, Rotation, Blocker)` returns the whole answer (`FSnapGridSwapPlan`: where each item goes,
and which of the three steps placed the other one).

- The server logs the step on every swap.
- Both new places are judged against one map, with your item's new cells reserved first, so the two can never share a
  cell.
- A release that lands on two or more items, a selection of several items, and a drag from another container never swap.
  - Except a **chest's or a body's item**: it is taken in, and the other item moves out of the way as if the chest item
    had first stood where the bag would have put it with no square named.
  - That is `ResolveGridSwapForArrival`: that spot, else the space it would have left, else the first free spot — which
    is exactly the move the server then makes.
- A pouch laid out as a grid follows the same rule inside its window (`ResolveStorageSwap`); see `SwapStoredItems` for
  the argument order there.

### Where the item lands, and what a release on an item does

⭐ **The item lands under its picture.** With `HeldItemPointer` `Centered` (the default) the picture is centered on the
pointer, and a release lands the item's box on the squares under the picture.

- Its top-left lands at the pointer less half the box, in cells, rounded (`USnapGridPlacementLibrary::CenteredGrabCell`:
  an odd side takes its middle cell, an even side the cell that half of the pointer's square says).
- This holds for one item from any source: the grid itself, a pouch, the worn column, a loot window.
- With `Beside` a long item grabbed by its middle cell is held by that cell: the picture hangs from it, and the release
  lands the item's box so that the grabbed cell is the one under the cursor.
  - See `USnapInventoryDragDropOperation::GrabCell`, turned with the item when it is turned while held, and
    `USnapInventorySpatialPanel::ResolveLandingCell`.
  - An item from anywhere else lands by its top-left cell.
- A gamepad carry has no pointer and lands by its grab point either way.
- A box that would start off the grid is painted red.

⭐ **A release on any cell of an item acts on that item.**

- Merging into a stack, storing into a pouch, repairing with a kit and exchanging with a worn or stored item work from
  every cell the item covers, not only its top-left one (`ResolveReleaseTargetAnchor`).
- The drag paints that item's own cells in the fit color — a release that will merge or store is never shown red.
- Anywhere else the release places the dragged item, as above.
- A chest's item merges into the same stackable item that way; anything else under it is a placement (a swap, above),
  never a store, a repair or a mix.

⭐ **A chest's item lands where it is let go.** A drag out of a loot window, or a gamepad carry of a chest item, is taken
onto its landing (`USnapInventoryComponent::TakeFromLootContainerToSlot`), turned as it was turned while held (R, or
RT on a pad).

- The plan, the badge and the take ask one answer (`DescribeLootTakeToSlot`).
- A release painted red — a full bag, a landing off the grid, two items in the way — sends nothing and the item stays in
  the container.
- A full bag still tops up a stack of the same item with room: a release over it takes the units that fit straight from
  the container (no free square is needed), and the rest stays there.
- A release fills only the stack it is let go on; a take with no square named — Shift+click, a pad's X, Take All — fills
  the bag's stacks of the item with room in square order.

⭐ **Red is not sent, and nothing pops up.** A release of the grid's own items, or of a chest's item, that the drag
painted red for the cell under the cursor is consumed without a request.

- There is no refusal notice, and the item stays where it was.
- One red release does say why: a chest's item refused because the bag is full shows the take's own notice
  ("Your inventory is full." — a full bank or other container names the item instead), and still sends nothing.
- A grid placement refused on the server anyway (another player took those cells a moment earlier) is logged and raises
  no popup either: `FSnapRefusal::bQuiet` ([Refusals](refusals.md)).
- Every other refusal keeps its notice.

⭐ **In a pouch window laid out as a grid, too.** A release on any cell of a stored item that would store into it (a
nested pouch) or combine with it (a stack) goes to that item.

- That is `USnapInventoryStorageWidget::ResolveStorageReleaseTargetAnchor`, asked of the item's anchor square
  (`USnapInventoryStorageSlotWidget::ResolveStorageTargetAction`, the rules its `HandleDrop` runs).
- The drag paints that item's cells in the fit color.
- An item lands under its picture as in the bag (`ResolveStorageLandingCell`).
- With `Beside`, an item of that pouch and one from the bag land by the cell they were grabbed by, and the drag's
  picture of a pouch item is drawn at the item's box, as in the bag.
- A release of one of that pouch's own items that the drag painted red is not sent.
- Two limits:
  - With `Beside`, an item dragged from another pouch keeps its top-left cell under the cursor (moving between pouches
    re-places it at a turn the window does not predict).
  - A release of an item from outside the pouch is still sent when painted red — a store treats the square as a request
    and may put the item in the first free spot.
- A bag item grabbed off its top-left goes to its landing square when it fits there.
- The swap above, and Sort and the edge scroll below, apply in a pouch window too.

### Sorting a grid

On a grid, Sort re-packs it: `ArrangeGrid()` places every item again, **largest first**, each at the first cell its
shape fits in reading order.

- Items are turned only when their `Footprint` sets `bCanRotate`.
- Items of one size keep the order they stood in.
- The panel's Sort button and menu and the Sort key all send it (`Server_ArrangeGrid`).
- The view order a bag of squares uses cannot move anything on a grid, where every item is drawn at its own cells.
- The Sort key also re-packs every open pouch window laid out as a grid (`ArrangeStorageGrid`).
- Choosing **Unsorted** from the menu moves nothing on a grid, and one press re-packs each grid once, however many
  panels show it.

It is all or nothing:

- When one item would have nowhere to go, nothing moves and the refusal says so (`InventoryFull`; `CanArrangeGrid()`
  answers first, for a button's state).
- Nothing is created, destroyed, stacked or split.
- The log line counts the items and units on the grid before and after.

### Scrolling under a held item, and the hover outline

A grid taller than its `MaxVisibleRows` scrolls by itself while an item is held near its top or bottom edge, so a long
stash can be crossed without letting go.

- It scrolls within `EdgeScrollBand` of the edge (40 units, also counted just outside the edge).
- It scrolls at `EdgeScrollSpeed` (400 units a second; zero turns it off).
- A pouch window's grid scrolls the same way.

Hovering an item in the bag's grid draws one outline around its whole box (`HoverOutlineColor`, `HoverOutlineThickness`;
an alpha of zero turns it off).

- So a multi-cell item reads as one thing however many cells draw it.
- The outline follows the HUD's hovered grid item (`USnapInventoryHUDComponent::GetHoveredGridItem`) and hides while a drag
  is in flight.
- A pouch window laid out as a grid draws it too, around the stored item its squares report under the cursor
  (`USnapInventoryStorageWidget::SetHoveredStorageItem`; its own `HoverOutlineColor` and `HoverOutlineThickness`).

### Turning an item

`R` turns the item being dragged between its two orientations — upright and sideways, a quarter turn and back, never all
the way round (`USnapGridPlacementLibrary::NextPlayerTurn`).

- An item a turn cannot change (a 1 x 1, a full square) never turns.
- It is an Enhanced Input action (`RotateInputAction` on **Snap Inventory System - UI Core → Input → Enhanced Input**, firing once
  per press).
- The panel also reads the key directly, so a focused screen can turn an item without the action being mapped — the key
  as the player has it.
  - A rotate key the player rebound in the key settings works there too
    (`USnapUICoreSettings::IsRotateKeyFor`), and holding it turns the item once.

⭐ **With nothing held, `R` turns the item under the cursor where it lies** — to its other orientation at the same anchor
(`USnapInventorySpatialPanel::TurnHoveredItem`; the screen's own key route and the HUD's action reach it through
`USnapInventoryScreenWidget::TurnHoveredItemInPanels`).

- It is sent only when the player's grid shows the turned shape fits there.
- Otherwise the turned shape is painted for `RefusedTurnPlanSeconds` (0.5 s) — its blocked cells in the refusal color —
  and nothing is sent: no swap with the item in the way, no notice.
  - A client whose grid is a round trip behind the server's can still send a turn the server swaps or refuses.
- A pouch window laid out as a grid does the same with the stored item under the cursor
  (`USnapInventoryStorageWidget::TurnHoveredStoredItem` / `CanTurnHoveredStoredItem`, its own `RefusedTurnPlanSeconds`) —
  through its own key route, the screen's and the HUD's action.

⭐ **On a gamepad.** The shipped context also maps `Gamepad_RightTrigger` to the rotate action, so the right trigger
turns the carried item (carry mode — Accept picks an item up and puts it down).

- On a spatial grid the carry plans as a mouse drag does: every focus move paints the drop plan for the focused cell
  (`USnapInventorySpatialPanel::UpdateDropPlanAtCell`) — fit, swap or refused.
- Accept puts the item down through the mouse release's own commit (`PutDownCarriedAtCell`, then `CommitDropAtCell`).
  - The cell it was picked up by lands on the focused cell.
  - A turn made while carried is kept.
  - A release onto an item acts on that item.
- ⭐ A put-down the plan paints refused sends nothing and the item stays carried — move the focus and try again.
- A pouch window laid out as a grid carries the same way:
  - The focused cell's plan is painted.
  - Accept puts down through the window's own commit (`USnapInventoryStorageWidget::PutDownCarriedAtStorageCell`, then
    `CommitStorageDropAtCell`).
  - An item picked up there keeps the cell it was picked up by.
  - A refused put-down of the pouch's own item keeps it carried.
  - The right trigger turns the carried item over the window too.

⭐ **The picture turns with the item.** An item lying turned draws its picture turned — at the item's authored
orientation, rotated by its quarter turns.

- This holds in the per-cell drawing and the one-widget drawing alike, in the bag's grid and a pouch window's
  (`SnapGridIconClip::ResolveTurned` / `ResolveSpanning`).
- It used to be stretched into the turned box.
- How the picture fills the box is the grid's `IconDrawing` (below): by default it is fitted, never stretched.

⭐ **A picture never stretched.** `IconDrawing` (`ESnapGridIconDrawing`, on `USnapInventorySpatialPanel` and on a pouch
window's `USnapInventoryStorageWidget`) says how a grid draws an item's picture over its box, turned with the item either
way (`SnapGridIconClip::ResolveTurned` / `ResolveSpanning`):

- **`Fit`** (the default): The whole texture at its own aspect, as large as fits the box less the slot's icon inset,
  centered — never stretched, never cropped.
  - A square icon on a 1x3 sword is one cell big in the middle; art authored at the footprint's aspect (128 x 384) fills
    the box.
- **`FootprintRegion`**: Each icon read as a SQUARE texture with the item drawn in its middle at its footprint's aspect
  — a 1x3 sword in the middle third, the rest transparent — and that region filling the box (the aspect comes from the
  item's authored footprint).
  - Right for icons authored that way; a square icon drawn across its diagonal shows only its middle strip.
- **`Stretch`**: The whole texture stretched over the box, the drawing a grid had before.

Every square widget elsewhere — a squares bag, the hotbar, an equipment slot, a tooltip, the drag picture — draws the
whole square, so an icon authored for the footprint region reads letterboxed there.

- The shipped `WBP_ExampleSpatialPanel` and `WBP_ExampleStorageWindow` draw `FootprintRegion`.
- Every example item that is not square has its icon drawn that way — the iron sword, axe and greatsword, the gold bar,
  the carved totem, the repair kit and both cuirasses.

⭐ **The held picture follows.** A drag's default picture starts at the turn the item lies at
(`USnapInventoryPanel::GetDragVisualShapeForItem`).

- Every turn while held re-sizes it to the turned box, turns its picture (`USnapInventoryDragVisualWidget::SetPictureTurn`)
  and keeps it centered on the pointer (with `Beside`, keeps the grabbed cell under the cursor —
  `USnapInventoryDragDropOperation::ApplyHeldTurnToVisual`).
- A Blueprint picture (`CreateDragVisual`) is left as it is.

Only an item whose `Footprint` fragment sets `bCanRotate` can be turned; `CanRotateHeldItem()` answers whether a rotate
prompt should be shown at all.

⭐ **One switch turns it all off.**

- `USnapInventorySettings::bAllowItemRotation` (on) covers every way an item turns:
  - A turn the player asks for.
  - The first-fit placement that turns an item to make it fit (a pickup, a store into a pouch, a layout change, Sort).
  - The swap that turns the other item into the space it is given.
- Off, none of them turns anything:
  - An item keeps the turn it has, a new one is placed upright, and the server refuses a move that would change a turn.
  - `USnapInventoryComponent::GetItemFootprint` reports `bCanRotate` false while it is off — the one place every
    placement search and turn gesture asks — so a project's own UI that reads the footprint follows the switch too.

⚠ **A drag that arrives from another container cannot be turned on the way in.**

- `MoveItemWithRotation` is the only mutator that writes a turn, and it moves an item WITHIN one container.
- So a drag out of a pouch, a vendor or a trade offer is dropped unturned — and `CanRotateHeldItem()` answers false for
  it, so no prompt appears for a gesture that could not be carried out.
- Turn the item after it lands.

A turn is applied only when it changes during the drag.

- Any other release of an item already turned is handled as an upright item's is — a turned stack released on its own
  kind merges — and a move keeps the turn.
- Turned back upright while held, it lands upright.
- A spatial pouch's window follows the same rule.

### Grid width is the MODEL's

`USnapInventoryPanel` has a `GridColumns` property and `USnapInventoryComponent` has `GridWidth`. In a spatial container
**the model wins**:

- The panel reads `GetGridWidth()` and `GetEffectiveGridHeight()` and ignores `GridColumns` entirely, because a
  placement is decided server-side and a widget cannot be allowed to disagree with it.
- A panel whose `GridColumns` differs reports the mismatch once, naming both numbers, and carries on with the model's.

### Seeing it in the demo

The demo map starts in `Slots` mode, which is what a project gets by default.

1. Step on **BAG LAYOUT** in the demo's FEATURES wing to switch every player's bag to the spatial grid for the play
   session (see [the demo map](demo-map.md#the-features-wing-switch-a-plugin-feature-for-this-session)).
2. Step on it again to switch back.

The pad changes the one value a project sets to use the grid everywhere, and nothing else:

- The paper-doll screen stays up, and its bag area turns from squares into the grid by itself (see
  [A screen that draws both layouts](#a-screen-that-draws-both-layouts)).

| Where | Set |
|---|---|
| `[/Script/SnapInventorySystem.SnapInventoryComponent]` in your `DefaultGame.ini` | `LayoutMode=Spatial` |

It is reversible and is not a plugin default.

### Changing the layout while a container holds things

Every placement is read through `LayoutMode` — a number is a square in `Slots` and a cell in `Spatial` — so writing it
under a container that holds items would leave each one meaning something else.

Change it with `USnapInventoryComponent::SetLayoutMode(NewMode)` instead (server only; `TrySetLayoutMode` returns the
refusal, and `CanSetLayoutMode` asks without changing anything). It plans every new placement first, and then either
applies them all or changes nothing:

- **To `Spatial`:** Each item takes the first cell its footprint fits, in the order the items stand now, turned when it
  may be turned and has to be.
  - If that leaves an item out, it plans again largest first.
  - Refused (`InventoryFull`) when neither order fits everything.
- **To `Slots`:** The items keep their reading order as squares 0, 1, 2, ... and every turn is undone.
  - Refused (`InventoryFull`) when there are more items than the Slots capacity (the base slot count plus bought and
    temporary slots; unlimited when the base is 0 or less).
- **A pouch with a grid goes with it.** A storage item's `StorageWidth` grid is used only in a `Spatial` container.
  - So the contents of every storage item with a grid — nested ones too — are re-placed inside its own grid by the same
    two rules.
  - One pouch whose contents would not fit refuses the whole change.
  - A pouch without a grid is not touched.

Details:

- Nothing is created, destroyed, stacked or split, and nothing leaves the container.
- Equipment, hotbar bindings and trade offers name items, not squares, so none of them changes.
- `LayoutMode` replicates with the contents (it follows `ReplicationScope`).
- The change ends with one `OnInventoryChanged` broadcast with no item: re-read everything.

### A screen that draws both layouts

A `USnapInventoryPanel` draws squares and a `USnapInventorySpatialPanel` draws a grid. A screen may hold **both**, bound to the
same container:

- The one that draws the container's `LayoutMode` shows, the other hides.
- They switch the moment the layout changes, with the screen open (`SetLayoutMode` on the server, the replicated layout
  on a client).
- Bind the two panels exactly as you bind one.

The example paper-doll screen (`WBP_ExampleInventoryScreenCharacter`) does: its bag area is squares in `Slots` and the
grid in `Spatial`, with the equipment, the Spells grid and the bank beside it unchanged.

- **A panel alone with its container is never hidden.** The rule needs a partner that draws the layout — bound to the
  same container, in the same screen, not hidden for want of a container, and not set `Collapsed` or `Hidden`.
  - A lone squares panel over a `Spatial` bag keeps drawing it, one square per item, as before.
- **Bind both before the squares panel builds its squares.** A screen binds its panels one at a time, and until the grid
  panel has its container the rule cannot hide the squares panel.
  - Bound and built first over a `Spatial` bag, it builds a square for every cell of the grid, only to drop them a
    moment later.
  - The example screen binds the squares panel, then the grid panel, then calls the squares panel's `BuildGrid`.
  - A squares panel the rule is about to hide — its grid partner still waiting for a container, or a cached screen
    opened again after the layout changed — does not report its `GridRows` shape.
  - A hidden grid panel builds no cells even when it was bound before it was constructed (the vendor screen binds its
    panels that way).
- **A hidden panel does nothing.** It hides the way a panel with no container does (its root collapsed, never its own
  `Visibility`), is not a window, is skipped by keyboard focus and by Sort, and does not refresh.
  - A squares panel drops its squares while hidden and builds them again, at the container's size, when it shows.
  - A grid panel builds no cells over a `Slots` container and does not log the grid-panel-over-`Slots` Warning, which a
    grid-only screen still gets.
- **The built-in buttons follow it.** CRAFT, DROP INVENTORY and the vendor's SELL JUNK go into the counter row
  of the first panel of each kind, so they are on whichever panel shows.
  - Give the grid panel a counter readout (`CounterValueTextWidgetName`) for them, as `WBP_ExampleSpatialPanel` has.
- **Give the two panels the same slot padding, or none.** A hidden panel's box slot keeps its padding; the example
  screen puts the gap under the bag on the Spells panel's slot instead.
- `USnapInventoryPanel::IsHiddenForLayout` says which one is hidden, and `bFollowContainerLayout` (on) opts a panel out.

`WBP_ExampleSpatialScreen` stays as an example of a grid-only screen (its squares panel is authored `Collapsed`).

- A screen built for one layout works as it always did.
- After a change, point `InventoryScreenClass` (**Snap Inventory System - Inventory UI → Screen**) at a screen for the new layout and call
  `USnapInventoryHUDComponent::DiscardCachedScreens`, which builds every screen again on its next open.

## The grid

A container is a fixed number of numbered slots.

- `MaxNumSlots` sets it (30 by default).
- `GetEffectiveMaxNumSlots()` adds any bonus slots bought with a `CapacityUpgrade` item.

In `Spatial` mode the grid **is** the capacity:

- `GetEffectiveMaxNumSlots()` answers `GridWidth * GetEffectiveGridHeight()`.
- A slot index is a cell index counted left to right and top to bottom.
- `MaxNumSlots` is not consulted.
- A bought capacity upgrade adds whole rows rather than loose cells, so the grid stays rectangular.

An item occupies one slot regardless of how many units it holds, so a stack of 10 potions and a single sword each cost
one square.

- In `Spatial` mode that is still true of the UNITS — a stack of 10 potions is one placement — but the placement covers
  whatever the item's footprint covers.

### Locked squares — the room an upgrade would add

With `bShowUnboughtBagSquares` on and the project's upgrade items listed in `UnboughtSquareUpgradeItems` (both on
**Snap Inventory System - Inventory UI → Screen → Unbought Squares**), the main bag also
draws the squares those items could still add, after its own, each with a lock picture.

- They are not slots: the bag's capacity, `GetEffectiveMaxNumSlots()` and the panel's `GetGridSlotCount()` are unchanged,
  and nothing can be put there. `USnapInventoryPanel::GetUnboughtSlotCount()` says how many follow.
- In `Spatial` mode they are whole rows below the grid, as a bought slot is.
- A temporary grant moves them along: they start after every square the bag has, borrowed ones included.
- More in [the README](../../README.md#carrying-more--purchasable-capacity-upgrades).

### Per-instance capacity — a chest that is a different size from the chest beside it

`MaxNumSlots` is `config`, so it is read into each CDO **per class**, and on its own that has a hard ceiling: two
*instances* of one class cannot differ.

- `SlotCountOverride` and `CarryWeightOverrideKilograms` lift it.
- Both are `EditAnywhere`, so a designer types a number into a placed actor's details panel — or into a Blueprint's
  class defaults.
  - That is how the Spells and Shards containers, two Blueprints of one native class, each have a size of their own.
- Both are replicated.

```
SlotCountOverride                  per-instance base slot count; -1 = unset, use MaxNumSlots
CarryWeightOverrideKilograms       per-instance base carry weight; negative = unset
GetBaseMaxNumSlots()               the override when set, otherwise MaxNumSlots
SetMaxNumSlots(N)                  set it at runtime (server); -1 clears
SetMaxCarryWeightKilograms(Kg)     the same, for weight; negative clears
```

⚠ **The unset value is `-1`, not `0`.**

- `MaxNumSlots <= 0` already means *unlimited* everywhere in this plugin, so `0` is a real override meaning
  "this instance is unbounded" and cannot also mean "unset".
- A validator warns if either field is given a value below `-1`.
- It warns again if a slot count is set on a `Spatial` container, where the grid geometry is the capacity and the number
  is never read.

### Temporary slots

A buff or a quest can lend capacity without buying it:

```
AddTempSlots(Source, Count)        grant N slots under a key; returns the new temporary total
RemoveTempSlots(Source)            drop that key's grant
GetTempSlotTotal()                 the sum of every live grant
```

- Grants **stack across sources** (two buffs both count) and **replace within one source** (re-granting under a key you
  already hold is a refresh, not a second buff).
- They are deliberately **not saved** — a buff that outlives its session is not temporary.
- The purchased `CapacityUpgrade` path is unchanged and unrelated: that one is tiered, highest-wins, permanent, and
  persisted.
- `Source` must be a real registered tag; an empty one is refused, because the tag *is* the key and two callers passing
  the default would share one grant.

⚠ A temporary grant does **not** widen a `Spatial` container, and `AddTempSlots` logs a warning when called on one.

- A spatial slot index is a cell, so a grant that later expires could leave items at coordinates the grid no longer has.

### Over capacity

Capacity can go **down** — a `SetMaxNumSlots` shrink, or a temporary grant expiring. When it does,
**nothing is destroyed, dropped or moved.**

- The items past the new limit stay exactly where they are.
- The container refuses new items until the player moves some out.

```
IsOverCapacity()          holding more than the capacity allows
GetOverCapacityCount()    how many entries are over the line
GetDisplaySlotCount()     how many cells a UI must build to SHOW everything held
```

`GetDisplaySlotCount()` is the one a panel wants, and it is `max(capacity, highest occupied slot + 1)`.

- Building a grid to the *capacity* would give the over-capacity items no cell at all — undestroyed but invisible, under
  a rule that asks the player to move them.
- `USnapInventoryPanel` uses it on both its build and its grow path.

```
GetItemInSlot(Index)          the instance in a slot, or null
GetSlotIndexForItem(Instance) where an instance sits, or INDEX_NONE
GetAllItems()                 every instance in the GRID -- not inside pouches
```

For a spatial container there are three more, and they answer in cells:

```
IsSpatialLayout()                       whether this container is laid out as a grid
GetGridWidth() / GetEffectiveGridHeight()   the grid, upgrades included
GetItemFootprint(Definition, OutShape)  the item's shape, or false when it has none
BuildOccupancyMap()                     what holds each cell, by anchor index
ResolveGridPlacement(Definition, Target, Rotation, Ignore)
                                        may this item go there? -- the one placement rule
ResolveGridSwap(Mover, Target, Rotation, Blocker)
                                        a release onto one other item: where both go
FindPlacementForItem(Definition)        where would it go if nothing said?
HasRoomForItem(Definition)              room for one more stack OF THIS ITEM
```

One mutator names a turn:

```
MoveItemWithRotation(Instance, NewSlotIndex, Rotation)   turn it as it moves
```

It is the only door through which a caller turns an item.

- Every other mutator preserves whatever turn an item already had, because a move and a multi-item drag are changes of
  place.
- The one exception is a swap's other item, which the swap rule may turn so it fits the space it is moved into.
- Moving to the slot an item is already in with a different turn rotates it where it stands.

`ResolveGridPlacement` is the single answer to "may this go here", in both modes.

- A drop preview asks the same function, so a UI cannot offer a placement the server will refuse.
- It returns the cells the item would cover and, when it does not fit, which of them are blocked and why.

There is one rule and two ways in:

- `ResolveGridPlacement` reads the container's current occupancy and can set aside up to two existing items — the item
  being moved, and the one it is swapping with.
- A gesture that moves SEVERAL items at once needs all of them set aside together, and each step judged against a grid
  the earlier steps have already claimed.
  - So it supplies its own occupancy map to `ResolvePlacementIn` and marks each accepted step with `ReserveCells`.
- Both routes run the same code; they differ only in which grid the question is asked about.
- Build a drop plan over a selection with that pair rather than writing a second rule.

⚠ `GetAllItems()` is grid-only. "Held here" in this plugin is the **disjunction** of the grid and the pouches:
`GetSlotIndexForItem(I) != INDEX_NONE || GetStorageOwnerOfItem(I) != nullptr`.

- Code that asks only the first question makes a pouched item invisible to whatever it is doing.

## Adding and removing

```
AddItem(Definition, Count)              -> USnapItemInstance*
AdoptInstance(Instance)                 take an existing instance in
RemoveItem(Instance, Count)
```

⚠⚠ **`AddItem` clamps to capacity and can add fewer units than you asked for.**

- It returns non-null for a partial add, so `if (!AddItem(...))` reads a partial add as a complete success.
- Where the remainder matters — a purchase, a craft yield, a loot take — compare the count you asked for against what
  actually landed, and decide what to do with the rest.

`RemoveItem` **refuses a loaded container**: a pouch with contents cannot be removed until it is emptied.

- Use `CollectAllFromStorage` first, and expect to need more than one pass for nested pouches.

## Stacking

Stacking is a property of the `Stackable` fragment, not of the container.

- `MaxStackSize` caps one stack.
- An item with no fragment does not stack at all.
- `MaxUnitsPerContainer` caps the total units of one item a single container may hold, across all of its stacks.
  - It is separate from `MaxStackSize`. With both at 10, the item fits in one full stack and the eleventh unit is
    refused.
  - Negative (the default) means no cap; 0 means a container may hold none of it.
  - A collect is clamped to it: five units with two of allowance left takes two and leaves three.
  - Known limit: clamped to zero, an item that is also equippable can't go straight onto a free equipment slot either,
    although worn items are outside every container's cap. No shipped definition is both.

```
SplitStack(Instance, CountToSplit, TargetSlotIndex)
```

⚠ `TargetSlotIndex` must name an **empty** slot.

- A negative index is refused, and so is an occupied one — splitting onto an occupied slot would be a merge or a swap,
  which are deliberately separate operations.
- There is no "find me a slot" overload; scan `GetItemInSlot(i)` for the first null yourself.

### The split window (Ctrl+drag)

Ctrl+drag part of a stack (or Ctrl+click a currency) and a window asks how many: a slider from 1 to the ceiling, - and +,
a readout "<amount> / <ceiling>", Cancel and Confirm.

- `StackSplitWidgetClass` (**Snap Inventory System - Inventory UI → Screen**) names the window: the example's by default. Empty turns it off, and Ctrl+drag then moves one
  unit.
- Shift (half) and a plain release (all) are unchanged. The grid's Ctrl+click is the multi-select toggle.

**The ceiling** is what the mutator that will run accepts, not the stack (`USnapInventoryStackSplitWidget::ResolveSplitCeiling`).
N is the stack's size, R the room left in a matching stack:

| Destination | Mutator | Ceiling |
|---|---|---|
| An empty bag square (from the bag, a worn slot or a pouch) | `SplitStack` / `TransferEquippedToInventory` / `TransferStoredToInventory` | N - 1 |
| An empty equipment slot (from a worn slot or the bag) | `SplitEquippedStack` / `TransferInventoryToEquipped` | N - 1 |
| An occupied, matching slot with room R | `MergeStacks`, `MergeEquippedStacks` or the same transfers | The smaller of N and R |
| A square in a container (pouch, chest) | `StoreItemPortionInStorage` | Empty square N - 1; matching square the smaller of N and R |
| Your own trade offer (bag items only) | `Server_OfferItemsToTrade` | N - 1 |
| The floor (an item or a worn item) | `DropItemStacksToWorld` / `DropEquippedToWorld` | N |
| The floor (a currency) | `DropCounterToWorld` | The balance |

- N - 1 is not an off-by-one: taking every unit is a move, which the split mutators refuse. So a stack of 2 onto an
  empty slot has one legal amount (the slider's ends meet).
- No window opens for a stack of one, for a worn item going into a container (`UnequipToStorage` takes no count), or for
  a pouched item going to the floor or onto the body.
- **Two more caps:** Units coming into the bag from a worn slot or a pouch count against the per-definition unit
  allowance. The RPC bounds (`FSnapNetLimits`: count and slot index) are never exceeded, since a failed `_Validate` closes
  the connection.

**How it behaves:**

- **It is modal.** A full-viewport, hit-testable scrim (`ScrimWidgetName`) takes every press, and a press outside the
  body (`BodyWidgetName`) cancels, as Escape and Cancel do.
- **It keeps keyboard focus**, so Escape cancels the split rather than closing the bag, and the bag's own keys don't run.
- **It opens at the cursor**, centered on it horizontally and `StackSplitCursorOffset` below it, clamped inside the
  viewport, at amount 1.
- **Its body grows to hold what it shows.** A body of a fixed size in a Canvas Panel is made taller while its content needs
  more room (a long description, the gamepad's amount keys), never shorter than its size in the Widget Blueprint, and kept
  inside the viewport. An auto-sized body is left alone.
- **Its parts are found by name**, defaulting to the example's: `SplitScrim`, `SplitBody`, `SplitItemIcon`,
  `SplitItemName`, `SplitItemDescription`, `SplitAmountSlider`, `SplitAmountText`, `SplitMinusButton`,
  `SplitPlusButton`, `SplitConfirmButton`, `SplitCancelButton`. A missing scrim or body logs a warning once.
- **Confirm checks the ceiling again.** If the source ran short the player is told (`NotEnoughLeft`); if the destination
  filled up they are told the rest stays (`NotAllFit`).
- **One Confirm per opening.** A currency Confirm also waits `ConfirmInFlightGraceSeconds` for the balance to move before
  another is accepted (`RequestInFlight`).

## Moving

```
TryMoveItem(Instance, NewSlotIndex)   within the container; an occupied target swaps with its item
```

Every gesture the UI offers routes through one plan function, so that the drag preview, the drop and the server's
re-derivation cannot disagree.

**Moving several items, and splitting on a drop:**

- **A drag of a selection keeps its shape.** The lowest selected slot lands on the square you release over, and every
  other item keeps its offset from it: selecting 0 and 2 and dropping on 6 moves them to 6 and 8.
  - An occupied target square swaps with its item; an empty one is a plain move.
  - It is all or nothing: if any item would land outside the grid, nothing moves.
  - The exception is a selection released on a storage item (a pouch): it goes inside, filling what fits.
- **One stack released with Shift held moves half**, rounded down (7 moves 3 and leaves 4): onto an empty square as a new
  stack, onto a matching stack as a merge, and from the bag onto a pouch, into the pouch.
- **With Ctrl held, the split window asks how many** (see [The split window](#the-split-window-ctrldrag)). With
  `StackSplitWidgetClass` empty, Ctrl moves one unit instead. Ctrl wins when both are held.
- Onto any other item the modifier is ignored, and the two swap.
- A multi-selection drop ignores the modifiers, because Ctrl and Shift also build the selection when you click.

## Rearranging — sort, compact, stack all

The sort a player sees is a **view**.

- `USnapInventoryPanel::ApplyDisplayOrder` re-orders the slot *widgets* and writes no slot index, so the grid's real layout
  is untouched.
- Every gesture that addresses a literal slot — drag and drop, split, store into a pouch, shift-click equip — keeps
  working unchanged.

That is not a limitation. It is the answer to a measured one:

- An earlier revision sorted by mutating the inventory, and it cost **35.8-40.0 ms** per sort.
- A sort that changed *nothing* still cost **5.9 ms**, because the expense was the per-changed-entry replication and
  broadcast cascade rather than the comparator.
- Rebuilt as a view it came to **0.342 ms**.

**How the view behaves:**

- A fresh panel draws Unsorted, which is slot order. The chosen sort lasts as long as the panel (the HUD keeps its panels
  for the session) and is never saved.
  - It is re-applied on every refresh, so a sorted bag stays sorted as items come and go.
- **Placing an item by hand returns the panel to Unsorted** (a drop, a split, a merge, a store into a pouch), so the sort
  never sweeps a manual placement away. A pickup or any other change from the world does not.
- **While a sort is shown,** a Shift range selects by slot index, so it can pick items scattered on screen. A multi-item
  drop lands on consecutive slot indices, which the sort may draw apart.
  - Nothing is lost, and the first manual placement puts the grid back in slot order.

Three server-authoritative operations sit beside it on `USnapInventoryComponent`.

- All three rearrange and none of them can change *what* the container holds.
- So none needs the `bAllowClientRequestedAdd` gate that `Server_AddItem` does.

| Call | What it does | Gate |
|---|---|---|
| `SortInventory(Mode, bDescending)` | Rewrites every occupied entry's slot index into `Mode`'s order | **`bServerSortWritesSlots`, which ships OFF**; refused on a spatial grid |
| `CompactInventory()` | Moves every item down to close the gaps, keeping the order it is in | Refused on a spatial grid |
| `StackAllInventory()` | Merges every partial stack of the same item, lowest slot first | None |

- Each has a `Try...` form that returns an `ESnapRefusal` instead of a bool, and a `Server_` form a client may call.
- The sort also has `CanSortInventory()`, which a UI can ask before offering a button.

### The sort's flag, and why it is off

The `bServerSortWritesSlots` flag (**Snap Inventory System - Inventory → Sorting**) buys one thing and costs one thing:

- **What it buys:** A **persisted** order. The slot indices themselves change, so the arrangement survives a save,
  reaches every client, and is what any other reader of the container sees.
- **What it costs:** The cascade above, on every sort.

Turn it on if the order is part of your game's state; leave it off if it is something the player is looking at.

- With it off, `SortInventory` refuses with `SortingDisabled` and the panel's dropdown behaves exactly as it always has.
- A project that upgrades and changes nothing keeps what it had.

⭐ Both sorts produce the **same** order, because both ask the same code.

- `ESnapInventorySortMode` lives in `Inventory/SnapInventorySortTypes.h` — a leaf header a non-UI class may include.
- The order itself lives in `Inventory/SnapInventorySortOrder.h`.

### Why a spatial grid refuses

In a spatial layout a slot index is an **anchor**: it carries a footprint and a rotation.

- Renumbering the occupied anchors therefore does not permute the *items*; it produces overlapping or out-of-bounds
  placements.
- `SortInventory` and `CompactInventory` answer `WrongLayout` rather than doing it.
- Packing a spatial grid is a different feature.

`StackAllInventory` is available in both layouts.

- Merging stacks removes anchors and renumbers nothing.
- So a spatial grid is left with strictly fewer occupied cells and every survivor exactly where it was.

### Sort is not compact

They are two requests and the plugin keeps them apart.

- `SortInventory` permutes the items among the slots **already in use**, so a project that keeps deliberate holes in its
  grid still has them afterwards.
- `CompactInventory` is the one that closes them.

`StackAllInventory` goes through `MergeStacks`, so every rule that mutator already enforces applies unchanged rather
than being restated somewhere it could drift:

- Same definition.
- The stackable fragment's cap.
- A partial merge moving what fits.

It conserves every unit of every definition; what changes is how many entries hold them.

## Searching

A search box narrows what is **drawn**.

- Nothing moves, no slot index is written, nothing replicates, and `GetAllItems()` answers exactly what it did before.
- That is the same relationship the view sort above has to the container, and for the same reason.

It is off until a project asks for it.

- Turning on `bShowSearchBox` on a panel (or on a storage window) builds a `UEditableTextBox` into the container named
  by `SearchBoxContainerName`, and narrows as the player types.
- A project that would rather build its own box can skip all of that and call `SetSearchQuery` directly; that is the
  function the built-in box calls.

| Call | On | What it does |
|---|---|---|
| `SetSearchQuery(Query)` | `USnapInventoryPanel` and `USnapInventoryStorageWidget` (each overrides the virtual hook on their shared base, `USnapItemGridHost`) | Narrows to items whose definition's `DisplayName` (not an affixed or learned name) contains `Query`, trimmed |
| `GetSearchQuery()` | Both | What was last set, trimmed |
| `DoesItemPassSearchQuery(Instance)` | Both | The predicate on its own |
| `IsAnySlotNarrowingActive()` | Both | Is a filter row or a query currently hiding anything? |

### It covers the bag AND the containers, and that is one promise rather than two features

The grid and a pouch window are both "things I am carrying", so a player who learns to type into one must not find the
other answering differently.

- What makes that a property of the code rather than an intention is that the MATCH and the TRIM both live in
  `Inventory/SnapInventorySearch.h`, which both surfaces call.
- Neither owns a copy that could drift.

⚠ **The queries themselves are independent**, deliberately.

- Each window narrows what *it* is looking at, so a query that matches nothing in a pouch leaves the bag behind it
  exactly as it was.

⚠ The **vendor's shop list and the craft recipe list are not covered**.

- Those are server-fed and already carry their own tag filter bars.
- A text box over them would be a different feature with a different source of truth.

### What it matches, and what it does not

The display name, case-insensitively, as a substring.

- Not tags: the filter bar beside it is already the tag question, configured by a project rather than typed.
- One box meaning both would make "armor" match a word in a name *and* a category with no way for a player to tell
  which.

A query and a filter row **compose**: an item must pass both.

- A player who has picked a filter and then types is asking a narrower question, not a different one.

Surrounding whitespace is trimmed, and a query of nothing but whitespace is *no search* rather than a search for a space
— so a stray keypress cannot blank a container.

⭐ Search and sort compose by construction, because the display order is derived over the **whole** grid rather than over
the visible subset.

- A narrowed-out slot still takes its place in the order.
- Sorting and then searching cannot disagree with searching and then sorting.

### A selection never outlives its visibility

This is the part worth knowing if you extend it.

- A selection feeds a drag's payload and the vendor's SELL subjects, so a selected-but-hidden slot would let a player
  move or sell something they cannot see.
- A collapsed slot receives no press of its own, so a direct click is safe by construction.
- But a **shift-range sweeps** across slots nobody clicked, and a query typed *after* a selection has to prune it.
- Both surfaces do both, and `IsSlotVisibleUnderFilter` (bag) / `IsStorageSlotVisibleUnderSearch` (storage) are the one
  predicate every selection rule asks.

⚠ `CollectAll` still collects **all**. A narrowing changes which squares are drawn; it never changes what a mutation
means.

## Pouches — containers inside the grid

An item carrying the `ItemStorage` fragment is a container in its own right. Open one and you get a floating window with
its own slots.

These are on **Snap Inventory System - Inventory → Screen | Storage**:

| Setting | Default | Effect |
|---|---|---|
| `bAllowNestedStorageItems` | On | A pouch may go inside a pouch |
| `MaxStorageNestingDepth` | 2 | How deep |
| `bDestroyEmptiedStorageItems` | **Off** | Destroy a grid pouch the moment its last item is taken |

⚠ `bDestroyEmptiedStorageItems` fires on a **transition**, not a state: only a pouch that held something and now holds
nothing is destroyed.

- A pouch picked up empty off the floor, or carried around empty all day, is never touched.
- It ships off because "my content deleted itself" is a far worse first impression than a little clutter.
- A *nested* pouch is never auto-destroyed either way.

### The pouch window

Opening a pouch shows a floating window with its own squares, one window per pouch. A pouch inside a pouch opens a
second window.

- **It is a window, not a dialog.** It blocks nothing, so items drag between it and the bag.
- It always drags and closes, whatever `bAllowWindowDragAndClose` says.
- **Escape (or B) closes every pouch window at once**, topmost first, before the bag.
  - A window above them (the item menu, or the sell, loot or craft window) closes first.
  - The X on a window closes just that one.

**What a release on a pouch square does.** In this order, the first match wins:

- Onto a container item: it goes inside. A container that can't take the item falls back to the swap.
- An item onto an item: a mix or a repair kit.
- A merge the spoilage policy refuses is refused (never under the shipped `Average` policy).
- Shift: a portion (half). Ctrl: the split window.
- A merge with a matching stack.
- Else a swap, or a move to the empty square.

Also:

- Dragging a pouch onto another pouch nests it while `MaxStorageNestingDepth` allows; else the two swap places.
- A swap onto the container the dragged item lives in is refused.
- A bag selection dragged in goes in as many as fit, into ascending free squares.

**Dragging an item out of a pouch:**

- It moves the whole stored stack: onto a bag square, an equipment slot, the floor or another pouch.
- Holding Ctrl while releasing onto a bag square, or onto another pouch, opens the split window, so you choose how many.
  Ctrl onto an equipment slot or the floor opens no window (those moves take no count).
- Shift is not a portion onto the bag: a Shift-drag from a pouch onto a bag square moves the whole stack. Between pouch
  squares, Shift moves half.
- A plain release onto a stack of the same stackable item merges into it. A different item, or a stack already at its
  `MaxStackSize`, swaps places.

**Selecting in a pouch window:**

- Selection follows the bag grid's rules: a click, Ctrl+click, a Shift+click range.
- Selecting in a pouch clears the bag's selection and every other pouch window's, so a shop's SELL has one answer.
- Shift+click on a pouch square, with nothing else selected (or only that square), takes the item out into the bag. At
  an open shop it sells the item instead, and in an open trade it offers it.

**The rest of the window:**

- The `bShowSearchBox` property gives a pouch window its own search box (off by default). Each window narrows only itself, by the same
  matching as the bag's, and a hidden square can't be selected, sold or dragged.
- Collect All (labeled Take All) moves what fits into the bag and leaves the rest. It is disabled on an empty pouch and
  hidden in a view-only window.
- A view-only window (the trade partner's half, or a pouch of yours in escrow) shows the contents and refuses every
  change. The server's own checks are the boundary.
  - A pouch of yours leaves escrow when you take it back out of the offer or the trade ends. Its open window is then
    writable again, with Collect All back. The partner's half stays view-only.

## Counters — currency and anything like it

Some things do not deserve a square. A definition whose `ContainerRouting` fragment marks it a **counter** is added to a
numeric total instead of the grid. The counters live on the player's wallet, `USnapWalletComponent`, a component beside
the main bag (`AddToCounter`, `CanPayCurrencyPrice`, `PayCurrencyPrice`); the bag keeps `GetCounterValue` and
`DropCounterToWorld`:

```
AddToCounter(CounterTag, Delta)    GetCounterValue(CounterTag)
CanPayCurrencyPrice(Price)         PayCurrencyPrice(Price)
DropCounterToWorld(CounterTag, Amount)
```

- `CounterTag` defaults to `SnapInventorySystem.Counter.Currency`, a natively-declared tag.
- A project with its own currency re-points it on **Project Settings → Plugins → Snap Inventory System - Inventory → Counter
  Readout**, which saves `CounterTag=` under `[/Script/SnapInventorySystem.SnapInventorySettings]` in `DefaultGame.ini`.

⚠ A counter is refused by every container in every admission mode.

- The counter test sits **above** the admission switch, so it cannot be routed into a grid by loosening a container's
  rules.

**To let a currency be dropped** (by a player or by a death drop), list a Counter-mode item for its tag in
`CounterItemDefinitions`, in your project's `DefaultGame.ini` under `[/Script/SnapInventorySystem.SnapInventoryComponent]`.

- It ships empty, so currencies are collect-only.
- An entry is a soft path, not a package reference, so also cook the folder the item is in. Otherwise the drop works in
  the editor and fails in a packaged build.
- When nothing resolves, the balance stays with the player and the log warns once per currency.

**Dropping currency from the readout:**

- Clicking the counter readout, or one entry of the wallet row, drops that currency in the world as one pickup, with no
  confirmation.
  - A plain click drops the whole balance. Shift (the `SplitHalf` gesture) drops half, rounded down.
  - Ctrl (`SplitSingle`) opens the amount window to choose how much. Ctrl is asked first, so Ctrl+Shift opens the window.
- A wallet entry names its own currency. The single readout needs `USnapInventorySettings::CounterTag` pinned.
  - Unpinned, the click is refused (logged at Verbose), since the number then shows whichever counter moved last.
- A gamepad's Accept on a wallet entry opens the amount window too (`USnapInventoryPanel::OpenCurrencyAmountWindow`).
- A second click at the same balance while the first request is in flight is refused, so a double-click can't drop
  twice (`CounterDropRequestGraceSeconds`, 1 s by default, on the panel).
- The panel's `bCounterClickDropsToWorld` (on by default, in its class defaults) turns the gesture off.

## Routing between containers

`ContainerRouting` names which container an item belongs in, and
`USnapItemLibrary::GetItemContainerTag(Definition)` reads it.

- The collection router sends a collected item to the container that claims its tag, falling back to the main bag.
- Which container is the main bag is decided by an explicit marker, `bIsMainInventory`, true on the base class and
  cleared in the constructor of every other container the plugin ships.
- See [extending](extending.md) — this is what lets you subclass the component without losing the hotbar.

### Naming a container without writing C++

`GetContainerTag()` is what every routing and lookup path asks, and the base class answers it from one field (a C++
subclass may override it instead).

The plugin's own Spells and Shards are [custom containers](custom-containers.md) that set that field in their class
defaults:

```
ContainerTagOverride      set this and GetContainerTag() answers it
```

- It is `EditAnywhere`, so it works in a Blueprint subclass's class defaults *or* on a single placed actor — an Armory
  chest and a Pantry chest can be two instances of one class.
- A C++ override still wins outright.

⚠ **Two containers on one actor must not answer the same tag.**

- Routing takes the first match, so a duplicate is a silently halved inventory — half the routed items in one, half in
  the other.
- A validator warns at edit time; nothing can catch it at runtime, because each component can only see itself.

`USnapInventoryPanel::ContainerTag` is the matching half on the UI side: set it and the panel resolves its own container
through `USnapInventoryLibrary::GetInventoryContainer` and binds to it.

- An explicit `SetInventoryComponent` always wins, so a vendor or trade screen showing a specific container is never
  overridden by a tag left in the widget.

**A panel with nothing to show hides itself.** It collapses its root widget instead of drawing an empty grid with a
`0/0` capacity. That covers:

- A panel bound to nothing — `SetInventoryComponent(nullptr)`, which is what the example screens hand their Spells panel
  in a project without `GameplayAbilities`.
- One whose `ContainerTag` its owner does not have when it is built.

Any bind to a real component shows it again.

- **A container attached later is found.** While it is on screen and unbound, a panel with a `ContainerTag` listens to
  `USnapInventoryComponent::OnAnyContainerBeganPlay()`.
  - That is a native event every container fires at the end of its `BeginPlay` — on the server and on every client.
  - The panel binds its own container when one appears: a registry row, a `GameFrameworkComponentManager` extension, a
    PlayerState that arrives late.
  - It needs the event because a hidden panel is not painted, and Slate ticks only what it paints.
- ⚠ **A panel with no tag cannot look for its own container.** It stays hidden until whoever bound it binds it again;
  the example screens do that every time they open.
  - Give a panel a `ContainerTag` if it should come back by itself while its screen is open.
  - The example container panel (`WBP_ExampleContainerPanel`) carries `SnapInventorySystem.Container.Spells`, so it
    does.
- **On a client too.** A PlayerState, with the containers on it, can begin play before that client's controller points
  at it — the two replicate separately — so the lookup misses.
  - The panel then takes the container that announced itself when it rides on a PlayerState owned by this player's
    controller, which is already true when it begins play.
  - Another player's container of the same tag is never taken, and neither is a container of another tag on the player's
    own PlayerState.
- **Only the root is touched.** The panel's own `Visibility` is never written:
  - A panel you collapsed stays collapsed.
  - A value you write to its root while it is hidden is kept.
  - The HUD's window layout and the equipment panel's shard row, which save and restore a panel's own `Visibility`, are
    unaffected.
  - Ask `IsCollapsedForMissingContainer()`, not `GetVisibility()`, whether the rule is hiding it.
- **A hidden panel is not a window** — it cannot keep the screen open after the player has closed every window they can
  see — and keyboard focus skips it.

## Admission — what a container takes

```
AcceptsItem(Definition)                       may this go in?
AcceptsItemWithReason(Definition, OutReason)  ...and if not, why not
GetAdmissionRefusal(Definition)               just the reason
```

Each container has an `AdmissionMode`:

| Mode | Takes |
|---|---|
| `ExactTag` | Only items whose routing tag matches this container's exactly |
| `MatchesTag` | That tag or any child of it |
| `TagQuery` | Whatever `AdmissionQuery` matches |
| `AdmitAll` | Anything that is not a counter |

- `USnapLootInventoryComponent` ships `AdmitAll`, so a corpse inherits a dead player's routed items.
- A project that wants the stricter behavior sets `AdmissionMode = ExactTag` — it is `EditAnywhere` and `config`.
- For a decision the modes cannot express, attach a `USnapInventoryAdmissionPolicy` — see [extending](extending.md).

## Weight

If `bEnableCarryWeight` is on, each unit's `Weight` fragment contributes to a carried total, and exceeding the limit
applies an overweight effect. See [durability, spoilage, weight](condition-and-weight.md).

## Events

- `OnInventoryChanged` fires for any change.
- `OnCounterChanged`, `OnDurabilityChanged` and `OnItemBroke` are narrower.
- All are in [appendix: events](appendix-events.md).

⭐ **A bulk change fires `OnInventoryChanged` once.**

- The world-drop doors, Take All, store and collect all, moves between containers, sort, compact and the grid arrangers
  run inside a change batch (`BeginChangeBatch` / `EndChangeBatch`, or the RAII `FSnapInventoryChangeBatch` for several
  containers at once).
- The per-item announcements are held and ONE fires when the call ends — naming the item when exactly one changed, else
  with no item, which means "re-read everything".
- `GetChangeBatchItems()` lists every item that changed, during that one broadcast.
- A client hears one per replication receive, after the receive has landed.
- Open your own batch around any C++ code of yours that changes many entries at once.

⚠ **On a client, `OnInventoryChanged` can pass no item even for a single change.** The item's object may not have
reached the client in the same receive as its entry.

- Treat a null item the same way as a bulk change: re-read the container.
- `GetAllItems()` already skips entries whose item has not arrived.

⚠ **A stat tag changed directly on a held item does not announce itself** (`USnapItemInstance::AddTagStack` /
`RemoveTagStack`).

- Call `NotifyInstanceChanged(Instance)` on the container that holds it afterwards, on the server.
- That fires `OnInventoryChanged` for it and sends the change to clients.

⚠ Two listeners on one delegate fire in registration order, so a widget that reads state another listener writes is at
the mercy of startup timing. Defer the read by one tick.
