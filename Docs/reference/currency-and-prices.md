# Currency and prices

Every shop keeps its own till. This page covers how a price's currency is chosen, paying, the icon beside a price,
changing what something costs, an item's own worth, the junk merchant, what a shop offers, shelves, restocking and
buyback.

- The shop functions below (the price hooks, selling, the sell plan and buyback) are on the player's
  `USnapVendorClientComponent`, and the pay functions on the player's `USnapWalletComponent`; both sit beside the main
  bag.
- The currency rows (`Currencies`), the global `CounterTag` and `SellPriceFraction` are on **Snap Inventory System - Inventory**;
  the shop settings (`VendorCurrencyTag`, the stock scope, restocking and `CounterIcon`) are on **Snap Inventory System - Economy**.

**Background:**

- A price used to be denominated in exactly one thing: `USnapEconomySettings::VendorCurrencyTag`, the project's
  own money.
- That setting still exists and is still the answer most of the time.
- But it is now the *last* step of a rule rather than the whole of it.
  - So a gem shop can stand beside a coin shop.
  - And the same sword can cost 50 of one thing in one and 50 of another in the other.

**On this page:**

- [The rule](#the-rule)
- [Setting up a second shop](#setting-up-a-second-shop)
- [Paying](#paying)
- [The icon beside a price](#the-icon-beside-a-price)
- [Changing what something costs](#changing-what-something-costs)
- [An item's own worth](#an-items-own-worth)
- [A shop that buys anything](#a-shop-that-buys-anything)
- [What a shop offers](#what-a-shop-offers)
- [A shop's own sell fraction](#a-shops-own-sell-fraction)
- [Naming a shelf](#naming-a-shelf)
- [Restocking](#restocking)
- [Buying back what you sold](#buying-back-what-you-sold)
- [Trade has no currency policy](#trade-has-no-currency-policy)

## The rule

One function answers "what currency is this price in?", and everything asks it:

```
USnapInventoryComponent::ResolveCurrencyTag(NamedTag, VendorTag)
```

**First valid tag wins.**

| Step | Where it comes from | Means |
|---|---|---|
| 1. What the price names for itself | `FSnapCraftingRecipe::CurrencyTag` | Costs this currency **anywhere** |
| 2. What the shop deals in | `USnapVendorStockDataAsset::CurrencyTag` | Costs the shop's currency |
| 3. The project's own money | `USnapEconomySettings::VendorCurrencyTag` | The fallback |

Two convenience forms sit on top of it:

```
ResolveActiveCurrencyTag(NamedTag)    NamedTag -> the OPEN shop -> the global.  What nearly everything wants.
GetActiveVendorCurrencyTag()          what the open shop deals in, or empty
GetDefaultCurrencyTag()               step 3 alone -- for a price with genuinely no context
```

⚠ **An empty tag means "inherit", never "no currency".** That is not a convenience, it is a safety property.

- `FGameplayTag` has a custom text serializer.
  - So a misspelled or undeclared tag name imports as an **empty tag** rather than failing, silently, with the bad name
    gone before anything can report it.
- Under "inherit", the worst outcome of that typo is a shop trading in the project's own money.
- Under "no currency" it would be a price denominated in nothing.
  - That reads as *free* or as *unaffordable* depending purely on which side of a comparison it lands, and logs neither.

⚠ This is also why nothing validates these tags.

- "Names a tag that does not exist" is not expressible by the time any validator runs.
- What *is* checked is the end of the chain: if step 3 is empty too, the project has not said what money is.
  - Then every priced door refuses **by name** instead of charging nothing.

## Setting up a second shop

1. Give the second `USnapVendorStockDataAsset` a `CurrencyTag`, and leave the first one's empty:

```
DA_CoinShop    CurrencyTag  (empty)                              -> the project global
DA_GemShop     CurrencyTag  SnapInventorySystem.Counter.Gems     -> gems
```

2. Declare the counter tag natively with `UE_DEFINE_GAMEPLAY_TAG_COMMENT`, or in your project's `Config/Tags`.
   - See the ⚠ above for what an undeclared one does.

**Nothing else is needed.**

- Prices in `Rows` are plain numbers.
- The currency is a fact about the shop, so the same row moved between two shops is priced in each shop's money.

**In the demo:** The demo map runs three shops this way, one per currency — gold, silver and gems. See
[the demo map](demo-map.md#gold-silver-and-gems) for the items each sells and the configuration that draws all three
balances in the bag.

## Paying

```
PayExactCurrencyPrice(CurrencyTag, Price)     exact: N of ONE counter, no change
PayCurrencyPrice(PriceInBaseUnits)            the wallet: across denominations, gives change
```

These are different products and both are wanted.

**How the wallet form pays:**

- It spends the largest denomination first.
- When the small change runs out before the price is met, it breaks one unit of the smallest denomination that closes
  the gap, and returns the overpayment as change, again largest first.
- Nothing ever converts upward on its own: change appears only when a unit had to be broken.
- `CanPayCurrencyPrice` builds the same plan (`FSnapCurrencyPaymentPlan`) without paying, so a UI's "you can afford this"
  cannot disagree with the payment.
- On a refusal the plan carries `Result`, `PriceInBaseUnits` and `WalletValueInBaseUnits`, and `Spend` and `Change` are
  empty. The plan balances in value, not in units.

**The shop uses the exact form.** All five priced doors — vendor buy, a buy-back, a recipe's currency cost, repair and
upgrade — debit through it.

- That is what makes "the number on the button equals the number that leaves your purse" true.
- The wallet form would give change, and it takes no tag at all, so it could not express *debit gems, not coins*.
- It also consults `USnapInventorySettings::Currencies`, which ships **empty** as the wallet's off switch.
  - So routing a shop through it would refuse every purchase in a project that never configured a wallet.

`PayExactCurrencyPrice` checks affordability before debiting.

- That check is the whole reason it is a function rather than a call to `AddToCounter` with a negative delta.
- `AddToCounter` **clamps at zero** rather than refusing.
  - So an unchecked debit empties the purse, reports an ordinary new total, and under-pays silently with the goods
    already promised.

⚠ **A refund hands back the currency that was charged, not the one you are standing in front of.**

- A crafting queue entry captures its currency at enqueue (`FSnapCraftQueueEntry::SnapshotCurrencyTag`) beside its cost.
- That is because the shop is gone by the time anything is canceled.
  - Craft at the gem shop, walk away, cancel, and re-deriving would refund coins for a purchase made in gems.

## The icon beside a price

Every price the plugin draws — a shop row, a buyback row, the cost per unit, a sell line, the SELL button, an upgrade, a
craft cost, a trade offer — shows the icon of the currency it is in:

```
USnapEconomySettings::ResolveCurrencyIconPath(CurrencyTag)
```

1. The `Currencies` row for that counter, when it names an `Icon`.
2. Else `CounterIcon`, when the currency is the one the counter readout draws (`CounterTag`, or with `CounterTag` empty
   the project's `VendorCurrencyTag`).
3. Else no icon: the number stands alone.

So a gem shop's prices wear the gem, and a project with one currency and only `CounterIcon` set sees exactly what it
always did.

A sale that pays into two currencies (`120 + 8`) has no single icon, so the SELL button shows none.

⚠ A currency with no icon of its own is drawn bare rather than borrowing the readout's coin. Give each currency an `Icon`
in `Currencies`, and every price shows it.

⚠ The SELL button's glyph is added beside its total once, when the sell window is built, in the total's own box.

- That box must be a HorizontalBox (the shipped `WBP_ExampleVendorSell` keeps `SellConfirmTotal` in one,
  `SellConfirmTotalRow`).
- A total in any other box keeps a bare number, and the window logs which box it found.

## Changing what something costs

Three `BlueprintNativeEvent` hooks on `USnapVendorClientComponent`, each handed the number the plugin worked out. To
override one, make a subclass and attach it beside the main bag (a row in a container registry of your own that includes
`DA_SnapContainerRegistry`, as in [custom containers](custom-containers.md), or your PlayerState Blueprint); the plugin
uses yours instead of adding its own:

```
ResolveSellPrice(Instance, DefaultPrice)     what a shop pays for one unit
ResolveUpgradeCost(Instance, DefaultCost)    what an upgrade costs
ResolveRepairCost(Instance, DefaultCost)     what a repair costs
```

- Return `Default` to change nothing. Return something else to change it.
- The shipped formula is therefore the default *by construction*.
  - An un-overridden hook returns the same integer as before the hooks existed, including the spoilage and durability
    folds and the deliberate floor at 1.

⭐ **The number is passed in rather than computed inside the hook, deliberately.**

- A Blueprint override replaces the native body and has no parent-call node.
- So a hook that owned the formula would force every override that wanted one small adjustment to reproduce all of it.
- And the folds above are exactly what a reimplementation drops silently.

⚠ A returned value is clamped at 0. A negative price is a money printer:

- Negative sell charges the player to sell.
- A negative repair or upgrade **pays** them to use the service.

⚠ The hooks answer *how much*, never *in what*.

- Currency is `ResolveCurrencyTag`'s question and is resolved separately.
- So an override cannot return a number in one currency that the debit charges in another.

## An item's own worth

`USnapItemFragment_Value` carries `BaseValue` and `CurrencyTag` — what one unit of an item is intrinsically worth,
independent of any shop.

**Why it exists:**

- A price otherwise lives on a `FSnapVendorStockRow`, which makes it a fact about a **shop** rather than about an **item**.
  - So an item no shop stocks has no price anywhere.
- `GetSellPriceForDefinition` answers 0 for it.
  - That is the right answer to *what does this shop pay for a thing it does not deal in*.
  - And the wrong answer to *what is this worth*.
- This fragment answers the second question, and the junk merchant below is what reads it.

⚠ `BaseValue` defaults to **-1**, meaning "nobody has said", because 0 is a legal authored value meaning "worthless".

- A new property on a fragment inside an already-authored asset keeps its C++ initializer.
- So a default of 0 would silently declare every existing item worthless.
- And worthless is a perfectly legal designer decision, so the mistake would be indistinguishable from an intention.

**Selling at all is a separate question:**

- Whether an item may be sold at all stays on `USnapItemFragment_Restrictions::bCanSell`.
- Restrictions says **whether**; Value says **how much**.

## A shop that buys anything

Set `bBuysAnything` on a `USnapVendorStockDataAsset` and that shop will put a price on an item it does not stock — the junk
merchant.

```
price of one unit = max(1, floor(BaseValue x SellPriceFraction))
```

- That is the **same** arithmetic a stocked row gets, applied to the item's intrinsic worth instead of to the row's price.
- It is then routed through the ordinary per-instance folds.
  - So a worn sword still fetches less than a pristine one, and a stale loaf still fetches less than a fresh one.

| The item says | The junk merchant pays |
|---|---|
| A `Value` fragment with `BaseValue` 40 | `floor(40 x 0.25)` = **10** |
| A `Value` fragment with `BaseValue` 5, at fraction 0.10 | `floor(0.5)` = 0, raised to **1** by the `max(1, ...)` |
| A `Value` fragment with `BaseValue` 0 | Nothing — refused `NotSellable` |
| `BaseValue` left at -1, or no `Value` fragment at all | Nothing — refused `NotSellable` |

⚠ **"Nobody has authored a value" is a refusal, never a guess and never free.**

- Inventing a number would put a price on every unvalued item in a project at once.
- And any number it invented would be indistinguishable from an authored one.

⭐ It is **strictly below full value by construction**, because the fraction is clamped to at most 1 and the result is
rounded down.

- That is the same property that stops buy-then-sell being profitable at an ordinary shop.

⚠ `bBuysAnything` is **false by default**, so no existing vendor changes behavior. An unstocked item prices at 0 and the
sale is refused, exactly as it always was.

### Which currency a junk sale pays in

- A **stocked** item is denominated in the **shop's** money.
  - The row is the shop's offer, so the currency is the shop's business.
  - Nothing an item says can re-denominate a row in the list.
- An **unstocked** item bought under `bBuysAnything` is denominated by the **item**, resolving `Value.CurrencyTag` → the
  shop → the project global.
  - The shop is not selling it, so the shop has no offer to denominate.
  - What it has is a willingness to pay for something valued elsewhere.

```
GetSellCurrencyTagForInstance(Instance)     which counter this sale pays into
GetSellPriceForInstance(Instance)           how much
```

Both are thin forwarders onto one derivation, so they cannot disagree about whether an item is stocked. That is the
single fact that decides both answers.

### One sale, more than one counter

Once an item can be denominated by itself, a single multi-item sale can legitimately span two currencies: a stocked sword
paid in coins beside an unstocked relic paid in gems.

```
PlanSellBundle(Selection, OutSubjects, ...)          decides WHAT the selection sells
CalculateSellTotalsByCurrencyForPlan(Subjects, ...)  prices that plan — what the button shows
CalculateSellTotalsByCurrency(Instances, ...)        prices a RAW selection, WITHOUT planning
FormatSellTotalText(Totals)                          "120" for one currency, "120 + 8" for two
```

⚠⚠ **Quote the PLAN, not the raw selection.**

- Every screen in this plugin calls `PlanSellBundle` and then a `...ForPlan` form. A project wiring its own sell UI must
  do the same.
- The raw `CalculateSellTotalsByCurrency` does **not** plan.
  - So a single member this shop will not buy makes it answer `NotSellable` and **0 for the whole selection** — while
    `SellToVendor` on that same selection succeeds and credits the rest.
- **Measured:** An iron sword beside an unvalued item quotes `(0, [], NotSellable)` through the raw form and **sells for
  20**.
- A UI built on the raw form would disable a sale the player is entitled to.

**Each counter is credited its own members' worth.**

- The whole sale is still all-or-nothing: every counter it would touch is checked **before** the first item is removed.
  - So a sale that could not credit one of them moves nothing.

⚠ **All-or-nothing is a property of the PLANNED sale, not of your selection.**

- Membership is decided first, by `PlanSellBundle`.
  - An item this shop will not buy is **dropped from the sale** and stays in the bag while the rest sells.
- Selling an iron sword together with an unvalued item credits the sword, leaves the unvalued one where it was, and
  returns no refusal.
  - That is what makes a multi-select usable, since selecting a row of loot should not have the whole sale blocked by one
    worthless piece.
- A selection with nothing sellable left in it **is** refused, carrying that member's own reason.
  - So a single-item sale of something worthless still refuses `NotSellable` and moves nothing.

⚠⚠ **`CalculateSellTotal`'s single number is the arithmetic sum across every currency**, so for a mixed bundle it is true
of neither.

- It is unchanged for every caller that predates this — a single-currency bundle has one entry whose value equals it
  exactly.
- But **a UI showing a total must render the breakdown**, not that number.

## What a shop offers

Three flags on the stock asset, all **true by default** so no existing vendor changes behavior:

| Flag | Turns off |
|---|---|
| `bOffersRepair` | Paying this merchant to repair |
| `bOffersUpgrades` | Paying this merchant to raise an item's rarity |
| `bOffersCrafting` | This merchant counting as a crafting station — its station tags, and a timed craft ordered there waiting at the shop when it finishes while the player is away |

- Each one gates its server door **and** the describer the button's enabled state reads.
  - So a switched-off service shows as a disabled button rather than one that is refused when clicked.
- Both halves read the stock asset, which replicates to the owning client, so the two cannot answer differently.

A refusal is `ServiceNotOffered`, deliberately **not** `CraftingDisabled` or `UpgradesDisabled`.

- Those mean *this game does not have that feature* and tell a player to stop looking.
- This means *not here*, and the blacksmith two doors down still does it.

⛔ **Field repair is out of reach of all three, and must stay so.**

- `Server_FieldRepairItem` has no vendor session by design — it is the away-from-a-shop path, spending materials instead
  of money.
- So a merchant's flag reaching it would delete field repair for every project.

⚠ `bOffersCrafting` narrows the **vendor** case only.

- `bEnableVendorCrafting` remains the project-wide switch over every station-gated recipe, including a standalone forge's.
- This is one merchant's decision.
- A player standing at a forge with a non-crafting merchant's shop open still crafts at the forge.

## A shop's own sell fraction

`SellPriceFraction` on the stock asset overrides the project's. **Negative means inherit**, and it is the default.

```
USnapInventoryComponent::ResolveSellPriceFraction(VendorFraction)   vendor, else global
ResolveActiveSellPriceFraction()                                       with the open shop supplied
```

One rule with two levels, exactly like `CurrencyTag` — not a second rule.

⚠ **Negative is the sentinel, not zero.**

- Zero is a legal authored fraction meaning *this shop pays nothing* — a museum, a donation box.
- A default of 0 would have made every existing vendor pay nothing on upgrade, for the reason `VendorId` below gives.

⚠ The result is clamped to `[0, 1]` where it is read, rather than trusted to the editor's clamp.

- That is because a config layer bypasses an editor clamp entirely, and a fraction above 1 is an unbounded money printer.

## Naming a shelf

`VendorId` on the stock asset gives a shop's shelf a stable name, independent of where the asset lives.

| `VendorId` | Shelf is keyed by |
|---|---|
| Set | `id:<VendorId>` |
| Unset (the default) | `path:<the asset's path>` |

- So renaming or moving a stock asset no longer orphans the counts recorded against it.
- Two assets that deliberately share an id share one shelf — a summer and a winter stock list for one merchant, drawing
  down the same limited goods.

⚠⚠ **Unset must mean "key me by path", and that is why it is the default.**

- A new property on an already-authored asset keeps its C++ initializer.
- So every stock asset in every existing project arrives with no id.
- Reading that as a shared id would file every shop in the project onto **one shelf**, silently, on the first launch
  after an upgrade.

⚠ It is not per-actor identity.

- Two NPCs pointed at one asset are still one shop to the ledger, with or without an id — `BeginVendorSession`'s only
  argument is the asset.
- Two merchants with independent shelves still means two assets.

## Restocking

A limited row — `StockCount` above 0 — comes back after it is bought down, by one of three rules.

- The rule is set for the whole shop on the stock asset (`RestockMode`).
- For a row that differs, it is set on the row itself (`bOverrideRestockMode` + `RestockMode`).

| Mode | A row comes back | Its time |
|---|---|---|
| **Per Item** (the default) | `RestockPerCycle` units every cycle, up to its count | The row's `RestockSeconds` — **0 (the default) never restocks** |
| **When Sold Out** | All at once, to its full count, once it has sold out and waited | The row's `RestockSeconds`, counted from the purchase that sold it out |
| **Whole Shop** | With every other Whole Shop row, all to their full counts | The asset's `ShopRestockSeconds`, counted from when the shop was last seen full |

Two more fields per row:

| Field | Means |
|---|---|
| `RestockSeconds` | Per Item: how long one cycle takes. When Sold Out: how long after the row sold out it comes back. **0 (the default) never restocks** |
| `RestockPerCycle` | Per Item: how many units come back per completed cycle |

⭐ **Per Item is the original rule, unchanged**, and every stock asset authored before the modes existed loads it.

- So an upgrading project's shops restock exactly as they did.
- **To make one row never come back** in a shop whose own mode restocks, override that row to Per Item and leave its
  `RestockSeconds` at 0.

⭐ **One answer, three readers.**

- `USnapVendorStockDataAsset::ResolveRowRestockRule` decides a row's rule.
- The ledger restocks by it.
- `DescribeRestock` turns it into the vendor window's line: "Restocks every 60 s", "Sold-out items return after 30 s",
  "Restocks one at a time, every 20 s", or "Restocks over time" for a shop whose limited rows follow different rules.
- A shop whose limited rows never restock shows no line.

⭐ **A project-wide override.** `USnapEconomySettings::bOverrideVendorRestock` with `VendorRestockOverride` (a
mode, its time and, for Per Item, the units per cycle) replaces every limited row's rule in every shop. It is for:

- A game mode in which shops behave differently.
- Trying a rule without editing every stock asset.
- A demo (the demo's **VENDOR RESTOCK** pad switches it).

A time of 0 restocks nothing.

`IsDataValid` refuses a timed mode with no time:

- A Whole Shop asset whose `ShopRestockSeconds` is 0.
- A row that follows When Sold Out with a `RestockSeconds` of 0.
- A row overridden to Whole Shop when the asset's `ShopRestockSeconds` is 0.

Per Item with 0 is not one of them — it is "never restocks".

⭐ **It is resolved lazily, on read — there is no timer and no sweep.**

- Each row records when it last completed a cycle, and elapsed time becomes units the moment somebody opens the shop.
- A sweep would walk every shelf in the game to mutate counts nobody is looking at.

⚠ So a shop restocks **at open**, not while you stand in it. Close and reopen to collect one.

⭐ **The clock starts the moment a shelf exists** — the first time this player opens that shop, not the second.

- A shelf is created by the write that first records it, and its rows are stamped right there.
- So the first cycle after a player's first purchase counts exactly like every cycle after it.

**Limits of a restock:**

- A restock is clamped to the row's authored `StockCount` and **never reduces** a shelf.
  - A project that deliberately topped one up above its authored count keeps it.
- A row with `StockCount` 0 is unlimited and is skipped: nothing is ever subtracted from it, so there is nothing to put
  back.
- A **full** row resets its clock rather than banking time.
  - Otherwise a row that sat full for an hour would refill instantly the moment it was bought down.
- A **full shop** does the same for Whole Shop, whose clock runs only while one of its rows is short.

⭐ **When Sold Out is timed from the purchase that sold the row out**, not from the next open.

- The ledger stamps the moment the purchase writes the empty count.
- So a player who stays at the shop after buying the last one does not make the wait longer.

⚠ **A rule that changes starts again.**

- Each row remembers the rule its clock was started under, and the shop remembers its Whole Shop time.
- When the rule a row resolves to is different at the next open — the override switched, an asset edited while the game
  runs — that clock starts over then.
  - It is credited nothing for the time before, exactly as crossing a level boundary is (below).
- A stamp means different things under different rules.
  - Reading one rule's stamp as another's would refill a shop at once or far too late.

### Where the counts live

Remaining stock lives in `USnapVendorStockLedgerSubsystem`, a game instance subsystem.

- It survives closing and reopening a shop, and level travel.
- It resets when the game restarts: it is not saved. The `VendorStock` store id is reserved for a project that wants it
  saved.
- It runs on the server only; a client sees the counts through its own component. The stock asset is never written.
- Under `VendorStockScope` PerPlayer (the default) a shelf is keyed by the player's persistence profile
  (`ResolveProfileKey`), so a returning player keeps theirs while that key is stable — see the README's
  [Which profile a player's data belongs to](../../README.md#which-profile-a-players-data-belongs-to).
- Past `MaxTrackedVendorShelves` (8192) the least recently used shelf is dropped, and restocks. A departed player's shelf
  just ages out.
- **To restock from code:** Under PerPlayer use `SetRemainingForProfile`, since `SetRemaining` writes a key space the
  plugin does not read there. Under PerVendor, `SetRemaining` is right. `ForgetVendor` drops every shelf of a vendor.
- Change `VendorStockScope` only with no shop open.

### Crossing a level boundary

Restock is measured in **world time**, matching the choice spoilage already made and for its reason:

- A wall clock lets a player restock a shop by changing their system date.
- And it keeps running while the game is closed.

**The price:** World time restarts near zero on every map load, while the ledger survives them.

- So each shelf records which **travel epoch** its stamps were taken in.
- A mismatch **rebases and restocks by zero**.
  - Time spent in another level is not time this shop spent restocking.
  - Crediting it would make traveling the fastest way to refill a merchant.

⚠ An epoch, not a world name and not a pointer:

- A name repeats the moment a player returns to the same map — where the clock has restarted and the name has not.
- An object pointer can be reused.

## Buying back what you sold

A shop can remember the last few things this player sold to it, and sell them back **at the price they fetched**.

- **Depth** is `USnapEconomySettings::VendorBuybackSlots`.
  - **0, the default, switches the feature off**: Nothing is recorded.
  - A list recorded while it was on is neither drawn nor sold back (`DescribeRepurchase` refuses with
    `ServiceNotOffered`).
  - 8 is a good depth to turn it on with.
- The ring evicts oldest-first.
- A shop's "buy back" rows follow at once — a new sale's row appears, a bought-back one goes.
- **The list is the player's own in every stock scope.** With `VendorStockScope` set to `PerVendor`, the stock counts are
  shared, but what one player sold is never offered to another.

**Paying back exactly what was paid out is the only figure that is neither punitive nor profitable:**

- Charging the shelf price would make an accidental sale expensive.
- Charging less than the credit would make sell-then-rebuy a money printer.

⭐ **A repurchased item is the item that was sold.**

- A buyback slot holds a *record*:
  - An item definition, a count, a price, a counter.
  - And what the item was: its rarity and, for an item that does not stack, its wear, its freshness and its memories (its
    rolled modifiers among them), as a dropped item carries them to the ground.
- Buying back mints a new instance with all of that, so selling and buying back is neither a reroll nor a repair.
- Only its identity is new.
- A stacking item carries its rarity alone: its units join a stack you may already hold, which keeps its own state.
- A memory is carried only when its struct is native and holds no object reference (every memory this plugin defines
  does).
  - Another comes back without that memory.

**That a slot is a record, not an instance, is a design decision, not an oversight.** It is exactly what makes buyback
unable to duplicate anything.

- There is no instance parked anywhere that could also still be in a bag, in a save, or replicated to somebody.
  - So travel, save and load, eviction and `ForgetVendor` have nothing to leak.
- Keeping the original would mean holding it alive and owned by somebody for as long as the slot lasts.
  - That is the shape of every item-duplication bug ever written.

⚠ The counter travels with the record and is never re-derived. Otherwise a player could sell to a gem shop, walk to a
coin shop and buy back in coins what they were paid in gems — a currency converter with a profit margin.

```
GetVendorBuyback()                                            the open shop's slots, oldest first
DescribeRepurchase(Slot, Qty, OutUnitPrice, OutCurrencyTag)   may I, and what does it cost
RepurchaseFromVendor(Slot, Qty)                               do it
```

`DescribeRepurchase` asks the same questions `RepurchaseFromVendor` asks, in the same order, so an enabled button leads
to a purchase that is accepted.

## Trade has no currency policy

- Player-to-player trade has no shop and no recipe, so there is nothing more specific to inherit.
- It asks `GetDefaultCurrencyTag()` by name and always trades in the project's own money.
