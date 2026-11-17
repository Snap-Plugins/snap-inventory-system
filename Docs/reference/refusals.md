# Refusals

When the plugin declines to do something, it says which thing and why, in one vocabulary.

**On this page:**

- [The vocabulary](#the-vocabulary)
- [Reaching the player](#reaching-the-player)
- [Which doors report](#which-doors-report)
- [Describe, then do](#describe-then-do)
- [Reading one in Blueprint](#reading-one-in-blueprint)
- [See also](#see-also)

## The vocabulary

```cpp
struct FSnapRefusal
{
    ESnapRefusalDomain Domain;     // Inventory, Equipment, Hotbar, Vendor, Trade, Crafting, Currency
    ESnapRefusal       Code;       // one shared enum, `None` plus 87 reasons
    const USnapItemDefinition* Subject;  // the item it is about, where there is one
    int32              Requested;  // NotEnoughLeft and NotAllFit: how many were asked for
    int32              Applied;    // NotEnoughLeft and NotAllFit: how many moved
    bool               bQuiet;     // no notice: the player was already shown it (still logged and broadcast)
    FGameplayTag       ReasonTag;  // a feature's own reason, said instead of the code's sentence when registered
};
```

A feature can add a reason without a new code: it raises the nearest code with its own `ReasonTag` and registers the
tag's sentence once with `USnapRefusalRegistry::RegisterReason`.

One enum rather than one per subsystem: a refusal can be logged, localized, matched and forwarded without every layer
knowing which domain raised it.

```
FSnapRefusal::Allowed(Domain)          the "no refusal" value
FSnapRefusal::Shortfall(Domain, Requested, Before, After)   a request that ran short, or Allowed
IsRefusal(Refusal)   IsOutcome(Refusal)
GetRefusalText(Refusal)                a localized sentence
GetRefusalTitle(Refusal)               a localized heading
```

- Every code carries a real sentence through `NSLOCTEXT`.
- `Subject` is unset for a refusal with no single subject (`NoSession`, `NoCurrenciesConfigured`), and `GetRefusalText`
  must read correctly either way.

⚠ **New values are appended, never inserted.**

- The enum is `BlueprintType` and travels over the wire, so its numbers are baked into consuming projects' assets and
  into replicated payloads.
- Grouping a new value with its thematic neighbors would renumber everything after it.
- A value therefore sits at the end whatever it is about, which is why the last block does not read as one subject.

A test walks every enumerator and asserts each has a non-empty sentence that is gatherable in the plugin's namespace.

- Nothing on this toolchain warns about an enumerator missing from a switch.
- So a new code with no sentence would otherwise ship as a notice with a title and no words — the silent refusal the
  whole channel exists to end.

⚠ **Refusal and outcome are different.**

- Some values are a *result*, not a *denial* — a trade screen wants to say "the other player declined" rather than
  silently doing nothing.
- `IsRefusal` and `IsOutcome` separate the two, and folding them would break a fail-closed invariant.

## Reaching the player

Three components carry the channel:

```
ReportRefusal(Refusal)              raise it
Client_ReportRefusal(Refusal)       carry it to the owning client
OnActionRefused                     the BlueprintAssignable a widget binds
```

It is on `USnapInventoryComponent`, `USnapEquipmentComponent` and `USnapHotbarComponent`.

The default consumer is a **registered handler**: `USnapInventoryHUDComponent::RaiseDefaultRefusalNotice`, which hands the
refusal to the notice service (`USnapNoticeService::RaiseDefaultRefusalNotice`, in `UICore/SnapNoticeService.h`); the
service shows the sentence (`USnapNoticeService::ShowPlayerNotice`).

- **Notice Style** (`NoticeStyle`, **Project Settings > Plugins > Snap Inventory System - Inventory UI > UI**) decides how it shows:
  - **Prompt**, the default: the sentence as a white line a little above the middle of the player's part of the screen,
    over every window. It stays about 3 seconds (`NoticePromptSeconds`, a second more for each further line), then
    fades. It takes no focus, cursor or input, so play goes on, and a new one replaces it.
  - **Window**: `USnapInventoryConfirmWindowWidget::ShowNotice`, the domain's title, the sentence and an OK button.
- The questions that act at once stay windows in both styles: DROP INVENTORY?, DISMANTLE?, REPAIR?, UPGRADE?, a
  controller's SELL? / DROP? / SELL JUNK? and a trade request.
- THESE WILL NOT COMBINE and the SELL JUNK reports go through `ShowPlayerNotice` too.

- It installs itself from its constructor through `USnapRefusalLibrary::SetDefaultNoticeHandler`.
- So `USnapRefusalLibrary` never names a widget type and the dependency points one way.

⭐ **It is a handler, not a second listener on `OnActionRefused`, and that is the design.**

- A default listener would make `IsBound()` true forever.
- So a project that bound its own consumer would get its handler *and* the plugin's popup — the opposite of
  "I want my own refusal UI".
- So a project replacing the refusal UI binds `OnActionRefused` and wins outright, exactly as before.

⚠ The default notice returns early when the controller is not local (`USnapNoticeService::RaiseDefaultRefusalNotice`).

- A refusal raised on the server for a **remote** client must not try to show that server a window.
- On a listen server with two players that produced two errors per refusal, for a window that could never appear.

⭐ **A quiet refusal raises no notice.** `FSnapRefusal::bQuiet` marks a refusal the player has already been shown.

- It is logged and still reaches a project's own `OnActionRefused` consumer (which can read the flag), but the default
  popup stays closed.
- A grid placement that does not fit — `TargetOccupied` or `NoSuchSlot` from a move in a `Spatial` container
  (`IsQuietGridPlacementRefusal`) — is the one quiet refusal:
  - The drag painted the cells red.
  - A release of a grid's own item that the drag painted red — in the bag's grid, or a pouch's own item in its window —
    is not sent at all.
  - One that still reaches the server means the player's view was a moment stale.

## Which doors report

**The list is the table in `SnapInventory.Refusals.EveryWiredDoorReachesTheChannel`**, one row per door, asserting that
each delivers its own code exactly once.

- It is there rather than here because a list in prose cannot fail: a door added without a report used to join the
  silent set with nothing to notice it.
- The sections below say *why* each door reports what it does, which a table cannot.

⚠ **That test catches a dropped, renamed or un-reported door — not a newly ADDED one.**

- A door added in a later release with no report will not fail it.
- So a door found silent later is evidence the table needs a row, never that the table was wrong.

⚠ **And it asserts channel DELIVERY, not that a notice appeared.**

- `DispatchRefusal` returns as soon as it has broadcast `OnActionRefused` — the "a project's own consumer wins outright"
  rule below.
- So a test listener short-circuits the notice path by design.

Two headless tests cover the handler mechanism either side of that:

- `SnapInventory.Refusals.TheHudInstallsTheDefaultNoticeHandler` asserts one was installed at all.
- `SnapInventory.Refusals.AProjectConsumerWinsOverTheDefaultNotice` asserts a bound consumer fires once and the notice
  does **not** also fire.
- ⚠ Neither asserts that a notice APPEARED.
  - The white line and `USnapInventoryConfirmWindowWidget::ShowNotice` both need a local player controller and a viewport,
    which a test world has neither of.
  - So the notice itself stays covered by manual passes.

**A request that ran short reports what moved.** Nine doors that move a count measure the units held at their source
before and after, and pass both to `FSnapRefusal::Shortfall`:

- `NotEnoughLeft`, with `Requested` and `Applied`, when the source could not cover the request and all of it moved.
- Nothing otherwise: a merge a full stack bounded, a refusal for another reason and an exact request all stay silent.
- On the inventory they are `Server_DropCounterToWorld`, `Server_MergeStacks`, `Server_DropItemStacksToWorld`,
  `Server_TransferStoredToInventory` and `Server_StoreItemPortionInStorage`.
- On the equipment, `Server_TransferEquippedToInventory`, `Server_TransferInventoryToEquipped`,
  `Server_MergeEquippedStacks` and `Server_DropEquippedStackToWorld`.
- The clamp itself is unchanged: what was there still moves.

**A portion its stack no longer covers is refused out loud.** A portion onto an empty destination must leave something
behind.

- So `Server_TransferStoredToInventory`, `Server_StoreItemPortionInStorage`, `Server_TransferEquippedToInventory`,
  `Server_TransferInventoryToEquipped` and `Server_SplitEquippedStack` refuse one that is the whole stack or more, and
  report `TooMany` — what `Server_SplitStack` says for the same rule.
- The client never asks for that, so it happens only when the stack shrank inside the round trip.
- Nothing moves, and the door reports this instead of running the shortfall rule.

**A request that has not come back yet is refused on the client.** The stack-split window's currency Confirm raises
`RequestInFlight` when `USnapWalletComponent::TryBeginCounterDropRequest` finds an earlier request that has not
landed.

- The latch clears only when the balance falls by at least the amount requested, or when its grace window ends.
  - So income or a smaller spend arriving in between does not reopen it.
- It never crosses the network.
- The same window raises `NotEnoughLeft` itself, on the client, when its source shrank while it was open and its clamp
  moves less than was chosen.
- It raises `NotAllFit` when the destination bounded the clamp instead — the rest is still in the source.
- The server's drag doors stay silent for that shape (the documented top-up), where nobody chose an amount.

**A merge the server refuses says why.** `Server_MergeStacks` reports each refusal
`USnapInventoryComponent::TryMergeStacks` names:

- `NoLongerThere` for a target stack that has gone (with no subject — the dragged stack is still there).
- `TargetOccupied` for another item or another tier.
- `SpoilageMismatch`, `StorageNotEmpty` and `StackFull`.

The shortfall wins where it applies, so a source that has gone still says `NotEnoughLeft`. A stack released onto itself
is the silent no-op it always was.

**A container used only where it is kept says so.** Every door the reach rule guards (a container with
`bRequireAccessSession` on — see
[custom containers](custom-containers.md#a-container-used-only-where-it-is-kept-the-reach-rule)) reports
`NotAtContainer` once, on the container that refused, with no subject.

- The item a deposit names is in the bag and reachable, and it is the container the player is away from.
- Nothing moves.
- The server log names the door and why ("no access session here", or the session it found had ended — out of range, a
  new pawn, the player gone).

**Item marks switched off say so.** With `bEnableItemMarks` off, a request to set marks (`Server_SetItemMarks` — the
item menu's Lock, Favorite and Junk rows, or a project's own UI) changes nothing and reports `MarksTurnedOff` once, with
no subject. See [Restrictions and marks](restrictions-and-marks.md#switching-marks-off).

**A selection too large for one request is refused on the client.** Four doors take a whole selection in one request and
cannot split it: a group move, a store into a pouch, a take out of one and a re-slot inside one.

- Past `FSnapNetLimits::MaxInstancesPerRpc` (256) items — where the server's validation would refuse the request and
  close the connection — the client raises `TooMany` and sends nothing
  (`USnapInventoryComponent::ReportMultiItemRequestRefusal`, public for a project's own door).

**Other doors:**

- `Server_MoveItem`, `Server_SplitStack` and `Server_DropItemsToWorld` report.
- Vendor and trade have their own older channels (`Client_ReportVendorRefusal`, `Client_ReportTradeRefusal`) which carry
  extra payload and are deliberately not folded in yet.
  - The vendor channel carries the item a refusal is about (`Subject`). Only an upgrade names one. See
    [Vendor](vendor.md#refusals).
- `Server_BeginStationSession` reports through a leg of its own, `Client_RefuseStationSession`, because it has a second
  thing to do.
  - The craft window was opened locally before the answer could arrive, so the refusal closes it as well as explaining
    itself.
  - It takes the station and closes only the window that is about it — see [crafting stations](crafting-stations.md).

**On the equipment side**, every `Server_` wrapper whose verb can put an item or units INTO a slot asks `DescribeEquip`
first and reports once:

- `Server_EquipItem`, `Server_EquipFromInventory`, `Server_EquipFromStorage`, `Server_TransferInventoryToEquipped`,
  `Server_SwapEquipmentSlots`, `Server_MergeEquippedStacks`, `Server_SplitEquippedStack` and
  `Server_AutoMoveToEquipment`.
- Taking something OUT is a different question and reports through `ReportAdmissionRefusalForUnequip` instead — it asks
  whether the bag will accept what is coming off.

**A drop onto a worn slot is refused on the client, before any RPC,** so it reports there.
`USnapEquipmentSlotWidget::DescribeEquipDropRefusal` names it:

- `IncompatibleWithSlot` when the item does not fit the slot's type — asked first, so a ring dropped on a chest slot is
  never told about its requirements.
- Otherwise `DescribeEquip`'s answer (`SlotLocked`, `SlotBlocked`, `RequirementsNotMet`).
- The three a player can act on raise one notice through the equipment component's `ReportRefusal`
  (`USnapEquipmentSlotWidget::DoesEquipDropRefusalRaiseNotice`).
- `IncompatibleWithSlot` and `WrongContainer` stay in the log and `OnDropRefused`.
- The bag's shift+click quick-equip is not refused on the client: its RPC is sent, and `Server_AutoMoveToEquipment`
  raises that gesture's one notice.

`Server_AutoMoveToEquipment` names no slot, so it can only report the refusal that is true of the whole gesture: every
slot the item fits turns it away.

- Where a locked slot and a spanned one are both in play it names the **spanned** one, because that is the one the
  player can act on.
  - Taking the spanning item off opens the slot, while a lock does not answer to anything they can do right now.
- A shortfall caused by room rather than admission is not a refusal and is not reported.

⚠ `DescribeEquip` answers about **admission only**.

- Whether the bag can take what an equip would turn out is a question about the gesture, not about the slot.
  - A merge into a worn stack displaces nothing, and a rarity upgrade drops what it replaces to the world.
- So `NoRoomForDisplaced` is raised by the three whole-item equip wrappers and never by a stack door.
- A client — which must over-offer rather than under-offer — does not pre-reject it at all.

Admission refusals are emitted by the **gesture**, not by the verb.

- `AddItem` and `AdoptInstance` are recovery legs for undo paths — a refusal reported from inside one fires during a
  rollback, which is exactly wrong.
- `CanAddItem` and `CanAdoptInstance` are const predicates, and a predicate answering false is correct and silent by
  design.

`AdmitInstanceForGesture` is that rule's one helper:

- It asks `CanAdoptInstanceWithReason`.
- It names which clause refused (a per-definition unit cap says `UnitCapReached`, not `InventoryFull`).
- It carries the item definition as the subject.
- It substitutes `WrongContainer` for a subclass that refuses without naming a reason.

`TakeFromLootContainer` and the unequip admission test both go through it.

- So taking an item out of a chest into a bag that will not have it now says which of routing, room or a cap turned it
  away.
- For want of room in the player's own bag, it says "Your inventory is full." (below).
- With no free square for the item the take first tops up the bag's stacks of it with room, and says nothing when part
  of it moved.
- The notice is for a take that moved nothing.

**A Take All says one thing for the whole take.** A chest's or a body's Take All (`TakeAllFromLootContainer`) asks the
same question of each item silently and reports once:

- Part of it taken (a stack topped up only in part counts): `NotAllFit` ("Not everything could fit.").
- Nothing taken for want of room: `BagFull` ("Your inventory is full.", no subject).
- Nothing taken for another reason: that reason — naming the item only when there was one.

A pouch's Take All (`Server_CollectAllFromStorage`) says the same for a pouch in the bag.

- `BagFull` is raised only where the destination is the player's own bag.
- A full bank — or a pouch kept in the bank, whose items go back into the bank — says `InventoryFull` with no name in
  front.

**The player's own full bag is said one way: "Your inventory is full."**

Wherever a gesture is refused because the player's main bag — or a pouch in it — has no room, the notice is `BagFull`
("Your inventory is full.") with no item named. It is not `InventoryFull`'s "<Item> -- There is no room for that."

The gestures:

- A chest's or a body's item taken one at a time (Shift+click, X, a drag) or let go red over the bag.
- An item taken out of a pouch.
- Several items stored into a full one.
- An unequip.
- A split.
- A move out of another container into the bag (Shift+click, the item menu's Move to Bag and its dimmed row's reason).
- A purchase, a buyback, a sale or an upgrade at a shop (and the item menu's dimmed Upgrade row).
- A quest's rewards.
- A mix.
- A Sort that cannot re-pack the bag or a pouch in it.

One helper decides it at the report: `USnapInventoryComponent::PhraseRoomRefusal`, asked of the container that ran
out of room.

- `InventoryFull` on a `bIsMainInventory` component comes back as `BagFull` without its subject.
- A bank, a storage chest, a routed container such as a Spells bag, a pouch kept in one of those, and every other code
  come back as they were — "your inventory" would be false there.
- ⚠ Only the notice changes: for room every oracle, plan and `Can*` still answers `InventoryFull`.
  - So a project's `OnActionRefused` receives `BagFull` at those sites while a `Describe*` it asks still says
    `InventoryFull`.

⭐ **And a refusal that is not about room is not called full.** A quest's rewards used to count any item they
could not place as room. Now:

- A turn-in asks each reward of the container it belongs in (`USnapQuestGiverComponent::TurnInFor`, through
  `USnapInventoryComponent::CanAddItemsWithReason`).
- A shop's upgrade into a different item asks the bag about the item it would give
  (`USnapVendorClientComponent::DescribeItemUpgrade`), before the click and before anything is charged.
- So the code is the condition that refused, naming the item:
  - Room (as above — another full container, a routed item's own, says "<Reward> -- There is no room for that.").
  - A carry limit: `UnitCapReached` ("<Item> -- You already hold as many of those as you are allowed.").
  - An item nothing there takes: `WrongContainer`.
  - A project rule: its own code.
- `TurnInFor`'s return and `OnActionRefused` carry it.
- A shop sale's pouch whose contents cannot come out still says room ("No room to empty it" on its line).

**A press on an item on the ground the bag has no room for says so.** The item keeps its prompt (see
[Pickup modes](pickup-modes-and-the-collect-feed.md)).

- `Server_TryPickUpItem` asks `USnapCollectionLibrary::GetGrantRefusalForActor` first.
- When the collect landed nothing, it reports `BagFull` — or `UnitCapReached` naming the item at its cap.
- Walking over it stays silent.
- A pile the bag can take part of is taken in part (what fits, the rest left as a smaller pile) and reports nothing.

A split refused by the split window reports too, routed by the request's own source:

- The equipment component for an equipped subject.
- The inventory component for a grid item, a stored item or a currency counter.

It is a client pre-check, so it never crosses the wire.

Still log-only, by decision:

- `TryStackItem`'s per-definition unit cap. Its client-facing wrapper is refused by default
  (`bAllowClientRequestedAdd`), and its reachable caller is a merge loop that tries it speculatively and falls back —
  the same rollback hazard that keeps `AddItem` silent.
- Three of `TakeFromLootContainer`'s branches, none of which has one honest reason:
  - The viewer may not loot it (out of range, dead, or looting switched off — three different sentences).
  - Somebody else took it first (a race the shared-chest feature expects).
  - It routes back to the container being looted (a project's own setup).

⚠ `USnapHotbarComponent::ReportRefusal` exists and works, and **no shipped hotbar verb calls it**. It is there for a project
raising its own.

⚠ `USnapInventorySlotWidget::ReportDropRefusal` is a different surface: a `OnDropRefused` BlueprintImplementableEvent on the
slot widget, not the channel.

- **No shipped example WBP implements it**, so out of the box it shows nothing.

⚠ A handful of codes ship declared but not yet raised by any door. They are part of the vocabulary so a project can
raise them from its own policy.

## Describe, then do

Every user-facing refusal has a matching `Describe*` that returns the same code without mutating:

```
DescribeCraft(...)            DescribeSellCandidate(...)
DescribeEquip(...)
DescribeItemUpgrade(...)      DescribeItemRepair(...)
DescribeEquipmentUpgrade(...) DescribeEquipmentRepair(...)
AcceptsItemWithReason(...)    GetAdmissionRefusal(...)
```

Both halves ask the same planner, so a grayed button and the server's answer cannot disagree.

⚠ A `Describe*` reachable from a client must not read `NotReplicated` state. See [networking](networking.md).

## Reading one in Blueprint

Bind `OnActionRefused`, then:

```
Break FSnapRefusal   ->   Domain, Code, Subject
GetRefusalText(Refusal)   ->   "There is no room for that."
```

You can filter on `Domain` to route inventory refusals to one banner and trade refusals to another.

## See also

- [Networking](networking.md)
- [Extending the plugin](extending.md) — raising your own
- [Appendix: events](appendix-events.md)
