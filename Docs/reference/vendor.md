# Vendor

A shop is a **stock asset** plus a **session**. The session is server-held and range-watched, so walking away closes it.

**On this page:**

- [Stock](#stock)
- [Opening a shop](#opening-a-shop)
- [Buying](#buying)
- [Selling](#selling)
- [Upgrades](#upgrades)
- [Repair](#repair)
- [Crafting at a shop](#crafting-at-a-shop)
- [Currency](#currency)
- [Refusals](#refusals)
- [See also](#see-also)

## Stock

`USnapVendorStockDataAsset` lists rows: a definition, a buy price, and a quantity.

- A quantity of 0 means unlimited.
- The demo's example is `/SnapInventorySystem/Examples/Items/DA_ExampleVendorStock`.
- `DebugVendorStock` ships empty (the example stock is demo content).

Remaining stock is tracked by a **ledger subsystem**, not by the asset, so selling out never edits content.

A limited row comes back one of three ways — the shop's `RestockMode`, or the row's own:

| Mode | How the row comes back |
|---|---|
| **Per Item** | Each row on its own `RestockSeconds` timer — 0, the default, never restocks |
| **When Sold Out** | A row that sells out is back to its full count `RestockSeconds` later |
| **Whole Shop** | `ShopRestockSeconds` after the shop was last full, everything is back |

- A restock is credited when the shop is next opened.
- The `bOverrideVendorRestock` setting replaces every shop's rule project-wide.
- The vendor window says the rule in one line under the shop's name (`RestockLineWidgetName`).
  - `WBP_ExampleVendorScreen` carries it as `VendorRestockLine`; a window without one gets it built there.
- See [Currency and prices](currency-and-prices.md#restocking).

## Opening a shop

The shop functions on this page are on the buyer's `USnapVendorClientComponent`, beside the main bag on the
PlayerState; the bag keeps most `Server_` doors, which forward to it (`Server_UpgradeEquippedRarity`,
`Server_RepairEquippedSlot` and `Server_SalvageByRarity` are the component's alone).
`USnapVendorClientComponent::OnVendorClientChanged` fires on a client each time the session replicates; on the server
only after a sale that writes buyback rows, a shared-stock purchase by another player or an upgrade.

```
BeginVendorSession(Stock)        Server_BeginVendorSession(Stock)        -- asset form, see below
                                 Server_BeginVendorSessionAt(Vendor)     -- a vendor in the world
EndVendorSession()               Server_EndVendorSession()
```

A shop opened from a world vendor goes through `Server_BeginVendorSessionAt`:

- The client names the interactable it is standing at.
- The server checks that its verb is `OpenVendor`, that it is switched on and that the caller is in reach before opening
  the stock list it reads off the component.
- `USnapInventoryHUDComponent::OpenVendor` does this for you whenever the focused interactable sells that stock.

⚠ The asset form, `Server_BeginVendorSession(Stock)`, is **refused unless `bAllowClientVendorSession` is on**, because
it lets a client choose any stock list it can reference.

- That includes the debug vendor key and a menu button that opens a shop with no vendor in focus.
  - `OpenVendor` refuses those up front, before closing the bag.
- Closing a shop (a null stock) is always allowed.

The session lives on the buyer's `USnapVendorClientComponent`, which sits beside the main bag on the PlayerState.

- Every purchase re-checks it, so a purchase sent after the session ended is refused with `NoSession`.

## Buying

```
BuyFromVendor(RowIndex, Quantity)    -> ESnapRefusal
Server_BuyFromVendor(RowIndex, Quantity)
GetVendorRemainingStock(RowIndex)
```

The server checks the session, the row, the stock, the price against the buyer's counter, and whether the bag has room —
then debits, mints and decrements in one step.

⚠ Because `AddItem` clamps to capacity, a purchase whose items do not all fit is handled explicitly rather than by
trusting a non-null return.

**Row colors and tags:**

- A row's name is drawn in the color of what a purchase would MINT:
  - A fixed tier's color.
  - Or white with "Random rarity" in its subtitle when the tier is rolled when bought. The buy panel then lists each
    tier's odds, from the rarity table's weights, in that tier's color.
- An item that rolls magic modifiers carries a blue "Magic" tag.
  - The buy panel says how many and how likely ("Magic: 60% chance to roll 1-2 magic modifiers").
- See `USnapInventoryVendorScreenWidget::ResolveShopNameColorForDefinition` and `DoesDefinitionRollMagic`.

**Badges and buttons:**

- A finite row's badge shows what is left.
- An **unlimited** row of a stackable item shows how many would fit in the bag right now.
  - That is the same number the buy panel's **BUY STACK (n)** offers
    (`USnapInventoryVendorBuyPanelWidget::ResolveBuyCeiling`).
- BUY is priced at the dialed amount and BUY STACK at all n.
  - Both go through `USnapInventoryVendorScreenWidget::CanAffordVendorTotal`, the check `RequestBuy` refuses with before
    anything is sent.
  - A button the purse cannot cover is drawn gray and cannot be pressed.
- **A capacity upgrade the bag already has, at that tier or a better one, is not sold.** Its row is dimmed, BUY is gray,
  and the server refuses the sale. See [Carry weight](condition-and-weight.md#carry-weight).

## Selling

Three ways to sell while a shop is open:

- **Click an item, then press SELL.**
  - The middle column switches to SELL, describes the item and its price and lets you dial a stack down.
  - Its SELL button commits.
- **Double-click or Shift+click an item** in your bag — or in an open pouch window — to sell that item alone, its whole
  stack, at once (never the selection it belongs to).
  - A container is never sold this way: a double-click opens it and a Shift+click leaves it alone.
  - An item this shop does not buy is refused by name, never used or equipped instead.
  - On the shop's Repair / Upgrade tab both clicks keep their ordinary meaning.
  - The item's card lists them: "Double-click -- Sell" and "Shift+Click -- Sell".
- **The item menu's Sell** (right-click) sells what the menu names — the whole selection when the item is part of one —
  at once.

`USnapEconomySettings::bConfirmVendorSales` (**off** by default) makes the last two ask first:

- They open the sell window on what was named.
- The menu's Sell opens it with the window's top-left exactly where the menu's top-left was.
- With it off, and with buyback off as well (`VendorBuybackSlots`, also off by default), a sale is final.
  - The sale's own refusals (a locked item is never sold) are the only guard.
- On a gamepad the menu's Sell still asks while it is off (`bConfirmPadItemLoss`).

```
USnapInventoryVendorScreenWidget::RequestSellItems(Items)        sells exactly these, whole stacks, at once
USnapInventoryVendorScreenWidget::RequestSaleOf(Items)           the same -- asking first with bConfirmVendorSales on
USnapInventorySlotWidget::TrySellToOpenShop(bClaimContainer)     a bag square's double-click / Shift+click sale
USnapInventoryStorageSlotWidget::TrySellToOpenShop(bClaimContainer)   the same, for a pouch square
```

```
GetSellPriceForDefinition(Definition)     GetSellPriceForInstance(Instance)
GetVendorUnitSellPrice(BuyPrice)          static, the shipped formula
DescribeSellCandidate(Instance, OutReason)
CalculateSellTotal(Instances, UnitsForSingle, OutReason)
PlanSellBundle(...)
SellToVendor(Instances, UnitsForSingle)   Server_SellToVendor(...)
```

`PlanSellBundle` is the one place that decides what a selection actually sells, so the price on the button and the price
the server charges cannot disagree.

- A selection can include a pouch, and a pouch sells with its contents.
- A pouch cannot leave the grid on its own, so selling it together with what it holds is the only honest reading.

`UnitsForSingle` is how many units of a single selected stack to sell; it is resolved once, for a one-item gesture.

**The item's card says what it sells for.** While a shop is open, an item's tooltip shows a price block in its header's
right column, under the weight and status lines:

- "Sells for [coin] 50".
  - "(10 each)" after a stack's total.
  - A pouch priced with its contents.
  - Two currencies as "120 + 8" with no coin.
- "Repair [coin] 35" when this shop would repair it (its `bOffersRepair`, and the item is damaged).
- "Not bought here" (muted) when the shop does not buy it.

How the block is filled:

- The card asks `PlanSellBundle` + `CalculateSellTotalsByCurrencyForPlan` and `DescribeItemRepair` — the SELL and REPAIR
  buttons' own answers — twice a second while it is shown, so its numbers are the buttons'.
- A worn item, which the sale cannot reach until it is in the bag, shows the price the shop pays for it.
- A learned item (kept by its container) shows none.
- With no shop open the block is collapsed.
- A tooltip Widget Blueprint needs no change: the block is built beside its status text.

## Upgrades

A shop upgrades an item in one of two ways:

- **An item with a `Rarity` fragment** (and no `Upgradeable` one) is traded up a rarity tier.
- **An item with the `Upgradeable` fragment** becomes a different item (`UpgradesInto`), and never takes the rarity-tier
  route.
  - Its `CurrencyCost` is the price: -1 (the default) prices it as a rarity upgrade, 0 makes it free.
  - It makes a new instance, so durability and spoilage start fresh.
  - It must be a bag item: validation refuses the fragment on an equippable or storage item.

```
DescribeItemUpgrade(Instance, OutCurrentRarity, ...)   -> ESnapRefusal
UpgradeItemRarity(Instance)          Server_UpgradeItemRarity(Instance)
DescribeEquipmentUpgrade(Slot, OutInstance, ...)
UpgradeEquippedRarity(Slot)          Server_UpgradeEquippedRarity(Slot)
ResolveUpgradeCurrencyCost(CurrentRank)   static
```

`Describe*` and the mutator return the same `ESnapRefusal`, so the UI can gray a button for exactly the reason the
server would refuse.

**An upgrade into a different item asks whether the bag takes the new item.** This is an item whose `Upgradeable`
fragment names `UpgradesInto`. `DescribeItemUpgrade` asks it for an item in the bag itself (not worn, not in a pouch),
with the old item's square counted free:

- The new item at its carry limit: `UnitCapReached`, "<Item> -- You already hold as many of those as you are allowed."
- A new item that belongs in a different container: `WrongContainer`, "<Item> -- That belongs in a different container -- making room here will not help."
- No room for the new item's shape: `InventoryFull`, said "Your inventory is full." in the player's own bag.
- The item named first is the item the upgrade would give.

Where the answer shows:

- **The item menu's Upgrade row** is drawn dimmed with that sentence, before the click.
- **The shop's Repair / Upgrade tab** keeps such a bag item selected and shows the same sentence. Its UPGRADE button
  is grayed.
- **The UPGRADE? confirmation:** If the upgrade stops being possible while it is open, pressing UPGRADE raises a notice
  with the same sentence and sends nothing.
  - An item that is gone by then is only logged.
- **The server** refuses before anything is charged, with the same sentence.
  - A project's admission policy is the exception. A client's preview never asks it, and no preview can ask it about
    the new item itself (that item does not exist yet). Such a refusal arrives only as the notice.
- **Only four codes are shown before the click:** `WrongContainer`, `RefusedByPolicy`, `InventoryFull` and
  `UnitCapReached` (`USnapVendorClientComponent::CanShowUpgradeRefusal`).
  - Any other code a project's admission policy returns leaves the row out, and the tab offers no UPGRADE. The
    server's notice still says it.

On the shop's Repair / Upgrade tab each worn slot is a row.

- Under its price it shows "Upgrade [coin] 120" and "Repair [coin] 35", each only while that action is on offer.
  - They are `USnapInventoryVendorUpgradeRowWidget`'s cost-line parts, built when the Widget Blueprint has none.
- A row is **selectable** — by a click, the accept key or a pad's A, and by the tab's own first pick — only when an
  offered upgrade or repair is one the purse covers (`IsSelectable`).
- Otherwise it is dimmed and its tier line says "Not enough Gold" (the shop's currency by its Display Name,
  `NotEnoughCurrencyFormat`).
- In a shop that doesn't upgrade (its stock's `bOffersUpgrades` is off), a worn item's tier line says "This trader does
  not offer that." (`ServiceNotOfferedLabel`).
  - The middle column says it too, while nothing is selected (`UpgradeServiceNotOfferedLabel`).

⚠ An upgrade **destroys the old instance and creates a new one**.

- So anything holding a pointer to the old one — a hotbar binding, a trade escrow entry — is re-resolved by identity.

## Repair

```
DescribeItemRepair(Instance, ...)      RepairItem(Instance)
DescribeEquipmentRepair(Slot, ...)     RepairEquippedSlot(Slot)
```

Three routes exist:

- Paying a shop.
- Spending a `RepairKit` item.
  - The kit route works from the bag or a pouch, onto an item in the bag, in a pouch or worn — see
    [condition and weight](condition-and-weight.md).
- Away from a shop, spending `FieldRepairMaterialItem` (off until that setting names a material).

## Crafting at a shop

The shop screen carries a CRAFT tab.

- A shop whose stock carries `StationTags` (or its own `RecipeSet`) is a crafting station: its tab lists only its own
  recipes.
- A shop whose tags match no recipe shows no tab.
- See [crafting](crafting.md) and [What a station lists](crafting-stations.md#what-a-station-lists).

## Currency

Prices are paid from a counter, named by `VendorCurrencyTag` (default `SnapInventorySystem.Counter.Currency`).

- `USnapWalletComponent::CanPayExactCurrencyPrice` and `PayExactCurrencyPrice` are the shops' debit path: exactly the
  price, from that one counter.
- `CanPayCurrencyPrice` / `PayCurrencyPrice` pay a base-unit price from the whole wallet. The bag's
  `Server_PayCurrencyPrice` is `PayCurrencyPrice`'s RPC, refused unless the bag's `bAllowClientPricedPayment` is on (off
  by default: purchases, crafts, repairs and upgrades price themselves on the server).

⚠ If your project has its own currency, set **both** `CounterTag` and `VendorCurrencyTag`.

- Setting only one gives you a shop that refuses every purchase while the bag fills correctly, and the symptom names
  nothing.

## Refusals

Every entry point returns `ESnapRefusal`, and `Client_ReportVendorRefusal` carries a server-side one back to the buying
client. See [refusals](refusals.md).

- It also carries the item the refusal is about (`Subject`), when there is one. Only an upgrade names one: the item the
  upgrade would give.
- `OnVendorRefusal`, `ISnapRefusalReporter::SnapShowVendorRefusal` and `USnapInventoryHUDComponent::ReportVendorRefusalToPlayer`
  receive it too.
- The notice then reads "<Item> -- <sentence>". Every other shop refusal names no item.

**The player hears and sees every one.** `USnapInventoryVendorScreenWidget::ReportVendorRefusal` is where every shop refusal
arrives, the server's and the client's own. It:

- Logs it.
- Raises `OnVendorRefusal`.
- Sends it through the plugin's standard refusal channel: the refused sound and the plugin's notice, or your own
  `OnActionRefused` consumer instead of the notice.

Details:

- A vendor screen Blueprint that implements `OnVendorRefusal` owns what the player is shown (the notice is not raised;
  the sound still plays).
- `NothingToDo` plays the sound alone.
- A BUY the purse cannot pay for is refused at the press — before the Buy sound, with no request sent.
- The server still decides every sale.

## See also

- [Trade](trade.md) — the player-to-player equivalent
- [Appendix: settings](appendix-settings.md) — the **Vendor** category
