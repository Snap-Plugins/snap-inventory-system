# Tooltips and stats

What a tooltip says an item does, and what it would cost you to wear it instead of what you have on.

A tooltip used to name the asset: an item granting `GE_Example_SwordPower` read **"Example Sword Power"**. It now:

- Reads the effect's actual modifiers and prints **"+5  Attack Power"**.
- Says under the item's name what the item is ("Weapon — Either hand").
- Groups what it does under small headers ("Base", "Modifiers", "Passive", "When used").
- For an item in the bag, adds what changes if you equip it.

**On this page:**

- [The three pieces](#the-three-pieces)
- [Why the table is curated, and what happens without a row](#why-the-table-is-curated-and-what-happens-without-a-row)
- [Where a magnitude comes from](#where-a-magnitude-comes-from)
- [Stats an item declares itself](#stats-an-item-declares-itself)
- [Comparing against what is worn](#comparing-against-what-is-worn)
- [Adding lines from a fragment](#adding-lines-from-a-fragment)
- [Requirements](#requirements)
- [How the lines are drawn](#how-the-lines-are-drawn)
- [What the item is: the type line](#what-the-item-is-the-type-line)
- [Shortcuts: Hold Alt](#shortcuts-hold-alt)
- [The top row: weight and status beside the title](#the-top-row-weight-and-status-beside-the-title)
- [With the mouse, the card stands beside the item](#with-the-mouse-the-card-stands-beside-the-item)
- [On a pad, the focused square shows its tooltip](#on-a-pad-the-focused-square-shows-its-tooltip)
- [A running effect's card](#a-running-effects-card)
- [The demo attribute set](#the-demo-attribute-set)
- [When the tooltip has nowhere to put a box](#when-the-tooltip-has-nowhere-to-put-a-box)
- [A definition alone cannot show rarity-scaled stats](#a-definition-alone-cannot-show-rarity-scaled-stats)
- [Without GameplayAbilities](#without-gameplayabilities)
- [See also](#see-also)

## The three pieces

| Piece | What it is |
|---|---|
| `FSnapTooltipLine` | One line: a `Label`, a `Value`, a `Kind`, a sort key and a direction |
| `USnapStatDisplayTable` | The curated map from an attribute (or a whole effect) to the words for it |
| `USnapItemStatReader` | Reads an item's own stat rows and its GameplayEffect modifiers, and produces the lines |

Point `StatDisplayTable` in **Project Settings > Snap Inventory System - Inventory UI > Tooltip** at a table. The plugin ships one,
`DA_ExampleStatDisplay`, wired by default.

## Why the table is curated, and what happens without a row

Nothing derives a display name from an identifier. An attribute's property name and a class name are both symbols a
programmer chose:

- They cannot be translated.
- They leak asset names to players.
- They are wrong the moment somebody renames a variable.

That is exactly what the old behavior did.

So **an attribute with no row prints no line**. That is deliberate, and because "prints nothing" is indistinguishable
from "is broken", a grant that contributes *nothing at all* is named once in the log:

```
Tooltip: 'GE_MyProjectEffect_C' grants nothing this project has named, so it contributes NO tooltip
line. Add a Grants row for it (or a Stats row for each attribute it modifies) to the table named by
StatDisplayTable. This is reported once per class.
```

An effect whose attributes *are* named needs no `Grants` row — it already produces stat lines.

### Two maps, because an ability has no attribute

`Stats` is keyed by `FGameplayAttribute`. `Grants` is keyed by the **effect or ability class** and covers the two cases
an attribute row cannot:

- A granted ability (an entry in `GrantedAbilityClasses`), which modifies no attribute at all and would otherwise
  vanish.
- An effect better described in one sentence than in four numbers — set `bSuppressStatLines` and its numbers are
  replaced by the prose rather than printed alongside it.

A `Grants` row's `Section` says which header its line is drawn under (see
[How the lines are drawn](#how-the-lines-are-drawn)).

- `Auto`, the default, decides by where the grant comes from: **Passive** for an ability or an effect the item applies
  while worn, **When used** for an effect it applies when used.
- Set `Ability` for an ability the player uses (a spell scroll's), so it is not read as something that runs by itself.

## Where a magnitude comes from

The reader mirrors the equip path rather than paraphrasing it — same rank source, same `SetByCaller` rule, same level,
and the same refusals.

- A reader that made its own choices would produce a tooltip that disagrees with the character it describes, and nobody
  would notice until they checked by hand.

| Authored as | Reported as |
|---|---|
| `AddBase` / `AddFinal`, magnitude 5 | `+5` |
| `MultiplyAdditive` / `MultiplyCompound`, magnitude 1.15 | `+15%` |
| `DivideAdditive`, `Override` | **Omitted** |
| `AttributeBased`, `CustomCalculationClass` | **Omitted** |

The op decides the arithmetic; the table decides only the wording and the precision.

- A display row may override the unit with `ESnapStatUnit`, but its default `FromModifierOp` leaves the op in charge —
  because two owners for one number is how the two come to disagree.

The omissions are refusals, not gaps:

- `DivideAdditive` summed-then-divided is not a percentage.
- An `Override` printed through a signed formatter reads as a bonus when it is a replacement.
- An attribute-based magnitude cannot be evaluated at all without a live spec.
- A wrong number is read as information; a missing line is not.

### Rarity-scaled effects

`FSnapItemScaledEffect` magnitudes are `BaseMagnitude + MagnitudePerRarityRank * rank`.

- The rank comes from the **instance's** own rarity.
- Or — when there is no instance, as on a crafting or vendor tooltip — from the definition's `FixedRarity`, which pins
  the tier for every instance it will ever mint.

When the slope is non-zero and no rank can be resolved, **the whole effect is skipped**, exactly as equipping skips it.
Substituting the flat part alone would advertise a bonus that equipping does not grant.

## Stats an item declares itself

An item can also say what it adds as plain data, with a `USnapItemFragment_StatContributions` fragment: rows of a
stat's bare name, a unit (`Flat` or `Percent`), an `Amount` and an optional `AmountPerRarityRank`.

- The reader adds those rows to the same totals as an effect's modifiers.
- So they print as ordinary stat lines, worded by the same display table, and appear in the compare block.

| Row | Line |
|---|---|
| `Armor`, `Flat`, 5 | `Armor  +5` |
| `MaxHealth`, `Percent`, 15 | `Max Health  +15%` |

- A `Percent` row's `Amount` is already in percent: write 15, not 1.15.
- Two rows of one stat and unit add together, and a row adds to an effect that moves the same stat.
- A row with `AmountPerRarityRank` follows the rule above: no resolvable rank, no line.

These rows are not a description of the item beside what it does — they **are** what it does.

- Equipping counts them toward `GetEquippedStatTotal`, and with `GameplayAbilities` applies them through the Stat Effect
  Map.
- All of it goes through the same function the tooltip uses. See
  [Equipment](equipment.md#stats-an-item-declares-itself).

⚠ A stat with no row in the display table draws no line, as for an effect.

- The row still counts toward the total.
- Saving the item **warns** about it: the fragment's validation names each such stat once, and the table it asked.
- With no table set it says nothing, because that is how stat lines are turned off.
- A table that does not load is reported by the tooltip's own lookup instead.

## Comparing against what is worn

With `bCompareToEquipped` on (the default), hovering an equippable item that is **not** worn appends **one** block
comparing it with the item worn in the slot an equip would actually use:

```
Compared with: Iron Ring (Left Ring)
 +5%  Max Health
 -2   Armor
```

The slot is the equip door's own choice, `USnapEquipmentComponent::FindQuickEquipSlotForItem` — the first free slot
the item fits, else the first it fits; the slot a right-click quick-equip would use. So:

- **An item that is already worn shows no block.** It replaces nothing.
  - It used to be compared with every *other* occupied slot it fitted, which put "Replaces: Ring 2" on a ring worn in
    Ring 1.
- **A free slot means no block.** A second ring with a ring slot empty, or a second sword with the off hand free,
  replaces nothing, so there is nothing to compare.
- **Two rings worn: one block**, naming the ring a quick-equip would replace — the first ring slot in the slot set's
  order.
  - Drag the ring onto the other slot to put it there instead.
- **Nothing differs: no block and no heading** — a heading with nothing under it reads as a lookup that failed.

Three details worth knowing:

- Deltas are **equip-time only**. A potion's consume effect is never applied by wearing something, so folding it into a
  swap comparison would promise a change that equipping cannot produce.
- **A difference that prints as zero draws no line** — no `+0` — judged at the precision the line would print with, so
  `0.3` at a Precision of 0 is dropped too.
  - A swap whose every difference prints as zero gets no block and no heading.
  - The item's own stat and affix lines follow the same rule: a value that prints as zero is not drawn.
- Color follows **direction, not sign**. Set `bLowerIsBetter` on a row for a cooldown or a recoil and a drop in that
  stat is green while printing a negative number.

Everything here reads class defaults and definition data. Nothing reads a live `AbilitySystemComponent`, so a tooltip is
correct on a client, correct for an item still in a shop, and correct for one lying on the ground.

## Adding lines from a fragment

```cpp
virtual TArray<FSnapTooltipLine> GetTooltipLines_Implementation(
        const USnapItemInstance* Instance) const override;
```

Called on every fragment of the definition; the whole set is sorted once afterwards, so a fragment controls placement
through `Kind` and `SortOrder` rather than through its position in the array.

It **returns** its lines rather than adding to a shared array, and that is deliberate.

- The obvious signature — `void AppendTooltipLines(Instance, TArray&)` — was written first and measured destroying every
  line gathered before it.
- A non-const `TArray&` on a `UFUNCTION` is an *out* parameter, so an override never receives the accumulated array: it
  produces a fresh one that is copied back over the caller's.
- Returning a value makes a fragment structurally incapable of clobbering another's lines.

`Instance` may be null — a crafting tooltip describes a definition that has not been minted yet.

This is the one hook on `USnapItemFragment` that runs on **whichever machine is drawing**, including a client, and
it must not mutate anything.

- The base implementation is a genuine no-op, which is what makes a Blueprint override free.
- A Blueprint override replaces the native body and has no parent-call node to recover it.

## Requirements

An item with a `USnapItemFragment_Requirements` lists each term below its own lines and above any compare block,
measured against the player looking at it:

| Term | Left | Right |
|---|---|---|
| A required tag | `Requires` | The tag's last segment, in words |
| A forbidden tag | `Not while` | The same |
| An attribute minimum | The attribute's stat-table name, else its name in words | `12 (you have 10)` |
| `MinLevel` | `Level` | `10 (you have 4)` |

An unmet term is drawn in `StatLossColor`, a met one in `StatNeutralColor`. The lines come from
`USnapItemStatReader::BuildRequirementLines`, which a Blueprint tooltip can call for itself.

- **Every term is drawn, even an attribute the stat table does not name.** That deliberately departs from the rule the
  rest of the tooltip follows: a hidden requirement would hide exactly why an item cannot be equipped.
  - The fallback name comes from the attribute's identifier, so it is **not translatable** — give the attribute a table
    row to get a localized name.
- **The viewer** is the owner of the equipment component the tooltip resolves, else the owning player's PlayerState,
  else their pawn — the owner the equip doors ask.
  - A definition tooltip (a vendor row, a craft preview) shows the block too.
  - A tooltip with no owning player draws every term neutral, with the required value alone.
- **A worn item's terms are neutral.** A worn item whose requirement lapses keeps working, so a line in the loss color
  on it would say something false.
  - The terms still show what it asks of whoever equips it next.
- **Numbers truncate**, so a value just short of a minimum never prints as reaching it.
- **A grid or pouch slot re-seeds its tooltip when the cursor arrives.** The viewer's values change without the slot's
  item changing — put a sword on and a ring's "(you have 10)" would otherwise be stale.
- **Requirement lines wrap** at the tooltip's wrap width, like every other line.
- The no-box fallback leaves the block out and logs once: an unmet term that cannot be colored reads as a plain fact
  about the item.

The context menu and a refused drop say the same thing in other places — see [Equipment](equipment.md).

## How the lines are drawn

The tooltip builds a `UVerticalBox` into its **own** widget tree, beside the widget named by `GrantsTextWidgetName`, and
collapses that text block while the box is drawing.

- No tooltip asset needs re-authoring.
- Each line gets its own color — which a single text block could not do.

The lines are drawn in **sections**, each under its own heading, in this order:

| Section | What is in it |
|---|---|
| Base | What the item ITSELF adds while worn: its equip-time effects' stats and its own stat rows |
| Modifiers | Its rolled **magic** modifiers — its affixes ([Affixes](affixes.md)), in the magic color |
| Passive | A named thing it does while worn: a granted ability, or an equip-time effect a `Grants` row describes in prose |
| When used | What using it does: its consume effects, as numbers or prose |
| Ability | An ability the player uses, for a `Grants` row whose `Section` says so |
| Set bonus | Its set's name with how many of its pieces you wear, then each tier — in the gain color while that tier is active |
| (No header) | Anything else, such as a fragment's own lines |
| Requirements | Who may equip it — see [Requirements](#requirements) |

- The compare block comes last, under its own "Compared with" heading.
- Inside a section the lines keep the reader's order — `(Kind, SortOrder, Label)` — so grouping changes nothing about
  how two lines of one section are ordered.

**Each section reads as its own block.**

- A thin rule sits in the space above it (`StatHeadingTopPadding`, 8 units, the rule `SectionDividerThickness` thick in
  its middle).
  - The rule is a `USnapFadeBar` whose two ends ease out to nothing over a quarter of the card's width each, so the line
    dissolves instead of stopping hard.
- Its heading is bold, a size under the body and letter-spaced (`StatHeadingLetterSpacing`), in **its own accent**.
- Its lines sit `StatLineIndent` (8) in from the heading, the number column kept.
- The same rule separates the title block from what follows it and sits above the status line and the flavor text.
- A rule is shown only where something visible sits above and below it, so a one-section item with no flavor text has
  none.

The accents are the **theme's** — the palette's Tooltip roles, so a picked theme reaches every one:

| Heading | Palette role | Plugin default |
|---|---|---|
| Base | `TooltipBaseHeading` | White |
| Modifiers | `TooltipModifiersHeading` | Gold |
| Passive | `TooltipPassiveHeading` | Teal |
| When used | `TooltipOnUseHeading` | Green |
| Ability | `TooltipAbilityHeading` | Blue |
| Set bonus | `TooltipSetBonusHeading` | Amber |
| Requirements | `TooltipRequirementsHeading`, or `TooltipRequirementsUnmetHeading` while any term is unmet | Muted gray / red |
| Compared with | `TooltipCompareHeading` | Neutral gray |

- The rule is the theme's `FrameOutline`, drawn at 0.35 and 1 unit thick by default.
- Every default clears 5 : 1 against the shipped tooltip plate under all three shipped themes.
- `USnapInventoryTooltipWidget::GetSectionHeadingColor` answers which color a section takes.
- `RefreshSectionDividers` (BlueprintCallable) re-decides the rules — call it after showing or hiding a block of your
  own from `OnTooltipRefreshed`.

Each `FSnapTooltipLine` carries a `Section`. `Auto`, the default, lets the line's `Kind` decide:

- A `Stat` line goes under Base — or under Modifiers when it is a rolled affix (`bMagicModifier`), so "Modifiers" only
  ever means magic, on every item.
- A `Requirement` goes under Requirements.
- A `Grant` with no stated section goes under no header — a fragment's own line is not a passive.

The reader states the section where only it knows the source: a consume effect's lines are `OnUse`, a worn item's named
grants `Passive`. A stat both a worn effect and a consume effect move is drawn once in each section rather than summed
into one line nobody could place.

A **modifier line is two columns, number first** — `+5` then `Max Health`.

- The number is right-aligned in a column as wide as the card's **widest** number (at least `StatValueColumnWidth`), so
  every name on the card starts at one place, across sections too.
  - This is `bAlignStatLabels`, on; off, each name follows its own number.
- **The number stands apart from its words:** Bold, in the theme's `TooltipStatValue` (pale gold).
  - Or, on a line that is better or worse than something (a comparison, a consumable's heal or damage), in
    `StatGainColor` / `StatLossColor`.
  - The words are drawn in `StatNeutralColor`.
- A number is never colored by its sign: a cooldown that went down is a gain.
- A **magic modifier** — a rolled affix ([Affixes](affixes.md)) — draws its number AND its words in the theme's
  `TooltipMagicModifier` (pale blue).
  - So a modifier's "+2 Attack Power" reads apart from the item's own "+5 Attack Power" under it.
  - The reader marks such a line (`FSnapTooltipLine::bMagicModifier`), and on a better / worse line its number keeps the
    gain / loss color.
- Every other line (a passive in prose, a requirement, a set tier) is one text.

**Every row is drawn in the tooltip's own body font** — the grants text's, else the status line's, else the flavor
text's, in its regular face — and a heading in its bold face, one size smaller and letter-spaced.

- Rows were built with no font at all, which is the engine's text block default, Roboto Bold 24: larger than the title.

If the box cannot be built — no grants widget, or one sitting in a container that accepts a single child — the lines
fall back into the text block, joined, losing the color and the headers.

- The compare block is not shown on that path: a delta block whose gains and losses are the same color is not a degraded
  compare block, it is a misleading one.

Colors, gaps and the number column are `EditAnywhere` on `USnapInventoryTooltipWidget`: `StatGainColor`, `StatLossColor`,
`StatNeutralColor`, `StatHeadingTopPadding`, `StatHeadingBottomPadding`, `SectionDividerThickness`,
`StatHeadingLetterSpacing`, `StatLineIndent`, `StatValueColumnWidth`, `StatValueGap`, `TypeLabelColor`.

- The headings' colors are not among them: they are the theme's, above.

### The size of the tooltip

The tooltip **grows with its text**.

- A short item is a compact card no smaller than `TooltipMinWidth` (220) by `TooltipMinHeight` (48).
- A longer one widens up to `TooltipMaxWidth` (340), and past that its text wraps and the card grows downward.
- An **empty square's** tooltip is sized to its title alone, with no minimum, and a slot whose empty title is blank
  shows no tooltip at all.
- The three are on the tooltip class (`EditDefaultsOnly`) and bound the `USizeBox` named by `WidthBoxWidgetName` — the
  example's `TipSizer`.

Every text block wraps at an **explicit** width — `TooltipMaxWidth`, less the block's own slot padding — never at
"auto".

- Auto-wrap wraps at the width a block was last *painted* at, and one tooltip is reused for every item a slot shows.
- After a short item, a long one would wrap at the short item's width and the card could never grow back.
- Keep padding **outside** the size box (the example's border carries it), or text can run into it.

Set `TooltipWidth` above 0 for the earlier behavior: one fixed width, only the height varying, and the authored text
blocks left exactly as authored.

- ⚠ Its default changed from 260 to 0 — a tooltip Blueprint that stored its own value keeps it and stays fixed.

## What the item is: the type line

Under its name the tooltip says what the item is, followed by the player's marks on it:

```
Iron Sword
Weapon — Either hand
```

```
Rotten Mush
Consumable  ·  Junk
```

The words come from `USnapItemStatReader::DescribeItemType` (BlueprintPure). The first rule that applies wins:

1. The item's own `USnapItemFragment_UIData::TypeLabel`, when set — the override, for a word of your own.
2. Its fragments:
   - Counted rather than carried (`ContainerRouting` in Counter mode) → **Currency**.
   - It holds items (`ItemStorage`) → **Container**.
   - `CapacityUpgrade` → **Bag upgrade**.
   - `RecipeUnlock` → **Recipe**.
3. The slot it is worn in:
   - A consumable slot → **Consumable**; a ring slot → **Ring**; the amulet slot → **Amulet**.
   - A weapon (`Item.Type.Weapon`, or worn in the main hand) → **Weapon — Two hands** when it blocks the off hand,
     **Weapon — Either hand** when the equip door fits it into both hands, else **Weapon — <slot>**.
   - Anything else held in the off hand → **Shield**.
   - Armor (`Item.Type.Armor`, or the head, chest, gloves, greaves or boots slot) → **Armor — <slot>**.
   - Any other slot → **Equipment — <slot>**.
4. The plugin's nine `Item.Type` tags: Currency, Consumable, Material (**Crafting material**), Container, Tool, Quest
   (**Quest item**), Weapon, Armor, Accessory.

Slot names are the slot set's own ("Left Ring", "Chest").

- Rule 4 reads the plugin's own tag names: an item classified only under a namespace of your own gets the
  fragment-derived word, or none — set `TypeLabel` where you want one.
- The example spell scrolls set it to "Spell", and the Carved Totem to "Junk" — a word on its type line, not a mark.
  - SELL JUNK still sells only what the player marked.
  - A mark the line already says is not said twice (a marked totem reads "Junk", not "Junk  ·  Junk").

The marks — **Junk**, **Locked**, **Favorite** — belong to an instance, so a craft or vendor preview shows the type
alone.

- The line is written to the text block named by `TypeTextWidgetName` ("TypeText").
- A tooltip without one gets one built under its title, in the body font's italic face and `TypeLabelColor` — quieter
  than the section headings.
- `None` opts out.
- `BuildTypeLabel` and `BuildTypeLabelForDefinition` are BlueprintPure, for a tooltip of your own.

## Shortcuts: Hold Alt

An item tooltip ends with a dim line, **Hold Alt for shortcuts**. While Alt is held it becomes the list of what each
gesture does with that item **on that square**, one per line:

```
Double-click  Consume
Shift+Drag  Take half
Ctrl+Drag  Take one
Drag onto Arcane Shard  Make Greater Health Potion
```

- **The right-click menu is never listed.** It is how every item is opened, not a shortcut.
  - An item whose only way in is the menu (a gold bar, a letter) shows **no** "Hold Alt" line at all, so the hint never
    promises an empty list.
- **A bag panel's currencies say it too.** The tooltip of each wallet entry — its number and its coin — ends with the
  same dim "Hold Alt for shortcuts" line.
  - While Alt is held it lists the ways to drop that currency from the live table: `Click  Drop all`,
    `Shift+Click  Drop half`, `Ctrl+Click  Drop an amount you choose` (not while the panel's `bCounterClickDropsToWorld`
    is off).
  - It is `USnapInventoryThemedTooltipWidget::SetShortcutLines`, which any themed tooltip can use.
- **It is the square's own answer, asked at that moment.** A sword's line reads `Shift+Click  Equip` — and, with a pouch
  window open, `Shift+Click  Store in Leather Pouch`, because that is what the click would do then.
  - A square answers through the same functions its gestures call, check-only, so the list and the click cannot
    disagree.
  - An action is named the way the right-click menu names it (`Consume`, `Equip`, `Unequip`, `Unpack`, `Move to Bank`, a
    project row such as `Learn`).
  - Something a gesture would refuse — a pouch that cannot take the item, a full bag — is not listed.
- **The chords come from the live gesture table** ([Input](input.md#ui-gestures)), so a rebind shows the new chord.
  - The hint names the `ShowShortcuts` key ("Hold Shift for shortcuts" once rebound to Shift).
- **With a gamepad the last input** (or a project forcing the pad with `GlyphDevicePreference`), the line shows the pad
  buttons for that square straight away — no hold, there is no Alt on a pad.
  - `A  Pick up` (the gamepad carry).
  - On a bag square, the plugin's pad actions — quick move, split, menu (`QuickMoveInputAction`, `SplitInputAction`,
    `ContextMenuInputAction`) — read from the player's live mappings, so an action whose context is not applied is not
    listed (split ships unmapped, so it is listed only where a project maps it).
  - Each button is drawn as a PICTURE of the player's pad (`GamepadGlyphs`, in the family `GamepadGlyphFamily` picks), a
    row per line — the picture, then what it does.
  - A button with no picture is named, through the same glyph hook as every other key the plugin draws
    (`FSnapInventorySystemModule::OnResolveInputGlyph`; a key it names is drawn as its text, never a picture).
  - A pad player sees it beside the square that has focus — see
    [On a pad, the focused square shows its tooltip](#on-a-pad-the-focused-square-shows-its-tooltip).
- **Which squares list what:**
  - The bag square (double-click, Shift+Click, a plain click choosing an item to sell while a shop is open, the split
    and mix drags, a repair kit's drag).
  - The worn slot (Shift+Click to unequip or store).
  - The pouch square (double-click, Shift+Click to unpack or offer, the drags).
  - The hotbar square (its own key and a click use it, a right-click takes it off the bar).
  - The loot square (Shift+Click takes it).
  - Your own offer square on the trade screen (Shift+Click takes it back; a partner's square acts on nothing and lists
    nothing).
  - While a trade is open a bag square's Shift+Click reads `Offer`.
  - A craft chip or a shop row describes an item nobody holds, so its tooltip has no line.
- **Your own square** implements `ISnapItemShortcutSource` (C++ or Blueprint):
  `GatherItemShortcuts(bGamepad, OutShortcuts)` returns its `FSnapItemShortcut` lines (set a pad line's `Key` and the
  tooltip draws that button's picture).
  - The tooltip finds it with `SetShortcutSource`, or as the widget that created the tooltip — which is how every plugin
    square does it (`CreateWidget(this, ...)`).
  - `BuildShortcutListText(bGamepad)` and `IsShowingShortcutList` are on the tooltip for a Blueprint of your own.
- **The line** is the text block named by `ShortcutTextWidgetName` ("ShortcutText").
  - A tooltip without one gets one built as the last child of its body panel — a size under the body text, italic, in
    the palette's `TooltipShortcutHint` (dull on purpose).
  - It is not a section: no rule is drawn above it.
  - `None` opts out, and `USnapInventoryUISettings::bShowTooltipShortcuts` (on) turns it off everywhere.
  - The pad list's picture rows go in the VerticalBox named by `ShortcutRowsWidgetName` ("ShortcutRows"), else in one
    built right under the line when the line sits in a VerticalBox (the built line always does); anywhere else the pad
    list stays text.
  - `WBP_ExampleItemTooltip` carries both parts, after `FlavorText`, exactly as the plugin would build them — move or
    restyle them there.
- Nothing is asked when a square re-seeds its tooltip (every square does on every inventory change): the list is built
  only while a tooltip is on screen, when the key goes down or a pad becomes the last input.

## The top row: weight and status beside the title

The item's **weight** sits at the top right of the card, on the title's row, and its **status line** (durability,
freshness) right-aligned directly under it, beside the type line — they have no rows of their own.

- The title and the type line wrap short of them (the right column is measured with its own fonts each time the card is
  seeded).
- The tooltip class's `bPlaceWeightAndStatusBesideTitle` (on) turns it off — the weight and status rows then stay where
  the tooltip's Blueprint put them.
- `HeaderColumnGap` (12) is the space between the two columns.
- It needs the title inside a VerticalBox and the status text named by `StatusTextWidgetName`; a tooltip without either
  keeps its authored layout.

## With the mouse, the card stands beside the item

A tooltip shown for the pointer does not follow it: the card sits **beside the item** and stays there while the pointer
moves within the item.

- It sits right of the item, left when the right does not fit, level with its top and kept inside the screen.
- It moves only when the pointer reaches another item.
- In a grid it sits beside the whole item, not the one cell under the pointer.
- It is the gamepad's placement (below), so the card is in the same place whichever device the player uses.

How it works:

- The card places itself: each tick it moves its own root by the difference between where Slate put it (at the pointer)
  and the place beside the square that owns it.
  - The move lands in the same frame, so it never trails the pointer (`USnapInventoryTooltipWidget::UpdateMousePlacement`;
    the item's rectangle is `USnapFocusedTooltipWidget::GetItemScreenRect`).
  - Any tooltip of this class that a widget created for itself is placed beside that widget — the bag, worn, hotbar,
    pouch, loot, trade, shop and craft squares alike.
- The place is rounded to whole PIXELS (`USnapInventoryTooltipWidget::ComputeBesideItemTranslation`), so at any display
  scale the card lands on the same pixel wherever the pointer is inside the item.
- The small tooltips stand beside their controls the same way: a craft queue tile, a filter tab (the bag's, a shop's, the
  craft panel's), a currency, a sort button, the craft panel's Clear button and headers. `InventoryThemedTooltip::Apply`
  records the control, and the tooltip (`USnapInventoryThemedTooltipWidget`) moves its root each tick by the same rule as the
  card (`USnapInventoryTooltipWidget::ComputeBesidePlacement`).
- A card one widget makes for another stands beside the widget that holds it: the sell window's icon, name and rows
  (`USnapInventoryTooltipWidget::SetPlacementSubject`).
- `USnapInventoryUISettings::bPlaceMouseTooltipBesideItem` (on) turns it off: every tooltip then follows the pointer,
  as Slate draws a tooltip by default.

## On a pad, the focused square shows its tooltip

Slate shows a tooltip only under the mouse pointer, so a player moving focus over the bag with a gamepad would never see
one — nor the pad buttons its shortcut line names.

With a gamepad the player's last input, the item square that has focus shows its tooltip **beside it**:

- The squares: the bag (list and grid), worn, hotbar and potion, pouch window, loot window and trade squares.
- Right of the square, left when the right does not fit, kept inside the screen, and following each focus move.
- In a grid (the bag's or a pouch window's) it sits beside the whole item, clear of every cell a multi-cell item covers.

Details:

- **It is the same card the pointer would show.** The square's own tooltip cannot be shown twice (Slate has already
  wrapped it).
  - So `USnapFocusedTooltipWidget` — one per player, built on first use, drawn above every plugin window and below the
    modals — holds a second copy of the square's tooltip CLASS and seeds it the way the square seeded its own
    (`USnapInventoryTooltipWidget::CopySubjectFrom`).
  - That copy has every section, the comparison, the weight, the pad list, and your `OnTooltipRefreshed`.
  - A square that shows the pointer no tooltip — an empty square with no title — shows none here either.
  - A re-seed of the square's tooltip (its item changed) reaches the card on the next frame.
- **"On a pad" is the shortcut line's own test**: `GlyphDevicePreference` first (ForceGamepad always, ForceKeyboard
  never), then the device the player used last.
  - So the card shows exactly when its line lists the pad buttons.
- **It hides:**
  - When the mouse or the keyboard is used.
  - When focus leaves the item squares.
  - During a drag or a gamepad carry.
  - While the item menu or a confirmation window is open.
  - When the square's window or screen closes.
  - While the square is the one under a visible mouse pointer (Slate's own card is there).
- `USnapInventoryUISettings::bShowFocusedTooltipOnGamepad` (on) turns it off: tooltips then follow the pointer only,
  and no layer is built.
  - `USnapInventoryHUDComponent::GetFocusedTooltipWidget` returns the layer (`GetTooltipTarget`, `GetCard`,
    `IsShowingCard`).
- **Your own item square** reports its focus with `USnapInventoryHUDComponent::ReportItemSquareFocus(Square, bFocused)` from
  its focus-received and focus-lost events, as the plugin's squares do (from its added-to / removed-from focus path
  events when a widget inside it takes the focus).
  - Its tooltip is whatever it attaches with `SetToolTip`, seeded through `SetFromItem`, `SetFromDefinition` or
    `SetFromEmptySlot`.

## A running effect's card

`SetFromActiveEffect(Effect)` seeds the card for one running effect — what a square of [the buff row](buff-row.md)
shows:

- The effect's name in its item's rarity color (a debuff in the theme's `ActiveEffectDebuffOutline`).
- "Buff" or "Debuff" (and "x2" when stacked) on the type line.
- What it does while it runs, as stat lines.
- The time it has left where an item's status line goes.
- No weight, price, compare block or shortcut line.

`RefreshActiveEffectTime(SecondsLeft)` re-words only the time; `OnTooltipRefreshedForActiveEffect` is its Blueprint
hook, fired instead of `OnTooltipRefreshed`.

The stat lines come from the effect's `Item` through each fragment's `GetActiveEffectLines(ItemDef)`, which returns its
lines like `GetTooltipLines` (the base returns none) — see [What its card says](buff-row.md#what-its-card-says).

## The demo attribute set

`USnapDemoAttributeSet` ships five stats — `MaxHealth`, `Health`, `Armor`, `AttackPower`, `AttackCooldown` — and two
meta attributes for incoming damage and healing (`IncomingDamage`, `IncomingHealing`), on `ASnapDemoPlayerState`'s
ability system component.

- `Health` starts at `MaxHealth`; it is there so damage wear, which ships on and watches `Health` by default, binds in
  the demo.

It exists so the example items move a real number, in the standalone example project as well as in a host game.

- A tooltip reads class defaults and needs no attribute set to exist, so this is not what makes the stat lines *work* —
  it is what makes them *true*.
- `AttackCooldown` is the one shipped stat where lower is better, so that the `bLowerIsBetter` color rule is exercised
  by the shipped content rather than only by a project that happens to have such a stat.

It is an example, not a framework. A project brings its own attribute set and its own table rows; nothing in the reader
knows this class exists.

## When the tooltip has nowhere to put a box

Per-line color needs one widget per line, so the stat lines are drawn in a `UVerticalBox` inserted beside the widget
named by `GrantsTextWidgetName`.

- A tooltip Blueprint that cannot host one still works — the same words are joined with newlines and written to the
  authored grants `TextBlock` instead.

Three layouts take that path:

- The tooltip has **no** grants widget at all (a legitimate design, so it is silent).
- The grants widget has **no parent panel** (it is the root) — logged.
- The grants widget sits directly inside a **single-child container** — a `Border`, `SizeBox`, `ScaleBox` or `Button` —
  so nothing can be inserted beside it.
  - Logged, with the fix: put the grants text in a `VerticalBox`.

**The compare block is dropped on that path, not merely uncolored.**

- A compare block whose gains and losses are painted the same color is not a degraded compare block, it is a misleading
  one.
- So the fallback re-gathers from the definition alone and shows stats and grants only.

When the box *is* drawing, the authored grants `TextBlock` is **collapsed**, or the tooltip would say everything twice.

`BuildGrantsTextForDefinition` is `BlueprintPure` and public: it is also the answer to "what does this item grant?"
anywhere outside a tooltip.

## A definition alone cannot show rarity-scaled stats

`ScaledEquipEffects` take their magnitude from the **instance's** rolled rarity.

- So a tooltip seeded from a definition — a craft recipe preview, a vendor row for something not yet minted — has no
  rarity to scale by and omits those lines.
- It shows flat `GrantedEffectClasses` and `Grants` rows normally.

This is the same rule as an unresolvable magnitude anywhere else:
**a line that cannot be computed is omitted, never shown as `0`**.

- An item whose stats are *entirely* rarity-scaled therefore reads as grants-only until it exists.
- Give such an item a flat effect as well if its preview should say more.

⚠ The same applies when an item's `FixedRarity` names a tag absent from the active `RarityTable`.

- The rank cannot be resolved, so the scaled effect is not applied and the tooltip does not claim it.
- The tooltip agrees with the character, which is the contract; but nothing warns you that the item is mis-tagged.
- So check `FixedRarity` against your table if an item is unexpectedly quiet.

## Without GameplayAbilities

Stat lines read from effects, and grant lines, both come from an item's `GrantedAbilitySet` fragment, which a project
built without `GameplayAbilities` does not compile, so an item shows neither.

- **Stat lines from the item's own data still draw** — give it a `StatContributions` fragment (see
  [Stats an item declares itself](#stats-an-item-declares-itself)), and `GetEquippedStatTotal` answers for them too.
- Affix lines, and lines a fragment adds through `GetTooltipLines`, are unaffected.
- A requirement measured against the player draws no line, because without an ability system no term is in force (see
  [Equipment](equipment.md#requirements)).

## See also

- [Items and fragments](items-and-fragments.md) — fragments, and where `GrantedEffectClasses`, `ScaledEquipEffects` and
  `ConsumeEffectClasses` live
- [Equipment](equipment.md) — slots, and what equipping actually applies
- [UI and theming](ui-and-theming.md) — the tooltip widget and how to reskin it
- [Input](input.md) — the gesture table the shortcut list is read from
- [GAS and Lyra](gas-and-lyra.md) — the ability system this reads from
