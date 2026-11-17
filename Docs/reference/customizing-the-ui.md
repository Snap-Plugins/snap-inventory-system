# Customizing the UI

Every surface the plugin and its demo draw — but two, listed under [Every widget class and its Blueprint](#every-widget-class-and-its-blueprint) —
comes as a **Widget Blueprint you can open**. This page lists them by surface, names the parts the plugin drives, and
shows how to move and restyle a part.

**To move a part, resize it, change its brush or its font:**

1. Open the Blueprint the setting names.
2. Edit it in the designer.
3. Save.

- The plugin finds the parts it drives **by name**, so a part keeps working wherever you put it, as long as its name
  stays.
- While a theme is picked, a text with a **font row** follows the theme's fonts.
  - Set that row's Source to *Authored* (or delete the row) to keep your own font.

**On this page:**

- [What the shipped Blueprints are](#what-the-shipped-blueprints-are)
- [The Blueprints, by surface](#the-blueprints-by-surface)
- [The demo's own widgets](#the-demos-own-widgets)
- [Every widget class and its Blueprint](#every-widget-class-and-its-blueprint)
- [Moving and restyling a part](#moving-and-restyling-a-part)

## What the shipped Blueprints are

- The example Blueprints under `Content/Core/UI` are the plugin's own widgets.
  - The demo's own few are in `Content/Demo` (its HUD and menu in `Demo/UI`, the training dummy's in
    `Demo/TrainingDummy`); the money changer's and keyring's widgets are with the examples, in `Content/Examples/UI`.
- Where a widget used to build itself in C++, its example is that C++ build copied into the designer, part for part.
  - The same widget tree, the same part names, the same layout, brushes and colors.
- With no change of yours, it draws exactly as the C++ widget did — measured pixel for pixel against it.

**What that means for you:**

- **Every part has a name.**
  - The ones the plugin drives are listed per class in [Named parts](ui-and-theming.md#named-parts).
  - The rest are layout (boxes, sizers, borders) you may rearrange freely.
- **Colors follow a theme through Themed Parts.**
  - Each example lists, in its class defaults (**Theme → Themed Parts**), the palette role every themed color takes.
  - So `DA_SnapClassicTheme`, `DA_ExampleSteelTheme` or your own theme recolors the Blueprint exactly as it recolors the
    native widget.
  - Change a color in the designer and it is yours under no theme.
  - Remove its row (or set the row's Source to *Authored*) to keep it under every theme.
  - See [Themed parts](ui-and-theming.md#themed-parts).
- **Fonts follow a theme through font rows.**
  - A theme's typography restyles text the plugin's C++ builds.
  - A Blueprint's text keeps the font it was given (the examples use the engine's Roboto font asset — the same faces the
    C++ widgets draw with), unless its Themed Parts list a **Font** row for it.
  - While a theme is picked, that text draws the row's typography role, exactly as the C++ widget would.
  - Each part-for-part copy in the table below lists a font row for every text its C++ build styles from a role (the
    themed tooltip's label is styled by the plugin itself, as the native one is).
    - So a theme's fonts reach it as they reach the native widget.
  - The older styled examples — `WBP_ExampleHotbar` and `WBP_ExampleHotbarSlot` among them — list none.
  - **To restyle the text of an example,** edit its TextBlocks and set their font rows' Source to *Authored* (or delete
    those rows). See [Themed parts](ui-and-theming.md#themed-parts) (**Font rows**).
- **Leaving the plugin's build in charge:** Set the widget's setting to empty, and the plugin builds the widget in C++
  again, with the theme's fonts.

## The Blueprints, by surface

| Surface | Example Widget Blueprint | Picked by | Parts |
|---|---|---|---|
| Every confirmation window and notice window (DROP INVENTORY?, REPAIR?, the trade invite and wait, and every refusal notice while `NoticeStyle` is **Window**) | `WBP_ExampleConfirmWindow` | `ConfirmWindowWidgetClass` (Project Settings, **Snap Inventory System - UI Core → Widget Classes**) | [Named parts](ui-and-theming.md#named-parts), `USnapInventoryConfirmWindowWidget` |
| The collect feed: what was picked up and what was done with it | `WBP_ExampleCollectFeed` | `CollectFeedWidgetClass` (Project Settings, **Snap Inventory System - Inventory UI → Collect Feed**) | `SnapFeedRows`, the VerticalBox its rows go into |
| One line of the collect feed | `WBP_ExampleCollectFeedRow` | `FeedRowWidgetClass` (class defaults of the feed Blueprint) | [Named parts](ui-and-theming.md#named-parts), `USnapInventoryCollectFeedRowWidget` |
| Why something failed, as a white line over the screen ("Your inventory is full.") | None: built in C++ | `NoticePromptWidgetClass` (Project Settings, **Snap Inventory System - UI Core → Widget Classes**), shown while `NoticeStyle` is **Prompt** | [Named parts](ui-and-theming.md#named-parts), `USnapInventoryNoticePromptWidget` |
| The trade banner ("they closed the trade", "nobody is close enough") | `WBP_ExampleTradeNotice` | `TradeNoticeWidgetClass` (Project Settings, **Snap Inventory System - UI Core → Widget Classes**) | [Named parts](ui-and-theming.md#named-parts), `USnapInventoryTradeNoticeWidget` |
| The interaction prompt over what the player looks at ("Open Chest", its key, the hold ring) | `WBP_ExampleInteractionPrompt` | `InteractionPromptWidgetClass` (Project Settings, **Snap Inventory System - Interaction → Interaction**) | The fixed names below |
| The craft panel's progress readout under the selected recipe (the bar, Cancel, "12s remaining") | `WBP_ExampleCraftProgress` | `CraftProgressWidgetClass` (class defaults of the craft panel Blueprint) | Below; the Widget Name properties of `USnapInventoryCraftProgressWidget` |
| An ingredient chip: an item's picture and its count, in a recipe, a shop's craft row, a purchase or a sale | `WBP_ExampleCraftChip` | `ChipWidgetClass` (class defaults of the craft panel, craft row, vendor buy panel and sell window Blueprints) | Below; the Widget Name properties of `USnapInventoryCraftChipWidget` |
| A tile of the craft queue: the item, its count, its progress and the cancel X; a finished craft waiting for room shows a full bar, a green check and no X | `WBP_ExampleCraftQueueRow` | `CraftQueueRowWidgetClass` (class defaults of the craft panel Blueprint) | Below; the Widget Name properties of `USnapInventoryCraftQueueRowWidget` |
| The trade screen: your bag, your offer and theirs, the currency controls, CONFIRM and CANCEL | `WBP_ExampleTradeScreen` | `TradeScreenClass` (Project Settings, **Snap Inventory System - Economy → Trade**) | Below; the Widget Name properties of `USnapInventoryTradeScreenWidget` |
| A square of a trade offer | `WBP_ExampleTradeOfferSlot` | `OfferSlotWidgetClass` (class defaults of the trade screen Blueprint) | The craft chip's parts, below |
| The craft quantity control: the count, minus, slider and plus | `WBP_ExampleCraftQuantity` | `CraftQuantityWidgetClass` (class defaults of the craft panel Blueprint) | Below; the Widget Name properties of `USnapInventoryVendorQuantityWidget` |
| The right-click menu over an item (the plate its rows are built into) | `WBP_ExampleContextMenu` | `ContextMenuWidgetClass` (Project Settings, **Snap Inventory System - Inventory UI → Context Menu**) | Below; Row Container Widget Name and Plate Widget Name on `USnapInventoryContextMenuWidget` |
| The sort dropdown under a bag's sort button (the plate its rows are built into) | `WBP_ExampleSortMenu` | `SortMenuWidgetClass` (Project Settings, **Snap Inventory System - Inventory UI → Context Menu**) | Below; Row Container Widget Name on `USnapInventorySortMenuWidget` |
| The green / red outline around the square under a dragged or carried item | `WBP_ExampleDropOutline` | `DropOutlineWidgetClass` (Project Settings, **Snap Inventory System - UI Core → Widget Classes**) | Below |
| The layer that shows a gamepad-focused square's tooltip beside it | `WBP_ExampleFocusedTooltip` | `FocusedTooltipWidgetClass` (Project Settings, **Snap Inventory System - UI Core → Widget Classes**) | `SnapFocusedTooltipLayer`, the root CanvasPanel the card is placed on; the card is a copy of the square's own tooltip Blueprint |
| The One Panel window (`bOnePanelInventoryScreen`): one frame, one title, the screen's panels inside | `WBP_ExampleOnePanelWindow` | `OnePanelWindowClass` (class defaults of the example inventory screens) | Below |
| The tooltip on the plugin's own buttons, glyphs and readouts (the sort button's "Choose what to sort by", a filter tab's name) | `WBP_ExampleThemedTooltip` | `ThemedTooltipWidgetClass` (Project Settings, **Snap Inventory System - UI Core → Widget Classes**) | Below; [Named parts](ui-and-theming.md#named-parts) |

### `WBP_ExampleConfirmWindow`

| Part | Type | What it is |
|---|---|---|
| `SnapConfirmRoot` | Overlay | The root: the scrim and the plate on top of each other |
| `SnapConfirmScrim` | Image | The see-through layer over the whole screen that blocks presses outside the window |
| `SnapConfirmSizer` | SizeBox | The plate's width; the plugin centers it or places it at the pointer |
| `SnapConfirmBody` | Border | The plate: its fill and its outline |
| `SnapConfirmColumn` | VerticalBox | Title, message, buttons and the gamepad row, top to bottom |
| `SnapConfirmTitle` / `SnapConfirmMessage` | TextBlock | The heading (tinted by the item's rarity when the decision is about one) and the sentence under it |
| `SnapConfirmButtonRow` | HorizontalBox | The two buttons |
| `SnapConfirmAccept` / `SnapConfirmAcceptLabel` | Button / TextBlock | The button that confirms, and its label |
| `SnapConfirmCancel` / `SnapConfirmCancelLabel` | Button / TextBlock | The button that declines (keyboard focus starts here), and its label |
| `SnapConfirmPadHint` and its two pictures and two words | HorizontalBox, Image, TextBlock | The gamepad's answer keys, shown while a gamepad is the last input |

- A notice in this window (a title, a sentence and OK) is the **Window** form of `NoticeStyle`. By default a refusal shows
  as the white notice line instead (below).

### `WBP_ExampleCollectFeedRow`

| Part | Type | What it is |
|---|---|---|
| `SnapFeedRowSizer` | SizeBox | The row's least height (the feed's Row Height — a wrapped line makes it taller), held by the feed while the row slides in or out |
| `SnapFeedRowLayers` | Overlay | The plate under the frame |
| `SnapFeedRowPlate` | SnapFadeBar | The soft plate behind the line, its ends fading out (the feed's Row Plate Color and Row Plate Edge Fade; found by `PlateWidgetName`, optional) |
| `SnapFeedRowFrame` | Border | Transparent: the padding around the line |
| `SnapFeedRowBox` | HorizontalBox | Icon, name and count, left to right |
| `SnapFeedRowIconHost` | Overlay | The icon with the cooldown shade over it; collapsed when the line has no icon |
| `SnapFeedRowIcon` | Image | The item's icon; an action line's cooldown is drawn over it as a hotbar square's (`SnapItemCooldownShade`, `SnapItemCooldownText`) |
| `SnapFeedRowName` / `SnapFeedRowCount` | TextBlock | The item's name or the action sentence ("Drank Health Potion"), and "x3" |

- Its class default **Draw Icon Like Native Row** (`bDrawIconLikeNativeRow`) is on: the icon is drawn at the feed's Icon
  Size and the ring sized around it in the theme's cooldown color.
- Turn it off to keep your own icon brush and ring.

### `WBP_ExampleTradeNotice`

- `SnapTradeNoticeRoot` (Overlay) holds `SnapTradeNoticeBody` (Border, the pill).
- `SnapTradeNoticeBody` holds `SnapTradeNoticeText` (TextBlock, the message).
- Move the pill with its slot's padding (96 from the top).

### The white notice line (`USnapInventoryNoticePromptWidget`)

- No example Widget Blueprint: the plugin builds it in C++, and `NoticePromptWidgetClass` takes one of your own.
- `SnapNoticePromptRoot` (VerticalBox) puts `SnapNoticePromptRow` (HorizontalBox) about 40% down the player's part of the
  screen.
- `SnapNoticePromptRow` holds `SnapNoticePromptText` (TextBlock, the message) between two spacers, so the text wraps at
  the middle 60% of the width.
- A Widget Blueprint of your own needs only a TextBlock named by `PromptTextWidgetName`. The whole widget fades.

### `WBP_ExampleInteractionPrompt`

The prompt finds its parts by these fixed names:

| Part | Type | What it is |
|---|---|---|
| `InteractionPromptRoot` | CanvasPanel | The layer over the screen |
| `InteractionPromptPlate` | Border | Placed over the focused object |
| `InteractionPromptStack` / `InteractionPromptLine` | | The boxes |
| `InteractionPromptKeyGlyph` | Image | The key's picture |
| `InteractionPromptText` | TextBlock | The verb and the object |
| `InteractionPromptRing` | SnapHoldRing | A hold's progress |
| `InteractionPromptCount` | TextBlock | "1 / 3" when several options are offered |
| `InteractionPromptProgress` | ProgressBar | The hold bar when Interact Hold Meter asks for a bar |

⚠ Keep the root self-hit-test-invisible and every part hit-test invisible.

- The layer covers the whole screen, and a hit-testable root swallows every click.

The prompt's **Prompt Font**, **Plate Color** and **Text Color** class defaults style only the C++ build.

- On the Blueprint, set the text's font and the plate's and text's colors directly.

### `WBP_ExampleCraftProgress`

| Part | Type | What it is |
|---|---|---|
| `SnapCraftDetailProgressColumn` | VerticalBox | The bar line over the state line |
| `SnapCraftDetailProgressRow` | HorizontalBox | The bar and the Cancel button, side by side |
| `CraftDetailProgressBar` | ProgressBar | How far the selected recipe's craft has come (Progress Bar Widget Name) |
| `CraftDetailProgressCancel` / `CraftDetailProgressCancelLabel` | Button / TextBlock | Cancels that craft, and its label |
| `CraftDetailProgressState` | TextBlock | "Queued", "12s remaining", "Waiting for room" (State Text Widget Name) |

- The readout is shown only while **Show Craft Progress Readout** (`bShowCraftProgressReadout`, **Snap Inventory System - Crafting → Crafting → Timed Queue**) is on —
  it ships off.
- It hides itself while the selected recipe has nothing in the queue.

### `WBP_ExampleCraftChip`

| Part | Type | What it is |
|---|---|---|
| `SnapCraftChipSizer` | SizeBox | The chip's square (Chip Sizer Widget Name); the host sizes it |
| `SnapCraftChipStack` | Overlay | The plate, the picture and the count on top of each other |
| `SnapCraftChipBG` | Image | The plate: its fill and outline (Chip Background Widget Name) |
| `SnapCraftChipIconFit` | ScaleBox | Keeps the picture in proportion; the plugin insets it when the chip is sized (Chip Icon Fit Widget Name) |
| `SnapCraftChipIcon` | Image | The item's picture (Chip Icon Widget Name) |
| `SnapCraftChipCountPill` | Border | Holds the count in the corner (Chip Count Pill Widget Name) |
| `SnapCraftChipCount` | TextBlock | "x3", or "2/3" in a recipe (Chip Count Text Widget Name); its font row takes `ChipCount`, its size is the chip's Count Font Size |

### `WBP_ExampleCraftQueueRow`

| Part | Type | What it is |
|---|---|---|
| `SnapCraftQueueTile` | SizeBox | The tile; its edge is the class default Tile Size |
| `SnapCraftQueueTileColumn` | VerticalBox | The picture over the progress bar |
| `SnapCraftQueueTileIconBox` / `SnapCraftQueueTileStack` | SizeBox / Overlay | The square, and the plate, picture, cancel and count on top of each other |
| `SnapCraftQueueTileBG` | Image | The plate: its fill and outline |
| `SnapCraftTileIconFit` / `CraftQueueRowIcon` | ScaleBox / Image | The item's picture, kept in proportion (Icon Widget Name) |
| `CraftQueueRowCancelButton` / `CraftQueueRowCancelButtonText` | Button / TextBlock | Cancels the craft, and its X (Cancel Button / Cancel Button Text Widget Name; font row `GlyphButtonLabel`) |
| `SnapCraftQueueTileCountPill` / `CraftQueueRowCount` | Border / TextBlock | "x3", how many are left (Count Pill / Count Text Widget Name; font row `TileCount`) |
| `CraftQueueRowProgress` | ProgressBar | How far the craft has come (Progress Bar Widget Name) |
| `SnapCraftQueueTileFinishedBadge` / `SnapCraftQueueTileFinishedGlyph` | Border / Image | A finished tile's check, top right: a round plate in the palette's `DropAccepted` with the class default Finished Badge Glyph (`T_Badge_Check`) in `TextBright`. Optional: built over the picture when the tile has none |

A finished tile (crafts done and waiting for room, `SetFinishedRow`) uses the same parts: the bar full, the count, the cancel
collapsed, the check shown, and its tooltip (or a themed status text) reads the Finished Label.

Every tile's tooltip names the item in its recipe's color (the color the recipe row draws: the tier the craft will make; a
finished stack, the tier it was made at), with its state under it one size smaller: the Finished Label in green, or "Queued"
or the time left in a quieter ink. The time left is rewritten in place each second, so an open tooltip never blinks.

### `WBP_ExampleCraftQuantity`

| Part | Type | What it is |
|---|---|---|
| `VendorQuantityBlock` | VerticalBox | The readout over the slider row (Quantity Block Widget Name) |
| `VendorAmountText` | TextBlock | The count (Amount Text Widget Name); its font row takes `Readout` |
| `SnapCraftQtySliderRow` | HorizontalBox | Minus, slider, plus |
| `VendorMinusButton` / `SnapCraftQtyMinusLabel` | Button / TextBlock | One fewer, and its "-" (font row `StepperGlyph`) |
| `VendorAmountSlider` | Slider | The count, dragged (Amount Slider Widget Name) |
| `VendorPlusButton` / `SnapCraftQtyPlusLabel` | Button / TextBlock | One more, and its "+" (font row `StepperGlyph`) |

### `WBP_ExampleTradeScreen`

The plugin's own trade screen, part for part. Every part's Widget Name property is in the
[widget appendix](appendix-widgets.md) under `USnapInventoryTradeScreenWidget`. The main ones:

| Part | Type | What it is |
|---|---|---|
| `ScreenRoot` / `TradeBody` | Overlay / Border | The root, and the window: its fill (Brush Color) and outline |
| `TradeWindowOverlay` | Overlay | The window's content; the close box is placed in it when the screen is built (Window Overlay Widget Name) |
| `TradeTitle` | TextBlock | "TRADE", then "TRADE - " and the partner's name (Title Widget Name; font row `ScreenTitle`) |
| `TradeBagPanels` | HorizontalBox | EMPTY: the equipment and bag panels from `InventoryScreenClass` are put here when the screen is built (Bag Panel Host Widget Name) |
| `TradeMyOfferList` / `TradeTheirOfferList` | VerticalBox | The offer squares go here, in rows; keep each inside its ScrollBox and SizeBox — the screen sizes that box so both offers stay the same height |
| `TradeMyCurrencyBlock` / `TradeTheirCurrencyBlock` | VerticalBox | The currency section: CURRENCY, the amount and its coin, the controls (font rows `Caption`, `Readout`, `StepperGlyph`) |
| `TradeMyCurrencySlider`, `TradeMyCurrencyMinus`, `TradeMyCurrencyPlus` | Slider / Button | The currency you offer |
| `TradeMyConfirmButton` / `TradeMyConfirmLabel`, `TradeMyCancelButton` | Button / TextBlock | CONFIRM (UNCONFIRM once pressed) and CANCEL (font rows `ButtonLabel`) |
| `TradeTheirConfirmBand` / `TradeTheirConfirmText` | Border / TextBlock | Whether the partner has confirmed (font row `ConfirmationBand`) |
| `TradeMyStatusText` | TextBlock | Why an offer was refused (font row `ScreenStatusLine`) |

- The partner's mirrored rows (`TradeTheirCurrencyRow`, `TradeTheirActionRow`) are Hidden: they keep both offers the same
  height.

⚠ The window's sizes are its widgets'.

- `TradeLayout`'s square size, columns and band height still size the squares it builds, so change both together.
- Or name a subclass of `USnapInventoryTradeScreenWidget` with no tree of its own to keep the C++ layout.

### `WBP_ExampleTradeOfferSlot`

- An offer square is a craft chip: the same parts as `WBP_ExampleCraftChip` (`SnapCraftChipSizer`, `SnapCraftChipStack`,
  `SnapCraftChipBG`, `SnapCraftChipIcon`, `SnapCraftChipCount` with its font row `ChipCount`).
- Its freshness bar, its wait spinner and its glow are added into `SnapCraftChipStack` while the trade runs.

### `WBP_ExampleContextMenu`

| Part | Type | What it is |
|---|---|---|
| `SnapContextMenuRoot` | Overlay | The root; it stays drawn (and catches presses) while the plate waits to be placed |
| `SnapContextMenuPlate` | Border | The plate: its fill, outline and padding (Plate Widget Name — hidden until placed) |
| `SnapContextMenuRowsSizer` | SizeBox | The plate's minimum width, so a one-row menu still reads as a menu |
| `SnapContextMenuRows` | VerticalBox | The rows go here, one per action, on every open (Row Container Widget Name) |

### `WBP_ExampleSortMenu`

| Part | Type | What it is |
|---|---|---|
| `SnapSortMenuPlate` | Border | The root and the plate: its fill, outline and padding |
| `SnapSortMenuRows` | VerticalBox | The rows go here, one per sort order, on every open (Row Container Widget Name) |

**The ROWS of both menus are not in the Blueprints.**

- The plugin builds each row on every open, from the actions or sort orders that apply, and styles it from the theme
  (their colors and fonts follow a theme as they always have).
- The Blueprints own the plate and its layout.
- Anything placed inside the rows panel in the designer is cleared when the menu opens.

### `WBP_ExampleThemedTooltip`

| Part | Type | What it is |
|---|---|---|
| `SnapThemedTooltipPlate` | Border | The plate (Plate Widget Name) |
| `SnapThemedTooltipText` | TextBlock | The tooltip's text (Label Widget Name — required) |

- Its class default **Style Like Native Tooltip** (`bStyleLikeNativeTooltip`) is on: the plugin styles the plate and the
  text for each variant exactly as its own tooltip.
  - That is the Item tooltip's plate and line (the item card's: the theme's `PanelBackground` and `TileOutline`), its
    padding, the System hint's smaller, whiter text, the theme's font and colors.
  - It stands beside the control it explains, as the item card stands beside its item (`bPlaceMouseTooltipBesideItem`).
  - So it looks as before under every theme.
- Turn it off to style the plate and the text yourself in the designer (both variants then look the same).
- A craft queue tile's tooltip adds a second line under the text, `SnapThemedTooltipState`, built at run time while
  Style Like Native Tooltip is on and the text is the plate's content.

### `WBP_ExampleDropOutline`

| Part | Type | What it is |
|---|---|---|
| `SnapDropOutlineLayer` | CanvasPanel | The root: the layer the frame is placed on, over the whole screen, hit-test invisible |
| `SnapDropOutlineFrame` | Border | The outline (Frame Widget Name): the plugin writes its brush (the theme's DropAccepted / DropRefused at the class defaults' Outline Thickness and Outline Corner Radius), its place and its visibility on every drag |
| `SnapDropOutlineSourceFrame` | Border | Built when absent: the frame round the square a gamepad carry or a hotbar pick came from — the theme's SelectionGlow at Source Outline Thickness, tinted at Source Fill Alpha; on the root canvas |
| `SnapDropOutlineBadge` | Border or Image | Built when absent (a round plate): the action's badge at the target's top-right for a gamepad carry or a hotbar pick, sized by Badge Size and painted DropAccepted / DropRefused; on the root canvas. Drawn only for an action the class defaults' Action Badge Glyphs names — the example names the plugin's ten `T_DropBadge_*` |
| `SnapDropOutlineBadgeGlyph` | Image | Built inside the plate when absent: the action's glyph, tinted TextBright |

### `WBP_ExampleOnePanelWindow`

| Part | Type | What it is |
|---|---|---|
| `PanelRoot` | Border | The window's frame: fill and outline (Frame Widget Name) |
| `Body` / `Header` | VerticalBox / HorizontalBox | The column, and the heading line |
| `Title` / `HeaderRule` | TextBlock / Image | The one title (the screen's One Panel Title, styled like the panels' headings) and the line under it |
| `Content` | NamedSlot | The screen's panels go here when the window opens (Content Slot Name) |
| `Footer` | VerticalBox | The bag's currency line and buttons move here (Footer Widget Name) |

### The cooldown shade in the item squares

A bag square and a pouch window's square shade an item whose use is on cooldown and count the seconds down, as a hotbar
square does.

Two OPTIONAL parts, by name — with neither, the plugin builds both just after the square's picture, inside the picture's
Overlay:

| Part | Type | What it is |
|---|---|---|
| `SnapItemCooldownShade` | Image | The shade over the picture; scaled from its bottom edge by the time left (its pivot is set to the bottom) |
| `SnapItemCooldownText` | TextBlock | The whole seconds left, rounded up |

- **The plugin drives** their visibility, the shade's scale and both colors (the palette's `HotbarSlotCooldownFill` and
  `HotbarSlotCooldownText`).
- Where they sit, the text's font and the shade's brush are yours.
- Keep a Fill-aligned shade's Image Size small: a brush's size is a vote for the square's own size.
- A shade the plugin builds covers the whole square, with no padding.
  - It takes its corners from the square's own background — the first full-square Rounded Box Image under the same
    Overlay — each time it paints.
  - With no such background, it is a plain rectangle.

### The new-item shine in the item squares

`WBP_ExampleInventorySlot`, `WBP_ExampleShardSlot` and `WBP_ExampleStorageSlot` carry the three parts of the shine a newly
gained item shows (a sweep across its picture once, a glow behind it until it is looked at), in the picture's Overlay:

| Part | Type | What it is |
|---|---|---|
| `SnapNewItemGlow` | Image | The soft glow behind the picture; just before `ItemIcon` |
| `SnapNewItemSweepClip` | Overlay | Just after `ItemIcon`, Clip To Bounds, FILLING the square (no padding): keeps the sheen inside the square |
| `SnapNewItemSweep` | Image | Inside the clip, Fill / Fill: the light band that crosses the whole square |

- **The plugin drives all three** — their visibility, their render transforms and their colors, the palette's
  `NewItemGlow` and `NewItemSweep`.
  - It writes their brushes from `NewItemShineTexture` at the Image Size you give each.
- Where they sit, their draw order and the clip's clipping are yours.
- **Keep the glow laying out the picture's box** (the picture's padding, Fill / Fill).
  - It is drawn in the picture's own space, so it grows and bounces with it.
- **Keep the clip filling the picture's parent Overlay** with no padding, and the band filling the clip.
  - The sheen's path is worked out in that box, so a padded clip cuts it short of the square's edges.
- **Author all three or none.**
  - With none, the plugin builds them beside the picture, as it does for a square of your own.
  - With only some, or the wrong types, the square shows no shine, and one Warning names the parts.

### `WBP_ExampleConfirmWindow`: layout

Its class default **Size Plate To Button Labels** (`bSizePlateToButtonLabels`) is on:

- The plate grows to fit the longest button label (DROP INVENTORY widens the window instead of being clipped).
- CANCEL's gap follows the one- or two-button form, exactly as the plugin's own window does.

Turn it off when your plate sets its own width and gap.

## The demo's own widgets

These demo surfaces come as Widget Blueprints too:

- The demo's heads-up display.
- Its welcome panel and Escape menu.
- The money changer's window.
  - The demo ships the money changer, `BP_Example_MoneyChanger`, but its map does not place one for now — drop one into a
    map to see the window.
- The numbers over the training dummies.

**How they are made:**

- The same way: the demo widget's own C++ build copied part for part, drawing exactly as it did (measured pixel for pixel
  in the demo).
- The demo's classes pick them.
- A project that uses the demo classes with no Blueprint set gets the C++ build, as before.

| Surface | Example Widget Blueprint | Picked by | Parts |
|---|---|---|---|
| The heads-up display: health, the stats, the effects line, the death line, and the anchors the hotbars are placed on | `WBP_ExampleDemoHUD` | `HUDWidgetClass` (class defaults of `BP_SnapDemoPlayerController`) | Below |
| The welcome panel and the Escape menu (one widget in two modes) | `WBP_ExampleDemoMenu` | `MenuWidgetClass` (`BP_SnapDemoPlayerController`) | Below |
| The money changer's window: your coins and the trades | `WBP_ExampleDemoExchangeWindow` | `ExchangeWindowClass` (`BP_SnapDemoPlayerController`) | Below |
| The bar, Armor line and hit numbers over a training dummy | `WBP_ExampleDemoDummy` | The Widget Class of `BP_Example_TrainingDummy`'s Health Widget component | Below |
| Lines that rise and fade (the dummy's numbers, the money changer's trade line) | `WBP_ExampleDemoFloatingText` | Embedded in `WBP_ExampleDemoDummy`; the Widget Class of `BP_Example_MoneyChanger`'s Trade Text component | `SnapDemoFloatingTextCanvas`, the CanvasPanel the lines are made in |

### `WBP_ExampleDemoHUD`

| Part | Type | What it is |
|---|---|---|
| `SnapDemoHUDRoot` | CanvasPanel | The root: everything is placed on it |
| `SnapDemoStatsPlate` | Border | The stats panel at the top left — move it to move the health and the stats together |
| `SnapDemoHealthText` / `SnapDemoHealthBar` | TextBlock / ProgressBar | "80 / 100" and the health bar |
| `SnapDemoHealthReport` | TextBlock | The line under the bar after a hit or a heal |
| `SnapDemoArmorValue` / `SnapDemoAttackPowerValue` / `SnapDemoAttackCooldownValue` | TextBlock | The three stat values |
| `SnapDemoEffectsRow` / `SnapDemoEffectsText` | HorizontalBox / TextBlock | The effects line (shown while an effect is on) and its text |
| `SnapDemoPadRebindHint` / `SnapDemoFeatureLine` | TextBlock | The gamepad rebind hint, and what a FEATURES pad just switched |
| `SnapDemoDeathText` | TextBlock | "You died", in the middle of the screen |
| `SnapDemoHotbarAnchor` / `SnapDemoConsumableAnchor` | SizeBox | Where the plugin places the hotbar (bottom middle) and the potion squares (just left of it) — move these to move the bars |
| `SnapDemoCrosshair` | CanvasPanel | The crosshair, shown while you aim |

### `WBP_ExampleDemoMenu`

| Part | Type | What it is |
|---|---|---|
| `SnapDemoMenuScrim` / `SnapDemoMenuPanel` | Border | The dim layer over the screen, and the panel |
| `SnapDemoMenuPanelSizer` / `SnapDemoMenuScroll` | SizeBox / ScrollBox | The panel's height cap (kept inside the screen) and the scrolling column |
| `SnapDemoMenuTitleText` / `SnapDemoMenuIntroText` | TextBlock | The title (WELCOME ... or the menu's), and the welcome tour shown in the welcome mode |
| `SnapDemoResumeButton` / `SnapDemoResumeLabel` / `SnapDemoLeaveButton` | Button / TextBlock / Button | Start or Resume, and Leave the demo |
| `SnapDemoMouseSection`, `SnapDemoLookSpeedSlider`, `SnapDemoLookSpeedValue`, `SnapDemoInvertLookCheck` | VerticalBox, Slider, TextBlock, CheckBox | The look speed and invert controls |
| `SnapDemoControlsHeading` / `SnapDemoControlsText` | TextBlock | The keyboard controls heading and list |
| `SnapDemoGamepadControlsHeading` / `SnapDemoGamepadControlsText` | TextBlock / VerticalBox | The gamepad heading, and the box its rows (a picture per button) are built into |

- The welcome tour, the keyboard list and the gamepad rows are written by the menu's C++ when it opens.
  - So the Blueprint never shows a stale copy of the demo's controls.
- Restyle those parts in the designer, and their text and rows still come from the demo.

### `WBP_ExampleDemoExchangeWindow`

| Part | Type | What it is |
|---|---|---|
| `SnapDemoExchangeRoot` / `SnapDemoExchangePanel` | Border | The full-screen root (draws nothing, takes no clicks) and the window |
| `SnapDemoExchangeTitle` / `SnapDemoExchangeClose` | TextBlock / Button | MONEY CHANGER and the X that closes it; the trade buttons take the X's look |
| `SnapDemoExchangeBalances` | TextBlock | "You hold ..." |
| `SnapDemoExchangeTrades` | VerticalBox | The trade rows go here, one per trade the wallet allows, built when the window is made |

- It is a Snap User Widget, so its 14 Themed Parts recolor it with a picked theme exactly as the native window.
- The demo builds it the first time the changer is used, after the THEME pad may have switched.

### `WBP_ExampleDemoDummy`

| Part | Type | What it is |
|---|---|---|
| `SnapDemoDummyRoot` | CanvasPanel | The root, the dummy's widget size |
| `SnapDemoDummyNumbers` | `WBP_ExampleDemoFloatingText` | The hit and heal numbers (any floating-text class) |
| `SnapDemoDummyArmor` / `SnapDemoDummyBar` | TextBlock / ProgressBar | The Armor line and the health bar |

**The demo's HUD and menu carry no Themed Parts** and keep their own fonts:

- The demo builds them in its own look (its session starts with no theme picked).
- The THEME pad does not rebuild them, so they look the same under every theme pad choice, as before.

## Every widget class and its Blueprint

- The plugin and its demo draw with 64 widget classes (17 of them abstract bases), plus the base `Snap User Widget`
  itself.
- This is the whole list: each class and the example Widget Blueprint(s) made from it.
- A class with no example builds itself in C++ and still takes a Widget Blueprint subclass that only sets class defaults.

| Class | Example Widget Blueprint(s) |
|---|---|
| `USnapActiveEffectSquareWidget` | `WBP_ExampleActiveEffectSquare` |
| `USnapActiveEffectsWidget` | `WBP_ExampleActiveEffects` |
| `USnapCraftWaitingWidget` | `WBP_ExampleCraftWaiting` |
| `USnapCurrencyStopWidget` | None (built in C++: a gamepad focus stop over one currency in the bag panel's wallet row; it draws nothing) |
| `USnapDemoCursorWidget` | None (built in C++: the demo's own mouse pointer) |
| `USnapDemoDummyWidget` | `WBP_ExampleDemoDummy` |
| `USnapDemoExchangeWindow` | `WBP_ExampleDemoExchangeWindow` |
| `USnapDemoFloatingTextWidget` | `WBP_ExampleDemoFloatingText` |
| `USnapDemoHUDWidget` | `WBP_ExampleDemoHUD` |
| `USnapDemoMenuWidget` | `WBP_ExampleDemoMenu` |
| `USnapDropOutlineWidget` | `WBP_ExampleDropOutline` |
| `USnapEquipmentCharacterPreviewWidget` | `WBP_ExampleCharacterPreview` |
| `USnapEquipmentPanel` (abstract) | `WBP_ExampleEquipmentPanel`, `WBP_ExampleEquipmentPanelCharacter` |
| `USnapEquipmentSlotWidget` (abstract) | `WBP_ExampleEquipmentSlot` |
| `USnapFocusRingWidget` | None (built in C++: the outline around a focused button or currency on the bag screen) |
| `USnapFocusedTooltipWidget` | `WBP_ExampleFocusedTooltip` |
| `USnapInteractionHoldWidget` | `WBP_ExampleInteractionHoldRing` |
| `USnapInteractionPromptWidget` | `WBP_ExampleInteractionPrompt` |
| `USnapInventoryActivatableScreen` | None (a CommonUI host to reparent a screen's Widget Blueprint to; compiled only when the project enables CommonUI) |
| `USnapInventoryCollectFeedRowWidget` | `WBP_ExampleCollectFeedRow` |
| `USnapInventoryCollectFeedWidget` | `WBP_ExampleCollectFeed` |
| `USnapInventoryConfirmWindowWidget` | `WBP_ExampleConfirmWindow` |
| `USnapInventoryContextMenuWidget` | `WBP_ExampleContextMenu` |
| `USnapInventoryCraftChipWidget` | `WBP_ExampleCraftChip` |
| `USnapInventoryCraftPanelBase` (abstract) | None: the data base of `USnapInventoryCraftPanelWidget` |
| `USnapInventoryCraftPanelWidget` | `WBP_ExampleCraftPanel` |
| `USnapInventoryCraftProgressWidget` | `WBP_ExampleCraftProgress` |
| `USnapInventoryCraftQuantityWidget` | `WBP_ExampleCraftQuantity` |
| `USnapInventoryCraftQueueRowWidget` | `WBP_ExampleCraftQueueRow` |
| `USnapInventoryDragVisualWidget` (abstract) | `WBP_ExampleDragVisual` |
| `USnapInventoryHotbarSlotWidget` | `WBP_ExampleDefaultHotbarSlot`, `WBP_ExampleHotbarSlot` |
| `USnapInventoryHotbarWidget` | `WBP_ExampleDefaultHotbar`, `WBP_ExampleHotbar` |
| `USnapInventoryLoadoutWindowWidget` | `WBP_ExampleLoadoutWindow` |
| `USnapInventoryLootSlotWidget` | `WBP_ExampleLootSlot` |
| `USnapInventoryLootWidget` | `WBP_ExampleLootWindow` |
| `USnapInventoryNoticePromptWidget` | None (built in C++; `NoticePromptWidgetClass` takes a Widget Blueprint of your own) |
| `USnapInventoryPanel` (abstract) | `WBP_ExampleContainerPanel`, `WBP_ExampleInventoryPanel`, `WBP_ExampleKeyringPanel`, `WBP_ExampleShardPanel` |
| `USnapInventoryPanelBase` (abstract) | None: the data base of `USnapInventoryPanel` |
| `USnapInventoryScreenWidget` (abstract) | `WBP_ExampleInventoryScreen`, `WBP_ExampleInventoryScreenCharacter`, `WBP_ExampleSpatialScreen` |
| `USnapInventorySlotWidget` (abstract) | `WBP_ExampleInventorySlot`, `WBP_ExampleShardSlot` |
| `USnapInventorySortMenuWidget` | `WBP_ExampleSortMenu` |
| `USnapInventorySpatialPanel` (abstract) | `WBP_ExampleSpatialPanel` |
| `USnapInventoryStackSplitWidget` (abstract) | `WBP_ExampleStackSplit` |
| `USnapInventoryStorageSlotWidget` | `WBP_ExampleStorageSlot` |
| `USnapInventoryStorageWidget` | `WBP_ExampleStorageWindow` |
| `USnapInventoryStorageWidgetBase` (abstract) | None: the data base of `USnapInventoryStorageWidget` |
| `USnapInventoryThemedTooltipWidget` | `WBP_ExampleThemedTooltip` |
| `USnapInventoryTooltipWidget` (abstract) | `WBP_ExampleItemTooltip` |
| `USnapInventoryTradeNoticeWidget` | `WBP_ExampleTradeNotice` |
| `USnapInventoryTradeOfferSlotWidget` | `WBP_ExampleTradeOfferSlot` |
| `USnapInventoryTradeScreenWidget` | `WBP_ExampleTradeScreen` |
| `USnapInventoryVendorBuyPanelWidget` | `WBP_ExampleVendorBuyPanel` |
| `USnapInventoryVendorCraftRowWidget` | `WBP_ExampleCraftRow` |
| `USnapInventoryVendorQuantityWidget` (abstract) | None (an abstract base; its subclasses have theirs) |
| `USnapInventoryVendorRowWidget` | `WBP_ExampleVendorRow` |
| `USnapInventoryVendorScreenBase` (abstract) | None: the data base of `USnapInventoryVendorScreenWidget` |
| `USnapInventoryVendorScreenWidget` | `WBP_ExampleVendorScreen` |
| `USnapInventoryVendorSellWidget` | `WBP_ExampleVendorSell` |
| `USnapInventoryVendorUpgradeRowWidget` | `WBP_ExampleVendorUpgradeRow` |
| `USnapItemGridHost` (abstract) | None: the base of the windows that draw a container as a grid of squares (`USnapInventoryPanelBase`, `USnapInventoryStorageWidgetBase`) |
| `USnapItemSlotWidget` (abstract) | None: the base of the five item squares (bag, pouch, loot, hotbar and equipment) |
| `USnapOnePanelWindowWidget` | `WBP_ExampleOnePanelWindow` |
| `USnapQuestMarkerWidget` | `WBP_ExampleQuestMarker` (the quest giver's floating "!": one TextBlock, `QuestMarkText`, themed by `InteractionOutline`) |
| `USnapWindowWidget` (abstract) | None: the base of the inventory's windows and screens (the bag, equipment and crafting panels, the pouch, loot, loadout and one-panel windows, and the bag, vendor and trade screens); the confirm, stack-split, sell, buy and craft-quantity windows and the demo's exchange window derive from `USnapUserWidget` instead |

**The hotbar and its squares draw from Widget Blueprints by default**, on the HUD and on the equipment panel:

- `HotbarWidgetClass` names `WBP_ExampleDefaultHotbar`, and `HotbarSlotWidgetClass` names `WBP_ExampleDefaultHotbarSlot`.
- They are the plugin's own bar and square copied part for part, with color and font rows.
- Both have **Apply Hotbar Appearance Settings** on (class defaults).
  - So every **Hotbar | Appearance** setting reaches them exactly as it reaches the C++ widgets: slot size, icon padding,
    corner radius, outline width, key and count font size, the key, outline, background and panel background colors, the
    cooldown, charges and locked colors.

**What turning the switch off costs:** A copy with it off keeps the values it was saved with.

- Those settings stop reaching it, and its frame rests on its own Brush Color on both surfaces.
- Its color and font rows still follow a picked theme.

**To keep one part of your own** and let the settings reach the rest, leave the switch on and set that part's Themed Parts
row to *Authored*.

- A Font row keeps the text's whole font, a color row that color.

`WBP_ExampleHotbar` and `WBP_ExampleHotbarSlot` remain as restyled examples (a plate around the row; a rounder, rimmed
square), with the switch off.

Not Widget Blueprints, by design:

- **The button classes** (`USnapInventoryContextMenuButton`, `USnapInventorySortMenuButton`, `USnapInventoryFilterButton`,
  `USnapInventoryCraftCategoryButton`, `USnapDemoExchangeButton`) are `UButton`s, not user widgets.
  - Their owner builds them on every open and styles them from the theme.
  - Restyle them through the theme, or through the plate or box their owner's Blueprint puts them in.
- **`USnapInventoryActivatableScreen`** (the CommonUI host, compiled only with CommonUI enabled) ships no example.
  - A Blueprint of it would not load in a project without CommonUI.
  - Make your own from it in a project that has CommonUI.

### Blueprints that are made of Blueprints

Each example is one widget. A screen or a panel that shows another surface EMBEDS that surface's own Blueprint — edit the
child once and every screen that embeds it changes:

| Blueprint | Embeds |
|---|---|
| `WBP_ExampleEquipmentPanelCharacter` | `CharacterSlot` (`WBP_ExampleCharacterPreview`), `ShardRow` (`WBP_ExampleShardPanel`) |
| `WBP_ExampleInventoryScreen` | `EquipPanel` (`WBP_ExampleEquipmentPanel`), `InvPanel` (`WBP_ExampleInventoryPanel`), `AbilPanel` (`WBP_ExampleContainerPanel`) |
| `WBP_ExampleInventoryScreenCharacter` | `EquipPanel` (`WBP_ExampleEquipmentPanelCharacter`), `InvPanel` (`WBP_ExampleInventoryPanel`), `SpatialInvPanel` (`WBP_ExampleSpatialPanel`), `AbilPanel` (`WBP_ExampleContainerPanel`), `BankPanel` (`WBP_ExampleContainerPanel`) |
| `WBP_ExampleSpatialScreen` | `EquipPanel` (`WBP_ExampleEquipmentPanel`), `AbilPanel` (`WBP_ExampleContainerPanel`), `SpatialInvPanel` (`WBP_ExampleSpatialPanel`), `InvPanel` (`WBP_ExampleInventoryPanel`) |
| `WBP_ExampleVendorScreen` | `VendorBuyPanel` (`WBP_ExampleVendorBuyPanel`), `VendorInventoryPanel` (`WBP_ExampleInventoryPanel`), `VendorSpatialPanel` (`WBP_ExampleSpatialPanel`) |
| `WBP_ExampleDemoDummy` | `SnapDemoDummyNumbers` (`WBP_ExampleDemoFloatingText`) |

- Rows, chips, squares and tooltips that a widget makes at runtime come from a class setting on it, never from a copy
  inside its tree.
  - For example the bag panel's slot class, the feed's row class, the vendor's row classes.
- Likewise the trade screen's bag and equipment panels come from `InventoryScreenClass` and are put into `TradeBagPanels`
  when the screen is built.

**Two examples carry no tree of their own:** `WBP_ExampleCraftPanel` and `WBP_ExampleCraftRow`.

- They set class defaults (which chip, row and readout classes to use), and the plugin builds their layout in C++.
- Restyle the chip, row and readout Blueprints they name.

## Moving and restyling a part

- **Position and size** are the designer's: drag a part to another box, change its slot's padding, alignment or size.
  - The plugin never writes a part's slot on a Blueprint.
  - The one exception is the confirmation window's plate width and CANCEL gap, and only with **Size Plate To Button
    Labels** on.
- **Renaming a part** that the plugin drives: also set the matching **Widget Name** property in the class defaults.
  - Otherwise the plugin reports the part missing (one Warning per class) and skips only that part's feature.
- ⚠ **Never name a widget after a property of the widget's class** (`TitleText`, `ConfirmButton`, ...).
  - The Blueprint then cannot compile.
  - The shipped names avoid them.
- **Your own copy:** Duplicate the example, edit the duplicate and point the setting at it.
  - Leaving the example untouched keeps a clean reference to compare against.
