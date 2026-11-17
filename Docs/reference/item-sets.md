# Item sets

A **set** is a named group of items that does something extra when several of them are worn at once — the
"2-piece bonus / 4-piece bonus" every set-bearing game has.

- **The example:** Wearing two pieces of the example Iron Armor grants one effect; wearing four grants that one **and**
  another.

Two assets and one rule:

| Part | What it is |
|---|---|
| `USnapItemSet` | The **tiers**: how many pieces each rung needs, and what it grants |
| `USnapItemFragment_ItemSet` | **Membership**: this item belongs to that set |
| The rule | A tier applies while enough **worn** pieces of its set are on, and is removed when they are not |

**On this page:**

- [Tiers are cumulative](#tiers-are-cumulative)
- [What counts as a worn piece](#what-counts-as-a-worn-piece)
- [Stat rows](#stat-rows)
- [The effects](#the-effects)
- [Where the bonuses are computed](#where-the-bonuses-are-computed)
- [Authoring a set](#authoring-a-set)
- [The set does not list its members](#the-set-does-not-list-its-members)
- [Tests](#tests)

## Tiers are cumulative

A set with a 2-piece tier and a 4-piece tier has **both** active at four worn pieces.

- This is not a preference — it is the only one of the two conventions that can express the other cheaply.
- An author who wants "the 4-piece tier *replaces* the 2-piece" writes one tier at 4 and nothing is duplicated.
- Under replace-the-lower, an author who wants cumulative has to copy every lower tier's effects into every higher tier.
  - That is a maintenance hazard no validator could catch (edit the 2-piece bonus, forget its copy inside the 4-piece
    row, and the two silently diverge).

Tiers are read **in order of requirement**, not in the order they were authored, so adding a 3-piece rung under a
4-piece one changes nothing.

## What counts as a worn piece

**One piece per occupied equipment slot.**

- A slot holding a stack of three identical rings is one thing being worn; two identical rings in two slots are
  correctly two.
- An item that blocks other slots (`BlocksSlots`) still occupies exactly one entry, so nothing is double counted.
- A set piece **in the bag counts for nothing** — "wearing four pieces" means four slots.

⚠ **A broken or spoiled piece is worn and does not count.**

- A set bonus is an equip-time GameplayEffect on the same ability system as every other one the equipment component
  applies, and all of those are gated the same way.
- So a broken chestpiece granting none of its own stats while still completing a four-piece set would be the odd
  reading.
- Repairing it brings the bonus back with nothing else changing.

> ⚠ Spoilage has one gap, and it is the pre-existing one: nothing re-derives an item's equip-time effects the moment an
> **already-equipped** item's sweep marks it Spoiled.
>
> - So a set piece that spoils while worn keeps its credit until the next equipment change.
> - Breaking does not have this gap: the durability crossing re-derives immediately.
> - Except when that crossing arrives *during* an effect re-derive this component is already performing, which it
>   declines rather than nests. The next equipment change covers it.

## Stat rows

A tier can say what it adds as **data**: `Stats`, the same rows an item's Stat Contributions fragment carries — a stat's
bare name, `Flat` or `Percent`, an `Amount`.

They are read the same two ways an item's rows are:

- `USnapEquipmentComponent::GetEquippedStatTotal` counts them while the tier is active, in **every build** and on
  every machine.
  - It counts the worn pieces itself, so a client answers what the server does.
- With `GameplayAbilities`, each row is applied through the project's **Stat Effect Map** when the tier becomes active.
  - It goes into the same per-tier ledger as its effects, so it comes off by handle with them.

So a set bonus written as rows works in a project **without** `GameplayAbilities`, which `Effects` cannot.

- ⚠ A tier's rows do not scale by rarity — a tier has no item to read a rank from — so only `Amount` counts.
  - Validation warns about an `AmountPerRarityRank` on one.
- The example Iron set's two tiers are rows.

## The effects

A tier's `Effects` are applied to the wearer's ability system at level 1 and removed **by handle** when the tier stops
being active.

- Removal by handle cannot mis-target, which is why a set bonus never needs an identifying tag.

Without `GameplayAbilities` no tier effect is applied — use [stat rows](#stat-rows) for a bonus that should work there.

- The worn pieces are still counted, and `USnapEquipmentComponent::GetWornPieceCountForSet` reads the count for a
  reward of your own.
- The Instant and SetByCaller checks below run only with `GameplayAbilities`.

⚠ **They must be Has Duration or Infinite, never Instant.**

- An Instant effect writes the attribute's *base* value and returns no handle, so there would be nothing for the tier to
  remove.
- Take the set off and put it back on and the base ratchets permanently.
- `IsDataValid` refuses an Instant tier effect, and the runtime logs a warning naming the asset if one reaches it
  another way.

⚠ **Their modifiers need a magnitude of their own, not SetByCaller.**

- A tier applies each effect from its class default with no SetByCaller values assigned.
- So a SetByCaller modifier holds a live handle and moves nothing: the effect is in the active list, every count agrees,
  and the attribute does not change.
- The plugin's per-item equip effects (`GE_Example_EquipArmor`, `GE_Example_EquipVitality`) are SetByCaller — their
  magnitude scales with rarity.
  - That is why a set needs effects of its own, or stat rows.
- `IsDataValid` warns about a SetByCaller tier effect.
  - It is a warning rather than an error because a project's own effect may assign the value another way.

⭐ **An ability-granting tier needs no ability array.**

- A `UGameplayEffect` can grant abilities itself (`UAbilitiesGameplayEffectComponent`), so
  "the 4-piece tier gives you a dash" is authored as an effect that grants it.
- That is why a tier has one list and not two.
- A second, ability-shaped list would double the bookkeeping and its removal symmetry — which is exactly where leaks
  live — to reach a case the engine already reaches.

## Where the bonuses are computed

`USnapSetBonusComponent::RefreshForEquipment` re-derives **every** set bonus from what is worn and applies
the difference.

- It runs from `HandleEquipmentListChanged`, the one funnel every "what is worn changed" passes through.
- It is **server only** — a GameplayEffect is authoritative state, and the ability system replicates the result itself.

A nested refresh is **dropped** rather than queued.

- Applying a gameplay effect can synchronously reach back into the equipment component — a gameplay cue callback, an
  ability reacting to a tag it just gained.
- A nested call would arrive while the outer one had applied a tier and not yet recorded it, see the tier as inactive,
  and apply it a second time.
- The outer call re-derives the whole worn set when it resumes, so nothing is lost by dropping the inner one.
- The attribute moves a refresh causes are also marked as the component's own, so a +MaxHealth set bonus dropping when a
  piece comes off is not read as a hit by the damage-wear listener.

⭐ It is a re-derive rather than an incremental apply-on-equip / remove-on-unequip, and that is the whole design.

- A set bonus is wanted by the **set**, not by any one piece.
- So an incremental implementation would take a 2-piece bonus off the moment the first of four pieces came off.
- Counting the worn set and diffing the tiers makes that correct by construction.
- It also makes the refresh idempotent: calling it when nothing changed applies and removes nothing.

Three readers exist for a HUD or a test:

```cpp
int32  GetWornPieceCountForSet(const USnapItemSet* Set) const;
bool   IsSetBonusActive(const USnapItemSet* Set, int32 TierIndex) const;
int32  GetActiveSetBonusEffectCount() const;   // asserted at zero: "the bonus came off" and
                                               // "the handles were released" are different claims
```

With `GameplayAbilities` (only that build has it), `USnapSetBonusComponent`, which lives beside the equipment manager on
the server, answers `IsSetBonusActive` and `GetActiveSetBonusEffectCount` for Blueprint too.

## Authoring a set

The plugin's example armor is a set: `DA_ExampleItemSet_Iron`, the **Iron Armor**.

- `ID_Chest_Iron`, `ID_Gloves_Iron`, `ID_Greaves_Iron` and `ID_Boots_Iron` each carry an Item Set fragment naming it.
- Two pieces reach **2 pieces: Ironclad** (a stat row: `MaxHealth` +10 `Percent`).
- Four add **4 pieces: Bulwark** (a stat row: `Armor` +5 `Flat`).
- Rows, so the bonus works with and without `GameplayAbilities`.
  - With it they move attributes of [the demo attribute set](tooltips-and-stats.md#the-demo-attribute-set) through
    `DA_ExampleStatEffectMap`.
  - So on an ability system without that set they apply and change nothing.

A project's own sets are its own item lists and balance numbers. To author one:

1. Create a `USnapItemSet` in your own content (**right-click > Snap Inventory > Item Set**).
2. Give it a `DisplayName` and one `Tiers` row per rung. Each row has:
   - `RequiredPieces`.
   - A `DisplayName` for the rung.
   - What it grants: `Stats` rows (see [Stat rows](#stat-rows)) and/or `Effects`, Has Duration or Infinite, with
     magnitudes of their own rather than SetByCaller (see [The effects](#the-effects)).
3. Add a `USnapItemFragment_ItemSet` to each item definition that belongs to the set, and point its `ItemSet` at the
   asset.
   - That is exactly what the four Iron pieces do.
4. Wear two of them. Nothing else is required.
   - The bonus is applied the moment the second piece goes into a slot and removed the moment it leaves.

> ⚠ **A set named only from a soft path is invisible to the cooker.**
>
> - The fragment names its set softly — a hard pointer in a fragment would pull every set asset, and through their tier
>   effects every GameplayEffect in them, into memory for every project that loads the module.
> - A set referenced by an item definition the cooker already follows needs nothing extra.
> - A set reachable *only* from a config default needs a `DirectoriesToAlwaysCook` rule, or it works in the editor and
>   is absent from a packaged build.

## The set does not list its members

Membership lives on the item and nowhere else.

- A set that also listed its members would be two derivations of one fact, and the day they disagreed neither would be
  obviously wrong.
  - It is the same reasoning that puts an affix **count** on the item rather than on the affix table.
- It also means a set needs no registry: the equipment component resolves each worn item's set and groups by the
  resolved asset.
  - So there is nothing for a project to configure.

## Tests

| Test | What it holds down |
|---|---|
| `SnapInventory.ItemSets.TiersAreCumulativeAndTheBoundaryHoldsFromBothSides` | One piece short applies nothing; the requirement applies it; four pieces apply both |
| `SnapInventory.ItemSets.TiersComeBackByRequirementWhateverOrderTheyWereAuthoredIn` | Authored order decides nothing |
| `SnapInventory.ItemSets.ATierRequiringNoPiecesNeverApplies` | Refused at runtime as well as by the validator |
| `SnapInventory.ItemSets.MembershipIsReadFromTheItemAndAnItemWithoutTheFragmentBelongsToNothing` | The resolver, and its control |
| `SnapInventory.ItemSets.ValidationRefusesAZeroRequirementTierAndAnInstantTierEffect` | Both refusals name their rule |
| `SnapInventory.ItemSets.ValidationWarnsAboutASetByCallerTierEffect` | A SetByCaller tier effect is warned about, and a Scalable Float one is not |
| `SnapInventory.ItemSets.ASetBonusAppliesAtItsCountAndComesOffSymmetrically` | The boundary on a live ability system, up and down, plus idempotence |
| `SnapInventory.ItemSets.ATierStaysOnWhileEnoughPiecesRemainAndGoesOffWhenTheyDoNot` | Losing a piece that is not the one that earned it |
| `SnapInventory.ItemSets.AnItemInNoSetIsUnaffectedAndAWearerWithNoSetHoldsNothing` | Defaults change nothing, with the control that says so |
| `SnapInventory.ItemSets.ABrokenPieceDoesNotCountTowardASetAndRepairingItRestoresTheBonus` | The durability gate, and the repair that proves it was the gate |
| `SnapInventory.ItemSets.ARefreshArrivingFromInsideTheApplyAppliesNothingTwice` | A nested refresh, driven from inside the apply itself, applies nothing twice |
