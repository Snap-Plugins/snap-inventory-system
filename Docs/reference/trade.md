# Trade

Two players swap items and currency through an escrowed, double-confirmed session. Nothing moves until both sides
confirm, and the handover is atomic.

- **An actor can be the other side too:** A trader NPC whose offer is already on the table and confirmed. See
  [Trading with NPCs](npc-trade.md).

**On this page:**

- [The flow](#the-flow)
- [Escrow](#escrow)
- [Looking inside an offered container](#looking-inside-an-offered-container)
- [The handover](#the-handover)
- [Ending](#ending)
- [The screen](#the-screen)
- [Testing it](#testing-it)
- [See also](#see-also)

## The flow

```
Server_RequestTradeWith(Partner)                 A invites B
Server_RespondToTradeInvite(bAccept, Inviter)    B accepts or declines
Server_OfferItemsToTrade(Instances, PortionCount)
Server_OfferEquippedSlotToTrade(SlotTag)
Server_WithdrawItemsFromTrade(Instances)
Server_SetTradeCurrency(Amount)
Server_SetTradeConfirmed(bConfirmed)             both sides
Server_CancelTrade()
```

- `BeginTradeSession(Partner)` and `EndTradeSession(Reason)`, on the player's `USnapTradeComponent` (`Partner` is the
  other player's trade component), are the C++-side entry points.
- `OnTradeChanged` fires on **both** sides whenever anything about the open session changes.

**A dead player cannot trade.** A session is refused when either side's main bag answers
`USnapInventoryComponent::IsOwnerDead`.

- A project without a death tag overrides it — see [Death and loss](death-and-loss.md#saying-the-player-is-dead).

`PortionCount` lets a player offer part of a stack: pass `-1` for the whole thing.

## Escrow

An offered item is moved into escrow, not merely marked.

- That is what makes the session honest — an item in escrow cannot be sold, dropped, equipped or offered to a second
  trade at the same time.
- Changing an offer **clears both confirmations**. A player who confirmed, then watched the other side swap a sword for
  a stick, has to confirm again.

**An offered container is locked while it is on the table.**

- Nothing can be taken out of it, stored into it, dropped, sold or used from inside it, so its contents can't change
  after the partner looks and confirms.
- The log says "Withdraw the container from the offer first". Withdrawing the whole container is always allowed.
- Code of yours that adds a new way to change a container's contents must refuse while
  `USnapInventoryComponent::IsStorageItemInTradeEscrow` is true for it.

## Looking inside an offered container

A container can be offered with things in it, and the partner can open it to look before agreeing.

- A double-click or accept on its square opens a view-only window.
- What the partner sees is real: every item at every depth, with its durability and freshness.

A player's bag replicates only to its owner (see [Networking](networking.md)), so an offered container's contents reach
the partner as a **copy**. Nothing that is not on offer is copied.

- **When the copy exists:** It is made when the container goes on the table and removed when it leaves — withdrawn,
  canceled, traded, or when either player disconnects.
- **The wait:** The copy and the items it names arrive separately from the offer itself, so for a moment a client can
  see the box before it can see inside it.
  - **The trade screen never opens a container in that moment.**
  - It draws a spinner over the square and opens the window by itself, with the items, as soon as they arrive.
  - If nothing has arrived after `USnapInventoryTradeScreenWidget::PartnerContentsTimeoutSeconds` (five), the square says it
    couldn't load the contents; opening it again retries.
  - Withdrawing the item or ending the trade cancels the wait without opening anything.
- The server and the owning player read the container directly and never wait.
- A custom trade UI should ask `GetStorageContentsAvailability` before drawing a count or opening a window for the
  partner's container — see [Networking](networking.md#can-this-machine-see-inside-a-container).

## The handover

On the second confirmation the server moves everything at once: both item sets and both currency amounts, or nothing. A
failure part-way is not possible.

⚠ **An instance changes owner here**, and its outer changes with it:

```cpp
Instance->Rename(nullptr, ReceivingActor, REN_DontCreateRedirectors | REN_DoNotDirty);
```

Two things hang off an instance's outer — ownership tests, and its network GUID.

- An ownership test widened to accept a foreign outer makes the equip refusal go away.
- It also leaves the network GUID pointing at the previous owner, so the item becomes a permanent null on clients once
  that player disconnects.
- The rename is the fix; widening the test is not.

## Ending

The session ends on cancel, on completion, on either player disconnecting, and on range.

- Each end carries an `ESnapRefusal` saying which, and both sides are told.

⚠ Opening the bag or a shop can end a trade session as a side effect of its own teardown.

- If you add a screen that closes another, resolve everything that can fail **before** closing anything, so an early
  return changes nothing.

## The screen

`USnapInventoryTradeScreenWidget` shows both sides: an offer board each, a currency field, a confirm button and the
partner's state. It is keyboard- and gamepad-operable.

**Board size:**

- Each board has `MaxTradeOfferEntries` squares (**Project Settings > Plugins > Snap Inventory System - Economy > Trade**; 12 by default, a 4x3 board).
  - The server refuses an offer past it.
  - The setting is capped at 64, the most one offer RPC carries.
- The board grows a row per `TradeLayout` offer-column count until it reaches `TradeLayout`'s `OfferBandMaxHeight`
  (**418**, which is six rows at the shipped 64-unit square and 6-unit gap).
  - Past that it stops growing and scrolls, so a large cap makes a longer LIST rather than a taller SCREEN.
  - 64 at four columns is 16 rows of squares inside a six-row band.
- Because it is a height rather than a row count, changing `OfferSquareSize` or `OfferSlotGap` changes how many rows the
  same 418 holds.
  - The band always stops on a whole row, so the last visible one is scrolled to rather than cut in half.
- Both boards take the SAME height, the larger of the two, so the two CONFIRM bands stay on one line.
- Keyboard and pad focus scrolls the band to whatever it lands on.
- Server and client read the same setting and must agree.

**Your own column** (your equipment and bag, side by side) grows with its panels up to `TradeLayout`'s
`BagColumnMaxHeight` (**920**), then scrolls. 920 shows the shipped equipment panel whole and keeps the window on a
screen 1080 units tall.

**Offer gestures** (your own offer board):

- **Selecting:** A click selects a square. Ctrl+click toggles one, and Shift+click extends the range from the square
  clicked before.
- **Taking an item back:** Shift+click with nothing else selected, the keyboard's or pad's Activate on its square, or a
  drag back to your side.
  - The bag is a dragged offer item's one destination. A release on the bag, a worn slot, or anywhere else on the trade
    screen takes it back to the bag; a worn slot does not equip it, and it is never dropped on the floor.
  - Dropped on a bag square, it merges into a matching stack or swaps places with whatever is there. It does not go
    inside a pouch or apply as a repair kit, as a drop from the bag would: withdraw it first, then drop it again.
  - A full bag refuses it, even onto a matching stack, and the item stays in the offer.
- **Offering:** A stack dropped on your board from the bag with Shift held offers half, rounded down. With Ctrl held, the
  split window opens.
  - A drag from a pouch or from worn gear, and a multi-item drag, offer whole stacks.
  - A loot window's item can't be offered directly: take it into your bag first.
- **The coin block:** Over it the wheel steps the amount (Shift: ten). While + or - has focus, LB / RB step ten, Y
  offers the whole balance and X none (see [Amount windows](input.md#amount-windows)).
- The partner's confirm band reads WAITING until they confirm, then CONFIRMED.

⚠ The band's scrollbar is deliberately hidden, so **nothing on screen signals that more squares are below**.

- A scrollbar would narrow a board that is width-bound to exactly `OfferColumns` squares and clip the last column.
- The mouse wheel, a drag and focus navigation all scroll it.
- If your project wants an affordance, add one on a Widget Blueprint subclass.

**Changing its layout:** Its gaps, padding, offer square size and columns are its `TradeLayout` class default.

1. Edit them on a Widget Blueprint subclass of `USnapInventoryTradeScreenWidget` that adds no widgets of its own.
2. Name it in `TradeScreenClass`.

See [UI and theming](ui-and-theming.md#layout-on-the-widget-class).

- The shipped screen, `WBP_ExampleTradeScreen`, is that C++ screen copied into a Widget Blueprint you can restyle part
  by part ([Customizing the UI](customizing-the-ui.md)).
- Its window is laid out by its own widgets, so `TradeLayout` edits belong on a subclass of the C++ class. Two of
  them reach it as well: the offer band's height and the bag column's ceiling (`BagColumnMaxHeight`).

Invite prompts are modal, and a modal answers keys before anything else does — so a key that "does nothing" during a
pending invite is modality, not a dead binding.

## Testing it

Trade needs two players.

- Ordinary Play-in-Editor is a listen server, so the host's UI runs on authority and a whole class of client-only defect
  is invisible there.
- Use **2 players, listen server** and drive the non-hosting client.

⚠ This matters most for **looking inside an offered container**.

- The host reads every container directly, so a copy that never arrived still looks perfect from the host's seat.
- Have the HOST offer a loaded container and open it on the CLIENT.

⚠ Any predicate both sides call must not read `NotReplicated` state.

- On a client such a field arrives default-constructed, so the comparison fails identically every time.
- And a UI gate built on it disables a perfectly legal action with no explanation.
- Gate unreplicated terms on `HasAuthority()` and let the client's answer be the permissive one.

## See also

- [Trading with NPCs](npc-trade.md)
- [Networking](networking.md)
- [Refusals](refusals.md)
- [Appendix: settings](appendix-settings.md) — the **Trade** category
