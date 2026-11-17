# Changelog

All notable changes to the Snap Inventory System plugin. Versions follow `VersionName` in
`SnapInventorySystem.uplugin`.

## Deprecation policy

From 1.0.0 on, a public name a project can reach is not removed in the release that retires it.

- **A public name:** A class, a Blueprint function or property, a config key, a widget name the plugin looks up, a
  gameplay tag.
- **The release that retires it:** That release keeps it working, marks it and lists it under **Deprecated**.
  - The mark is a `DeprecatedProperty` or `DeprecatedFunction` meta, or `UE_DEPRECATED` in C++.
  - A rename is marked with a redirect.
- **Removal:** It is removed no sooner than the next minor release, and listed there under **Removed**.

## 1.0.0 — Initial release

First public release.

**Replicated, server-authoritative grid inventory and equipment for Unreal Engine 5.8:**

- Data-driven item definitions composed from fragments.
- Exclusive-custody equipment with Gameplay Ability System grants.
- Multiple containers (your own as data-only Blueprints) and nested storage items.
- A multi-denomination wallet.
- Vendors (buy / sell / repair / upgrade / craft).
- Timed crafting queues with recipe discovery and dismantling.
- Player-to-player escrow trade.
- World pickups, loot containers and lootable bodies.
- A plugin-native interaction system.
- A paged hotbar with Enhanced Input keys.
- Durability, spoilage and carry weight.
- Floating windows.
- An always-on HUD mirror.

**It ships with a playable demo map:**

- Epic's UE5 mannequin as the character.
- A model of its own for every example item.
- Three shops in three currencies.
- Castable spell scrolls.
- Heal and damage pads.
- A death you can come back from.
- Attacks against training dummies: a punch, a weapon in either hand, a shield that blocks.

In a copy compiled with GameplayAbilities it brings its own settings for the play session, so it plays the same in any
project, and a wing of pads switches the plugin's features on and off.

### What is in the box

- **Items are data.**
  - An item is a `USnapItemDefinition` carrying composable fragments: stacking, rarity, equippable, granted
    abilities and effects, container routing, durability, spoilage, weight, UI data.
  - Magic items roll modifiers from an affix table.
  - Most items need no C++.
- **Containers.**
  - A main bag plus containers beside it, each a data-only Blueprint and one registry row.
  - Containers inside containers (a pouch, a chest, a quiver).
  - Each container is a numbered bag or a spatial grid.
  - The bag can show the room its upgrades could still add, as locked squares.
- **Equipment.** With exclusive custody, ability and effect grants, item sets, visuals attached to sockets, and a
  paper doll.
- **Currencies, shops, crafting and trade.**
  - A multi-denomination wallet.
  - Vendors that buy, sell, repair, upgrade and craft, with restock rules.
  - Crafting stations with timed queues.
  - Player-to-player escrow trade and trade with any actor.
- **The world.**
  - Pickups, loot containers, lootable bodies and world drops that expire and come back.
  - The plugin's own interaction system.
- **Input and screens.**
  - A paged hotbar, Enhanced Input or raw keys, and gamepad focus navigation.
  - A themed UI with floating windows, item cards, a buff and debuff row, a collect feed and sounds.
- **Saving and loading.** Including world state, behind a provider you can replace.
- **Content in three parts.** `Core` holds what every project uses and is cooked by one line; `Examples` the example
  items and actors; `Demo` the demo map and what only it uses, with its art in `DemoArt` and the mannequin in
  `EpicDemoContent`.
- **Tooling.**
  - Eleven Project Settings pages (Snap Inventory System - Foundation, Items, Inventory and the rest), every setting
    documented in a generated appendix.
  - A Setup Check.
  - Automation tests.

### Requirements

- **Engine:** Unreal Engine 5.8.
- **Required plugins:** `ModularGameplay` and `EnhancedInput`, engine plugins declared in the `.uplugin`.
- **Optional plugins:** `GameplayAbilities` and `CommonUI`.
  - A copy installed into the engine is used as it was compiled, which is expected to be the build without both.
  - See *Without GameplayAbilities* and *A precompiled copy* in the [README](README.md).

### A promise about configuration

- **What the plugin's own file sets:** The plugin's own `Config/Game.ini` assigns nothing except `AvailableSlots`, and
  it stays that way.
- **Why:** A plugin's `Game.ini` is applied on top of a project's own config. So any other key there would silently
  beat your `DefaultGame.ini`.
- **Where your settings go:** Every other setting of the plugin belongs in your project's `Config/DefaultGame.ini`.

### Known limitations

The README's [Known limitations](README.md#known-limitations) section names what is verified, what is partially
verified and what is not verified at all. Read it before you rely on a dedicated server or a gamepad.

### Upgrading

Nothing to upgrade from: this is the first release.

- **A window subclass made against a pre-release copy:** `HandleWindowCloseStateChanged` (the close box's hover and
  press handler) is declared once, on `USnapWindowWidget`, and `SetSearchQuery` once, on `USnapItemGridHost`, each as
  a virtual `UFUNCTION`.
  - A C++ subclass that declares its own `UFUNCTION()` of either name no longer compiles: Unreal Header Tool refuses a
    `UFUNCTION()` on an override. Drop it and declare the function `virtual ... override`.
  - A Blueprint subclass cannot add a function of its own with either name.
