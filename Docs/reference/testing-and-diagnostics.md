# Testing and diagnostics

Four things help you see what the inventory is doing:

- An automation test suite you can run in your own project.
- A **SnapInventory** page in the gameplay debugger.
- Two console commands.
- A **SnapInventory** stats group.

**On this page:**

- [The automation tests](#the-automation-tests)
- [The network tests](#the-network-tests)
- [The gameplay debugger page](#the-gameplay-debugger-page)
- [Console commands](#console-commands)
- [The stats group](#the-stats-group)
- [See also](#see-also)

## The automation tests

The plugin includes a `SnapInventorySystemTests` module. It is declared `DeveloperTool` in the `.uplugin`.

- Unreal Build Tool builds a module of that type only when a target's `bBuildDeveloperTools` is true.
  - By default it is for editor and Development targets, and is not for Shipping or Test targets.
- Nothing in the runtime module depends on it.

Without `GameplayAbilities` the tests that need it are not compiled:

- They live in `Source/SnapInventorySystemTests_RequiresGAS/`, which the tests module adds only when it is on.
- `StartsWith:SnapInventory` then finds **569** tests instead of **706**.
- Two of the eighteen network tests need it as well, so sixteen remain.
- Three of the twenty `Death.*` tests are among those left out: the ones that raise or read the death tag through an
  ability system.
  - The other seventeen live in the tests module itself and run either way.
  - The two that need a second container on the player use a custom container configured like the shipped Shards one
    (`USnapCustomContainerComponent`), which is always built.

The tests that check editor tools — `Editor.AssetDefinitions`, `Editor.ItemThumbnail`, the eighteen `Editor.SetupCheck`
tests and the seven `Editor.Footprint` tests — live in the editor module, `SnapInventorySystemEditor`, instead.

- A Development game target builds the tests module but cannot link an editor module.
- They run in the editor, with the rest. See [Editor tools](editor-tools.md).

### Running them

Type `Automation RunTests StartsWith:SnapInventory` in the editor console.

- Each test writes a `Test Completed. Result={Success}` or `Result={Fail}` line to the output log, followed by the
  assertion that failed.
- Each test builds its own throwaway game world, adds the components it needs to plain actors, and uses transient item
  definitions made in memory.
- The suite saves nothing.
- The editor tests build no world: they make transient assets.
  - The footprint tests edit theirs through the rows a Details panel is built from, and undo through the editor's own
    undo history.
  - The thumbnail test renders four small thumbnails. Where nothing can render (a `-nullrhi` run) that test skips its
    pixel comparison and says so in its output.

### What they cover

| Test | Claim |
|---|---|
| `Smoke.AddItemFillsStacksToTheirCap` | An add fills stacks to their cap, in slot order |
| `Conservation.SplitMergeStoreTakeDropCollect` | Units are conserved across split, merge, store, take, drop and collect, and every step really happened |
| `Refusals.SplitStackNamesEachReason` | The reasons a split in a slot-layout container can be refused, each asserted by value, beside a split that succeeds |
| `Refusals.MoveItemNamesEachReason` | A move's ownership and range refusals, beside a move to an empty slot and the swap an occupied slot performs instead |
| `Persistence.RecordRoundTripIsFieldEqual` | A save record exported and imported into another container reproduces it, record against record and in live state |
| `AntiCheat.ClientPricedPaymentIsShutUntilOpened` | The `bAllowClientPricedPayment` flag really refuses a client-stated price, and really allows it when on |
| `AntiCheat.ClientVendorSessionIsShutUntilOpened` | The `bAllowClientVendorSession` flag really refuses a client-named shop; closing a shop is never refused |
| `NetLimits.StoredSlotMoveBoundIsTheSharedLimit` | The stored-item move accepts indices up to one below `FSnapNetLimits::SlotIndexLimit` and refuses the limit and a negative index |
| `Validation.DuplicateContainerTagWarns` | Two containers answering one tag on one actor are reported; distinct tags are not |
| `Debugger.PageShowsTheSelectedPlayersContainers` | The debugger page describes the selected player, not the one looking |
| `Stats.SweepScopesCountOnlyWhileSweeping` | The stats group counts the spoilage sweep and an add while they run, and reads zero for the sweep while it does not |
| `Theme.PrecedenceIsPropertyThenWidgetThenProject` | Every combination of a color's Source, its widget's Theme Usage and the project's Default Theme Usage resolves to the first level with an opinion — all 27, written out literally, plus a value outside the enum reading as no opinion |
| `Theme.NoPickedThemePaintsTheAuthoredColor` | With no theme picked the authored color paints even when every level says Theme; with a theme picked the same arguments paint the role |
| `Theme.EffectivePaletteCarriesTheProjectDefault` | The palette a widget paints from carries `DefaultThemeUsage` on both branches — no theme set, and a theme that loads |
| `Theme.SaveWarningNamesThemeSourcedRestyles` | The save warning names exactly the colors that differ from the parent class's defaults while their own Source is *Theme* — not a color on *Inherit* or *Authored*, not one whose widget's Theme Usage is *Theme*, not a copy equal to a restyled parent, and it does name one equal to the C++ value but not to its parent (its deliberate breaks compared with the C++ defaults, honored the widget's Theme Usage, and dropped the Source condition) |
| `Theme.EveryAuthoredColorSourceGovernsARealColor` | The nine widget classes carry 22 color Sources, each naming a real color beside it, and one Theme Usage each, all defaulting to Inherit (its deliberate break was a changed default on the class default object) |
| `Theme.TypographyDefaultsAreTheShippedLiterals` | Every typography role's shipped default is the size, typeface and spacing that text used before roles existed — written out literally, one row per role — and the table covers every role; a copy with two roles changed must be named exactly |
| `Theme.TypographyBuildsTheShippedFontInfo` | Each role, with no family named anywhere, builds the identical `FSlateFontInfo` its text used to build; the hotbar keeps its TextBlock's font; an authored Prompt Font, a role's own family and the theme's Font Family win in that order |
| `Theme.EffectiveTypographyFollowsTheTheme` | With no theme set the hotbar roles' sizes follow their settings; with a theme that loads, its typography is used and those settings are not consulted |
| `Death.EachKnobMovesOnlyItsTarget` | Each death-drop policy knob keeps exactly its own target, through the death door and a direct `DropEverythingToWorld` |
| `Death.TagAndDoorDropOnce` | The death tag and `NotifyOwnerDied` for the same death drop once; the tag's removal re-arms it |
| `Death.OfferedItemDropsThroughTheDoor` | An item on the trade table comes home and drops through the door; a direct `DropEverythingToWorld` leaves it in escrow |
| `Death.AProjectCanSayItsPlayerIsDead` | A bag subclass overriding `IsOwnerDead` decides who is dead with no tag and no ability system: a dead owner's trade is refused from both sides, a live one's opens, the trade path asks the override, and the plain component on the same rig answers alive |
| `Death.TheScreenAsksTheSameDoor` | The inventory screen's dead check asks the main bag's `IsOwnerDead`, reading alive and then dead as the override does, and alive for the plain component |
| `Death.WhichDeathTagEachCheckReads` | The screen's and the trade path's dead checks give one answer for every combination of the death-tag setting and the component's own member; every row that moved when they began asking one door moved from alive to dead |
| `Death.AControllerHostedBagAsksThePlayerStatesAbilitySystem` | A main bag on the controller, with the ability system on the PlayerState and no pawn, reads a dead player as dead through the screen and the trade path, and alive once the tag is removed |
| `Death.AFallBelowTheKillZDropsAtTheLastWalkablePlace` | A death below the world's KillZ leaves its body and scatter at the last walkable place the character stood, above the KillZ; with none recorded the player keeps everything; a death above the KillZ, or with the bounds checks off, drops where the character died |
| `Consume.AFragmentOfYourOwnMakesAConsumable` | An item whose only consume behavior is a fragment's `OnConsumed` is consumed: the hook runs once, on the main bag, and one unit is spent |
| `Consume.OnlyADeclaringFragmentIsOffered` | The bag menu offers Consume only for an item whose fragment declares a grant; a hook that declares nothing and an item with no payload are not offered, and consuming the first is refused without running its hook or spending it |
| `Consume.ARefusedGrantCostsNothing` | A fragment whose `CanApplyConsumeGrant` says no refuses the whole consume and keeps the unit, and the menu draws no Consume that could only refuse; saying yes spends one unit |
| `Consume.AHookJoinsTheEffectPayload` | With an effect payload as well, the effect still lands and the hook still runs once |
| `Trade.OfferedContainerContentsAreCopiedWhileOffered` | Offering a loaded container copies its whole tree, nested contents included; an empty container still gets an entry and a plain item none; a withdraw, the end of the session and a completed trade each leave no copy behind |
| `WidgetClass.ResolverRefusesWhatCannotWork` | The widget-class resolver refuses an unloadable path, a wrong type, an abstract class and a designer-built tree on a class with no named parts, and accepts a class-defaults-only Blueprint, a native subclass and a designer-built hotbar; the designer-tree test and the contract table are pinned row by row; one warning per knob and path |
| `WidgetClass.NamedPartDefaultsAreSafe` | Every `...WidgetName` default on the confirmation window, themed tooltip, trade banner and feed row is not None, is unique within its class, and is not the name of a property on the class chain (a widget with that name would leave a Blueprint unable to compile); the count of named parts per class is pinned |
| `Layout.CraftDerivedValuesFollowTheirInputs` | `FSnapCraftPanelLayout`'s framed detail content width, header and empty-list wrap widths and standalone queue width follow a moved column, padding, icon, inset and gutter, with their floors; a default struct is the shipped layout, field by field; the struct's fields are exactly the inputs, none named like a measured band or a derived width; `CraftLayout` is a class default (its deliberate break returned the default content width whatever the inputs) |
| `Layout.TradeDerivedValuesFollowTheirInputs` | `FSnapTradeScreenLayout`'s offer grid width, grid height for a row count, status wrap width and column count follow a moved square size, gap and column count, with their floors; a default struct is the shipped layout; the fields are exactly the inputs, with no row count; `TradeLayout` is a class default (its deliberate break returned the default status wrap width whatever the grid) |
| `Localization.EveryShownStringIsGatherable` | Every refusal reason and title, every trade label, the count, key-label and tooltip-line formats, the import report's currency message and one message from each asset validator (one from each namespace merged into `SnapInventorySystem` among them) is gatherable in the `SnapInventorySystem` namespace; counts read `x1234` and `1234/2000`, never grouped; a reduced currency is named by its `Currencies` row, and by its tag without one |
| `Localization.ShippedAssetTextIsKeyed` | Every shipped asset under the plugin's mount (not `Content/Tests`) is loaded and every text property of every saved object read: a culture-invariant text off the eleven-entry allow-list fails, naming asset, object and property, and so does an entry that no longer matches; widget Blueprints are read through their designer tree, and a package needing a class the build lacks is skipped before it loads |
| `Localization.ShippedAssetTextScanFindsAPlant` | That scan reports planted culture-invariant text on an instanced subobject, in an array of structs and in a map of structs, and not keyed, empty or transient text |
| `StackCounts.AnEquipmentSlotDrawsBareDigits` and the eleven beside it | A stack of 1234 reads `1234`, never `1,234`, on each surface that draws a stack count — the example equipment slot, a hotbar square, a pouch square, a loot square, a trade offer square, the drag ghost, a HUD readout mirrored through `EquipmentHUDBindings`, the collect feed (`x1234`), a sale's line in the example sell window and in the example vendor screen's buy panel (`x1234`) and the example grid and shard slots — one test per surface, each first checking that the running culture groups 1234 |
| `BadgeNumbers.HotbarChargesOfAMaxAreBareDigits` and the twelve beside it | 1234 reads bare on each badge that is not a stack count — a hotbar square's charges (`1234/2000`, and `1234` with no limit) and cooldown seconds, the page indicator (`1/1234`), the interaction prompt's option count, the craft queue tile (`x1234`, with a themed tile's name line kept at `x1,234` as the control), a recipe row's output badge, requirement chip and `+1234` overflow chip, the crafting detail pane's output badge and have/need chip, and a shop row's stock and buyback badges — one test per readout, each first checking that the running culture groups 1234 |
| `Input.ShippedMappingNamesAreUnique` | No two shipped input actions answer to one rebinding-row name, by their own settings or through a shipped mapping context; two planted actions sharing a name are caught |
| `Commands.GiveItemArgumentsNameOneItemAndACount` | `Snap.GiveItem` matches an asset name ignoring case, lists every path for a name two folders share instead of picking one, picks one by full or package path, matches nothing for an unknown name or a folder, and accepts counts from 1 to `FSnapNetLimits::MaxUnitsPerRpc` only |
| `Editor.AssetDefinitions.EveryTypeIsNewUnderSnapInventory` | Each of the thirteen authored types resolves to its own asset definition, named and described, in exactly the Snap Inventory category, with one factory that shows in New under Snap Inventory and not Miscellaneous and creates exactly that class; the engine's data asset classes and factory are not in it |
| `Editor.ItemThumbnail.DrawsTheItemsIcon` | Item definitions, and not a vendor stock asset, use the item thumbnail renderer; it redraws on property change, declines an item with no UI Data or no icon, and draws two items iconed white and gray as their own icons. Without rendering (`-nullrhi`) the engine creates no thumbnail renderer at all, so the test reads the registration from the thumbnail manager, checks the rules on a renderer it makes itself, and skips the rest with a Warning |
| `EquipmentSlotSet.PrecedenceFollowsTheSetting` | Unset, a loaded set, an empty set in force and an unloadable one each give the ruled answer |
| `EquipmentSlotSet.RowOrderPicksTheFreeSlot` | The set's row order is the auto-equip tie-break, asserted in both orders |
| `EquipmentSlotSet.NamesFallBackPerRow` | The set's name wins, a blank row falls back to the tag split, and an unlisted slot is still named |
| `EquipmentSlotSet.WearMultiplierReplacesTheMap` | The set replaces the wear map, and a row at 1.0 means the normal rate rather than the map's value |
| `EquipmentSlotSet.ValidationRefusesUnusableRows` | Empty, duplicate, tagless and parent-of-another rows are refused; sibling leaves validate |
| `EquipmentSlotSet.GhostIconPrefersTheMapThenTheSet` | The authored map wins by exact tag and by parent, a slot it does not cover takes the set's icon, and a mapped entry that will not load stops the walk instead of falling through |
| `EquipmentSlotSet.VisualTakesTheItemSocketThenTheRowSocket` | The item's own socket wins, the slot row's is the default, and the offset comes from whichever side supplied the socket; the rig's sockets are asserted to resolve first |
| `EquipmentSlotSet.DroppingASlotChangesOnlyThatSlot` | Dropping a slot changes exactly that slot's `CanEquipInstanceInSlot` and `FindFreeSlotForItem` rows and no others, adding one does the same in reverse, and clearing restores the table |
| `EquipmentLock.DoorsRefuseALockedSlot` | The oracle, four selectors and six put-in doors all refuse a locked slot, and every one of those gestures succeeds after `UnlockSlot` |
| `EquipmentLock.RelockKeepsAndReleasesWhatIsWorn` | Locking a slot under a worn item neither removes it nor traps it, while a new put-in stays refused |
| `EquipmentLock.UnlocksSurviveARecordRoundTrip` | An explicit unlock exports and imports into a different component; a tag-derived one is never written |
| `EquipmentLock.ImportRestoresIntoALockedSlot` | A worn line restores into a slot that is still locked, while the ordinary door on that slot is still refused |
| `EquipmentLock.UnlockRecordsOnlyWhatItMust` | An unlocked slot, a slot outside the set and a repeat all answer yes and record nothing extra |
| `EquipmentLock.AnUnlockTagOpensAndClosesTheSlot` | Granting the tag opens the slot and lets an item in, losing it re-locks while what is worn stays worn, and an explicit unlock outranks the tag |
| `Editor.SetupCheck.KeysAndTagsFindOnlyRealMistakes` | A struct-form `FKey` and an unregistered tag name are reported; a bare key, an unset key, a registered tag and a blank one are not |
| `Editor.SetupCheck.SeverityIsTheRuledTable` | Severity is a pure function of the code, only the two cook-gap codes and the outline-stencil code warn, and the table has a row for every code |
| `Editor.SetupCheck.CookGapsWarnOnlyWhenPackagingWouldDrop` | An uncovered folder warns; a covering entry, a parent entry, `bCookAll` and an outside referencer each silence it; `NeverCook` beats `AlwaysCook`; one finding per folder names every property |
| `Editor.SetupCheck.ExampleDefaultsNameOnlyExampleContent` | Example-content paths are reported per property and other plugin content is not |
| `Editor.SetupCheck.AssetManagerIdsAndDuplicates` | An undeclared item type and a shared asset name are reported, with different wording for a clash the Asset Manager scans |
| `Editor.SetupCheck.NoWidgetNameContractClashesWithItsOwnClass` | Nothing clashes, and a non-reflected member, an inherited contract and cross-class part names are correctly not counted |
| `Spatial.MaskedAnchorIsNeverShared` | In a 4 x 4 spatial bag, an L whose top-left cell is empty and a one-cell item added after it carry different indices (the one-cell item takes cell 2), and so do an L turned 270 degrees in place and a one-cell item after it; a move onto the L's empty corner swaps the two; no index and no cell is held twice (its deliberate break stopped the occupancy map claiming the empty corner) |
| `Spatial.MaskedShapeRefusedAnOccupiedAnchor` | With a one-cell item on cell 0, an L whose top-left cell is empty does not fit at 0 — refused as `TargetOccupied`, naming exactly cell 0 — and an add lands it at cell 1 (its deliberate break stopped the fit test checking the anchor cell) |
| `Spatial.AddNeverDuplicatesAnIndex` | The same masked add over a held corner leaves the two items on different indices even when the fit test is wrong (its deliberate break removed that check together with the list's own guard) |
| `Spatial.MaskedAnchorIsNeverSharedInAPouch` | In a pouch with a 4 x 4 grid, a stored L keeps a one-cell item off its empty corner (cell 2), and an L asked for a corner a one-cell item holds lands at cell 1 (its deliberate breaks stopped the pouch's map claiming the corner, and removed the pouch's guard together with the fit test's check) |
| `Spatial.DrawOrder.NotchDrawsAboveTheShape` | A T's empty cell and a one-cell item standing in another of its empty cells are both drawn above the T; its covered cells and a cell outside its box keep the cell layer (its deliberate break drew every item at the two old layers) |
| `Spatial.DrawOrder.InterlockIsReported` | Two corner shapes each standing in the other's empty cell are both reported, the lower index is drawn on top, and listing them the other way round changes nothing (the same deliberate break) |
| `Footprint.TheFiveRefusalsKeepTheirWords` | A box below 1 x 1, a wrong-length mask, a mask covering nothing, an empty row, an empty column and two pieces each produce exactly one Error with its unchanged localization key, in the plugin's namespace; two valid masks and a rectangle produce nothing |
| `Footprint.AFlaggedShapeStaysValid` | IsDataValid on a shape with an empty top-left cell returns not-Invalid with no error and exactly one warning, the anchor one; a split shape is Invalid with one error |
| `Footprint.UncoveredAnchorWarnsInEveryAllowedTurn` | The warning table: an L cut at the corner warns; an L covering it is silent unless it may turn; a T is silent until it may turn; a ring and every rectangle are silent |
| `Footprint.ResizeKeepsEachCellWhereItWas` | The pure remap, a typed width through the edit hooks, and a drag through the hooks that passes a width of 1 on its way to 2 all keep every cell at its (x, y); an empty mask stays empty |
| `Footprint.AFullMaskIsStoredEmpty` | Painting the last uncovered cell through the edit hook stores an empty mask; a mask one cell short of full is kept |
| `Editor.Footprint.OneUndoRestoresSizeAndShape` | Through a row generator's Width handle, a typed commit and the numeric slider's exact sequence (its transaction, Interactive sets, the final set) each crop a 3 x 3 L by (x, y), each is one transaction, and one Undo restores width and shape |
| `Editor.Footprint.TheValidityLineIsIsDataValid` | The validity line for a split shape reports an Error carrying IsDataValid's own message; a valid shape's line says it is valid |
| `Editor.SetupCheck.UnfoundAttachClassesAreNotedPerField` | An unfound receiver and an unfound component class are reported per field with their own wording, and a disabled row is not |
| `Editor.SetupCheck.AGameplayAbilitiesOnlyRowIsExpectedWithoutGameplayAbilities` | Without `GameplayAbilities` a row naming the listed class yields nothing, for either field; with it the row is checked whole; a misspelled plugin class in the same package is reported in both builds |
| `Rarity.ValidationWarnsAboutASetByCallerGrantedEffect` | A tier whose `GrantedEffect` has a `SetByCaller` modifier is warned about once, by name; a Scalable Float tier and an unset tier are not |
| `Rarity.ValidationWarnsAboutAnInstantGrantedEffect` | A tier whose `GrantedEffect` is Instant is warned about once, by name; Infinite and Has Duration tiers and an unset tier are not |
| `StatContributions.ARowTheDisplayTableCannotWordIsWarnedAbout` | A stat the display table has no row for is warned about once, though two rows name it; a worded stat, a cleared setting and an unloadable table say nothing |
| `FocusEntry.TheFirstFocusableCellSkipsWhatCannotTakeFocus` | Eight cell lists built from real widgets: empty, a usable first cell, a collapsed one, a hidden one, a null entry, nothing usable at all, and one that is visible and enabled but was never made focusable (its deliberate break dropped that last term and returned that cell); a final row pins the deliberate asymmetry between this leaf-only pick and the ancestor-walking `IsCellUsable` |
| `FocusEntry.TheResolverWalksPanelsInOrderUntilOneOffersAFocusableCell` | The real panel walk with collection handed in: nothing open resolves to nothing and asks every panel in order (its deliberate break swapped the bag and the shop rows); a bag present with every cell collapsed does NOT end the search (a second break ended it at the first panel that was merely present); and the order is checked against the reflected enum rather than a second hand-written list |
| `FocusEntry.TheResolverSeamHonorsItsCollectorContract` | What the resolver promises a collector a project wrote itself, over inputs the plugin's own collector cannot produce: cells handed back with a FALSE return are ignored, a panel that answers present with no cells does not end the search, every panel is handed a fresh array, and each is asked exactly once and none after the resolution (its deliberate break ignored the collector's return value and resolved to the cell a panel had disowned) |
| `Affixes.TheFragmentRefusesAClientBeforeTheTableIsEverAsked` | A client does not merely end up with no affixes — it never asks the table to roll, and the fragment says why. A counting table answers the first half and a log counter the second, with the count shown to move on the authority control (its deliberate break deleted the fragment's own authority return, leaving the write seam underneath to refuse: the outcome was unchanged and the table was asked anyway) |

⭐ **Every test in this table has been run against a deliberate break of the code it covers and seen to fail**, and seen
to pass again once the break was removed.

- A test that passes on broken code proves nothing, so none is listed here that was not shown to fail.

### A host project's own subsystems

The test worlds run your project's world subsystems too.

- A subsystem that never calls `Super::PostInitialize` or `Super::OnWorldBeginPlay` makes the engine raise an ensure for
  every world, which would fail every test for a reason unrelated to the inventory.
- So those ensure messages are expected rather than counted.
- Log errors raised **while a test world is being created or destroyed** are not counted.
- Anything else logged while a test is running still fails it.

### What the suite does not cover

- **Replication.** Every test runs on a single authority.
  - What reaches a client is covered by the separate network suite below, not by this one.
- **Spatial layout.** Apart from the masked-footprint rows above, the refusal tests use slot-layout containers.
  - A spatial container's other shape refusals are not asserted here.
- **Slate input routing.** No test routes a real event.
  - Which widget a real pointer is over, and which one a real click, key press or focus change reaches, is not
    exercised.
  - Some tests call a widget's own drag-over, drop, key or focus handler directly — the spatial panel's gesture tests
    do, on a grid painted offscreen.
- **Focus entry through the HUD.** The `FocusEntry` rows test the resolver the HUD calls
  (`USnapInventoryHUDComponent::ResolveFirstNavigableCell`), not the HUD's own call (`ResolveFirstNavigableSlot`).
  - Only the bag's choice is driven through the HUD, by `FocusBag.TheMainInventorysGridGetsThePad`.
  - The other panels need state that only a live open fills, and a game viewport.

## The network tests

`SnapInventoryNet.*` uses the engine's multi-client test harness (CQTest's `FPIENetworkComponent`).

- It starts a play-in-editor session with a server and two clients in one editor process, and asserts what each client
  received.
- They are a separate suite because of what starting one does.

### Running them

1. ⚠⚠ **Save your level first.** The harness's first step creates a new, unsaved map in place of the one you have open,
   and every test ends play.
   - That is why the ordinary command above says `StartsWith:` — without it, `Automation RunTests SnapInventory` matches
     every test whose name contains the word, and starts these too.
2. Type `Automation RunTests StartsWith:SnapInventoryNet` in the editor console.

The session uses the engine's own `UGameInstance` and `AGameModeBase`, not your project's.

- So your project's online subsystem and experience setup do not take part.
- Each test spawns replicated actors on the server, gives each a container owned by one client, and uses the shipped
  example potion and pouch definitions.
  - A definition made in memory cannot be sent to a client.

The two stat tests below use a test-only item instead, `ID_TestStatPlate`, which lives under `Content/Tests`.

- `Config/FilterPlugin.ini` keeps that folder out of the packaged plugin.
- So in a packaged copy each of them warns `SKIPPED, NOT PASSED` and checks nothing.
- A copy that has the folder but not the item fails them instead, because that is a broken copy rather than a packaged
  one.

### What they cover

Every test below runs twice:

- Under `ListenServerReplication` and under `DedicatedServerReplication`.
- Or, for the one that needs `GameplayAbilities`, under `ListenServerGameplayAbilities` and
  `DedicatedServerGameplayAbilities`.

The dedicated-server classes are opt-in; see below.

| Test | Claim |
|---|---|
| `EveryoneContainerReachesEveryClient` | An `Everyone` container's stack arrives on both clients, including the one that owns nothing |
| `OwnerReceivesContentsAndLaterChanges` | The owner of an `OwnerOnly` bag receives its contents, then a later change to its units and its capacity |
| `OwnerOnlyContentsNeverReachANonOwner` | A client that does not own an `OwnerOnly` bag receives none of its grid rows, stored rows or capacity, and — asserted separately — none of its item objects; believed only after an `Everyone` container spawned beside it has arrived on that same client |
| `TradeCopyReachesThePartnerOnlyWhileOffered` | The copy a trade makes of an offered `OwnerOnly` pouch's inside arrives on the partner with every row resolved, and is gone once the pouch is withdrawn |

⭐ **Every test in this table has been run against a deliberate break of the code it covers and seen to fail**, against
both servers, and seen to pass again once the break was removed.

`BlockedSlotReachesTheClient`: a client reads the off hand as blocked while the server has a two-handed item worn, names
the item that blocks it, and reads the slot free again once the item comes off.

`RolledAffixesReachTheOwnerUnchanged`: an item's rolled affixes reach their owner with the server's own magnitudes
rather than ones the client worked out.

`OwnerHearsACarryLimitChange`: when the server changes the bag's carry limit (`SetMaxCarryWeightKilograms`), the owning
client hears the bag's `OnInventoryChanged` and then the encumbrance component's `OnEncumbranceChanged`, once each, and
reads the new limit when the second arrives; the server's own copy hears each once as well. Clearing the limit again is heard the same way.

The last two read a worn item's stat rows on the owning client:

| Test | Claim |
|---|---|
| `EquippedStatTotalReachesTheClient` | After the server equips the test item, the owning client's `GetEquippedStatTotal` answers the server's Armor 5 (`Flat`) and MaxHealth 15 (`Percent`), a listener bound to the client's copy hears `OnEquippedStatsChanged`, and the total it read when it last heard it is 5; after the unequip, the total and the last total a listener heard are both 0 |
| `EquippedStatEffectsReachTheClient` | Needs `GameplayAbilities`. With the example Stat Effect Map, the effects equipping applies reach the owning client carrying their definitions — the example armor and vitality effects — and only then are Armor 5 and MaxHealth 115 asserted there, because an attribute value replicates on its own; after the unequip both effects are gone and the attributes read 0 and 100 |

⭐ Each was run against deliberate breaks and failed at the row that names the break, with no other network test failing:

- A total that sums nothing without authority.
- The change event raised on the server only.
- The equipment list announcing nothing once a removed entry is gone, which left the last total a listener heard at 5.
- The effects applied as an instance built at runtime instead of the authored class, which fails with the engine's own
  error that the client received an effect with no definition.

⭐ Against a dedicated server as well, opted in, in a project generated from this plugin, each of these failed the
listen-server AND the dedicated-server version of the test that covers them, at their rows, and no other test:

- A slot that never reads blocked without authority.
- The total that sums nothing without authority.
- The effects built at runtime.
- An item's memories withheld from its owner.
- An `OwnerOnly` bag's contents sent to every connection.

### The dedicated-server tests are opt-in

`DedicatedServerReplication` and `DedicatedServerGameplayAbilities` do nothing unless
`Snap.Tests.AllowDedicatedServerPIE` is 1.

- With it at 0 (the default) each test **warns that it skipped** and starts no session.
- A dedicated-server session runs the server as a dedicated-server instance inside the editor.
- A project whose online subsystem cannot create one for a dedicated server stops the editor outright when the session
  starts.
  - Lyra's CommonUser session subsystem asserts there, so leave it at 0 in a Lyra project.

Turn it on in a project where that works — for example in `Config/DefaultEngine.ini`:

```ini
[ConsoleVariables]
Snap.Tests.AllowDedicatedServerPIE=1
```

Or pass it to a command-line run, before the automation command and separated from it by a comma:
`-ExecCmds="Snap.Tests.AllowDedicatedServerPIE 1,Automation RunTests StartsWith:SnapInventoryNet;Quit"`.

⚠ **A skipped arm is not a pass, and the automation framework cannot say so on its own.**

- A test is Success or Fail — there is no state a test can report for "did not run".
- So with the variable at 0 these eight (seven without `GameplayAbilities`) still appear under Success in the log.
- What they carry is a warning naming the variable, which puts them in the report's succeeded-with-warnings group
  instead of among the plain passes.
- If you are counting the suite as evidence for a dedicated server, that warning is the thing to read.

⭐ **What says a dedicated-server test ran is its log line, not its Success**: Each one logs
`SnapInventoryNet: server net mode DedicatedServer` when its session starts.

- With the variable at 1, in projects generated by `Tools/New-ExampleProject.ps1` (in the plugin's source repository,
  not the packaged plugin) — no Lyra, with and without `GameplayAbilities` — every dedicated-server test logged that
  line once and passed.
- The same project with the variable at 0 logged it for none of them.

### Log output while play starts

A project's own systems can log errors and warnings every time play starts or stops.

- Those are not counted while the session is starting and ending.
- From the first assertion on, an error fails the test.

### What the network suite does not cover

- **A dedicated server, unless you opt in.** The dedicated-server arm is off by default, so nothing in the suite is
  evidence about a dedicated server until `Snap.Tests.AllowDedicatedServerPIE` is 1 in a project that can start one.
  - It cannot run in a Lyra project; it was run in projects generated from this plugin.
- **The registered subobject list on a dedicated server.** Those generated projects replicate item objects through the
  engine's legacy subobject path.
  - The registered subobject list (Lyra turns it on) is covered by the listen-server arm only.
- **Gameplay through a packaged dedicated server.** The tests run a dedicated-server world in single-process
  play-in-editor.
  - The generated demo project was also run by hand as a separate, uncooked dedicated-server process (`-server`) with
    two game clients — a pickup, an automatic equip and a completed trade.
  - Its packaged Development server target was cooked, run and joined by two packaged clients, which were then played
    through it by hand — a pickup, a drop to the world and a trade.
  - Not driven: an item one player wears seen by the other.
- **Iris.** The suite switches Iris replication off for its run.
- **The order in which a client applies an inventory removal.** The equipment list's is covered: a break that stopped it
  announcing again once a removed entry is gone failed `EquippedStatTotalReachesTheClient`.
  - The inventory list's is reasoned about — see **Known limitations** in the plugin README.
- **A worn item whose rarity or durability changes, on a client.** `OnEquippedStatsChanged` is raised for those on the
  server only, and neither stat test changes them.
- **Which remote procedure calls a client may send.** The tests change state on the server directly.

## The gameplay debugger page

1. Open the gameplay debugger (the apostrophe key by default).
2. Aim at a player, your own pawn or anyone else's.
3. Switch on the **SnapInventory** category.

The page lists every inventory container the selected player holds:

- Its tag.
- Its fill against its effective capacity.
- Carried weight against the limit.
- Each stack by slot, with pouch contents indented beneath.
- The wallet.

Where it looks:

- The containers are found on the selected actor itself, or else on its player state.
- With nothing selected the page describes you.
- The gameplay debugger collects a category's text where the game has authority and sends it to the viewer, so the page
  shows the server's view of the containers.

## Console commands

Two commands help while testing.

- Both are compiled out of Shipping builds.
- Both run only where the game has authority: typed into a client's console they are refused, because a client's
  inventory is only a copy.
- In a listen server, type them in the host's window.

| Command | What it does |
|---|---|
| `Snap.GiveItem <item> <count>` | Gives the local player `<count>` of an item definition |
| `Snap.DumpInventory` | Writes every player's containers to the console and the log |

In the editor, `Snap.SetupCheck` also writes the Setup Check's findings to the console; see
[Editor tools](editor-tools.md).

**`Snap.GiveItem`.**

- `<item>` is an item definition's asset name, such as `ID_Potion_Health` (case does not matter), or its full path, such
  as `/SnapInventorySystem/Examples/Items/ID_Potion_Health.ID_Potion_Health`.
- When two item definitions in different folders share a name, the command lists both paths and gives nothing; type the
  full path instead.
- `<count>` is a whole number from 1 to 100000 (`FSnapNetLimits::MaxUnitsPerRpc`).
- The items go to the first local player's pawn, or to its controller when it has none, through the same grant a pickup
  uses.
  - They route to their container or counter, stack, equip themselves where an item would, and write the `[GRANT]` line
    to the log.
- The reply says how many were given, and says so when some did not fit.
- A world with no local player, such as a dedicated server or the editor's level outside play, refuses and says why.

**`Snap.DumpInventory`.** For each player, in player id order, the command writes the lines the gameplay debugger page
shows, without its color tags:

- Each container with its fill and weight, its stacks and pouch contents, and its wallet.
- Worn equipment is not listed, as on the page.

## The stats group

Type `stat SnapInventory` in the console. The group times:

| Scope | Where |
|---|---|
| Spoilage sweep, Craft queue sweep | The two periodic sweeps |
| Add item, Move item, Split stack, Merge stacks, Store in storage, Drop to world | The authority bodies behind the most frequent requests |
| Export record, Import record | A save record being written or read |
| Panel refresh | The inventory panel rebuilding its slots |

- Each scope is the first statement of its function, so a request that is refused is still counted.
- ⚠ Only the **Spoilage sweep** and **Add item** scopes are driven by the automated test above.
  - The craft queue sweep runs only from its own timer, and the other scopes are placed but not exercised by a test.

## See also

- [Networking](networking.md)
- [Persistence](persistence.md)
- [Refusals](refusals.md)
