# Affixes

Two swords of the same kind, and one of them is better. An **affix** is a rolled modifier an item instance carries for
life — `+4 Strength`, `of the Bear` — decided once, on the server, when the item is created.

**This is the data model, the roll, and what a roll does.** A rolled affix:

- Draws its own tooltip line and comparison delta, and moves the item's sell price — see
  [What a player sees](#what-a-player-sees) and [What a roll is worth](#what-a-roll-is-worth).
- When its row names a stat, moves that stat while the item is worn — see [What a roll moves](#what-a-roll-moves).
- Names the item too: "Sturdy Iron Sword of Fury" — see [Names](#names).

Each item sets which affixes it may roll, how often it is magic at all, how many it rolls and how strong each is — see
[Tuning an item's roll](#tuning-an-items-roll).

- Filtering or searching on affixes is not built.
- `GetInstanceAffixes` is the seam all of it reads, and a project can read it too.

**On this page:**

- [The shape](#the-shape)
- [The table](#the-table)
- [The fragment](#the-fragment)
- [Tuning an item's roll](#tuning-an-items-roll)
- [Reading an item's affixes](#reading-an-items-affixes)
- [What a player sees](#what-a-player-sees)
- [What a roll is worth](#what-a-roll-is-worth)
- [What a roll moves](#what-a-roll-moves)
- [Names](#names)
- [Changing definition](#changing-definition)
- [Rules worth knowing](#rules-worth-knowing)
- [Authoring a table](#authoring-a-table)
- [Replacing the roll](#replacing-the-roll)
- [What is not built](#what-is-not-built)
- [See also](#see-also)

## The shape

| Type | What it is |
|---|---|
| `USnapItemAffixTable` | A data asset: weighted rows, one per affix, each with a magnitude band |
| `USnapItemFragment_Affixes` | On an item definition: *which* table this item rolls from, and *how many* |
| `FSnapRolledAffix` | What one instance rolled: a tag and a number |
| `FSnapAffixState` | The instance's affix memory — the array of the above |

- A roll happens in `OnInstanceCreated`, on the server, and the result lands in the instance's memories.
- That is what makes it per-instance, replicated and saved without any of the three being built for it:
  - Memories are a replicated property on the instance.
  - The persistence record already carries them.

## The table

```cpp
USnapItemAffixTable
```

| Field | Default | Means |
|---|---|---|
| `Rows` | Empty | The affixes this table may yield |
| `bUniquePicks` | `true` | No affix is rolled onto one item twice |
| `MagnitudeScaleByRarity` | Empty | Optional: a multiplier per rarity tier on every value this table rolls |

And one row:

```cpp
FSnapItemAffixRow
```

| Field | Default | Means |
|---|---|---|
| `AffixTag` | None | The affix's identity — the key everything downstream uses |
| `DisplayName` | Empty | Its tooltip line's plain label — the stat it is about (`Attack Power`) |
| `Prefix` | Empty | A word put **before** the item's name (`Sturdy`) — see [Names](#names) |
| `Suffix` | Empty | Words put **after** the item's name (`of Fury`) |
| `PickWeight` | `1.0` | Relative pick weight; **`0` retires the row completely** |
| `MinMagnitude` | `0.0` | The low end of the rolled number |
| `MaxMagnitude` | `0.0` | The high end |
| `Precision` | `0` | Decimal places when the number is drawn |
| `bShowAsPercent` | `false` | Draw it as `+15%` rather than `+15` |
| `bLowerIsBetter` | `false` | Which way a compare delta is *good* — for coloring only |
| `SortOrder` | `0` | Tie-break among tooltip lines of the same kind, low first |
| `ValueMultiplier` | `1.0` | What carrying this affix multiplies the item's worth by |
| `StatAttributeName` | None | The stat this affix moves while the item is worn (`AttackPower`, `Armor`) |
| `StatUnit` | `Flat` | `Flat` (`+3`) or `Percent` (`+5%`) — the unit the Stat Effect Map binds the stat under |

- Everything from `Precision` down decides how an affix **reads, prices and acts**.
- Every one of them defaults to *contributes nothing* — a table that sets none of them behaves exactly as it did before
  they existed.

**Weights and magnitudes:**

- Weights are relative and normalized at roll time.
  - So a table using `3 / 2 / 1` and one using `30 / 20 / 10` behave identically.
  - Adding a row does not mean rebalancing every other number.
- A weight of zero is the one way to retire a row without deleting it.
- Inverted magnitude bounds are swapped at roll time rather than trusted.
- A negative magnitude is a perfectly ordinary thing to author: `-3 Agility` is a cursed affix.
- A band of `0 .. 0` is a real value too, and it is what a flag-like affix — `Soulbound` — wants.

### `bUniquePicks` deduplicates by tag, not by row

- `+2 Strength` and `+8 Strength` are legitimately two rows of *one* affix at different weights.
- A rule that took only the drawn *row* out of the running would leave the other able to win a later draw.
  - So one item could carry Strength twice while the flag read true.
- Deduplicating by tag is what a designer means by "do not repeat an affix".

It defaults **on**, which differs from `USnapLootTable`.

- A loot roll yielding two potions is ordinary.
- An item carrying the same affix twice is almost always a mistake.

A roll cannot yield more affixes than the table has distinct ones. Asking for five out of a two-affix table yields two and
says so in the log.

## The fragment

```cpp
USnapItemFragment_Affixes
```

| Field | Default | Means |
|---|---|---|
| `AffixTable` | None | The table instances of this item roll from |
| `AffixCount` | `1` | How many to roll, or the low end of a range |
| `MaxAffixCount` | `-1` | The high end, or `-1` for "roll exactly `AffixCount`" |
| `AffixCountWeights` | Empty | How likely each count in the range is; empty makes every count equally likely |
| `MagicChancePercent` | `100` | The chance, 0-100, that an instance rolls **any** affix |
| `AllowedAffixTags` | Empty | Which of the table's affixes this item may roll; empty is the whole table |
| `AffixOverrides` | Empty | Per affix: its own % chance on this item, and its own value range |

**How many to roll is per-item data, not per-table.**

- One shared "weapon affixes" table can serve a dagger that rolls one and a legendary blade that rolls three.
- Putting the count on the table as well would be two owners of one number.

**`MaxAffixCount` is `-1` rather than `0` or `1` deliberately.**

- A fragment authored before that field existed keeps the C++ default on load.
- `-1` is the one value no designer can mean:
  - `0` would silently turn "roll two" into "roll between none and two".
  - `1` would cap every pre-existing item at one affix.
  - Neither is distinguishable from a deliberate choice, which is the whole hazard.
- `GetAffixCountRange` is the single place that sentinel is interpreted.
- The roll itself receives a plain inclusive range and has no opinion about "unset".

## Tuning an item's roll

Four knobs on the fragment decide what one item rolls, on top of the table it shares with other items. Each defaults to
"roll as before it existed".

| Knob | Example | Means |
|---|---|---|
| `MagicChancePercent` | `60` | The item is magic 60% of the time; the other 40% it is an ordinary item with no affix |
| `AffixCountWeights` | `1: 75, 2: 25` | When it is magic, it asks for one affix three times in four and two the rest |
| `AllowedAffixTags` | `Fury`, `Sturdy` | Only those may roll on it — a parent tag allows every affix under it |
| `AffixOverrides` | `Fury: 50%, +3 to +6` | Fury has its own 50% chance on this item, and rolls +3 to +6 here |

**The order of a roll:**

1. The magic chance first.
2. Then how many (the count weights, or every count in the range equally likely).
3. Then the allowed list narrows the table.
4. Then each affix with its own chance rolls once. A miss takes it out of the running for this item.
5. Then affixes are picked by the table's relative `PickWeight`, as always.
6. Then each picked affix rolls its value — in this item's own range when an override sets one — and the table's
   `MagnitudeScaleByRarity` multiplies it.

**Good to know:**

- **A per-affix chance is not a weight.**
  - Fury at 50% means at most one item in two *can* roll Fury.
  - Among the affixes still in the running, the table's weights decide as before.
  - Two rolled affixes still need the count to say two.
- **Count weights list the counts that can roll.**
  - Once the list has a row, a count in the range with no row (or a weight of 0) never rolls — the asset validator names
    it.
  - A list with no usable row is ignored.
- **The rarity scale is stored.**
  - A rare item rolling `+4` on a `x1.25` tier carries `+5` from then on, in its tooltip, its stat and its price.
  - Changing the map re-values only items rolled afterwards.
  - When a table scales by rarity, the item's rarity is resolved before its affixes roll, whatever order its fragments
    are listed in.
- **Every default rolls exactly as before** — the same random numbers in the same order, so a seeded roll gives the same
  affixes.
  - A knob that is set draws only what it needs.
- **They travel on the roll request** (`FSnapAffixRollRequest`), so a Blueprint that replaces `RollAffixes` sees them and
  `RollAffixesDefault` applies them.

**The validator names what cannot work:**

- A chance outside 0-100.
- An allowed or overridden affix the table does not define.
- An override the allowed list excludes.
- An override whose low end is above its high end.
- A count weight outside the range.
- A count in the range with no weight.
- A rarity multiplier below 0 or with no tier.

## Reading an item's affixes

```cpp
USnapItemFragment_Affixes::GetInstanceAffixes(Instance);                        // everything it rolled
USnapItemFragment_Affixes::GetInstanceAffixMagnitude(Instance, Tag, OutValue);  // one of them
USnapItemFragment_Affixes::HasRolledAffixes(Instance);                          // has it rolled at all
```

```cpp
USnapItemFragment_Affixes::GetItemAffixTable(ItemDef);             // the table, loading it if needed
USnapItemFragment_Affixes::GetInstanceValueMultiplier(Instance);   // what its roll is worth, 1.0 for none
```

All of them are `BlueprintPure`. An item with no affix fragment answers empty, `false`, `false`, null and `1.0`.

`GetInstanceAffixes` returns tags and numbers, not names.

- The label lives on the table row and is looked up when something wants to draw it.
- That keeps a save culture-neutral (a translated string written into a save file renders in the culture it was written
  in, forever).
- It also means renaming or rebalancing an affix reaches items that already rolled it.

**To write affixes outright** — a crafted item whose modifiers are chosen rather than drawn, a quest reward — use
`SetInstanceAffixes`.

- It is server-only, and after it the instance counts as rolled.

## What a player sees

A rolled affix draws **one tooltip line each**, labeled from the row's `DisplayName` — a plain stat — and carrying the
magnitude that item actually rolled:

```
+3   Attack Power
+12% Max Health
```

- It is drawn in the theme's `TooltipMagicModifier` color (pale blue) — its number and its words.
  - So it reads apart from the item's own stat of the same name ("+2 Attack Power" from Fury under the sword's own "+5
    Attack Power").
- The line is an **ordinary stat line** — `ESnapTooltipLineKind::Stat`.
  - So it sits with the item's other stats rather than in a block of its own.
  - `SortOrder` is how a project groups affixes together or pins one above the rest.
- Two entries of one affix read as **one** line carrying their sum, the same way two effects moving one attribute do.

Swapping to an affixed item reports the difference under the usual "Compared with" heading:

```
Compared with: Iron Cuirass (Chest)
  +3   Edge          (the candidate has it, the worn item does not)
  -5   Guard         (the worn item has it, the candidate does not)
  +5   of the Bear   (both have it, at different magnitudes)
```

- A delta is the **union** of what either item carries, with the side that lacks an affix contributing zero.
  - So an affix only the *worn* item has is a loss of its whole magnitude, because swapping gives it up.
- An affix both carry at the same magnitude draws no line (no `+0`).
- Where both items' tables define one affix, the **incoming** item's row names it.
  - The tooltip is about the incoming item.
  - The worn item's table is the only source for an affix the incoming one does not carry.
- A swap that changes **only** affixes still gets its heading.
- A swap that changes nothing still gets none.

> **Nothing here is re-derived.**
>
> - Every number is the stored roll read back. The table supplies the words and the formatting and nothing else.
> - Two reads of one item are identical by construction.
> - That is what keeps a tooltip from showing a different sword each time the cursor moves over it.

### An affix your table cannot name is still drawn

This is when an item carries an affix the table has no row for — a row deleted after items had rolled it, a table that
did not load.

- The line is drawn anyway, labeled from the tag in words.
- The log says so once.
- The item *is* carrying it, so hiding it would hide something the player owns.
- That label is derived from an identifier and is **not translatable**.
  - That is why a rollable row with no `DisplayName` is an editor-time warning.

## What a roll is worth

`ValueMultiplier` on a row is what **carrying** that affix multiplies the item's sell price by.

- An item's affixes compose multiplicatively: two at `1.15` make it worth `1.3225x`.
- It folds in beside the freshness and durability factors the sell price already carries, all into one rounding.
- It is switched at `bAffixesAffectSellPrice` (**Snap Inventory System - Economy → Vendor**; **on**, and inert until an item carries affixes whose rows set a multiplier other than `1.0` — the example content's `DA_ExampleAffixTable` does: x1.25 and x1.15).

It is **flat, not scaled by the rolled magnitude** — `+2 Strength` and `+8 Strength` from one row are worth the same.

- Scaling by magnitude sounds obviously right, and needs a reference point to scale against.
- The band's own ends are the only candidate.
- Using them couples an item's *price* to a balance number a designer expects to retune freely.
  - So widening a band would silently re-price every item already carrying the affix.
- **A project that wants it has the seam already:** `USnapVendorClientComponent::ResolveSellPrice` is handed the computed
  number and can read the instance's roll.

> **It is the one sell factor that can RAISE a price.** Freshness and durability each scale down. That is why the affix
> factor is accumulated separately from them and the result is clamped explicitly rather than assumed to fit.

**An affix with no row contributes exactly `1.0`** — not zero and not a guess.

- An affix table named only by a soft path can be absent from a packaged build.
- So a table that loads on the server and not on a client is a real state.
- `1.0` is the one answer that is the same either way.
  - That is what keeps the number on the button and the number credited from disagreeing.

## What a roll moves

A row that names a stat in `StatAttributeName` makes its affix **do** something: while the item is worn, the magnitude it
rolled is added to that stat, exactly as an item's own `Stat Contributions` rows are.

The same two places read it:

- `USnapEquipmentComponent::GetEquippedStatTotal` counts it, in every build.
- With GameplayAbilities, equipping applies the effect the **Stat Effect Map** binds to (`StatAttributeName`, `StatUnit`),
  with the rolled number as its SetByCaller magnitude.
  - That is the bridge an item's stat rows already use, so unequip, swap, drop and death take it away with the rest.

```
Fury    StatAttributeName AttackPower   StatUnit Flat      rolled +4  ->  Attack Power +4 while worn
Vigor   StatAttributeName MaxHealth     StatUnit Percent   rolled +6  ->  Max Health +6% while worn
```

`USnapItemFragment_Affixes::ResolveAffixStatContributions(Instance, OutRows)` is the one function that turns an
instance's rolls into those stat rows, and a project's own stat system can call it too.

- **It is drawn once, as the affix's own line.**
  - The tooltip does not add a second "Attack Power" stat line for it.
  - So word the row's `DisplayName` as the stat it moves — the example table's read `Attack Power`, `Armor`,
    `Max Health`.
  - Set `bShowAsPercent` for a `Percent` stat.
  - ⚠ An item whose own stat rows move the same stat therefore shows two lines of that name (`+5 Attack Power` from the
    sword, `+2 Attack Power` from its affix).
- **The unit must be one the Stat Effect Map binds.**
  - A stat with no binding still counts toward the total and moves no attribute, with the same once-per-stat Warning an
    unbound stat row gets.
  - `From Modifier Op` is refused at save (a rolled number has no modifier to ask).
- **A broken or spoiled worn item moves nothing**, as no equip-time grant does.
- **An affix whose row names no stat moves nothing** — every table authored before this field, and every flag-like affix.

## Names

An item with affixes is **named** by them: a row's `Prefix` goes before the item's name and its `Suffix` after it.

```
Iron Sword  +  Sturdy (prefix)  +  of Fury (suffix)   ->   Sturdy Iron Sword of Fury
```

- **At most one prefix and one suffix**: Each from the **first** affix the item rolled that has one, in the order it
  rolled them.
  - A sword that rolled Fury and then Vigor — two suffixes — is "Iron Sword of Fury".
  - "First" rather than "strongest" because rolled values of different affixes are not comparable (+3 Attack Power
    against +6% Max Health).
  - And because the roll order is stored, so a loaded item keeps its name.
- **One function names an item everywhere.** `USnapInventoryLibrary::GetInstanceDisplayName` answers it
  (through `USnapItemFragment_Affixes::GetInstanceAffixedName`).
  - Every surface that names an *item* asks it:
    - The tooltip's title — in the bag, the hotbar, loot and storage windows and trade squares.
    - The item menu's header.
    - The sell window and sell rows.
    - A vendor's sell-mode and repair / upgrade titles.
    - The collect feed.
  - A *kind* of item — a shop's stock, a recipe's output — keeps its plain name.
- **The word order can be translated.**
  - The name is three keyed patterns — `{Prefix} {Item}`, `{Item} {Suffix}` and `{Prefix} {Item} {Suffix}`.
  - They are in namespace `SnapInventorySystem`, keys `AffixedItemNamePrefix`, `AffixedItemNameSuffix`,
    `AffixedItemNamePrefixSuffix`.
  - So a language can put the words where it needs them.
- **It is composed where it is drawn**, from the stored tags, never sent as text.
  - The collect feed's report carries the item's rolled affixes.
  - The player's own machine names it in its own language (`USnapInventoryComponent::GetReportedItemAffixes` during
    the broadcast; `ComposeAffixedName(ItemDef, Affixes)`).
- **A table whose rows set no prefix or suffix names nothing**, so a project that authored affixes before this keeps every
  name as it was.
- **Each item decides which words its name takes** — the fragment's `AffixNaming` (`ESnapAffixNaming`).
  - The choices: `Both` (the default, the rule above), `Prefix Only`, `Suffix Only` or `None (Plain Name)`.
  - It is for an item whose own name already ends in "of ...".
  - The example Ring of Vitality and Amulet of Mending are `Prefix Only`.
    - So a Sturdy roll reads "Sturdy Ring of Vitality".
    - And a Fury- or Vigor-only roll keeps the plain name (instead of "Ring of Vitality of Fury").
  - It decides the NAME only: every rolled affix still draws its line and moves its stat.
  - An item authored before the setting existed stores no value for it and keeps `Both`.

## Changing definition

- An item that **transforms** — food rotting into its spoiled form — is the same physical item.
  - So its identity and its memories travel to the replacement.
- Its affixes travel **only when both definitions name the same affix table**.
  - Otherwise the carried tags could never be looked up again in the new item's own table.
  - So the replacement keeps the roll it performed for itself instead.
- An item that is **upgraded** always rolls fresh.
  - An upgrade is a different item rather than the same one in a different state.
  - It has always minted its target without carrying memories across.

## Rules worth knowing

- **The roll is the server's, and a client never rolls.**
  - A client that rolled its own affixes would show a different sword from the one the server thinks it has, silently,
    until the two disagreed about damage.
  - The fragment refuses without authority before it even loads the table, and the memory write underneath refuses
    again.
  - A client's copy arrives by replication like every other piece of per-instance state.
- **Determinism means save and load, not reproducing a seed.**
  - What has to hold is that the sword you put down is the sword you pick back up — carried by the instance's memories,
    with no seed stored anywhere.
  - A stored seed would be a second, weaker way to say the same thing.
  - And the two would disagree the first time somebody edited the table.
- **An existing roll is never rolled over.**
  - The creation hook runs on every path that mints an instance, and several of those are preserving an item rather than
    making one.
  - Without that rule, dropping an item and picking it back up would be a way to re-roll it until the affixes were good.
- **An affixed item must not be stackable**, and the item definition's validator refuses the combination.
  - A roll belongs to one instance, so a stack merge would discard one of two rolls.
  - And a stack split, which mints a new instance, would roll fresh affixes for the half that came off.

## Authoring a table

- The plugin's runtime ships **no affix table and no affix names** — nothing outside the example content names one.
- An affix vocabulary is entirely a project's own.

**The example content** ships one for the demo: `Content/Examples/Items/DA_ExampleAffixTable`. It has three affixes over
the demo's attributes, with their tags in the plugin's tag file (`SnapInventorySystem.Example.Affix.*`):

- Fury (`Attack Power` +2 to +5, suffix "of Fury").
- Sturdy (`Armor` +1 to +3, prefix "Sturdy").
- Vigor (`Max Health` +3 to +8%, suffix "of Vigor").

**In the demo:**

- The example weapons, armor and jewelry roll none, one or two of them.
- The Iron Axe shows the knobs.
- The Ring of Vitality and Amulet of Mending take a prefix only (`AffixNaming`).
- It goes with the examples; see [the demo map](demo-map.md#magic-items).

**To author your own:**

1. Create a `USnapItemAffixTable` in your own content (**right-click → Snap Inventory → Affix Table**).
2. Declare your affix names in a gameplay tag source.
   - A tag no source declares imports as **empty** rather than failing, so a misspelled affix silently becomes an affix
     nothing can look up.
   - That is why the table's validator refuses a row with no tag.
   - The plugin's own `Config/Tags/SnapInventorySystemTags.ini` is the worked example of a tag source that ships with a
     plugin.
3. Add a `USnapItemFragment_Affixes` to the item definitions that should roll.
4. Point it at the table.
5. Set the count.

> **A table reachable only from a soft path is invisible to the cooker.**
>
> - The fragment names its table softly — a hard pointer in a fragment reachable from a class default pulls the asset
>   into memory for every project that loads the module.
> - A table referenced by an item definition the cooker already follows needs nothing extra.
> - A table reachable *only* from a config default needs a `DirectoriesToAlwaysCook` rule, or it works in the editor and
>   is absent from a packaged build.

## Replacing the roll

`RollAffixes` is a `BlueprintNativeEvent` on the table.

- Subclass `USnapItemAffixTable` as a Blueprint and override it to replace the algorithm outright — pools that read the
  player's level, pity timers, a table that never gives one player the same affix twice.
- `RollAffixesDefault` is public and static, so an override can call it and then add to or filter its result rather than
  reimplementing weighting.

## What is not built

- **Filtering and searching on affixes.**
  - The search box matches the item's definition name, not the name its affixes give it, so "Fury" finds nothing.
  - Both of the surfaces that would host it are closed by rules held on purpose:
    - The search box matches an item's **display name** and only that, so that one box never means two things.
    - The filter bar beside it is **definition-level** — it has no instance in its signature, and a rolled affix is
      per-instance, so two items of one definition would filter differently.
  - Either is a deliberate change to a surface a project configures rather than an oversight to patch.
  - Sorting by name sorts by the definition name too.
- **Sorting** needs nothing: an affixed item already sorts differently under the existing `Value` mode, because that mode
  reads the per-instance sell price and a roll now moves it.

## See also

- [Items and fragments](items-and-fragments.md) — the fragment list, and writing one of your own
- [Loot tables](loot-tables.md) — the weighted-row asset this one follows
- [Persistence](persistence.md) — the record a rolled affix travels in
- [Networking](networking.md) — why the server decides and a client is told
- [Tooltips and stats](tooltips-and-stats.md) — the line kinds a rolled affix is drawn as
