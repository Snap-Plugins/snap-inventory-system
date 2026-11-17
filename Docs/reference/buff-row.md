# The buff row

The buff row is a strip of small squares on the HUD, one for every effect running on the player — a buff or a debuff. This
page covers placing it, what a square shows, where the effects come from, the Active Effect Rows setting and what an
effect's card says.

- **Each square** has its picture and the time it has left.
- **Squares fill a row from left to right.** When a row is full, the next one starts **above** it.
  - So the block grows upward from where it is placed.
  - It never covers what is under it (a health bar, say).
- **The squares are translucent,** so the game shows through them.
- **When an effect ends,** the squares close up.
- **It works with GameplayAbilities and without it.** The widget asks one question, and the plugin answers it from both.

**On this page:**

- [Placing it](#placing-it)
- [What a square shows](#what-a-square-shows)
- [Where the effects come from](#where-the-effects-come-from)
- [Active Effect Rows](#active-effect-rows)
- [What its card says](#what-its-card-says)
- [The demo](#the-demo)

## Placing it

The plugin does not put the buff row on screen — a HUD has its own layout — so add it to yours:

1. **Add `WBP_ExampleActiveEffects`** (or the native `USnapActiveEffectsWidget`) to your HUD Widget Blueprint, in a Canvas
   Panel.
2. **Put its bottom-left corner where the first row should start:**
   - Anchors at the top left.
   - **Alignment (0, 1)**. With alignment (0, 1), a second row grows upward from there.
   - Size To Content on.
   - Position just above your health bar.
3. **Leave room above it** for the rows you expect.
   - A 30-unit square and a 4-unit gap: two rows take 64 units.

**Its own settings, on the widget:**

| Setting | Value | Notes |
|---|---|---|
| `SquareSize` | 30 | |
| `SquareGap` | 4 | |
| `MaxSquaresPerRow` | 8 | |
| `RefreshSeconds` | 0.25 | How often it asks for the effects. The countdowns run smoothly in between. |
| `SquareWidgetClass` | `WBP_ExampleActiveEffectSquare` | The square as a Widget Blueprint you can restyle. Empty is the native square. |

**On the square:** `TooltipWidgetClass` picks the card its hover shows.

- Empty (as shipped) is the item card your inventory screen's squares use, as the HUD hotbar squares do.

The demo places it just above its stats panel, whose health bar is the panel's first line.

## What a square shows

- **The picture:** It comes through the art pack's icon resolve, like every item picture.
  - An effect with no picture shows the first letter of its name instead.
- **The seconds left** in the bottom-right corner: "12", or "3m" from a minute up.
- **A shade** that grows over the square as its time is used up.
  - An effect that lasts until something removes it shows neither the seconds nor the shade.
- **A red frame on a debuff.** A buff's frame is light.
- **"x2"** in the top-left corner when the effect is stacked.
- **Hover it** for its card. See [What its card says](#what-its-card-says).
  - It is an item card, sized and colored like an item's.
  - It shows the effect's name in its item's rarity color (a debuff's in `ActiveEffectDebuffOutline` red).
  - Under the name: "Buff" or "Debuff" (and "x2").
  - What it does while it runs: "+5 Attack Power", "+4 Health per second", "+50% Speed", "+25 Armor".
  - The time it has left, re-worded each second.
  - Nothing about weight, price, cooldowns or shortcuts.
- **The card is a mouse one,** because the HUD is not part of a screen a gamepad moves through.
  - During play, hold `CursorPeekKey` (the middle mouse button) for a cursor to point with.
  - The demo HUD hides the row while the bag is open.

**Colors** come from the theme's Active Effects roles:

- `ActiveEffectBackground`
- `ActiveEffectBuffOutline`
- `ActiveEffectDebuffOutline`
- `ActiveEffectIcon` (its alpha makes the picture see-through)
- `ActiveEffectElapsed`
- `ActiveEffectTimeText`

## Where the effects come from

The row draws `SnapActiveEffects::Gather` for its owning player.

- **In Blueprint:** **Get Active Effects**, on the Snap Inventory System library.

That is the **union** of two sources:

- **Your own effects, through `ISnapActiveEffectSource`.**
  - Implement it on the player's controller, PlayerState or pawn — or a component of one.
  - List what is running (`GetSnapActiveEffects`):
    - A tag that says what each effect is.
    - The item it came from.
    - A name and a picture, if you have them.
    - Its `bDebuff` flag.
    - The seconds left and the whole length.
  - **Everything a source lists is shown.**
  - This is the route without GameplayAbilities.
  - It is also the one for effects that do not live on an ability system at all (a potion system of your own).
  - The demo's stats component is one.
- **With GameplayAbilities, the player's ability system** (the PlayerState's, else the pawn's): its active effects that a row
  of **Active Effect Rows** names.
  - **An ability-system effect no row names is not shown.** An ability system also holds cooldowns and helper effects that
    are nobody's buff.

**The same buff is not shown twice:**

- An ability-system effect that carries a tag a source already listed is skipped.
- A source should not list what lives on the ability system.

## Active Effect Rows

**Where:** **Project Settings > Plugins > Snap Inventory System - Foundation > Active Effects > Active Effect Rows**. It is empty as shipped.

Each row says which effects it is about and how they look:

| Field | What it does |
|---|---|
| `Tag` | Matches an effect that carries this tag or a child of it: a source effect's own `Tag`, or any tag an ability-system effect grants or carries as an asset tag. |
| `EffectClass` | Or matches an ability-system effect of this GameplayEffect class (or a subclass). |
| `DisplayName` | The name its square's tooltip shows. Empty: the effect's own name, else `IconItem`'s. |
| `Icon` | The square's picture. Unset: `IconItem`'s picture, else the effect's own. |
| `IconItem` | Or borrow an item's picture (and its name when `DisplayName` is empty) — the scroll a spell came from. An effect with no `Item` of its own takes this one as its `Item`, so its card says what it does. |
| `bDebuff` | A harmful effect: its square gets the red frame. |

- **The first row that matches wins.**
- **A potion needs no row.** An effect a source lists with an `Item` and no row shows that item's picture and name.
- **The demo's two rows** (set for its session): Haste and Ward by their state tags, each drawn with its scroll's picture.

## What its card says

**How the card is kept up to date:**

- The square seeds its card with `USnapInventoryTooltipWidget::SetFromActiveEffect`.
- Afterwards it re-words only the time (`RefreshActiveEffectTime`), so a card that is up never closes and reopens.
- The card's Blueprint hook is `OnTooltipRefreshedForActiveEffect`.
- With no card class anywhere, the square falls back to the themed plate with the name and the seconds.

**What it does** comes from the item the effect came from — its `Item`:

- That is the potion that was drunk, or the item a row names in `IconItem` (the scroll a spell came from).
- Every fragment of that item is asked `GetActiveEffectLines`.
  - It returns its lines (`FSnapTooltipLine`, as `GetTooltipLines` does).
  - It returns none on the base fragment.
- Write what the effect does **while it runs** — "+5" and "Attack Power".
  - Not what using the item does: no cooldown, nothing instant, no duration (the card shows the time left).
- The lines are drawn in `SortOrder`, as two columns.
  - They sit under no heading unless a line names its `Section`.

**To describe an effect of your own,** give it an item whose fragment answers `GetActiveEffectLines` (in C++ or in Blueprint).
Then either:

- A source lists the effect with that `Item`.
- Or a row names it in `IconItem`.

An effect with neither shows its name, Buff or Debuff, and its time.

**In the demo:**

- The potions' fragment lists their raises and their heal over time.
- The scrolls' fragment lists Haste's speed and Ward's armor.
- These are the same numbers the use and the cast apply, on either route.

## The demo

- **Try it:** Drink a timed potion (Might, Ironskin, Elixir of Life, Regeneration, Vigor, Battle Tonic) or cast Haste or Ward.
  - You should see a square appear above the stats panel, counting down.
- **While any of them runs,** the demo character wears a soft rim of light (`ASnapDemoCharacter::BuffGlowMaterial`,
  `M_ExampleBuffAura`), fading in and out.
  - Other players see it too.
