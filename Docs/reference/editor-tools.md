# Editor tools

The plugin's editor module, `SnapInventorySystemEditor`, adds editor-only helpers. This page covers each one, how to
register item definitions with the Asset Manager, and how to remove the module.

**What it adds:**

- Two conveniences to the Content Browser:
  - A **Snap Inventory** group in the New menu.
  - Item thumbnails that show each item's icon.
- A **footprint shape editor** in the Details panel.
- A **setup check** that reports what a project has not set up yet.

**It is editor-only:**

- It is declared `Editor` in the `.uplugin`, so it never reaches a packaged game.
- Nothing else in the plugin depends on it.

**The console commands are not editor tools.** The two development console commands, `Snap.GiveItem` and
`Snap.DumpInventory`, live in the runtime module so they also work in a Development build. See
[Testing and diagnostics](testing-and-diagnostics.md#console-commands).

**On this page:**

- [Creating assets](#creating-assets)
- [Item thumbnails](#item-thumbnails)
- [Painting a footprint](#painting-a-footprint)
- [The setup check](#the-setup-check)
- [Registering item definitions with the Asset Manager](#registering-item-definitions-with-the-asset-manager)
- [Removing the editor module](#removing-the-editor-module)

## Creating assets

1. **Right-click in the Content Browser** and open **Snap Inventory**.
   - It lists the thirteen asset types the plugin asks you to author (table below).
2. **Pick an entry.** Each entry creates exactly that class and opens it.

| Entry | Class | What it is |
|---|---|---|
| Affix Table | `USnapItemAffixTable` | The weighted affixes an item can roll when it is created, and the range each one rolls in |
| Container Registry | `USnapContainerRegistry` | Which inventory containers are added to which actor classes automatically |
| Equipment Slot Set | `USnapEquipmentSlotSet` | Which equipment slots this project has, and each slot's name, ghost, attach socket and wear rate |
| Item Definition | `USnapItemDefinition` | One kind of item and the fragments that decide how it behaves |
| Item Set | `USnapItemSet` | A named set of items and what wearing more of its pieces grants, tier by tier |
| Loot Table | `USnapLootTable` | What a chest, crate or corpse may contain |
| Quest | `USnapQuestDefinition` | What a quest giver takes in, gives back and unlocks ([Quest givers](quest-givers.md)) |
| Rarity Table | `USnapItemRarityTable` | The rarity tiers items can roll |
| Recipe Set | `USnapCraftingRecipeSet` | A group of crafting recipes |
| Stat Display Table | `USnapStatDisplayTable` | What a tooltip says about the effects an item grants |
| Stat Effect Map | `USnapStatEffectMap` | Which GameplayEffect moves each stat an item declares in its Stat Contributions rows |
| Theme | `USnapInventoryTheme` | The colors and typography the inventory screens paint with |
| Vendor Stock | `USnapVendorStockDataAsset` | What a shop sells and buys, and how it restocks |

The same thirteen types show a color strip in the Content Browser, one color per type.

⚠ A loot table or an affix table can be a Blueprint subclass, for a project that replaces how it rolls.

- Create one of those with the engine's **Blueprint Class** entry, and pick `SnapLootTable` or `SnapItemAffixTable`
  as the parent.
- The Snap Inventory entry makes a plain asset of the table.

## Item thumbnails

An item definition's Content Browser thumbnail is its icon.

- It is the texture in its **UI Data** fragment, the one every slot widget shows.
- The icon is drawn the way the texture's own thumbnail is drawn.

**Good to know:**

- An item with no UI Data fragment, no icon set, or an icon that does not load shows the engine's default thumbnail
  instead.
- The thumbnail redraws when you edit the item, including when you change its icon. It does not redraw every frame.
- Only item definitions get this. The other twelve types show the engine's default thumbnail.

## Painting a footprint

A step-by-step walk-through, from a new definition to the demo's grid bag, is [Make a grid item](make-a-grid-item.md).

An item definition's **Footprint** fragment (see [Items and fragments](items-and-fragments.md)) is edited as a picture
rather than as a list of ticks. Under the fragment's `Width`, `Height` and `bCanRotate` rows:

- **Shape:** One square per cell of the box.
  - Click a square to cover or uncover that cell.
  - The item's icon, from the same definition's **UI Data** fragment, is drawn behind the squares, unrotated — the way the
    game draws it.
  - An empty top-left square is tinted orange: that is the cell the item is stored by.
- **Outlines:** Beside the grid, one small outline per turn the item may take.
  - The shape as authored, and its sideways turn too when `bCanRotate` is on.
  - A shape a turn cannot change, such as a full square, shows one.
- **Size presets:** One button per common size — 1 x 1, 1 x 2, 2 x 1, 2 x 2, 1 x 3, 2 x 3, 3 x 2 (width x height).
  - It makes the item that size and a full rectangle.
- **Reset to rectangle:** Empties the mask, so the item covers its whole box.
- **Validity line:** Exactly what asset validation will report for this shape, updated as you paint.

**Editing notes:**

- Every click, the reset and each size preset is one step in the undo history.
- Changing `Width` or `Height` — typed or dragged — keeps each painted cell where it was.
  - One Undo restores the size and the shape together.
- A mask that covers the whole box is stored empty. That is what a rectangle is, so nothing changes in play.
- With several item definitions selected, the shape is not painted. A line says so, and the other rows still edit them
  all.
- The grid is drawn for boxes up to 16 x 16. A larger box keeps the raw `CellMask` row instead, with a line saying why.
- The icon is loaded when the panel is built. If it does not change after you edit **UI Data**, select the item again.

## The setup check

**Tools → Snap Inventory Setup Check** reports what this project has not set up yet.

- Every mistake it finds is silent in the editor: the project looks fine.
- The consequence shows up at runtime, or only in a packaged build.

**This is the whole list.** There are sixteen kinds, one per enumerator of `ESnapSetupCheckCode`, and every one of them
is below.

- `RunSetupCheck` returns every finding it raised.
- `USnapSetupCheckLibrary::GetSeverityForCode` is the one place that decides which kind is a warning.

Every finding names its **subject** — the setting, row or asset it is about.

- So the list below says what a kind means and what to do about it.
- The finding itself says which of your values it found.

**The findings:**

- **A key that names nothing** (`ESnapSetupCheckCode::MalformedKey`)
  - **What it means:** An `FKey` in config must be written **bare** — `ToggleKey=I`, never `ToggleKey=(KeyName="I")`.
    An `FKey` is saved as just its key name, so the struct form is not refused: it imports a key named after the opening
    bracket, and the binding never fires.
  - **What to do:** Rewrite the value bare. The finding names the setting and the key name it really imported.
- **A gesture that can never fire** (`ESnapSetupCheckCode::GestureBindingIneffective`)
  - **What it means:** A row in **Gesture Bindings** that cannot do what it says: no keys listed, a field filled in on a
    side that does not read it, or a modifier requirement nothing consults. The gesture is simply unreachable.
  - **What to do:** Fix the row the finding names. Its words come from the same runtime collector the gesture itself
    uses, so they name the exact field.
- **A gameplay tag that is not registered** (`ESnapSetupCheckCode::UnresolvableTag`)
  - **What it means:** A tag name that was never declared is imported as an **empty** tag rather than refused. An empty
    tag is legal — an empty container tag means "not routed" — so once the value is imported there is nothing left to
    tell a typo from a deliberate blank.
  - **What to do:** Declare the tag, or correct the spelling. A blank value is never reported: that is how a setting says
    "unset".
- **A plugin folder packaging would drop** (`ESnapSetupCheckCode::CookFolderNotCooked`) — **warning**
  - **What it means:** The cooker takes your maps and what they reference. A folder that only a class default points
    into is not reached, so a mapping context or a material named only by a setting is missing from the package. A
    folder that only the demo points into is a note instead (below).
  - **What to do:** Add the `+DirectoriesToAlwaysCook=` line the finding prints, or reference the asset from something
    that is already cooked.
- **A plugin folder something forbids cooking** (`ESnapSetupCheckCode::CookFolderNeverCooked`) — **warning**
  - **What it means:** A `DirectoriesToNeverCook` entry covers a folder a setting points into. Never-cook **beats**
    always-cook, so the folder is dropped even when it is named in both.
  - **What to do:** Remove the never-cook entry, or point those settings at content you do cook.
- **A demo folder nothing cooks** (`ESnapSetupCheckCode::CookFolderOnlyDemo`)
  - **What it means:** The same gap, in a folder only the demo points into: defaults of the demo's own classes (their
    headers are in the module's `Demo` folders), or the example item spawner's folder while **Enable Example Item
    Spawner** is off. A project that does not ship the demo leaves that folder out on purpose, so it is a note.
  - **What to do:** Nothing, if you do not ship the demo. To ship it, add the `+DirectoriesToAlwaysCook=` line the note
    prints. For `/SnapInventorySystem/Demo` the note also prints the demo map's `+MapsToCook=` line (**List of maps to
    include in a packaged build**), because a cook folder line brings in the folder's assets, never a map in it; it leaves
    that line out once your map list names the map. Any other setting pointing into the same folder, at an asset nothing
    outside the folder references, makes it the warning above again, naming every pointer.
- **A setting on the demo's content** (`ESnapSetupCheckCode::ExampleContentDefault`)
  - **What it means:** A knob points at the plugin's example or demo content (`/SnapInventorySystem/Examples` or
    `/SnapInventorySystem/Demo`), which a packaged build has only while that folder is cooked. The plugin's settings do
    not point there by default; the demo's own classes and the example item spawner do, so a note for one of those is
    expected.
  - **What to do:** Nothing, if you meant it. A project with content of its own usually repoints the setting, and can
    then leave the Examples folder out of its build.
- **A receiver class that does not resolve** (`ESnapSetupCheckCode::UnresolvedReceiverClass`)
  - **What it means:** A configured class path names nothing that can be loaded, so the class you named is never used.
  - **What to do:** Repoint the setting, or fix the path — a renamed or moved Blueprint is the usual cause.
- **A component class that cannot be loaded** (`ESnapSetupCheckCode::UnloadableComponentClass`)
  - **What it means:** An attachment row — the container registry's, or **Snap Inventory System - Foundation → Attachment → Component Attach Specs** when
    `ContainerRegistry` is cleared or fails to load — names a component class that fails to load. So that row attaches nothing and the
    container it was for never exists.
  - **Not reported:** A row naming one of the plugin's `GameplayAbilities`-only classes — today the Spells container,
    `BP_Example_SpellsContainer` — is **not** reported in a project built without `GameplayAbilities`. It attaches
    nothing there by design, so the whole row is skipped.
  - **What to do:** Fix the row's component class, or delete the row if the container is no longer wanted.
- **No Asset Manager entry for item definitions** (`ESnapSetupCheckCode::AssetManagerTypeMissing`)
  - **What it means:** Nothing scans item definitions, so looking an item up by primary asset id finds nothing. The
    plugin deliberately ships no entry — two declarations of one type is itself an error.
  - **What to do:** Add one **only** if your own code looks items up by id, or you want the Asset Manager to decide which
    items are cooked. Otherwise leave it.
- **Two item definitions with one name** (`ESnapSetupCheckCode::DuplicateItemAssetName`)
  - **What it means:** A primary asset id is the asset's **name**, so two definitions called the same thing in different
    folders share one id.
  - **What to do:** Rename one of them. Folders do not separate them.
- **A widget name that collides** (`ESnapSetupCheckCode::WidgetNameCollision`)
  - **What it means:** A default widget-name contract asks for a widget whose name matches a `UPROPERTY` on the same
    class chain. That is the shape that puts a Widget Blueprint into a **permanent compile error** the moment someone
    adds the widget.
  - **What to do:** Give the widget another name in the Widget Blueprint and point the contract (a class default of that
    widget class) at it.
- **A key this plugin raw-binds that something else consumes** (`ESnapSetupCheckCode::MappingContextKeyClaim`)
  - **What it means:** One of the plugin's raw key bindings uses a key an `InputMappingContext` in this project also
    maps, to an action with **Consume Input** ticked. The finding names the key, the binding and every consuming
    claimant.
  - **What to do:** Move one of the two off the key. See below for what this can and cannot know — it reports the
    conflict, not which side wins.
- **A Blueprint subclass nothing reaches** (`ESnapSetupCheckCode::InertBlueprintSubclass`)
  - **What it means:** A Blueprint of yours derives from one of the plugin's types and nothing in the project points at
    it — no setting names it and no asset references it — so it is built but will never be constructed.
  - **What to do:** Point the setting that takes that type at it, or embed it in an asset that is itself reached.
    Ordinary for a Blueprint you are still building. See below for what counts as reaching one.
- **A plugin your project enables that this build lacks** (`ESnapSetupCheckCode::DependencyNotCompiled`)
  - **What it means:** Your project enables `GameplayAbilities` or `CommonUI`, and the copy of the plugin you use was
    compiled without it — as a storefront's *Install to Engine* copy is expected to be. The features behind it are
    absent; the runtime logs the same thing once at startup.
  - **What to do:**
    - Nothing, if you do not need them: an engine-installed copy is expected to be this build.
    - To get them, put the plugin in your project's own `Plugins/` folder, where it is compiled with the project — a
      Blueprint-only project's too, when the editor offers to on opening — with the plugin enabled in the `.uproject`
      first.
    - A copy already in the `Plugins/` folder that was compiled before you enabled it only needs rebuilding: from your
      IDE, or delete its `Binaries` and `Intermediate` folders and reopen.
    - The note names the fix for where your copy is.
- **An outline the renderer cannot draw** (`ESnapSetupCheckCode::OutlineStencilNotWritten`) — **warning**
  - **What it means:** The interaction outline is on (`bInteractionOutlineEnabled`, on **Snap Inventory System - Interaction → Interaction → Outline**,
    the default), and `r.CustomDepth` is
    not 3. The outline reads the custom-depth stencil, which the renderer clears and writes only at 3 — the engine default
    is 1 — so the prompt appears and nothing is outlined.
  - **What to do:** Set **Project Settings → Engine → Rendering → Postprocessing → Custom Depth-Stencil Pass** to
    **Enabled with Stencil** (`r.CustomDepth=3`), or turn `bInteractionOutlineEnabled` off. Silent at 3 and with the
    outline off.

**Warnings and notes:**

- The two cook-gap findings and the outline finding are **warnings**. A demo folder nothing cooks is a note.
- Everything else is a note, and nothing is ever an error: none of it is a failure.
- A project can ship with notes outstanding.

**A cook finding is silent when:**

- The folder is already covered by a `DirectoriesToAlwaysCook` entry (its own or a parent's).
- `Cook everything in the project content directory` is on.
- Or some asset **outside** the folder already references it.
  - The cooker follows references, so referencing a mapping context from a Blueprint that is cooked works just as well as
    listing its folder.
  - A reference from inside the same folder does not count, because the same gap would drop that too.

⚠ `DirectoriesToNeverCook` beats `DirectoriesToAlwaysCook`, so a folder named in both is still dropped, and that is
reported as well.

### A contested key

A key finding is raised only when the claiming action **consumes** input.

- A mapped key is not a consumed key.
  - An action whose Consume Input is clear passes the key along, and the plugin's raw binding fires perfectly.
  - So reporting that would be telling you a working keybind is broken.
- The plugin's own bindings never consume.
  - Every one of them clears the flag, so that a plugin binding cannot quietly take a control away from your game.
  - That is why the other side's flag is the one that decides.

⚠ **It reports the conflict, not an outcome.** Which of the two wins depends on where each is bound.

- `APlayerController` builds its input stack with the pawn's input component before the controller's, and then walks it
  in reverse.
  - So a raw binding on the controller can be evaluated *first*.
- Either the raw binding never fires (a dead key) or both fire for one press (a double action).
- Both are bugs and both are worth the note.
- Naming the wrong one would send you hunting for a dead key while your key fires twice.

⚠ **It cannot know which contexts your game applies.**

- It reads the mapping context *assets* in the project.
  - So a context you never add to a player is reported here and claims nothing in practice.
- The runtime side is the one that can tell: play, and the log names the keys that were really contested on the player it
  saw, from the live applied mappings.
- This scan is the half that can answer before anyone presses Play.
- It also covers two bindings the runtime warning never did:
  - The screen-open ability keys.
  - The test pickup spawner's key (`SpawnKey`, `L` by default) while **Enable Example Item Spawner** is ticked.

**What is not gathered:**

- Keys whose "bind raw" setting is off are not gathered at all: nothing binds them, so nothing can clash with them.
- The same goes for the spawner's key while the spawner is switched off.
- Neither are the two hotbar page **modifiers**.
  - They are read from the live key state while the page key is pressed, and a key read that way cannot be consumed by
    anything.

### A Blueprint subclass nothing reaches

A Blueprint that derives from one of the plugin's types — a slot widget, a panel, an inventory component — and that
nothing in the project points at is inert.

- The plugin has no way to construct it, so it does nothing at run time.
- The check finds them through the asset registry's own record of each Blueprint's native parent, so it loads nothing.

There are **two** ways to reach one, and either is enough:

- A **setting** names it — a widget-class knob, a component class on an attach row.
- **Any asset references it** — a parent Widget Blueprint that embeds it, an equipment slot set or the container registry
  naming its class, a level that places it.

⚠ **The second is the usual one, and it is why this check is quiet.**

- Most widget subclasses are reached by being dropped into a parent Widget Blueprint rather than by being named in Project
  Settings — on the project this was measured against, 25 of 37.
- A check that asked only about settings would report all 25 correct Blueprints as dead.
- And a report that cries wolf is worse than no report.

The finding names the **plugin type** the Blueprint derives from, rather than a setting to change.

- Several knobs take the same base type, so the check cannot know which one you meant.
- Naming the wrong one would send you to edit something unrelated.

It skips Blueprints inside the plugin's own `/SnapInventorySystem/` content.

- The plugin ships more than one example of several surfaces — two inventory screens, a spatial screen, a keyring panel, a
  hotbar.
- The alternatives you do not switch to are unwired on purpose.
- Reporting those would be the setup check reporting the plugin to itself.

A Blueprint you have not wired up yet looks exactly like one you forgot, which is why this is a note.

### Three ways to run it

- **Tools → Snap Inventory Setup Check** fills the **Snap Inventory Setup** message log and opens it.
- **`Snap.SetupCheck`** writes the same findings to the console and to the same message log.
  - It deliberately does **not** open the window.
  - An open message log floats over the viewport and swallows mouse clicks, which would break an automated session that
    ran the command.
- **At editor start**, a quiet pass writes one summary line under `LogSnapInventorySetupCheck`.
  - It asks for a notification only when something warns.
  - It opens no window whatever it finds.
  - It loads nothing: it reads class defaults, raw config and the asset registry's index.
  - It waits for the registry's first scan to finish, so that "nothing references this" is an answer rather than a guess.
  - The checks that would need an asset in memory — the attach rows and the key claims — are **skipped** there and say so
    in the log, rather than reporting a clean result from a partial picture.
  - Run it from the Tools menu to include them.

**The API:** `USnapSetupCheckLibrary::RunSetupCheck` returns the findings as an array, and that array is the whole API.

- The message log, the console output and the startup summary are all views of it.
- It is `BlueprintCallable`, so a project that wants its own report, or a commit gate of its own, can call it and read the
  findings directly.

## Registering item definitions with the Asset Manager

**Nothing in the plugin needs the Asset Manager.**

- Item definitions report a primary asset id of type `InventoryItemDefinition`, named after the asset.
- So a project that looks items up by id, or wants the Asset Manager to decide which items are cooked, can register the
  type itself.
- The plugin does not register it, and ships no configuration for it.
  - That is because a project may already declare the type, and two declarations of one type are an error.

Add this to your project's `Config/DefaultGame.ini`, listing the folders your item definitions live in:

```ini
[/Script/Engine.AssetManagerSettings]
+PrimaryAssetTypesToScan=(PrimaryAssetType="InventoryItemDefinition",AssetBaseClass="/Script/SnapInventorySystem.SnapItemDefinition",bHasBlueprintClasses=False,bIsEditorOnly=False,Directories=((Path="/Game/Items"),(Path="/SnapInventorySystem/Examples/Items")),SpecificAssets=,Rules=(Priority=-1,ChunkId=-1,bApplyRecursively=True,CookRule=Unknown))
```

- **One entry for the type.**
  - A second `PrimaryAssetType="InventoryItemDefinition"` entry makes the engine log an error saying only one entry per
    type is supported.
  - If your project already declares the type, edit that entry instead of adding this one.
- **Asset names must be unique across the listed folders.**
  - The id is the asset's name, not its path, so `/Game/Items/ID_Sword` and `/Game/Weapons/ID_Sword` would have the same
    id.
  - The engine logs a warning for the second one.
  - In the editor it also shows a notification naming both, and outside the editor it raises an ensure.
- **`CookRule` is your decision.**
  - `Unknown` leaves cooking to the usual rules.
  - Set `AlwaysCook` if the Asset Manager should cook every item in those folders whether or not anything references it.
- **Restart the editor** after changing the block.

## Removing the editor module

The editor module can be deleted.

**To remove it:**

1. Remove `Source/SnapInventorySystemEditor`.
2. Remove its entry in `SnapInventorySystem.uplugin`.

The runtime module, the CommonUI module and the tests module do not depend on it, and nothing outside it refers to it.

**What you lose:**

- The Snap Inventory menu.
- The item thumbnails.
- The footprint shape editor.
- The setup check.
- The editor tests listed in [Testing and diagnostics](testing-and-diagnostics.md).

**Footprints are still validated without it:** The rules live in the runtime module.
