# The demo map

A playable example you can open, press Play, and use. It exists so that "does this plugin do what it says" is a
sixty-second question rather than an afternoon of setup.

This page is a guided tour of the map: what is in each area, what to try there, and how each part is built, so you can
copy it.

**On this page:**

- [Open the map and press Play](#open-the-map-and-press-play)
- [What you can do in the first minute](#what-you-can-do-in-the-first-minute)
- [The HUD, the hotbar keys and the menu](#the-hud-the-hotbar-keys-and-the-menu)
- [Playing with a gamepad](#playing-with-a-gamepad)
- [The spell scrolls](#the-spell-scrolls)
- [The potions](#the-potions)
- [The heal and damage pads, and dying](#the-heal-and-damage-pads-and-dying)
- [Attacks, the shield and the training dummies](#attacks-the-shield-and-the-training-dummies)
- [Your character, and the sword in its hand](#your-character-and-the-sword-in-its-hand)
- [The demo's two routes](#the-demos-two-routes)
- [The classes, and the recipe they are](#the-classes-and-the-recipe-they-are)
- [An item you cannot wear yet](#an-item-you-cannot-wear-yet)
- [An item that looks like itself on the floor](#an-item-that-looks-like-itself-on-the-floor)
- [Gold, silver and gems](#gold-silver-and-gems)
- [The shops, the forge and the signs](#the-shops-the-forge-and-the-signs)
- [The bank, and a world that remembers](#the-bank-and-a-world-that-remembers)
- [Magic items](#magic-items)
- [Item rules: an item you cannot wear yet, and a quest item](#item-rules-an-item-you-cannot-wear-yet-and-a-quest-item)
- [The money changer](#the-money-changer)
- [Two players](#two-players)
- [Signs that teach](#signs-that-teach)
- [The FEATURES wing: switch a plugin feature for this session](#the-features-wing-switch-a-plugin-feature-for-this-session)
- [Filling a map with items](#filling-a-map-with-items)
- [What the demo map is not](#what-the-demo-map-is-not)
- [See also](#see-also)

## Open the map and press Play

1. In the Content Browser, select **All** and search for the map.
   - A copy in your project is under **Plugins**.
   - A copy installed into the engine is under **Engine → Plugins**. It is listed only after you tick
     **Settings → Engine Content**.
2. Open **`/SnapInventorySystem/Demo/Maps/L_SnapInventoryDemo`**.
3. Press Play.
   - You should see the welcome panel: the keys, where everything is, and the mouse's look speed and Invert Y.

The map names its own GameMode, and that GameMode wires everything the plugin needs.

**Where its files are.** The plugin's content is in three folders of its own, and the demo uses all three:

- `/SnapInventorySystem/Core` — what every project uses: the default widgets and their pictures, the sounds, the
  tables, the input, the effects, the container registries and the shipped themes.
- `/SnapInventorySystem/Examples` — the example items and actors you can copy: the items and their icons, models and
  sounds, the chests, the vendor stall and forge, the quest giver and trader, the spells.
- `/SnapInventorySystem/Demo` — what only the demo uses: this map (`Demo/Maps`), its character, game mode and
  controller (`Demo/Blueprints`), its HUD and menu (`Demo/UI`), its signs, pads and their pictures (`Demo/SignKeys`,
  `Demo/Props`, `Demo/FeaturePictures`), the training dummy, the currency icons and the combat sounds.
- `/SnapInventorySystem/DemoArt` holds the demo's own art, swapped in by its art pack, and `EpicDemoContent` the
  mannequin.

### With or without GameplayAbilities

- [The demo classes](#the-classes-and-the-recipe-they-are) are compiled with or without `GameplayAbilities`.
- They read health and stats through one layer with two routes (see [The demo's two routes](#the-demos-two-routes)).
- **In a copy compiled with `GameplayAbilities`, everything below works.**
  - That is the plugin in your project's own `Plugins/` folder, compiled with it enabled.
  - A Blueprint-only project compiles it when the editor offers to, on opening the project, so enable it first.
- **In a copy without it, the map's health, damage, death and respawn, the HUD, the pads, `K` and the training dummies
  work on the built-in route.**
  - An *Install to Engine* copy is compiled without it.
  - Your attacks go through the attack component (the second route in
    [Attacks](#attacks-the-shield-and-the-training-dummies)).
  - Worn items' stats, the iron set's bonuses, the potions and the ring's and amulet's passives count there too.
  - The five spell scrolls cast there (see [The spell scrolls](#the-spell-scrolls)).

### The demo's own settings

**It brings its own settings.** Press Play and, for that play session only, the demo switches this plugin to its own setup,
whatever your project has set, so it plays the same in any project:

- The example equipment slots.
- Its three currencies and their pictures.
- The hotbar on its HUD.
- The example rarity table.
- Its death.
- Fixed windows.
- The plugin's own neutral look (no UI theme, the plugin's own colors).
- Its own save slot.

It also sets each switch its FEATURES pads flip, so every pad starts on the demo's own side (item marks start off).

- **When the session ends every value goes back to yours**, and no setting is written to your project.
  - The demo writes only its own save files, `SnapInventoryDemo_<player>`, `SnapInventoryDemo_SnapWorld` and
    `SnapInventoryDemo_Mouse` (the look speed and Invert Y), in your project's `Saved/SaveGames`.
- ⚠ **While it runs, this plugin's settings pages in Project Settings show the demo's values. Don't edit them during
  Play.**
  - An edit made then is undone when the session ends.
  - An edit to a list setting (the currencies, say) saves the whole section, the demo's values with it, into your
    `Config/DefaultGame.ini`.
- Your developer keys (the example item spawner, the vendor and trade keys) stay as your project has them.
- The demo also attaches its player's containers from its own list, so a container your project adds to every player (a
  registry of your own) is not in the demo.
- The whole setup is one function, `ApplyDemoProfile` in `Demo/SnapDemoSession.cpp` (`USnapDemoSessionSubsystem`).
  - A game of your own built on `ASnapDemoGameMode` turns it off with **Apply Demo Profile**.
- ⚠ A key you rebind on the demo's hotbar is saved by Enhanced Input in your project's `EnhancedInputUserSettings.sav`,
  which the plugin does not choose.

## What you can do in the first minute

- You start at the head of one long avenue.
- On your right, the map's sections stand one after another, each on its own colored ground with its sign, in the order
  you kit up as you walk down it.
- On your left: the tall signs with the keys, then the FEATURES pads in one line, each block on its own colored ground.
- Each section's sign says what to do there.

### Before you walk

- **A welcome panel:** It opens when you start: the keys, where everything is, and the mouse's look speed and Invert Y.
  - **Start playing** (or `Escape`) closes it.
  - `Escape` brings the same panel back at any time.
- **The tall signs on your left,** the first and tallest in the map: **SNAP INVENTORY SYSTEM** on top, **BAG TIPS: MOUSE
  AND KEYBOARD** and **BAG TIPS: GAMEPAD** under it, **TWO PLAYERS** at the bottom.
  - They show the keys, each drawn as a picture of the key or of your controller's button. Read them first.
  - See [Signs that teach](#signs-that-teach).
- **The bag:** Press `I`.
- **Your health and stats:** Top left: health with its numbers, then Armor, Attack Power and Attack Cooldown.
  - Wear the sword, the Ring of Vitality or the iron armor and watch them change.
  - Above them, while anything is running, is the **buff row**: a square for Haste, Ward or a potion, each with its
    seconds left.
- **The hotbar:** Along the bottom of the screen: keys `1`–`0` in the middle, and the two potion squares, `Q` and `R`,
  just left of them.
  - Both show the hotbar's page, `1/3`, and page together.
  - A potion you pick up lands on the first free potion square by itself.
  - To put it (or a spell scroll) on a `1`–`0` square, open the bag and drag it there, or point at it and press the key.
  - The five potions and the five spell scrolls are the example items a bar accepts.
- **A hotbar key you would rather change:** Open the bag, point at a hotbar square and press the key you want.
- **The menu:** `Escape` when nothing else is open: Resume, Leave the demo, the mouse's look speed and Invert Y, the
  controls.
- **`K`:** You die where you stand, to try the death drop.
  - Everything you carry goes into a pack on the ground, and five seconds later you are back at the start.
  - Walk back and take it all; the pack goes once it is empty and you walk away.

### Down the avenue, section by section

**The sections down the avenue, on your right:** Walk over the items, or look at one and press `E`. Every one takes both.

- Each of the 43 example items lies in exactly one section (the three potions you make by mixing are made there).
- See [A route through the map](#a-route-through-the-map).

#### WEAPONS

- **The sword:** It equips itself into your character's right hand, blade forward.
  - A second one (the General Store sells it) goes into the left hand.
- **The axe, the greatsword and the shield beside the sword:**
  - The Iron Axe is one-handed: with a sword already in your right hand it goes into your left.
  - The greatsword takes both hands.
  - The shield goes on your off hand and blocks while you hold right click.

#### ARMOR

- **Five helmets under the HELMETS sign:** One helm in five rarities, Common first to Legendary last as you walk.
  - Wear one, then walk onto a rarer one: it swaps in.
  - The Armor at the top left goes up, and the head square in the bag glows in the new rarity's color.

#### JEWELRY AND A QUEST

- **The Amulet of Mending:** It heals you a little every second while you wear it, more the rarer it is.
  - Get hurt on the red pad and watch your health climb back.
- **The Courier:** Bring the Sealed Letter (it lies there too, under the **ITEM RULES** sign) and press the interact key.
  - You get a Ring of Vitality, and the second ring slot unlocks for good.
  - See [Quest givers](quest-givers.md).

#### POTIONS AND MIXING

- **The potions under the POTIONS sign:** The Potion of Regeneration, the Elixir of Might, the Ironskin Potion and the
  Tonic of Vigor, four of each, two Health Potions and two Arcane Shards.
  - That is enough to drink some and mix the rest.
  - Open the bag and double-click one to drink it (the same as right-click → **Consume**), and watch the stats at the top
    left.
  - See [The potions](#the-potions).
- **The two arcane shards:** One goes into your bag at a time: it is a crafting material.
  - The bag's **CRAFT** window turns one into two health potions.
  - The forge makes the shield from one and two gold bars.
  - The General Store's craft tab makes the Ring of Vitality from one.

#### HEAL AND DAMAGE PADS

- **A green pad and a red pad under the HEAL AND DAMAGE PADS sign,** in their own section after the potions:
  - Stand on the red pad with nothing worn, then wearing the iron set, and watch the line under your health bar.
  - Stand on it long enough and you die: walk back to your body and take your things back.
  - See [The heal and damage pads](#the-heal-and-damage-pads-and-dying).

#### SPELLS

- **Five spell scrolls under the SPELL SCROLLS sign:**
  1. Pick one up. It lands in your bag.
  2. Open the bag, right-click it and choose **Learn** (or double-click it). It moves into your Spells for good.
  3. Point at it there and press a hotbar key (or drag it onto a square).
  4. Close the bag and press that key.
  - See [The spell scrolls](#the-spell-scrolls).

#### COMBAT

- **Three training dummies under the TRAINING DUMMIES sign:**
  - Left click punches. Wear the sword and the same click slashes, harder.
  - Right click is your off hand: a second weapon slashes with it, the shield blocks while you hold the button.
  - A number rises over a dummy for every hit, and its body swings away from the hit on a spring.
  - The three have Armor 0, 10 and 20, and each heals back three seconds after its last hit.
  - Step right up in front of one and it leans back, then swings at you. Raise the shield to block it.
  - Its bar shows when you are within 8 m of it, or for 4 s after you hit it.
  - See [Attacks, the shield and the training dummies](#attacks-the-shield-and-the-training-dummies).
- **The crate:** Hit it (left click) until it breaks.
  - It bursts into pieces that tumble and shrink away, and three to five items fly out and land around it.
  - It is back 45 seconds later.

#### CRAFTING

- **Three ways to craft:**
  - Press the interact key at the forge in **CRAFTING**, just before the shops. Its window lists what the forge makes.
  - Open the bag and press **CRAFT**: every recipe; one made elsewhere says where.
  - Use a shop's craft tab.
    - The General Store's makes the Ring of Vitality, two Health Potions, two Repair Kits, Iron Gauntlets and a Leather
      Pouch (5-12 seconds each, so several can wait there at once).
    - The Gem Trader's and the Junk Merchant's make the recipes that need no station.
  - Each keeps its own queue: a craft shows only where it was ordered, and a full bag never stops one -- it waits there,
    finished, until there is room.

#### SHOPS

- **The three vendor stalls:** Walk up to one, look at it, press the interact key (`X` on a controller).
  - Each deals in one currency. In the order you reach them: the General Store (gold), the Gem Trader (gems), the Junk
    Merchant (silver).
  - See [Gold, silver and gems](#gold-silver-and-gems).
- **Gold coins, gems and silver coins, each in front of its own stall:** Walk over them.
  - Your bag shows gold, silver and gems in a row at its foot.
  - They come back 30 seconds after you take them.
- **The Trader, at the end of the shop row:** Press the interact key: the trade window opens with a Health Potion on the
  table.
  - Put any item down and confirm.
  - See [Trading with NPCs](npc-trade.md).

#### LOOT

- **The chest and the body:** Look at either and press the interact key: the loot window opens with your bag and equipment
  beside it.
  - Each fills again a minute after you open and empty it.
- **The heavy chest, beside the chest:** Hold the interact key for a second the first time.
  - Once anyone has opened it, a press opens it, until it refills a minute after it is emptied.
- **The scatter chest, between the heavy chest and the body:** Hold the interact key for a second.
  - It throws its loot on the ground all around it instead of opening a window.
  - It offers nothing while it is empty.
  - A minute later it refills, like the chest.

#### BANK

- **The bank chest under the BANK sign, at the end of the avenue:** Look at it: the prompt reads
  **(E) Open your bank  ·  Hold (E) Deposit everything**.
  - Tap `E` and your bank opens beside the bag: drag items in and out, or Shift+click one to send it across.
  - Or HOLD `E` for a second (the ring after the words fills): every bag item goes in at once.
  - Stop playing, start again: what you left in the bank is still there.
  - See [Your bank](#your-bank).

### Along the left of the walkway

- **The FEATURES pads:** Forty pads in one line, a plugin feature each, in seven blocks, each on its own colored ground
  with a title sign in the same color behind it.
  - Step on one: the sign over it shows the new value, and the feature is switched for this play session.
  - See [The FEATURES wing](#the-features-wing-switch-a-plugin-feature-for-this-session).

### The keys

| Key | Does |
|---|---|
| `W`/`A`/`S`/`D` | Move |
| The mouse | Look |
| `Space` | Jump |
| Hold `Left Shift` | Sprint: half as fast again, and Haste on top of that. Not while you carry more than you can (Haste still works then), and not with the bag or the menu open |
| Left click | Attack with your main hand |
| Right click | Attack with your off hand (held, a shield blocks) |
| `E` | Interact. At the bank chest a tap opens your bank and a hold deposits everything |
| `Escape` | Close whatever is open; when nothing is, open the demo's menu |
| `I` or `Tab` | Open and close the bag |
| The mouse wheel | Step to the next thing, where a prompt offers more than one |
| Hold the middle mouse button | A cursor: point at a hotbar or buff square for its tooltip |
| `K` | Die where you stand, to try the death drop |
| `Shift` and the mouse wheel | Page the hotbar |
| The key of the Lightning or Explosion square | Show a ring on the ground: left click casts there; right click, `Escape` or the key again cancels |

- The camera sits over your right shoulder.
- The crosshair in the middle of the screen is where an attack or a spell goes: when one starts, your character turns to
  the point under it.
- On a controller, click the left stick while you move to sprint. The sprint lasts until the stick comes back to rest.
- With a controller, see [Playing with a gamepad](#playing-with-a-gamepad).
- The menu's Controls list names every one of these keys, in two columns: **Mouse / keyboard controls** and **Gamepad
  controls**.

### A route through the map

- You start at the head of one long avenue, looking down it, so every section is in front of you.
- The welcome panel gives the same directions in a few lines; this is the longer version.
- Each section stands on its own colored ground, so you can tell which items a sign is about. Across the walkway, each
  block of pads does the same.
- A 6-m walkway runs between the sections and the pads, so walking down it picks nothing up and flips no pad.

1. **The tall signs on your left:** **SNAP INVENTORY SYSTEM** on top, **BAG TIPS: MOUSE AND KEYBOARD** and **BAG TIPS:
   GAMEPAD** under it, **TWO PLAYERS** at the bottom.
   - They show the keys: read them first.
   - Open the bag with `I` or `Tab` (`Y` on a controller).
2. **BAGS AND POUCHES.** Three bag upgrades, the Leather Pouch and the Cooler.
   - Use a bag upgrade from the bag for more squares. The best one you have used counts.
   - The bag shows the squares they could still add as locked squares at its end; an upgrade opens its own.
3. **WEAPONS.** The Iron Sword, the Iron Axe, the Iron Greatsword and the Iron Shield.
   - The sword and the axe fit either hand, and the greatsword takes both.
   - The shield blocks while you hold right click.
4. **ARMOR.** The iron set (cuirass, greaves, sabatons, gauntlets), the Gilded Cuirass, the helm in five rarities and a
   Repair Kit.
   - Wear them and the stats at the top left change as each goes on.
   - The armor appears on your character, piece by piece.
5. **JEWELRY AND A QUEST.** The Amulet of Mending, the Ring of Vitality, the Warrior's Signet and the Sealed Letter.
   - The Warrior's Signet needs Attack Power 12: wear the sword first.
   - The Sealed Letter is a quest item: hand it to the Courier standing there.
6. **POTIONS AND MIXING.** Drink one; drop one potion onto another in the bag to mix a new one.
   - The **MIXING** sign over the potions lists the three pairs.
7. **HEAL AND DAMAGE PADS.** Stand on the red pad with nothing worn, then in the iron set, and read the line under your
   health bar.
   - The green pad heals; the dark pad is heavy.
8. **FOOD.** Raw Meat, Bread and Rotten Mush.
   - The meat goes bad five minutes after the map starts; the Cooler from the first section keeps it fresh.
9. **SPELLS.** Five scrolls: pick one up, learn it from the bag, give it a hotbar key, close the bag and cast it.
10. **COMBAT.** The three training dummies and the crate.
    - Punch a dummy, wear the sword and slash it, and with the shield on hold right click as a dummy swings at you.
    - Hit the crate until it breaks.
11. **CRAFTING.** The forge, with two gold bars and a recipe scroll in front of it.
    - Press `E` at it and its craft window opens on its own and lists what the forge makes, the sword and the shield.
    - Each shop crafts too, on its CRAFT tab.
12. **SHOPS.** The General Store (gold), the Gem Trader (gems) and the Junk Merchant (silver), each with its money in
    front of it.
    - The Carved Totem by the Junk Merchant is junk to sell there.
    - The Trader is at the end: trade it any item for a Health Potion.
13. **LOOT.** Look at the chest or the body and press `E`: your bag opens beside it.
    - Hold `E` at the heavy chest the first time (then a press opens it) and at the scatter chest.
14. **BANK.** Your own bank chest, at the end of the avenue.
15. **The FEATURES pads, along the left of the walkway, in seven blocks.** Step on a pad and watch its sign change.

### If a digit key does nothing

- `1`–`0` reach the hotbar through the plugin's own mapping context, which it applies at priority `0`.
- A project that already binds those digits — most action templates do — wins the key, and the hotbar never hears the
  press.
- Everything else on the hotbar still works: clicking a slot takes the identical route a key does, so a slot that draws is
  a slot that activates.

**To fix it,** do one of these:

- Raise **Hotbar Mapping Context Priority** (**Project Settings → Plugins → Snap Inventory System - Equipment → Hotbar → Input**)
  above the context
  that already owns the digits.
- Bind your own action to `USnapEquipmentSettings::HotbarSlotInputActions`.

The plugin does not raise it for you: taking a key a project has already spent is not a decision a plugin should make on
its own.

The two `HotbarConsumableInputActions` ship with **no keyboard key at all** and are yours to bind. The demo keys them to
`Q` and `R` with a mapping context of its own (below).

## The HUD, the hotbar keys and the menu

- Three things on screen are the demo's own example code, in `SnapDemoHUD.h` beside the demo classes.
- The plugin does not need any of them; they show what a project's own HUD does with it.

### The stats panel

**The stats panel** (`USnapDemoHUDWidget`, top left) reads the demo's stats layer (`USnapDemoStatsComponent`, see
[The demo's two routes](#the-demos-two-routes)).

- **It shows** health as current / max with a bar, Armor, Attack Power and Attack Cooldown.
  - It is collapsed while the bag or another of the plugin's screens is up, like the hotbars.
- **Above it, the plugin's buff row** (`USnapActiveEffectsWidget`) shows every timed effect you have running, as a square
  with its seconds left (`USnapDemoStatsComponent::GetActiveEffects`, on either route).
  - That is Haste, Ward, a buff potion, a heal over time.
- **Each number follows the layer's `OnStatChanged`**, so it moves the moment an item goes on or comes off.
  - On the GameplayAbilities route, the ability system's value-change delegate feeds it.
- **Beside the health value,** a red number adds up every hit while hits keep coming, and a green one every heal
  (`SnapDemoHealthLoss` / `SnapDemoHealthGain`).
  - Each is gone 2.5 s after its last change.
  - The line under the bar says what the last one was for 3 s.
- **In split screen** each player's stats panel, hotbars, crosshair and menu are in that player's own part of the screen.

**The numbers, with `GameplayAbilities`:**

| Item | Gives |
|---|---|
| The sword | Attack Power 15 and Attack Cooldown 0.8 s |
| The axe | Attack Power 24 and Attack Cooldown 1.3 s |
| The greatsword | Attack Power 27 and Attack Cooldown 1.6 s |
| The Ring of Vitality | Max Health 105 |
| The iron armor | Armor 4 at two pieces and 12 at four, with its two set bonuses (Max Health +10 % at two pieces, Armor at four) |

- The iron armor's numbers are read on Common copies with no magic properties.
- The sword and the iron armor read the same numbers on the native route.

### The hotbars, and Q and R

**The hotbars** are the plugin's own Ability bar (keys `1`–`0`), in the middle, and its two Consumable squares (keys `Q`
and `R`).

- **They are put on that HUD the way a project puts them on its own.**
  - The HUD keeps two empty widgets at the bottom of the screen: `SnapDemoHotbarAnchor` at the bottom center and
    `SnapDemoConsumableAnchor` just left of it.
  - The demo's settings name them for the play session.
- Neither bar has a plate behind it: each square draws its own frame.
- Each square shows its key and, while its item is cooling down, the time left.
- An empty square shows a faint outline of what goes there:
  - A ring with a plus on the Ability bar (any item can go there).
  - A potion on the two Consumable squares.
- **Both bars show the page, `1/3`.** One page holds every bar, so the potion squares page with the Ability bar.
  - The demo turns on `bShowConsumableBarPageIndicator` to say so.

**To place the bars in a game of your own,** put the same lines under
`[/Script/SnapInventorySystem.SnapEquipmentSettings]` in your `Config/DefaultGame.ini`. Without them the bars show
only inside the bag:

```ini
+HotbarHUDPlacements=(BarTag=(TagName="SnapInventorySystem.Hotbar.Bar.Ability"),HostWidgetClass="/Script/SnapInventorySystem.SnapDemoHUDWidget",AnchorWidgetName="SnapDemoHotbarAnchor")
+HotbarHUDPlacements=(BarTag=(TagName="SnapInventorySystem.Hotbar.Bar.Consumable"),HostWidgetClass="/Script/SnapInventorySystem.SnapDemoHUDWidget",AnchorWidgetName="SnapDemoConsumableAnchor")
```

**Q and R, and the consumable squares.**

- The plugin ships its two consumable-square actions unbound.
- The demo keys them with a mapping context of its own, `IMC_SnapDemoConsumables` (`Q` and `R`, one use per press).
  - The demo's controller adds it for its player and registers it with the key settings.
- The demo's slot set has no consumable equipment slots, so a potion stays in the bag, and the squares refer to it there.
- A picked-up potion lands on the first free square by itself (`bAutoBindConsumablesToHotbar`).
- **To put one on a square yourself,** open the bag and do one of these:
  - Point at it and press `Q` or `R`.
  - Drag it onto the square.
  - Shift-click it (it fills the first free square).
- Pressing `Q` or `R` in play drinks from that square.

### The bag and its windows

**The bag** is the example Character screen.

- **The equipment panel is a paper doll.** Each square sits by where its piece is worn on the figure.
  - The figure faces you, so its right hand is on your left.
  - Head is above it.
  - Amulet, Main Hand, Right Ring and Gloves are on your left.
  - Chest, Off Hand, Left Ring and Greaves are on your right.
  - Boots are below.
  - Then come the two potion squares and the Ability bar, under the titles POTIONS and HOTBAR.
- The potion squares and the Ability bar are drawn more translucent than the bag's own squares, an empty one showing a
  faint plus (the HUD's bars show nothing in an empty square).
- The bag is beside it, with the Spells panel directly under it, and the whole screen is centered.
- The bag scrolls from the start: its last three rows are the locked squares the bag upgrades could still open (48 in all).

**The bag's windows stay put** in the demo.

- Its settings turn `bAllowWindowDragAndClose` off for the play session, so no window has a drag strip or a close box.
- **The craft window starts closed** behind the bag's **CRAFT** button, and opens in the paper doll's place beside the bag.
  - The doll steps aside while it is open, so the window covers nothing.
  - The doll comes back when **CRAFT** is pressed again, when **B** or **Escape** puts the window away, or when the bag is
    closed.
- Turned on (a project's own setting, outside the demo), each window floats and the craft window opens in the middle.

### Changing a hotbar key

1. Open the bag.
2. Point at a square of the bag's own bars. The bars at the bottom step aside while the bag is open.
3. Press the key you want. This works on the `Q` and `R` squares too.
   - You should see the square take it and its label change at once. The old key no longer fires it.

- A key another square already has swaps the two, so no square is left without one.
- The square's own key still fires it.
- **These are never taken:** Escape, a modifier on its own, the mouse buttons, a controller, the keys the demo character
  moves with, and any key something else already uses (`I`, `E`).
- **The new key is saved.** It goes through the player's Enhanced Input key settings (the same rows a key-binding screen
  edits).
  - It is saved with them in `Saved/SaveGames/EnhancedInputUserSettings.sav`, so it is still there after a relaunch.
  - With two players on one screen that file is both players': after a relaunch both have the keys whichever player saved last.
  - Delete that file to get `1`–`0` back.

### The menu and the welcome panel

**The menu** (`USnapDemoMenuWidget`) opens on `Escape` when nothing of the plugin's is open.

- `Escape`, **B** on a pad, or **Resume** closes it.
- With the bag, a shop or a window up, `Escape` closes that first, as it always has.
- **Leave the demo** quits through the game's own quit: a packaged build closes, and Play in Editor stops.
  - On a second player's menu in split screen it takes that player off the screen instead, as **SPLIT SCREEN** does.
- **Its Mouse and stick section** (on the welcome panel too) has one **Look speed** slider (0.2x to 3x) and **Invert Y**.
  - Both settings apply at once.
  - The demo keeps them between sessions in its own save slot, `SnapInventoryDemo_Mouse`, never in the project's input
    settings (`ASnapDemoPlayerController::SetLookSensitivity` / `SetLookYInverted`, read by `ASnapDemoCharacter` on every
    look input).
- Its controls are two titled columns, **Mouse / keyboard controls** and **Gamepad controls**.
- ⚠ **The menu is opened by a Slate input processor, not a key binding.** In Play in Editor the editor's own Stop command
  takes `Escape` before the game's input does, so a key binding would only ever end the session.

**The welcome panel** is that menu in its first-time dress (`USnapDemoMenuWidget::SetWelcomeMode`).

- **It has:**
  - A welcome title.
  - A short tour of where everything is.
  - The same mouse settings.
  - The same two controls columns.
  - **Start playing** where Resume is.
- On a short viewport its contents scroll (`SnapDemoMenuPanelSizer`).
- The demo's controller opens it once, half a second after you start. A respawn does not bring it back.
- A second player who joins gets their own welcome panel, in their part.
  - While any player's menu or welcome panel is up, no player's character moves: the engine's one viewport takes no game
    input then.
- The `bShowWelcomeOnStart` setting on `ASnapDemoPlayerController` turns it off.

### The pointer

**The pointer** is the demo's own as well: an ornate gold-and-iron arrow (`USnapDemoCursorWidget`, in the same header).

- The demo controller draws it instead of the system's arrow and hand, on its own game viewport while it plays: over the
  bag, the shops and the menus.
- Its picture is `T_Example_DemoCursor` (a plain arrow drawn by a script).
  - The art pack swaps it for the gold one in `/DemoArt` (`ASnapDemoPlayerController::CursorPicture`, `bUseDemoCursor`).
- A text box keeps the system's text beam.
- A host whose game viewport has software cursors switched off (Lyra's does, on desktop) gets them switched on while
  the demo controller plays.
  - The host's own software cursors for the other shapes are set aside meanwhile.
  - Everything is put back when the controller ends play.
- A dragged item hides the pointer like the system's arrow: the item is the cursor (`HeldItemPointer`).
- It is drawn 28 pixels across (`USnapDemoCursorWidget::PictureSide`) whatever the screen size: the engine draws a software
  cursor in window pixels.

## Playing with a gamepad

- The demo plays on a controller.
- The tables below name the buttons for an Xbox pad. On a PlayStation pad, A is Cross, B Circle, X Square and Y Triangle.
- **The menu's Gamepad controls column** (the welcome panel's too), beside the **Mouse / keyboard controls**, draws each
  button as a PICTURE of your pad, a row per control.
  - The pictures are the plugin's neutral Xbox-layout or PlayStation-layout set, whichever `GamepadGlyphFamily` picks
    (AUTO: the pad you used last; or the GAMEPAD ICONS pad).
  - Its sentences name the pad's own buttons in that pad's words (RT or R2, Menu or Options).
  - With no glyph set, the rows show the buttons' short names in those words instead.

### Buttons in play

| In play | Does |
|---|---|
| Left stick | Move |
| L3 (click the left stick) | Sprint, while you move; it ends when the stick comes back to rest, or on a second click |
| Right stick | Look. The menu's **Look speed** and **Invert Y** apply to it as to the mouse: unticked, pushing it up looks up |
| A | Jump |
| X | Interact: tap it, or hold it where the prompt says hold (at the bank chest a tap opens your bank and a hold deposits everything) |
| B | The next interaction option, when a prompt offers more than one; with a spell's ring up, cancel it; close the money changer's window |
| Y | Open the bag |
| RT | The main hand: the punch, or the weapon in your right hand |
| LT | The off hand; hold it and the shield blocks |
| LB, RB | Drink from the two consumable squares |
| D-pad | Hotbar squares 1 to 4; Lightning Strike and Explosion show their ring — RT casts, B or LT cancels |
| X held + D-pad | Hotbar squares 5 to 8 (up, right, down, left); the squares draw X and the arrow. A hold X began (the bank's deposit) is called off |
| View | The next hotbar page (after the third, the first again) |
| Start | The menu; Start again, B, or **Resume**, closes it |

### Buttons in the bag

| In the bag | Does |
|---|---|
| D-pad, left stick | Move between the squares: the bag, the paper doll, the Ability bar and the two consumable squares |
| A | Pick an item up, and put it down on another square: a weapon on a hand, a potion or a scroll on a hotbar square (it is bound there). The square it came from stays blue, and the target's corner shows what the put-down does |
| A, on a chest's or a body's item | Pick it up too: put it on the bag square you choose, or back on the loot window |
| A, on a hotbar square | Pick it, then A on another square of that bar swaps the two |
| B | Cancel what you are carrying (a chest's item stays in the chest); again, close the bag |
| X | Quick-move the bag item under focus, as a shift-click does; on a chest's item, take it into the bag |
| Y | The item's menu (Consume, Split Stack, Drop); A does the row it is on |
| Right stick | Turn the paper doll's figure; R3 puts the view back |
| LB, RB | The previous / next section: the paper doll, the bag, its filter row, its CRAFT / DROP / currency row, Spells, the consumable squares, the Ability bar |
| A, on CRAFT | The craft window in the paper doll's place, focus on the selected recipe |
| In the craft window | The D-pad walks the recipes, and Right reaches the quantity |
| B, in the craft window | Put the window away, and focus comes back to CRAFT (as A on CRAFT again does); the next B closes the bag |
| LT, RT | The previous / next filter tab |
| RB, with a loot window open | Take everything from it |
| RT, carrying an item in a grid bag | Turn it |
| RT, on a hotbar square | Change its pad button — see below |

### How the gamepad controls are put together

- **The sticks, A and RT are raw bindings** on the demo's character and controller, like W/A/S/D and the mouse: no context
  maps them.
- **The other buttons are the demo's own mapping context, `IMC_SnapDemoGamepad`** (with `IA_SnapDemo_OffHand` for LT).
  - `ASnapDemoPlayerController` applies it only in gameplay (while nothing of the plugin's is open and the menu is not),
    and above every other context.
- ⚠ **That is not a nicety.** The plugin's own bag context keys X, Y, LB, RB and LT to its gamepad verbs (quick-move, the
  item menu, the sections, take all, the filter tabs).
  - Those take the buttons in gameplay too, so a lower context, or a raw binding, on any of them never fires.
- **While the bag is up, the demo's context is off** and the five buttons are the bag's again.
  - A button held across the change does nothing until it is released.
- **The plugin itself now opens the bag on Y during play the same way** (its `GamepadOpenKeys` context, one priority above
  its bag context).
  - On this map the demo's own Y row sits higher still and wins.
  - The two share one claim, so a press opens the bag once.
- Start goes through the same input processor as Escape.
- **`K` has no button.** A test death one press away on a controller is too easy to hit, and the red pad does the same.
- **X + the D-pad is a second demo context, `IMC_SnapDemoPadChords`:**
  - `IA_SnapDemo_HotbarShift` on X, which takes nothing from it.
  - `IA_Hotbar_Slot05` to `08` on the D-pad, each chorded on it.
  - It is always on, and one above that, so X still interacts and the plain D-pad does not also fire squares 1 to 4 while
    X is held.

### Changing a square's pad button

1. In the bag, move focus onto a hotbar square.
2. Press RT.
   - You should see a line under the stats at the top left say it is waiting.
3. Press the pad button of another square of the same bar.
   - The two squares swap buttons: the D-pad's four, on the Ability bar's first four squares.

- **While it waits:** B cancels, the square's own button ends it with nothing changed, and any other button is refused.
- **A pad rebind swaps two squares' buttons.** A pad has no free button to give a square: every one already means
  something in play or in the bag.
- **A square with no pad button has none to swap:**
  - 5 to 0 on the Ability bar (5 to 8 answer X + the D-pad, which no swap moves).
  - The two consumable squares, whose LB and RB belong to the demo's gameplay context rather than to the key settings.
    - Registering a second context for the same actions folds both into one key-settings row, and the Q and R squares
      would lose their keyboard row.
- The swap goes through the player's Enhanced Input key settings and is saved with them, as the hover rebind's keys are
  (`SnapDemo::DecidePadRebind` is the rule).
- A square's pad button is the one that works now (the live mappings), even when a saved key profile holds an empty slot
  for it.

### The Spells panel and reordering the hotbar

**The Spells panel is reached like the bag.**

- Down from the bag's last row moves onto the Spells square in line with it, and Up comes back.
- Down from the Spells panel's last row reaches the hotbar.
- A scroll there is carried like any bag item: Accept on it (it stays framed in blue while you move), then Accept on a
  hotbar square puts it there (see [Input](input.md)).

**Reordering the hotbar in the bag:**

1. Click a filled square, or press A on it, to pick it.
2. Click another square of the same bar, or press A on it, and the two swap. On an empty square, it moves there.

To put it back, click the picked square again or press A on it, or press B / Escape.

### How it was checked

- The demo was played with a physical Xbox-style controller, and what that found is fixed.
- The fixes were checked by driving:
  - The gameplay buttons with Enhanced Input's injected key presses.
  - The bag with simulated pad keys and the keyboard's arrows, Enter and Escape — the same navigation Slate gives a D-pad,
    A and B.
- They were not checked on the controller again.

## The spell scrolls

- Five scrolls stand under the **SPELL SCROLLS** sign, in the **SPELLS** section.
- Each is a spell you cast from the hotbar, on either of the demo's routes (see
  [The demo's two routes](#the-demos-two-routes)) and with the same numbers: the scroll carries them.
- Two act on you; three are aimed with the camera and hit the training dummies, the crate and other players.

### Learn and cast a scroll

1. **Pick it up:** Walk over one, or look at it and press `E`.
   - It goes into your **bag**, like any other item.
2. **Learn it:** Open the bag (`I`), right-click the scroll and choose **Learn**.
   - Or double-click it, or drag it onto the Spells panel.
   - You should see it move into **Spells** for good, called by its spell there ("Fireball"), and the feed say
     "Learned Fireball".
   - A spell you already know is refused ("You already know that.") and the scroll stays in the bag.
3. **Give it a key:** Point at the scroll in the Spells panel and press the hotbar key you want it on (or drag it onto a
   square).
   - A scroll still in the bag casts nothing: its key says to learn it first.
4. **Close the bag.**
5. **Cast it:** Press that square's key. For Lightning and Explosion, then aim the ring and left click.
   - You should see the spell take effect and the square count down its cooldown.
   - Pressing again before then only says it is not ready.

**A learned spell is yours for good: Spells keeps it.**

- You can move it around in Spells and between hotbar squares.
- You cannot move it back to the bag, onto the floor, to a shop or into a trade
  ("That stays in the container that keeps it.").
- A death leaves it in Spells while the body takes the rest.

### What each scroll does

| Scroll | Cast from the hotbar | Numbers | Cooldown |
|---|---|---|---|
| Scroll of Haste | You run half again as fast (600 to 900 cm/s) | Lasts 8 s | 15 s |
| Scroll of Warding | 25 extra Armor — watch the stats panel | Lasts 12 s | 20 s |
| Scroll of Fireball | Press the key: a burning bolt flies from your hand toward the middle of the screen and bursts on the first thing it hits | 25 damage within 150 cm; flies 3000 cm | 2 s |
| Scroll of Lightning | Press the key: a ring on the ground follows your aim. Left click: lightning strikes there a third of a second later | 35 damage within 200 cm; lands up to 1500 cm away | 4 s |
| Scroll of Explosion | Press the key, aim, left click: half a second later a blast that damages and throws back what it can move — the dummies and the items on the floor | 30 damage within 350 cm; lands up to 1200 cm away | 8 s |

- **Damage is before the target's Armor**, through the same rule as the attacks, so a dummy shows less than the table
  says.
  - Armor 10 takes 25 of the Lightning's 35.
- **The crate takes it too**, counted against its hit points.
  - Its 20 are fewer than any of the three spells deal, so whichever reaches it breaks it.
  - Its loot flies out as when it is hit.

### Aiming Lightning and Explosion

⭐ **The key shows the ring on the ground under the middle of the screen:**

- Cyan where the spell can land.
- Red where it cannot: beyond its range, on a wall, or with nothing under your aim (the sky).

**Left click (RT) casts at the ring; right click (B or LT on a controller), `Escape` or the key again cancels.**

- While the ring is up, those buttons are the aim's, not your hands' or the menu's.
- A click on red casts nothing and spends no cooldown, and the ring stays so you can aim again.
- The ring also goes away by itself if you die or open the bag or the menu.
- With the bag or the menu already up, their keys cast nothing and say to close the bag, on either route, while the other
  three spells cast as usual.
- The aim looks through the training dummies, other players and the pickups to the ground behind them, so sweeping it
  across a dummy never makes the ring jump.
- Only you see the ring. Everyone sees the strike, and a warning circle marks where it will land and how wide for the
  moment before it does.
- Casting turns you to face the point under the crosshair.

**The Explosion's knock-back** pushes anything that simulates physics, and knocks every item lying in reach across the
floor: 320 cm from the center of the blast, down to 90 at its edge.

- A knocked item stays on the floor, stops in front of a wall rather than going through it, and is still there to pick
  up.
- It never launches a player: a blast against a wall must not throw anybody over it.

### How the spell scrolls are built

How they are built, so you can make your own:

- **The item** (`ID_Example_SpellScroll_Haste` and the other four) carries a **`USnapDemoSpellFragment`**, its Hotbar
  fragment.
  - That is an `Event` item allowed on the Ability bar (`bAllowOnAbilityBar`).
  - It holds the spell's numbers (`FSnapDemoSpellNumbers`: the spell, its duration, cooldown, Armor, damage, radius,
    range, knock-back and its looks).
  - For the GameplayAbilities route, the item also carries a Granted Ability Set fragment whose **Container Abilities**
    names the spell's ability.
  - Nothing else makes it a spell — see [Custom containers](custom-containers.md).
  - It has **no** Container Routing fragment, so a pickup lands in the bag.
- **Learning** is the demo's spellbook, `BP_Example_DemoSpellsContainer`: a Blueprint of
  **`USnapDemoSpellbookComponent`** (`SnapDemoSpellbook.h`, compiled in every build).
  - It is the container tagged `SnapInventorySystem.Container.Spells`, with eight squares.
  - Its `bActivateHeldItemAbilities` is on.
  - The plugin's `bKeepsWhatEnters` is on, so nothing a player asks for takes a learned spell out, and a death leaves it
    there.
  - It admits only spell scrolls it does not already hold (a second copy is refused with `AlreadyKnown`), by the item
    rather than by routing. That is also how a save restores a learned spell.
- **The Learn row** is `USnapDemoLearnSpellAction`.
  - The demo's profile adds it to the project's context-menu rows for the session.
  - It asks for the double-click (`bPerformsOnDoubleClick`).
  - It asks the server through the caster (`Server_LearnSpell`).
  - The row, a double-click and a drag onto Spells all reach the server as the plugin's `MoveItemsToContainer`.
  - The book raises the "Learned" feed line where they arrive (`OnItemsMovedIn`).
- **The key** goes to the scroll's spell fragment (`OnHotbarEventActivated`), and from there to the player's
  **`USnapDemoSpellCaster`**, on the player state, compiled in every build (`SnapDemoSpellCaster.h`).
  - It casts Haste, Ward and the Fireball at once, and shows the ring for the other two: ONE aim for both routes.
  - Then it casts by the demo's route.
  - **The GameplayAbilities route** activates the ability the Spells container granted, and hands it the point it aimed
    at.
  - **The native route** asks the server (`Server_CastSpell`).
    - The server checks that the scroll is in the player's Spells, the player is alive and off cooldown, and that a
      ground spell's point is within its range (150 cm of slack for the walk).
    - Then it starts the cooldown on the demo layer and casts with the server's own copy of the scroll's numbers.
  - The square's countdown asks the demo layer for the spell's cooldown on whichever route is active
    (`GetEventCooldown`).
- **The GameplayAbilities spells** are example classes in `SnapDemoSpells.h`, built only with GameplayAbilities.
  - The scrolls name them by soft class path, so a project built without GameplayAbilities loads them with no warning.
  - Each reads its numbers from the spell fragment of the scroll its container holds (`ResolveSpellNumbers`).
  - Its own class defaults come from the same table the scrolls were filled from (`FSnapDemoSpellNumbers::Shipped`), and
    are used only when no scroll stands behind the grant.
  - `USnapDemoSpellAbility` (Haste and Ward) commits its cooldown, applies its effect to the caster for the spell's
    duration and ends.
    - Each spec carries the spell's own tag, so one effect class and one cooldown class serve every spell.
    - Ward's Armor rides on the spec as a SetByCaller, the scroll's own number.
  - `USnapDemoAimedSpellAbility` (Fireball, Lightning Strike, Explosion) takes the point the caster aimed at and sends it
    to the server as ability target data.
    - The server decides: it checks the range, commits the cooldown and spawns the strike or the bolt, which lands the
      damage itself.
    - The cooldown is spent only by a cast.
- **The native route's state** is the demo layer's.
  - Haste and Ward put their state tag on for their duration (`ApplyNativeTimedTag`).
  - Ward's Armor is a timed raise like a potion's.
  - The cooldowns are the layer's own list.
- **One registry for both builds.** The demo's profile names `DA_ExampleDemoContainerRegistry`.
  - It lists the default rows itself, with the spellbook in place of the plugin's own Spells container
    (`BP_Example_SpellsContainer`), plus the personal bank.
  - The spellbook is on no GameplayAbilities-only list, so each build attaches exactly one Spells container.
  - Its scrolls sit on the Ability bar, and cast on either route. With `bActivateHeldItemAbilities` on, a custom
    container is one the Ability bar draws from in every build.
  - A key casts only a scroll in Spells, on both routes. With `GameplayAbilities` that is where its spell is granted, and
    the native route keeps the same rule.
- **The aim, the ground rule and the spawning** are plain functions in `SnapDemoSpellCasting.h`, compiled in every build.
  Both routes call them with the scroll's numbers.
- **The looks** are data-only Blueprints of the actors in `SnapDemoSpellActors.h`, in `Examples/Spells`:
  `BP_Example_Fireball` and `BP_Example_FireballBurst`, `BP_Example_SpellAimRing`, `BP_Example_LightningStrike`,
  `BP_Example_Explosion`.
  - They are our own meshes (`SM_Example_SpellOrb`, `SM_Example_SpellRing`, `SM_Example_SpellBolt`) with one unlit
    additive material (`M_Example_SpellGlow`) and a light.
  - No particle system, so the plugin depends on no other plugin for them.
- **What you see of Haste** is the demo character answering its state tag, on every machine.
  - Haste raises `USnapDemoMovementComponent::SpeedScale`, a scale on top of `MaxWalkSpeed`.
  - So it composes with the plugin's overweight slowdown, which writes `MaxWalkSpeed`, instead of fighting it.
  - Ward needs no answer: Armor is an attribute the stats panel already shows.
  - A character of your own answers neither until it watches the same tag.
- **The models** are one shape, `SM_Example_SpellScroll_Haste`, `_Ward`, `_Fireball`, `_Lightning` and `_Explosion`, in
  five finishes.

### Make your own targeted spell

1. **Copy a scroll.** Its spell fragment holds everything the spell does.
2. **Set its numbers:** `Spell` (Fireball for a bolt, Lightning Strike or Explosion for a ground strike), `Damage`,
   `Radius`, `Range`, `KnockbackSpeed`, `Cooldown`.
3. **Set its looks:** A bolt names a `ProjectileClass` and a `BurstClass`; a ground strike an `AimRingClass` and a
   `StrikeClass`.
   - That is the whole spell on the native route.
4. **For the GameplayAbilities route,** point the copy's Container Abilities at a `USnapDemoAimedSpellAbility` subclass, in
   C++ or a Blueprint.
   - It reads the numbers from the scroll; its own `Damage`, `Radius`, `Range` and the rest are only for a grant with no
     scroll.
   - ⚠ A spell's cooldown is its kind's (`SnapDemoSpellCasting::CooldownTagFor`), on both routes: two scrolls of one kind
     share one timer.
5. **Make its look:** A Blueprint of `ASnapDemoSpellProjectile`, `ASnapDemoSpellStrike` or `ASnapDemoSpellVisual`.
   - Set its meshes, their material and the light.
   - Then set the timeline: `RevealDelay` (when a strike lands), `VisibleSeconds`, the start and end scales, `GlowPeak`.
   - The meshes are drawn for a blast of `AuthoredRadius` cm, and the spell scales the look to its own `Radius`.
   - ⚠ The light's reach grows with that scale and its lumens do not, so a wide blast lights a wide area.
     - The Explosion's light is 2500 lumens and the Lightning Strike's 3000, each reaching 450 cm for each 100 cm of
       blast.
     - A brighter one clips a pale floor to white.
6. **Put the scroll in Spells and on the Ability bar.**
   - You should see the same key aim and cast it on either route.

## The potions

- **Eight consumables:** Five to find and three to mix (below).
  - They work on either route (see [The demo's two routes](#the-demos-two-routes)), so the stats at the top left move
    when you drink one.
- **To drink one,** double-click a potion in the bag, or put it on `Q` or `R`.
  - A double-click is the same as right-click → **Consume**, through the same code.
- A potion you pick up lands on the first free consumable square by itself.

**Drinking takes half a second, and the potion works at the sip.**

- The character raises the bottle to its mouth and tips it up, the cork at the lips. You can walk meanwhile.
- The heal or the buff, the cooldown and the line in the pickup feed all arrive at the moment it tips its head back.
- With the demo's art pack on, that drink is the pack's own motion, fitted to the same half-second sip and finished
  by script.
  - With `Snap.Demo.ArtPack 0` it is the demo's own pose.
- Die before then and the potion is not spent.
- An attack pressed while you drink does nothing.
- Anything else you use meanwhile waits for the drink to finish (a second potion then meets the shared cooldown).
- The demo session sets this up for itself (`ItemUseSeconds`, see
  [A use that takes time](items-and-fragments.md#a-use-that-takes-time)).
- **The other poses:**
  - A read for a recipe scroll and for learning a spell scroll (half a second, `USnapDemoSpellCaster::LearnSeconds`; the
    scroll reaches the spellbook at the end).
  - A short tap for the repair kit, a mix and anything else that is used up.

| Potion | What it does | Where |
|---|---|---|
| Health Potion | Restores 50 Health at once; refused at full health ("Your health is full" — a `ConsumeCondition`, so nothing is spent and the shared cooldown does not start; the Greater Health Potion too) | The pickup row; the General Store (40 gold) and the Junk Merchant (15 silver) |
| Potion of Regeneration | Restores 4 health every second for 20 seconds, the first a second after you drink it: 80 in all | The **POTIONS** table; the Junk Merchant, 20 silver |
| Elixir of Might | +10 Attack Power for 30 seconds | The **POTIONS** table; the Junk Merchant, 30 silver |
| Ironskin Potion | +15 Armor for 30 seconds — the red pad's hits shrink | The **POTIONS** table; the Junk Merchant, 30 silver |
| Tonic of Vigor | +25 Max Health for 60 seconds — the cap, not your Health: heal up to it on the green pad | The **POTIONS** table; the Junk Merchant, 35 silver |

**Every potion shares one 10-second cooldown.**

- Drink one and none of them can be drunk for ten seconds: not from the `Q` and `R` squares, not by a double-click or the
  bag menu's Use.
- Both squares count the wait down, and a press during it says "Potions are cooling down" under the stats.
- The route pad's flip ends it, as it ends every cooldown.
- **After it, drinking another of the same kind while one works restarts its timer instead of adding to it.**
  - Two Elixirs of Might give +10 Attack Power for thirty seconds from the second one, not +20.

### Mixing potions

**Drop one item onto another** (in the bag, in a pouch, or from one to the other).

- When the pair is one of the demo's three mixes, both are spent and the new potion is made in the bag.
- These are the plugin's mix rules (see [Mixing items](crafting.md#mixing-items)).
  - The demo applies its own set, `DA_ExampleMixes`, for the play session.
  - A project that names none mixes nothing.
- One of each of the six inputs lies just behind the potions, under the **MIXING** sign, which lists the three mixes.
  - The **MIXING** sign stands on top of the **POTIONS** sign.

| Mix (one of each, either order) | Makes | What it does |
|---|---|---|
| Elixir of Might + Ironskin Potion | Battle Tonic | +10 Attack Power and +15 Armor for 30 seconds |
| Health Potion + Arcane Shard | Greater Health Potion | Restores 120 Health at once |
| Tonic of Vigor + Potion of Regeneration | Elixir of Life | +25 Max Health for 60 seconds, and 4 Health every second for 20 seconds |

- **A pair that cannot mix right now is refused** with the reason, and nothing is spent: the bag has no room for the
  result, a potion you locked.
- The made potions share the potions' cooldown.
- The Battle Tonic's second buff is the consumable's second raise (`SecondTimedStat`), which runs beside the first and
  ends with it.
  - The buff row shows the tonic once.

### How the potions are built

**Each potion carries its numbers in a `USnapDemoConsumableFragment`** (`SnapDemoConsumables.h`, compiled with or without
GameplayAbilities):

- An instant heal.
- Heals over time.
- A timed raise of one stat (and an optional second one).
- Whether a second one restarts the first.
- Its cooldown: a tag every potion shares, `SnapInventorySystem.Example.Cooldown.Potion`, and 10 seconds.

**The fragment joins the plugin's consume seam** (`DeclaresConsumeGrant` / `CanApplyConsumeGrantTo` / `ApplyConsumeGrantTo`, and
`GetConsumeCooldownFor` for the squares' countdown).

- The cooldown is a cooldown effect on the GameplayAbilities route, and the layer's own list on the built-in one
  (`USnapDemoStatsComponent::StartCooldown`).
- The fragment asks the demo's stats layer to do it: `USnapDemoStatsComponent::ApplyConsumable`.
  - Every heal goes through the layer's own heal.
  - A timed raise is a `USnapDemoTimedStatEffect` on the GameplayAbilities route, and part of the native numbers on the
    other.
- The When used lines in its tooltip are built from the fragment's own numbers: one line per effect, number first.
  - Its `UseText` is shown only by a consumable that has none of those effects.
  - Where the demo runs (its game state), the list ends with the shared cooldown: "Cooldown: 10 seconds (Global)".
- On a player with no demo layer (a project that is not the demo), the fragment does nothing.
- **The Health Potion keeps its consume ability, `GA_Example_Consume`**, so a project with its own consume ability
  (`ConsumeAbilityTag`) still routes it through that.
- The effects the potions used to name (`GE_Example_PotionRestore` and `USnapDemoRegenerationPotionEffect` with its three
  siblings) are still in the plugin, and no example item names them.

### Food

**Two example foods are eaten the way a potion is drunk:** A double-click in the bag, or right-click → **Consume**.

- The character lifts it to its mouth (the eat pose, for any item tagged `SnapInventorySystem.Example.Food`).
- The half-second use time applies, and the heal lands at the moment.
- The pickup feed says *Ate Raw Meat* / *Ate Bread*.
- Neither shares the potions' cooldown, and neither has one of its own.

| Food | What it does | Where |
|---|---|---|
| Raw Meat | Restores 15 Health at once. It spoils into Rotten Mush five minutes after it appears (on the ground too), which is eaten like the meat but HURTS: a hit of 15 (see [Condition and weight](condition-and-weight.md)) | The pickup row; the Junk Merchant, 5 silver; the crate |
| Bread | Restores 2 Health every second for 10 seconds, the first a second after you eat it: 20 in all. Eating another while it works restarts the count instead of adding to it, and it does not spoil | The pickup row, beside the meat; the Junk Merchant, 4 silver; the crate |

- Both carry their numbers in the same `USnapDemoConsumableFragment` as the potions, so they work on either route.
- On a player with no demo layer, the fragment does nothing, and the food is used up with no effect.

## The heal and damage pads, and dying

- Three pads lie in their own section after **POTIONS AND MIXING**, under the **HEAL AND DAMAGE PADS** sign.
- They are there to show what the equipment you wear actually does.

| Pad | While you stand on it |
|---|---|
| Green | Heals 10 a second, up to your Max Health |
| Red | Hits you for 20 a second, less your Armor — and it can kill |
| Dark (HEAVY, nearest the FEATURES wing) | Hits you for 50 a second, less your Armor — so every point of Armor still counts after the red pad's hits are down to 1 |

**Every hit and heal is written under the health bar:**

- `-20` with nothing worn.
- `-5 (Armor blocked 15)` in the iron set.
- `+10`.
- `+0 (Max Health 100)` once you are full.

### Things to try on the pads

1. **Stand on the red pad with nothing on.**
   - Each hit takes 20; you are dead about four seconds after stepping on.
2. **Put the iron set on and stand on it again.**
   - Your Armor (the stats panel) comes off every hit: 20 less 15 takes 5.
   - The same pad now takes about nineteen seconds to kill you.
   - A hit always takes at least 1, so no amount of Armor makes you safe on it forever.
   - Cast the Scroll of Warding from the hotbar and step on: every hit reads `-1 (Armor blocked 19)` until the ward ends
     twelve seconds later, then `-20` again.
3. **Wear the Ring of Vitality and stand on the green pad.**
   - Your Max Health goes up but your Health does not; the pad fills you to the new Max Health.
   - Take the ring off at full health and your Health comes down with the Max Health.
4. **Step on the dark pad, HEAVY.**
   - It hits for 50 a second, less your Armor: 50 with nothing worn, 35 in the iron set, 10 with the Scroll of Warding on
     top.
   - The red pad shows Armor running out; this one shows it still paying.
5. **Die, then come back for your things.**
   - Stay on the red pad, or press `K` anywhere.
   - You fall where you stood, and the plugin drops everything you carried into a pack on the ground: what you wore and
     the bag (potions included), with whatever is inside your pouches.
   - Your gold, silver and gems land beside it.
   - Your learned spells stay in Spells.
   - Five seconds later you are back at the start with full health.
   - The pack waits for you however long you take: walk back, look at it and press `E`, and take your things back.
   - It shows its flap open while anything is left, and once you have taken everything it is gone.
   - You can loot it from beside the red pad without stepping on it.
   - In a game with other players, anyone close enough can loot it too: the plugin decides who may loot a body by
     distance, not by whose it was.

### How the pads and the death are built

- **The damage rule** is one function both routes call, `SnapDemoStats::ComputeDamageTaken(Raw, Armor)`: Raw minus Armor,
  never less than 1.
  - **On the GameplayAbilities route,** `USnapDemoAttributeSet` applies it: two meta attributes, `IncomingDamage` and
    `IncomingHealing`, that an instant effect adds to.
  - The set takes the result off Health for a hit, adds a heal up to Max Health, and keeps Health between 0 and Max
    Health.
  - So any effect of yours that adds to `IncomingDamage` (a trap, an attack) gets the same rule.
  - **On the native route,** the layer applies it to its own numbers.
  - `USnapDemoHealthLibrary::ApplyDemoDamage` and `ApplyDemoHealing` apply one on whichever route is in use.
- **The pads** are `ASnapDemoHealthPad` (`SnapDemoHealth.h`, compiled either way; a tick goes through the demo layer).
  - `Effect`, `AmountPerTick` and `TickIntervalSeconds` are set per placed pad.
  - A player is ticked once on arrival and once per interval while they stay.
  - The pawn's capsule and mesh count as one player, and stepping off and straight back on does not tick early.
  - The model is `SM_Example_Pad` in three finishes.
  - The heavy pad's darker top is `MI_ExampleProp_PadHeavy`, and its glow is turned down on the placed pad.
- **The death is the plugin's own.** At 0 Health the demo applies the tag `SnapInventorySystem.Example.Status.Death`.
  - The demo's profile names that tag in `DeathTagName` (the plugin's setting), and on the bag with
    `bDropEverythingOnDeath`.
  - It points `LootableBodyClass` at `BP_Example_LootableBody` (the fallen-adventurer pack, 64 slots: everything a demo
    player can carry).
  - The plugin then closes the bag, hides the prompts and drops everything onto that body.
  - The profile also sets `LootableBodyLifetimeSeconds` to 0 and turns on `bRemoveEmptiedLootableBodies`, so the pack has
    no deadline and goes once it is empty and nobody is left beside it.
  - The demo game mode respawns the player `RespawnDelaySeconds` (5) later.
  - With **Apply Demo Profile** off on the game mode, your own settings decide.
  - With no death tag, the character still dies and comes back, losing nothing.
  - See [Death and loss](death-and-loss.md) for the death drop and its policy.
- **`K`** is a raw key binding on `ASnapDemoPlayerController`, so it needs no input asset.
  - It asks the server, through `Server_KillDemoPlayer`, for one hit of the player's Health plus their Armor through the
    same damage rule the red pad uses.
  - So the line under the health bar reads it like any other hit.
  - It does nothing while you are dead.
  - The hotbar's point-at-a-square rebinding never hands `K` to a square.

## Attacks, the shield and the training dummies

Left click is your **main hand**, right click your **off hand**. What each one does depends on what that hand holds:

| The hand holds | Left click (main hand) | Right click (off hand) |
|---|---|---|
| Nothing | A right punch: half your Attack Power | A left strike: half your Attack Power — unless the main hand holds a weapon, which this button swings OVERHEAD instead (the sword, the axe or the greatsword) |
| A sword or the axe | A sweeping slash: your Attack Power — the sword fast and light, the axe slower and harder (its swing plays at 0.8, `SwingRate`) | Worn in the off hand, it slashes with the left: your Attack Power. With NOTHING in the off hand, right click swings the main hand's sword or axe **overhead** (raised over your head and brought down in front), with the same damage, cooldown, sounds and rate as its slash |
| The greatsword | A heavy two-handed sweep from your right shoulder across the front, every click: one and a half times your Attack Power, the slowest and hardest | The same weapon swung overhead (raised over your head and brought down in front), with the same damage and cooldown. It fills both hands, so both buttons swing it |
| The shield | — | **Block while held**: a hit from the front, and the red pad's, loses three quarters before your Armor; no attacks while it is up |

### Hits and the training dummies

**Every hit goes through the demo damage rule:** What is left after a block, minus the target's Armor, never below 1.

- So the three **training dummies** north of the stalls, with Armor 0, 10 and 20, show Attack Power against Armor at a
  glance.
- A number (big enough to read from across the room) rises over a dummy for every hit, and its bar drops.
- It is back to full three seconds after the last hit. A dummy knocked to 0 stands back up a moment later.
- **Your own hits on the red pad read the same way** in the line under your health bar: "-1 (shield blocked 15, Armor 4)".
  - The same rising number a dummy shows rises over your own head, red for a hit and green for a heal (on your screen).
- Attacks are ignored while the bag, a shop or the menu has the mouse, and while you are dead.
- Hits break the crate too.
- A swing hits only what can take a hit (a player, a dummy, the crate), never an item lying on the ground.

### A dummy's body and its swing

**A dummy's body swings on a spring.**

- Its stand stays put. The body above it sits on a physics hinge 30 cm up, tips up to 30 degrees either way, never
  twists, and a spring stands it back up.
- Every hit pushes it away from where the hit came from: an attack from the attacker, on either route, and every spell's
  blast from its center.
- A bigger hit pushes harder: a sword's 15 tips it about 10 degrees, a Fireball's burst about 15.
- It is upright again in under two seconds, after a couple of swings.
- The body is only for show and collides with nothing: it never pushes you.
- What your attacks and spells hit is the dummy's model, invisible and fixed (`DummyMesh`), so a hit lands exactly where
  it always did.

**A dummy swings back.**

- Come within 220 cm in front of one and it leans back for 0.6 s: the warning.
- Then it throws itself forward and hits you, the dummy as the source.
- **The hit is 16 plus 1.2 for every point of its own Armor** (`SwingDamage`, `SwingDamagePerArmor`): 16, 28 and 40 for
  the three, before your shield and Armor.
  - In the four-piece iron set (Armor 12 on Common copies), the Armor 20 dummy's 40 leaves 28.
- Hold the shield up facing it and the hit loses three quarters before your Armor, exactly like a hit from a player.
- Step back during the warning and it swings at nothing.
- It rests three seconds between swings, and it never swings at anyone behind it or at a dead player.

**Its bar shows when it matters**: Within 8 m of you, or for 4 s after it was hit, on your own screen — never from across
the map.

- Each player sees the bars near them.

### The weapons' numbers

The weapons' numbers, on Common copies (base Attack Power 10, Attack Cooldown 1 s):

| Weapon | Attack Power | Attack Cooldown | A hit |
|---|---|---|---|
| The sword | +5 | -0.2 s | 15, one every 0.8 s |
| The axe | +14 | +0.3 s | 24, every 1.3 s |
| The greatsword | +17 | +0.6 s | 40.5, every 1.6 s |

Armor is taken off every hit, so against the Armor 20 dummy the sword does 1, the axe 4 and the greatsword 20.5.

### Combat sounds

**Combat has sounds of its own:**

- A punch that lands (`USnapDemoAttackComponent::UnarmedImpactSound`).
- A punch on the crate that does not break it: a knock, instead of the punch's impact (`UnarmedKnockSound`).
- A dummy's swing as it throws itself forward (`ASnapDemoTrainingDummy::SwingSound`), and its hit when it lands on you
  (`HitSound`).
- Health you lose to anything that made no sound of its own, such as the red pad or a spell's blast
  (`USnapDemoAttackComponent::HurtSound`).

Each sound defaults to the demo's own plain sound in `Demo/Audio/Combat`; the demo's art pack plays its own takes (each
picked from several) instead. The punch's swing has a hook too (`UnarmedSwingSound`) and is silent.

**One sound per hit:** A hit that made its own sound at you (a weapon's or a punch's impact, a dummy's hit, a shield's
block) plays no hurt beside it.

### Two ways to make a weapon change the click

**Two ways to make a picked-up weapon change the click**, side by side, so you can copy the one your game uses. The demo's
route ([The demo's two routes](#the-demos-two-routes)) decides which one a click reaches:

1. **GameplayAbilities** (the GameplayAbilities route).
   - The demo PlayerState gives your character a punch, `USnapDemoAttack_Unarmed`, once for each hand.
   - A weapon's Granted Ability Set grants its own attack while you wear it:
     - `USnapDemoAttack_OneHanded` for the iron sword and the axe.
     - `USnapDemoAttack_TwoHanded` for the greatsword.
     - `USnapDemoAttack_Shield` for the shield.
   - That ability answers the click for the hand the item is in. It asks the equipment which slot its item is worn in,
     which is why one sword slashes from the right hand and strikes from the left.
   - Take the weapon off and its ability goes with it: the punch is back.
2. **No GameplayAbilities** (the native route).
   - Each of those items also carries a `USnapDemoWeaponFragment`: the weapon's style, which the first way reads too.
   - Its equip and unequip hooks (every item fragment's `OnEquipped` and `OnUnequipped`) tell a plain
     `USnapDemoAttackComponent` what each hand now holds.

**One set of numbers and one hit, on both.**

- A hit deals your Attack Power times the weapon style's share (`SnapDemoCombat::DamageScaleFor`: a punch half, a sword or
  the axe all of it, the greatsword one and a half).
- The time between attacks is your Attack Cooldown, one for both hands.
- Both are read off the demo's stats layer (`SnapDemoCombat::ResolveAttack`).
  - So the sword (which shortens the cooldown), the axe (which lengthens it) and the Elixir of Might move both routes
    alike.
  - The cooldown itself is the layer's on either (`USnapDemoStatsComponent::GetCooldown`).
- As an attack starts, your character turns to the point under the crosshair (`SnapDemoCombat::GetAimYaw`).
  - The camera sits over the right shoulder, so its own direction would pass beside the body.
- The hit lands in front of you along that direction (`SnapDemoCombat::LandHit`, the same on both routes), even if you
  turn while it winds up.

**Which route a click reaches:**

- In a copy with `GameplayAbilities`, the demo starts on the GameplayAbilities route.
- In a copy without it, the native route is the only one, and the **DEMO RUNS ON** pad in the FEATURES wing says so and
  refuses to switch.
- Stepping on that pad switches the whole demo's route: stats, damage and the attacks together, never one without the
  other (`ASnapDemoGameState::SetDemoRoute`, server only; see [The demo's two routes](#the-demos-two-routes) for what a
  flip does).
- The log names the route of every hit:

```
LogSnapInventorySystem: BP_SnapDemoCharacter_C_0: Slash landed (GameplayAbilities route, SnapDemoAttack_OneHanded) -- 15.0 damage (Attack Power x 1.00) to: BP_Example_TrainingDummy_C_0
LogSnapInventorySystem: BP_SnapDemoCharacter_C_0: Slash landed (native route) -- 15.0 damage to: BP_Example_TrainingDummy_C_1
```

### The animations

**The punches are Epic's; the swings are ours.**

- `MM_Attack_01` (the right punch) and `MM_Attack_02` (the left strike) come from the engine's Third Person template and
  ship beside Manny in `EpicDemoContent/Mannequin/Anims/Attack/`.
- **The weapon swings are keyed by us** on the same mannequin:
  - `AS_SnapDemo_SlashRight`: a one-handed slash, up over the right shoulder, across the front, low across the body.
  - `AS_SnapDemo_SlashLeft`: the same with the left hand (a sword or the axe in the off hand), starting and ending in the
    off-hand hold below, so it never snaps.
  - `AS_SnapDemo_HeavySwing`: the greatsword, both fists on the grip all the way through, two swings one after the
    other.
    - First an overhead chop (raised in front, over the head, brought down in front, low).
    - Then a sweep from the right shoulder across the front to the left.
- **The montages are ours too:**
  - `AM_SnapDemo_Punch`, `AM_SnapDemo_LeftStrike`, `AM_SnapDemo_Slash`, `AM_SnapDemo_OffHandSlash`.
  - `AM_SnapDemo_HeavySlash` and `AM_SnapDemo_HeavySlash2`: the two halves of the greatsword's sequence, the overhead chop
    on right click, the sweep on left click.
  - `Demo/Animations/AM_SnapDemo_OverheadSlash`: the one-handed overhead's own montage, which plays our slash a little
    slower, its crossing at 0.40 s, when the demo's art pack is off.
  - The looping `AM_SnapDemo_ShieldRaise`: a raised-shield pose keyed by us, since the template has no block, its forearm
    turned so the shield faces forward over the hand.
- **So is the off-hand hold `AS_SnapDemo_OffHandHold`**, blended onto the left wrist before the slot while a one-handed
  weapon is in the off hand and you stand still.
- **`ABP_SnapDemoMannequin` plays them on the upper body only**, so you can walk while you swing.
  - Its class defaults name them, all but the one-handed overhead's. That one is a C++ default of
    `USnapDemoAnimInstance` (`OverheadSlashMontage`), so the Animation Blueprint was not re-saved.
  - An item names a weapon style, an attack names a movement, and the character's animation answers with a montage.
- Delete `EpicDemoContent` and the attacks still land, without a movement.

**With the demo's art pack on (the default), four of the swings are the pack's.**

- The one-handed slash, its mirror for the off hand, the one-handed overhead and the greatsword's overhead play the
  pack's own motions, the chosen takes, from `DemoArt/Animations` (the pack's `Animations`
  rows).
- They are fitted to the moments the hits land, both fists on the greatsword's hilt on every frame.
- The greatsword's sweep, the punches and the raised shield are ours either way.
- `Snap.Demo.ArtPack 0` swings with ours.

### The Ring of Vitality's passive

**The Ring of Vitality has a real passive**: Below a third of your Max Health it restores 20 Health, at most once every 30
seconds.

- It is its `USnapDemoPassiveFragment`, which the demo's stats layer runs on either route.
- Get hurt on the red pad to see it.
- The Warrior's Signet has no passive.

## Your character, and the sword in its hand

**The demo character wears Epic's UE5 mannequin (Manny)**, which the plugin ships as demo content in its own folder,
`Content/EpicDemoContent/Mannequin/`. That folder holds:

- The mesh, its skeleton, physics asset, materials and textures.
- Six forward-only animations (idle, walk, jog, jump, fall, land).
- A small Animation Blueprint and blend space of ours that drive them.
- In `Anims/Attack/`, two of the template's attack clips (the punches), beside the montages, weapon swings and poses of
  ours (see [Attacks, the shield and the training dummies](#attacks-the-shield-and-the-training-dummies)).

It is Epic Games content shipped as demo content, kept apart so you can see exactly what it is — and delete it.

- `BP_SnapDemoCharacter` names it through two soft pointers of its own, **Character Mesh** and **Character Anim Class**
  (**Class Defaults → Snap Demo**).
- The mesh the character wears is the first of these three that gives one, and the log line it writes at spawn says
  which:

1. **Demo Character Mesh** in **Project Settings → Plugins → Snap Inventory System - Inventory UI → Examples** — your own rig, when you
   set it.
2. The Blueprint's own **Character Mesh** — the shipped mannequin — used only when its package is installed.
3. `/Engine/EngineMeshes/SkeletalCube`, which every install has.

```
LogSnapInventorySystem: BP_SnapDemoCharacter_C_0: wearing mesh 'SKM_Manny_Simple' from this Blueprint's CharacterMesh, animation class 'ABP_SnapDemoMannequin_C'.
```

**The animation class comes from the same source as the mesh.**

- Set Demo Character Mesh and the character uses **Demo Character Anim Class** alone. Empty leaves your rig in its
  reference pose.
- The shipped Animation Blueprint is never put on a skeleton it was not made for.

**Delete `Content/EpicDemoContent/` and the demo keeps working**, on the cube.

- The character checks that the mannequin's package is installed before loading it, so for the character a deleted
  folder is a quiet fallback rather than a load warning.
- **Outside the folder, only the demo's own content names it:**
  - The character (by soft pointer).
  - The two example NPCs (the Courier and the Trader) and their four material instances.
  - The demo's art made on the mannequin's skeleton (the skinned worn pieces and the drink in `DemoArt`).
- **Measured on 5.8.3 with `DemoArt` present:**
  - The map then loads with 8 load errors, all naming the two NPCs' Blueprints and material instances (their mannequin
    body cannot load).
  - The art pack still registers.
  - Worn pieces draw nothing on the cube, which has no hand or foot bones.

> **Note:** The cube is an editor convenience.
>
> - It is loaded by path, so it creates no cook reference and nothing force-cooks `/Engine/EngineMeshes`.
> - So in a **packaged** build without the mannequin, the character is invisible (handled, and logged).
> - The shipped mannequin has no such problem: the Blueprint's soft pointer is a cook reference, so a packaged demo wears
>   it with nothing configured.

**The mesh also decides where an equipped weapon lands**, and that is worth understanding because it is the first thing
anybody notices:

| | The engine cube | The shipped mannequin, or your humanoid rig |
|---|---|---|
| Body | A 55 x 55 cm column as tall as the capsule, standing on the floor (sized from the cube's own bounds) | The mannequin, or your character |
| `hand_r` present | No — the cube has two bones | Yes |
| Where `ID_Sword_Iron`'s visual attaches | The mesh origin — the column's base, on the floor — with `AttachOffset` at its authored length, so the sword hangs inside the column, its grip about 35 cm off the floor | The `hand_r` bone: the grip in the right fist, the blade pointing the way the character faces |

**Neither is an error.**

- `USnapItemFragment_Equippable::AttachSocketName` names the socket or bone an item wants.
- `AttachOffset` nudges it from there.
- A name the rig does not have is **detected** rather than silently ignored.
  - `USnapEquipmentVisualComponent::SpawnEquipmentVisualFor` falls back to the mesh origin and logs which of the two paths it
    took:

```
LogSnapInventorySystem: Verbose: SpawnEquipmentVisual: BP_Example_SwordVisual_C attached to socket/bone 'hand_r' on CharacterMesh0, from the item's own socket.
LogSnapInventorySystem: Verbose: SpawnEquipmentVisual: BP_Example_SwordVisual_C asked for socket/bone 'hand_r', which CharacterMesh0 does not have -- attached at the mesh origin with AttachOffset at its authored length instead. Author that name on this rig, or adjust AttachOffset to suit it.
```

- Turn those on with `log LogSnapInventorySystem Verbose`.
- Without the fallback, an item authored for one rig and worn on another attaches at the component root with nothing
  logged, which is the classic "the sword is floating at the character's feet" problem.

**`DemoCharacterMeshOffset` and `DemoCharacterMeshRotation` place your rig against the capsule** (**Project Settings →
Plugins → Snap Inventory System - Demo → Examples**, a different page from Demo Character Mesh).

- The shipped values are the usual convention for a rig whose origin is between its feet.
- They are not applied to the fallback cube. `ASnapDemoCharacter` places that one itself, sized from the cube's own bounds
  (`SkeletalCube` is a 25 cm cube whose origin is its base), so it stands on the floor and fills the capsule.

### What the character wears

Equip an example item that can be seen and it is drawn on the body, in play and in the bag's character preview, and other
players see it too:

| Item | Drawn as |
|---|---|
| `ID_Chest_Iron`, `ID_Chest_Gilded` | The cuirass on the upper body (`spine_03`), pauldrons, belt and faulds included |
| `ID_Greaves_Iron` | A greave on each shin (`calf_l`, `calf_r`) |
| `ID_Gloves_Iron` | A gauntlet on each hand (`hand_l`, `hand_r`) |
| `ID_Boots_Iron` | A sabaton on each foot (`foot_l`, `foot_r`) |
| `ID_Example_Helmet` | The Knight's Helm on the head (`head`) |
| `ID_Example_Shield` | The round shield on the left forearm (`lowerarm_l`), face outward, the left hand on the grip on its back |
| `ID_Sword_Iron`, `ID_Example_Axe_Iron` | In the right fist (`hand_r`) in the Main Hand, in the left fist (`hand_l`) in the Off Hand — standing still, the demo's animation turns the left wrist (`AS_SnapDemo_OffHandHold`) so it points ahead like the right one |
| `ID_Example_Amulet` | The pendant on the chest, on the bone the cuirass rides (`spine_03`), so it hangs in front of the cuirass when both are worn; its chain lies on the cuirass's collar round the neck and down the outside of the plate (one model for every wearer: on a bare chest it stands clear of the skin where the collar would be) |
| `ID_Example_Greatsword` | In the right fist (`hand_r`), as before |

The rings are worn but not drawn: too small to read at play distance.

- **Each armor piece is a rigid model pinned to one bone** by `ASnapEquipmentPartsVisual` — see
  [Armor made of pieces](equipment.md#armor-made-of-pieces).
  - The example Blueprints (`BP_Example_CuirassIronVisual`, `BP_Example_GreavesVisual`, ...) hold nothing but their
    pieces.
  - Their offsets were fitted to the shipped mannequin from its own skin in its reference pose, so a rig of a different
    build may want its own Blueprint.
- **With the art pack on, the gauntlets, boots and shin guards are the exception.** Each pair is one skinned mesh that
  bends with the body (`SK_DemoArt_Gauntlet_Worn`, `SK_DemoArt_Sabaton_Worn`, `SK_DemoArt_Greave_Worn` — the pack's
  `SkinnedParts` rows).
  - The gauntlets' finger plates curl with the character's fingers.
  - The boots' collars follow the ankle.
  - The shin guards follow the shin.
  - With the pack off, they are the example's rigid pieces.

⚠ **The shield, a second sword and the helmet need slots the plugin's default list does not have.**

- The demo applies `DA_ExampleEquipmentSlotSet` for each play session, which adds an Off Hand beside the Main Hand and a
  Head slot.
- A project of yours that keeps the default slots keeps the shield and the helmet in the bag.

On the engine-cube fallback the armor equips as usual and draws nothing: the cube has none of those bones, and a piece
whose bone the mesh lacks is skipped rather than drawn at the mesh origin.

**The example swords' `AttachOffset` was measured on the shipped mannequin at idle:**

- A rotation that lays the blade along `hand_r`'s forward axis with its edges up and down.
- A translation that puts the grip on the palm's center.
- A rig of yours with a `hand_r` bone oriented the same way takes them as they are.
- A rig whose hand bone points another way needs its own `AttachOffset`.

## The demo's two routes

- **Every demo number and the death are read and written through one layer**, `USnapDemoStatsComponent` (on the
  PlayerState, and on each training dummy).
  - The numbers are Health, Max Health, Armor, Attack Power and Attack Cooldown.
  - So the pads, the dummies, the HUD, `K` and the death never touch an ability system themselves.
- The layer answers from ONE of two backends at a time:

| Route | Backend | Available |
|---|---|---|
| **GameplayAbilities** | The ability system and `USnapDemoAttributeSet` (`USnapDemoGasStatsBackend`) | Only in a copy compiled with `GameplayAbilities`; there it is the default |
| **Native** | Plain replicated numbers on the layer itself | Always; the only route without `GameplayAbilities` |

- **The route is one replicated value on the demo's game state**, `ASnapDemoGameState::SetDemoRoute` (server only).
  - It refuses GameplayAbilities in a copy without it.
- **Both routes call the same rules and start from the same numbers**, so a red pad tick costs the same on either:
  - The same Armor rule (`SnapDemoStats::ComputeDamageTaken`: a hit minus Armor, never below 1).
  - The same shield share (`SnapDemoStats::ComputeBlockedShare`).
- **On the native route, the death calls the plugin's own doors:**
  - `USnapInventoryComponent::NotifyOwnerDied` for the drop and the body.
  - `USnapInventoryHUDComponent::DismissScreensForDeath` on the dying player's machine.
  - `NotifyOwnerRespawned` at the respawn.
  - And the PlayerState reports the death tag through `IGameplayTagAssetInterface`, which is what the plugin's
    "is this player dead?" refusals read.
- See [Death and loss](death-and-loss.md#saying-the-player-is-dead).

**What you wear counts on both routes.**

- The example items carry their base stats as stat rows (see [Equipment](equipment.md#stats-an-item-declares-itself)),
  and the iron set's tiers are rows too.
- **The GameplayAbilities route** applies them through `DA_ExampleStatEffectMap`.
- **The native route** derives Max Health, Armor, Attack Power and Attack Cooldown from
  `USnapEquipmentComponent::GetEquippedStatTotal`, every time what you wear changes
  (`USnapDemoStatsComponent::RecomputeNativeStats`).
  - It works out (base + Flat) x (1 + Percent / 100), the number the ability system's own arithmetic reaches.
- The layer is also the plugin's stat source (`ISnapStatSource`), so an item's requirement (the Warrior's Signet's Attack
  Power 12) reads the route that is active.
- On the native route a hit wears your equipment through `USnapDamageWearComponent::ApplyDamageWearFor` (the C++ form
  of the Blueprint `ApplyDamageWear`), which the
  GameplayAbilities route reaches through the plugin's own Health watch.

**So do the potions.** Each potion's numbers are its `USnapDemoConsumableFragment`'s, and the layer does them on the active
route (`USnapDemoStatsComponent::ApplyConsumable`):

- The heals go through its own heal.
- A timed raise is an effect on the GameplayAbilities route, and in `RecomputeNativeStats` on the native one: the same
  (base + Flat) x (1 + Percent / 100).
- The layer's own timers run the heals over time and the raise's end, on both.
- See [The potions](#the-potions).

**And the two passives.** The Ring of Vitality's restore and the Amulet of Mending's regeneration are each a
`USnapDemoPassiveFragment` on the item.

- It hands its numbers to the layer when the item goes on (`USnapDemoStatsComponent::AddPassive`), and takes them back
  when it comes off.
- Every heal goes through the layer's own heal, on the route active when it lands.
- The ring's restore watches the layer's Health on either route.
- Nothing heals while the item is broken or spoiled, while you are dead, or at full Health.

**And the attacks.**

- A click reaches the GameplayAbilities route's attack abilities, or the native route's `USnapDemoAttackComponent`, by
  the route itself.
  - There is no separate weapon switch to disagree with it.
- Both deal and cool down from the layer's Attack Power and Attack Cooldown. See
  [Attacks](#attacks-the-shield-and-the-training-dummies).

### Switching routes

**Switching routes: the DEMO RUNS ON pad, and what a flip does.**

- One pad in the FEATURES wing switches the whole demo.
  - Its sign reads the route in force.
  - Without `GameplayAbilities` it reads BUILT-IN ONLY: NO GAS IN THIS BUILD, and a step changes nothing.
- The flip is `ASnapDemoGameState::SetDemoRoute`'s, on the server, and every client follows.

For every player and every training dummy:

| A flip... | |
|---|---|
| Ends | Every timed buff (the potions, Haste, Ward) and every cooldown (the attacks', each spell's) — on BOTH routes, so nothing waits on the route you left; a raised shield comes down; a ground spell's ring goes away with no cooldown spent |
| Re-reads | Every stat from what is worn, in the new route |
| Keeps | Health, capped at the new route's Max Health — 120 of 125 with a Vigor potion working reads 100 of 100 after it |
| Never moves | An item: each player's item count is logged before and after the flip (`item counts across the route flip: EQUAL`), and a Warning names a player whose count moved |
| Waits | While a player is dead — the death lives on the route it happened on — and the pad says so |

- A late joiner and a respawned player get the route in force.
- What a worn item does is not a buff, and keeps running across a flip: the ring's restore and the amulet's regeneration
  (above), with the ring's 30-second wait before it restores again.

## The classes, and the recipe they are

- Five classes on plain engine bases, so they run in an empty template and in a project that already has its own
  framework.
- They are compiled with or without `GameplayAbilities`:
  - In a copy with it, `ASnapDemoPlayerState` also makes an Ability System Component and `USnapDemoAttributeSet` (the
    layer's GameplayAbilities route reads them).
  - In a copy without it, neither exists, and the layer's native route is the only one.

| Class | Base | Does |
|---|---|---|
| `ASnapDemoGameMode` | `AGameModeBase` | Names the other four; attaches components on `PostLogin` |
| `ASnapDemoGameState` | `AGameStateBase` | Holds the demo's route (`SetDemoRoute`), set on the server and replicated so every machine agrees |
| `ASnapDemoPlayerController` | `APlayerController` | Attaches components on `BeginPlay` |
| `ASnapDemoPlayerState` | `APlayerState` | Holds the player's numbers (`USnapDemoStatsComponent`, and with GameplayAbilities the Ability System Component) |
| `ASnapDemoCharacter` | `ACharacter` | Third-person pawn; attaches components on `PossessedBy` and, on every machine, `BeginPlay` |

- Blueprint children of all but the game state live in `/SnapInventorySystem/Demo/Blueprints/`, and the map names
  `BP_SnapDemoGameMode`.
- So you can change the pawn, the camera or the movement speeds without opening a compiler.

### If your project is not built on Modular Gameplay Actors

**This is the part worth copying.**

- The plugin's automatic attachment matches **receiver classes by path**, and the shipped rows name the Modular Gameplay
  Actors bases.
- A plain `APlayerState` is not a Modular Gameplay receiver, so those rows attach nothing to it — correctly, and quietly.

The answer is one call:

```cpp
USnapFoundationLibrary::EnsureSnapInventoryComponents(PlayerState, PlayerController, Pawn);
```

- It reads the **same** settings rows the automatic path reads, so the two can never describe different component sets.
- It is idempotent — it checks for each component before adding it.
- The demo calls it from four places, because there is no single moment at which all three actors exist:

1. `ASnapDemoGameMode::PostLogin`, after `Super` — the controller and its PlayerState exist.
2. `ASnapDemoPlayerController::BeginPlay` — covers a controller the automatic path never saw.
   - In a standalone or packaged launch the engine loads the map and logs the first player in *before* the plugin's
     requests are registered.
   - So the first player of every such session takes this path.
   - A player added during play (split screen) has no local player yet at its BeginPlay, so the demo waits for it
     (`ReceivedPlayer`); `PostLogin`'s call attaches the components in between. The plugin's own components wait for the
     local player themselves, so a call from BeginPlay is fine for them; the demo waits for its own HUD and menus.
3. `ASnapDemoCharacter::PossessedBy` — the pawn finally exists, and this one **runs again on every respawn**, which the
   other two do not.
4. `ASnapDemoCharacter::BeginPlay` — on **every machine, for every pawn**, other players' included.
   - The interactable a pawn carries (it is what puts **Trade** on another player) is local-only.
   - It is never replicated, so the copy the server adds reaches nobody, and a client that did not add its own never sees
     the option.
   - The first three all run on the server or for the local player. Copy this one without an authority check.

It is also callable from Blueprint, so a Blueprint-only project can do the same from its own GameMode and
PlayerController.

## An item you cannot wear yet

- `ID_Example_WarriorsSignet` is a ring that asks for **Attack Power 12**.
- The demo character starts at 10, and `ID_Sword_Iron`'s stat row adds 5.
- **Where to get it:**
  - It lies under the **ITEM RULES** sign in **JEWELRY AND A QUEST**.
  - The Gem Trader also sells it for 20 gems.
  - Or give it to yourself from the console:

```
Snap.GiveItem ID_Example_WarriorsSignet 1
```

**Try it:**

1. Hover it. You should see the tooltip read `Attack Power  12 (you have 10)` in the loss color.
2. Right-click it. You should see **Equip** grayed, naming the requirement.
3. Drag it onto a ring slot. You should see a notice say why.
4. Put the sword on. All three change: the line turns neutral with your new value, and the ring equips.

## An item that looks like itself on the floor

- **Every example item carries a `WorldRepresentation` fragment** naming its own low-poly model in
  `Content/Examples/Meshes/`, at a scale of its own.
  - So the sections show the items themselves.
- **An item without the fragment lands as one of four small engine shapes**, picked by its name and tinted by its rarity.
  - An item of yours does exactly that until you give it a mesh.
- Each example item's icon in the bag is a render of that same model (`Content/Examples/Art/`).
- The empty equipment slots show flat silhouettes of the demo art's own models (`Content/Core/UI/Textures/`, with the
  plugin's core art, so with the art pack off too — see Demo art provenance).
- Every example item lies in one of the sections: the totem by the Junk Merchant, the shield and the greatsword in
  **WEAPONS**, the Warrior's Signet and the Sealed Letter under the ITEM RULES sign.

To see one land the way a dropped item does, give it to yourself from the console and drop it:

```
Snap.GiveItem ID_Example_Totem 1
```

## Gold, silver and gems

- The demo has three currencies, and every shop deals in exactly one of them.
- Your bag draws all three in a row at its foot: gold, silver, gems, in that order.
- Point at a number for its name.

| Currency | Where you get it | What it buys, and where |
|---|---|---|
| Gold — the gold coins, `ID_Example_Coins`, which feed the plugin's own `Counter.Currency` | The 250 gold coins in front of the General Store; the crate; selling to the General Store | The General Store (red stall, the first you reach): the iron armor and sword, the Ring of Vitality, potions, the pouch, repair kits, gold bars and the bag upgrades — and its repairs, upgrades and crafting |
| Silver — `ID_Example_SilverCoins`, `Counter.Silver` | 60 silver coins in front of the Junk Merchant; the crate; selling to the Junk Merchant, which buys anything that has a value | The Junk Merchant (ochre stall, the third): Health Potions, Repair Kits, Raw Meat, Bread and the Carved Totem |
| Gems — `ID_Example_Gem`, `Counter.Gems` | 25 gems in front of the Gem Trader; now and then from the crate; selling a gem item — the Gilded Cuirass from **ARMOR**, say — to the Gem Trader | The Gem Trader (teal stall, the second): the Warrior's Signet, the Iron Shield, the Iron Greatsword, the Gilded Cuirass and the scrolls of Haste, Fireball and Warding — sold nowhere else, and only for gems |

What each shop charges, in its own currency. A shop pays back a quarter of its price for what it sells; the Junk Merchant
pays a tenth.

| Shop | Sells | Buys |
|---|---|---|
| General Store (gold) | Iron Sword 250, Iron Cuirass 180, Knight's Helm 120, Iron Greaves 140, Iron Sabatons 110, Iron Gauntlets 95, Ring of Vitality 320, Health Potion 40, Leather Pouch 150, Repair Kit 30, Gold Bar 60, bag upgrades 400 / 900 / 1800 | What it sells |
| Junk Merchant (silver) | Health Potion 15, Repair Kit 12, Raw Meat 5, Bread 4, Carved Totem 40, Potion of Regeneration 20, Elixir of Might 30, Ironskin Potion 30, Tonic of Vigor 35 | What it sells, and anything else that has a value: Rotten Mush for 1 silver, the Iron Sword for 20 — in silver, unless the item names its own currency |
| Gem Trader (gems) | Warrior's Signet 20, Iron Shield 12, Iron Greatsword 25, Gilded Cuirass 30, Scroll of Haste 8, Scroll of Fireball 12, Scroll of Warding 10 | What it sells |

**Enough to spend, more than once.**

- The gold, silver and gem pickups come back 30 seconds after they are taken (`RespawnSeconds` on the placed pickups).
- So a few minutes of play pays for every shop several times over. In three minutes:
  - The 60 silver comes back six times: 360 silver, against 187 for one of everything the Junk Merchant sells.
  - The 25 gems come back as often: 150 gems, against 113 for one of everything at the Gem Trader.
  - The 250 gold coins give 1,500 gold for the General Store.
- The crate adds silver and gems each time it breaks, and it is back 45 seconds later.
- Anything you buy sells back: a shop pays a quarter of its price, the Junk Merchant a tenth.

**How the shops deal:**

- A BUY you cannot afford is grayed, and the server refuses the purchase too, taking nothing.
- **An item sells only for its own currency.**
  - The General Store and the Gem Trader buy back only what they sell.
  - The Junk Merchant buys anything with a value, and pays in that item's own currency where it names one: the
    greatsword and the gilded cuirass are worth gems, so it pays gems for them.
  - The Warrior's Signet, the shield and the scrolls carry no value, so only the Gem Trader buys them.
- **Only the General Store repairs and upgrades**, in gold; the other two shops show those services as not offered.
- **All three craft:** The General Store its own list, the Gem Trader and the Junk Merchant the recipes that need no
  station.
  - A timed craft ordered at any of them that finishes while you are away waits at that stall, its icons over it, until
    you open it again.
- **Every shop restocks on the VENDOR RESTOCK pad's rule** (see
  [the FEATURES wing](#the-features-wing-switch-a-plugin-feature-for-this-session)).
  - When the demo starts, one unit of each limited item comes back every 20 seconds, in every shop — even the Gem
    Trader's one greatsword.
  - The pad switches every shop to **when sold out** (a row that sells out is back 30 seconds later) and to **whole shop**
    (60 seconds after a shop was last full, everything is back).
  - It is the plugin's project-wide restock override (`bOverrideVendorRestock`), so for the play session it replaces the
    shops' own assets.
  - The Junk Merchant's own asset brings two potions back every 15 seconds and a repair kit every 20; the other two shops'
    own assets never restock.
  - The vendor window says the rule under the shop's name, and a restock is collected when you next open the shop.
- When you die, all three land beside your body as pickups (see [the pads](#the-heal-and-damage-pads-and-dying)).

**The three currencies are configuration, not code.** The demo sets them for its play session. A game of your own that
wants the same row puts these lines in its `Config/DefaultGame.ini`:

```ini
[/Script/SnapInventorySystem.SnapInventorySettings]
+Currencies=(CounterTag=(TagName="SnapInventorySystem.Counter.Currency"),DisplayName=NSLOCTEXT("SnapInventoryDemo", "Currencies.Gold", "Gold"),Icon="/SnapInventorySystem/Demo/Icons/T_Currency_Gold.T_Currency_Gold",BaseUnitValue=10)
+Currencies=(CounterTag=(TagName="SnapInventorySystem.Counter.Silver"),DisplayName=NSLOCTEXT("SnapInventoryDemo", "Currencies.Silver", "Silver"),Icon="/SnapInventorySystem/Demo/Icons/T_Currency_Silver.T_Currency_Silver",BaseUnitValue=1)
+Currencies=(CounterTag=(TagName="SnapInventorySystem.Counter.Gems"),DisplayName=NSLOCTEXT("SnapInventoryDemo", "Currencies.Gems", "Gems"),Icon="/SnapInventorySystem/Demo/Icons/T_Currency_Gems.T_Currency_Gems",BaseUnitValue=100)

[/Script/SnapInventorySystem.SnapInventoryComponent]
+CounterItemDefinitions=/SnapInventorySystem/Examples/Items/ID_Example_Coins.ID_Example_Coins
+CounterItemDefinitions=/SnapInventorySystem/Examples/Items/ID_Example_SilverCoins.ID_Example_SilverCoins
+CounterItemDefinitions=/SnapInventorySystem/Examples/Items/ID_Example_Gem.ID_Example_Gem
```

- **`Currencies` is what draws a balance in the bag.** The plugin ships it empty, which shows the gold readout alone.
- **Each row's `Icon`** is a render of that currency's own pickup model (the gold coins, the silver coins, the gems).
  - Every price in that currency wears it, the SELL button included.
- **`CounterItemDefinitions` says what each currency drops as.** Without the silver and gem lines, a death keeps those two
  in the purse.
- **`BaseUnitValue` matters only to `PayCurrencyPrice`**, the wallet that spends across currencies, which no demo shop
  uses.
- **Each shop names its own currency on its stock asset:** `DA_ExampleJunkMerchant` silver, `DA_ExampleGemShop` gems.
  - The General Store's, `DA_ExampleGeneralStore` (a copy of the plugin's `DA_ExampleVendorStock` that adds only its
    name), names none, so it uses the project's, gold.
- See [Currency and prices](currency-and-prices.md).

## The shops, the forge and the signs

Every prop in the map is one of the plugin's own low-poly models, built the same way as the items and sharing their
material, `M_ExampleProp`:

- **The three vendor stalls** (`SM_Example_VendorStall`, drawn by `BP_Example_VendorStall`):
  - One Blueprint serves every shop: the awning's colored stripe is material slot 2, overridden on each placed stall —
    red for the General Store, teal for the Gem Trader, ochre for the Junk Merchant.
  - Each stall keeps its own `VendorStock`.
  - A sign on its awning names the shop in its currency's color: GENERAL STORE in gold, GEM TRADER in green, JUNK
    MERCHANT in silver.
  - Under each name a line says how that shop restocks right now: an `ASnapDemoVendorRestockSign` pointed at the stall's
    own stock, which follows the **VENDOR RESTOCK** pad.
- **The forge** (`SM_Example_Forge`, drawn by `BP_Example_Forge`): A stone hearth with glowing coals and a chimney, an
  anvil on a stump and a workbench with tools.
- **The chest** (`SM_Example_Chest`, opening to `SM_Example_ChestOpen`, emptied `SM_Example_ChestOpenEmpty`):
  - Set on the placed chest as its `ContainerMesh`, `OpenedMesh` and `EmptiedMesh`.
  - Opening it shows the treasure heap while anything is left, and the open chest with nothing in it once you have taken
    everything.
- **The body** (`SM_Example_BodyPack`, opened `SM_Example_BodyPackOpen`, emptied `SM_Example_BodyPackLooted`): A fallen
  adventurer's camp, set on the placed body.
  - Opening it throws the pack's flap open and leaves the shield and sword on the ground while anything is left.
  - Once it is empty the gear is gone and the bedroll unrolled.
  - Placed in the map, it never times out: `LootableBodyLifetimeSeconds` counts down only a body spawned while the game
    runs.
  - A body the plugin spawns when a player dies is `BP_Example_LootableBody` in the example project: the same pack and
    the same three looks, because the project names it as `LootableBodyClass`.
  - The plugin's own default class draws the engine cube at half size.
- **The bank chest** (`SM_Example_Chest`, drawn by `BP_Example_BankChest`):
  - The loot chest's model with its iron bands in gold, so the two read apart.
  - It never opens: it is a way into your bank, not a container of its own.
- **The crate** (`SM_Example_Crate`, drawn by `BP_Example_DestructibleCrate`):
  - Breaking it hides it, exactly as it hid the cube it replaced.
  - It bursts into eight pieces cut from its model (`Examples/World/CratePieces`, the `BreakPieces` list) that tumble and
    shrink away.
  - It throws out three to five drops from `DA_ExampleCrateLoot`.

**Their sizes and places:**

- The forge (1.7 times its model, 2.9 m tall) and the three stalls (1.5 times, 3.3 m) stand taller than the player.
- The crate is 1.6 times its model.
- The forge stands in **CRAFTING**, just before the shops.
  - Its sign is centered over it, high enough that the icons of a finished craft waiting over it stay below the sign.
  - The **VENDORS** sign over the Gem Trader is as high over the stalls, for the same reason.
- Each prop stands on its origin and collides with its own shape, so the player stops at a counter, an anvil, the chest
  or the crate.
- The body, like every lootable body, lets the player walk through it.

### The chest, the body and the crate

- The chest and the body each show three states, and every machine shows the same one.
  - The look comes only from replicated state: whether the container has been opened, and whether anything is left in
    it.
- The body here is placed in the map, so it never times out, and an emptied one stays where it is.
- The pack a death leaves is the one that goes once it is empty and nobody is beside it.

| State | The chest | The body |
|---|---|---|
| Not opened yet | A closed chest | The pack closed, the shield and the sword on the ground |
| Opened, loot left | The lid up on a heap of gold | The pack's flap open, the shield and the sword still there |
| Emptied | The lid up on an empty chest | The flap open, the gear gone, the bedroll unrolled |

**They refill.**

- On the demo map the chest and the body each fill again a minute after they are opened and emptied, re-rolled from their
  loot table.
- The crate is back 45 seconds after it breaks.
- This is `SpawnPolicy.RespawnSeconds` on the placed instances (60, 60 and 45); the plugin's own default is never.
- A chest or body that still holds something when its minute is up is left as it is, and nothing is armed again until it
  is next opened.
- The pack a death drops never refills.

**Hit the crate and it breaks.**

- It takes 20 damage: four punches or two slashes.
  - The plugin's own crate also breaks when walked into; the demo's has `bBreakOnOverlap` off, so the attacks have
    something to do.
- Three to five drops are thrown out of it onto the side away from you, and land at least 1.6 m from where you stood.
  - The drops: gold or silver coins, gems, a health potion, meat, bread, a gold bar or a piece of iron armor.
- A drop in the air cannot be walked into; if you are standing on one when it lands, you pick it up then.
- See [Loot and interaction](loot-and-interaction.md).

### The walls

**The floor is one long hall, about 195 m by 40 m**: A subtle 2 m grid, with fainter lines every 50 cm between.

- The sections run down one side, each on a platform 2 cm high in its own color.
  - The platforms are `MI_ExampleSection_*`, material instances of `M_ExampleGridFloor` in `Demo/Maps/Materials`, so
    their grid lines run on from the floor's.
- The pads run down the other side, each block on a platform of its own color (`MI_ExampleSection_Features*`), with a
  6-m walkway between.

**It is ringed by walls 5 m tall** that carry the same grid in their own gray.

- `M_ExampleGridWall` (beside the map, in `Demo/Maps/Materials`) lays the floor's grid on each wall's own plane, so it
  reads the same on every side.
- The demo character jumps about 1.28 m, and nothing a player can stand on is higher than a stall's roof (3.3 m).
- So no jump from anywhere in the map reaches the top of a wall (3.3 + 1.28 m is under 5).
  - The walls used to be 1.2 m curbs, which a running jump from the floor cleared.

### Every pickup takes both routes

**Every pickup in the map uses the class default, `Both`, with no auto-pickup filter:** Walk over it, or look at it and
press `E`.

- The sword used to be the one exception (a filter left it to the key alone), and now behaves like the rest.
- See [Pickup modes and the collect feed](pickup-modes-and-the-collect-feed.md) to make a pickup take only one route.

### Interaction reach

**The interaction prompt floats just above the top of an actor's colliding shape.**

- On a stall that is the awning, 3.4 m up, and reach is measured from the player to that point.
- So the three stalls carry a wider reach of their own (`InteractionRangeOverride` 440 on each placed one, the forge
  360).
- A stall offers itself from about 2.4 m in front of its counter, the forge from about 2 m in front of it, and the chest
  and the body from about 2.8 m away.

### The sign boards

**Every sign in the map is the demo's sign actor, `BP_Example_DemoSign`** (see [Signs that teach](#signs-that-teach)).

- It is a title and rows on a dark board, `SM_Example_SignPlate`, which has no collision and casts no shadow.
- A stall's sign is its shop's name alone, on the same board, and is a plain text render on its awning.
- Their text is drawn with `M_ExampleSignText`: the engine's own text material made unlit, so a sign shows its own color
  in any light.
- Every sign's text is keyed in the `SnapInventoryDemo` namespace (see [Localization](localization.md)).

> ⚠ A text render's color is applied as given, as a linear value: a bright display color set as-is renders pale.
> The demo's sign colors are the linear values of the colors they show.

## The bank, and a world that remembers

### Your bank

- **The chest under the BANK sign, to the right of the training dummies, is YOUR bank:** A container that belongs to your
  demo profile rather than to your pockets.
- Look at it and the prompt reads **(E) Open your bank  ·  Hold (E) Deposit everything**: one key, two actions (the
  plugin's tap action, see [One key, tap or hold](loot-and-interaction.md#one-key-tap-or-hold)).

**The two actions:**

- **Open your bank:** TAP `E`.
  - The bag opens with the bank's 24 squares beside it.
  - Drag items between the two, or Shift+click one to send it across: into the bank from the bag, back out of the bank (a
    spell scroll goes back to Spells).
  - Right-click offers **Move to Bank** / **Move to Bag**.
  - A pouch that still holds something stays in the bag, and currency never goes in.
  - Close the bag and the bank closes. Walk more than 4.5 m away and the bank closes with the bag.
- **Deposit everything:** HOLD `E` for a second. The ring after the prompt's words fills as you hold.
  - Every item in your bag goes in at once; what you wear stays worn.
  - When the bank is full the rest stay in your bag, and a notice says so.

**It is saved.**

- The bank is written to the demo's own save (`SnapInventoryDemo_<player>`, a store of its own, `SnapDemoBank`) when its
  window closes, after a deposit and when you stop playing.
- It is read when you start.
- Nothing else in the demo is saved between plays unless you switch **SAVE YOUR BAG** on in the FEATURES wing (the
  whole-inventory auto-save starts off in the demo).
  - So the bag starts empty each time, and the bank does not.
- A death never touches it: the demo's death-drop policy keeps it.
- In a two-player session each player has a bank of their own, under their own profile.

To see it:

1. Put something in.
2. Stop playing.
3. Start again — even after restarting the editor.
   - You should see it still there.

**It is used only at its chest — and the server checks.**

- Opening the bank (or depositing) opens an access session at that chest on the server.
- Closing the bank, walking more than 4.5 m away or dying ends it.
- Every move into or out of the bank without one is refused with "That container can only be used where it is kept." —
  even from a modified client.
- This is the plugin's reach rule, switched on for the bank:
  [Custom containers](custom-containers.md#a-container-used-only-where-it-is-kept-the-reach-rule).

How it is built: [Custom containers](custom-containers.md#a-container-that-takes-anything-saved-on-its-own-the-bank-example)
and [Persistence](persistence.md#a-container-that-keeps-its-own-save).

### A world that remembers

- **The plugin can save the world itself and restore it next time** (`USnapWorldPersistenceSubsystem`): pickups lying
  around, the chest's and the body's contents, the crate.
- The demo starts every session with it **off**.
- **To turn it on,** step on the **WORLD MEMORY** pad in the FEATURES wing, or call
  `ASnapDemoGameMode::SetWorldMemoryEnabled(true)`.
  - It restores what was saved last time.
  - From then on it saves the world every three seconds into the demo's own world save (`SnapInventoryDemo_SnapWorld`).
- Off, nothing is restored and nothing is saved.
- The switch lasts for the play session only.

**To see it work:**

1. Turn it on.
2. Drop something near the bank.
3. Stop playing.
4. Start again. The item is gone, because world memory starts off.
5. Turn it on. You should see the item back.

- ⚠ **It is the WHOLE map, not a corner of it.** A restore puts every pickup and container back as it was saved, and
  removes world items the save does not list.
  - So turn it on first thing in a session. Anything picked up before that comes back to the ground as well as staying in
    the bag.
- ⚠ **A world saved before the map took its current layout puts things back where they lay in the old map**, some outside
  the hall's walls.
  - Delete `SnapInventoryDemo_SnapWorld.sav` from `Saved/SaveGames` first.

## Magic items

- **The example weapons, armor and jewelry each roll none, one or two magic properties** when they are made, from
  `DA_ExampleAffixTable`.
- They are the iron sword, the greatsword, the axe, the shield, the five iron and gilded pieces, the helmet, the amulet
  and both rings.

| Affix | Names the item | Rolls | Moves, while worn | Worth |
|---|---|---|---|---|
| Fury | Iron Sword **of Fury** | +2 to +5 Attack Power | Attack Power | x1.25 |
| Sturdy | **Sturdy** Iron Sword | +1 to +3 Armor | Armor | x1.15 |
| Vigor | Iron Sword **of Vigor** | +3 to +8% Max Health | Max Health | x1.15 |

- **A magic item says so in its name** ("Sturdy Iron Sword of Fury"), in its tooltip, the bag, the sell window and the
  collect feed.
- **The Iron Axe shows how one item tunes its roll:**
  - It is magic 60% of the time.
  - When it is, it asks for one property three times in four and two the rest, only Fury or Sturdy.
  - Fury has its own 50% chance (which can leave a two-property roll at one) and rolls +3 to +6 on the axe.
- **Pick up a few and hover them.** Each property is a line in the tooltip with the number that item rolled, and two items
  of one kind compare property by property.
- **Wear one and the property moves your stats:** Attack Power and Armor by the number it rolled, Max Health by that
  percent.
- A shop pays more for it (each property multiplies the price).
- The chest, the body and the crate drop them too: the chest and the body can give a Ring of Vitality, the crate a pair of
  iron gauntlets.
- **The roll is the server's and is never re-rolled.** Drop the item and pick it up again and it is the same item.
- Some come out with none, which is what makes the others read as magic.
- How it works: [Affixes](affixes.md#what-a-roll-moves).

## Item rules: an item you cannot wear yet, and a quest item

In **JEWELRY AND A QUEST**, the **ITEM RULES** sign stands over two items:

- **The Warrior's Signet** needs Attack Power 12 — see [An item you cannot wear yet](#an-item-you-cannot-wear-yet).
- **The Sealed Letter** (`ID_Example_SealedLetter`) is a **quest item**: its `Restrictions` fragment refuses every way of
  losing it.
  - **Try them:** Drop it on the floor, sell it to a shop, offer it in a trade, break it down at the forge. Each is
    refused, and a notice says why.
  - Die with it (`K`) and it stays with you while everything else goes into the body.
  - Its tooltip says what it is: *Quest item*.
  - Bring it to the Courier, beside it: a Ring of Vitality, and your second ring slot unlocks for good (see
    [Quest givers](quest-givers.md)).

A restriction is the designer's; a player cannot lift it. The marks a player sets (Lock, Favorite, Junk) are the other
half: see [Restrictions and marks](restrictions-and-marks.md).

## The money changer

- **The demo ships a money changer**, `BP_Example_MoneyChanger` in `Examples/World`, that the map does not place for now.
  - Drop one into a map of yours to try it.
- It CONVERTS your coins.

**To use it,** press `E` at its booth (the prompt reads **Exchange coins**). A small window opens with what you hold and
four trades, each once (**x1**) or ten times (**x10**):

- **10 Silver for 1 Gold**, and **1 Gold for 10 Silver**
- **10 Gold for 1 Gem**, and **1 Gem for 10 Gold**

**What happens:**

- **The rates are the wallet's own.** The demo's three currencies say what each coin is worth (`BaseUnitValue`):
  **1 gem = 10 gold = 100 silver**.
  - The window offers a trade between each coin and the next one up (`ASnapDemoMoneyChanger::BuildTrades`).
  - So no rate is written twice, and a wallet with other coins gets other trades.
- Each trade floats up above the booth (*+1 Gold*, *+10 Silver*) on every player's screen, and a line under your stats
  says what you paid and what you got.
- Short of what a trade takes, it is refused and nothing moves; the line says what you hold.
- **To close it,** press `Escape`, the window's **X** or a pad's `B`. Walking away from the booth, dying, or opening the
  bag or the menu closes it too.
- While it is open the prompt hides, and a click beside the window is not an attack.
- The other three shops keep one currency each.

### How the money changer works

**It is the demo's own server flow**; the plugin leaves currency math like this to a project.

- The window names a TRADE and a count, never an amount (`ASnapDemoPlayerController::Server_ExchangeAtMoneyChanger`).
- The server re-reads the trade from its own settings, checks you are alive and within reach, takes the coins and then
  gives the others.
- It puts the take back if the give does not land (`ASnapDemoMoneyChanger::ServerExchange`, built on
  `USnapWalletComponent::AddToCounterFor`).
- The floating text is `USnapDemoFloatingTextWidget`, the widget that raises the training dummies' numbers too.
- See [Currency and prices](currency-and-prices.md).

## Two players

The **TWO PLAYERS** sign, at the bottom of the tall signs at the start, gives the editor steps:

1. Open the Play menu (the button beside **Play**).
2. Set **Number of Players** to 2.
3. Set **Net Mode** to **Play As Listen Server**.
4. Press Play.
   - You should see two windows open, one per player.

Then, together:

- **Trade**: Look at the other player and press `E`; the prompt reads *Trade with <name>*. Both of you offer items and
  coins, both confirm, and the swap happens at once ([Trade](trade.md)).
- **Pick up each other's drops**: Anything one player drops, the other can walk over or loot.
- **Loot a body**: One player presses `K`; the other opens the body that falls and takes what is in it.
- **See each other**: Each player sees the other's armor, weapons and spells.

On one screen too: step on **SPLIT SCREEN** in the MOTION, SOUND, CONTROLLERS block -- in a Standalone game, or in the host's
window when two players play (the other window cannot add one). Player 2 joins on a controller, the screen splits, and each
player's screens stay in their own half. Step on it again to leave.

More:

- Each player keeps a bank and a save of their own.
- A packaged build has no join screen in the demo; a project brings its own (see [Networking](networking.md)).

## Signs that teach

- Short signs sit where the feature already is.
- **Every sign is one kind, `ASnapDemoSign` (`BP_Example_DemoSign`):** A title and rows, of key pictures + what they do,
  or of words alone.
- **A controller button is the player's own pad picture:** The plugin's glyph set, re-read twice a second, so it follows
  the GAMEPAD ICONS pad and the pad in your hands.
- **A keyboard or mouse key is one of the demo's own key caps** (`/Demo/SignKeys`, drawn through `M_ExampleSignKey`).
  - A row that names both draws a "/" between them.
- The two BAG TIPS signs (**MOUSE AND KEYBOARD** and **GAMEPAD**) read their rows from the demo's controls table
  (`SnapDemoControls`), the same rows the demo menu lists.
- Every sign in the map reads from both sides (its rows are drawn on the back of its plate too), except the three stall
  names on the awnings.
- Each section's sign has the color of its section's ground, and each FEATURES block's sign the color of its block's.

**Every sign that names a key or a click also names the controller button the demo binds to it:**

| Key or click | Controller button |
|---|---|
| `E` | `X` |
| `I` | `Y` |
| Left and right click | `RT` and `LT` |
| The mouse wheel | `View` |
| `1`-`0` | The D-pad (the first four) |

**The signs, by place:**

- **FOOD SPOILS**, over the meat, the bread and the mush in **FOOD**:
  - Meat goes bad as time passes (it turns into Rotten Mush, which the Junk Merchant buys for 1 silver).
  - Food inside the Cooler (a storage item) spoils at a quarter of the speed.
  - Double-click food in the bag to eat it: the meat heals at once, the bread over ten seconds.
  - See [Condition and weight](condition-and-weight.md).
- **GEAR WEARS**, over the repair kit:
  - The armor you wear takes wear when you are hit.
  - The General Store repairs it for gold.
  - A Repair Kit restores 50 durability anywhere: drag it onto the damaged item in your bag.
- **BAG TIPS: MOUSE AND KEYBOARD** and **BAG TIPS: GAMEPAD**, side by side under the **SNAP INVENTORY SYSTEM** sign at the
  start, over **TWO PLAYERS**:
  - The bag screen's keys and buttons, row by row, from the demo's controls table — the same rows the menu's Controls
    list shows.
- **MIXING**, on top of the **POTIONS** sign, over the potions and the two arcane shards:
  - Drop one item onto another, either order, in the bag or a pouch — see [Mixing potions](#mixing-potions).
- **TRADER**, over the Trader at the end of the shops; **COURIER**, over the Courier in **JEWELRY AND A QUEST**; **SCATTER
  CHEST**, over the chest that throws its loot.
  - Each says in two lines what it does and which key does it.
- The welcome sign also says how to sprint: hold `Left Shift`, or click the left stick (L3) on a controller while you
  move.

## The FEATURES wing: switch a plugin feature for this session

**Along the left of the walkway stand the FEATURES pads:** Forty pads in one line, in seven blocks.

- Each block is across from the sections it touches.
- A sign over each block says what the block is about. The block stands on ground of that sign's color.
- A sign over each pad names its feature and shows its value **now**, on both sides of its plate, so a pad reads from
  behind too.
- Behind each pad, under its sign, a **picture** shows what the feature looks like switched on, so you can see what a pad
  changes before you step on it (see [The pictures](#the-pictures)).

**Step on a pad and the feature switches to its next value:** ON and OFF, or the next of its choices.

- The pad's top turns green for ON and red for OFF (teal for a pad with choices, and for **SPAWN EVERY ITEM**).
- A line under your stats names the change for four seconds, on every player's screen.
- A pad whose feature the bag's screens draw has them built again so they show it: **CARRY WEIGHT**, **DURABILITY**,
  **FOOD SPOILS**, **ITEM MARKS**, **THEME**, **FLOATING WINDOWS**, **STACK PLATES**, **TIMED CRAFTING**, **ONE PANEL**
  and **LOCKED INVENTORY SLOTS**. **THEME** and **STACK PLATES** build the hotbars on the HUD again too.

**The rules every pad follows:**

- **Once per arrival.** Standing on a pad does nothing more; step off and on again to switch it again.
- **For this play session only.**
  - Every pad starts on the demo's own value, whatever your project has set, and when the session ends your project's
    values come back.
  - No setting is written to your project; **SAVE YOUR BAG** and **WORLD MEMORY** write only the demo's own save files.
- **The sign reads the live value.** It is read again twice a second, so a value changed some other way (another pad, the
  console, your own code) shows on the sign too.
- **A switch that would strand an item says no:**
  - **NESTED POUCHES** does not go off while a pouch holds another storage item, in a bag or on the ground (a pouch inside
    a pouch could not be put back).
  - **TIMED CRAFTING** does not go off while a craft is queued (it would stop where it is).
  - **BAG LAYOUT** does not change while anything a player carries would have nowhere to go in the other layout (every
    player's bag is asked first).
  - The line under your stats says what to do first; nothing changes.
- **In a two-player session the server's value is the one that counts**, and each player's sign shows it.
  - The world memory, the demo's route, **SPAWN EVERY ITEM** and **SPLIT SCREEN** are the server's alone (**SPLIT SCREEN** adds a
    player to the host's screen only).

### The INVENTORY RULES block

- **CARRY WEIGHT:** Carrying weight (`bEnableCarryWeight`); the slowdown has its own switch, below.
  - **Look for:** Off, the bag's weight readout goes and nothing is ever over.
- **OVERWEIGHT SLOWDOWN:** The slowdown alone (`bEnableOverweightSlowdown`).
  - The demo slows you the further over 75 kg you are (`OverweightSlowdownMode` Proportional: a quarter of the walk speed
    at 112.5 kg).
  - **Look for:** Off, an overloaded character walks at full speed.
- **DURABILITY:** Durability, and the wear a hit puts on what you wear (`bEnableDurability`, `bEnableDamageDurabilityWear`).
  - **Look for:** Off, the red pad's hits no longer wear your armor.
- **FOOD SPOILS:** Spoilage (`bEnableSpoilage`).
  - **Look for:** Off, the meat in **FOOD** stops going bad.
- **NESTED POUCHES:** A storage item inside another (`bAllowNestedStorageItems`).
  - **Look for:** Off, a pouch no longer goes into a pouch.
- **ITEM MARKS:** Lock, Favorite and Junk (`bEnableItemMarks`).
  - **Off when the demo starts.**
  - **Look for:** On, right-click an item: Lock, Favorite and Junk are in its menu. Off again, the marks you set stop
    counting, and come back when it is on.
- **TURN ITEMS:** Items may be turned in a grid (`bAllowItemRotation`): `R` while dragging or over an item, the gamepad's
  turn button, and a placement that turns an item to make it fit.
  - On when the demo starts.
  - **Look for:** Off, with the bag on GRID (**BAG LAYOUT**), `R` no longer turns an item and nothing is turned to make it
    fit.

### The LOOK AND LAYOUT block

- **THEME:** The UI theme: NEUTRAL, STEEL, CLASSIC (`UITheme`).
  - **Look for:** The bag, its windows and the hotbar on the HUD take the theme's colors.
- **FLOATING WINDOWS:** Windows you can drag and close (`bAllowWindowDragAndClose`).
  - Off when the demo starts.
  - **Look for:** On, open the bag: its parts are windows with a close box.
- **ONE PANEL:** The bag screen as one window (`bOnePanelInventoryScreen`).
  - The paper doll and the equipment, the bag with its currency line under it and Spells under that, centered against the
    equipment, share one frame, one title and one close box.
  - Off when the demo starts.
  - **Look for:** On, open the bag: one frame titled INVENTORY round everything, the money and its buttons along the
    bottom. With FLOATING WINDOWS on too, drag it by its title and the whole thing moves.
- **STACK PLATES:** A plate behind a stack's count (`bShowStackCountBackground`).
  - Off when the demo starts.
  - **Look for:** On, a potion stack's number sits on a plate.
- **COMPARE TOOLTIPS:** Comparing an item with what you wear (`bCompareToEquipped`).
  - **Look for:** Off, a sword's tooltip no longer shows how it compares with the one you wear.
- **BAG LAYOUT:** SQUARES, GRID: one item per square, or a grid where each item covers its real shape.
  - It switches each player's bag, through `USnapInventoryComponent::SetLayoutMode`; the paper-doll screen's bag area
    follows it.
  - **Look for:** GRID, open the bag: the iron sword covers three cells and the gold bar two, and an axe in the pouch is
    turned to fit the pouch's own grid.
  - More in [SAVE YOUR BAG and BAG LAYOUT, in detail](#save-your-bag-and-bag-layout-in-detail).
- **SCREEN BACKDROP:** What is drawn behind the bag, vendor and trade screens and a crafting station's window opened on its
  own: CLEAR, DIM, BLUR (`ScreenBackdropMode`).
  - CLEAR when the demo starts.
  - **Look for:** DIM or BLUR, open the bag or a shop: the world behind it darkens or blurs. A chest's loot window or a
    pouch window opened on its own is not a screen and gets neither.
- **LOCKED INVENTORY SLOTS:** The room the bag upgrades could still add, as locked squares at the bag's end
  (`bShowUnboughtBagSquares`, with the three bag upgrades listed).
  - On when the demo starts. Its sign says ON (SHOW POTENTIAL UNLOCKS) or OFF (NO LOCKED SLOTS SHOWING).
  - **Look for:** Open the bag and scroll down: 18 squares with a lock, which take no click, hover or gamepad stop. Use a
    bag upgrade: its squares open. Off, the bag ends at the squares it has. On a pad, moving down to the last open row
    brings the locked rows into view.

### The PICKUPS AND LOOT block

- **AUTO PICKUP:** Walking into a pickup to take it (`bAllowOverlapPickup`).
  - Its sign says ON (WALK OVER TO TAKE) or OFF (INTERACT TO TAKE).
  - **Look for:** Off, walking over an item takes nothing: look at it and press `E`.
- **AUTO-EQUIP:** Wearing what you pick up when its slot is free (`bAutoEquipOnPickup`).
  - **Look for:** Off, a sword or a piece of armor you pick up goes into the bag.
- **RARITY SWAP:** A rarer copy of the item you wear swapping in for it (`bSwapToBetterRarityOnPickup`; another item never
  swaps).
  - **Look for:** Off, walking onto a rarer helmet under the **HELMETS** sign leaves it in the bag.
- **RARITY GLOW:** A soft light around every item lying in the world, in its rarity tier's color (`bShowPickupRarityGlow`).
  - On when the demo starts.
  - **Look for:** Off, the pickups on the floor lose their colored glow.
- **PICKUP FEED:** The rows naming what you just took (`bEnableCollectFeed`).
  - **Look for:** Off, no row appears when you pick something up.
- **FEED ON TOP:** The feed in the screen's top-right corner, newest line on top (the feed's `bNewestOnTop`).
  - Off when the demo starts.
  - **Look for:** On, pick up a few items: the lines stack downward from the top right, the newest first. Off, back to the
    bottom left.
- **OUTLINE:** The outline around what you look at (`bInteractionOutlineEnabled`).
  - **Look for:** Off, the chest and the stalls are not outlined.
- **KEY IN PROMPTS:** The key in a prompt (`bShowInteractKeyInPrompt`).
  - **Look for:** Off, a prompt drops its key: `(E) Open your bank` reads `Open your bank`.

### The CRAFTING, DEATH, SAVING block

- **TIMED CRAFTING:** Crafts that take time (`bEnableTimedCrafting`).
  - **Look for:** Off, a craft is done at once.
- **ON DEATH:** What a death costs: DROP ALL, KEEP WORN (the death-drop policy).
  - **Look for:** KEEP WORN, press `K`: what you wear stays on you. The bank is kept either way.
- **RAGDOLL BODY:** What a death leaves (`bRagdollLootableBodies`): the character itself, lying limp where it fell and still
  wearing its gear.
  - Each worn piece disappears from it as you take it, and it sinks into the ground once emptied and you walk away.
  - On when the demo starts.
  - **Look for:** Off, a death leaves the plugin's body marker. On, press `K` while wearing armor and loot your own body:
    the helmet vanishes from the head as you take it.
- **SAVE YOUR BAG:** Saving your whole inventory as you play (`bEnablePersistenceAutoSave`).
  - **Look for:** See [SAVE YOUR BAG and BAG LAYOUT, in detail](#save-your-bag-and-bag-layout-in-detail), below.
- **WORLD MEMORY:** Saving the world itself — the same switch as [A world that remembers](#a-world-that-remembers).
  - **Look for:** On, it restores what was saved last time.

### The MORE TO TRY block

- **DOUBLE-CLICK USE:** Double-clicking a consumable to use it (`bConsumeOnDoubleClick`).
  - **Look for:** Off, double-clicking a potion no longer drinks it.
- **AUTO EQUIP POTIONS:** A potion you pick up landing on a potion square, `Q` or `R` (`bAutoBindConsumablesToHotbar`).
  - Not **AUTO-EQUIP**, which puts on gear; the line it shows says so.
  - **Look for:** Off, a potion you pick up is not put on `Q` or `R`. On, only a potion that starts a new stack is put
    there; one that joins a stack in the bag is not.
- **SELECTABLE HOTBAR:** A 1-0 key (a D-pad button) takes its square's item in hand, and left click (RT) uses it
  (`bSelectableHotbar`).
  - The square rises and the worn weapon is put away.
  - Left click uses it: a weapon swings, a potion is drunk, a scroll is cast.
  - The key again or Esc (Start) puts it away.
  - Off when the demo starts.
  - A weapon you wear goes on a square too (drag it from the equipment column) and stays worn: its key swaps back to it,
    and the key of the weapon already in your hand only raises its square.
  - **Look for:** Off, every 1-0 key uses its square at once and no square rises.
- **DEMO RUNS ON:** GAMEPLAY ABILITIES, BUILT-IN (NO GAS): which route the whole demo runs on
  ([The demo's two routes](#the-demos-two-routes)).
  - Stats, damage, potions, buffs, cooldowns, spells, passives and the attacks switch together.
  - A flip ends every buff and cooldown, re-reads the stats from what you wear and caps Health.
  - BUILT-IN ONLY without GameplayAbilities: the pad says why and changes nothing.
  - **Look for:** A dummy takes the same hits on either; the log names the route of each.
- **SPAWN EVERY ITEM:** Lays out one pickup of every example item behind the pads.
  - **Look for:** Step on it again: the last batch still on the ground is cleared first.

### The MOTION, SOUND, CONTROLLERS block

- **ITEM MOTION:** Item squares that move: the picture grows under the cursor or a pad's focus, and bounces when an item
  lands (`bReduceItemMotion`, the other way round).
  - On when the demo starts.
  - **Look for:** Off, hovering a square and moving an item leave every picture still.
- **SOUNDS:** The plugin's sounds (`bPlaySounds`): items, pouches, chests, bodies, actions, buttons and windows make the
  plugin's own example sounds.
  - On when the demo starts.
  - **Look for:** Off, drag an item or open the bag: silence. On again: the sounds come back.
- **GAMEPAD ICONS:** Which gamepad's pictures a pad button is drawn with: AUTO (the pad you used last), XBOX, PLAYSTATION
  (`GamepadGlyphFamily`).
  - AUTO when the demo starts.
  - **Look for:** With a pad in your hand (or `GlyphDevicePreference` ForceGamepad), XBOX draws the interact prompt's
    button as a blue X and PLAYSTATION as a square. The D-pad pictures on the hotbar are the same in both.
- **SPLIT SCREEN:** A second player on this screen, through the engine's local players.
  - On, player 2 joins in front of the pad and plays on the controller; player 1 keeps the mouse and keyboard. Off, player 2
    leaves, and what they carried drops where they stood.
  - Off when the demo starts. In a Standalone game or on the host's screen of a listen server; a client's player, or a
    dedicated server (Play As Client), gets a line saying why.
  - With no controller connected, the pad says so and adds nobody.
  - **Look for:** Each player's bag, shops, windows, HUD and menu stay in that player's half, and each controller moves only
    its own player's screens. The mouse keeps turning player 1 while player 2's screens are open.

### The SHOPS block

- **VENDOR RESTOCK:** How every shop's limited items come back: PER ITEM (one unit every 20 s), WHEN SOLD OUT (back in
  30 s), WHOLE SHOP (every 60 s).
  - It is the plugin's `bOverrideVendorRestock` and `VendorRestockOverride`.
  - PER ITEM when the demo starts.
  - **Look for:** Buy the last Iron Shield from the Gem Trader, close the shop and wait: on WHEN SOLD OUT it is back after
    30 seconds, not before.
  - Each stall's restock line (`ASnapDemoVendorRestockSign`) and the vendor window's line under the shop's name say the
    rule now.
- **SELL CONFIRMATION:** Asking before a sale (`bConfirmVendorSales`).
  - Off when the demo starts.
  - **Look for:** On, at a shop the item menu's **Sell**, a double-click and a Shift+click on an item open the sell window
    instead of selling at once.
- **BUYBACK:** Buying back what you sold (`VendorBuybackSlots`: 8 when on, 0 off).
  - Off when the demo starts.
  - **Look for:** On, sell an item at a shop: its list offers it back (the very item, its wear and its magic) until it is
    bought or pushed out.

### SAVE YOUR BAG and BAG LAYOUT, in detail

**SAVE YOUR BAG.** On, the demo brings back what it saved last time **on top of** what you hold, and then saves your
inventory as you play, into the demo's own save (`SnapInventoryDemo_<player>`).

- Nothing you hold is replaced, and an item already in your bag is not brought back twice.
- Inside a saved pouch you still carry, what is still there counts as already here, and what you used up since goes back
  into it.
- Off, it stops saving.

To see it:

1. Switch it on.
2. Pick a few things up.
3. Stop playing.
4. Start again and step on it. You should see them back.

- **If a saved item cannot come back** (no room for it, or the pouch it was saved in is not in your bag), the switch goes
  back off and the line says how many.
  - Saving then would write a save without them.
  - Make room, put the pouch back in your bag, and step on it again.
- With nothing missing, the line says everything from your last save is already here.

**BAG LAYOUT.** GRID places everything each player carries where its shape fits, and SQUARES puts them back one per square
in their reading order.

- GRID goes in the order the items stood, a sword or a gold bar turned when it has to be, and the inside of a pouch with
  a grid of its own too.
- **Nothing is ever dropped.** A change that would leave something with nowhere to go is refused and nothing moves (a
  pouch whose contents would not fit its own grid keeps it on SQUARES too: the Leather Pouch's grid is 3 x 2, six
  cells, so an Iron Greatsword in it (1 x 4) fits neither upright nor turned. An Iron Sword and an Iron Axe in it
  (1 x 3) fit turned, so they keep it on SQUARES only while TURN ITEMS is off).
- The paper doll, the Spells grid and the bank stay where they are: only the bag area changes, from squares to the grid,
  live, with the bag open.
- **The grid is as wide as the squares bag, 6 cells, and 14 rows deep.**
  - Every example item the bag can take fits with rows to spare.
  - It shows 5 rows at a time; it scrolls.
- Each item fills its cells at its own shape: a sword upright in its three, the gold bar lying across two. The README's
  grid section lists every example item's size.
- A bag saved on GRID by **SAVE YOUR BAG** comes back into a SQUARES bag in a later session with every item unturned.
  - One whose cell number is past the last square goes to the first free square, and the log says which.

### The pictures

**Every pad has a board just behind it**, under its sign, with a picture of its feature switched **on**.

- The picture is on the board's front and again on its back, the right way round from either side.
- **A pad with choices shows one that is not where the demo starts:**
  - **THEME** shows STEEL.
  - **BAG LAYOUT** the grid.
  - **SCREEN BACKDROP** the blur.
  - **ON DEATH** KEEP WORN.
  - **GAMEPAD ICONS** the PlayStation buttons.
  - **VENDOR RESTOCK** a shop that restocks every 60 s.
  - **DEMO RUNS ON** the built-in route.
  - **FEED ON TOP** the feed at the top right.
  - **SELL CONFIRMATION** the window that asks before a sale.
  - **BUYBACK** a sold item back in the shop's list.
- **A feature you cannot see shows the part of it you can:**
  - **SAVE YOUR BAG** a bag brought back from its last save, with the line that says so.
  - **WORLD MEMORY** dropped items on the ground.
  - **SOUNDS** (heard, not seen) the plugin's own Sound settings with Play Sounds ticked.
- **LOCKED INVENTORY SLOTS** shows the bag scrolled to its end: its last open row over rows of locked squares.
- **SPLIT SCREEN** shows the screen split in two, each player's bag open in their half.
- Each picture was taken in this demo, of the plugin's own screens and of this map.

**The picture is the pad's own:** `ASnapDemoFeaturePad::Picture` (a texture, `T_FeaturePicture_<Feature>` in
`Demo/FeaturePictures`).

- It is drawn through `PictureMaterial` (the unlit `M_ExampleSignPicture`, texture parameter `Picture`) on `PictureMesh`
  (the engine's `/Engine/BasicShapes/Plane`).
- It is 200 by 100 cm, on a board made of the sign plate's own mesh and material.
- A pad of yours with no picture (or no material or mesh) shows no board at all.
  - Give it a 2:1 texture of your own and the same material and mesh, and it shows one.
- The material instance it draws through is made when the pad is built or begins play, and is never saved with the map.
- **In play, the picture is read through the art pack's icon map** (`USnapArtPackSubsystem::ResolveIcon`), on the server
  and on every client.
  - It is drawn again when a pack is registered or unregistered (`OnArtPacksChanged`; a client's pads begin play before
    its game state registers the pack).
  - With a pack that names a picture, its replacement shows.
  - With none, or with `Snap.Demo.ArtPack 0`, the pad's own shows.
- The editor's own level always shows the pad's own picture.

**The demo's own art pack names a replacement for all forty pictures:** The same pictures taken again with the pack
on, so they show the demo's models and icons.

- They are `T_DemoArt_FeaturePicture_<Feature>` in `DemoArt/FeaturePictures`: captures of the demo showing the pack's
  models.
- With the pack on (the default), the boards show those.
- With it off, or in a build that does not cook `DemoArt`, the boards show the pad's own pictures of the example art,
  which match the example models shown then.
- The pads still name only their own pictures.

### How the FEATURES pads are built

**The pad is `ASnapDemoFeaturePad`** (`Demo/SnapDemoFeatures.h`), a piece of example code you can read and copy.

- **Its sign** is as wide as its wider line, but never wider than 250 cm (the pads stand 320 cm apart).
  - So a line too long for that shrinks its letters to fit.
  - Its two lines are written again on the back of the plate (`BackTitleText`, `BackStateText`).
- The picture board behind it is `PictureBoard`, `PictureFront` and `PictureBack`.
- **Every switch is a session value**, set through `USnapDemoSessionSubsystem::OverrideForSession` and put back when the
  session ends.
  - After a change, a pad whose `RebuildsScreens` is true calls `USnapInventoryHUDComponent::DiscardCachedScreens` for each
    local player on every machine, and one whose `RebuildsHUDBars` is true (**THEME**, **STACK PLATES**) also calls
    `USnapInventoryHUDComponent::RebuildHUDHotbars`, so the things built only once are built again.
- **On a network the server flips the switch** (it owns the overlap) and replicates the choice it reads
  (`ServerChoice`).
  - Each client applies the same session value to its own copy of the settings when that arrives. The UI switches
    (the theme, windows, plates, marks, tooltips, the prompt) are read on the client.
  - Switches only a server reads (walk-over, the death policy, saving) are applied on clients too, harmlessly, so a
    client's sign reads its own value.
  - The server-only actions (world memory, the demo's route, the spawn) show the server's reading.
- **Stepping on a pad flips it once per arrival.** Standing on it does not flip it again.
- **SPLIT SCREEN adds and removes a local player** (`UGameplayStatics::CreatePlayer` / `RemovePlayer`) on the tick after the
  step, never inside the overlap.
  - Windows gives player 1 the keyboard, the mouse and the first controller, so the join hands that controller to player 2
    (`IPlatformInputDeviceMapper::Internal_ChangeInputDeviceUserMapping`); it goes back when player 2's controller ends play.
  - The leave ends player 2's trade and drops what they carry and wear (`DropEverythingToWorld`, with the demo's drop-all
    death policy) before removing them.
  - The demo's HUD and menu go in each player's part through the plugin's own screen placement, which is private to the
    plugin's module: a controller copied into your own module can use `UUserWidget::AddToPlayerScreen` instead.
- Each pad logs every player's item count before and after its change.
- **BAG LAYOUT calls the plugin's `USnapInventoryComponent::SetLayoutMode` on each player's bag**, and nothing else (see
  [Changing the layout while a container holds things](inventory-and-containers.md#changing-the-layout-while-a-container-holds-things)).
  - The screen's bag area follows the layout by itself
    ([A screen that draws both layouts](inventory-and-containers.md#a-screen-that-draws-both-layouts)).
- **The four plugin switches it added are ordinary project settings, on by default:** `bEnableItemMarks`,
  `bAllowOverlapPickup`, `bAutoEquipOnPickup` and `bSwapToBetterRarityOnPickup`.
  - See [Restrictions and marks](restrictions-and-marks.md#switching-marks-off),
    [Pickup modes](pickup-modes-and-the-collect-feed.md#switching-walk-over-off-for-the-whole-project) and
    [Equipment](equipment.md#what-a-pickup-equips-by-itself).

## Filling a map with items

- `USnapTestPickupSpawner` spawns a ring of every item definition in a folder around you, on a key.
- It is a development aid and ships off.

**To use it:**

1. Tick **Project Settings → Plugins → Snap Inventory System - World → Examples → Enable Example Item Spawner**.
2. Press `L` in gameplay.
   - You should see a ring appear around wherever you are standing.

- Pressing `L` again clears the previous ring first.
- **Untick it before you package** — it is a cheat key in any build a player gets.
- A Shipping build never creates the spawner, whatever the tickbox says; a Development or Test build does.
- Its other values (the key, the folder, how many copies) are listed in the plugin's `Config/Game.ini`. Set them in your
  own `DefaultGame.ini` under `[/Script/SnapInventorySystem.SnapTestPickupSpawner]` (see
  [The test spawner](loot-and-interaction.md#the-test-spawner)).

## What the demo map is not

**It is a fixture, not a showcase.**

- Its floor and walls are engine cubes from `/Engine/BasicShapes`, finished with the plugin's own grid floor and stone
  materials.
- It has no visual effects (its sounds are the plugin's own example sounds, made by a script).
- Everything standing on the floor (the items, the shops, the forge, the chest, the crate and the body) is one of the
  plugin's own low-poly models.
- The character is Epic's mannequin, shipped as demo content.
- **That is deliberate:** The map must open in a clean engine install with nothing but this plugin enabled, so everything
  it shows ships inside the plugin.

**It is deliberately not a World Partition map**, and you should not convert it to one if you intend to set it as your
editor's startup map.

- A partitioned startup map deadlocks the editor on its first tick, because the map load begins while the asset registry's
  initial scan is still running.

**It does not save your bag unless you ask it to.**

- The `bEnablePersistenceAutoSave` setting ships off, so every launch starts with an empty one.
- The **SAVE YOUR BAG** pad turns saving on for a session (see
  [The FEATURES wing](#the-features-wing-switch-a-plugin-feature-for-this-session)).
- **The hotbar is saved, but a binding comes back only for an item you are holding when the profile loads**, about twenty
  seconds after the start.
  - The demo holds nothing then, so after a relaunch the hotbar is empty, and it stays empty when you pick the item up
    again.
- See [Persistence](persistence.md) to turn saving on.

## See also

- [QUICKSTART](../QUICKSTART.md) — installing the plugin into a project you already have
- [Equipment](equipment.md) — slots, granted abilities and effects, equipment visuals
- [Loot and interaction](loot-and-interaction.md) — pickups, containers, bodies, the interact key
- [Appendix: settings](appendix-settings.md) — every setting, generated from the headers
