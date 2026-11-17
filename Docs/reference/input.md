# Input

This page covers the plugin's three input mechanisms: Enhanced Input, the UI gestures and the raw key bindings. It then
covers how the screens work on a gamepad and with the keyboard.

Three separate mechanisms, and knowing which is which saves a lot of confusion.

| Mechanism | Used for | Rebindable |
|---|---|---|
| **Enhanced Input** via `IMC_SnapInventorySystem` | Opening screens, hotbar keys, interact, sort, drop | Yes |
| **UI gestures** — a mouse/keyboard table inside the widgets | Click, right-click, shift-click, split | Yes, per gesture |
| **Raw `FInputKeyBinding`** | A legacy zero-setup fallback | No |

**On this page:**

- [Enhanced Input — the normal route](#enhanced-input--the-normal-route)
  - [The shipped contexts and actions](#the-shipped-contexts-and-actions)
  - [Rebinding rows](#rebinding-rows)
  - [The interact action](#the-interact-action)
  - [Keys and buttons that are settings](#keys-and-buttons-that-are-settings)
- [UI gestures](#ui-gestures)
- [Raw key bindings](#raw-key-bindings)
- [Gamepad and keyboard navigation](#gamepad-and-keyboard-navigation)
  - [Pad button pictures](#pad-button-pictures)
  - [Questions and tooltips on a pad](#questions-and-tooltips-on-a-pad)
  - [Closing, and where focus goes back](#closing-and-where-focus-goes-back)
  - [Hotbar focus and the focus glow](#hotbar-focus-and-the-focus-glow)
  - [The bag screen on a pad](#the-bag-screen-on-a-pad)
  - [Loot windows and vendors on a pad](#loot-windows-and-vendors-on-a-pad)
  - [A screen with more than one inventory grid](#a-screen-with-more-than-one-inventory-grid)
  - [A spatial bag is walked as it is drawn](#a-spatial-bag-is-walked-as-it-is-drawn)
  - [A packed pouch is walked as it is drawn](#a-packed-pouch-is-walked-as-it-is-drawn)
- [See also](#see-also)

## Enhanced Input — the normal route

- The plugin applies its own mapping context at priority 0 when `bAddInventoryMappingContext` is on (the default).
  These settings are on **Project Settings → Plugins → Snap Inventory System - UI Core → Input**; the interact ones on
  **Snap Inventory System - Interaction → Interaction → Input**.
- It registers that context, so its actions appear in a rebinding screen.
- **To drive input entirely yourself**, do one of these:
  - Clear `bAddInventoryMappingContext` and add `IMC_SnapInventorySystem` to your own contexts.
  - Map your own actions onto the plugin's settings (`ToggleInputAction`, `InteractInputAction`, `ConsumeInputAction`, …).

### The shipped contexts and actions

Two contexts ship:

- `/SnapInventorySystem/Core/Input/IMC_SnapInventorySystem` — screens and world actions
- `/SnapInventorySystem/Core/Input/IMC_SnapHotbar` — the hotbar keys

The actions are ordinary `UInputAction` assets under `/SnapInventorySystem/Core/Input/`:

| Action | Ships mapped to |
|---|---|
| `IA_Inventory_Toggle` | `I` (and `Tab`, through `AdditionalToggleKeys`; during play `Gamepad_FaceButton_Top` opens the bag through `GamepadOpenKeys` — both below) |
| `IA_Interact` | `E` (no trigger, so it can be held — see below) |
| `IA_InteractCycle` | *(Unmapped)* |
| `IA_Inventory_Consume` | `U` |
| `IA_UI_Rotate` | `R` (turns the item being dragged), `Gamepad_RightTrigger` (only an item carried or dragged) |
| `IA_UI_QuickMove` | `Gamepad_FaceButton_Left` (no keyboard key) |
| `IA_UI_ContextMenu` | `Gamepad_FaceButton_Top` (in a screen; no keyboard key) |
| `IA_UI_Split` | *(Unmapped)*; the item menu's Split row splits on a pad |
| `IA_TakeAll` | `Gamepad_RightShoulder` (with a loot window open; no keyboard key) |
| `IA_Sort` | *(Unmapped)*; the bag screen's sort control is a pad stop |
| `IA_UI_SectionPrev` / `IA_UI_SectionNext` | `Gamepad_LeftShoulder` / `Gamepad_RightShoulder` (the bag screen's sections; no keyboard key) |
| `IA_UI_FilterPrev` / `IA_UI_FilterNext` | `Gamepad_LeftTrigger` / `Gamepad_RightTrigger` (the bag's filter tabs; no keyboard key) |
| `IA_Drop` | *(Unmapped)* |
| `IA_Hotbar_Slot01` … `IA_Hotbar_Slot10` | `1`–`9`, then `0`; slots 1–4 also on the d-pad (up, right, down, left) |
| `IA_Hotbar_PageNext` / `PagePrev` | *(Unmapped)*; a raw modifier-plus-wheel binding exists behind `bBindHotbarPageKeysRaw`, which ships off |
| `IA_Hotbar_Consumable01` / `02` | *(Unmapped)* |

- **A chorded pad mapping of a slot is drawn on the square as both buttons** (`USnapInventoryHUDComponent::ResolveBarSlotChord`).
  - The demo's X + the d-pad for 5–8 is one.
- **Hotbar slot actions are zero-padded to two digits on purpose.** A settings screen that sorts rows case-insensitively
  puts "Hotbar Slot 10" before "Hotbar Slot 2" otherwise.

### Rebinding rows

**Every action has its own rebinding row.**

- Enhanced Input keys a player's rebinding rows by the action's Player Mappable Key Settings `Name` (`SnapInventoryToggle`,
  `SnapInventoryRotate`, ...).
- So two actions that share a name share ONE row: one label, both actions' keys under it.
- Give each action in a copy its own name.
- `SnapInventory.Input.ShippedMappingNamesAreUnique` fails if two shipped actions — or two mappings in a shipped context —
  answer to one name.
- Each row's name and category are keyed text (see [Localizing](localization.md)).

### Hooks for your own keybinding feature

Two delegates on `FSnapInventorySystemModule` let a project's own keybinding feature take part. Bind each once from your
own module's `StartupModule`, and unbind it in `ShutdownModule`.

- **`OnResolveInputMappingContext()`** swaps the InputMappingContext the plugin applies for a local player.
  - Unbound, or returning the source context, changes nothing.
  - It exists for a feature that applies a different asset than the one asked for. For example, a "requires modifier"
    setting that duplicates the context, adds a chord trigger to flagged actions and swaps the duplicate in: a plugin that
    called `AddMappingContext` directly would be invisible to it.
  - The resolved context is also the one `HasMappingContext` is asked about.
  - It hands over a `ULocalPlayer`, so the plugin names no game module.
- **`OnDoesActionRequireModifier()`** answers whether an InputAction currently needs a modifier held, so a chorded hotbar
  square can show a marker instead of a bare "3".
  - Unbound means never; only the marker is lost.
- Both are function-local statics, so they work before this module starts.

### The interact action

**Interact must be holdable.**

- The `IA_Interact` row in `IMC_SnapInventorySystem` carries no trigger.
  - So it reports `Started` on the press and `Completed` on the release.
  - The detector binds those, never `Triggered` (which repeats every frame while a key is held).
- **A `Pressed` trigger on that row breaks holds.** It reports the release one frame after the press, which ends every
  hold before it can run.
  - A copy of the context made before holds existed has one, and should lose it.
- The detector logs once when a hold is released within a frame of starting.

**A project with its own interaction system** that keeps `E` for itself can leave `IA_Interact` with no key.

- The plugin's pickups and chests then prompt with no key and cannot be used.
- To make them usable, do one of these:
  - Map the plugin's action.
  - Point `InteractInputAction` at your own action (with no trigger on its mapping, as above).
  - Call the detector from your handler.
- See [Loot and interaction](loot-and-interaction.md), *A project with its own interaction system*.

**To cycle between a target's options, map `IA_InteractCycle`** (the `InteractCycleInputAction` setting) in your own
context or in a copy of the plugin's.

- The prompt's count names the key once one is mapped: "1/2 (Mouse Wheel)".
- The example demo maps the mouse wheel, both directions, in its own context.
- It ships with no key, so installing the plugin claims none.
  - That also means a rebinding screen lists it only once it is mapped.
- It is bound on `Started`, so each press steps once.

### Keys and buttons that are settings

**More keys for the bag.** `AdditionalToggleKeys` (ships `{Tab}`) are keys that do what `ToggleKey` does.

- **They are a setting, not a mapping.**
  - The HUD component maps them at run time in a small context of its own, to a run-time action of its own.
  - So they are not on a rebinding screen, and `IMC_SnapInventorySystem` does not change.
  - The `bBindToggleKeyRaw` flag raw-binds them too.
- **They close the screen or window that holds keyboard focus:** A screen, a container window or a loot window.
  - They close it from `NativeOnPreviewKeyDown`, before a focused square or button can turn Tab into focus navigation.
  - Not while Shift, Ctrl or Alt is held, and not in a text box.
- **A project that uses Tab for itself** writes `^AdditionalToggleKeys=` in its `DefaultGame.ini`.

**A pad button that opens the bag.** `GamepadOpenKeys` (ships `{Gamepad_FaceButton_Top}`, Y) are pad buttons that OPEN the
bag during play.

- **They never close anything.**
  - The HUD component maps them at run time in a small context of its own.
  - That context is applied only while nothing of the plugin's is open (no screen, container window, loot window, shop
    or trade window), and taken off the moment one opens.
  - So inside the bag Y is the focused item's menu again (and the most in an amount window), and B closes.
- **The context sits one priority above `InventoryMappingContextPriority`.**
  - So during play it outranks the shipped context's own Y.
  - A project's own context at that priority or lower loses the button while nothing of the plugin's is open.
- **Like the keys above, the buttons are a setting:**
  - Not on a rebinding screen.
  - Never raw-bound.
  - `IMC_SnapInventorySystem` unchanged.
  - They need `bAddInventoryMappingContext`.
- **A project that uses the button during play** (a reload, a jump) writes `^GamepadOpenKeys=` in its `DefaultGame.ini`,
  or lists other buttons (`+GamepadOpenKeys=Gamepad_FaceButton_Left`).

**Hold the middle mouse button for a cursor.**

- `CursorPeekKey` (ships `MiddleMouseButton`) held during play shows the cursor and holds the camera still.
  - So the HUD's squares can be pointed at for their tooltips.
  - Released, play resumes.
- `USnapInventoryHUDComponent::BeginCursorPeek` / `EndCursorPeek` / `IsCursorPeeking` drive it from your own input.
- The press that starts a peek and its release are taken from the game.
- `CursorPeekKey=None` turns it off.

## UI gestures

- Inside a screen, what a click does is decided by one table rather than by hardcoded tests in each widget.
- It has fourteen gestures:

```
Primary        ContextMenu    AutoMove      ToggleSelect
RangeSelect    OpenContainer  PreviewOrbit  SplitSingle
SplitHalf      Close          Confirm       Activate
UseItem        ShowShortcuts
```

- Each row names a mouse button or key plus a modifier requirement.
- A modifier is `Ignored`, `Required` or `Forbidden` — so "shift-click" and "click without shift" are distinct,
  expressible rows rather than a bool somebody has to remember to test.

**The defaults are conventional:**

| Gesture | Does |
|---|---|
| Left-click | Pick up |
| Right-click | The menu |
| Shift-click | Quick-move |
| Ctrl-click | Add to a selection |
| Shift-click a second square | Select a range |

- **A right-click on another item while the menu is open opens that item's menu.**
  - That works on an item under the open menu too: its right-click closes the menu and opens the menu of the square
    beneath (`USnapInventoryContextMenuAnchor::ReopenContextMenuUnderCursor`).
  - A second right-click on the same item closes it.
- **A plain click on a square that is already selected** narrows a selection of several to it.
  - When it is the only one selected, the click lets it go.
  - This works on the bag grid, in a pouch window and in a loot window.
  - The release decides, so a drag of it still works.

**A double-click does the obvious thing with the item.**

- `OpenContainer` and `UseItem` share one chord: a left double-click with no modifier.
- The slot tells them apart by the ITEM:
  - A storage item (a pouch) opens.
  - Anything else is offered to the action menu's **Consume**, through the same code the menu uses
    (`USnapInventorySlotWidget::TryUseItem`, on the bag grid and in a pouch window).
- So double-clicking a potion drinks one.
- Whatever the menu's Consume refuses, the double-click refuses too: a broken or spoiled item, an item with nothing to
  apply.
- A locked potion is drunk by both (a lock stops you losing an item, not using it — see
  [Restrictions and marks](restrictions-and-marks.md)).
- It uses the item under the cursor, never the whole selection.
- The `bConsumeOnDoubleClick` setting (on by default) switches it off without touching the row.
- A modified double-click still reaches the press handler, as it always has.

**Hold Alt to read an item's shortcuts.**

- `ShowShortcuts` is a key row that is HELD rather than pressed.
- While one of its keys is down, every item tooltip turns its dim "Hold Alt for shortcuts" line into the list of what that
  square does with that item — see [Tooltips and stats](tooltips-and-stats.md#shortcuts-hold-alt).
- It ships as Left Alt.
- A modifier key counts on **either side** (the rule every chord in this table follows), so either Alt key works.
- `USnapGestureLibrary::IsGestureKeyHeld` answers "is it held right now" for any key row:
  - A modifier, from Slate's modifier state, wherever focus is.
  - Any other key, from the player controller, which only knows keys that reached the game viewport.
  - So rebinding it to a letter a screen widget handles itself may never read as held.
- A modifier is the safe choice.

Re-point a gesture in **Project Settings → Plugins → Snap Inventory System - UI Core → Input → Gestures**.

## Raw key bindings

**Every raw binding ships disabled.**

- A raw `FInputKeyBinding` is registered straight on the PlayerController's `UInputComponent`.
- It works with zero setup, which is why it exists.
- It is the wrong default for a project that already owns its input, for two reasons:

1. **A key can be claimed twice and the bind still reports success.**
   - If an applied mapping context maps the same key to an action that *consumes* input, two mechanisms compete for one
     press — and `BindKey` still logs `'<key>' bound to ...`.
   - ⚠ Which way it breaks depends on where your project binds its actions.
   - `APlayerController::BuildInputStack` pushes the *pawn's* input component before the *controller's* and evaluates the
     stack in reverse.
   - So a raw bind on the controller can run **in addition** to a pawn-hosted action rather than being swallowed by it:
     one press, two actions.
2. **A raw binding is invisible to a rebinding UI.** A player who rebinds some other action onto that key silently kills
   yours.

**The plugin detects (1) and warns**, naming the conflicting action.

- It warns only when that action actually consumes input.
- A non-consuming mapping is not a conflict, and warning on it told a working keybind it was dead.

The seven flags, all `false`, are spread over five pages: `bBindToggleKeyRaw` and `bBindConsumeKeyRaw` (**Snap Inventory System - UI Core
→ Input → Keys**), `bBindInteractKeyRaw` (**Snap Inventory System - Interaction → Interaction → Input**), `bBindVendorKeyRaw`
(**Snap Inventory System - Economy → Vendor**), `bBindHotbarPageKeysRaw` (**Snap Inventory System - Equipment → Hotbar → Pages**),
`bBindTradeDebugKeyRaw` (**Snap Inventory System - Economy → Trade**), `bBindDurabilityDebugKeyRaw` (**Snap Inventory System - Items →
Durability → Debug**).

**The vendor, trade and durability keys are development aids.**

- They are compiled out of Shipping builds: there those keys are never bound, whatever the flags say.
- The durability key is also bound only while `bEnableDurability` is on, and only then reported as contested.

⚠ **Turning a raw bind on is not enough on its own.** A raw bind **off** plus an **empty** action row is a screen with no
way to open it. Set one or the other.

## Gamepad and keyboard navigation

- Screens are focus-navigable.
- A controller can move focus between squares, carry an item from one to another, work the shop and the craft queue, and
  answer a modal.
- The focus helpers live in `USnapFocusLibrary`.
- With more than one local player, each player's pad moves focus only in that player's own screens.
  - A screen or window takes focus for the Slate user of the player who owns it (`SetUserFocus`), not the first player's.
  - So a second player's bag opens with that player's pad on its first square, and the first player's focus stays where it was.
  - This holds with split screen turned off too: it follows the number of local players, not the layout.
  - With one local player a screen takes keyboard focus, as it always has.
  - The mouse and the keyboard are the first player's (Slate user 0), whoever owns the screen under them.
  - See the README's [Split screen](../../README.md#split-screen) for the layout, the fit and what every player shares.

### Pad button pictures

**A controller button is drawn as a picture of the pad the player holds.**

- The interaction prompt, the hotbar squares and an item tooltip's pad shortcut list draw a pad key as its picture from
  `GamepadGlyphs`.
- `GamepadGlyphs` is the plugin's own `USnapGamepadGlyphSet`, `DA_SnapGamepadGlyphs` under
  `/SnapInventorySystem/Core/Input/Glyphs`.
- Its pictures are NEUTRAL stand-in pictures drawn for this plugin, in two sets:
  - One set follows the Xbox controller LAYOUT: plain monochrome A / B / X / Y, LB / RB, LT / RT, View / Menu.
  - One set follows the PlayStation LAYOUT: a direction mark for each face button (four dots, the pressed button's
    position filled: bottom, right, left, top), L1 / R1, L2 / R2, and plain marks for Create / Options.
  - The two sets share the D-pad and the sticks.
  - They are plain shapes and letters with no brand colors, console logos or console symbols.
- The prompt's words then leave the key out.
- The `GamepadGlyphFamily` setting picks the set:
  - `Auto` (the default) follows the pad the player used last.
  - `Xbox` or `PlayStation` forces one.
- The console names say only which controller layout a set follows; the pictures are not official art.

**A developer licensed to use a console maker's button art** does this, and nothing else changes:

1. Copy `DA_SnapGamepadGlyphs`.
2. Put that art in the copy's rows: one row per pad key, a picture per family.
3. Point `GamepadGlyphs` at the copy.

**What `Auto` can tell apart.**

- It reads the hardware the engine records for the pad the player pressed LAST.
  - That is their most recent device (`UInputDeviceSubsystem::GetMostRecentlyUsedHardwareDevice`) when that is a pad.
  - It is remembered for each player, so a keyboard player shown a pad key still gets the pad they hold.
- It draws the PlayStation set when that name is one of `PlayStationHardwareIdentifiers` ("DualSense" and "DualShock4"
  by default).
- Any other pad, and a player who has not used a pad yet, gets the Xbox set.
- **It does not use `GetLatestDeviceOfType` alone.**
  - In 5.8 that call keeps the FIRST pad of a session.
  - So a player who swapped an Xbox pad for a DualSense would keep the Xbox pictures.
  - It is only the fallback before a pad press has been seen.
- ⚠ **A PlayStation input plugin should declare its device in `Input.ini`** (`HardwareDevices`,
  `PrimaryDeviceType=Gamepad`), as the engine's WinDualShock does.
  - Without it, the engine records that pad's presses as an unknown kind of device, and the plugin's "is this player on a
    pad?" answers no.
- **In 5.8 the engine records a pad's hardware only from an input device REGISTRY entry:**
  - XInput registers "XInputController" (an Xbox pad — and a PlayStation pad that Steam Input or DS4Windows presents as
    XInput).
  - GameInput registers "XboxOne", "Xbox360" or "Hid".
  - The engine's own WinDualShock plugin registers none (it pushes only the legacy input scope, which 5.8 no longer reads).
- **So on a stock Windows project a DualSense most likely draws the Xbox set.** To change that, either:
  - Set the family to `PlayStation` for a PlayStation-first release.
  - Or add the name your PlayStation input plugin registers to the list.

**Where a key stays text, it reads by its short name in the same family.**

- A key stays text in these places:
  - The option count's cycle key.
  - A hotbar key with its modifier marker.
  - A tooltip shortcut line with no picture for its button.
  - A widget of yours with no picture part.
- The names come from `SnapInventoryText::GetShortKeyLabel`. They are keyed text, so a translation can rename them.

| Pad | Short names |
|---|---|
| An Xbox pad | A, B, X, Y, LB, RB, LT, RT, View, Menu |
| A PlayStation pad | Cross, Circle, Square, Triangle, L1, R1, L2, R2, Create, Options |
| Both | An arrow for each D-pad direction, L3, R3, LS, RS |

- A keyboard or mouse key is always text, named as the engine names it.

**Your own art and your own widgets.**

- Point `GamepadGlyphs` at your own set (one row per pad key, a picture per family).
- Or clear it to draw every key by its short name.
- `USnapInputGlyphLibrary` answers the same questions for a Widget Blueprint of yours:
  - `GetKeyGlyph`: the picture, or none.
  - `GetKeyLabel`: the words.
  - `GetGamepadGlyphFamily`.
- A Widget Blueprint square or prompt names its picture part `HotbarSlotKeyGlyph` (`KeyGlyphWidgetName`) or
  `InteractionPromptKeyGlyph`.
- A square without one gets an Image built beside its key text when that text sits in an Overlay or a CanvasPanel.
  Otherwise it keeps the text.
- A tooltip's pad rows go in `ShortcutRows` (`ShortcutRowsWidgetName`), else in a box built under its shortcut text.

**`FSnapInventorySystemModule::OnResolveInputGlyph` still comes first.** A key it answers is drawn as its text, never a
picture.

### Questions and tooltips on a pad

**On a pad, a one-press loss asks first** (`bConfirmPadItemLoss`, on by default).

- When the player's last input was a gamepad, a single press that drops, sells, destroys or dismantles items raises a
  confirmation window first. These presses ask:
  - The item menu's **Drop** row.
  - A project row whose action has `bCostsTheItem`.
  - The vendor's **SELL JUNK**.
  - The item menu's **Sell** while it sells at once (`bConfirmVendorSales` off, the default).
- A mouse or keyboard press acts at once, exactly as before.
- What already asks is not asked twice (DROP ALL, Dismantle, Repair, Upgrade).
- A sale through the sell window or the vendor's SELL button is already a second step that shows what goes and for what.
- The confirmation then names the pad's answer keys under its buttons: the accept button's picture and "Select", the back
  button's and "Cancel".
  - That is because a pad's accept presses the FOCUSED button, never a fixed one.

**On a pad a confirmation opens on its action.**

- Every confirmation opens with focus on its action button when a gamepad was the last input: DROP?, DROP INVENTORY,
  Repair, Upgrade, Dismantle.
  - So A answers the question the player asked, and B declines.
- A keyboard or mouse player gets CANCEL first, so Enter on a fresh question declines.
- **A trade invite (and the trade wait) keeps CANCEL first on a pad too**, because it appears unasked during play, where A
  may be a game button (`USnapInventoryConfirmWindowWidget::bDeclineFirstOnGamepad`).
  - `FocusDefaultButton` / `ResolveDefaultFocusButton` pick the button; `FocusDeclineButton` still asks for CANCEL.
- **A refusal asks nothing.** With `NoticeStyle` **Prompt** (the default) its reason shows as a white line that fades on
  its own: there is no OK to press, and the focus stays where it was. **Window** brings back the OK window.
- **The press that opened a question never answers it.** A held button's repeats are ignored until a fresh press
  (`FSnapFreshAcceptGate`, also on the amount and sell windows).
- Your own UI asks the same way through `USnapInventoryHUDComponent::AskBeforePadItemLoss`. It returns false when no question
  is needed, and the caller acts at once.

**On a pad, the focused square shows its tooltip** (`bShowFocusedTooltipOnGamepad`, on by default).

- Slate shows a tooltip only under the mouse pointer.
- With a gamepad the player's last input, the item square that has focus shows the same tooltip beside it.
- The card follows each focus move, its pad shortcut line included, so a pad player reads what each button does with that
  item.
- It hides on a mouse or keyboard input, during a drag or a carry, under the item menu or a confirmation, and when the
  screen closes.
- See [Tooltips and stats](tooltips-and-stats.md#on-a-pad-the-focused-square-shows-its-tooltip).

**On a pad, the item menu opens at the focused square.**

- Y (`IA_UI_ContextMenu`) opens the menu with its top-left at the middle of the square that has focus.
- Near an edge it flips and stays inside the view, as a right-click's menu does.
- A key you map to `IA_UI_ContextMenu` opens it at the focused square too.
- A right-click still opens it at the cursor.

**A window a pad opens from a square, a menu row or a currency opens beside it.**

- The item menu's Split Stack window, its Dismantle question and the pouch window its Open opens appear at the row you
  pressed.
- Pressing the row with Enter does the same.
- The amount window that Accept on a currency opens appears at that currency.
- A key you map to `IA_UI_Split` opens its split window at the focused square.
- With the mouse they open at the cursor, as before.

### Closing, and where focus goes back

**The pad's back button closes.**

- The `Close` row of `GestureBindings` holds Escape and the gamepad's back button.
  - The back button is `Virtual_Gamepad_Back`: B on an Xbox pad, Circle on a PlayStation one (the virtual key, matched per
    platform).
- Every close path asks that row: B cancels a carry, then closes the topmost window, then the screen, as Escape does.

**After a window closes, focus goes back to the square you were on.**

- This applies when a window or a confirmation closes: the stack-split window, "DROP?", the craft or loot window,
  a pouch window.
- Keyboard focus goes to whatever is still open on top, as before.
- **Where that is a pouch window,** focus goes to the square you were on inside it.
  - Or back to the bag square you were on, if you had moved there since the pouch opened.
  - Else (on a pad) the pouch's first square.
- **Where that is a screen itself** (the bag, a vendor or a trade screen), focus now goes to the item square that last had
  focus instead, when that square is still shown in that screen.
  - So a pad or keyboard player carries on from where they were, its focused-square card included.
- **It goes back the way it came.** A square the mouse pressed is not grown as if a key had focused it.
- **A square that has gone or is hidden is passed over**, and the screen takes focus, as before.
  - Gone: a grid cell another item now covers, or a square of a closed pouch window with no opener (a trade partner's).
  - Hidden: a filter, a collapsed panel.
- The memory is forgotten when a screen closes.
- **The bag's craft window gives focus back to what opened it, its CRAFT button,** when the player opened it, floating or
  docked.
  - A crafting station's view or window keeps the square-you-were-on rule above.
  - A station's view that closes the bag its press opened gives focus back to the game.
- **A pouch window does the same with the square that shows its pouch** (the one its Open or a double-click was on), when
  focus was in the window at the close.
  - B and Escape close every pouch window, the topmost first. A pouch opened inside another gives focus back to its square
    in the outer window, and the outer one to the bag square.
  - When focus had moved out of it (LB to the bag), it stays there.
- `USnapInventoryHUDComponent::GetLastFocusedItemSquare` returns it.
- A square of your own is remembered through `ReportItemSquareFocus`, the call it already makes for the focused-square
  card.

### Hotbar focus and the focus glow

**The hotbar a pad reaches is the one on screen.**

- A pad reaches the bars the open screen draws itself (`USnapEquipmentPanel::GetEmbeddedHotbars`: the example Character
  screen's Ability bar and its consumable squares).
  - They are the bars of the first equipment panel the player can see that draws any. A panel inside a collapsed or
    hidden container is passed over, and so are its bars.
  - Where two or more are drawn, each bar is its own row, linked by where they are drawn, with focus entering on the top
    one.
- A screen that draws none reaches the HUD's bars that are shown, all of them.
- A HUD bar a host hides while a screen is up is never a focus target.
  - An explicit navigation rule into a hidden square would still move focus there.
- A natively built hotbar square draws a focus ring while it has focus: a white outline, its part named by
  `FocusedGlowWidgetName`.
- A Widget Blueprint square draws its own `FocusedGlow`, if it authors one.

**A mouse click lights no square.**

- Slate gives keyboard focus to the square under every mouse press.
- So every square (bag, pouch, equipment, loot and hotbar) draws its focus glow only:
  - For focus the keyboard or a pad gave.
  - Or for a mouse press's focus while a gamepad was the last input.
- `USnapFocusLibrary::ShouldDrawFocusGlow` is that rule, and the focus ring follows it too.
- A focused square re-asks every frame, so picking up a pad shows the glow and going back to the mouse hides it.
- The focus itself is unchanged: only what is drawn.

### The bag screen on a pad

- **On the bag screen, everything a pad can stop on is wired as one set of SECTIONS**, linked by where each is drawn:
  - The inventory grids.
  - The paper doll's equipment column.
  - Each hotbar bar.
  - The screen's buttons and currencies, in rows.
- The trade screen is not wired this way: it keeps its own fixed layout.
- From the edge of a section, the D-pad goes to the section drawn on that side. It lands on the square or button in line
  with the one it left.
- A side with nothing drawn there keeps the panel's own link (a pouch window to the bag's right).
- **A locked square** (the room an upgrade would add, `bShowUnboughtBagSquares`) is never a stop: Down from the last row
  with open squares goes on to the section below. An upgrade that opens squares makes them stops.

On the example Character screen:

| From | Down | Up | Left | Right |
|---|---|---|---|---|
| The bag's last row | The filter and sort row | — | The worn slot in its row | A pouch window, if open |
| The filter and sort row | CRAFT, DROP and the currencies | The bag | The paper doll | — |
| CRAFT, DROP and the currencies | The Spells grid | The filter and sort row | The paper doll | — |
| The Spells grid | The hotbar | CRAFT, DROP and the currencies | The hotbar | — |
| The boots | The potion squares | The slots above (the map) | — | — |
| The potion squares | The hotbar | The boots | — | The Spells grid |

**Every button and currency is a stop.** Moving focus reaches:

- Every focusable button the bag screen shows, in its own tree or in the trees of the panels it holds: the filter tabs,
  the sort control, CRAFT, DROP, a bank's Take All, a button of your own.
- Each currency of the wallet row (`USnapCurrencyStopWidget`, laid over the entry).

Within that:

- Buttons on one drawn line are one row. A currency the wallet wraps onto a second line stays in its row.
- A window's close box is not a stop: Back closes a window.
- A disabled or hidden button is passed over.

**A focused button or currency draws an outline** (`USnapFocusRingWidget`, a rounded outline in the theme's
SelectionGlow).

- It draws while a pad is the last input or focus came from the keyboard.
- `FocusRingWidgetClass` takes a Widget Blueprint of your own (its root a CanvasPanel, the ring the Border named by
  `RingWidgetName`).
- A UButton has no focused look of its own.
- **Every popup draws the same outline on its focused button:** A confirmation, the amount window, the sell window, the
  trade screen.
  - It draws while a pad is the last input or focus moved by the arrow keys.
  - The focus a popup places by itself when it opens shows nothing to a mouse player (`ShouldOutlinePopupFocus`).
  - The item menu and the sort list draw it inside the focused row. They are drawn above every other layer, so the ring
    is part of the row there (`FSnapRowFocusOutline`).
- **The D-pad never leaves a popup.** Every edge of its buttons is a stop, worked out from where the buttons are drawn
  (`USnapFocusLibrary::ApplyPopupNavigationByLayout`).

**Accept on a currency opens its amount window** (`USnapInventoryPanel::OpenCurrencyAmountWindow`).

- That is the window a Ctrl-click opens, choosing how much to drop.
- It never drops the whole balance.
- With no amount window configured (`StackSplitWidgetClass` empty), it does nothing.
- On a pad the window opens with DROP focused:
  - The D-pad walks -, +, DROP and CANCEL.
  - A presses the focused one (DROP drops the amount shown).
  - B cancels.
- The item menu's **Split Stack** window works the same way.

**The sort control on a pad.**

- Accept opens the list with focus on the current mode.
- Up and Down walk it, Accept picks a mode, Back closes it, and focus comes back to the sort control.
- A mouse click opens it as before, without moving focus.

**The bumpers and triggers.** LB / RB move focus to the previous / next SECTION, in this order:

1. The paper doll (or the recipe list of a craft window docked in its place).
2. An open loot window.
3. The bag.
4. The topmost open pouch window.
5. What is drawn with the bag, top first: its filter row, its CRAFT / DROP / currency row, the other grids such as
   Spells.
6. The hotbar's bars, top first: the potion squares, the hotbar.

- LB / RB wrap from the last to the first, and land on the section's first square or button that can take focus.
  - A section the filter has emptied is passed over.
- **LT / RT step the bag's FILTER TABS**, All first and wrapping.
  - They step the bar of the panel that holds focus, else the main bag's.
  - When the filter hides the square that held focus, focus moves to the first square still drawn, or to the tab.
- **With a pouch open,** RB from the bag enters the pouch and LB comes back.
- **With a loot window open,** LB from the bag enters it.
- **Two buttons keep a second job:**
  - With a loot window open, RB is Take All and steps nothing.
  - While an item is carried or dragged, RT turns it and steps nothing (with nothing held, a pad's RT turns nothing).
- Split is in the item menu (Y, then Split); sort is the sort control.
- With the bag screen closed, the four do nothing.
- `USnapInventoryHUDComponent::StepBagScreenSection` / `StepBagScreenFilter` are the same steps, Blueprint-callable for a key
  of your own.
- The Verbose line `LB / RB step through N section(s): ...` names the order.

**A bag that scrolls is walked to its last row.**

- Each square that takes focus is scrolled into view (the flowing and the grid layouts).
- Focus leaves the grid only from its last row.

**The right stick turns the paper doll.**

- With the bag screen (or a shop) open, the right stick turns the character preview like a drag.
  - Right turns the figure's right side toward you; up tilts the view up.
- R3 puts the view back.
- The stick is only observed (`USnapInventoryHUDComponent::GetPadPreviewStick`), so nothing else loses it.
- Turning off `bAllowPreviewGamepadControl` on the preview switches it off.

**The paper doll moves the way it is drawn, and you can map it.**

- `USnapEquipmentPanel::SlotNavigation`, in the equipment panel's class defaults beside `SlotColumns`, is a per-slot map.
- For each of Up, Down, Left and Right, it names one of these:
  - Another slot on the panel.
  - The inventory grid (the square in line).
  - A hotbar bar by its tag (the square in line).
  - Stop.
- A direction left Automatic keeps the plugin's own choice, so a row needs only the moves that differ.
- The map is applied after the sections are linked, so it wins.
- The example `WBP_ExampleEquipmentPanelCharacter` carries the shipped map:
  - Across a row, from one column to the other.
  - The gloves and the greaves down to the boots.
  - The boots down to the potion squares.
  - The chest up to the head.
  - The right column into the bag, in line.

**The Verbose log lines name every section**, its rectangle, and what each of its four sides leads to:

- The line `focus chain over N section(s) by where they are drawn: ...`
- The line `N button / currency stop(s) on the bag screen in N row(s): ...`

**In a project with CommonUI** (a game viewport client derived from `UCommonGameViewportClient`, as Lyra's is):

- CommonUI's analog cursor turns a pad's A into a simulated mouse click at a cursor it parks on the focused widget.
- The plugin hands that A to the focused item square or currency first
  (`USnapInventoryHUDComponent::TakeGamepadAcceptForFocusedSquare`).
  - So the carry and the amount window work as they do everywhere else.
  - A button still takes the click.
- The pad pictures follow CommonInput's answer about the last device.
- The focused square's card hides only under a pointer that is drawn.

### Loot windows and vendors on a pad

#### Pouch windows

**A pouch window opens with its first square focused** whenever the plugin would place focus for a pad
(`GamepadAutoFocusPolicy`).

- That holds however it opens: the item menu's Open, a double-click, any `USnapInventoryStorageWidget`.
- The D-pad walks inside it, and A picks an item up.
- A direction off its edge goes back to the bag, and A on a bag item swaps the two.
- **Opened with the mouse,** the window holds focus itself, and **any direction starts focus** on its first square
  (`USnapInventoryStorageWidget::NativeOnNavigation`).
- **A row of the item menu that opens a window** (Open, Split Stack, Sell) leaves focus to that window.
  - One that opens nothing hands focus back to the square the menu was raised on.
- B or Escape closes it and gives focus back to the pouch's square (see
  [Closing, and where focus goes back](#closing-and-where-focus-goes-back)).

#### Loot windows

**A picks the focused loot item up; X takes it straight into the bag.**

- A chest's or a body's item is carried like a bag item (the operation a mouse drag builds,
  `USnapInventoryLootSlotWidget::BuildDragOperation`).
- You put it down with A where you choose: a bag square or grid cell, a pouch square, a worn slot.
- **On a bag square or grid cell** (`USnapInventoryComponent::TakeFromLootContainerToSlot`):
  - A free square takes it.
  - The same stackable item takes the units.
  - A different item gives up its square and moves to the first free one.
  - With the bag full, a stack of the same item with room still takes what fits, and the rest stays in the chest.
  - With the bag full, anything else is outlined red: A sends nothing, and the take's own notice says
    "Your inventory is full."
- RT turns it over a grid bag, and a put-down a grid paints red keeps it in hand.
- **A on the loot window puts it back, and B cancels.**
  - Nothing leaves the container before the put-down.
  - Closing the window ends the carry.
- **X (quick-move) takes it into the bag** (`TakeIntoInventory`), as the mouse's Shift+click does.
  - With the bag full, it goes into the bag's stacks of the same item with room, the rest staying in the chest.
- RB still takes everything that fits, letting go of a carried chest item first.
- With `bEnableGamepadCarry` off, A takes it into the bag as X does.
- A carried bag item over a loot square shows the refused outline, and A there keeps it in hand: nothing goes into a chest
  or a body.

**A loot window opens with its first item focused** whenever the plugin would place focus for a pad
(`GamepadAutoFocusPolicy`).

- That is any `USnapInventoryLootWidget`: a chest or a body. The example demo's bank is not a loot window: it opens the bag
  screen, so a pad starts on the bag's first square.
- An empty window focuses its close box: Take All is disabled while the window is empty, and a disabled button cannot
  hold focus.
- Taking the last item with the window open moves focus to its close box too, so A closes the window (and the bag it
  opened).
- Opened with the mouse, the window holds focus itself, and then **any direction starts focus** on that first item
  (`USnapInventoryLootWidget::NativeOnNavigation`).

**Take All and the close box are one move away.**

- Down from a square with nothing under it reaches Take All.
- Up from the top row reaches the close box.
- Each leads back to the squares in line.
- The one holding focus draws the focus outline (the window's own `USnapFocusRingWidget`, from `FocusRingWidgetClass`, one
  above the window) while a pad is the last input or a key moved focus there, as a popup's does: with the mouse last,
  the focus the window moves by itself (onto the close box after the last take) draws none.
- B (Back) still closes the window.
- The squares scroll into view as focus walks them, so focus leaves the list only from its last row.

#### Vendor screens

**A vendor screen is wired the bag screen's way.** These are its sections, linked by where they are drawn:

- Its row list (the active tab's: Buy/Sell, Repair/Upgrade or Craft).
- Its tab strip.
- Its buttons: the shop's filter tabs, the center column's quantity buttons, BUY, BUY STACK, REPAIR and CANCEL.
- The bag side's grid, filter tabs, sort control and wallet.

Buttons are grouped into rows per panel, so two columns' buttons at the same height stay apart.

On the example General Store:

| From | Press | Reaches |
|---|---|---|
| A shop row | Right | The center column (the quantity row, then BUY, BUY STACK and CANCEL below it) |
| The center column | Right | The bag |
| The center column | Left | The row the column shows (on the Craft tab, the selected recipe) |
| The first row | Up | The tabs |
| The last row | Down | The shop's filter tabs |
| The bag's last row | Down | Its filter and sort row, then the wallet |

- A button that turns on (BUY once an item you can afford is picked) joins the map when it does.
- The shop list, the Repair/Upgrade list and a craft station's recipe list and queue scroll with focus.

#### The crafting menu

**The crafting menu on a pad.** Wherever a craft panel is up — the bag's craft window, docked or floating, a shop's Craft
tab, a crafting station's view:

- Its recipe rows and its queue are sections drawn where they are.
- Its buttons are stops.
- Its own exits are written over the rest (below).

**The panel's own exits:**

- Left from the details column (-, +, CRAFT, DISMANTLE, a queue's CLEAR there) goes to the SELECTED recipe (the first the
  panel would pick when none is).
- Up from the craft filters goes to the SELECTED recipe too.
- Right from a recipe goes to the details control in line with it, else the column's first.
- Up / Down stay inside the details column.
- Down from the last recipe reaches the craft filter row.
- The map follows the buttons. When an item turns CRAFT, DISMANTLE, CLEAR or a queued craft on or off while the panel is up,
  the map is rebuilt at once, so the D-pad reaches a CRAFT that has just turned on.

**The queue:**

- A craft you have just queued joins the map once it is drawn (a frame later).
- Where the queue sits in the details column, Down from CLEAR reaches it.
- Left from its first craft goes back to the selected recipe.
- A cancels the focused craft.
- Focus moves on to the next queued craft (else the selected recipe) when the one it was on is canceled or done.

**Where focus starts:**

- A crafting station's view, and a station's craft window on its own, open with focus on the selected recipe.
  - A station's craft window on its own is its own focus host: the bag screen behind it is not part of the map.
- **The bag's CRAFT button opens the craft window with focus on the selected recipe** (else the first) whenever the plugin
  would place focus for a pad (`GamepadAutoFocusPolicy`).
  - That holds with the window docked or floating.
- Opened with the mouse and no pad connected, the floating window holds focus itself.
  - Then **any direction starts on the selected recipe**, so the keyboard's arrows walk the list too
    (`USnapInventoryCraftPanelWidget::NativeOnNavigation`).
- From the list, the edges lead where they are drawn, as above.

**Putting the bag's craft window away:**

- **B (or Escape) in the bag's craft window puts it away and gives focus back to CRAFT.**
  - That holds floating, or docked with windows fixed, where it is what CRAFT pressed again does, and the next B closes
    the bag.
- With focus moved out of the window (LB / RB to the bag), B closes what is on top first — a pouch window over the bag
  before a docked craft window — and focus stays where it is.

#### Selling

**Selling on a pad.**

- The item menu (Y on a focused bag square) works on a vendor's bag grid too.
- Its **Sell** row opens the sell window.
- Opened from a pad, focus starts on SELL.
- Up reaches the quantity's - and +; Down reaches CANCEL.
  - The window keeps every other key, so the sale stays modal.
- The focused one draws the focus outline. A presses it, and B cancels.

#### Amount windows

**Amount windows on a pad.** Every amount picker takes these keys (`SnapAmountKeys`):

- The pickers are the amount window (a split or a currency), the sell window, a shop's and a forge's quantity, and the
  trade screen's coins.
- On a pad, a row of their pictures under the slider says so.

| Key | Amount |
|---|---|
| LB / RB | -10 / +10 (held: they keep stepping) |
| Y | The most this picker allows (its own ceiling, as the slider's end) |
| X | The least |
| Page Down / Page Up | -10 / +10 |
| Home / End | The least / the most |
| Shift + mouse wheel | -10 / +10 (the wheel alone moves 1) |

- The amount and sell windows answer them wherever focus is inside the window.
- A shop's or forge's quantity and the trade coins answer them only while their - / + (or slider) has focus.
  - So LB / RB still step the bag screen's sections everywhere else.

### A screen with more than one inventory grid

**Every inventory grid the screen shows is a pad target**, and the grids are linked by where each is drawn.

- Each grid is walked as its own rectangle, at its own drawn width.
- At a grid's edge, focus moves to the nearest grid on that side, onto the square in line with the one it left.
  - With nothing drawn between them, Down from the bag's last row reaches a grid drawn under it, and Up comes back to the
    square above.
- On the bag screen, the paper doll, the hotbar bars and the rows of buttons and currencies are linked the same way.
  - So on the example Character screen the filter row and the CRAFT row sit between the bag and the Spells grid (see
    [The bag screen on a pad](#the-bag-screen-on-a-pad)).
- A side with nothing drawn on it keeps the bag's own link: an open pouch or loot window to the right, the hotbar below.

**Focus starts on the player's bag**: The first grid the player can see, in widget-tree order, that is bound to a
component whose `bIsMainInventory` is set.

- If no grid the player can see is bound to one, focus starts on the first grid in tree order that the player can see.
- That grid is also where the equipment column, the hotbar and a pouch hand back to.
- All the grids are one panel, `ESnapFocusPanel::Bag`, so `GetFocusedPanel` answers `Bag` on any of them.

**Position, not tree order, links the grids.** The shipped screens disagree about where a second grid goes:

- The Character screen draws Spells under the bag.
- `WBP_ExampleInventoryScreen` draws its abilities grid to the bag's right.
- `WBP_ExampleSpatialScreen` draws it to the bag's left.

Each is reached by the direction it is drawn in, and a grid on a side takes that edge before the panel registered there.

- On the shipped `WBP_ExampleSpatialScreen` — abilities grid first, then the spatial bag — the main-bag rule is what starts
  the pad on the spatial bag rather than on the abilities grid, which is one Left away.
- **A bag area that follows the layout is one grid to the pad.** The Character screen holds a squares panel and a grid
  panel for the bag.
  - The panel that is hidden for the other layout is skipped.
  - So focus lands on whichever one the bag's `LayoutMode` shows.
- **A grid joins once it has been drawn.**
  - The links come from where each grid is on screen.
  - So a grid that has not been painted yet sits that rebuild out, and the rebuild asks for another pass by itself.
  - A grid that appears is linked within a frame or two.
  - Dragging a window rebuilds the links from where it was dropped.
  - The Verbose log line `focus chain over N grid(s): ...` names every grid and what each of its four sides leads to.
- **The pad's verbs on the other grids.**
  - Carry and put down work from every grid. A scroll carried from the Spells grid onto a hotbar square is bound there.
  - **What a put-down would do is drawn on the target** (`USnapDropOutlineWidget`):
    - The symbol a mouse drag's picture shows (Swap, Mix, Merge, Place, Equip, Store, the red X), at its top-right
      corner, over the green / red outline.
    - Over a grid's plan, the symbol alone, its wash being the frame.
    - The square the item came from stays framed and tinted in the theme's `SelectionGlow` until it is put down or
      canceled.
  - On the bag screen's hotbar, A picks a square, and A on another of that bar swaps the two
    ([the hotbar](hotbar.md#moving-a-square-on-the-bag-screen)).
  - Quick-move, the item menu and split look for the focused square in every grid of the bag screen. So there they act on
    whichever grid holds focus.
- ⚠ **Only the screen's own widget tree is walked.** A grid inside another widget's own tree — the example equipment
  panel's Shards row — is not reached.
- **Among several grids bound to the main bag, tree order decides where focus starts.**
  - A grid the player cannot see is passed over: its own visibility, or that of any container around it, is
    `Collapsed` or `Hidden`.
  - So a classic grid a screen keeps collapsed beside a spatial one never takes focus.
  - A grid inside a collapsed container is passed over the same way, whatever it is bound to (the example demo's bank
    column while the bank is shut).
- **An equipment panel the player cannot see is passed over the same way.**
  - The pad's equipment column is the first equipment panel in tree order that is not `Collapsed` or `Hidden`, itself or
    in any container around it.
  - The bars such a panel draws are never the screen's bars.
- **A window the player closed is passed over.**
  - With window drag and close on, a grid or equipment panel whose window is closed (or inside a closed window) is never
    where the pad starts or goes.
  - The next grid in tree order is where focus starts instead.
  - The rules between panels stop pointing into it.
- **The choice is made each time navigation is rebuilt**, so a grid whose component binds late is picked up by the next
  rebuild.
  - Opening a surface, closing or reopening a window, and any container change request one.

### A spatial bag is walked as it is drawn

The pad walks the bag's grid as the rectangle on screen: **a row ends at the grid's last drawn column, and Down moves one
drawn row.**

- What Right does past the end of a row is `FocusEdgeRule`'s, as on any grid.
- For a spatial bag, the width is the container's `GridWidth`, not the panel's `GridColumns`.
  - A spatial panel is drawn at the container's width and ignores `GridColumns` (see
    [inventory and containers](inventory-and-containers.md)).
- It does not depend on `bDrawSpatialItemsPerCell`. Turning per-cell drawing off changes how an item is drawn, not how the
  pad walks the grid.
- A panel that lays its cells out as a rectangle of some other width overrides `USnapInventoryPanel::GetLayoutColumns`. The
  flowing grid's own sizing keeps reading `GridColumns`.

### A packed pouch is walked as it is drawn

Focus navigation over a pouch window follows the same rule: **a row ends at the pouch's last drawn column, and Down moves
one drawn row.**

- **A *packed* pouch** is drawn at its grid width, so it is walked at that width whatever the window's `SlotsPerRow` says.
  - Packed means its storage fragment sets `StorageWidth`, and the container holding it is laid out `Spatial`.
- **A *flowing* pouch** (one held by a `Slots` container) is drawn at its own `StorageWidth` when it sets one (whole rows
  in both layouts), and walked at it.
- **A pouch that sets no `StorageWidth`** is drawn `SlotsPerRow` across and walked `SlotsPerRow` across.
- The window's `SlotsPerRow` sets the row length of a flowing pouch that authors no `StorageWidth`; one that authors it is
  drawn and walked at it.
- The number the navigation is built from is `USnapInventoryStorageWidget::GetFocusColumns`.

## See also

- [Appendix: settings](appendix-settings.md) — the **Input** categories
- [Hotbar](hotbar.md) — the bar keys in detail
