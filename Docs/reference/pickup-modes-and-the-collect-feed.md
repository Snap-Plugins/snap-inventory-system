# Pickup modes and the collect feed

How an item is taken off the floor — by walking, by pressing, or either — and how the player is told what arrived, and
what they just did with an item.

**On this page:**

- [Pickup modes](#pickup-modes)
- [The press](#the-press)
- [The collect seam](#the-collect-seam)
- [The feed](#the-feed)
- [Action lines](#action-lines)

## Pickup modes

`ASnapPickupActor` has two fields that decide how it is collected.

| Field | Default | Means |
|---|---|---|
| `PickupMode` | `Both` | `Overlap` (walk into it), `Interact` (press to take), or `Both` |
| `AutoPickupItemTags` | Empty | Which items the **overlap** may take; empty means all of them |

### The two together, and why neither works alone

The overlap has no filter of its own — before modes existed it collected anything it touched.

- So in `Both` with an **empty** filter, walking into the sphere takes the item before a press can matter.
- The mode is then indistinguishable from `Overlap` wearing a prompt.

`AutoPickupItemTags` is what separates the paths:

```
PickupMode         = Both
AutoPickupItemTags = Item.Type.Currency, Item.Type.Material
```

- Coins and ore are hoovered up on contact.
- A sword is left on the floor with a **Pick Up Iron Sword** prompt over it.
- **Set them together, or neither is observable.**

The match is `HasAny`, not `HasAll`:

- A filter is a list of kinds you are happy to collect automatically, and an item carries one type tag.
- So `HasAll` of a two-kind filter would match nothing.

### Empty means everything

An empty `AutoPickupItemTags` admits every item.

- That is what makes `Both` a **non-breaking** default: every pickup authored before modes existed has an empty filter
  and still collects on walk-over.

> This cannot be written as a bare container predicate.
>
> - `FGameplayTagContainer::HasAny` of an empty argument is trivially **false** and `HasAll` of one is trivially
>   **true**.
> - So the emptiness is an explicit branch in `IsOverlapCollectAllowed` ahead of either test.

### What `Both` changes that you will notice

It is non-breaking for collection and **not** invisible:

- Every pickup whose mode allows a press also carries a `USnapInteractableComponent`, so it gains a focus prompt and —
  because `bShowOutlineWhenFocused` defaults true — an **outline**.
  - A densely populated world reads differently for that reason alone.
- Set `PickupMode` to `Overlap` to get the old, silent pickup back.

### The combination that collects nothing

`Overlap` + a non-empty filter the item does not match has no route in at all: the overlap refuses it and there is no
press to fall through to.

- Nothing at runtime reports this (the overlap gate declines silently by design, or an `Interact` pickup would log once
  per footstep).
- So `IsDataValid` warns about it on save and at cook instead.

### Switching walk-over off for the whole project

The `bAllowOverlapPickup` setting (**Project Settings > Snap Inventory System - World > Pickups**, on by default) overrides
every pickup's walk-over at once. Off:

- Walking into a pickup that has a press does nothing.
  - `Both` behaves as `Interact`, and the player looks at it and presses to take it.
- A pickup in mode `Overlap` still collects on a walk-in: it has no press, so refusing it would leave it impossible to
  collect.

The setting is read on every overlap (`ASnapPickupActor::IsOverlapCollectAllowed`), so pickups already in the world
follow a change made during play.

## The press

`ESnapInteractionVerb::PickUpItem` is the fourth verb on the interaction system the plugin already ships, with its world
registry, focus cone, prompt widget and outline. A pickup composes a `USnapInteractableComponent`; nothing about the
pipeline is new.

A prompt appears where the collect would land — and where only room stands in its way:

- `USnapInteractableComponent::CanBeInteractedWithBy` asks `USnapCollectionLibrary::GetGrantRefusalForActor` — the
  plugin's own dry run of the real grant, with its reason named.
- An item the bag has no room for (`InventoryFull`, or `UnitCapReached` for an item at its container's unit cap —
  `IsRoomGrantRefusal`) keeps its outline and its key.
  - The press collects nothing and says "Your inventory is full." (`BagFull`), or names the item at its cap, and the
    item stays where it is.
- Every other refusal shows no prompt — an unmet pickup requirement, or no container on the player that takes the item —
  because making room would not change it.
- Walking over an item with a full bag stays silent.
- The dry run is optimistic on one axis: it cannot evaluate a fragment's `CanEquip` without minting an instance, so a
  fragment veto can still turn a prompt into a refusal on press.

⭐ **A pile the bag cannot take whole, it takes what fits of.** Five bread against a full bag whose bread stack has room
for two:

- The press, or walking over it, takes two — the stacks' room first, then free squares, within the item's carry limit
  and after any worn stack has taken its part.
- Three stay on the ground as a smaller pile (its count lowered on the server and replicated; a dropped pile's freshness
  and tier ride with the units either way).
- A press that took part says nothing; one that took nothing says "Your inventory is full."
- The walk-over says nothing either way.
- The collect feed shows what was taken.
- A pile that fits whole is collected as before.
- `GetGrantRefusalForActor` (and `CanGrantItemToActor`) answer `None` — a collect — when room for one unit is there.
- ⚠ The rest of a pile taken in part is not offered again while the player stands on it when room frees up.
  - Only a collect that took nothing is re-offered ([Loot and interaction](loot-and-interaction.md)).
  - Stepping off and back on takes more.

### It crosses to the server

`PerformInteraction` runs on the **viewing client** and a collect is a grant, so `PickUpItem` forwards to
`USnapInventoryComponent::Server_TryPickUpItem`. Before anything is granted, the server re-checks:

- That the actor is a pickup.
- That its mode admits a press.
- That the pawn is within `InteractionScanRange` (plus the target's own bounds radius).

The server deliberately does **not** re-test the view cone.

- It sees the presser's control rotation a round trip late, so re-testing the angle would refuse legitimate presses made
  while turning.
- Bounding the reach is what matters — everything inside it is something walking there would have collected anyway.

> **The key.** `E` reaches `TryInteract()` through the plugin's own `IA_Interact` (see [Input](input.md)).
>
> - A project whose own mapping context already maps `E` should decide which action owns the key.
> - The raw-bind fallback, `bBindInteractKeyRaw`, ships `false`.

## The collect seam

```cpp
USnapInventoryComponent::OnItemCollected(const USnapItemDefinition* ItemDef,
                                            int32 GrantedCount,
                                            ESnapCollectSource Source)
```

It reports **what was granted**, never what was asked for.

- The collection door returns everything, some of it, or none.
- So a partial collect into a nearly-full bag raises the smaller number, and a fully refused grant raises nothing at
  all.

Raise your own with `NotifyItemCollected(ItemDef, Count, Source)`.

- Do not broadcast the delegate directly — `NotifyItemCollected` is what decides where the notice has to be delivered.

**An item with affixes is named on the feed** — "Sturdy Iron Sword of Fury".

- The report carries the item's rolled affixes as tags (`NotifyItemCollectedWithAffixes` in C++).
- A listener reads them with `GetReportedItemAffixes()` during the broadcast.
- The feed composes the name on the player's own machine ([Affixes — Names](affixes.md#names)); rows bundle only when
  the name matches too.
- The plugin passes them for a world pickup, a craft handed over at a station, a crafted item and a quest reward.

### Exactly once, on the machine where the player's UI lives

`GrantItemToActor` returns 0 without authority, so it only ever runs on the server.

- A delegate raised there reaches the **server's** copy of the component, and a real client's copy never sees it.
- On a listen host that is the same object, so a feed built without a client leg passes every ordinary PIE test and is
  empty in a shipped game.

`NotifyItemCollected` therefore calls `Client_ReportItemCollected` on authority and returns.

- With a remote owner that crosses the wire.
- On a listen host or in standalone the owning connection is local, so the engine runs it in place.
- One notice, every net mode. It is the same branch `ReportRefusal` uses.

> **This is a per-player notice, not a server-side gameplay hook.**
>
> - On a dedicated server it is delivered to the owning client and the server does not raise it.
> - Server-side code that reacts to a ledger change binds **`OnInventoryChanged`**, which already fires on authority for
>   every mutation.
> - Two seams, two jobs.

### Where it fires from, and where it does not

| `ESnapCollectSource` | Raised by the plugin? |
|---|---|
| `WorldPickup` | **Yes** — `ASnapPickupActor`, both walk and press |
| `LootContainer` | No — raise it from your own take handler |
| `Vendor` | No — raise it from your own purchase handler |
| `Crafting` | **Yes**, when finished crafts that waited (at a station, or for room in the bag) are handed over; a craft that lands at once raises only its `Crafted` action line |
| `Trade` | No — raise it when a trade settles |
| `Script` | **Yes** for a quest giver's rewards and the `Snap.GiveItem` console command; otherwise the default for your own grants (debug grants, mail) |

**So the feed is a collect feed, not an "everything that entered my bag" feed.**

- A vendor purchase (`USnapVendorClientComponent::BuyFromVendor`), a take out of a chest, a craft output that lands at
  once (`USnapCraftingComponent`) and a trade receipt (`USnapTradeComponent`) all place items with the bag's own add
  calls.
- They do not pass through `GrantItemToActor`, so they raise nothing.
- The other members are not dead surface: `GrantItemToActor` takes the source as a parameter and is `BlueprintCallable`,
  so a project's own collect code passes the member that describes it.

To put a vendor purchase in the feed, call `NotifyItemCollected(Def, Count, ESnapCollectSource::Vendor)` from your
purchase handler.

## The feed

`USnapInventoryCollectFeedWidget`, hosted by `USnapInventoryHUDComponent`. Built on the **first** collect, so a project that
never collects anything pays for nothing.

- The first three rows below are on **Project Settings > Plugins > Snap Inventory System - Inventory UI > Collect Feed**.
- The rest are properties of `USnapInventoryCollectFeedWidget`: set them in a Widget Blueprint subclass and name it in
  `CollectFeedWidgetClass`.

| Setting or property | Default | Means |
|---|---|---|
| `bEnableCollectFeed` | True | Does a collect draw a row |
| `CollectFeedWidgetClass` | `WBP_ExampleCollectFeed` | The feed's Widget Blueprint; empty uses the native `USnapInventoryCollectFeedWidget` |
| `bShowActionsInCollectFeed` | True | Does an action draw a line too — see [Action lines](#action-lines) |
| `RowDurationSeconds` | 6.0 | How long a row stays fully shown before it starts to fade (each row on its own clock) |
| `RowFadeSeconds` | 0.6 | How long the fade takes; the row then slides shut and is removed |
| `MaxRows` | 8 | Rows on screen at once; a further collect retires the **oldest** |
| `RowGap` / `RowHeight` / `FeedWidth` | 4 / 40 / 320 | Layout, in slate units; `RowHeight` is a row's LEAST height |
| `FeedAnchor` / `FeedAlignment` / `FeedOffset` | (0,1) / (0,1) / (24,-24) | Where it sits: the bottom-left corner |
| `bNewestOnTop` | False | The newest row on top and the list growing downward, for a top corner — see [In a top corner](#in-a-top-corner) |
| `bWrapLongLines` | True | A line too long for its row goes on to a new line and the row grows |
| `RowPlateColor` / `RowPlateEdgeFade` | (0,0,0,0.24) / 0.25 | The plate behind each row, and how much of its width each end spends fading out |
| `RowExpandSeconds` / `RowCollapseSeconds` | 0.12 / 0.18 | How long a row takes to slide open, and to slide shut after its fade |
| `NewestRowIndent` / `NewestRowPopScale` / `NewestRowPopSeconds` | 12 / 1.05 / 0.3 | How the newest line is marked: set in, and a pop |

**The newest row is at the bottom and pushes the older ones up.**

- The feed's viewport slot sizes itself to its rows and is held by its bottom edge (`FeedAlignment` (0,1)), so it grows
  upward as rows arrive.
- Every row is `FeedWidth` wide.
- Each row fades on its own clock — so the oldest, at the top, goes first.
- A bundle into a row restarts its clock and shows it whole again.

### How it moves

**Nothing jumps.** A row is never added, removed or resized inside the feed's own paint.

- A `UUserWidget` ticks there, after the screen has already sized and placed it for the frame.
  - So a row removed in its tick left the feed one row too tall for that frame, and every other row drew one row higher,
    then dropped back (measured: 44 units, at every removal).
- The feed moves its rows on a world timer that runs once a frame, before the screen is laid out, and only while it has
  rows.

The motions:

- A row that has faded out **slides shut** over `RowCollapseSeconds`: its height — measured from its content — and the
  gap it leaves ease to nothing, and it is removed.
- A new row **slides open** over `RowExpandSeconds` once it has been measured, fading in.
- The rows around them glide.
- **The newest line is marked**: The line last added, or last bundled into (it stays where it is), is set in by
  `NewestRowIndent` away from the side the feed is anchored to.
  - It pops — grows to `NewestRowPopScale` and settles back over `NewestRowPopSeconds`, like a landing item.
  - Reduce Item Motion drops the pop and sets the line in at once.
  - The feed writes each row's render transform for this, so a row Widget Blueprint's own render transform is replaced.
- **A long line wraps** (`bWrapLongLines`) at an explicit width — the row less its padding, the icon and the count — so
  the row's height is right on the frame it appears, and the row grows.
- **The plate** is a `USnapFadeBar`: one color whose left and right ends fade out on an eased ramp, behind a transparent
  frame.
  - A row Widget Blueprint gets it through its optional `PlateWidgetName` part (`WBP_ExampleCollectFeedRow` has one);
    without the part a row keeps its own look.

A row Widget Blueprint without a sizer part cannot slide: it appears and goes in one step, still before the frame is
laid out.

### In a top corner

1. Set `bNewestOnTop`.
2. Set a top anchor: `FeedAnchor` (1,0), `FeedAlignment` (1,0), `FeedOffset` (-24,24) for the top-right corner, or
   (0,0), (0,0), (24,24) for the top-left.

You should see the newest line as the top row, the list growing downward; the oldest, now the bottom row, still goes
first.

- `SetNewestOnTop` re-orders rows already showing (call `ApplyViewportPlacement` after changing the placement).
- The demo's FEED ON TOP pad does exactly this to the live feed.

**An empty `CollectFeedWidgetClass` means "use the built-in", not "no feed".**

- The widget builds its whole tree in C++, so there is no configuration of that setting that produces a broken feature.
- A class that will not load is refused with a warning and the native class is used instead.

### Bundling

Collect rows are keyed on the **definition alone** (an action line on the definition and its action — see
[Action lines](#action-lines)).

- Collecting five potions one at a time produces one row reading `x5`, not five rows.
- The row's clock restarts each time, because a player still collecting the same thing is still interested in it.
- Two different definitions produce two rows.
- Source is deliberately not part of the key: five arrows off the floor and five from a shop are ten arrows to the
  player reading the feed.

### It is not modal

`AddToViewport` stretches the root across the whole viewport, so a hit-testable feed would swallow every click on screen
with nothing logged. Closed twice:

- The slot's anchors are collapsed to a point with an explicit size.
- **And** the whole tree is `HitTestInvisible` (inherited by children, unlike `SelfHitTestInvisible`).

The slot is written **once**, as one `FGameViewportWidgetSlot` through `UGameViewportSubsystem::SetWidgetSlot`.

- `SetPositionInViewport` and `SetDesiredSizeInViewport` reset the slot's anchors to (0,0) as they write, so anchors set
  before them are lost.
  - Anchors lost that way draw the feed above and to the left of the screen, out of sight.
- A subclass that places the feed itself should write the slot the same way.

The native rows draw their name and count in the theme's **RowName** typography role (Regular 14), which fits the row's
band.

- A Widget Blueprint row takes that role through its font rows (`WBP_ExampleCollectFeedRow` lists one for the name and
  one for the count).
- It keeps its own fonts where it lists none.

The slot is AUTO-SIZED (a zero size with point anchors):

- The canvas measures the rows every frame, so nothing reads the widget's desired size — `(0,0)` until Slate has run a
  prepass — and nothing needs a deferred retry.
- Every row is `FeedWidth` wide (its sizer's width override), so only the height changes as rows come and go.

### Sound and color, per item

`USnapItemFragment_Presentation` holds seven sound fields (lift, place, pickup, equip, consume, break, drop) and an
accent color.

**The sound fields are per-item overrides.**

- The plugin plays its item sounds itself — an item sounds like its type
  (**Project Settings > Plugins > Snap Inventory System - UI Core > Sound**).
- A field set here replaces that one moment for that one item.
- An empty field means "the type's sound".
- See [Sounds](sounds.md) for the whole order, the switch and the volume.
- `ResolvePickupSound(ItemDef)` and its siblings answer only this fragment's own field.

There is no `GameplayCue` path: `GameplayAbilities` is optional for this plugin, and these sounds work the same with or
without it.

The fragment is **absent-safe**: most of the forty-three example items do not carry one.

- The two swords, the axe, the eight potions and the recipe scroll do — the potions and the scroll only for their
  `UsedFeedText`.
- An item without one sounds like its type and warns about nothing.

`AccentColor` defaults to **transparent black**, which is the "unset" sentinel.

- No designer authors it deliberately, so an unauthored fragment is distinguishable from one that meant a color.
- Tick `bHasAccentColor` to select a transparent accent on purpose.
- The feed uses the accent when an item has one and the item's rarity color otherwise, so rows are tinted out of the box
  with no authoring.
- `AccentColor` is **not** `USnapItemFragment_UIData::RarityColor`: rarity is about the item's tier, an accent is about
  its identity, and an item may carry both.

## Action lines

The feed also says what the player **did** with an item — "Drank Health Potion", "Mixed Battle Tonic",
"Repaired Iron Sword", "Dismantled Iron Sword", "Learned Fireball", "Crafted Health Potion" — and when one of their
items **broke** ("Broke Iron Sword").

```cpp
USnapInventoryComponent::OnItemActionPerformed(const USnapItemDefinition* ItemDef, int32 Count,
                                                  ESnapItemAction Action,
                                                  float CooldownRemaining, float CooldownDuration)
```

| `ESnapItemAction` | Raised by the plugin from |
|---|---|
| `Used` | The bag's consume door (`ConsumeItem`) and a worn or hotbar slot's (`USnapEquipmentComponent::ConsumeSlot`) |
| `Mixed` | A mix (`MixItems`) — for the result, with the count made |
| `Repaired` | A repair kit (`ApplyRepairKit`), a field repair (`FieldRepairItem`) and a vendor repair (`RepairItem`) |
| `Dismantled` | `DismantleItem`, with the units spent |
| `Crafted` | A craft's completion — `CraftRecipe` at once, or each unit of a timed craft as it comes out of the queue — for the made item, with the units made |
| `Broke` | `ApplyDurabilityDamage`, right after `OnItemBroke`, for the item whose durability reached zero |
| `Learned` | Nothing by itself — a learning path raises it with `NotifyItemAction` |

Only an action that **succeeded** is reported, and only by the server.

- Each door raises it after the change has landed, so a client never shows a line for something that was refused.
- A purchase is not an action — the shop that sold it shows it — and neither is equipping.

It is delivered exactly the way the collect seam is.

- `NotifyItemAction` routes to the owner's **main** container (the one the feed binds), then takes the same authority
  branch, so one line arrives in every net mode.
- Raise your own with `NotifyItemAction(Def, Count, Action)` — but a path that also goes through one of the doors above
  already has its line, and would show two.

### The line

One keyed sentence per action, with `{Item}` where the name goes: "Used {Item}", "Learned {Item}", "Mixed {Item}",
"Repaired {Item}", "Dismantled {Item}", "Crafted {Item}", "Broke {Item}".

- A Used line takes the item's own sentence when its Presentation fragment sets **`UsedFeedText`** ("Drank {Item}",
  "Read {Item}").
- It is one text for the whole line, so a translation can put the verb anywhere.

Lines bundle by definition **and** action:

- Two potions drunk read `x2`, and drinking one never folds into the line that said you picked it up.
- Every line — collect or action — also bundles only with the same **rarity tier**.
  - The collect and action reports carry the item's tier (`USnapInventoryComponent::GetReportedItemRarity`).
  - So an uncommon and a rare sword picked up together are two rows, each in its tier's color.

### The cooldown on the line

A used item that now has a cooldown carries it to its line, drawn exactly as a hotbar square draws it:

- A shade over the icon that wipes away as the cooldown runs out.
- The whole seconds left (`FSnapItemCooldownOverlay`).

A cooldown **never keeps the line up**: it fades on its own clock like any other, and over `MaxRows` the oldest line
goes first, whatever it shows.

The cooldown is `USnapHotbarComponent::GetItemConsumeCooldown` — the item's fragments' `GetConsumeCooldown` answers and,
with GameplayAbilities, the cooldown of the project's consume ability — the same answer a consumable hotbar square
draws.

- It is measured **on the server**, right after the use, and carried in the notice.
- A client that read its own replicated cooldown when the notice arrived could read nothing, because the notice leaves
  the server before that frame's property replication does.

> The fragments are handed the owner's `USnapHotbarComponent`. With none, a fragment that reads its owner through it answers
> no cooldown, and the line has no ring.

A Widget Blueprint row draws the shade with its own named parts — an Image `SnapItemCooldownShade` and a TextBlock
`SnapItemCooldownText` — or, with neither, has them built beside its icon when the icon sits in an Overlay.

### Text lines

`USnapInventoryComponent::NotifyFeedText` puts a plain line on the owning player's feed — no icon, no count, never
bundled, faded on its own clock.

- The plugin raises one at a quest turn-in, "Quest complete: <title>", just before the reward lines (so it sits just
  above them).
- It raises "Some crafted items are still waiting -- make room." when finished crafts wait for room in a full bag: once
  when a craft starts waiting, and each time a station's or shop's window opens with crafts still waiting there. The
  refusal `CraftsStillWaiting` still reaches `OnActionRefused` at those moments, quiet (`bQuiet`), so it raises no notice.
- A project can raise its own (`OnFeedTextReported` fires on the main container; the HUD calls
  `USnapInventoryCollectFeedWidget::PushTextLine`).

The `bShowActionsInCollectFeed` setting turns action lines off.

- Pickup lines are unaffected, and `OnItemActionPerformed` still fires for a project that draws its own.
