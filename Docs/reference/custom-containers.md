# Custom containers: add your own storage

An extra container beside the main bag — "Spells", "Shards", "Quests", "Runes", anything — is a **custom
container**: a data-only Blueprint of `USnapCustomContainerComponent`. This page covers what ships, how to add one, and
the worked examples (Spells, Bank, spellbook) with the rules they show.

- **No C++.** Its name, its tag, its capacity and whether the abilities of what it holds are active are all values in that
  Blueprint.

**On this page:**

- [What ships](#what-ships)
- [A container's definition is its Blueprint's class defaults](#a-containers-definition-is-its-blueprints-class-defaults)
- [Adding one, with no C++](#adding-one-with-no-c)
- [Abilities while held: the Spells example](#abilities-while-held-the-spells-example)
- [A container that takes anything, saved on its own: the Bank example](#a-container-that-takes-anything-saved-on-its-own-the-bank-example)
- [A container that keeps what enters it: the spellbook example](#a-container-that-keeps-what-enters-it-the-spellbook-example)
- [A container used only where it is kept: the reach rule](#a-container-used-only-where-it-is-kept-the-reach-rule)

## What ships

Spells and Shards are in `Attachment/`, the demo's two in `Examples/Blueprints/`.

| Blueprint | Tag | Slots | Held-item abilities | Attached by the default setup |
|---|---|---|---|---|
| `BP_Example_SpellsContainer` ("Spells") | `SnapInventorySystem.Container.Spells` | 8 | On | Yes, when the plugin is built with GameplayAbilities |
| `BP_Example_ShardsContainer` ("Shards") | `SnapInventorySystem.Container.Shards` | 7 | Off | No — one line attaches it (below) |
| `BP_Example_KeyringContainer` ("Keyring") | `SnapInventorySystem.Container.Keyring` | 32 | Off | No — add its row to turn it on |
| `BP_Example_BankContainer` ("Bank") | `SnapInventorySystem.Container.Bank` | 24 | Off | No — the example demo attaches it (`DA_ExampleDemoContainerRegistry`) |

- The tags are declared in the plugin's tag source (`Config/Tags/SnapInventorySystemTags.ini`).
- The example screens show Spells with `WBP_ExampleContainerPanel` and Shards with `WBP_ExampleShardPanel`.
- A panel hides itself while the player has no such container.

**Shards in one line.** The plugin ships a second registry, `DA_SnapContainerRegistry_WithShards`.

- It names the default registry in its **Included Registry** and adds one row, the Shards container.
- Point the setting at it, and every player gets the default containers plus Shards:

```ini
[/Script/SnapInventorySystem.SnapFoundationSettings]
ContainerRegistry=/SnapInventorySystem/Core/Attachment/DA_SnapContainerRegistry_WithShards.DA_SnapContainerRegistry_WithShards
```

## A container's definition is its Blueprint's class defaults

| Class default | What it is |
|---|---|
| `ContainerTagOverride` | **Required.** The container's identity: the tag an item's Container Routing fragment names to land here, the tag a panel binds by, and the tag a save restores by. |
| `DisplayName` | **Required.** What a panel calls it. Keyed text, so it localizes. |
| `SlotCountOverride` | Its capacity. Every instance takes it when it is created; left unset (-1), a custom container has 8. |
| `Icon` | Optional art for a panel header or a tab. |
| `bActivateHeldItemAbilities` | The abilities the items on its own grid grant are active while they are held, and the hotbar's Ability bar draws its items from it (below). The grants need GameplayAbilities; the Ability bar half works in every build. |
| `bKeepsWhatEnters` | An item that enters **stays**: the player can rearrange it inside and bind it to a hotbar, never take it out, and a death keeps it (below). Off as shipped. |

**Why a Blueprint per container, and not one asset handed to one component.** Two engine facts decide it:

- `UGameFrameworkComponentManager` attaches one component per class per actor and gives the component no data from the
  row that asked for it.
- A client creates a replicated component from its class — and begins play with it — before any of that component's
  replicated values arrive.

Class defaults are on both ends from the first frame. So the server, the client, a save and a panel always agree about
which container is which.

- A custom container is never the main bag (`bIsMainInventory` is off).
- Several can sit on one actor side by side, each with its own tag and capacity.

## Adding one, with no C++

1. **A tag.** Declare it like any gameplay tag (Project Settings, or a tag `.ini`).
2. **A Blueprint.** Create a Blueprint Class whose parent is **Snap Custom Container**, and fill in its class defaults
   above.
   - Saving it checks the definition: no tag, or no name, is an error.
3. **An attach row.** Make a container registry of your own:
   - Its **Included Registry** is `DA_SnapContainerRegistry`.
   - Its one row names your Blueprint, with `bAddUnique` **off**.
   - Point **Container Registry** (**Project Settings → Plugins → Snap Inventory System - Foundation → Attachment**) at it.
   - The shipped rows come first, yours after, and a later change to the shipped rows reaches you with nothing to copy.
   - A registry that lists every row itself works too, and so do the settings rows. Either way, your row goes **after**
     the main bag's — see [Inventory and containers](inventory-and-containers.md) for why the order matters.
   - A project that attaches by hand with `EnsureSnapInventoryComponents` reads the same rows.
4. **Items.** Give each item that belongs there a `USnapItemFragment_ContainerRouting` with `Mode = Container` and
   `ContainerTag` = your tag.
   - A container with a tag refuses anything not routed to it, so a keyring stays a keyring.
5. **A panel.** Put `WBP_ExampleContainerPanel` (or any `USnapInventoryPanel`) in your screen, and set its `ContainerTag` to
   your tag.
   - It binds itself, and hides while the player has no such container.
   - It titles itself with the container's `DisplayName` through `ContainerNameTextWidgetName` — one panel Blueprint
     serves every container.

⚠ **A custom container with no tag refuses every item.**

- An empty tag means "the main bag" to every routing path, so an untagged container would otherwise take ordinary items
  from the real bag.
- It says so once in the log, and saving the Blueprint reports it.

## Abilities while held: the Spells example

**With `bActivateHeldItemAbilities` on:**

- Every item on the container's own grid grants the abilities its Granted Ability Set fragment lists in
  `ContainerAbilities`.
  - It grants them at a level equal to its stack count (summed across entries of the same ability).
  - The grant is cleared when the item leaves.
- An item inside a pouch the container holds grants nothing, and neither does a broken or spoiled one.
- The grant names the **container** as its source.
  - That is how the hotbar's Ability bar runs the container's copy of an ability that a worn item also grants.
- The Ability bar draws its items from the main bag and every container with this switch on.
- The grants are made on the server and replicate to the owning player.
- **Get Held Ability Level** (`USnapContainerHeldAbilities::GetContainerHeldAbilityLevel`) reads the level a container
  confers, from the same code the grant uses.

**Without GameplayAbilities,** nothing is granted (one Log line says so).

- But the switch still makes the container one the Ability bar draws from: an `Event` item that ticks
  `bAllowOnAbilityBar` sits on the bar from it.
- The shipped Spells row is skipped before anything loads it
  (`USnapFoundationSettings::GetGameplayAbilitiesOnlyComponentClasses`).
- A project that wants a Spells container in that build attaches a class of its own, as the demo does below.

**The demo's spell scrolls are the worked example.**

- `ID_Example_SpellScroll_Haste` and the other four each carry two fragments that make them spells:
  - A Granted Ability Set whose `ContainerAbilities` names the spell.
  - The demo's spell fragment (`USnapDemoSpellFragment`, an `Event` Hotbar fragment that ticks `bAllowOnAbilityBar` and
    holds the spell's numbers).
- **Picked up,** a scroll lands in the bag.
- **LEARNED,** it moves into the demo's spellbook (below) and, with GameplayAbilities, its spell is granted there.
- **On a hotbar square,** it is cast by that square's key on the demo's active route, and the square counts down the
  spell's cooldown.
- **The demo attaches its own Spells container in both builds,** `BP_Example_DemoSpellsContainer`.
  - It is a Blueprint of the demo's `USnapDemoSpellbookComponent`, tagged `SnapInventorySystem.Container.Spells`, on no
    GameplayAbilities-only list.
  - It is attached through `DA_ExampleDemoContainerRegistry`, which lists the default rows itself with it in place of
    the shipped Spells container.
  - Each build attaches exactly one Spells container.
- See [The spell scrolls](demo-map.md#the-spell-scrolls).

## A container that takes anything, saved on its own: the Bank example

The example demo's personal bank is a custom container like the others, with four class defaults set differently:

| Class default | Bank | Why |
|---|---|---|
| `AdmissionMode` | AdmitAll | It takes any item, not only items routed to it. Routing still decides where a PICKUP goes — by the item's own container tag — so a container that admits everything is never where a pickup lands, and no container ever admits currency. |
| `bSaveWithActorInventory` | Off | It keeps its own save: see [Persistence](persistence.md#a-container-that-keeps-its-own-save). |
| `SlotCountOverride` | 24 | |
| `bRequireAccessSession` | On | It is used only at its chest: see [the reach rule](#a-container-used-only-where-it-is-kept-the-reach-rule). |

**Moving items between your own containers.** Dragging an item from one container's grid onto a square of another
container on the same player MOVES it there (`USnapInventoryComponent::MoveItemsToContainer`, sent as
`Server_MoveItemsToContainer`).

- It goes onto that square when it is empty, otherwise onto the first free one.
  - There is no merge and no swap between containers.
- It moves only when the other container admits it and has room (`CanMoveItemToContainer` says why not).
  - A container that admits only items routed to it — Spells, Shards, the Keyring — still refuses an ordinary item.
  - A pouch that still holds something stays where it is.
- The moved item keeps everything about itself: its identity, its rarity, its marks.
- It checks ownership, admission and room — and, for a container that asks for it, where the player stands (the reach
  rule, below).

**A container open beside the bag gets the quick gestures too.**

- Register it while its window is up — `USnapInventoryHUDComponent::SetOpenTransferContainer` (or
  `SnapSetTransferContainer` on `ISnapScreenHost`), cleared with `ClearOpenTransferContainer`.
- Shift+click and the right-click menu then move items to and from it with the same rules.
- The plugin clears it when the bag screen closes or the pawn changes.
- The demo's bank registers itself (`SnapDemoBank::SetBankOpenForTransfers`).
- See the README's storage section.

**More about the demo's bank:**

- A death keeps the bank because the demo's death-drop policy (`BP_Example_DemoDeathPolicy`) lists its tag in
  `KeptContainerTags`.
- Crafting never takes from it unless `bCraftingConsumesFromSiblingContainers` (**Snap Inventory System - Inventory → Crafting**) is
  on.
- The bank's window is a panel (`WBP_ExampleContainerPanel` bound to the bank's tag) inside a `BankColumn` in the example
  bag screen.
  - It is Collapsed as authored, and shown by the demo's controller while the player stands at the bank chest.

⚠ A drop onto a SPATIAL grid from another container is refused in this release. The demo's bag uses one item per square.

## A container that keeps what enters it: the spellbook example

**Tick `bKeepsWhatEnters`** (Keeps What Enters) in a custom container's class defaults, and an item that is in it **stays
there**.

- The player can still rearrange it inside — move, swap, split, merge, sort, rotate, the pouches it holds — and bind it
  to a hotbar.
- But every request that would take it out is refused with the notice "That stays in the container that keeps it."
  (`ESnapRefusal::KeptInContainer`, naming the item), and nothing moves:

| Refused | How |
|---|---|
| Moving it to another container | The drag, a Shift+click to an open container, the menu's "Move to" row (`CanMoveItemToContainer`) |
| Dropping, selling, trading, dismantling it, the bulk sells and salvages | `DescribeRemoval`, which the menu, the drag picture and every one of those doors ask |
| Using it up, removing it | `Server_ConsumeItem`, `Server_RemoveItem` (and the menu never offers Consume on it) |
| Spending it | As a mix ingredient (`DescribeMix`), a repair kit (`DescribeRepairKitUse`), a craft ingredient or a repair material (it is never a source) |
| Losing it in a death | The death drop and the body it fills leave it with the player |

- A save restores it like everything else.
- It is off as shipped, so every container behaves exactly as before until you turn it on.
- **An item kept here can look different** from the same item anywhere else.
  - Give its `UIData` fragment a **Learned Icon**, and its square, its hotbar square and its drag picture draw that while
    it is kept (`GetInstanceIcon`).
  - The example spell scrolls do.
- The answers are the container's class default and its replicated entries.
  - So the menu and the drag picture show the X on the player's own machine before anything is sent.

**Dragged OUTSIDE every window,** such an item shows the red X too (`USnapInventoryDragDropOperation::DescribeWorldDrop`,
asked by the screen while the cursor is over the world).

- Letting go there is a CANCEL: nothing is sent and no sound plays (`RequestWorldDrop` returns false).
- The same holds for anything else the world would refuse — an item that may not be dropped, a locked one.

⚠ **It governs what a PLAYER may ask for**, exactly as the reach rule does.

- Server code you write yourself — `RemoveItem`, `ReleaseInstance`, `ImportRecord` — is trusted.
- So a project can still take a kept item away on purpose (a "forget this spell" of its own).
- `USnapInventoryComponent::DoesKeepWhatEnters`, `FindContainerHolding` and `IsKeptByItsContainer` answer the question
  for your own code.

**A container can hear what a player moved into it.**

- `USnapInventoryComponent::OnItemsMovedIn` (a Blueprint Native Event) runs on the server, on the DESTINATION.
- It runs after `MoveItemsToContainer` has moved items into it from another container of the same owner.
  - The drag between two containers, a Shift+click to an open container and the menu's "Move to" row all go through it.
- A pickup routed there, a save being restored, `AddItem` and `AdoptInstance` never call it.

**The example demo's spellbook is the worked example.** Its Spells container is `BP_Example_DemoSpellsContainer`, a
Blueprint of the demo's `USnapDemoSpellbookComponent`, which:

- Keeps what enters.
- Admits only spell scrolls it does not already hold (a second copy is refused with `ESnapRefusal::AlreadyKnown`, "You
  already know that.").
- Raises the "Learned Fireball" feed line from `OnItemsMovedIn`.

See [The spell scrolls](demo-map.md#the-spell-scrolls).

## A container used only where it is kept: the reach rule

A container can require an **access session**: tick `bRequireAccessSession` (Require Access Session) in its class
defaults.

- It is off as shipped, so every container behaves exactly as before until you turn it on.

**With it on,** every request a player sends that would move an item **into or out of** the container is refused, unless
that player has a live session on it.

- The notice reads "That container can only be used where it is kept." (`ESnapRefusal::NotAtContainer`).
- No item is named: the item may be in the bag and perfectly reachable. It is the container the player is away from.
- Nothing moves.
- The server decides. A modified client cannot talk its way past it.

**A session is opened by server code at a place.**

- `BeginAccessSession(Player, Anchor, MaxDistance)` records that this player may use the container while the pawn that is
  standing there now stays within `MaxDistance` of `Anchor` (an actor: your chest, your banker).
- It refuses, opening nothing, when the player has no pawn or that pawn is already farther away.
- Calling it again refreshes the session.
- ⚠ It trusts its caller about WHICH place this is.
  - So a door that lets a client name the anchor checks first that it really is this container's place.
  - The demo's bank RPCs take a bank chest and read its range from the server's own copy.

**A session ends:**

- When `EndAccessSession(Player)` is called (your window closing).
- When the pawn that opened it walks out of range, is replaced (a death and respawn) or is gone.
- When the player leaves.
- When the anchor is gone.
- For the owner's own containers, when the plugin's death drop runs.

A server watch at `ProximityCheckInterval` (**Snap Inventory System - Inventory → Screen**, 0.25 s) ends them. Every request is measured again at the moment it arrives, so nothing
is decided on a stale position.

**The requests that ask it:**

- `MoveItemsToContainer` at both ends — so the grid's drag between two containers, `Server_MoveItemsToContainer`, and any
  deposit your own server code makes for a player.
- Every client request that takes the container's own items away:
  - `Server_ConsumeItem` (a double-click, the menu's Consume, a hotbar key).
  - `Server_RemoveItem`.
  - `Server_DropItemsToWorld`, `Server_DropItemStacksToWorld`, `Server_DropStoredItemToWorld`,
    `Server_DropStoredItemsToWorld`.
  - `Server_DismantleItem`, `Server_DismantleItems`, `Server_SalvageByRarity`.
  - `Server_SellToVendor`, `Server_SellAllJunk`.
  - `Server_OfferItemsToTrade`.
  - `Server_ApplyRepairKit`, `Server_FieldRepairItem` and `Server_CraftRecipe`.

Crafting from sibling containers (`bCraftingConsumesFromSiblingContainers`) leaves out, on the server, a container this
rule refuses.

**What never asks it**, by design:

- Moving items **inside** the container — dragging, swapping, splitting, merging, sorting, rotating, and the pouches it
  holds.
- Marking an item, and a vendor's repair or upgrade of an item where it lies.
- Items the game routes **into** it — a pickup, loot, a purchase, a craft's result — and the developer-only adds.
- **Every call your server code makes itself**: Persistence (`ImportRecord`, `ExportRecord`), a death-drop policy,
  `AddItem`, `RemoveItem`, `ReleaseInstance`, `AdoptInstance`.
  - That is what keeps a save loading, a death dropping and your admin tools working.
  - The rule governs what a client may ask for, and your server code is trusted to have checked what it does.

⚠ **Not on the main inventory.**

- The equipment, the hotbar's binding and crafting use the main bag directly and do not ask the rule.
- So it is ignored there (`IsAccessSessionRequired`), and `IsDataValid` warns where it was set.
- Put it on a container beside the bag.

⚠ **Sessions are server state and are not replicated.**

- A client cannot see one (`HasAccessSession` reads false there).
- So your UI closes the container's window by its own distance test, with the same range.
- The drag's own answer (`CanMoveItemToContainer`) does not ask the rule.
  - A client that said "no" to a move the server would allow would be hiding something the player may do.
  - So the client offers it, and the server's refusal and notice cover the rest.

⚠ A consume routed through your own ability (`ConsumeAbilityTag`) still spends the item through `Server_ConsumeItem`,
which refuses.

- But an effect that ability applies by itself before the spend is the ability's to gate.

**Your own server function** that moves items into or out of a container for a player asks `CheckAccessFor(Player)` first
— `None` means go ahead — exactly as the plugin's doors do.

**The demo's bank uses it.** `BP_Example_BankContainer` has it on.

- The bank chest's **Open** shows the bank's panel on the player's machine and asks the server to open the session at that
  chest (`ASnapDemoPlayerController::Server_OpenBankAt`).
  - It is measured with the chest's `BankReach` (450 cm) — the same distance the panel (and the bag the bank opened) closes at, so the two halves agree.
- **Deposit everything** opens it too before it deposits (`Server_DepositEverythingInBank` names the chest).
- Closing the panel, walking away or dying ends it (`Server_CloseBank` ends it and saves the bank).
