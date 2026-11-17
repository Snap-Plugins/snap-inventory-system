# Reference

One page per subsystem, plus three appendices generated from the headers. Use this page to find the one you need.

## Start here

- **New to the plugin?** Start with the [Quickstart](../QUICKSTART.md). It puts a working bag in your project.
- **Want to see it working first?** Open [the demo map](demo-map.md).
- **Looking for something else?** The [documentation index](../README.md) lists every page.

## I want to ...

| I want to ... | Read |
|---|---|
| Make my own item | [Items and fragments](items-and-fragments.md) |
| Make an item several squares big | [Make a grid item](make-a-grid-item.md) |
| Add storage of my own beside the bag | [Custom containers](custom-containers.md) |
| Let players wear items | [Equipment](equipment.md) |
| Change how the screens look | [UI and theming](ui-and-theming.md), then [Customizing the UI](customizing-the-ui.md) |
| Change or rebind the keys | [Input](input.md) |
| Set up the hotbar | [Hotbar](hotbar.md) |
| Add a shop | [Vendor](vendor.md) and [Currency and prices](currency-and-prices.md) |
| Add crafting | [Crafting](crafting.md) and [Crafting stations](crafting-stations.md) |
| Put loot and pickups in the world | [Loot and interaction](loot-and-interaction.md) and [Loot tables](loot-tables.md) |
| Let players trade | [Trade](trade.md) and [Trading with NPCs](npc-trade.md) |
| Save and load what players carry | [Persistence](persistence.md) |
| Translate or reword the plugin's text | [Localizing and rewording](localization.md) |
| Use it with Lyra or the Gameplay Ability System | [GAS and Lyra](gas-and-lyra.md) |
| Check what my project has not set up yet | [Editor tools](editor-tools.md) (the Setup Check) |
| Find out why an action was refused | [Refusals](refusals.md) |
| Test or debug it | [Testing and diagnostics](testing-and-diagnostics.md) |
| Look up a setting, a widget name or an event | The [appendices](#appendices), below |

## Every page, by topic

### Getting started

| Page | Covers |
|---|---|
| [The demo map](demo-map.md) | The playable example, its four classes, and the recipe for a non-Modular project |
| [GAS and Lyra](gas-and-lyra.md) | Ability system integration, Lyra specifics |

### Items

| Page | Covers |
|---|---|
| [Items and fragments](items-and-fragments.md) | Definitions, instances, the twenty-six fragments, writing your own |
| [Make a grid item](make-a-grid-item.md) | A walk-through: an item several squares big, its shape, its turns, trying it in the demo |
| [Affixes](affixes.md) | Rolled per-instance modifiers, the affix table, weights, how a roll is shown and priced |
| [Item sets](item-sets.md) | Set bonuses, cumulative tiers, what counts as a worn piece |
| [Durability, spoilage, weight](condition-and-weight.md) | Wear, breaking, repair, shelf life, carry weight |
| [Restrictions and marks](restrictions-and-marks.md) | Locking, favorites, junk, the seven removal doors, bulk verbs |
| [Tooltips and stats](tooltips-and-stats.md) | What an item says it does, the stat display table, comparing against what is worn |

### Containers, equipment and the hotbar

| Page | Covers |
|---|---|
| [Inventory and containers](inventory-and-containers.md) | The grid, stacking, pouches, counters, admission, capacity |
| [Custom containers](custom-containers.md) | Your own storage beside the bag (Spells, Shards, anything) as a data-only Blueprint: tag, name, capacity, abilities while held |
| [Equipment](equipment.md) | Slots, equipping, granted abilities, ownership, wear |
| [Hotbar](hotbar.md) | Two bars, pages, what may be bound, persistence |

### Screens, input and sound

| Page | Covers |
|---|---|
| [UI and theming](ui-and-theming.md) | Widget contracts, reskinning, floating windows |
| [Customizing the UI](customizing-the-ui.md) | The example Widget Blueprint behind each surface, its parts, moving and restyling them |
| [The buff row](buff-row.md) | A player's running buffs and debuffs as translucent squares on the HUD, on both routes: the widget, the source interface, the rows |
| [Input](input.md) | Mapping contexts, UI gestures, raw bindings |
| [Sounds](sounds.md) | Item, container, action and interface sounds: the type rows, per-item overrides, the theme's sounds, the switch |
| [Localizing and rewording](localization.md) | Gathering the plugin's text, translating it, rewording the English, identifier fallbacks |

### Shops, money, trade and crafting

| Page | Covers |
|---|---|
| [Vendor](vendor.md) | Stock, buying, selling, upgrades, repair |
| [Currency and prices](currency-and-prices.md) | What money is, per-shop currencies, the price hooks, paying |
| [Trade](trade.md) | Escrowed player-to-player trade |
| [Trading with NPCs](npc-trade.md) | Any actor a player trades with in the trade window: the trade-partner component, its offer, the auto-confirm |
| [Crafting](crafting.md) | Recipes, dismantle, discovery, timed queues |
| [Crafting stations](crafting-stations.md) | A forge that is not a shopkeeper, the station session, one station-tag derivation |
| [Quest givers](quest-givers.md) | Any actor that takes in items for rewards and slot unlocks: the quest asset, the giver component, the "!" |

### The world

| Page | Covers |
|---|---|
| [Loot and interaction](loot-and-interaction.md) | Pickups, drops, containers, bodies, the interact key |
| [Loot tables](loot-tables.md) | Shared loot assets, nesting, guaranteed rows, seeded rolls |
| [Pickup modes and the collect feed](pickup-modes-and-the-collect-feed.md) | Walk or press to collect, the collect seam, the stacking feed, the presentation fragment |
| [Death and loss](death-and-loss.md) | What a death drops, the death-drop policy, the doors for a project without GAS |

### Tools, saving and the internals

| Page | Covers |
|---|---|
| [Editor tools](editor-tools.md) | The Snap Inventory New menu, item thumbnails, painting a footprint, the Setup Check, registering items with the Asset Manager |
| [Testing and diagnostics](testing-and-diagnostics.md) | The automation tests, the gameplay debugger page, the console commands, the stats group |
| [Refusals](refusals.md) | `ESnapRefusal`, how a refused action explains itself |
| [Networking](networking.md) | Authority, replication, what a client may compute |
| [Persistence](persistence.md) | The provider seam, records, versioning, world state |
| [Extending the plugin](extending.md) | Settings, admission policies, subclassing, fragment hooks, your own HUD |

## Appendices

Generated by `Tools/generate_docs.py`, which is in the plugin's source repository rather than the packaged plugin.
Do not edit them by hand.

| Appendix | Rows |
|---|---|
| [Settings](appendix-settings.md) | 380 settings in 67 categories, across the eleven Snap Inventory System pages, and the 46 fields of the 11 structs some of them hold |
| [Widget contracts](appendix-widgets.md) | 252, across 39 widget classes and three headers of shared types |
| [Events](appendix-events.md) | 53 delegates, across 26 classes |
