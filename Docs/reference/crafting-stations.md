# Crafting stations

A forge is not a shopkeeper. This page covers how to author a crafting station, what it lists and opens, its session, how
the server trusts it, refusals, walking away, and how finished crafts wait at the station.

**Background:**

- `FSnapCraftingRecipe::RequiredStationTag` gates a recipe to a station, and the gate has always worked.
- What was missing was a second way to be *at* one.
  - The only source of station tags used to be an open vendor session.
  - So the only way to ship a forge was to give it a `USnapVendorStockDataAsset` and a shop screen it did not want.
- `ESnapInteractionVerb::OpenCraftingStation` is that second source.

**On this page:**

- [Authoring one](#authoring-one)
- [The session](#the-session)
- [Trust](#trust)
- [When the server says no](#when-the-server-says-no)
- [Walking away](#walking-away)
- [Finished crafts wait at the station](#finished-crafts-wait-at-the-station)
- [Settings](#settings)
- [See also](#see-also)

## Authoring one

Put a `USnapInteractableComponent` on the actor and give it an option:

```
Options            [ { Verb = OpenCraftingStation, Enabled } ]
StationTags        SnapInventorySystem.Crafting.Station.Forge
StationRecipeSet   (optional) a USnapCraftingRecipeSet of its own
```

- **One actor can be several things at once.** A blacksmith is `[ { OpenVendor }, { OpenCraftingStation } ]`.
  - The first available option is what the key performs.
- **The payload fields are always visible,** whatever the options say.
  - The details panel can only key a condition off another PROPERTY, and no property says what a list contains.
  - So hiding them behind a verb made a two-option blacksmith impossible to configure at all.

**What you fill in decides what it is:**

- **Tags only** — a forge that makes part of the shared `DefaultRecipeSet`: the recipes whose `RequiredStationTag` is one of
  its tags. The common case.
- **Recipe set only** — a bench that offers its own list and gates nothing: the recipes in its list that need no station.
- **Neither** — a misconfiguration.
  - The component offers no prompt at all, rather than one that opens an empty window.
  - `IsDataValid` says so.

### What a station lists

**Each station lists only its own recipes.**

- Its craft window shows the recipes whose `RequiredStationTag` is one of its `StationTags`.
  - They come from its own `StationRecipeSet` when it has one, else from `DefaultRecipeSet`.
- Or, for a bench with a list and no tags, it shows the recipes in its list that need no station.
- A recipe that needs another station is not offered there at all.
- The recipes that need no station are made from the bag's CRAFT window.

**A shop can be a station the same way.** A shop whose `USnapVendorStockDataAsset` carries `StationTags` (or its own
`RecipeSet`) is a station:

- Its CRAFT tab lists its own recipes.
- A shop whose tags match none shows no CRAFT tab.

**A shop that is no station** — no tags and no list of its own — lists the recipes it can make there:

- The ones that need no station, from the list it crafts from (its `RecipeSet`, else `DefaultRecipeSet`).
- A shop with none of those shows no CRAFT tab.

**In the demo:**

- The demo's forge lists the Iron Sword and the Shield.
- The General Store's CRAFT tab lists the five recipes of its own list (`DA_ExampleGeneralStoreRecipes`: the Ring of
  Vitality, two Health Potions, two Repair Kits, Iron Gauntlets and a Leather Pouch).
- The Gem Trader's and the Junk Merchant's list the recipes that need no station.

**The bag's own CRAFT window lists every recipe.**

- One whose station is not active there says where it is made, on its row and in its status line: "Made at: General Store".
- The names come from `CraftingStationNames` in the settings (station tag -> keyed text).
- A tag with no entry is called by its last segment, spaced (`SnapInventorySystem.Crafting.Station.Forge` -> "Forge").
- The setting ships empty. The example demo names `SnapInventorySystem.Crafting.Station.Vendor` "General Store" for its own
  session.

**What decides it:** `USnapInventoryCraftPanelWidget::SetStationView`.

- The HUD sets it when it opens the craft window for a station (`ArmStationRangeWatch`).
- The shop screen sets it from its stock.
- Every other open clears it (`OpenCraftWindow`).

**The shipped example:** `BP_Example_Forge` in `/SnapInventorySystem/Examples/Blueprints`. There is one placed in the demo
map.

⚠ Station tags belong under `SnapInventorySystem.Crafting.Station`.

- `USnapCraftingRecipeSet::IsDataValid` refuses a `RequiredStationTag` outside that namespace.
- So a tag anywhere else can never match a recipe the validator let through.

⚠ Declare your own station tags **natively** (`UE_DEFINE_GAMEPLAY_TAG_COMMENT`) or in your project's `Config/Tags`.

- A tag that is not declared anywhere imports **empty**.
- That silently turns `RequiredStationTag` into "craftable anywhere".
- And then a test that a forge recipe crafts at the forge *and* a test that it refuses elsewhere can both fail to mean
  anything.

## The session

```
GetActiveStationTags()                          every station you currently count as
BeginStationSession(Tags, RecipeSet)            server-side, trusted caller
EndStationSession()
Server_BeginStationSession(Station)             client-reachable — takes the COMPONENT
Server_EndStationSession()
IsStationSessionOpen()   GetStationRecipeSet()
```

- Pressing the station opens the craft window and begins a session.
- Walking out of range closes both.

### What a station opens

**Your bag, in the station's view** (`bCraftingStationOpensInventory`, true) — the way a shop opens its own screen. The bag
screen comes up with:

- The station's craft window where the paper doll stands.
- The station's name over the screen (`CraftingStationNames`, else the station tag's last part — "FORGE" in the demo;
  "CRAFTING" when it has none).
- The queue inside the details column (only the crafts ordered at this station), and the details framed like a shop's middle
  column.
- The bag column as a shop shows it: the grid, its filters and sort and the wallet.
  - Without DROP INVENTORY, the bag's CRAFT button or the other containers' panels (Spells).
- The screen backdrop (`ScreenBackdropMode`), which comes with the screen.

How the view behaves:

- **It is docked, with windows floating too.** The window is part of the screen: no close box, never a separate focus root.
- **On a pad, focus starts on the selected recipe** (else the first one the panel would pick).
  - The crafting menu's exits are the same as everywhere ([input](input.md)).
- **B / Escape, walking out of range and a server refusal close the view**, and the bag with it when the station's press
  opened it.
  - When the bag goes with it, focus goes back to the game.
  - A bag that was already open stays open, back to its own look (one B closes the view, the next the bag).
  - The toggle key and dying close both.
- **Finished crafts waiting at the station are handed over when the view opens** (`CraftDeliveryMode` AtStation).
  - The docked window tells the server it is open, as the window on its own does.
- **A project's own HUD** is asked `SnapSetInventoryScreenVisible(true)` and then `SnapIsInventoryScreenVisible`
  (`ISnapScreenHost`; false refuses the press, and the interface's default answer is false), then `SnapOpenCraftScreen`
  (`ISnapCraftStationHost`).
  - The view itself is the shipped HUD's (`USnapInventoryHUDComponent::BeginCraftStationOpen`, `IsCraftStationViewOpen`).

**With the setting false, only the craft window** opens, on its own over the world — the bag stays closed.

- `USnapInventoryHUDComponent::EnsureCraftWindow` builds the inventory screen's widget without showing it, because the craft
  window IS that screen's craft panel.
- It is asked before the session begins, so a window that cannot open (a dead player, a screen that hosts none) commits
  nothing.

How the window behaves:

- **It is always a real window.**
  - With nothing else on screen, it takes the cursor and the screen input state itself, as the loot window does.
  - It opens centered, holds keyboard focus, and has a close box — whatever `bAllowWindowDragAndClose` says.
  - That is because a fixed, uncloseable window there would be one the player could not leave.
  - On a pad it is its own focus host: focus starts on the selected recipe and the crafting menu's exits apply
    ([input](input.md)).
- **It looks like the station's view.**
  - The station's name is its heading (the panel's own heading when the station has none).
  - The queue sits inside the details column, and the details are framed.
  - The screen backdrop (`ScreenBackdropMode`) is drawn behind it, as behind a shop's screen.
  - The backdrop is drawn only: a drag does not move the backdrop, and a press outside the window still reaches the world.
  - The backdrop goes when the window closes.
- **Escape** or its **close box** closes it — the interact key does not.
  - Like every plugin window, it suppresses interaction while it is up.
  - **Walking out of range**, **dying** and a server refusal close it too, and each hands the cursor back.
- **The toggle key opens the bag exactly as it always opens it**.
  - The station's window gives way (and its backdrop with it — the bag draws its own).
  - The bag opens with its craft window closed, behind the bag's CRAFT button, whatever `bAllowWindowDragAndClose` says.
  - With it off, CRAFT opens the window in the equipment panel's place beside the bag.
  - The station's session carries on while the player stays in range, so a station recipe is still craftable from that
    button.
- A vendor's craft tab is the same either way.
- A project's own HUD is asked `SnapEnsureCraftScreen` (its default answers true) and then `SnapOpenCraftScreen`
  (`ISnapCraftStationHost`), with no bag shown first.

### One derivation

- `GetActiveStationTags()` returns the **union** of an open vendor session's `StationTags` and an open station session's.
- It is the only place in the plugin that derives them.

**That matters more than it looks:**

- The expression it replaced was copy-pasted at six sites:
  - Four server-side (`Server_CraftRecipe`, `Server_DismantleItem`, `Server_DismantleItems`, `SalvageByRarity`).
  - Two client-side (the context menu's dismantle rows and the shop's CRAFT tab).
- So a second source of station tags would otherwise have meant widening six copies.
- And the next door added would have missed it silently.

It is also what makes the client's craft preview agree with the server **by construction**.

- Both halves of the union are replicated `COND_OwnerOnly`.
- So the owning client and the server run the same function over the same state and cannot reach different answers.

⚠ This is why `USnapInventoryCraftPanelWidget` asks the component instead of being handed a snapshot at seed time. It is also
why `SeedCraftPanel` no longer takes a station-tag parameter.

- A captured value is never re-read.
- So on a client — where `Server_BeginStationSession` is asynchronous and the window opens first — the panel would sit at
  "no station" until something re-seeded it.
- On a listen host, it would work perfectly all the while.
- See [Networking](networking.md).

### A station session and a vendor session are independent

- Both may be open at once. A player standing at a forge inside a shop is at both.
- Neither teardown touches the other's state:
  - `EndStationSession` clears only the station fields.
  - `EndVendorSession` clears only the vendor ones.
- That independence is structural rather than a rule each door has to remember, which is deliberate.
  - The alternative shape produced a real bug once, where one screen's teardown ended the session another screen was
    relying on.

## Trust

`Server_BeginStationSession` takes the **interactable component**, never a tag container. The server reads `StationTags`
off the named component after checking:

- That its verb really is `OpenCraftingStation`.
- That it is switched on.
- That the caller is really within `InteractionScanRange` of it.

**Why not a tag container:**

- A raw `FGameplayTagContainer` parameter would be exactly as forgeable as the `StationTags` parameter `Server_CraftRecipe`
  already discards.
  - Any client could claim a forge from anywhere and craft the whole station-gated tree.
- The client names *which* station. It never asserts *whether* it is at one.

⚠ The discard rule at the craft and dismantle doors is unchanged.

- Those functions still ignore the `StationTags` a client sends and re-derive from `GetActiveStationTags()`.
- Widening where the server *derives* from is not the same as starting to trust the parameter.

**The server's reach test is range-only** — deliberately more permissive than the client's, which also applies a view cone.

- The server sees the presser's control rotation a round trip late.
- So re-testing the angle would refuse legitimate presses made while turning.

## When the server says no

- The window opens locally and the session crosses the wire.
- So on a real client, the two answers arrive a round trip apart.
- A refused session therefore has to reach back:

```
Client_RefuseStationSession(Station, Refusal)    the refused half of Server_BeginStationSession
IsCraftWindowOpenForStation(Station)             on the HUD — is the open window about THIS station?
```

**Each of the three checks above reports one code** through the ordinary [refusal channel](refusals.md):

- No enabled `OpenCraftingStation` option and a switched-off station both say `CraftingDisabled`.
  - To a player those are one fact — crafting is not available here.
  - The log lines are where they are told apart.
- A failed reach test says `OutOfRange`.

The client closes the craft window and shows the reason.

⚠ **It closes only the window it is about.**

- The refusal is asynchronous.
- So between the press and its answer, the player can legitimately have a different craft window up — a second station that
  was accepted, or one opened from the bag.
- `FSnapRefusal` carries a domain, a code and an item definition, so it cannot identify a session.
- The test is the station the HUD armed its range watch to.
- A mismatch still shows the reason and closes nothing.

⚠ It does not end the session or disarm the watch, because a refused request opened neither.

⚠ **The fourth early return — a station that no longer exists — stays silent.**

- That is the ordinary race of a destroyed or streamed-out actor, not something a player did.
- It is logged at `Verbose`.

**On a listen host or in standalone,** the RPC runs in place.

- So its answer is already knowable at the press, and no window is opened at all.
- The refusal itself still goes out through the same leg.
- That is what makes a host and a client agree about what a refusal looks like.

## Walking away

**Pressing a station arms a range watch:**

- It is a looping client-side timer (`ProximityCheckInterval`, a quarter of a second by default).
- It holds a weak reference to that station.
- It uses the same `InteractionScanRange` the prompt appeared at.
- Walking out of it ends the session and closes the window.

⚠ **The watch belongs to the session, not to the window**, and that is where it stops mirroring the shop's.

- A vendor session has no meaning once its screen is gone, so `CheckVendorRange` stands down with the screen.
- A station session does have meaning without its window — the player is still standing at the forge.
- And `USnapInventoryScreenWidget`'s construct closes the craft window unconditionally, as a documented per-open "start closed"
  assertion.
- Tying the session to the window would therefore make *opening your bag at a forge* silently drop the station.

So closing the craft window leaves the session open, and the watch stands down when the **session** ends.

- That also covers one ended by any other route.

More about the watch:

- A craft window opened any other way — the bag's own toggle button, a Blueprint — begins no session and is deliberately
  **unwatched**.
  - Closing a hand-opened window because the player walked away from an unrelated forge would be worse than not policing
    it.
- A destroyed or streamed-out station counts as out of range.
- `ArmStationRangeWatch` takes the station as a **parameter** rather than reading the detector's current focus.
  - So the arm cannot depend on whether focus survived the window's open path.
  - And walking from one forge to another re-points it.

## Finished crafts wait at the station

A timed craft can finish while its crafter is somewhere else.

- **By default** (`CraftDeliveryMode` **AtStation**), a craft ordered at a crafting station — or at a shop that crafts — goes
  into the bag only while the player is within reach of that station **with its window open**.
  - A shop's window is its shop screen.
- One that finishes while they are away, or with the window closed, **waits at the station**.

A craft that waits at the station is:

- **For that player only.**
  - Kept on the server, sent to that player's machine alone (`GetCraftsWaiting`).
  - Saved with their profile like the craft queue, so a logout or a death loses nothing.
- **Shown by a row of its icons floating over the station** on that player's screen, gently bobbing.
  - One light, rounded square per waiting stack (the theme's `SlotOccupied`, as a bag square).
  - Its count when there is more than one, and nothing around the row.
  - Nobody else sees them.
  - The row is `CraftWaitingWidgetClass` (the example `WBP_ExampleCraftWaiting`, a copy of the native
    `USnapCraftWaitingWidget`), `CraftWaitingHeightAboveTop` above the station's top.
- **Announced with the craft sound for that player when it finishes,** wherever they are — and nothing else.
  - No collect-feed line, since nothing reached the bag.
  - The event is `OnCraftWaitingReported` on the player's main container; the HUD plays `ESnapActionSound::Craft`.
- **Handed over the next time the player opens that station's window** within the server's reach of it.
  - As much as fits goes into the bag at once (one bag update), each stack with a line on the collect feed.
  - What does not fit keeps waiting, and a line on the player's collect feed says "Some crafted items are still waiting --
    make room."
  - Freeing room while standing there with the window open hands over the rest.

A craft that finishes while the window is open and the player in reach goes straight into the bag. With the bag full it waits
there too, drawn in the station's craft queue as a finished tile (a full bar, its count and a green check), and moves in as
soon as room frees while the window is open (`bCraftWhenBagFull`; see [Timed queues](crafting.md#timed-queues)).

**Where a craft waits.** The place is decided when the craft is ordered (`FSnapCraftQueueEntry::SnapshotStationKey`):

- A recipe gated on a station is made at the session that carries that station's tag.
- A recipe gated on nothing is made at the crafting station whose craft window is open, else at a crafting shop that is
  open.
- A craft ordered away from any station — the bag's own CRAFT window — goes straight into the bag wherever the player is.
  With the bag full it waits, finished, in the bag's own queue and moves in as soon as there is room, wherever the player is.

**The station's identity** is its interactable's object path with any play-in-editor prefix removed
(`SnapCraftDelivery::MakeStationKey`).

- A station placed in a level is loaded under that name every time, on every machine.
  - So a save finds the same forge again, and a client can find the station its icons belong over.
- A station **spawned while the game runs** has no such name, so a craft ordered at one counts as the bag's own: it shows in
  the bag's queue and goes into the bag wherever the player is.
- Two level instances of one level share their actors' names, so their stations share one place to wait.

**What the server checks.**

- The server cannot see a window, so the player's machine tells it when the craft window opens and closes
  (`Server_SetCraftWindowOpen`).
- A hand-over also needs the station session — begun on the server after its own reach check.
- And the server measures its reach again at every hand-over: when the window opens, when a craft finishes and when room
  frees.
  - So a client that lies about its window, or keeps its session after walking away, gets its crafts no sooner than by
    opening the window at the station.
- `ClaimCraftsWaitingAt` hands over what waits at a station from server code (it does not check where the player is).

`CraftDeliveryMode` **Immediate** puts every finished craft straight into the bag wherever the player is; one that finds the
bag full waits for room, then moves in wherever they are.

## Settings

These are on **Project Settings → Plugins → Snap Inventory System - Crafting** (the delivery, the waiting icons and
`bCraftQueuePerStation` under **Crafting | Timed Queue**).

- The `bEnableVendorCrafting` setting is the master switch for *station-gated* recipes — every recipe whose
  `RequiredStationTag` is set, not only ones reached through a vendor.
  - Its name predates stations being separable from shops.
- The `bEnableInventoryCrafting` setting governs anywhere-recipes.
- The two are independent.
- The `bCraftingStationOpensInventory` setting (true) decides whether a station opens the bag in its station's view or only
  its craft window. See [What a station opens](#what-a-station-opens).
- `CraftDeliveryMode`, `CraftWaitingWidgetClass` and `CraftWaitingHeightAboveTop` decide where a finished timed craft goes
  and how its waiting icons look. See [Finished crafts wait at the station](#finished-crafts-wait-at-the-station).
- `bCraftQueuePerStation` (true) gives each station, crafting shop and the bag's own craft window its own queue: each shows
  only its own crafts and crafts them on its own clock. `bCraftWhenBagFull` (true) lets a craft go on into a full bag. See
  [Timed queues](crafting.md#timed-queues).

## See also

- [Crafting](crafting.md) — recipes, the planner, the queue
- [Loot and interaction](loot-and-interaction.md) — the verb list, prompts, focus
- [Vendor](vendor.md) — the shop session and its own `StationTags`
- [Networking](networking.md) — why the session is replicated
- [Appendix: settings](appendix-settings.md) — the **Crafting** category
