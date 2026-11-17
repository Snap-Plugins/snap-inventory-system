# Trading with NPCs

A trade partner is any actor — a trader NPC, a market stall, a talking statue — that a player trades with in the
plugin's own trade window, exactly as they trade with another player. Nothing here needs code.

- **The prompt:** Walking up to it offers **Trade with <its name>**.
- **The window:** It opens with the partner's offer already on the table.
- **The partner's side:** The partner confirms once the player puts something down.

## Trading with NPCs in two steps

1. **Add a Snap Trade Partner component** (`USnapTradePartnerComponent`) to an actor that **replicates** (`bReplicates`
   on).
   - The player's window draws the partner's side from replicated state.
   - The component is an interactable, so the actor needs no other interaction component.
2. **Fill in `Offer`** — The items it puts on the table in every trade, each with a count.
   - Place the actor.

The example is `BP_Example_TraderNPC` — the Trader:

- It offers one Health Potion in every trade.
- It takes anything you put down (but not nothing).
- So a player can try the trade window with nobody else around.

## What a player sees

- **The prompt.** "Trade with Trader" (`TradePromptText`, where `{Name}` is `DisplayName`).
  - `DisplayName` is also the window's title: "TRADE - Trader".
- **The partner waits for your side.** Its offer is on the table.
  - While your side is empty it stays unconfirmed.
  - It confirms `ReconfirmDelaySeconds` after you put something down.
- **Changing your offer.** Any change to either offer clears both confirmations — the rule that protects every trade.
  - The partner confirms again after `ReconfirmDelaySeconds` (0.5 by default), so the player always confirms last.
  - CONFIRM completes the trade.
  - CANCEL, the window's close box or walking out of range moves nothing.
- **Refusals are the ordinary ones.**
  - An item that cannot be traded is refused as it is in a trade between players ("That item cannot be traded.").
  - A partner trading with someone else tells the next player "They are busy right now."
  - A partner trades with **one player at a time**.
  - See [Refusals](refusals.md).

## How it works

The partner trades from a **ledger** — an ordinary inventory component on its own actor.

- When the actor has none, the component makes one at `BeginPlay`: a `USnapTradeLedgerComponent` (replicated to
  everyone, with no carry weight, `LedgerSlots` squares).
- An actor that brings its own inventory component trades from that one.
  - Add a `USnapTradeLedgerComponent` to set its size.
  - A plain inventory component on an actor no player owns must have `ReplicationScope` **Everyone**.
- The trade itself is an ordinary trade session between the player's bag and that ledger
  (`USnapTradeComponent::BeginTradeSession`).
  - So escrow, room, weight, restrictions, the double confirm, the commit, the range rule and the window are the
    [player-to-player ones](trade.md).

Step by step:

1. The "Trade" option sends `USnapTradeComponent::Server_TradeWithPartner` from the player's machine.
   - The server checks reach the way it does for a pickup and calls `USnapTradePartnerComponent::BeginTradeFor`.
2. The ledger is topped up to every `Offer` line's count, and the session opens.
   - An offer that came back from a canceled trade is offered again, never doubled.
   - The offer goes on the table and the partner confirms if `AcceptsOffer` says yes (by default it says no while the
     player's side is empty).
3. While the session is open the partner re-confirms after each change, if `AcceptsOffer` says yes.
4. When the trade completes, what the player gave lands in the ledger through the commit and is then **consumed** —
   removed right after the trade.
   - So a partner that gives freely never fills up.
   - Turn on `bKeepReceivedItems` to keep it instead. A full ledger then refuses further trades
     ("Their inventory is full."), as a full bag would.
   - Currency the player adds goes to the ledger's counter.

For scripts:

- `OnTradeCompleted` fires on the server after each completed trade.
- `BeginTradeFor` may be called from a server-side script.

## Asking for something in return

`AcceptsOffer(Player, OfferedItems, OfferedCurrency)` is a Blueprint Native Event asked before every confirm the partner
makes.

- By default it answers **yes to any item or currency, and no while the player's side is empty** — nothing is given for
  nothing.
- Override it to ask for a particular price, or to say yes to an empty side for a partner that should give freely (a
  gift-giving NPC).
- While it answers no, the partner does not confirm and the trade cannot complete.

## Settings

On the component, under **Trade Partner**:

| Setting | Default | What it does |
|---|---|---|
| `DisplayName` | Trader | The window's title and the prompt's `{Name}` |
| `TradePromptText` | Trade with {Name} | The prompt |
| `Offer` | Empty | What the partner puts on the table in every trade |
| `bAutoConfirm` | On | The partner confirms by itself |
| `ReconfirmDelaySeconds` | 0.5 | How long after a change it confirms again (0 = on the next frame) |
| `bKeepReceivedItems` | Off | Keep what players give instead of consuming it |
| `LedgerSlots` | 40 | Squares of the ledger the component makes (ignored when the actor brings its own) |

## See also

- [Trade](trade.md) — the session, escrow and the window
- [Loot and interaction](loot-and-interaction.md#a-subclasss-own-options) — the extra-option seam this component uses
- [Quest givers](quest-givers.md) — the other shipped user of that seam
