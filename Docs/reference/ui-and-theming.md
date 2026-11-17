# UI and theming

Every widget the plugin drives is a Blueprint you own. The plugin finds the parts it needs **by name** inside your
widget tree, and skips anything it cannot find.

**On this page:**

- [The contract](#the-contract)
- [Which class does what](#which-class-does-what)
- [Reskinning, cheapest first](#reskinning-cheapest-first)
- [The theme](#the-theme)
- [Which setting names which widget](#which-setting-names-which-widget)
- [Replacing a native surface's class](#replacing-a-native-surfaces-class)
- [The HUD mirror](#the-hud-mirror)
- [Layout on the widget class](#layout-on-the-widget-class)
- [Floating windows](#floating-windows)
- [One panel](#one-panel)
- [Locked squares](#locked-squares)
- [Scroll bars](#scroll-bars)
- [What is drawn behind a screen](#what-is-drawn-behind-a-screen)
- [Item squares that move](#item-squares-that-move)
- [The character preview](#the-character-preview)
- [The character preview's exposure](#the-character-previews-exposure)
- [Things that will bite you](#things-that-will-bite-you)
- [Tooltips](#tooltips)
- [What a bag open builds, and what it keeps](#what-a-bag-open-builds-and-what-it-keeps)
- [See also](#see-also)

## The contract

A widget class declares `FName` properties ending in `WidgetName`. At construction the plugin does
`WidgetTree->FindWidget(Name)` and drives whatever it finds.

```
USnapInventorySlotWidget::HoverGlowWidgetName       default "HoverGlow"
USnapInventorySlotWidget::SelectionGlowWidgetName   default "SelectionGlow"
```

Three consequences worth internalizing:

- **Nothing is required.** Leave the name blank and that feature switches off cleanly.
  - A slot with no `HoverGlow` simply draws no hover highlight.
  - Some names ship blank on purpose: `IconWidgetName` is `None` until your slot names its icon, and a grid slot's stack
    count is written by its own Blueprint's `RefreshSlot`, not by a name.
- **Names, not types, are the contract.** Rename the property's default in class defaults and the plugin looks for the
  new name instead.
- **Your widget can be anything.** A `UBorder`, a `USizeBox`, whatever — as long as it is the type the property expects.

All 252 contracts are in [appendix: widget contracts](appendix-widgets.md).

## Which class does what

| Class | Is |
|---|---|
| `USnapInventoryScreenWidget` | The whole screen — grid, equipment, counters |
| `USnapInventoryPanel` | One container's grid of squares |
| `USnapInventorySlotWidget` | One grid square |
| `USnapEquipmentPanel` | The worn-item column |
| `USnapEquipmentSlotWidget` | One worn slot |
| `USnapInventoryStorageWidget` | A pouch window |
| `USnapInventoryStorageSlotWidget` | One pouch square |
| `USnapInventoryTooltipWidget` | The item tooltip |
| `USnapInventoryDragVisualWidget` | What follows the cursor during a drag |
| `USnapInventoryContextMenuWidget` | The right-click menu |
| `USnapInventoryConfirmWindowWidget` | Confirmations and notices |
| `USnapInventoryNoticePromptWidget` | The white line that says why something failed: the theme's `TextBright` in the `WindowTitle` font, outlined and shadowed in `TextShadow` |
| `USnapInventoryStackSplitWidget` | The split-a-stack window |
| `USnapInventoryHotbarWidget` / `...SlotWidget` | A bar and one of its squares |
| `USnapInventoryVendorScreenWidget` and friends | The shop |
| `USnapInventoryTradeScreenWidget` | Player-to-player trade |
| `USnapInventoryCraftPanelWidget` and friends | Crafting |
| `USnapInventoryLootWidget` / `...SlotWidget` | A loot container window |
| `USnapEquipmentCharacterPreviewWidget` | The 3D character portrait in the bag — see [its exposure](#the-character-previews-exposure) |

## Reskinning, cheapest first

1. **Point the plugin at a theme.** Duplicate `DA_SnapDefaultTheme`, retint it, and set
   **Project Settings > Plugins > Snap Inventory System - UI Core > UI > UI Theme** to your copy.
   - That is every color and every font the plugin's own widget code paints with — no Blueprint edit, no C++, one asset.
   - The appearance colors a widget Blueprint *authors* keep their authored values unless you hand them to the theme;
     see **Authored colors win, unless you say otherwise** below.
2. **Change the settings.** Most sizes, paddings and behavior are settings, not Blueprint edits.
   - There are 380 of them; the **Screen** (Snap Inventory System - Inventory UI), **Floating Windows** (Snap Inventory System - UI Core),
     **Item Filters** (Snap Inventory System - Items) and **Hotbar|Appearance** (Snap Inventory System - Inventory UI) categories are where
     most of the visual ones live.
   - The crafting panel's and the trade screen's own widths, gaps and padding are the exception: they are class defaults
     on the widget. See [Layout on the widget class](#layout-on-the-widget-class).
3. **Reparent an example.** Duplicate `WBP_ExampleInventorySlot`, restyle it, and point your panel's `SlotWidgetClass`
   (a class default on `USnapInventoryPanel`) at your copy.
   - ⚠ A copy of an example pouch or loot slot, window, shop row or craft row carries the example's choice to hand some
     colors to a picked theme.
   - Restyle one of those and set its Source back to *Inherit* or *Authored*.
   - Saving the copy warns about each such color; ignore the warning if the theme should win. See
     **Authored colors win, unless you say otherwise** below.
4. **Start from nothing.** Create a `UUserWidget` Blueprint whose parent is the plugin's class, add widgets with the
   contract names, and point the setting at it.
   - You never have to reproduce the example's layout.

The default widgets under `/SnapInventorySystem/Core/UI/` exist to be replaced. Nothing in the plugin references them except
settings defaults.

## The theme

`USnapInventoryTheme` is a data asset holding one `FSnapInventoryPalette`, one `FSnapInventoryTypography` and the
interface sounds (`UISounds`; see [Sounds](sounds.md)). The palette and the typography are two sets
of **roles** — named for the job a color or a font does rather than for a widget — that the plugin's native widget code
paints with.

- `Accent` is every heading and the fill of every primary button.
- `TextMuted` is every subtitle, craft time and status line.
- `SlotEmpty` / `SlotOccupied` / `SlotHover` / `SlotSelected` are the four states of a grid square, wherever the square
  lives.
  - The first two are mostly see-through by default (30% and 40%), so the panel behind shows through every square and
    its outline marks it.
- The fonts work the same way, and have [their own section](#typography) below.

Three themes ship:

- **`DA_SnapDefaultTheme`** holds exactly the plugin's own values — the neutral look below — so setting it changes
  nothing.
  - It is there to be duplicated, and to be the control when you want to know whether a difference came from your theme
    or from something else.
- **`DA_SnapClassicTheme`** is a warmer look: gold headings and accent buttons, warm bronze frames.
  - Classic draws the item tooltip's section-heading roles in the plugin's accents, and a new item's glow in the
    plugin's warm gold.
- **`DA_ExampleSteelTheme`** is a cool blue-and-steel palette with wider-tracked headings and a monospace face for
  figures.
  - Its new-item glow is a cool silver, apart from its blue selection.

Setting either of the last two repaints:

- Everything the plugin paints.
- The example widgets' colors wherever those examples hand a color to the theme — which, on the example slots, is
  deliberately not all of them.
- The example widget Blueprints' own titles, header rules, frames and buttons, which they list as
  [themed parts](#themed-parts).

### The plugin's own look

With no theme set the plugin paints a **neutral** look, like most generic inventories:

- Translucent dark panels (the world shows through a little).
- White and light-gray text.
- Thin gray frames.
- Quiet gray slot states.
- No gold or warm accents.

The shipped example widget Blueprints author the same look, so a project that uses them as they are gets it everywhere.

Colors that carry information keep their hues:

- The drop plan (green where a release is accepted — a swap too — and red where it is refused), durability, spoilage and
  the over-capacity weight.
- The close box's red, the green drag-over square, and rarity, which is the rarity table's, not the theme's.
- The **cyan selection cue** — a selected square has to stay distinct from the white ring keyboard focus draws and from
  the faint white hover wash.
- The gold outline on a pickup you look at in the world (`InteractionOutline`), which is a 3D highlight rather than a
  panel color.
- The item tooltip's **section headings** — one accent per kind of section (Modifiers gold, Passive teal, When used
  green, Ability blue, Set bonus amber, Requirements muted or red), the palette's Tooltip roles; see
  [Tooltips and stats](tooltips-and-stats.md#how-the-lines-are-drawn).
- The **numbers of the tooltip's stat lines** (`TooltipStatValue`, pale gold), a **magic modifier's** whole line
  (`TooltipMagicModifier`, pale blue), and the bag panel's wallet amounts (the `CurrencyAmount` typography role, Bold
  12).

**To get the earlier look back**, set **UI Theme** to `DA_SnapClassicTheme` (the `/SnapInventorySystem/Core`
cook line carries it into a packaged build). That brings back:

- Every color the plugin's code paints from a role — headings, accent buttons, the native hotbar squares' keys and
  frames, the selected filter, the craft window.
- The example colors that already follow a theme (the example pouch and loot windows' selected square, the recipe and
  shop rows).
- The example widget Blueprints' panel titles, header rules, panel frames and plates, window buttons, slot outlines and
  slot hover glow, which they list as [themed parts](#themed-parts).

Two kinds of color do **not** come back with it:

- **A color a widget authors on
  *Inherit*** — the interaction prompt's words among them — keeps its authored, now neutral, value, because **Default
  Theme Usage** ships *Authored* (see *Authored colors win, unless you say otherwise*, below).
  - Set **Default Theme Usage** to *Theme* as well and those follow Classic too.
  - ⚠ That also hands the example pouch and loot squares' own empty, occupied and drag-over colors to Classic's slot
    roles, which are close to, not the same as, what those squares used to author.
- **A designer color no example lists as a themed part.** A few stay neutral under every theme: the lock and favorite
  badge pads, the tooltip's grants line, and colors the plugin's code paints itself from a role anyway (the hotbar
  squares' keys and frames, the page indicator, the sell window's plate and price, the drag pill).
  - Where an earlier gold was not a palette value, its part takes the nearest role, so it comes back close rather than
    exact — the slot hover glow's outline keeps the neutral look's quieter opacity.

A theme set while a session runs reaches what is built after it:

- The bag's hotbar squares, rebuilt with the bag, take the earlier gold.
- The HUD's bars, built when play began, keep their colors until they are rebuilt.

The demo plays in this look whatever theme your project picked: its profile clears **UI Theme** for the play session and
puts the color settings that mirror a role back to the plugin's own (see [the demo map](demo-map.md)).

A theme is read where a color is **painted** and where text is **built**, so it takes effect for every widget built or
refreshed after you set it.

- Set it in Project Settings and it is simply what the game looks like.
- Change it while a session is already running and you will see a mixture until each screen is rebuilt.
  - The bag screen is kept built between opens — `USnapInventoryHUDComponent::DiscardCachedScreens` rebuilds it, as the
    demo's theme pad does.
  - A label rewritten on every refresh follows immediately, while a surface colored once when its widget was built keeps
    the old color until that widget is rebuilt.
  - **Every font is written when its widget is built**, so text keeps its old font until then.
- Nothing is cached and nothing is stale after a restart — but live re-theming is not a supported workflow, and this is
  why.

The log says which of the three it took, once per outcome:

- *"UI colors are coming from the theme asset 'X'"*.
- *"UI colors are the plugin's own -- no UI Theme is set"*.
- Or the Warning naming a path that would not load.

That is worth knowing, because a theme whose roles happen to match the shipped ones looks exactly like a theme that
never applied.

### Precedence is total

- **No theme set** (the shipped state) — The plugin's own colors and fonts paint.
  - For the roles that mirror a setting (`HotbarSlotKeyColor` and its neighbors, and the two hotbar font sizes), that
    setting still decides, so an existing `.ini` line keeps working exactly as it did.
- **A theme set** — Its palette is the whole truth for every color the plugin paints and its typography for every font:
  the mirrored settings are not consulted.
  - The appearance colors a widget Blueprint authors are a separate question with its own rule, below.
- **A theme set that will not load** — The plugin's own colors and fonts paint and a Warning names the path.
  - This is the one case that is neither, and it is loud because a silently-ignored theme looks exactly like a theme
    that was never applied.

Two colors are not roles and are read from the settings either way: `HotbarEmptySquareGhostTint` and
`HotbarPanelEmptySquareGhostTint`.

- They are the tints of an empty hotbar square's ghost picture on the HUD and on the equipment panel (see
  [Hotbar](hotbar.md#what-an-empty-square-shows)).

There is no per-role merge.

- Two sources that each supply half an answer is how they begin to disagree invisibly.
- It also makes "why is this one color not coming from my theme?" unanswerable without reading the source.

### It ships turned off

The container registry ships **set**; the theme ships **empty**.

- The plugin's own look needs no asset.
- A theme named only from Project Settings has no referencer the cooker can find.
- So naming one by default would make every packaged project depend on a `DirectoriesToAlwaysCook` line it never asked
  for.

### Authored colors win, unless you say otherwise

Ten widget classes declare appearance colors in their class defaults — exactly the colors a widget Blueprint subclass
restyles:

| Class | Colors |
|---|---|
| `USnapInventoryStorageSlotWidget` | `EmptySlotColor`, `OccupiedSlotColor`, `HoverSlotColor`, `SelectedSlotColor` |
| `USnapInventoryLootSlotWidget` | `EmptySlotColor`, `OccupiedSlotColor`, `HoverSlotColor`, `SelectedSlotColor` |
| `USnapInventoryStorageWidget` | `SelectedSlotColor`, `PlanFitColor`, `PlanBlockedColor` |
| `USnapInventorySpatialPanel` | `PlanFitColor`, `PlanBlockedColor` |
| `USnapInventoryLootWidget` | `SelectedSlotColor` |
| `USnapInventoryVendorRowWidget`, `USnapInventoryVendorUpgradeRowWidget`, `USnapInventoryVendorCraftRowWidget` | `SelectedColor`, `UnselectedColor` |
| `USnapInteractionPromptWidget` | `PlateColor`, `TextColor` |
| `USnapInteractionHoldWidget` | `FillColor`, `TrackColor` |

- A swap is washed in `PlanFitColor` like any accepted release (the drag picture's badge says it is a swap).
- A square under a drag is outlined by the HUD's drop outline (`DropAccepted` / `DropRefused`) rather than tinted.

The interaction prompt's two have palette roles of their own, `InteractionPromptPlate` and `InteractionPromptText`,
shipped at exactly the prompt's own colors — so a theme that does not author them changes nothing.

- `TextColor` also paints the prompt's "2/3", at 70% of its opacity, and the fill of its hold bar.
- The prompt is built the first time something is focused, so a theme change reaches it in the next session, like every
  surface colored when it is built.

A picked theme does **not** replace them. A widget Blueprint looks exactly the way it was authored until somebody
chooses otherwise, at one of three levels:

1. **The color.** Directly under each color sits its **Source** — `EmptySlotColorSource`, `PlanFitColorSource` and so
   on: *Inherit*, *Authored* or *Theme*.
2. **The widget.** **Theme > Theme Usage** in the widget's class defaults: *Inherit*, *Authored* or *Theme*, for every
   color on that widget whose Source is *Inherit*.
3. **The project.** **Project Settings > Plugins > Snap Inventory System - UI Core > UI > Default Theme Usage**: *Authored*, which is
   what
   ships, or *Theme*.

**Per-color, then per-widget, then the project default. The first level that expresses an opinion decides that color outright. There is no merge and no averaging.**

- A color whose Source is *Authored* keeps its value inside a widget set to *Theme*, inside a project set to *Theme*.
- **With no theme picked, every authored color paints**, whatever any level says. The levels decide who wins against a
  theme that is in force; with none, there is nothing to yield to.
- **"The theme wins, except where I say"** is *Default Theme Usage = Theme*, plus *Authored* on the widgets or colors
  you want to keep.
  - Setting the project default to *Theme* is also how to have a picked theme recolor every one of these colors.
- **A color is judged by the object it is read from.** A square inside a pouch or loot window paints the *window's*
  `SelectedSlotColor`, so that color follows the window's `SelectedSlotColorSource` and **Theme Usage** — never the
  square's.

⚠ **The examples hand their un-authored colors to the theme.**

- `WBP_ExampleStorageSlot` authors its empty, occupied and drag-over colors, and `WBP_ExampleLootSlot` its empty and
  occupied ones.
  - Those are left on *Inherit*, so they keep their authored values while *Default Theme Usage* is *Authored*.
- Every other color on the example widgets has its Source set to *Theme*, so `DA_ExampleSteelTheme` shows both behaviors
  at once — *this is mine, that follows the theme*.
- **A copy of an example carries those choices.** Restyle one of them in your copy and set its Source back to *Inherit*
  or *Authored*, or a picked theme will paint over it.
- **Saving checks this for you.** When a widget Blueprint is saved, every color that differs from its parent class's
  value while that color's own Source is still *Theme* gets a validation warning naming it.
  - The comparison is with the parent class's defaults — the value the reset arrow restores — so an untouched copy is
    quiet.
  - Only the color's own Source counts: a color left on *Inherit* is not reported, even inside a widget or a project set
    to *Theme*.
  - A color that follows the theme on purpose, as a fallback for projects with no theme picked, is legitimate: ignore
    the warning.
  - It appears on every save, and nothing turns it off.

**The craft window's recipe rows follow a picked theme through the example screens.**

- The three example inventory screens and `WBP_ExampleVendorScreen` build their craft panel from
  `WBP_ExampleCraftPanel`, whose `CraftRowWidgetClass` is `WBP_ExampleCraftRow`.
- The row sets `SelectedColorSource` and `UnselectedColorSource` to *Theme* and leaves both colors at their native
  values.
- Both Blueprints set class defaults only and have no designer tree, so the rows keep the native outline, theme fonts
  and output-count badge, and the vendor keeps its two-column craft tab.

⚠ **Keep a craft panel or row subclass free of a designer tree unless you mean it.**

- A craft panel with its own tree is hosted whole in one column of the vendor's craft tab.
- A row with its own tree is found by its part names and filled on every refresh, but it is not restyled: the outline,
  fonts and icon sizing are yours to lay out.
- A screen that names no panel class builds the native panel and native rows, whose Sources are *Inherit*.

⚠ **A selected square's `SelectedSlotColor` is a fallback, not the cue.**

- A square whose tree has a selection glow draws the glow instead — every shipped slot does.
- So that color and its Source only show on a square without one.

### Themed parts

A theme repaints every color the plugin's own code paints from a role. A color a designer authors on a widget in a
**Widget Blueprint's tree** — a panel title, a header rule, a frame, a button's style — is a different thing: nothing
tells the plugin which role it plays, so no theme reached it.

A **themed part** is that sentence: *the widget named `Title` takes `Accent`*.

- Every plugin widget class built on `UUserWidget` derives from **`USnapUserWidget`** (the demo's own HUD, menu and
  training-dummy widgets aside).
- It carries a list of them, **Themed Parts**, in its class defaults (**Theme > Themed Parts**).
- A project's own Widget Blueprint can use it too: reparent it to **Snap User Widget**.

| Field | What it is |
|---|---|
| **Widget Name** | The widget, by its name in this Widget Blueprint's tree. None leaves the row out. |
| **Target** | Which of its colors: **Color** (a TextBlock's text, an Image's Color and Opacity), **Brush Color** (a Border's), **Outline** (the outline of a Border's Background or an Image's Brush), a Button's **Normal / Hovered / Pressed / Disabled** tint or outline, a Progress Bar's **Progress Fill** (its fill color) or **Progress Background** (its track's tint), or a Slider's **Slider Bar** or **Slider Handle** color — or a TextBlock's **Font** (see **Font rows**, below). |
| **Role** | The palette role, by its name in a theme asset: `Accent`, `TextBright`, `FrameOutline`, `PanelBackground`, `ButtonPlate`, ... any color of the palette. On a **Font** row, a typography role instead: `Readout`, `ChipCount`, `TileCount`, `ButtonLabel`, `PanelHeading`, ... any role of the theme's Typography. |
| **Scale** | Multiplies the role channel by channel before it is painted. White is the role exactly; (1, 1, 1, 0.6) is the role at 60% opacity; (0.61, 0.61, 0.61, 1) the role darkened. Not used by a Font row. |
| **Offset** | Added to the role after **Scale**, red, green and blue only; zero adds nothing. For a color derived by *adding* to a role — the plugin's own buttons lift their plate by a fixed amount for the hover fill — so a theme moves it by the same amount. Not used by a Font row. |
| **Source** | *Theme* by default — listing a part is the opt-in. *Authored* keeps the widget's own color (on a Font row, its own font); *Inherit* lets the widget's Theme Usage and then the project's Default Theme Usage decide. |

- **The same chain as every authored color.** A part is judged per color, then per widget, then by the project default,
  exactly as *Authored colors win, unless you say otherwise* (above) describes.
  - **With no theme picked nothing is applied**: The authored colors paint.
  - ⚠ Because a part's **Source** defaults to *Theme*, a widget's Theme Usage set to *Authored* does not switch listed
    parts off; set the part's own Source to *Authored* or remove the row.
- **Applied once, when the widget is built** — like every surface the plugin colors when it builds it.
  - A theme picked during a session reaches widgets built after it; a cached screen keeps its colors until it is
    rebuilt.
- **A part that cannot apply is skipped and reported**: A widget the tree does not have, a role the palette (or, for a
  Font row, the typography) does not have, or a color that widget type does not carry (**Outline** on a TextBlock,
  **Font** on anything but a TextBlock).
  - Saving the Widget Blueprint adds one validation warning per such part.
  - Building it with a theme in force logs one Warning per class per session.
  - Only that part is skipped.
- **`DA_SnapDefaultTheme` still changes nothing on the examples.** Each example part's **Scale** is chosen so the
  default role times it is the color the Blueprint authors.
  - Under the default theme every part paints its own color again, and under Classic or Steel it paints that theme's
    role in the same relationship.

**Font rows.** A row whose **Target** is **Font** gives a TextBlock a *typography* role — the font the plugin's own C++
build writes on the same text — so a Blueprint's text reads like the native widget's under every theme.

- While a theme is picked, the text takes the role's typeface and letter spacing, its size, and its family when the role
  or the theme's **Font Family** names one (otherwise it keeps its own family, outline and material).
- With no theme picked nothing is applied: the Blueprint's own font draws.
- ⭐ **To keep a font of your own under every theme, set that row's Source to *Authored*, or delete the row.**
  - The text then keeps its whole font — family, typeface, size and spacing — whatever theme is picked.
  - It is the same switch a color row uses; there is no other.
- **The plugin's own styling yields to a font row.** A few routines style a Blueprint's text by name: the shop's
  quantity readout and its minus and plus glyphs, an action button's label (the sell window's total), the multi-item
  count badge, the label of a themed tooltip styled like the native one.
  - Each writes no font when the Blueprint lists a font row for that text: the row decides.
- **A role sized by a setting keeps the text's own size.** `ChipCount`, `MenuLabel`, `SortSelected`, `SortUnselected`
  and `TooltipLabel` carry no size (see **Typography**, below).
  - So a font row with one of them changes the face and spacing only; an ingredient chip's count still takes its size
    from the chip's **Count Font Size**.

**A worked example — `WBP_ExampleInventoryPanel`:**

| Widget Name | Target | Role | Scale |
|---|---|---|---|
| `Title` | Color | `Accent` | (1, 1, 1, 1) |
| `HeaderRule` | Color | `FrameOutline` | (0.61, 0.61, 0.61, 1) |
| `PanelRoot` | Brush Color | `PanelBackground` | (1, 1, 1, 1) |
| `PanelRoot` | Outline | `FrameOutline` | (1, 1, 1, 0.6) |
| `InvCounterValueText` | Color | `Accent` | (1, 1, 1, 1) |

- With no theme the panel paints exactly as authored.
- With `DA_SnapClassicTheme` its title is the earlier gold, its rule the earlier brown, its frame the earlier bronze at
  60%.
- With `DA_ExampleSteelTheme` all three are Steel's.

Twenty-two example widget Blueprints list their parts — 158 in all: every panel's title, rule, frame and plate, the
split, loot, storage and shop windows' buttons, the slots' outlines and hover glow, the tooltip's plate, outline and
flavor line.

### Typography

`FSnapInventoryTypography` is every font the plugin's native widget code writes, as roles named for the job:

- `PanelHeading` is the crafting panel's heading.
- `SectionHeading` is every sub-heading inside a panel.
- `ButtonLabel` is both action-button recipes.
- `Readout` is a quantity readout and the count badges beside it.
- `HotbarKey` is the key label on a hotbar square.

A role is a **Font Family** (a font asset), a **Typeface** within it, a **Size** and a **Letter Spacing**. Every shipped
default is the exact value that text used before roles existed, so a theme that leaves the block alone changes no text.

- **One edit changes the face of every role.** Set **Font Family** at the top of the block and every role that names no
  family of its own draws with it.
  - A role's own family wins over it. (Three labels are not roles — see **What a theme does not cover**.)
- **Five roles take no size.** `MenuLabel`, `SortSelected`, `SortUnselected` and `TooltipLabel` are sized from
  **Filter Icon Size**, and `ChipCount` by the ingredient chip's own **Count Font Size**.
  - A size there would silently delete that setting's effect, so those roles offer only family, typeface and spacing.
- **The hotbar mirrors its settings.** With no theme set, `HotbarKey` and `HotbarCount` take their sizes from
  **Hotbar Slot Key Font Size** and **Hotbar Slot Count Font Size**, exactly as before.
  - With a theme set, the theme decides and those two settings are not consulted — the rule the mirrored color roles
    follow.
  - With no family named, a hotbar square keeps its TextBlock's own font.
- **An authored font wins.** The interaction prompt's **Prompt Font**, when set, supplies the family; the
  `InteractionPrompt` role still supplies the typeface, size and spacing.
  - Text a native window copies from a TextBlock your Widget Blueprint provides — the sell window's lines, the vendor
    screen's secondary text, the item tooltip's value lines — keeps following that TextBlock.
- ⚠ **A few Blueprint TextBlocks are styled by the plugin, by name.**
  - The shop's quantity readout and its minus and plus glyphs take `Readout` and `StepperGlyph`.
  - An action button's label (the sell window's total) takes `ButtonLabel`.
  - The multi-item count badge takes `Readout`.
  - The label of a themed tooltip styled like the native one (`bStyleLikeNativeTooltip`) takes `TooltipLabel`.
  - The plugin finds them by name because it also drives them, so the styling comes with the name.
  - To give one your own font, set that role in your theme — or list a **Font** row for it in the Blueprint's Themed
    Parts (with its Source on *Authored* to keep the font you gave it): a font row decides, and the plugin then writes
    no font there.
- **Native text has no per-font Source.** The text these roles style is built by the plugin's native code and has no
  authored value to protect, so a picked theme styles it directly, exactly as it does the plugin's own colors.
  - Text in a Widget Blueprint follows a theme's fonts only through a **Font** row (see **Font rows**, under Themed
    parts), which carries a Source like any themed part.

⚠⚠ **A size is a layout decision, not only a style one.** The plugin's panels, wrap widths and reserved bands are
measured against the shipped sizes.

- Raised by 4, the confirmation window's message wraps onto a second line and the window grows 41 units taller, and the
  bag's inventory panel widens by 27.5.
- **The thinnest surface is the trade screen's status line** (`ScreenStatusLine`): its reserved band holds the longest
  shipped refusal at size 12 with 0.707 units to spare, in English.
  - So a larger size — or a wider family — pushes a two-line reason past the band and over the frame's edge.
- A project that changes a size owns that consequence; open the screens it reaches.

⚠ **A typeface name belongs to its family.** Change the family and a name it does not have (`Bold` in a font with a
single face) falls back to that font's first typeface without complaint.

### What validation catches

`USnapInventoryTheme` validates on save, on Validate Assets and at cook. It rejects two things that cannot be seen by
looking at the palette:

- **Ink at alpha 0.** Slate lays the label out and draws nothing.
  - It reads as a missing widget, not as a color someone typed. (A *surface* at alpha 0 is legitimate and is allowed.)
- **Two roles alike where the difference is the information.** The two drop-plan colors are the only thing a grid's wash
  says about a drop — accepted (a swap included) or refused.
  - Painting them the same deletes the answer while leaving a highlight on screen that looks like it is working.
  - Same for the drop outline's accepted-vs-refused, empty-vs-occupied squares, selected-vs-unselected rows, and the
    close box's rest-vs-hover.

It also rejects **a typography role at size 0**, whose text is laid out at no height and drawn at no size — the font
twin of ink at alpha 0.

- The details panel stops a size at 1; an asset written by a script does not pass through the details panel.

Flattening `TextBright` against `TextMuted` is a warning rather than an error: nothing becomes unreadable, and a flat
hierarchy is a defensible art choice.

### What a theme does not cover

Nine single-use color literals remain in the plugin's widget code, and they are deliberate:

- The action button's **hovered, pressed and disabled plates** are offsets from the `ButtonPlate` role, so they follow a
  retinted plate.
  - But their *relationships* (lifted and warmer, half a step darker, flatter and translucent) were tuned against the
    default and are not themselves roles.
- The disabled button's hairline and a handful of one-off tints are single-use values that no other widget shares (the
  crafting panel's heading rule used to be one; it now follows `FrameOutline`).
  - A role for each would make the palette longer without making anything else move with it.

A color a Widget Blueprint authors in its tree is reached only through a [themed part](#themed-parts); one that no part
lists keeps its authored value under every theme.

Rarity colors are not in the theme either, and never were: they live in the rarity table (`USnapItemRarityTable`),
which is already an asset you own.

Three texts keep a TextBlock's own font at the size the plugin gives them, and are not typography roles:

- The floating windows' close glyph.
- The crafting detail pane's output count.
- A recipe row's count badge.

The bag's currency figures are a role: `CurrencyAmount`.

**Brushes, corner radii and spacing** are not part of a theme at all. The crafting panel's and trade screen's spacing is
a class default on the widget; see [Layout on the widget class](#layout-on-the-widget-class).

### Adding a role later

A theme asset serializes the roles that existed when it was saved.

- A role added by a later version of the plugin is absent from your package, so it keeps the plugin's default and
  nothing logs it.
- There is no value of a color that means "unset", so this cannot be detected.
- After a plugin update, open your theme and check the new roles.

A typography role added later behaves the same way, with a gentler result: it takes the plugin's shipped font, which is
exactly the look your theme was saved against.

## Which setting names which widget

| Setting | Widget |
|---|---|
| `InventoryScreenClass` | The screen opened by the toggle key |
| `StorageWindowWidgetClass` | A pouch window |
| `StackSplitWidgetClass` | The split window |
| `VendorScreenClass` / `SellWindowClass` | The shop and its sell panel |
| `TooltipWidgetClass` — a class default on each slot widget, not a setting | The item tooltip |
| `LootWindowClass` | A loot container window |
| `HotbarWidgetClass` / `HotbarSlotWidgetClass` | A hotbar and one of its squares — see [Hotbar](hotbar.md#restyling-a-bar) |
| `ConfirmWindowWidgetClass` | Every confirmation window and notice window (`NoticeStyle` **Window**) — see [below](#replacing-a-native-surfaces-class) |
| `ThemedTooltipWidgetClass` | The tooltip on the plugin's own buttons, glyphs and readouts |
| `TradeNoticeWidgetClass` | The transient trade banner |
| `NoticePromptWidgetClass` | The white line that says why something failed (`NoticeStyle` **Prompt**) |
| `CollectFeedWidgetClass` | The collect feed — see [Pickup modes and the collect feed](pickup-modes-and-the-collect-feed.md) |
| `ContextMenuWidgetClass` | The right-click menu's popup — see [below](#replacing-a-native-surfaces-class) |
| `SortMenuWidgetClass` | The sort dropdown's popup — see [below](#replacing-a-native-surfaces-class) |

## Replacing a native surface's class

Some surfaces are built entirely in C++ and need no asset. Each takes an optional class, and every one of those knobs is
decided by the same rule.

| Knob | Where it lives | Builds | A Widget Blueprint with its own tree |
|---|---|---|---|
| `ConfirmWindowWidgetClass` | Project Settings, **Snap Inventory System - UI Core > Widget Classes** | Every confirmation window and every `ShowNotice` popup | Works — see [named parts](#named-parts) |
| `ThemedTooltipWidgetClass` | Project Settings, **Snap Inventory System - UI Core > Widget Classes** | The tooltip on the plugin's own buttons, glyphs and readouts | Works — see [named parts](#named-parts) |
| `TradeNoticeWidgetClass` | Project Settings, **Snap Inventory System - UI Core > Widget Classes** | The transient trade banner | Works — see [named parts](#named-parts) |
| `NoticePromptWidgetClass` | Project Settings, **Snap Inventory System - UI Core > Widget Classes** | The white line that says why something failed | Works — see [named parts](#named-parts) |
| `HotbarWidgetClass` | Project Settings, **Snap Inventory System - Equipment > Hotbar > Placement** | Every hotbar | Works — see [Hotbar](hotbar.md#restyling-a-bar) |
| `ChipWidgetClass` | Class defaults of `USnapInventoryVendorBuyPanelWidget` and `USnapInventoryVendorSellWidget` | The item chip on each sell line | Works |
| `OfferSlotWidgetClass` | Class defaults of `USnapInventoryTradeScreenWidget` | Each trade offer square | Works |
| `FeedRowWidgetClass` | Class defaults of `USnapInventoryCollectFeedWidget` | Each collect-feed row | Works — see [named parts](#named-parts) |
| `ContextMenuWidgetClass` | Project Settings, **Snap Inventory System - Inventory UI > Context Menu** | The right-click menu's popup | Works — its rows go into a panel it names, see below |
| `SortMenuWidgetClass` | Project Settings, **Snap Inventory System - Inventory UI > Context Menu** | The sort dropdown's popup | Works — its rows go into a panel it names, see below |
| `TradeScreenClass` | Project Settings, **Snap Inventory System - Economy > Trade** | The trade screen | Works — its parts are found by the Widget Names listed in the [widget appendix](appendix-widgets.md) |

A knob on a widget is set in the class defaults of your Widget Blueprint subclass of that widget, which your project
then uses in place of the plugin's:

- Through `TradeScreenClass`, `CollectFeedWidgetClass` or `SellWindowClass`.
- Or, for the buy panel, as the panel inside your vendor screen Blueprint.

The rule:

- **Empty means the plugin's own class.** Several knobs ship naming an example Widget Blueprint that is the plugin's own
  widget copied part for part, so it draws exactly as the native class did (they are listed in
  [Customizing the UI](customizing-the-ui.md)).
  - Set one to empty for the native class.
- **A Widget Blueprint that only sets class defaults works on every one of them.** It has no widget tree of its own, so
  the plugin builds the tree exactly as it does for its own class.
- **A Widget Blueprint with its own tree works on every knob whose surface finds its parts by name.**
  - The chip — and the offer square, which is a chip — finds its parts through `ChipIconWidgetName` and its siblings.
  - The hotbar finds its row through `RowWidgetName`.
  - The confirmation window, the themed tooltip, the trade banner and the feed row find theirs through the Widget Names
    listed under [Named parts](#named-parts).
  - The trade screen finds its 28 through the Widget Names in the [widget appendix](appendix-widgets.md) (see the note
    under the table below).
- **The two menu popups take one too.** `ContextMenuWidgetClass` and `SortMenuWidgetClass` build the menu's ROWS on
  every open, into the panel your Blueprint names in **Row Container Widget Name** (`SnapContextMenuRows` /
  `SnapSortMenuRows`; whatever that panel holds in the designer is cleared first).
  - The context menu also keeps the widget named by **Plate Widget Name** (`SnapContextMenuPlate`) hidden until the menu
    has been placed — keep a root above the plate, as `WBP_ExampleContextMenu` does.
  - The rows themselves are built and styled by the plugin from the theme, so a Blueprint restyles the plate and its
    layout, and what the rows say comes from [context menu actions](extending.md).
  - A tree with no usable rows container gets the plugin's own chrome, with one Warning per class — a menu never opens
    empty.
- **What is refused, and what you see.** A path that will not load, a class of the wrong type and an abstract class
  build the plugin's own class instead, with one Warning naming the knob, the path and the reason.
  - So do two designer-built trees that could not work: a confirmation window with neither a confirm nor a cancel
    button, and a tooltip with no label.
  - Each is logged once per knob and path (or once per class) per session, however many widgets are built from it.
- **A soft class path in a packaged build** needs its folder cooked — the same caveat `StorageWindowWidgetClass`
  carries.

### Named parts

A designer-built confirmation window, themed tooltip, trade banner or feed row is **driven, not rebuilt**.

- Each part below is found by the name in its Widget Name property, in the class defaults of your Widget Blueprint.
- Every default is the name the plugin's own build gives that part, so a tree that uses the defaults needs no setting at
  all.

How it behaves:

- **Your Blueprint's look is kept.** The plugin writes text, visibility, button bindings and state.
  - Fonts, colors, brushes, padding and sizes stay yours — with two exceptions that carry information or layout the
    plugin depends on, both listed below: an item's rarity color on a title or name, and the feed row's height.
  - A hotbar square or bar with **Apply Hotbar Appearance Settings** on — the shipped default copies — takes the Hotbar
    | Appearance settings as well; see [Hotbar](hotbar.md#restyling-a-bar).
- **A missing part** — no widget of that name, or one of another type — is reported in one Warning per class per
  session, naming every missing part and what is skipped without it.
  - Only that part's feature is skipped.
- **Set a Widget Name to None** to leave a part out on purpose. Nothing is logged.
- ⚠ **Never name a widget after a property of the class** (`TitleText`, `ConfirmButton`, `NameText`, ...).
  - A widget and a property with one name leave the Blueprint unable to compile.
  - That includes a C++ property the class inherits, such as a widget called `DurabilityBarWidgetName` on a bag square:
    widget names and the class's properties share one scope on the generated class.
  - Saving does not fix that compile error. Rename the widget.
  - The defaults all start with `Snap` (the trade screen's with `Trade`) for that reason.

| Class | Widget Name property | Type | Required | What the plugin does with it |
|---|---|---|---|---|
| `USnapInventoryConfirmWindowWidget` | `ConfirmButtonWidgetName` (`SnapConfirmAccept`) | Button | One of the two buttons | Binds it to confirm; hides it in the one-button notice form |
| | `CancelButtonWidgetName` (`SnapConfirmCancel`) | Button | One of the two buttons | Binds it to decline; keyboard focus is placed on it |
| | `TitleWidgetName` (`SnapConfirmTitle`) | TextBlock | No | Writes the title, and the rarity color of the item a decision is about (your own color otherwise) |
| | `MessageWidgetName` (`SnapConfirmMessage`) | TextBlock | No | Writes the message |
| | `ConfirmLabelWidgetName` (`SnapConfirmAcceptLabel`) | TextBlock | No | Writes the confirm label ("DROP INVENTORY", "REPAIR") |
| | `CancelLabelWidgetName` (`SnapConfirmCancelLabel`) | TextBlock | No | Writes "CANCEL", or "OK" on a notice |
| | `ScrimWidgetName` (`SnapConfirmScrim`) | Any widget | No | Makes it hit-testable, so presses outside the plate are blocked |
| | `PlateSizerWidgetName` (`SnapConfirmSizer`) | SizeBox in an Overlay slot | No | Centers it, or places it at the pointer; its width stays yours unless `bSizePlateToButtonLabels` is on |
| | `PadHintRowWidgetName` (`SnapConfirmPadHint`) | HorizontalBox | No — quiet | The gamepad row under the buttons: shown while a gamepad is the last input, hidden otherwise. Found with the four parts below or not at all, and never reported missing |
| | `PadHintAcceptGlyphWidgetName` / `PadHintBackGlyphWidgetName` (`SnapConfirmPadAcceptGlyph` / `SnapConfirmPadBackGlyph`) | Image | No — quiet | The held pad's accept and back key pictures |
| | `PadHintAcceptTextWidgetName` / `PadHintBackTextWidgetName` (`SnapConfirmPadAcceptText` / `SnapConfirmPadBackText`) | TextBlock | No — quiet | Writes "Select" and "Cancel" |
| `USnapInventoryThemedTooltipWidget` | `LabelWidgetName` (`SnapThemedTooltipText`) | TextBlock | **Yes** | Writes the tooltip's text |
| | `PlateWidgetName` (`SnapThemedTooltipPlate`) | Border | No | Styled exactly like the plugin's own plate for the Item and System variants while the class default `bStyleLikeNativeTooltip` is on (it also styles the label: font, color, wrap); with it off, nothing is written to your tree |
| `USnapInventoryTradeNoticeWidget` | `NoticeTextWidgetName` (`SnapTradeNoticeText`) | TextBlock | No | Writes the message |
| `USnapInventoryNoticePromptWidget` | `PromptTextWidgetName` (`SnapNoticePromptText`) | TextBlock | No | Writes the message; the whole widget fades |
| `USnapInventoryCollectFeedRowWidget` | `NameTextWidgetName` (`SnapFeedRowName`) | TextBlock | No | Writes the item's name, colored by its accent or rarity |
| | `CountTextWidgetName` (`SnapFeedRowCount`) | TextBlock | No | Writes `x3` when more than one was collected |
| | `IconWidgetName` (`SnapFeedRowIcon`) | Image | No | Sets the item's icon texture into your brush (size kept); collapses it when the item has none |
| | `RowSizerWidgetName` (`SnapFeedRowSizer`) | SizeBox | No | Sets its height to the feed's `RowHeight` — the feed sizes itself as `RowHeight` × `MaxRows` |
| | `IconHostWidgetName` (`SnapFeedRowIconHost`) | Overlay | No — quiet | Collapsed while the row shows no icon, so a line without an icon takes the icon's room |
| | `PlateWidgetName` (`SnapFeedRowPlate`) | `USnapFadeBar` | No — quiet | Given the feed's `RowPlateColor` and `RowPlateEdgeFade`; a row without one keeps its own look |
| `USnapInventoryCollectFeedWidget` | `RowContainerWidgetName` (`SnapFeedRows`) | VerticalBox | **Yes** | Every row is added to it, the newest at the bottom (the feed grows upward) |
| `USnapInventoryContextMenuWidget` | `RowContainerWidgetName` (`SnapContextMenuRows`) | A panel that takes several children (VerticalBox) | **Yes** — without one the plugin's own chrome is built, one Warning | Cleared, then the menu's rows are built into it on every open |
| | `PlateWidgetName` (`SnapContextMenuPlate`) | Any widget, not the root | No | Hidden from the build until the menu has been placed, then shown |
| `USnapInventorySortMenuWidget` | `RowContainerWidgetName` (`SnapSortMenuRows`) | A panel that takes several children (VerticalBox) | **Yes** — without one the plugin's own plate is built, one Warning | Cleared, then the sort rows are built into it on every open |

What happens when a required part is missing:

- **Confirmation window with neither button**: Refused.
  - A window no pointer can dismiss would stand over the screen, so the plugin's own window is built instead, with one
    Warning.
  - A window with only one of the two works; with no cancel button, keyboard focus goes to the window, and Escape still
    closes it.
- **Tooltip with no label**: Refused before anything is built from it, with one Warning.
  - Those controls keep the engine's plain text tooltip, so no sentence is lost.

Also worth knowing:

- A designer tooltip looks the same for the item and the UI-hint variants: the difference between the two is entirely
  styling, and your tree is not styled.
- A designer banner is made hit-test invisible as a whole, so no tree can make it swallow presses.
- The plate's adaptive width and the text wrap widths of the plugin's own confirmation window are not applied to yours.
  - Unless its class defaults turn on `bSizePlateToButtonLabels`, as `WBP_ExampleConfirmWindow` does because its tree is
    the plugin's own.
- The trade offer square takes the chip's parts; its freshness bar, spinner and glow look for an Overlay named
  `SnapCraftChipStack` and are skipped, with a Verbose line, when the tree has none.
- **The trade screen** finds 28 parts through the `USnapInventoryTradeScreenWidget` Widget Names in the
  [widget appendix](appendix-widgets.md):
  - The title, your status line, CONFIRM and its label, CANCEL, the currency slider and its two steppers.
  - For each side: the offer list, the confirm line and band, the currency section, amount, controls row, coin and its
    box, and the action row.
  - Its bag and equipment panels are not parts: they are embedded into the panel `BagPanelHostWidgetName` names
    (`TradeBagPanels`) from `InventoryScreenClass`, unless your tree placed a `USnapInventoryPanel` or `USnapEquipmentPanel` in
    that panel.
  - Keep each offer list inside a `UScrollBox` inside a `USizeBox`: the screen sizes that box so both offers stay the
    same height.

The context menu's and the sort menu's popups take a Blueprint with its own tree through `ContextMenuWidgetClass` and
`SortMenuWidgetClass` (above).

- The plate and its layout are yours, while their rows are button subclasses the plugin builds and styles on every open,
  with no knob of their own.
- The `bPreviewNativeElements` flag is not available yet.
- The item tooltip is a different widget and already has `TooltipWidgetClass`.

## The HUD mirror

Two settings paint into a HUD widget your game already has: `EquipmentHUDBindings` (Snap Inventory System - Foundation > HUD
Mirror) and `ContainerRefreshBindings` (Snap Inventory System - Inventory UI > HUD Mirror, beside `HUDRarityOpacity`). Both ship empty, which
turns them off.

**`EquipmentHUDBindings`:** One row per worn slot to paint into your HUD.

- The plugin finds `HostWidgetClass` in the viewport at runtime. It is a class path, so the plugin holds no reference to
  your HUD. Each player's mirror paints only that player's own widgets, never another split-screen player's.
- Then it finds the named children inside it:

| Row field | Type | What the plugin does with it |
|---|---|---|
| `SlotWidgetName` | Any widget | The child for this slot, when the host shows several slots; None when the host is the slot |
| `IconWidgetName` | Image | Paints the worn item's icon, shown while the slot is occupied |
| `EmptyIconWidgetName` | Image | Shows it while the slot is empty and hides it otherwise; nothing else |
| `CountTextWidgetName` | TextBlock | The stack count, shown for 2 or more |
| `RarityColorWidgetName` | Border | Tinted with the item's rarity color (`SetBrushColor`) at `HUDRarityOpacity` (0.15; negative keeps the Border's own alpha) |

- **The border's empty color is not configured.** The plugin remembers the color the Border had the first time it
  touched it, and restores that when the slot empties.
- **`SlotTag` must be one of the equipment component's slots** (`USnapEquipmentComponent::AvailableSlots`). It is
  checked when the row binds.
  - An empty or mistyped tag disables that row, with a Warning. A mistyped `FGameplayTag` in an `.ini` imports as empty,
    silently.
  - A real tag the component has no slot for is logged too; that readout stays empty.

**`ContainerRefreshBindings`:** When a player's container `ContainerTag` changes, the plugin calls `FunctionName` on every
widget of `HostWidgetClass` on that player's own screen. In split screen it never calls another player's widget.

- The function must take no parameters. It is called with no parameter frame, so one that has parameters is refused
  with a log line.
- `ContainerTag` None, or a mistyped tag that imported as empty, means the main bag.
- `USnapInventoryHUDComponent::RefreshContainerReadouts` fires the matching rows at once, for a HUD rebuilt behind the
  plugin's back.

## Layout on the widget class

The crafting panel and the trade screen lay themselves out from **layout inputs** that live on the widget class, not in
the settings:

- `CraftLayout` (an `FSnapCraftPanelLayout`) on `USnapInventoryCraftPanelWidget`.
- `TradeLayout` (an `FSnapTradeScreenLayout`) on `USnapInventoryTradeScreenWidget`.

Their defaults are the shipped layout, so nothing changes until you edit them.

1. Edit them in the **Class Defaults** of a Widget Blueprint subclass that adds no widgets of its own, under **Layout**.
2. Name that class where the widget is chosen:

| Widget | Edit the layout on | Name the class in |
|---|---|---|
| Crafting panel | A subclass of `WBP_ExampleCraftPanel` (or of `USnapInventoryCraftPanelWidget`) | `CraftPanelWidgetClass` on your inventory screen and `VendorCraftPanelWidgetClass` on your shop screen |
| Trade screen | A subclass of `USnapInventoryTradeScreenWidget` | The **Trade Screen Class** setting (`TradeScreenClass`) |

| Struct | Its inputs |
|---|---|
| `FSnapCraftPanelLayout` | `WindowListWidth`, `WindowListMaxHeight`, `DetailColumnWidth`, `ColumnGutter`, `PanelPadding`, `DetailIconEdge`, `DetailIconGutter`, `ListEmptyInset`, `EmptyHeaderLineGap`, `QueueDockGap` |
| `FSnapTradeScreenLayout` | `ColumnGap`, `SectionGap`, `PanelPadding`, `CurrencySliderTopPad`, `CurrencyCoinSize`, `CurrencyCoinGap`, `CurrencyStepperGap`, `OfferSquareSize`, `OfferSlotGap`, `OfferColumns`, `OfferBandMaxHeight`, `OfferSpoilageBarHeight`, `OfferSpoilageBarInset`, `BagColumnMaxHeight` |

**Only inputs are editable.** Two kinds of number are deliberately left out of both structs:

- **Band heights.** The crafting panel's status, progress, detail header, craft time, ingredient heading and queue
  heading each reserve a fixed height, and so does the trade screen's status line.
  - Those heights were measured against the shipped fonts. A theme whose fonts are larger can still overflow them.
- **Derived sizes.** The crafting detail panel's content width (`DetailColumnWidth` less `PanelPadding`'s left and
  right), its wrap widths, the offer grid's width and height, and the trade status line's wrap width are computed from
  the inputs by functions on the struct.
  - So they follow whatever you set.

⚠ **Your shop screen's columns are authored on the shop screen, not driven by `CraftLayout`.**

- `WindowListWidth` (336) matches `WBP_ExampleVendorScreen`'s shop column content width (its 380-wide column less 22 +
  22 of padding).
- `DetailColumnWidth` (280) matches its middle column.
- On the shop's CRAFT tab the panel derives its detail content width from `DetailColumnWidth`, so if you change
  `DetailColumnWidth`, give your shop screen's middle column the same width.

⚠ The trade screen's offer board holds three rows of squares before it grows.

- Fewer `OfferColumns` put the twelve squares on more rows, and both offers grow together so they stay the same height.

⚠ **The shipped trade screen, `WBP_ExampleTradeScreen`, has its own tree**, so its window is laid out by its widgets:
edit those.

- `TradeLayout` still sizes the squares any trade screen builds (square size, columns, band height), so change the two
  together.
- Or name a subclass of `USnapInventoryTradeScreenWidget` itself — no tree — to keep the C++ layout that follows
  `TradeLayout` entirely.

## Floating windows

Pouch, craft and loot windows float over the screen and can be dragged and closed.

- The chrome — a close box — is added natively.
- The **drag** is run by the HUD's window manager (`USnapWindowManager`); each window class forwards four handlers
  (`NativeOnPreviewMouseButtonDown`, `NativeOnMouseButtonDown`, `NativeOnMouseMove`, `NativeOnMouseButtonUp`): the first to
  the HUD component's `RaiseInventoryWindowToFront` (a pouch window's to `RaiseStorageWindowToFront`), the click-to-front,
  and the other three to its `HandleWindowMouseDown` / `HandleWindowMouseMove` / `HandleWindowMouseUp`.
- A window class with only the first two arms a drag, takes the mouse capture and never releases it.
- A window opens at the cursor and remembers where you left it, keyed by widget name; the positions are saved with the
  player's profile (`bPersistWindowLayout`, Snap Inventory System - UI Core > UI, on; see
  [Persistence](persistence.md#window-layout)).
  - A pouch window is remembered per pouch (the item), for the session, and only once the player has dragged it.
  - Until then every open of it spawns at the cursor: centered on it, just below it.
  - When the cursor can't be read, it opens centered, and each further window one `CascadeStep` (28, 28) along.
- A pouch window that a pad or a key opens with the item menu's Open appears at that row instead.

**With windows fixed** (`bAllowWindowDragAndClose` off, the plugin's default) the bag's own panels and the craft window
beside the bag don't float.

- Pouch, loot, vendor and trade windows still drag and close, and so does a crafting station's craft window
  opened without the bag.

**With windows fixed, the craft window takes the equipment panel's place** while it is open.

- The inventory screen names a panel for it, `CraftWindowDockName` (the example screens author an empty, collapsed
  Overlay called `SnapCraftWindowDock` in front of their equipment panel).
- It also names the widget it stands in for, `CraftWindowDockReplacesName` (`EquipPanel`).
- Opening the craft window from the bag puts it in that panel and collapses the equipment panel, so the window and the
  bag re-center as one group and nothing is covered at any aspect from 4:3 up.
- **CRAFT** again, **B** or **Escape** (they put the docked window away before they close the bag), or closing the bag,
  puts the equipment panel back.
- A screen with no panel of that name, or with no CRAFT button, keeps the fixed spot at the top left.
- A crafting station's window, opened with no bag, is never docked.
- A docked window is part of the screen's layout, so a click on its empty background counts as a click on empty space in
  the screen — it clears the bag's and the open windows' selections.
  - An item released over it is still taken by the window: nothing is dropped.

## One panel

The `bOnePanelInventoryScreen` setting (Snap Inventory System - Inventory UI > Screen, off) draws the bag screen's panels as **one framed window** — one title,
one close box, dragged as one:

- The paper doll and the equipment on the left.
- The bag on the right with its currency line and CRAFT and DROP INVENTORY buttons right under it, and Spells
  under that.
- The bag's column centered against the equipment.

Details:

- **What moves.** The screen moves the widget that holds its panels — `OnePanelBodyName`, `Row` in both shipped bag
  screens — into a `USnapOnePanelWindowWidget` (`OnePanelWindowClass`) that takes that widget's place and slot, so the
  group sits where it always sat.
  - It is done once, when the screen is built, before it has Slate.
- **What changes inside.** Every panel loses its own frame (fill and outline both cleared) and the line under its
  heading (`OnePanelPanelRuleName` `HeaderRule` — collapsed, still in the tree, still found by name).
  - Its heading (`OnePanelPanelTitleName` `Title`) stays as a small **section label** — EQUIPMENT, INVENTORY, SPELLS —
    at `OnePanelSectionLabelFontSize` (11) in the theme's muted text color, its words unchanged, so a grid in the window
    still has a name.
  - A heading that is not a TextBlock is left as it is.
  - The window copies the main bag panel's frame, heading and line for its own, and is titled `OnePanelTitle`
    (`INVENTORY`) — the only title, so INVENTORY also reads as the bag's label under it.
- **The currency line stays under the bag**, as without One Panel.
  - The screen's `bOnePanelCurrencyLineInFooter` (off) moves it (`GetCounterRowContainer`) into the window's footer
    instead, across its whole width (`USnapInventoryPanel::RelocateCounterRow`).
  - With a squares panel and a grid panel on one container each brings its line and only the one drawing the container's
    layout shows it.
  - The panel keeps finding its readout there, and a click on the money still drops it (the window hands the press to
    the panel).
  - With the switch off the footer stays empty and takes no room.
- **The columns beside the equipment are centered against it** (their slots in `Row` become vertically centered), so the
  bag and Spells sit in the middle of the paper doll's height rather than at its top.
  - A column holding an equipment panel keeps its alignment; without One Panel nothing changes.
- **Nothing that finds a panel changes.** The window's content is a named slot, and every sweep over the screen's tree —
  selection, sorting, the currency push, focus navigation, the craft window's dock, a `FindWidget` by name — dives into
  a named slot's content.
  - So the panels are found exactly where they were.
- **Both bag layouts** work inside it. With windows fixed the craft window still takes the doll's place in it
  (`CraftWindowDockName`).
- **With floating windows on** it is one floating window.
  - It drags from its title and frame (its footer too, while the currency line is in it; and from the equipment panel's
    gaps — the bag's gaps still mean "deselect").
  - Its close box closes the screen, and it comes back where it was left, keyed `SnapOnePanelWindow`.
  - The panels inside are nested windows, so none has a close box or a drag of its own.
  - With windows fixed there is no close box, as for every panel.
- ⚠ **Read when the screen is built.** The bag screen is cached, so a change reaches it after
  `USnapInventoryHUDComponent::DiscardCachedScreens` (the demo's ONE PANEL pad calls it) or a restart.
  - The vendor and trade screens never combine (`OnePanelBodyName` None).
- **A Widget Blueprint of the window** may author its own tree: a Border `PanelRoot` (the frame), a TextBlock `Title`,
  an Image `HeaderRule`, a **NamedSlot `Content`** (required) and a panel `Footer` — the names are properties of the
  class.

## Locked squares

With `bShowUnboughtBagSquares` on (Snap Inventory System - Inventory UI > Screen > Unbought Squares, off), the main bag draws the room its listed capacity upgrades
could still add as squares after its own (see
[the README](../../README.md#carrying-more--purchasable-capacity-upgrades)).

- **The look:** an empty square of the bag's own class with `UnboughtSquarePicture` (the equipment slots' lock,
  `T_Example_SlotLocked`) drawn over the whole square, with the square's corners, and dimmed by the palette's
  `UnboughtSquareTint` (0.12: the padlock at about 35% of white), as a locked equipment slot's lock is. Any texture works.
  - A square may author that picture's image itself, named `SnapUnboughtSquarePicture`.
  - A square whose picture is not in an Overlay draws it in its item picture image (`IconWidgetName`) instead.
- **No input:** no hover or focus glow, no card (not even the empty square's title), no selection, drag, drop outline, item
  menu or gamepad stop. The square keeps its visibility, so a press on it stops there and does nothing.
- A Widget Blueprint's own graph can ask `IsUnbought()` on the square.

## Scroll bars

Every scroll box the plugin draws has the theme's bar: the squares grid, a grid's and a pouch window's, the loot window,
the crafting window's lists, the trade screen, the vendor screen's lists, the demo menu.

- The bar is a rounded thumb in the palette's `ScrollBarThumb` (`ScrollBarThumbHovered` while pointed at or dragged) on
  a track of `ScrollBarTrack`, which is clear (none drawn) by default.
- Only a box whose bar style is still the engine's default is restyled, so a designer's own style is kept.
- The bar's thickness and size do not change.
- Each plugin widget styles its own tree when it is constructed, and a box built later is styled where it is made.
- A project's own box calls `USnapUserWidget::ApplyThemedScrollBarStyle`.

## What is drawn behind a screen

`ScreenBackdropMode` (Snap Inventory System - UI Core > Screen | Backdrop) decides it. It covers:

- The bag screen and its subclasses, the vendor and the trade screen, from their shared base `USnapInventoryScreenWidget`.
- A crafting station's craft window opened on its own (drawn behind that window, under every other window —
  [crafting stations](crafting-stations.md)).

All of them apply it through one rule:

| Mode | What is drawn |
|---|---|
| `None` (the default) | Nothing; the world shows through |
| `Darken` | A black fill at `ScreenBackdropDarkenAmount`, 0.6 |
| `Blur` | A background blur at `ScreenBackdropBlurStrength`, 8 |

Every screen reads it each time it opens, so a change shows at the next open with no restart.

- ⚠ **A loot window or a pouch window opened on its own is not a screen**, so it is never dimmed or blurred, whatever
  the mode.
- ⚠ **Blur degrades to nothing, not to a darken**, where the platform cannot sample the backbuffer: choose Darken if a
  guaranteed treatment matters more than the look.

The demo's SCREEN BACKDROP pad cycles the three — CLEAR, DIM, BLUR — for the play session.

## Item squares that move

Every item square's PICTURE comes alive:

- It grows a little under the cursor or a keyboard / pad focus and eases back.
- It pops once when an item lands in the square.
- An item the player has just gained shines until it is looked at.

The square's frame, glows and count never move. The motion is a uniform scale on the picture's render transform,
composed onto whatever that picture already draws (your own authored transform, or a spatial grid's placement), about
the center of what it draws.

| Setting (Project Settings > Snap Inventory System - UI Core > Item Motion) | Default | What it does |
|---|---|---|
| `bReduceItemMotion` | Off | On: no grow, no bounce, no new-item shine — every picture goes back to its size at once. |
| `ItemHoverScale` | 1.08 | The grown size; 1 turns the grow off. |
| `ItemHoverGrowSeconds` / `ItemHoverShrinkSeconds` | 0.12 / 0.15 | How long the grow and the shrink take (95 % of the way). |
| `ItemLandingBounceScale` | 1.18 | The pop's peak; 1 turns the bounce off. |
| `ItemLandingBounceSeconds` | 0.25 | How long the bounce takes. |
| `bSheenOnItemLanding` | On | A sheen crosses the whole square, edge to edge, every time an item lands in it (bag, pouch, worn slot, hotbar). |
| `ItemSheenDirection` (the Inventory UI page) | Top Left to Bottom Right | Which way the sheen crosses (a diagonal runs corner to corner of the item's box). |
| `ItemSheenSeconds` | 0.6 | How long the sheen takes to cross a square — smaller is faster. |
| `bMarkNewItems` | On | A new item gets the sheen once and a faint glow until it is looked at or the bag closes. |
| `bMarkOnlyCreatedItems` | On | Only items MADE for the player are new: crafted, bought, mixed, a reward, traded — not picked up or looted. |
| `NewItemGlowOpacity` | 0.35 | How strong the new-item glow is, against the palette's `NewItemGlow`. |
| `NewItemShineTexture` | `T_NewItemShine` (in `/SnapInventorySystem/Core/UI/Textures`) | The soft picture the glow and the sweep are drawn with; empty draws plain shapes. |

- **Your own squares move too**, as long as they derive from the plugin's square classes and name their picture the way
  each class already asks (`IconWidgetName` and friends).
  - The motion leaves the picture's render transform alone at rest, and puts back exactly the transform it found when it
    comes to rest.
- **Clipping.** A grown picture is drawn larger than its slot box.
  - The example bag square insets its picture 9 units and clips nothing — 58 of 76 in a list, 62 of 80 in a grid — so
    even the grow and the bounce together (1.08 x 1.18) stay inside the square.
  - A square of your own that clips its picture tight (`Clip To Bounds` on a box the picture fills) will clip the grow —
    leave a few units of padding, or turn the scales down.
  - A multi-cell item drawn per cell grows within its own box: each cell clips its share, so the picture's outer edge
    can be trimmed at the peak of a bounce.
- **Landings** are decided per item by a bag or a pouch window and per square by a worn slot and a hotbar square.
  - A screen opening, a filter, a list's sort order and a hotbar page turn are changes of view and bounce nothing (a
    grid's Sort really moves items, and each item it moves bounces).
  - Neither does the stored-hotbar restore putting saved bindings back (`USnapHotbarComponent::IsAnnouncingRestore`).
  - `USnapInventoryHUDComponent::OnItemLandedInSquare` fires once per landing on screen, however many squares show it (a
    potion bound to the hotbar bounces in the bag and on the bar) — hang a place sound off it.
- **The sheen** crosses the WHOLE square — edge to edge, inside its frame — every time an item lands in it, in
  `ItemSheenDirection` over `ItemSheenSeconds`.
  - A multi-cell grid item is crossed corner to corner as ONE sweep across the rectangle that holds the whole item.
  - The grid hands each of its cells that box (`SnapGridIconClip::ItemBoxInCell`), so the band does not depend on what
    the cells drew before the item landed.
  - The bag's squares, a pouch window's, the worn slots and the hotbar squares all draw it (`bSheenOnItemLanding`; the
    palette's `NewItemSweep`).
- **New items** — made for the player: crafted, bought, mixed, a quest's or a script's reward, received in a trade
  (`bMarkOnlyCreatedItems`; off, a pickup or a loot counts too) — get the sheen once, the first time a square shows
  them.
  - They also get a faint glow behind the picture (palette `NewItemGlow` at `NewItemGlowOpacity`) until the player
    hovers or focuses the square for a fifth of a second, or closes the bag.
  - A project's own grant marks what it hands out with `USnapInventoryComponent::NotifyItemCreated`.
  - What only MOVED between the player's own containers (bag, pouches, equipment, every other container on the
    PlayerState) is not new, and neither is a stack split from one the player already had.
  - The HUD component's record compares each item's unit total across all of those containers with the total before it
    appeared, so a pickup merged into a stack shines and a merge of two of the player's own stacks does not.
    - Each container is seeded, not compared, when the record first sees it: for at least 1.5 s, and until a client has
      received every entry the container is owed (at most 5 s).
    - Items that reach a container after that without being gained (a save loaded mid-session, a starting kit handed
      out late) read as gained. With `bMarkOnlyCreatedItems` off they shine once, the first time a square shows them.
    - On a client the two halves of a split can replicate in either order. A candidate is decided a moment after it
      appears, against the largest recent total, so a split is never new.
  - The bag's squares and a pouch window's draw it with three parts: a glow just before the picture and a clipping
    Overlay holding the sweep just after it, inside the picture's parent Overlay (the clip FILLS that Overlay, so the
    sheen reaches the square's edges).
  - A Widget Blueprint square may author them itself, by name — `SnapNewItemGlow` (Image), `SnapNewItemSweepClip`
    (Overlay) and `SnapNewItemSweep` (Image, inside the clip), as the example squares do (see
    [Customizing the UI](customizing-the-ui.md#the-new-item-shine-in-the-item-squares)).
  - Otherwise they are built there when the square is initialized, and a picture that is not inside an Overlay gets no
    shine.
  - For your own widgets, `USnapInventoryHUDComponent::IsItemNew`, `MarkItemSeen` and `MarkAllItemsSeen`.
  - It is local to each player's machine, never replicated and never saved.

## The character preview

`USnapEquipmentCharacterPreviewWidget` draws a lit copy of the player's own pawn, wearing what is equipped, into the UImage
named `PreviewImageWidgetName`.

- **What it copies.** The pawn's meshes, per-slot materials and anim class, at runtime.
  - With no pawn mesh it uses `FallbackPreviewMesh` / `FallbackPreviewAnimClass`. When those are empty (as shipped) it
    uses the settings' `DemoCharacterMesh` / `DemoCharacterAnimClass`.
  - If nothing resolves, the widget collapses.
- **A modular character** (Lyra's: an invisible base mesh running the AnimBP, visible parts on attached actors) is
  mirrored whole.
  - The `bMirrorPawnPose` property (on) leader-poses every rig mesh straight off the pawn component that drives it, so the portrait
    plays the character's live animation. Clear it only for a self-contained AnimBP.
  - Armor worn by swapping the pawn's own meshes or materials is picked up by `ResyncPawnMeshMirrors`, a few ticks after
    each equipment change.
- **The rig** is spawned `PreviewWorldOffset` above the level (below would fall under KillZ), with three lights of its
  own.
  - The lights use the engine's unitless scale: a local light defaults to 5000, and small numbers give a black figure.
  - They orbit with the camera and aim at `LightAimHeightFraction`, below the chest, so the legs are lit.
  - The rig's equipment visuals come from `USnapEquipmentVisualComponent::SpawnEquipmentVisualFor`, the function that
    dresses the pawn, and these copies never replicate (the pawn's own visual can).
- **A transparent background** (`bTransparentBackground`, on) cuts the figure out with a second, mask capture, combined
  by `BackgroundCompositeMaterial`.
  - That material's texture parameters are `PreviewColor` and `PreviewMask`; it is AlphaComposite, with the color not
    multiplied by alpha.
  - It costs one extra render per frame while the screen is open.
  - The edge is hard. Raise `RenderTargetSize` for a softer one.
  - A RenderOpacity below 1 brightens the portrait instead of fading it.
- **Turning it.** A left-drag turns the figure (`PreviewOrbitPitchMin` / `PreviewOrbitPitchMax`, -50 to 50) and the
  wheel zooms toward the head. On a pad the right stick turns it and R3 resets it.
  - The `bAllowPreviewMouseControl` and `bAllowPreviewGamepadControl` properties switch them (both on).
  - With the mouse control on, a press on the figure no longer starts the window drag; a plain click still clears the
    selections.
  - The zoom is smoothed on the tick, so a Widget Blueprint with TickFrequency Never must call
    `AdvanceZoomInterpolation` or set `PreviewZoomInterpSpeed` to 0.
  - The view survives closing the bag; a possession change resets it.
- **Closing the bag keeps the rig, paused.** The next open resumes it, unless the pawn or its meshes changed.

## The character preview's exposure

`USnapEquipmentCharacterPreviewWidget` — the 3D portrait in the bag (`WBP_ExampleCharacterPreview` in the examples) —
photographs a copy of the player's character with three lights of its own and a locked exposure.

One fixed exposure cannot suit every character:

- With the shipped lights the UE5 mannequin, whose materials are light, came out with 73% of its pixels clipped to
  white, while Lyra's darker character came out right.
- At an exposure that suited the mannequin, Lyra's character went black.

So the preview **calibrates its exposure to the figure**. The first time it is built for a mesh, before it spawns any
equipment visual, it renders its color capture, reads it back with the silhouette from its mask capture, and measures
the figure alone:

| What | How it is measured | Where it should be |
|---|---|---|
| The middle tone | The median luma inside the silhouette, 0-255 display values | Inside `CalibrationMedianRange`, 120-200 by default |
| The highlights | The 99th percentile of the brightest channel | At or under `CalibrationHighlightLimit`, 248 by default |

- A figure that is already inside keeps the authored `ExposureBias` exactly — one frame measured, nothing moved.
- A figure that is not is moved the **least** that brings it in: a blown-out figure down to just under the top of the
  range, a dark one up to just over its floor.
  - But never past the point its highlights would clip, which wins over the floor.
- The shipped figures settle in one to four frames, and never more than `MaxCalibrationFrames` (5).

Measured in the demo, in one session with the setting off as the control:

- The mannequin went from 72.6% of its pixels clipped (median 255) to 0.1% (median 188, highlights 232) at an exposure
  4.3 stops lower.
- The Iron Sword's blade went from 42.6% clipped to 1.1%.
- The engine's cube went from 100% to 0%.
- A figure lit a hundred times dimmer went from black (median 0) to a median of 129.

Details:

- **The range is deliberately wide.** At the exposures judged right for each, the mannequin's median sits near 200 and
  Lyra's character's near 132, both with their brightest 1% at about 240.
  - A narrow range would make one look like the other; this one keeps each looking like itself.
- **It is remembered, per widget,** keyed by the mesh the preview resolved (on a modular character, its pose-driving
  mesh), the three light intensities, `ExposureBias` and the range.
  - The HUD keeps one screen widget across opens, so a character is measured once.
  - **Equipping, unequipping or re-dressing does not re-calibrate.** The measurement runs before the preview spawns any
    equipment visual, so a sword or a shield is never part of it.
  - A character that wears its armor by swapping its own meshes or materials (a modular character) is measured as it is
    dressed the first time the bag opens, and keeps that exposure afterwards.
  - Either way what the player puts on keeps its true brightness against the body.
  - `ForgetCalibratedExposures` clears it.
- **It costs a synchronous readback per measured frame**, once per character per widget: an immediate render and a
  `ReadPixels`.
  - Measured: 29 to 69 ms in the editor for two to four measured frames, measured while another game was running on the
    same PC.
- **Off** with **Calibrate Preview Exposure** (`bCalibratePreviewExposure`, *Project Settings > Plugins > Snap Inventory
  System - UI Core > UI*): the preview then uses `ExposureBias` exactly as before calibration existed.
  - Calibration is also inert when `bLockExposure` is clear.
- **When it cannot measure** — no render scene, no render target, a figure covering almost none of the frame, a readback
  that fails — it keeps `ExposureBias`, logs once, and tries again on the next build.

`GetExposureCalibrationReport` says what the last build did:

- The outcome (*Disabled*, *Unchanged*, *Calibrated*, *Cached* or *Could Not Measure*).
- The bias the portrait now uses.
- How many frames it measured.
- The figure's median, highlights and clipped share at the authored exposure and at the applied one.
- How long it took.

`GetAppliedExposureBias` reads the bias off the capture itself.

## Things that will bite you

- **A widget added to the viewport is stretched across it.** `AddToViewport` puts the root in a full-screen slot, so a
  hit-testable root swallows every click on screen.
  - The plugin collapses the slot's anchors to a point and sizes it to the widget's own desired size.
  - But desired size is `(0,0)` until Slate has run a prepass, so the placement is deferred one tick.
- **`AutoWrapText` makes a widget's size a two-pass quantity.** On the frame a panel first appears there is no cached
  wrap width, so a wrapping `UTextBlock` measures as one long line and the panel is a different size per item.
  - Give wrapping text an explicit `WrapTextAt` inside a fixed-size box.
- **Desired size is not arranged size.** A `Fill`-aligned slot ignores a child's desired size entirely, so a widget can
  report a perfect square and be drawn as a rectangle.
- **A collapsed widget keeps the last rectangle it was painted at.** Test visibility before trusting cached geometry, or
  an invisible widget claims clicks.
- **Hit-testable, visible and consuming are three independent things.** A press on a hit-testable leaf still bubbles to
  the owning user widget unless that leaf handles it.

## Tooltips

Every item square (the grid, pouch, equipment, loot and hotbar slots) inherits `TooltipWidgetClass` from their shared
base, `USnapItemSlotWidget` (null in C++; each example square's Widget Blueprint sets it); the crafting panel uses the
class its host screen hands it from the grid's squares (`SetItemTooltipWidgetClass`, kept in the transient
`ItemTooltipWidgetClass`).

- A hotbar square whose own is unset uses the grid slot class's; see [Hotbar](hotbar.md#tooltip-and-drag-ghost).
- `USnapInventoryTooltipWidget` reads the instance, not the definition, so it shows the rolled rarity, the current
  durability and the spoilage state.
- It grows with its text between a minimum and a maximum width, and says under the item's name what the item is.
- It groups what it does into sections — a thin rule above each, a letter-spaced heading in the section's own accent
  from the theme, the lines indented under it — every line in the tooltip's own body font. See
  [Tooltips and stats](tooltips-and-stats.md#how-the-lines-are-drawn).

A bag, pouch-grid or worn square fills its card when the card can be seen:

- When the pointer enters the square.
- When the square takes the keyboard's or a gamepad's focus (the gamepad's card beside the square is a copy of it).
- On every refresh while it stays hovered or focused.

A refresh of a square nobody is looking at leaves the card as it was, and a square makes its card on its first hover or
focus. Filling every card on every refresh was most of what a bag open and a move cost.

## What a bag open builds, and what it keeps

The bag screen is built **once** and kept built (`bKeepInventoryScreenBuilt`, on by default).

- Shortly after the player's character is possessed — once the character and its bag exist (on a client the bag arrives
  by replication) — the inventory HUD builds the screen hidden, before its first open.
  - First its Slate tree and every widget's construct, then its craft window and its paper doll a tick each later, so
    the first open has nothing left to build.
- Nothing of it is drawn or can be hit while it is closed, building it moves no focus, and the player's input is left as
  it was.
  - A screen whose construct sets an input mode (the example screens' Blueprint does) gets the earlier one put back.
  - A focus or input mode your game asks for in the same frame, just before the build (a menu opened as the character is
    possessed, or a loading screen shown before `PrebuildInventoryScreen`), is kept: only what the build itself changed is
    put back.
- A close keeps the whole tree; an open shows that same tree again.

Details:

- **What runs once, and what runs on every open.** The screen's Event Construct and Event Destruct — and those of its
  panels, squares and paper doll — run once per play session.
  - They run again only when the screen is rebuilt: `USnapInventoryHUDComponent::DiscardCachedScreens` (which builds it
    hidden again at once) or a window moved into the floating-window layer on the first open.
  - What they did on every open runs in each Snap widget's `NativeOnScreenShown` / `NativeOnScreenHidden` instead
    (`USnapInventoryScreenWidget::NotifyScreenShown` / `NotifyScreenHidden` walk the tree).
  - A bag panel stops listening to its container on close (its selection, filter and in-flight marks cleared) and
    rebinds and refreshes on the next open — an item picked up while the bag was closed is there.
  - A grid is sized from its container again; a square comes back at rest; the craft window starts closed and does not
    follow the bag while it is closed.
  - A screen Blueprint that did something on every open does it in **On Screen Shown**
    (`USnapInventoryScreenWidget::OnScreenShown`, on every open, the first one included, once the screen is up and the input
    mode applied) and **On Screen Hidden** on every close.
- **The paper doll keeps its rig across a close, paused.** `USnapEquipmentCharacterPreviewWidget` stops both of its
  captures, hides the rig and the worn pieces and stops its meshes ticking (`IsPreviewPaused`).
  - It picks them up again on the next open — or builds a fresh rig when the character or its meshes changed.
  - Gear changed while the bag was closed is drawn once, at the open.
  - A new character while the bag is closed (a respawn) tears the rig down, and the HUD builds one for the new
    character, paused.
  - The rig is torn down when the screen goes for good (the HUD's end of play, a discard of the cached screens, a world
    being torn down).
- **Tooltip cards are filled when they can be seen** (see [Tooltips](#tooltips)).
- **A move refreshes each bag panel once.** A move announces the item that moved and then the one it displaced; both
  share one `USnapInventoryComponent::GetContentsVersion`.
  - A panel (and the craft panel) that rebuilt at that version skips the second.
  - A change made by a listener during an announcement moves the version on, so it is never skipped.
- **Your own moment to build it:** `USnapInventoryHUDComponent::PrebuildInventoryScreen` builds the screen (all three parts
  at once) when you call it — behind a loading screen, say.
  - `IsInventoryScreenKeptBuilt` says whether the next open has nothing to build.
  - With `bKeepInventoryScreenBuilt` off, or a screen class that does not derive from `USnapInventoryScreenWidget`, the
    screen is rebuilt on every open, as it was.

**Measured** in the editor on the demo map, a bag of 48 squares (or a 6 x 17 grid) with 25 items, 8 worn and a full
bank:

- An open of the kept screen takes about 8-10 ms (the squares about 8, the grid about 11), its first frame about 23 ms
  against a 9 ms idle frame.
- The same bag rebuilt on every open (the setting off, the same session) took about 164 ms, and before the trims of the
  same release about 400.
- The session's first open is as fast as the later ones.
- The first open after the bag grows (bag upgrades) or after its layout changes makes the new squares once (30-100 ms).
- Building the screen hidden costs one frame of about 115 ms in the editor right after the character appears (the screen
  itself, about 80 ms), then two short ones (the craft window, about 2 ms; the paper doll, about 30 ms the first time,
  most of it the doll's exposure measurement).
- A closed bag draws nothing: with both of the doll's captures stopped, the GPU frame is what it is with no bag at all.
- Opening the bag while walking: the engine still releases the held keys when the bag takes the keyboard
  (`bShouldFlushPressedKeysOnViewportFocusLost`, which is what prevents stuck keys after alt-tab), but the open no
  longer adds a long frozen frame to it.

## See also

- [Appendix: settings](appendix-settings.md) — every visual knob
- [Input](input.md) — which gesture does what
- [Hotbar](hotbar.md) — the bars are widgets too
