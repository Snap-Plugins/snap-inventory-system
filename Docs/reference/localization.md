# Localizing and rewording

Every string the plugin shows a player or a designer is gatherable text: a `NSLOCTEXT` or `LOCTEXT` in C++, in the
single localization namespace `SnapInventorySystem`. This page covers gathering that text, translating it, rewording the
English without C++, finding text that is not translatable, and the few places where the plugin shows an identifier
instead of a sentence.

- **What that covers:** Refusal sentences, button labels, readouts, import messages and the editor's asset validation
  messages (the art pack's validation messages use their own namespace, `SnapArtPack`).
- **The demo's own namespaces:** The demo's example code keeps eight namespaces of its own:
  - `SnapInventoryDemoFeatures` (the FEATURES pads' signs and lines).
  - `SnapDemoTrainingDummy` (the training dummy).
  - `SnapInventoryDemo` (three demo lines).
  - `SnapDemoSpellbook` (the spellbook and its Learn Spell action).
  - `SnapDemoSpellCaster` and `SnapDemoSpellCasting` (casting the demo's spells).
  - `SnapDemoGameState` (the demo route's messages).
  - `SnapInventorySystemDemo` (the demo's quest giver).
- **Who collects them:** Your project's own localization target collects them.
  - The plugin ships no localization target of its own and no string table.

**On this page:**

- [Gather it into your project's target](#gather-it-into-your-projects-target)
- [Reword the English without changing C++](#reword-the-english-without-changing-c)
- [Other ways to own the wording](#other-ways-to-own-the-wording)
- [Find text that cannot be translated](#find-text-that-cannot-be-translated)
- [Which counts are digit-grouped](#which-counts-are-digit-grouped)
- [Identifier fallbacks](#identifier-fallbacks)
- [Text in your config](#text-in-your-config)
- [Editor text](#editor-text)
- [The example content](#the-example-content)
- [Tested](#tested)

## Gather it into your project's target

Add the plugin's source to the **Game** target in the Localization Dashboard (or in
`Config/Localization/Game_Gather.ini`):

- **Gather from text in source files.** A project that already searches `Plugins` (the Lyra layout does:
  `SearchDirectoryPaths=Plugins`) needs nothing more.
  - Otherwise add `Plugins/SnapInventorySystem/Source` as a search directory, with the file filters `*.h` and `*.cpp`.
- **Gather from `.ini` files** too if your currency names, filter labels or craft category labels live in config (see
  [Text in your config](#text-in-your-config)).
  - Add the `*.ini` filter and your project's `Config` directory.

Then gather, translate and compile as you would for your own text.

- Nothing about the plugin's text needs special handling.
- Every key of the plugin's own code lives in the one namespace.

### Why no plugin localization target

A plugin target would produce a second set of archives that you then translate and ship alongside your own, and its
translations would load separately from yours.

- Gathering into the project's own target keeps one set of archives, one translation workflow and one compiled result.

### Why no string table

Refusal sentences stay in C++ (`SnapRefusal.cpp`) as `NSLOCTEXT`. A string table would only move the English into an
asset:

- It would not make rewording easier than the native-culture route below.
- It would add an asset every project must ship and keep in step with the code.

## Reword the English without changing C++

Translate English into English. When your project's native culture is `en`, the translations you author for `en` are
loaded as the display text. So an `en` translation of a plugin key replaces the plugin's wording everywhere that key is
shown:

1. Gather (above). The plugin's keys appear under the `SnapInventorySystem` namespace.
2. In the Localization Dashboard, open the translation editor for your native culture (or export it to a `.po` file).
3. Enter your wording for the keys you want to change, for example `RefInventoryFull`.
4. Compile text. The compiled native-culture translations are what your builds display.

- A key you leave untranslated shows the plugin's own English.
- ⚠ When a plugin update changes the English of a key you reworded, the dashboard lists that translation as needing
  review.
- ⚠ Keep what each `{0}` stands for. The craft queue tile's and the craft details' `TimeRemainingFormat` ("{0}
  remaining") take a duration that is already formatted ("1m 30s").
  - So write no unit after `{0}`: "{0}s remaining" shows "1m 30ss remaining".

## Other ways to own the wording

- **Per screen, on the widget.** Many labels and formats are `FText` class defaults on the widget classes.
  - For example `DetailOutputCountFormat` and `DismantleButtonCountFormat` on the crafting panel, `BuyStackFormat` on
    the shop's buy panel.
  - A Widget Blueprint subclass can change one of them for one screen.
  - That text lives in your asset, so your asset gather collects it.
- **Refusals, in your own UI.** Bind `OnActionRefused` and show your own sentence for the refusal codes you care about.
  See [Refusals](refusals.md).

## Find text that cannot be translated

Run the editor or a Development build with `-culture=keys`.

- Every gatherable text then displays as its key and namespace, for example `RefTitleInventory, SnapInventorySystem`.
- Anything still shown as words is not gatherable: a player's name, a number, an identifier fallback from the list
  below, or text in your own assets and config that was authored without a localization key.
- The `keys` culture is compiled into Debug, Development and Test builds, not Shipping.

## Which counts are digit-grouped

A number drawn as a badge — in its own small text: a pill, a chip, a plate or an indicator — is never grouped.

- 1234 reads `1234`, not `1,234`, whatever the culture.
- A badge sized for a few digits has no room for a separator.

The badges:

- **Stack counts:**
  - The count on an equipment slot, a hotbar square, a pouch square, a loot square, a trade offer square and the drag
    ghost.
  - On a HUD readout mirrored through `EquipmentHUDBindings`.
  - The collect feed's `x1234`.
  - A sale's line chips (`x1234`, in the sell window and in the vendor's buy panel).
  - The count badge on a multi-item drag (`x1234`).
  - A container item's occupancy readout on its slot (`1234/2000`).
  - The example grid and shard slots write their count from their own Blueprint's `RefreshSlot`, whose
    **To Text (Integer)** node has **Use Grouping** off; a slot Blueprint of your own should do the same.
- **Other badges:**
  - A hotbar square's charges (`1234/2000`, or `1234` with no limit) and its cooldown seconds (a 20-minute cooldown
    reads `1200`).
  - The hotbar page indicator.
  - The interaction prompt's option count (`2/3`).
  - The craft queue tile's `x1234`.
  - A recipe row's output badge (`x1234`).
  - Requirement chips and `+1234` overflow chip.
  - The crafting detail pane's output badge and have/need chips (`1234/2000`).
  - A shop row's stock badge (the stock left, or a buyback slot's count).

Other numbers follow the culture's grouping:

- On purpose: a currency amount (a balance, a price, a total) and the split window's `{0} / {1}`.
- A number inside a sentence-width line: the craft queue tile's name line (`{0}  x{1}`, on a themed tile), a pouch
  window's capacity line, the pickup prompt's `Pick Up {0} x{1}`, a tooltip's status lines.

Every bare number goes through `SnapInventoryText::AsCount` (in `UICore/SnapCountText.h`).

- Its shared patterns (`x{0}`, `{0}/{1}`, `{Marker}{Key}`, `{Label}  {Value}`) are themselves gatherable, and so is each
  widget's own pattern around a badge.
  - So a culture that orders them differently can translate the pattern.
- The hotbar's modifier marker, `HotbarSlotModifierMarker`, is your project's own symbol and is never translated; only
  its position relative to the key label is.

## Identifier fallbacks

These appear only when the project supplied no name.

- Each is an identifier rather than a sentence, so a gather does not collect it.
- Supply the name to replace it.

| Where | Shows | Supply |
|---|---|---|
| An equipment slot's name, on the slot's label, the shop's upgrade row and the tooltip's comparison heading | The slot tag's last segment, split at capitals (`MainHand` reads Main Hand) | Author the label in the slot Widget Blueprint and leave `SlotLabelWidgetName` unset |
| A craft category row with no `Label` | The leaf of its first match tag | The row's `Label` |
| A currency with no `DisplayName` | Its counter tag, in the wallet tooltip, the craft cost chip and an import message | The `Currencies` row's `DisplayName` |
| A trade partner with no player name | The owning actor's object name | A player name |
| A queued craft whose output no longer loads | The recipe's identifier | The missing asset |
| An import message about content that no longer exists | The saved asset path, container tag or slot tag | Nothing to supply: the content is gone |

Player names are data and are never translated.

## Text in your config

An `FText` value in an `.ini` file (a `Currencies` row's `DisplayName`, an item filter's label, a craft category's
`Label`) is only gatherable when it is written in the `NSLOCTEXT` form:

```
+Currencies=(CounterTag=(TagName="SnapInventorySystem.Counter.Currency"),DisplayName=NSLOCTEXT("MyGame", "CurrencyCoins", "Coins"))
```

- `DisplayName=INVTEXT("Coins")` and a plain `DisplayName="Coins"` still display, but carry no key.
  - So they show as words under `-culture=keys` and cannot be translated.
- The example project's three currencies are written this way, in the `SnapInventoryDemo` namespace (see
  [the demo map](demo-map.md#gold-silver-and-gems)).

## Editor text

The asset validation messages (a footprint with no cells, a loot table row with no item, and the rest) are compiled only
into the editor.

- A gather collects text inside `#if WITH_EDITOR` code only when its `ShouldGatherFromEditorOnlyData` is true, which a
  project's **Game** target normally leaves false.
- To translate them, gather the plugin's source into a target that gathers editor-only data.

## The example content

The plugin's shipped assets — the example data assets, input actions, widget Blueprints, Blueprints and demo map — are
not part of a project's asset gather unless you add the plugin's content to it.

Every text a player can see in them is keyed, one namespace per asset type, under a key that names the asset and the
field. For example:

- `ID_Sword_Iron.DisplayName`, `ID_Sword_Iron.UIData.FlavorText`
- `DA_ExampleItemSet_Iron.Tier2.Name`
- `IA_Inventory_Toggle.DisplayName`
- `WBP_ExampleShardPanel.Title.Text`
- `BP_Example_Forge.PromptText`

| Example assets | Namespace |
|---|---|
| The item definitions (`ID_*`): display name, description, flavor text and type label | `SnapInventoryItems` |
| `DA_ExampleItemSet_Iron`: the set's name and its tier names | `SnapInventoryItemSets` |
| `DA_ExampleStatDisplay`: the name of every stat and grant row | `SnapInventoryStatDisplay` |
| `DA_ExampleGeneralStore`, `DA_ExampleGemShop` and `DA_ExampleJunkMerchant`: the vendor name | `SnapInventoryVendors` |
| `DA_ExampleRarityTable`: the six tier names | `SnapInventoryRarity` |
| The input actions (`IA_*`): each key-rebinding row's name and category, and `IA_UI_Rotate`'s description; `IMC_SnapDemoConsumables`' description | `SnapInventoryInput` |
| The example widget Blueprints: panel titles, tab and button labels, the empty-slot tooltip title | `SnapInventoryWidgets` |
| `BP_Example_Forge` and `BP_Example_VendorStall`: the interaction prompt | `SnapInventoryInteraction` |
| `L_SnapInventoryDemo`: the area signs, the stall signs, the title and the hint | `SnapInventoryDemo` |
| `L_SnapInventoryDemo`: the feature name and value on each FEATURES pad's sign (written by the pad's code) | `SnapInventoryDemoFeatures` |

A few are keyed in `SnapInventorySystem` instead:

- The slot names in `DA_ExampleEquipmentSlotSet`.
- The display names of `ID_Example_Greatsword` and `ID_Example_Shield`.
- `ID_Example_Totem`'s name.

`DA_ExampleVendorStock` sets an empty vendor name, so its shop shows no heading. The demo map's General Store uses
`DA_ExampleGeneralStore`, a copy that names it.

What stays culture-invariant is deliberate, and it is two kinds of text:

- **Glyphs and placeholders**, where there is nothing to translate: the slot's F, J and L markers, the stack splitter's
  minus and plus signs, and a counter's 0 that the panel replaces with the real number.
- **Designer defaults that C++ replaces with keyed text** every time the widget is shown: `WBP_ExampleLootWindow`'s
  title, Take All label and "Empty" line, and `WBP_ExampleStorageWindow`'s Take All label.
  - To reword one of those, set the property the C++ writes from: `WindowTitle`, `LootAllLabel` and `EmptyLabelText` on
    the loot window, `CollectAllLabel` on the storage window.

⚠ A label typed from Python with `unreal.Text(...)` is culture-invariant.

- Build keyed text instead, for example by importing `NSLOCTEXT("Namespace", "Key", "Source")` into a struct that has a
  text field and reading it back.

## Tested

`SnapInventory.Localization.EveryShownStringIsGatherable` checks that:

- Every refusal reason, refusal title and trade label, the count formats, the key-label and tooltip-line patterns, the
  import report's currency message and one message from each asset validator is gatherable in the `SnapInventorySystem`
  namespace.
- The two bare-digit formats do not group.

The twelve `SnapInventory.StackCounts` tests build each surface that draws a stack count around a stack of 1234, and the
thirteen `SnapInventory.BadgeNumbers` tests each badge around 1234.

- They compare the text exactly (`1234`, `x1234`, `1234/2000`, `+1234`), after checking that the running culture groups
  1234 as `1,234` — so a grouped readout cannot pass.
- The craft queue tile's test also checks that a themed tile's name line still reads `x1,234`.

`SnapInventory.Localization.ShippedAssetTextIsKeyed` loads every shipped asset under the plugin's mount (not
`Content/Tests`).

- It reads every text property of every saved object in each package through reflection (a Blueprint graph pin's literal
  is not a property and is not read).
- It fails on a culture-invariant text that is not on its allow-list — naming the asset, the object and the property.
- It also fails on an allow-list entry that no longer finds its text, so the list cannot go stale.
- The allow-list is exactly the glyphs, placeholders and designer defaults above, each with its reason.
- A widget Blueprint is read through its designer tree; the compiled tree is rebuilt from it on every compile.
- It runs in the editor only, where content is authored, and takes well under a second.

`SnapInventory.Localization.ShippedAssetTextScanFindsAPlant` proves the scan finds a planted text on an instanced
subobject, in an array of structs and in a map of structs, and ignores keyed, empty and transient ones. See
[Testing and diagnostics](testing-and-diagnostics.md).
