# Sounds

The plugin plays a sound at each of the moments below. Every one of them can be replaced, and one switch turns them all
off.

- **Items:** When an item is lifted, put down, collected, used, put on or dropped.
- **Containers:** When a pouch, a chest or a body opens or a crate breaks.
- **Actions:** When a spell is learned, two items are mixed or an item is repaired or dismantled.
- **The interface:** When a button, a tab, a check box or a window is used.

Sounds are **on by default** and point at the plugin's own example sounds, which are made by a script (no recorded or
third-party audio).

**On this page:**

- [The switch and the volume](#the-switch-and-the-volume)
- [Item sounds: by type, with per-item overrides](#item-sounds-by-type-with-per-item-overrides)
- [Containers](#containers)
- [Actions](#actions)
- [Interface sounds: part of the theme](#interface-sounds-part-of-the-theme)
- [Who hears what](#who-hears-what)
- [Replacing a sound](#replacing-a-sound)
- [Seeing what played](#seeing-what-played)

## The switch and the volume

**Project Settings > Plugins > Snap Inventory System - UI Core > Sound**

| Setting | What it does |
|---|---|
| `bPlaySounds` (on) | Off: the plugin plays nothing. A sound your own code plays is yours and is not affected. |
| `SoundVolume` (1) | Multiplies every sound the plugin plays. 0 is silent. |
| `WorldSoundInnerRadius` / `WorldSoundFalloffDistance` (300 / 2200 cm) | How far a world container's sound carries (below). |

- Everything the plugin plays goes through one player, `USnapInventorySoundSubsystem`.
  - So the switch and the volume apply to all of it alike.
  - So does an **art pack**: a `USnapArtPack`'s `Sounds` (an old sound and its replacement) swaps any of them in the
    world it is registered in, the way its `Meshes` and `Icons` swap models and icons (README, "The demo's art pack").
- A replacement that does not load plays the authored sound.

## Item sounds: by type, with per-item overrides

An item has three sounds:

| Sound | When it plays |
|---|---|
| **Lift** | A drag picks it up, or it is picked up from the ground — one pick-up sound for both. With the bag open, the landing in its square is not heard over it. |
| **Place** | It lands in a square — a move, a swap, a merge, a store, a take from loot, a purchase, a craft — or is put on or dropped. |
| **Use** | It is used, at the moment the use lands. |

`ItemTypeSounds` chooses them **by the item's type**:

- The row whose `ItemType` the item's `ItemTags` carry (or a child of it) gives the item its sounds.
- The most specific matching row wins.
- An item no row matches — or a field a matching row leaves empty — takes `DefaultItemSounds`.

The plugin ships one row per `Item.Type` tag it declares:

| Item type | Lift | Place | Use |
|---|---|---|---|
| `Item.Type.Weapon` | A blade drawn | A blade set down | — |
| `Item.Type.Armor` | Plates rattling | A plate set down | — |
| `Item.Type.Accessory` | A small chime | A glassy clink | — |
| `Item.Type.Consumable` | A slosh | Glass and a slosh | Two gulps |
| `Item.Type.Material` | A wooden knock | A wooden clack | — |
| `Item.Type.Container` | Leather | A soft thump | — |
| `Item.Type.Currency` | A jingle | Clinking coins | — |
| `Item.Type.Quest` | A paper flick | Paper settling | A paper unroll |
| `Item.Type.Tool` | A heavy scrape | A heavy set-down | — |

- Your own rows replace these.
- A Use the row leaves empty falls back to the item's Place sound.

**One item's own sound:** Add a `Presentation` fragment to the item and set the field for that moment: `LiftSound`,
`PlaceSound`, `ConsumeSound` (its use), `PickupSound`, `EquipSound`, `DropSound`, `BreakSound`.

- A field left empty means "the type's sound".
- `PickupSound` falls back to the item's Lift sound, `EquipSound` and `DropSound` to its Place sound; `BreakSound` to
  the Break action sound.
- Turning on `bPlayUseSoundWhenUseStarts` plays the item's use sound as a timed use STARTS rather than when it lands.
  - A potion's cork pop belongs at the start of the drink (the example potions turn it on).
  - An instant use still sounds once, when it lands.
- A drop to the world plays the item's Drop sound once per gesture, on the dropping player's machine as the request
  leaves.
  - The gestures: the item menu's Drop, DROP INVENTORY, a drag out of the window, the split window's drop, a click on
    the money.
  - When several go, it is the first item's sound.
  - A drop the server then refuses also plays the refusal.
- An item whose durability reaches zero plays its `BreakSound` (else the Break action sound) on its owner's machine when
  the server reports the break.
- `USnapInventorySoundLibrary::ResolveItemSound` answers the whole order for an item and a moment.

Several example items carry their own sounds this way:

| Example items | Their sounds |
|---|---|
| The recipe scroll, the spell scrolls and the sealed letter | Paper — the recipe scroll's use is the Read sound |
| The meat, the bread and the rotten mush | Food — eating the meat or the bread is the Eat sound |
| The gem and the arcane shard | Gems |
| The gold bar | Heavy |
| The bag upgrades | Leather |
| The cooler and the carved totem | Wood |

## Containers

- **A pouch** (an item with an `ItemStorage` fragment): Its window plays `StorageOpenSound` / `StorageCloseSound`
  (**Snap Inventory System - UI Core > Sound**), or the fragment's own `OpenSound` / `CloseSound` when set.
- **A chest or a lootable body** (`ASnapLootContainer` and its subclasses): `OpenSound` / `CloseSound` on the
  actor, played **at the container**, so they fall off with distance.
  - A chest ships with the example chest sounds and a body with cloth; clear them for silence.
  - The player who opens it hears the opening when the loot window opens.
  - Other players nearby hear it the **first** time the container is opened.
  - What reaches them is the container's opened latch, which is set once and cleared by a respawn.
  - So a second opening is heard by the opener alone.
- **A crate** (`ASnapDestructibleContainer`): `BreakSound` on the actor (the example crate's break by default)
  plays **at the crate** when it breaks, for the players near it.
  - A crate that reaches a player already broken — walking into relevance, a restored save — is silent.
  - With `bDestroyOnBreak` a player on another machine may not hear it: the crate can be gone before its broken state
    arrives.

## Actions

`ActionSounds` holds the sounds of what a player does with an item.

- **Learn**, **Mix**, **Repair**, **Dismantle** and **Craft** play when the server reports the action
  (`USnapInventoryComponent::OnItemActionPerformed`).
  - So a timed use (`ItemUseSeconds`) sounds when it lands, not at the press, and a timed craft as each unit comes out.
  - A use plays the item's Use sound the same way.
- An item breaking is reported the same way and plays the item's `BreakSound`, else **Break**.
- **Buy** and **Sell** play when the shop's BUY or SELL is pressed and the request goes to the server.
  - SELL JUNK plays Sell when the server reports that something sold.

**One sound per action.** An action's sound takes the place of the moved item's landing tick in the same moment:

- Learning a scroll plays the Learn sound and not the scroll's Place in the spellbook.
- A mix plays Mix and not its result's Place.
- A craft plays Craft and not the made item's Place.
- A worn piece that breaks and is moved to the bag plays its break and not a Place.

When both reach the player's machine in the same moment, in either order, one is heard:

- A Place that already started is stopped while it is still playing.
- One asked for after the action is not played.
- A Place that has already finished cannot be taken back. So on a remote client a drag's release (which plays its Place
  as the request leaves) can still tick before a learn arrives when the round trip is longer than that Place.
- The landing still bounces; only the tick goes.
- Other items that land in the same moment — a dismantle's materials — still tick.

A project's own action can do the same with `USnapInventorySoundLibrary::ClaimItemMoveSound`. An action whose sound you
empty keeps the tick.

## Interface sounds: part of the theme

The interface sounds are in the theme (`USnapInventoryTheme::UISounds`):

| Sound | Plays for |
|---|---|
| **Click** | Every button the plugin draws |
| **Tab** | A filter row |
| **Toggle** | A check box |
| **Window Open** / **Window Close** | The bag, a vendor, the craft window, a trade |
| **Refused** | A red release, a refusal notice, a gamepad put-down turned away |

The theme also holds **Hotbar Select** and **Hotbar Page** (a page step). The stored-hotbar restore that puts a saved
page and its bindings back is silent, and its squares do not play a place sound; a project's own widget can tell it
apart with `USnapHotbarComponent::IsAnnouncingRestore`.

- A gamepad carry sounds like a drag: the item's Lift when it is picked up, and its put-down plays what a pointer
  release on that square would.
- With no theme picked, the plugin's example sounds play; a theme that does not set them sounds like the plugin.

Binding the sounds to buttons:

- A plugin widget binds its own buttons and check boxes when it is built.
- A project's own widget that derives from `USnapUserWidget` gets the same.
- For a button built later, call `BindInterfaceSound`.

## Who hears what

- Item, interface, pouch and action sounds are **2D and local**: they play on the machine of the player who did the
  thing, for that player only, and keep playing while the game is paused.
- A world container's sound is **3D, at the container**, and is heard by players near it.
- One moment reaching the plugin twice plays once:
  - A drag's release and the item landing in its new square.
  - A pickup and its landing in the bag.
  - One landing seen by the bag and the hotbar.
  - An action and the landing of the item it moved.
- No more than four plugin sounds start in one frame.
- `GetSupersededCount` / `DescribeCounts` count a Place an action's sound stopped.
- A dedicated server plays and loads nothing.

## Replacing a sound

Point the setting, the row, the theme field or the item's field at your own Sound Wave or Sound Cue.

- The plugin's default sounds live in `/SnapInventorySystem/Core/Audio`.
- ⚠ They are named only by C++ defaults, so a packaged build cooks them only when `DirectoriesToAlwaysCook` covers
  them: the README's one `/SnapInventorySystem/Core` line does.
  - That is the same rule as the filter icons in `Core/UI/Textures`.
- A sound that does not load logs one Warning naming its path and is not played.

## Seeing what played

- `Log LogSnapInventorySound Verbose` logs one line per sound played (and one per sound the switch silenced).
- `USnapInventorySoundSubsystem::GetPlayCount` / `DescribeCounts` count them by event (`ItemPlace`, `UIClick`,
  `ContainerOpen`, ...).
