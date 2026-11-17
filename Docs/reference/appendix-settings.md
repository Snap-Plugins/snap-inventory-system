<!-- GENERATED FILE -- do not edit by hand.
     Produced by Tools/generate_docs.py from the shipped headers. Tools/ is in the plugin's source
     repository, not in the packaged plugin.
     Re-run it after changing a declaration; `--check` fails when it is stale. -->

# Appendix: settings

Every field of the plugin's settings classes, one class per part of the plugin, each a page under
**Project Settings -> Plugins**. A `config` field can also be set in your own `DefaultGame.ini`, under
its class's section:

| Class | Project Settings page | Section |
|---|---|---|
| `USnapCraftingSettings` | Snap Inventory System - Crafting | `[/Script/SnapInventorySystem.SnapCraftingSettings]` |
| `USnapEconomySettings` | Snap Inventory System - Economy | `[/Script/SnapInventorySystem.SnapEconomySettings]` |
| `USnapEquipmentSettings` | Snap Inventory System - Equipment | `[/Script/SnapInventorySystem.SnapEquipmentSettings]` |
| `USnapFoundationSettings` | Snap Inventory System - Foundation | `[/Script/SnapInventorySystem.SnapFoundationSettings]` |
| `USnapInteractionSettings` | Snap Inventory System - Interaction | `[/Script/SnapInventorySystem.SnapInteractionSettings]` |
| `USnapInventoryDemoSettings` | Snap Inventory System - Demo | `[/Script/SnapInventorySystem.SnapInventoryDemoSettings]` |
| `USnapInventorySettings` | Snap Inventory System - Inventory | `[/Script/SnapInventorySystem.SnapInventorySettings]` |
| `USnapInventoryUISettings` | Snap Inventory System - Inventory UI | `[/Script/SnapInventorySystem.SnapInventoryUISettings]` |
| `USnapItemsSettings` | Snap Inventory System - Items | `[/Script/SnapInventorySystem.SnapItemsSettings]` |
| `USnapUICoreSettings` | Snap Inventory System - UI Core | `[/Script/SnapInventorySystem.SnapUICoreSettings]` |
| `USnapWorldSettings` | Snap Inventory System - World | `[/Script/SnapInventorySystem.SnapWorldSettings]` |

> Most are read from the class-default object at class load, so a change takes effect on the
> next editor start rather than the next Play session. The few read per call say so.

**380 settings in 67 categories**, and after them the 46 fields of the 11 structs some settings hold.

## Active Effects

| Setting | Class | Type | Default | What it does |
|---|---|---|---|---|
| `ActiveEffectRows` | `USnapFoundationSettings` | `TArray<FSnapActiveEffectRow>` | `*(none)*` | Which running effects the buff row shows, and how. |

## Attachment

| Setting | Class | Type | Default | What it does |
|---|---|---|---|---|
| `ComponentAttachSpecs` | `USnapFoundationSettings` | `TArray<FSnapComponentAttachSpec>` | `*(none)*` | What gets attached where: the plugin's seven components (six without GameplayAbilities) on Lyra's modular base classes. |
| `ContainerRegistry` | `USnapFoundationSettings` | `FSoftObjectPath` | `*(none)*` | The container registry asset that supplies the attach rows. |
| `bAutoAttachComponents` | `USnapFoundationSettings` | `bool` | `true` | Master switch for automatic component attachment. |
| `bWarnWhenNothingAttached` | `USnapFoundationSettings` | `bool` | `true` | Log one warning per session when a player logs in without the components the enabled rows should have given them. |

## Carry Weight

| Setting | Class | Type | Default | What it does |
|---|---|---|---|---|
| `DefaultItemWeightKilograms` | `USnapItemsSettings` | `float` | `1.0f` | Weight in kg of one unit of an item with no Weight fragment (or an unset one), so every item weighs something without re-authoring assets. |
| `bCarryWeightIncludesEquipment` | `USnapInventorySettings` | `bool` | `true` | Whether worn items count toward the carried total. |
| `bEnableCarryWeight` | `USnapInventorySettings` | `bool` | `true` | Master switch for the whole carry-weight feature. |

## Carry Weight|Overweight

| Setting | Class | Type | Default | What it does |
|---|---|---|---|---|
| `OverweightGameplayEffect` | `USnapInventorySettings` | `FSoftClassPath` | `*(none)*` | Optional: a GameplayEffect applied to the owner's ability system while over capacity and removed when back under. |
| `OverweightGameplayEffectLevel` | `USnapInventorySettings` | `float` | `1.0f` | Level the OverweightGameplayEffect is applied at, so one curve-driven effect can scale with how badly overloaded the carrier is. |
| `OverweightProportionalFloor` | `USnapInventorySettings` | `float` | `0.25f` | Proportional only: the slowest the slowdown makes a carrier, the multiplier at OverweightProportionalFullSlowdownAt and beyond. |
| `OverweightProportionalFullSlowdownAt` | `USnapInventorySettings` | `float` | `0.5f` | Proportional only: how far over capacity the slowdown reaches OverweightProportionalFloor, as a fraction of capacity (0.5 is 150 percent). |
| `OverweightSlowdownMode` | `USnapInventorySettings` | `ESnapOverweightSlowdownMode` | `ESnapOverweightSlowdownMode::Flat` | Flat (the default) or proportional: proportional slows a carrier more the further over capacity it is, re-applied each sweep. |
| `OverweightSpeedMultiplier` | `USnapInventorySettings` | `float` | `0.5f` | What MaxWalkSpeed is multiplied by while over capacity: 0.5 halves it, and 1.0 turns the direct route off. |
| `OverweightSpeedScaleFunctionName` | `USnapInventorySettings` | `FName` | `*(none)*` | A function on your movement component that scales the character's speed, for one that ignores MaxWalkSpeed. |
| `OverweightSpeedScaleGetterName` | `USnapInventorySettings` | `FName` | `*(none)*` | The read half of OverweightSpeedScaleFunctionName: a function on the same component that takes nothing and returns the current speed scale. |
| `OverweightSweepInterval` | `USnapInventorySettings` | `float` | `0.5f` | How often, in seconds, the overweight consequence is re-derived. |
| `bEnableOverweightSlowdown` | `USnapInventorySettings` | `bool` | `true` | Slow the character down while over capacity. |

## Carry Weight|Readout

| Setting | Class | Type | Default | What it does |
|---|---|---|---|---|
| `WeightNormalTint` | `USnapInventorySettings` | `FLinearColor` | `FLinearColor(0.92f, 0.92f, 0.96f, 1.f)` | Linear tint the weight readout draws at while under capacity. |
| `WeightOverCapacityTint` | `USnapInventorySettings` | `FLinearColor` | `FLinearColor(0.9050f, 0.0623f, 0.0413f, 1.f)` | Linear tint the weight readout switches to while the carrier is over capacity. |
| `WeightReadoutDecimals` | `USnapInventorySettings` | `int32` | `0` | Decimal places the weight readout draws: 0 by default, so whole numbers. |
| `WeightUnit` | `USnapInventorySettings` | `ESnapWeightUnit` | `ESnapWeightUnit::Icon` | The unit the weight readout uses: kilograms, pounds, or (the default) an icon beside the number. |
| `WeightUnitIconSize` | `USnapInventorySettings` | `float` | `14.f` | The icon's square size in Slate units while WeightUnit is Icon. |
| `WeightUnitIconTexture` | `USnapInventorySettings` | `FSoftObjectPath` | `FSoftObjectPath(TEXT("/SnapInventorySyste...` | The icon drawn beside the number while WeightUnit is Icon; empty draws a bare number. |
| `bShowStorageSlotCount` | `USnapInventorySettings` | `bool` | `true` | Show the pouch window's slot count ("3 / 8"), independently of its weight. |
| `bShowStorageWeightReadouts` | `USnapInventorySettings` | `bool` | `true` | Show a storage item's contents weight on the open container window's "used / total" line. |

## Collect Feed

| Setting | Class | Type | Default | What it does |
|---|---|---|---|---|
| `CollectFeedWidgetClass` | `USnapInventoryUISettings` | `TSoftClassPtr<USnapInventoryCollectFeedWidget>` | `TSoftClassPtr<USnapInventoryCollectFeedWi...` | The widget class for the collect feed. |
| `bEnableCollectFeed` | `USnapInventoryUISettings` | `bool` | `true` | Whether a collect puts a row on the stacking collect feed. |
| `bShowActionsInCollectFeed` | `USnapInventoryUISettings` | `bool` | `true` | Whether actions also put a line on the feed: an item used, learned, mixed, repaired or dismantled ("Drank Health Potion"). |

## Consume

| Setting | Class | Type | Default | What it does |
|---|---|---|---|---|
| `ConsumeAbilityTag` | `USnapInventorySettings` | `FGameplayTag` | `*(none)*` | An ability the consume key activates instead of spending a unit itself. |
| `ConsumeSlots` | `USnapFoundationSettings` | `TArray<FGameplayTag>` | `*(none)*` | Which equipment slots ConsumeKey may spend from, in order: the first occupied one wins. |
| `ItemInHandLingerSeconds` | `USnapInventorySettings` | `float` | `0.3f` | How long the item stays in the hand after a use lands, in seconds, so the hand lowers with it. |
| `ItemUseSeconds` | `USnapInventorySettings` | `TMap<FGameplayTag, float>` | `*(none)*` | How long using an item takes, by item tag; empty (the default) makes every use instant. |
| `bShowItemInHandDuringUse` | `USnapInventorySettings` | `bool` | `true` | While a timed use runs, draw the item in the player's hand (the selectable hotbar's held layer), seen by every player. |
| `bSpendWhenConsumeAbilityEndsWithoutSpending` | `USnapInventorySettings` | `bool` | `false` | Spend the item even when the consume ability declines on its first frame. |

## Context Menu

| Setting | Class | Type | Default | What it does |
|---|---|---|---|---|
| `ContextMenuActions` | `USnapInventoryUISettings` | `TArray<FSnapInventoryContextActionEntry>` | `*(none)*` | The project's own right-click rows, in order: USnapInventoryContextAction subclasses, offered where CanPerform says yes for every selected item. |
| `ContextMenuPolicyClass` | `USnapInventoryUISettings` | `TSoftClassPtr<USnapInventoryContextMenuPolicy>` | `*(none)*` | A USnapInventoryContextMenuPolicy that may remove and reorder the item menu's rows, but never change what a row does. |
| `ContextMenuWidgetClass` | `USnapInventoryUISettings` | `TSoftClassPtr<USnapInventoryContextMenuWidget>` | `TSoftClassPtr<USnapInventoryContextMenuWi...` | Widget class for the right-click menu's popup: the plate the rows are built into, not the rows. |
| `SortMenuWidgetClass` | `USnapInventoryUISettings` | `TSoftClassPtr<USnapInventorySortMenuWidget>` | `TSoftClassPtr<USnapInventorySortMenuWidge...` | Widget class for the sort dropdown's popup, ContextMenuWidgetClass's sibling on the panel's sort button. |

## Counter Readout

| Setting | Class | Type | Default | What it does |
|---|---|---|---|---|
| `CounterIcon` | `USnapEconomySettings` | `FSoftObjectPath` | `*(none)*` | Icon drawn beside the counter number on the inventory screen; empty leaves a bare number. |
| `CounterTag` | `USnapInventorySettings` | `FGameplayTag` | `*(none)*` | Which slotless counter the readout shows. |

## Crafting

| Setting | Class | Type | Default | What it does |
|---|---|---|---|---|
| `CraftingStationNames` | `USnapCraftingSettings` | `TMap<FGameplayTag, FText>` | `*(none)*` | What each crafting station is called where a recipe that needs it is listed away from it ("Made at: Forge"), by station tag. |
| `DefaultRecipeSet` | `USnapCraftingSettings` | `TSoftObjectPtr<USnapCraftingRecipeSet>` | `*(none)*` | The recipe set the plugin's craft surfaces fall back on: the bag's craft window, the item menu and a vendor with no set of its own. |
| `DismantleReturnFraction` | `USnapCraftingSettings` | `float` | `0.5f` | The default fraction of each ingredient a dismantle returns. |
| `MixRuleSet` | `USnapCraftingSettings` | `FSoftObjectPath` | `*(none)*` | The mix rules this project allows: drop one item onto another and, when a row names the pair, both are spent and the row's result is made. |
| `bCraftWhenBagFull` | `USnapCraftingSettings` | `bool` | `true` | A full bag does not stop a craft: a unit that finds no room waits in the queue it was ordered from, marked finished, and moves into the bag once there is room -- at its station or shop while that window is open, or anywhere for the bag's own crafts (with CraftDeliveryMode Immediate, anywhere). |
| `bCraftingConsumesFromPouches` | `USnapCraftingSettings` | `bool` | `true` | Whether a grid shortfall of ingredients may be met from the pouches this actor holds, nested ones too. |
| `bCraftingConsumesFromSiblingContainers` | `USnapInventorySettings` | `bool` | `false` | Whether crafting also counts and spends ingredients in the owner's other inventory components, such as a shard bag. |
| `bCraftingStationOpensInventory` | `USnapCraftingSettings` | `bool` | `true` | Whether a crafting station opens the bag with its craft window, like a shop. |
| `bEnableDismantle` | `USnapCraftingSettings` | `bool` | `true` | Master switch for dismantling items. |
| `bEnableInventoryCrafting` | `USnapCraftingSettings` | `bool` | `true` | Master switch for recipes with no RequiredStationTag, craftable anywhere. |
| `bEnableVendorCrafting` | `USnapCraftingSettings` | `bool` | `true` | Master switch for recipes with a RequiredStationTag, craftable only where that tag is supplied (an open station or shop that lists it). |
| `bGroupCraftRecipesByCategory` | `USnapCraftingSettings` | `bool` | `true` | Group the recipe list under collapsible category headers, on both crafting surfaces. |

## Crafting|Timed Queue

| Setting | Class | Type | Default | What it does |
|---|---|---|---|---|
| `CraftDeliveryMode` | `USnapCraftingSettings` | `ESnapCraftDeliveryMode` | `ESnapCraftDeliveryMode::AtStation` | Where a finished timed craft goes. |
| `CraftQueueSweepInterval` | `USnapCraftingSettings` | `float` | `1.f` | How often, in seconds, the server checks the craft queue for finished units and retries stalled entries. |
| `CraftWaitingHeightAboveTop` | `USnapCraftingSettings` | `float` | `40.f` | How far above the top of a station its waiting icons float. |
| `CraftWaitingWidgetClass` | `USnapCraftingSettings` | `TSoftClassPtr<UUserWidget>` | `TSoftClassPtr<UUserWidget>(FSoftObjectPat...` | The row of icons a station draws over itself while finished crafts wait there for the local player. |
| `MaxCraftQueueLength` | `USnapCraftingSettings` | `int32` | `5` | How many rows one queue may hold at once: each station's, crafting shop's and the bag's own with bCraftQueuePerStation on, the player's whole queue with it off. |
| `bCraftQueuePerStation` | `USnapCraftingSettings` | `bool` | `true` | Each crafting station, each crafting shop and the bag's own craft window keeps its own queue: it shows only the crafts ordered there, crafts them on its own clock (bSequentialCraftQueue) and holds MaxCraftQueueLength rows of its own. |
| `bEnableTimedCrafting` | `USnapCraftingSettings` | `bool` | `true` | Master switch for timed recipes (CraftDurationSeconds above 0). |
| `bSequentialCraftQueue` | `USnapCraftingSettings` | `bool` | `true` | One craft at a time: only the first entry of a queue counts down, and the next starts when it finishes its whole batch. |
| `bShowCraftProgressReadout` | `USnapCraftingSettings` | `bool` | `false` | Show the selected recipe's progress readout (bar, state line and Cancel) under the quantity stepper. |

## Currencies

| Setting | Class | Type | Default | What it does |
|---|---|---|---|---|
| `Currencies` | `USnapInventorySettings` | `TArray<FSnapCurrencyDefinition>` | `*(none)*` | The currencies this project has, each with its own balance, icon and worth in base units. |
| `CurrencyRowEntryHeight` | `USnapEconomySettings` | `float` | `24.f` | Height of one currency entry in the row, in slate units. |
| `CurrencyRowIconSize` | `USnapEconomySettings` | `float` | `20.f` | Side length of a currency icon, in slate units. |
| `CurrencyRowIconSpacing` | `USnapEconomySettings` | `float` | `4.f` | Gap between the icon and its number, in slate units. |
| `CurrencyRowSpacing` | `USnapEconomySettings` | `float` | `12.f` | Gap between two currency entries, in Slate units, on both axes, so it is also the gap between wrapped rows. |
| `CurrencyRowTextColor` | `USnapEconomySettings` | `FLinearColor` | `FLinearColor(0.92f, 0.92f, 0.96f, 1.f)` | Color of the numbers in the currency row. |
| `CurrencyRowTopPadding` | `USnapEconomySettings` | `float` | `4.f` | Space above the currency row, in slate units. |

## Durability

| Setting | Class | Type | Default | What it does |
|---|---|---|---|---|
| `DurabilityBreakBehavior` | `USnapItemsSettings` | `ESnapDurabilityBreakBehavior` | `ESnapDurabilityBreakBehavior::BreakInPlace` | What happens when an item's durability reaches 0, project-wide (ESnapDurabilityBreakBehavior). |
| `DurabilityRarityRankBonus` | `USnapItemsSettings` | `float` | `0.25f` | How much each rarity rank raises an item's effective max durability, before wear: round(MaxDurability x (1 + Rank x this)). |
| `bEnableDurability` | `USnapItemsSettings` | `bool` | `true` | Master switch for the whole durability feature. |

## Durability|Appearance

| Setting | Class | Type | Default | What it does |
|---|---|---|---|---|
| `BrokenDimTint` | `USnapItemsSettings` | `FLinearColor` | `FLinearColor(0.25f, 0.25f, 0.25f, 1.f)` | Linear tint multiplied onto a broken item's icon (grid, storage, and equipment slots alike). |
| `DurabilityBrokenTextTint` | `USnapItemsSettings` | `FLinearColor` | `FLinearColor(0.95f, 0.03f, 0.03f, 1.f)` | The tint a durability readout draws a broken item's number in. |
| `DurabilityCautionFraction` | `USnapItemsSettings` | `float` | `0.5f` | Below this fraction (and above DurabilityWarningFraction) the tooltip's durability line uses DurabilityCautionTint, a middle tier like spoilage's. |
| `DurabilityCautionTint` | `USnapItemsSettings` | `FLinearColor` | `FLinearColor(1.f, 0.5327f, 0.0025f, 1.f)` | Ink color of the tooltip's durability line between DurabilityCautionFraction and DurabilityWarningFraction. |
| `DurabilityWarningFraction` | `USnapItemsSettings` | `float` | `0.25f` | Below this fraction of effective max durability, the item tooltip's durability line is drawn in DurabilityWarningTint. |
| `DurabilityWarningTint` | `USnapItemsSettings` | `FLinearColor` | `FLinearColor(0.95f, 0.35f, 0.1f, 1.f)` | Ink color of the item tooltip's durability line below DurabilityWarningFraction. |

## Durability|Damage Wear

| Setting | Class | Type | Default | What it does |
|---|---|---|---|---|
| `DamageWearBoneToSlot` | `USnapEquipmentSettings` | `TMap<FName, FGameplayTag>` | `*(none)*` | Bone name to equipment slot tag, for HitPiece; an entry here wins over the built-in mannequin table, matched exactly. |
| `DamageWearHealthAttributeName` | `USnapItemsSettings` | `FName` | `FName("Health")` | The attribute whose fall means the owner was hurt, by name. |
| `DamageWearMultiplierBySlot` | `USnapItemsSettings` | `TMap<FGameplayTag, float>` | `*(none)*` | Wear scaling per armor type: equipment slot tag to multiplier. |
| `DamageWearPointsPerHealthPoint` | `USnapItemsSettings` | `float` | `0.1f` | Durability points lost per point of the attribute lost, before the slot and rarity multipliers. |
| `DamageWearTarget` | `USnapItemsSettings` | `ESnapDamageWearTarget` | `ESnapDamageWearTarget::AllEquipped` | Which worn pieces a hit wears down. |
| `bDamageWearFallsBackToAllEquipped` | `USnapItemsSettings` | `bool` | `true` | In HitPiece mode, what to do when the hit piece can't be identified (no hit result, an unmapped bone, an empty slot). |
| `bEnableDamageDurabilityWear` | `USnapItemsSettings` | `bool` | `true` | Master switch for damage-driven wear: taking a hit wears the armor worn. |
| `bUseDefaultMannequinBoneMap` | `USnapEquipmentSettings` | `bool` | `true` | Fall back to a built-in UE5 mannequin bone map for any bone DamageWearBoneToSlot doesn't name. |

## Durability|Debug

| Setting | Class | Type | Default | What it does |
|---|---|---|---|---|
| `DurabilityDebugDamageAmount` | `USnapItemsSettings` | `int32` | `10` | Points DurabilityDebugKey subtracts per press. |
| `DurabilityDebugKey` | `USnapItemsSettings` | `FKey` | `EKeys::F6` | The key bBindDurabilityDebugKeyRaw binds. |
| `bBindDurabilityDebugKeyRaw` | `USnapItemsSettings` | `bool` | `false` | Development aid: a raw key (DurabilityDebugKey) that damages an item, to test breaking. |

## Durability|Repair

| Setting | Class | Type | Default | What it does |
|---|---|---|---|---|
| `DurabilityRepairCostPerPoint` | `USnapItemsSettings` | `int32` | `1` | Currency cost per missing durability point, for a repair at a vendor. |
| `FieldRepairMaterialItem` | `USnapItemsSettings` | `FSoftObjectPath` | `*(none)*` | The item a field repair (away from a vendor) is paid in, taken from the bag at FieldRepairMaterialPerPoint; at a vendor, repair costs currency instead. |
| `FieldRepairMaterialPerPoint` | `USnapItemsSettings` | `float` | `0.1f` | Units of FieldRepairMaterialItem per missing durability point, for a field repair. |
| `RepairMaxWearFraction` | `USnapItemsSettings` | `float` | `0.05f` | Repair wear: every repair (vendor or kit) permanently lowers an item's effective max durability by this fraction of its authored MaxDurability, never below 1. |
| `bFieldRepairDrawsFromPouches` | `USnapItemsSettings` | `bool` | `true` | Whether a field repair may take its material from carried pouches (nested ones too), after the grid. |

## Durability|Sell Price

| Setting | Class | Type | Default | What it does |
|---|---|---|---|---|
| `DurabilitySellValueFloorFraction` | `USnapItemsSettings` | `float` | `0.25f` | What fraction of its full-durability sell price a fully broken item still fetches: 0.25 is a quarter. |
| `bDurabilityAffectsSellPrice` | `USnapItemsSettings` | `bool` | `true` | Scale a damaged item's sell price by its durability, as bSpoilageAffectsSellPrice does by freshness; the factors multiply. |

## Equipment

| Setting | Class | Type | Default | What it does |
|---|---|---|---|---|
| `EquipmentSlotSet` | `USnapEquipmentSettings` | `FSoftObjectPath` | `*(none)*` | The asset that says which equipment slots this project has. |
| `bEquipIntoBlockedSlotUnequipsBlocker` | `USnapEquipmentSettings` | `bool` | `true` | Equipping into a slot a worn item blocks takes that item off. |

## Equipment|Requirements

| Setting | Class | Type | Default | What it does |
|---|---|---|---|---|
| `RequirementLevelAttributeName` | `USnapItemsSettings` | `FName` | `*(none)*` | The attribute an item's MinLevel requirement is compared against, by property name (for example "PowerLevel"). |

## Equipment|Stats

| Setting | Class | Type | Default | What it does |
|---|---|---|---|---|
| `StatEffectMap` | `USnapItemsSettings` | `FSoftObjectPath` | `FSoftObjectPath(TEXT("/SnapInventorySyste...` | With GameplayAbilities: which GameplayEffect moves each stat an item declares (a USnapStatEffectMap); equipping applies it with the row's amount. |

## Examples

| Setting | Class | Type | Default | What it does |
|---|---|---|---|---|
| `DemoCharacterAnimClass` | `USnapInventoryUISettings` | `TSoftClassPtr<UAnimInstance>` | `*(none)*` | Animation Blueprint run on DemoCharacterMesh. |
| `DemoCharacterMesh` | `USnapInventoryUISettings` | `TSoftObjectPtr<USkeletalMesh>` | `*(none)*` | The skeletal mesh the demo character wears; point it at your own mannequin. |
| `DemoCharacterMeshOffset` | `USnapInventoryDemoSettings` | `FVector` | `FVector(0.0, 0.0, -96.0)` | Where the mesh sits relative to the character's capsule. |
| `DemoCharacterMeshRotation` | `USnapInventoryDemoSettings` | `FRotator` | `FRotator(0.0, -90.0, 0.0)` | Yaw applied to the demo character's mesh: -90 turns a rig authored facing +Y toward the capsule's forward. |
| `bEnableExampleItemSpawner` | `USnapWorldSettings` | `bool` | `false` | Development aid that fills a map with one of every item, on a key, to try pickup, equipping and stacking without placing actors. |

## Floating Windows

| Setting | Class | Type | Default | What it does |
|---|---|---|---|---|
| `WindowChromeForegroundColor` | `USnapUICoreSettings` | `FLinearColor` | `FLinearColor(0.86f, 0.86f, 0.88f, 1.f)` | Color of the close glyph at rest. |
| `WindowDragThreshold` | `USnapUICoreSettings` | `float` | `5.f` | How far, in screen pixels, the cursor must move before a press on a window becomes a window drag, so clicks on the window still work. |
| `WindowMinVisiblePixels` | `USnapUICoreSettings` | `float` | `48.f` | How much of a dragged window must stay inside the viewport, in screen pixels, so it can always be grabbed again. |
| `WindowOpaqueBackgroundAlpha` | `USnapUICoreSettings` | `float` | `1.f` | Minimum alpha forced onto each window's background while floating windows are on, so a dragged window hides what's under it. |
| `bAllowWindowDragAndClose` | `USnapUICoreSettings` | `bool` | `false` | Let each of the bag screen's panels be dragged and closed on its own. |

## Floating Windows|Close Button

| Setting | Class | Type | Default | What it does |
|---|---|---|---|---|
| `WindowCloseButtonColor` | `USnapUICoreSettings` | `FLinearColor` | `FLinearColor(0.f, 0.f, 0.f, 0.35f)` | Fill behind the close glyph at rest. |
| `WindowCloseButtonHoveredColor` | `USnapUICoreSettings` | `FLinearColor` | `FLinearColor(0.72f, 0.12f, 0.12f, 0.92f)` | Fill behind the close glyph while the cursor is over it. |
| `WindowCloseButtonInset` | `USnapUICoreSettings` | `float` | `6.f` | How far the close box sits in from the window's top-right corner, in slate units. |
| `WindowCloseButtonPressedColor` | `USnapUICoreSettings` | `FLinearColor` | `FLinearColor(0.92f, 0.2f, 0.2f, 1.f)` | Fill behind the close glyph while it is held down. |
| `WindowCloseButtonSize` | `USnapUICoreSettings` | `float` | `20.f` | Side length of the square close box in each window's top-right corner, in Slate units. |
| `WindowCloseGlyphHoveredColor` | `USnapUICoreSettings` | `FLinearColor` | `FLinearColor(0.98f, 0.35f, 0.35f, 1.f)` | Color of the close glyph while the cursor is over it: red by convention. |
| `WindowCloseGlyphPressedColor` | `USnapUICoreSettings` | `FLinearColor` | `FLinearColor(1.f, 0.62f, 0.62f, 1.f)` | Color of the close glyph while it is held down. |
| `bWindowCloseButtonDrawsPlate` | `USnapUICoreSettings` | `bool` | `false` | Draw a filled plate behind the close glyph. |

## Gameplay Integration

| Setting | Class | Type | Default | What it does |
|---|---|---|---|---|
| `AbilitiesUsableWhileScreenOpen` | `USnapFoundationSettings` | `TArray<FSnapScreenOpenAbilityBinding>` | `*(none)*` | Abilities that stay reachable by key while a plugin screen is open. |
| `AbilityInputBlockedTagName` | `USnapFoundationSettings` | `FName` | `*(none)*` | The loose gameplay tag put on the player's ability system while a screen is open, to block ability input (Lyra's is Gameplay.AbilityInputBlocked). |
| `DeathTagName` | `USnapFoundationSettings` | `FName` | `*(none)*` | The gameplay tag that means the player is dead: while it is on their ability system the inventory screen is closed and can't open. |

## HUD Mirror

| Setting | Class | Type | Default | What it does |
|---|---|---|---|---|
| `ContainerRefreshBindings` | `USnapInventoryUISettings` | `TArray<FSnapInventoryHUDContainerRefreshBinding>` | `*(none)*` | When a container's contents change, call a function on your own HUD widget so it rebuilds itself (FSnapInventoryHUDContainerRefreshBinding). |
| `EquipmentHUDBindings` | `USnapFoundationSettings` | `TArray<FSnapInventoryHUDEquipmentBinding>` | `*(none)*` | Which equipment slots are mirrored onto your always-on HUD, and into which widgets (FSnapInventoryHUDEquipmentBinding). |
| `HUDRarityOpacity` | `USnapInventoryUISettings` | `float` | `0.15f` | How strongly the rarity color tints a HUD quality plate (RarityColorWidgetName). |

## Hotbar|Appearance

| Setting | Class | Type | Default | What it does |
|---|---|---|---|---|
| `HotbarEmptySquareGhostTint` | `USnapInventoryUISettings` | `FLinearColor` | `FLinearColor(1.0f, 1.0f, 1.0f, 0.22f)` | The tint an empty square's ghost picture is drawn with (HotbarEmptySquareGhosts). |
| `HotbarEmptySquareGhosts` | `USnapInventoryUISettings` | `TMap<FGameplayTag, FSoftObjectPath>` | `*(none)*` | A ghost picture for an empty hotbar square, per bar tag: a potion outline on the consumable bar, for example. |
| `HotbarPanelEmptySquareGhostTint` | `USnapInventoryUISettings` | `FLinearColor` | `FLinearColor(1.0f, 1.0f, 1.0f, 0.22f)` | The tint for HotbarPanelEmptySquareGhosts' pictures on the equipment panel; a ghost taken from HotbarEmptySquareGhosts keeps that map's tint. |
| `HotbarPanelEmptySquareGhosts` | `USnapInventoryUISettings` | `TMap<FGameplayTag, FSoftObjectPath>` | `*(none)*` | The equipment panel's own ghost pictures, per bar tag, so the panel and the HUD can differ. |
| `HotbarPanelSlotBackgroundColor` | `USnapInventoryUISettings` | `FLinearColor` | `FLinearColor(0.0369f, 0.0451f, 0.0578f, 0...` | Background color for a hotbar square on the equipment panel: lighter than HotbarSlotBackgroundColor, which would vanish against the panel's dark frame. |
| `HotbarSlotBackgroundColor` | `USnapInventoryUISettings` | `FLinearColor` | `FLinearColor(0.0070f, 0.0087f, 0.0125f, 0...` | The color of the translucent square itself. |
| `HotbarSlotCornerRadius` | `USnapInventoryUISettings` | `float` | `3.0f` | Corner radius for a hotbar square's rounded frame, in Slate units. |
| `HotbarSlotCountFontSize` | `USnapInventoryUISettings` | `int32` | `12` | Font size for the stack count in a square's bottom-right corner. |
| `HotbarSlotHoverColor` | `USnapInventoryUISettings` | `FLinearColor` | `FLinearColor(0.077f, 0.077f, 0.08f, 0.75f)` | The square's color while a drag this bar would accept hovers over it, and, on a square with no hover part of its own, while the cursor is over it. |
| `HotbarSlotIconPadding` | `USnapInventoryUISettings` | `float` | `0.0f` | Inset between a square's frame and its item icon, in Slate units. |
| `HotbarSlotKeyColor` | `USnapInventoryUISettings` | `FLinearColor` | `FLinearColor(0.8f, 0.8f, 0.82f, 1.f)` | The color of the key glyph in a square's top-left corner. |
| `HotbarSlotKeyFontSize` | `USnapInventoryUISettings` | `int32` | `10` | Font size for the key glyph, in points at 96 DPI: size 10 is about 16 Slate units tall, so change it with the square size. |
| `HotbarSlotLockedMarker` | `USnapInventoryUISettings` | `FString` | `TEXT("*")` | Text drawn in the lower-left corner of a hotbar square that can't be cleared or moved (abilities-only mode). |
| `HotbarSlotModifierMarker` | `USnapInventoryUISettings` | `FString` | `TEXT("^")` | Text drawn beside a square's key when that key needs a modifier held: a symbol, since "Shift+3" doesn't fit. |
| `HotbarSlotOutlineColor` | `USnapInventoryUISettings` | `FLinearColor` | `FLinearColor(0.27f, 0.27f, 0.28f, 0.85f)` | Outline color around every hotbar square, on both surfaces; it keeps edge-to-edge icons apart. |
| `HotbarSlotOutlineWidth` | `USnapInventoryUISettings` | `float` | `1.5f` | Outline thickness for a hotbar square, in Slate units. |
| `HotbarSlotSize` | `USnapInventoryUISettings` | `float` | `64.0f` | Width and height of one hotbar square, in Slate units. |
| `HotbarSlotSpacing` | `USnapInventoryUISettings` | `float` | `4.0f` | Gap between neighboring squares, in Slate units: the space between them, never an inset from the row's edge. |
| `bShowEquipmentPanelHotbarTitles` | `USnapInventoryUISettings` | `bool` | `false` | Show the titles over the hotbar and potion bar on the equipment panel. |

## Hotbar|Bars

| Setting | Class | Type | Default | What it does |
|---|---|---|---|---|
| `HotbarAbilityAllowList` | `USnapEquipmentSettings` | `TArray<TSoftObjectPtr<class USnapItemDefinition>>` | `*(none)*` | The item definitions that fill the ability bar in abilities-only mode, in slot order: element 0 lands on slot 0. |
| `HotbarAbilitySlotCount` | `USnapEquipmentSettings` | `int32` | `10` | How many slots the ability bar has. |
| `HotbarBarSpecs` | `USnapEquipmentSettings` | `TArray<FSnapHotbarBarSpec>` | `*(none)*` | Every hotbar bar as data (FSnapHotbarBarSpec), in draw and focus order. |
| `HotbarConsumableSlotCount` | `USnapEquipmentSettings` | `int32` | `2` | How many slots the consumable bar has. |
| `HotbarPanelAbilitySlotCount` | `USnapEquipmentSettings` | `int32` | `10` | How many ability-bar squares the equipment panel draws, when fewer than HotbarAbilitySlotCount; anything higher shows them all. |
| `HotbarPanelConsumableSlotCount` | `USnapEquipmentSettings` | `int32` | `2` | As HotbarPanelAbilitySlotCount, for the consumable bar drawn on the equipment panel. |
| `bAutoBindConsumablesToHotbar` | `USnapInventorySettings` | `bool` | `false` | Let consumables fill the consumable bar by themselves. |
| `bHotbarAbilitiesOnlyMode` | `USnapEquipmentSettings` | `bool` | `false` | Abilities-only mode: the ability bar is filled from HotbarAbilityAllowList, and its squares can't be cleared, only reordered by swapping. |
| `bHotbarAbilityBarAcceptsAbilitiesOnly` | `USnapEquipmentSettings` | `bool` | `false` | Restrict the ability bar to items whose Hotbar fragment declares Kind = Ability. |
| `bShiftClickSwapsFirstConsumableSquare` | `USnapEquipmentSettings` | `bool` | `true` | With bAutoBindConsumablesToHotbar on and every consumable square taken, Shift+click puts the item in the first square instead of refusing. |
| `bShowConsumableBarPageIndicator` | `USnapEquipmentSettings` | `bool` | `false` | Draw the page indicator ("2/3") on the built-in consumable bar too: one stored page holds every bar, so paging changes both. |

## Hotbar|Input

| Setting | Class | Type | Default | What it does |
|---|---|---|---|---|
| `GlyphDevicePreference` | `USnapUICoreSettings` | `ESnapGlyphDevicePreference` | `ESnapGlyphDevicePreference::Auto` | Which device's key every on-screen input glyph names. |
| `HotbarConsumableInputActions` | `USnapEquipmentSettings` | `TArray<TSoftObjectPtr<UInputAction>>` | `*(none)*` | The input actions that fire consumable-bar slots, in slot order. |
| `HotbarMappingContext` | `USnapEquipmentSettings` | `TSoftObjectPtr<UInputMappingContext>` | `*(none)*` | The mapping context with the shipped 1-9, 0 hotbar layout (IMC_SnapHotbar). |
| `HotbarMappingContextPriority` | `USnapEquipmentSettings` | `int32` | `0` | Priority for HotbarMappingContext. |
| `HotbarSlotInputActions` | `USnapEquipmentSettings` | `TArray<TSoftObjectPtr<UInputAction>>` | `*(none)*` | The input actions that fire ability-bar slots, in slot order: element 0 fires slot 0. |
| `bAddHotbarMappingContext` | `USnapEquipmentSettings` | `bool` | `true` | Add HotbarMappingContext to the local player and register it with Enhanced Input's user settings, so the ten hotbar keys work and can be rebound. |
| `bBagScreenHotbarClickPicks` | `USnapEquipmentSettings` | `bool` | `true` | A click on the bag screen's hotbar picks the square, and a click on another square of that bar moves it there (a swap, or a move into an empty one). |
| `bHotbarKeyAssignsHoveredItem` | `USnapEquipmentSettings` | `bool` | `true` | With the inventory screen open, a hotbar key binds the item under the cursor to its square instead of activating it. |

## Hotbar|Pages

| Setting | Class | Type | Default | What it does |
|---|---|---|---|---|
| `HotbarPageCount` | `USnapEquipmentSettings` | `int32` | `3` | How many stored hotbars (pages) the player cycles between. |
| `HotbarPageModifierKey` | `USnapEquipmentSettings` | `FKey` | `EKeys::LeftShift` | The modifier that must be held for HotbarPageNextKey and HotbarPagePreviousKey to page. |
| `HotbarPageModifierKeyAlt` | `USnapEquipmentSettings` | `FKey` | `EKeys::RightShift` | Also accept this second key as the page modifier, so left/right pairs both work. |
| `HotbarPageNextInputAction` | `USnapEquipmentSettings` | `TSoftObjectPtr<UInputAction>` | `*(none)*` | Enhanced Input action that steps to the next stored hotbar: the rebindable route to HotbarPageNextKey. |
| `HotbarPageNextKey` | `USnapEquipmentSettings` | `FKey` | `EKeys::MouseScrollUp` | Key that steps to the next stored hotbar while HotbarPageModifierKey is held. |
| `HotbarPagePrevInputAction` | `USnapEquipmentSettings` | `TSoftObjectPtr<UInputAction>` | `*(none)*` | Enhanced Input action that steps to the previous stored hotbar. |
| `HotbarPagePreviousKey` | `USnapEquipmentSettings` | `FKey` | `EKeys::MouseScrollDown` | Key that steps to the previous stored hotbar while HotbarPageModifierKey is held. |
| `bBindHotbarPageKeysRaw` | `USnapEquipmentSettings` | `bool` | `false` | Bind the page keys as raw key bindings. |
| `bPersistHotbarPages` | `USnapEquipmentSettings` | `bool` | `true` | Save the player's stored hotbars per local profile, on the owning client. |
| `bRequireHotbarPageModifier` | `USnapEquipmentSettings` | `bool` | `true` | Whether the raw page keys need a modifier held. |

## Hotbar|Placement

| Setting | Class | Type | Default | What it does |
|---|---|---|---|---|
| `HotbarHUDPlacements` | `USnapEquipmentSettings` | `TArray<FSnapHotbarHUDPlacement>` | `*(none)*` | Where each bar is placed on the game HUD. |
| `HotbarSlotWidgetClass` | `USnapEquipmentSettings` | `TSoftClassPtr<USnapInventoryHotbarSlotWidget>` | `TSoftClassPtr<USnapInventoryHotbarSlotWid...` | Widget class for each hotbar square. |
| `HotbarSupersededWidgets` | `USnapEquipmentSettings` | `TArray<FSnapHotbarSupersededWidget>` | `*(none)*` | Widgets a placed hotbar replaces that live in a different host from its anchor (FSnapHotbarSupersededWidget). |
| `HotbarWidgetClass` | `USnapEquipmentSettings` | `FSoftClassPath` | `FSoftClassPath(TEXT("/SnapInventorySystem...` | Widget class for every hotbar bar, on the HUD and the equipment panel alike. |
| `bHotbarReplacesConsumableEquipmentSlots` | `USnapEquipmentSettings` | `bool` | `true` | On the equipment panel, draw the consumable hotbar in place of the slots listed in ConsumeSlots. |

## Hotbar|Selectable

| Setting | Class | Type | Default | What it does |
|---|---|---|---|---|
| `HeldItemAttachOffset` | `USnapEquipmentSettings` | `FTransform` | `FTransform(FQuat::Identity, FVector(-8.0,...` | Where a held world model's center sits in that socket's space, and its rotation and scale. |
| `HeldItemMaxSize` | `USnapEquipmentSettings` | `float` | `18.0f` | The longest a held world model is drawn, in centimeters; a longer model is shrunk to fit. |
| `HeldItemSlot` | `USnapEquipmentSettings` | `FGameplayTag` | `*(none)*` | The equipment slot a held item is drawn in; the worn look there is hidden while something is held. |
| `HeldItemSocketName` | `USnapEquipmentSettings` | `FName` | `TEXT("hand_r")` | The socket (or bone) where a held item with no worn look (a potion, a scroll) is drawn, as its world model. |
| `SelectableHotbarBars` | `USnapEquipmentSettings` | `FGameplayTagContainer` | `*(none)*` | Which bars select rather than use while bSelectableHotbar is on. |
| `SelectedHotbarSquareRise` | `USnapEquipmentSettings` | `float` | `6.0f` | How far the selected square rises, in Slate units, easing over ItemHoverGrowSeconds and ItemHoverShrinkSeconds. |
| `bSelectableHotbar` | `USnapEquipmentSettings` | `bool` | `false` | Make a hotbar key select its square and hold its item in the hand, instead of using it at once. |

## Input|Enhanced Input

| Setting | Class | Type | Default | What it does |
|---|---|---|---|---|
| `ConsumeInputAction` | `USnapUICoreSettings` | `TSoftObjectPtr<UInputAction>` | `*(none)*` | Enhanced Input action that uses one unit from the first occupied ConsumeSlots slot: the rebindable route to ConsumeKey. |
| `ContextMenuInputAction` | `USnapUICoreSettings` | `TSoftObjectPtr<UInputAction>` | `*(none)*` | Opens the item menu on the focused slot. |
| `DropInputAction` | `USnapUICoreSettings` | `TSoftObjectPtr<UInputAction>` | `*(none)*` | Enhanced Input action that raises the drop-all confirmation on the open screen. |
| `FilterNextInputAction` | `USnapUICoreSettings` | `TSoftObjectPtr<UInputAction>` | `*(none)*` | Moves to the bag screen's next filter tab. |
| `FilterPrevInputAction` | `USnapUICoreSettings` | `TSoftObjectPtr<UInputAction>` | `*(none)*` | Moves to the bag screen's previous filter tab. |
| `InventoryMappingContext` | `USnapUICoreSettings` | `TSoftObjectPtr<UInputMappingContext>` | `*(none)*` | The input mapping context the plugin applies for its own (non-hotbar) actions. |
| `InventoryMappingContextPriority` | `USnapUICoreSettings` | `int32` | `0` | Priority for InventoryMappingContext: low on purpose, so a project's own contexts win. |
| `QuickMoveInputAction` | `USnapUICoreSettings` | `TSoftObjectPtr<UInputAction>` | `*(none)*` | Quick move: the gamepad version of a Shift+click transfer, on the focused slot. |
| `RotateInputAction` | `USnapUICoreSettings` | `TSoftObjectPtr<UInputAction>` | `*(none)*` | Turns the item being dragged over a spatial grid a quarter turn clockwise. |
| `SectionNextInputAction` | `USnapUICoreSettings` | `TSoftObjectPtr<UInputAction>` | `*(none)*` | Moves to the bag screen's next section. |
| `SectionPrevInputAction` | `USnapUICoreSettings` | `TSoftObjectPtr<UInputAction>` | `*(none)*` | Moves to the bag screen's previous section. |
| `SortInputAction` | `USnapUICoreSettings` | `TSoftObjectPtr<UInputAction>` | `*(none)*` | Enhanced Input action that re-sorts every grid on an open screen. |
| `SplitInputAction` | `USnapUICoreSettings` | `TSoftObjectPtr<UInputAction>` | `*(none)*` | Enhanced Input action that splits the focused stack. |
| `TakeAllInputAction` | `USnapUICoreSettings` | `TSoftObjectPtr<UInputAction>` | `*(none)*` | Enhanced Input action for Take All on an open loot window. |
| `ToggleInputAction` | `USnapUICoreSettings` | `TSoftObjectPtr<UInputAction>` | `*(none)*` | Enhanced Input action that toggles the screen: the rebindable route, and the one to prefer. |
| `bAddInventoryMappingContext` | `USnapUICoreSettings` | `bool` | `true` | Add InventoryMappingContext to the local player and register it with Enhanced Input's user settings, so the plugin's keys appear in a rebinding screen. |

## Input|Focus

| Setting | Class | Type | Default | What it does |
|---|---|---|---|---|
| `FocusEdgeRule` | `USnapUICoreSettings` | `ESnapFocusEdgeRule` | `ESnapFocusEdgeRule::HandOff` | What a directional move does at the edge of a grid. |
| `GamepadAutoFocusPolicy` | `USnapUICoreSettings` | `ESnapGamepadFocusPolicy` | `ESnapGamepadFocusPolicy::WhenGamepadConne...` | Whether opening a screen puts focus on a slot by itself, which draws its focus glow. |
| `bConfirmPadItemLoss` | `USnapUICoreSettings` | `bool` | `true` | On a gamepad, ask before a one-press loss: a pad press that drops, sells, destroys or dismantles items opens a confirmation with focus on its decline button. |
| `bEnableFocusNavigation` | `USnapUICoreSettings` | `bool` | `true` | Master switch for gamepad and keyboard focus navigation: every slot becomes a focus target, and the arrow keys, D-pad and left stick move between them. |
| `bEnableGamepadCarry` | `USnapUICoreSettings` | `bool` | `true` | Gamepad carry: Accept on a focused slot picks its item up and a second Accept puts it down, since a pad can't drag. |

## Input|Gestures

| Setting | Class | Type | Default | What it does |
|---|---|---|---|---|
| `GestureBindings` | `USnapUICoreSettings` | `FSnapGestureBindings` | `*(none)*` | Every mouse and key gesture the plugin's UI understands, in one table read by USnapGestureLibrary. |
| `bConsumeOnDoubleClick` | `USnapUICoreSettings` | `bool` | `true` | Double-clicking a consumable in the bag or a pouch window uses it, exactly as the item menu's Consume does. |

## Input|Glyphs

| Setting | Class | Type | Default | What it does |
|---|---|---|---|---|
| `GamepadGlyphFamily` | `USnapUICoreSettings` | `ESnapGamepadGlyphFamily` | `ESnapGamepadGlyphFamily::Auto` | Which gamepad a pad key is drawn for: its picture and short name ("A" or "Cross"). |
| `GamepadGlyphs` | `USnapUICoreSettings` | `FSoftObjectPath` | `*(none)*` | The pictures pad keys are drawn with, both families in one USnapGamepadGlyphSet. |
| `PlayStationHardwareIdentifiers` | `USnapUICoreSettings` | `TArray<FName>` | `*(none)*` | The hardware names that mean a PlayStation pad under GamepadGlyphFamily Auto, compared without case. |

## Input|Keys

| Setting | Class | Type | Default | What it does |
|---|---|---|---|---|
| `AdditionalToggleKeys` | `USnapUICoreSettings` | `TArray<FKey>` | `{ EKeys::Tab }` | More keys that open and close the screen, like ToggleKey. |
| `ConsumeKey` | `USnapUICoreSettings` | `FKey` | `EKeys::U` | Key that uses one unit of the first occupied slot in ConsumeSlots. |
| `CursorPeekKey` | `USnapUICoreSettings` | `FKey` | `EKeys::MiddleMouseButton` | Key that, held during play, shows the cursor so the HUD can be pointed at (a hotbar square's card, a buff's time left). |
| `GamepadOpenKeys` | `USnapUICoreSettings` | `TArray<FKey>` | `{ EKeys::Gamepad_FaceButton_Top }` | Gamepad buttons that open the screen during play. |
| `ToggleKey` | `USnapUICoreSettings` | `FKey` | `EKeys::I` | The key that toggles the inventory screen. |
| `bBindConsumeKeyRaw` | `USnapUICoreSettings` | `bool` | `false` | Whether to bind ConsumeKey as a raw key binding. |
| `bBindToggleKeyRaw` | `USnapUICoreSettings` | `bool` | `false` | Whether to bind ToggleKey as a raw key binding. |

## Input|Screen

| Setting | Class | Type | Default | What it does |
|---|---|---|---|---|
| `HeldItemPointer` | `USnapUICoreSettings` | `ESnapHeldItemPointer` | `ESnapHeldItemPointer::Centered` | Where a held item's picture sits against the pointer. |
| `ScreenAbilityBlockMode` | `USnapUICoreSettings` | `ESnapAbilityBlockMode` | `ESnapAbilityBlockMode::LooseTag` | How abilities are stopped while a screen is open (ESnapAbilityBlockMode). |
| `ScreenInputMode` | `USnapUICoreSettings` | `ESnapScreenInputMode` | `ESnapScreenInputMode::GameAndUI` | What opening a full-screen widget does to input. |
| `ScreenMouseLockMode` | `USnapUICoreSettings` | `EMouseLockMode` | `EMouseLockMode::DoNotLock` | Whether the cursor is locked to the viewport while a screen is open. |
| `bShowCursorWhileScreenOpen` | `USnapUICoreSettings` | `bool` | `true` | Whether the mouse cursor is shown while a screen is open. |
| `bSuppressLookInputWhileScreenOpen` | `USnapUICoreSettings` | `bool` | `true` | Whether camera look is suppressed while a screen is open. |

## Interaction

| Setting | Class | Type | Default | What it does |
|---|---|---|---|---|
| `InteractionMaxAngleDegrees` | `USnapInteractionSettings` | `float` | `35.f` | How far off the camera's forward direction an interactable may sit, in degrees, measured from the player's pawn; the smallest angle wins. |
| `InteractionPromptWidgetClass` | `USnapInteractionSettings` | `TSoftClassPtr<UUserWidget>` | `TSoftClassPtr<UUserWidget>(FSoftObjectPat...` | The widget class that draws the prompt over the focused target. |
| `InteractionScanRange` | `USnapInteractionSettings` | `float` | `300.f` | How far, in world units, the detector looks for an interactable. |
| `InteractionScanRate` | `USnapInteractionSettings` | `float` | `0.1f` | Seconds between focus re-evaluations: 0.1, as in Lyra. |
| `InteractionTraceChannel` | `USnapInteractionSettings` | `TEnumAsByte<ECollisionChannel>` | `ECC_Visibility` | The channel of the line-of-sight trace, the last step of choosing a target. |

## Interaction|Input

| Setting | Class | Type | Default | What it does |
|---|---|---|---|---|
| `InteractCycleInputAction` | `USnapInteractionSettings` | `TSoftObjectPtr<UInputAction>` | `*(none)*` | Enhanced Input action that steps the prompt to the focused target's next option ("Shop 1/2" to "Forge 2/2"), once per press. |
| `InteractInputAction` | `USnapInteractionSettings` | `TSoftObjectPtr<UInputAction>` | `*(none)*` | Enhanced Input action that interacts with what the player is looking at: the rebindable route, used by default. |
| `InteractKey` | `USnapInteractionSettings` | `FKey` | `EKeys::E` | The key bBindInteractKeyRaw binds, and the key the prompt names only when that bind is on and InteractInputAction has nothing mapped. |
| `bBindInteractKeyRaw` | `USnapInteractionSettings` | `bool` | `false` | Whether to bind InteractKey as a raw key binding. |

## Interaction|Outline

| Setting | Class | Type | Default | What it does |
|---|---|---|---|---|
| `InteractionOutlineColor` | `USnapInteractionSettings` | `FLinearColor` | `FLinearColor(1.f, 0.78f, 0.32f, 1.f)` | The outline's color, pushed into the material's OutlineColor parameter. |
| `InteractionOutlineMaterial` | `USnapInteractionSettings` | `TSoftObjectPtr<UMaterialInterface>` | `TSoftObjectPtr<UMaterialInterface>(FSoftO...` | The post-process material that turns the stencil write into a visible outline. |
| `InteractionOutlineStencilValue` | `USnapInteractionSettings` | `int32` | `250` | The stencil value written to a focused target's meshes while it is highlighted. |
| `InteractionOutlineThickness` | `USnapInteractionSettings` | `float` | `2.f` | The outline's width in screen pixels, pushed into OutlineThickness, so it doesn't thin with distance. |
| `bInteractionOutlineEnabled` | `USnapInteractionSettings` | `bool` | `true` | Flag the focused interactable's meshes for custom-depth rendering (bRenderCustomDepth and a stencil value), which is what an outline reads. |

## Interaction|Prompt

| Setting | Class | Type | Default | What it does |
|---|---|---|---|---|
| `InteractHoldIndicatorDelaySeconds` | `USnapInteractionSettings` | `float` | `0.2f` | How long the interact key must be held before a hold's meter appears, in seconds, so a normal press never flashes a ring. |
| `InteractHoldMeter` | `USnapInteractionSettings` | `ESnapInteractHoldMeter` | `ESnapInteractHoldMeter::Ring` | How a hold shows itself filling: a ring over the target (the default, InteractHoldWidgetClass) or a thin bar under the prompt's words. |
| `InteractHoldWidgetClass` | `USnapInteractionSettings` | `TSoftClassPtr<UUserWidget>` | `TSoftClassPtr<UUserWidget>(FSoftObjectPat...` | The widget class that draws the hold ring over the target (InteractHoldMeter Ring). |
| `InteractPromptKeyPattern` | `USnapInteractionSettings` | `FText` | `NSLOCTEXT("SnapInventorySystem", "SnapInt...` | How the key and label are put together while bShowInteractKeyInPrompt is on: {Key} is the key's name, {Label} the prompt's words ("(E) Forge"). |
| `InteractTapMaxSeconds` | `USnapInteractionSettings` | `float` | `0.2f` | On an option with both a tap and a hold, only a release within this many seconds counts as the tap. |
| `InteractionPromptScreenMargin` | `USnapInteractionSettings` | `float` | `24.f` | How far inside the top and sides of the player's view a moved prompt stays, in screen units. |
| `bKeepInteractionPromptOnScreen` | `USnapInteractionSettings` | `bool` | `true` | Keeps the prompt on screen over a tall object: when the prompt above it would leave the top of the player's view (looking at a stall's counter, or standing close), it slides down the object toward its middle, as high as fits. |
| `bShowInteractKeyInPrompt` | `USnapInteractionSettings` | `bool` | `true` | Whether the prompt names the key that performs it ("(E) Forge" rather than "Forge"). |

## Item Filters

| Setting | Class | Type | Default | What it does |
|---|---|---|---|---|
| `FilterBarHeight` | `USnapItemsSettings` | `float` | `36.f` | Height of each filter button, in slate units. |
| `FilterBarTopPadding` | `USnapItemsSettings` | `float` | `6.f` | Space between the grid above and the filter bar, in slate units. |
| `FilterButtonHoveredColor` | `USnapItemsSettings` | `FLinearColor` | `FLinearColor(0.92f, 0.92f, 0.96f, 1.f)` | Tint while the cursor is over the button. |
| `FilterButtonNormalColor` | `USnapItemsSettings` | `FLinearColor` | `FLinearColor(0.62f, 0.62f, 0.66f, 1.f)` | Tint of a filter button's icon (or label) at rest. |
| `FilterButtonPlateOpacity` | `USnapItemsSettings` | `float` | `0.18f` | How solid the rounded plate behind a filter button is, as a fraction of its state color. |
| `FilterButtonSelectedColor` | `USnapItemsSettings` | `FLinearColor` | `FLinearColor(1.f, 1.f, 1.f, 1.f)` | Tint of the button whose filter is applied, used for its hovered state too, so the active filter stays distinct. |
| `FilterButtonSpacing` | `USnapItemsSettings` | `float` | `4.f` | Gap between filter buttons, in Slate units. |
| `FilterIconSize` | `USnapItemsSettings` | `float` | `30.f` | Side length of the icon inside a filter button, in Slate units. |
| `ItemFilters` | `USnapItemsSettings` | `TArray<FSnapItemFilter>` | `*(none)*` | The filter buttons drawn below any inventory grid whose panel sets bShowFilterBar; selecting one collapses every slot that doesn't match. |

## Item Marks

| Setting | Class | Type | Default | What it does |
|---|---|---|---|---|
| `bEnableItemMarks` | `USnapItemsSettings` | `bool` | `true` | The item marks feature (Locked, Favorite, Junk) as a whole. |

## Item Motion

| Setting | Class | Type | Default | What it does |
|---|---|---|---|---|
| `ItemHoverGrowSeconds` | `USnapUICoreSettings` | `float` | `0.12f` | How long the grow takes, in seconds (95 % of the way). |
| `ItemHoverScale` | `USnapUICoreSettings` | `float` | `1.08f` | How large an item's picture grows while its square is hovered or focused by keyboard or pad (1.08 is 8% larger); 1 turns it off. |
| `ItemHoverShrinkSeconds` | `USnapUICoreSettings` | `float` | `0.15f` | How long the shrink back takes, in seconds (95 % of the way). |
| `ItemLandingBounceScale` | `USnapUICoreSettings` | `float` | `1.18f` | How large an item's picture pops when the item lands in a square: moved, split, merged into, picked up, bought, looted or mixed. |
| `ItemLandingBounceSeconds` | `USnapUICoreSettings` | `float` | `0.25f` | How long the landing bounce takes, in seconds. |
| `ItemSheenDirection` | `USnapInventoryUISettings` | `ESnapItemSheenDirection` | `ESnapItemSheenDirection::TopLeftToBottomR...` | Which way the sheen crosses a square -- a landing's and a new item's alike. |
| `ItemSheenSeconds` | `USnapUICoreSettings` | `float` | `0.6f` | How long the sheen takes to cross a square, in seconds -- a landing's and a new item's alike. |
| `NewItemGlowOpacity` | `USnapUICoreSettings` | `float` | `0.35f` | How strong a new item's glow is, from 0 to 1, as a fraction of the palette NewItemGlow's own opacity. |
| `NewItemShineTexture` | `USnapUICoreSettings` | `FSoftObjectPath` | `FSoftObjectPath(TEXT("/SnapInventorySyste...` | The soft picture a new item's glow and sweep are drawn with (white; its alpha is the shape). |
| `bMarkNewItems` | `USnapUICoreSettings` | `bool` | `true` | New items shine: an item the player just gained gets one sheen and a faint glow until it is hovered or focused, or the bag closes. |
| `bMarkOnlyCreatedItems` | `USnapUICoreSettings` | `bool` | `true` | Mark only items made for the player as new: crafted, bought, mixed, a reward, received in a trade. |
| `bReduceItemMotion` | `USnapUICoreSettings` | `bool` | `false` | One switch for all of an item square's motion: on, pictures don't grow on hover or focus, don't bounce on landing, and new items don't shine. |
| `bSheenOnItemLanding` | `USnapUICoreSettings` | `bool` | `true` | A sheen crosses a square each time an item lands in it, edge to edge, in ItemSheenDirection over ItemSheenSeconds. |

## Loot

| Setting | Class | Type | Default | What it does |
|---|---|---|---|---|
| `DefaultBodyPresentation` | `USnapInventorySettings` | `ESnapLootPresentation` | `*(none)*` | The same choice for a body left behind on death, separate so chests and corpses can differ. |
| `DefaultLootPresentation` | `USnapInventorySettings` | `ESnapLootPresentation` | `*(none)*` | What happens when a loot container is opened, for any container that hasn't overridden it. |
| `DropAllLootBagClass` | `USnapInventorySettings` | `TSoftClassPtr<ASnapLootableBody>` | `*(none)*` | The actor DropAllLootBagThreshold's bag is. |
| `DropAllLootBagThreshold` | `USnapInventorySettings` | `int32` | `20` | When a player drops more whole stacks than this at once (Drop All, a multi-item drop), they go into one loot bag in front of them instead of scattering. |
| `LootRange` | `USnapInventorySettings` | `float` | `300.f` | How far, in world units, a player may stand from a loot container and still take from it; walking away closes the window. |
| `LootWindowClass` | `USnapWorldSettings` | `TSoftClassPtr<USnapInventoryLootWidget>` | `*(none)*` | The window opened on a loot container. |
| `LootableBodyClass` | `USnapInventorySettings` | `TSoftClassPtr<ASnapLootableBody>` | `*(none)*` | The actor spawned on death when DefaultBodyPresentation is OpenLootWindow: at the player's feet, or, for a death below the world's KillZ, at the last walkable place the server recorded (with none above the KillZ, nothing drops). |
| `LootableBodyLifetimeSeconds` | `USnapWorldSettings` | `float` | `300.f` | How long a body spawned at runtime (a death) stays, in seconds; 0 means it never despawns. |
| `bLootWindowOpensInventory` | `USnapWorldSettings` | `bool` | `true` | Opening a chest or a body also opens the bag screen beside it, so the player sees their room while taking things. |
| `bRagdollLootableBodies` | `USnapWorldSettings` | `bool` | `true` | Make the dead character itself the body you loot: a copy of it falls limp where it died, wearing what it wore, and worn items vanish from it as they are taken. |
| `bRemoveEmptiedLootableBodies` | `USnapWorldSettings` | `bool` | `false` | Remove a body spawned at runtime once everything is taken out of it. |

## Persistence

| Setting | Class | Type | Default | What it does |
|---|---|---|---|---|
| `PersistenceAutoSaveCoalesceSeconds` | `USnapFoundationSettings` | `float` | `2.0f` | How long an auto-save waits after the first change before writing, so a burst of changes is one write; later changes don't restart the wait. |
| `PersistenceMigratorClass` | `USnapFoundationSettings` | `TSoftClassPtr<USnapPersistenceMigrator>` | `*(none)*` | The class that brings an older saved record up to date (a USnapPersistenceMigrator subclass). |
| `PersistenceProviderClass` | `USnapFoundationSettings` | `TSoftClassPtr<UObject>` | `*(none)*` | The class that decides where saved data goes. |
| `PersistenceSaveSlotPrefix` | `USnapFoundationSettings` | `FString` | `TEXT("SnapInventory_")` | Prefixed to the profile key to make the shipped provider's save slot name. |
| `PersistenceSaveUserIndex` | `USnapFoundationSettings` | `int32` | `0` | The UserIndex passed to UGameplayStatics::SaveGameToSlot. |
| `PersistenceWorldProfileKey` | `USnapFoundationSettings` | `FString` | `TEXT("SnapWorld")` | The reserved profile key the world store is written under: one world file per level, for the whole server. |
| `bEnablePersistenceAutoSave` | `USnapFoundationSettings` | `bool` | `false` | Write a player's inventory automatically whenever it changes. |
| `bEnableWorldPersistence` | `USnapFoundationSettings` | `bool` | `false` | Save and restore the world's own inventory state: dropped pickups, loot chests, crates and lootable bodies (USnapWorldPersistenceSubsystem). |
| `bLogPersistenceVerbose` | `USnapFoundationSettings` | `bool` | `false` | Log every store read and write at Verbose. |
| `bRestoreWorldOnBeginPlay` | `USnapFoundationSettings` | `bool` | `false` | Restore the world automatically when the level begins play. |

## Pickups

| Setting | Class | Type | Default | What it does |
|---|---|---|---|---|
| `bAllowOverlapPickup` | `USnapWorldSettings` | `bool` | `true` | Whether walking into a pickup collects it. |
| `bAutoEquipOnPickup` | `USnapInventorySettings` | `bool` | `true` | Whether a collected item that fits a free equipment slot is worn at once. |
| `bPickupRarityGlowAsLight` | `USnapWorldSettings` | `bool` | `true` | How the rarity glow looks: on (the default), a soft light in the tier's color with a faint rim on the item; off, a flat colored disc under it. |
| `bShowPickupRarityGlow` | `USnapWorldSettings` | `bool` | `true` | A glow in the rarity tier's color under every item lying in the world. |
| `bSwapToBetterRarityOnPickup` | `USnapWorldSettings` | `bool` | `true` | Upgrade on contact: a collected item swaps in for a worn copy of the same item at a lower rarity. |

## Screen

| Setting | Class | Type | Default | What it does |
|---|---|---|---|---|
| `InventoryScreenClass` | `USnapInventoryUISettings` | `TSoftClassPtr<UUserWidget>` | `TSoftClassPtr<UUserWidget>(FSoftObjectPat...` | The inventory screen widget to show. |
| `ProximityCheckInterval` | `USnapInventorySettings` | `float` | `0.25f` | How often, in seconds, an open loot window, shop or crafting station checks that the player is still in reach, closing when not. |
| `ScreenZOrder` | `USnapInventoryUISettings` | `int32` | `50` | Z-order the screen is added to the viewport at: above default HUD content, below modal and system UI. |
| `StackSplitCursorOffset` | `USnapInventoryUISettings` | `float` | `12.f` | How far below the cursor the split window's top-left corner opens, in Slate units. |
| `StackSplitWidgetClass` | `USnapInventoryUISettings` | `TSoftClassPtr<UUserWidget>` | `TSoftClassPtr<UUserWidget>(FSoftObjectPat...` | The "how many?" window shown when the player Ctrl+drags part of a stack, or Ctrl+clicks a currency. |
| `bKeepInventoryScreenBuilt` | `USnapInventoryUISettings` | `bool` | `true` | Build the bag screen once (hidden, soon after the character is possessed) and keep it between opens. |
| `bOnePanelInventoryScreen` | `USnapInventoryUISettings` | `bool` | `false` | Draw the bag screen's panels as one framed window: one title, one close box, dragged as one. |

## Screen|Backdrop

| Setting | Class | Type | Default | What it does |
|---|---|---|---|---|
| `ScreenBackdropBlurStrength` | `USnapUICoreSettings` | `float` | `8.f` | Blur strength in Blur mode (UBackgroundBlur::BlurStrength). |
| `ScreenBackdropDarkenAmount` | `USnapUICoreSettings` | `float` | `0.6f` | Alpha of the black fill in Darken mode: 0 is invisible, 1 is opaque black. |
| `ScreenBackdropMode` | `USnapUICoreSettings` | `ESnapScreenBackdropMode` | `ESnapScreenBackdropMode::None` | What is drawn behind the inventory and vendor screens, and behind a station's craft window opened on its own. |

## Screen|Storage

| Setting | Class | Type | Default | What it does |
|---|---|---|---|---|
| `MaxStorageNestingDepth` | `USnapInventorySettings` | `int32` | `2` | The deepest a storage item may sit, counting from the grid: an item in the grid is at depth 1. |
| `StorageWindowWidgetClass` | `USnapInventorySettings` | `TSoftClassPtr<UUserWidget>` | `TSoftClassPtr<UUserWidget>(FSoftObjectPat...` | The window shown when the player opens a storage item (a pouch, a chest) by right-click or double-click. |
| `bAllowNestedStorageItems` | `USnapInventorySettings` | `bool` | `true` | Whether a storage item may go inside another (a pouch in a pouch), up to MaxStorageNestingDepth. |
| `bDestroyEmptiedStorageItems` | `USnapInventorySettings` | `bool` | `false` | Destroy a storage item when its last stack is taken out. |

## Screen|Unbought Squares

| Setting | Class | Type | Default | What it does |
|---|---|---|---|---|
| `UnboughtSquarePicture` | `USnapInventoryUISettings` | `TSoftObjectPtr<UTexture2D>` | `*(none)*` | The picture drawn over each unbought square, edge to edge, dimmed by the theme's Unbought Square Tint. |
| `UnboughtSquareUpgradeItems` | `USnapInventoryUISettings` | `TArray<TSoftObjectPtr<class USnapItemDefinition>>` | `*(none)*` | The capacity upgrade items the unbought squares are counted from. |
| `bShowUnboughtBagSquares` | `USnapInventoryUISettings` | `bool` | `false` | Show the room capacity upgrades could still add to the main bag, as squares with a lock picture that take no input. |

## Sorting

| Setting | Class | Type | Default | What it does |
|---|---|---|---|---|
| `bServerSortWritesSlots` | `USnapInventorySettings` | `bool` | `false` | Whether SortInventory may rewrite slot indices. |

## Sound

| Setting | Class | Type | Default | What it does |
|---|---|---|---|---|
| `ActionSounds` | `USnapUICoreSettings` | `FSnapActionSounds` | `*(none)*` | The sounds of a spell learned, two items mixed, an item repaired or dismantled, a craft finished, a purchase, a sale. |
| `DefaultItemSounds` | `USnapUICoreSettings` | `FSnapItemSounds` | `*(none)*` | The sounds of an item no ItemTypeSounds row matches (and of any field a matching row leaves empty). |
| `ItemTypeSounds` | `USnapUICoreSettings` | `TArray<FSnapItemTypeSounds>` | `*(none)*` | An item's Lift, Place and Use sounds by its type: the most specific row whose ItemType the item's tags carry wins, and an unmatched item uses DefaultItemSounds. |
| `SoundVolume` | `USnapUICoreSettings` | `float` | `1.0f` | How loud the plugin's sounds play: 1 is as authored, 0 silent. |
| `StorageCloseSound` | `USnapUICoreSettings` | `TSoftObjectPtr<USoundBase>` | `SNAP_EXAMPLE_SOUND("Containers", "S_Conta...` | A pouch window closing, for a storage item whose own fragment names no CloseSound. |
| `StorageOpenSound` | `USnapUICoreSettings` | `TSoftObjectPtr<USoundBase>` | `SNAP_EXAMPLE_SOUND("Containers", "S_Conta...` | A pouch window opening, for a storage item whose own fragment names no OpenSound. |
| `WorldSoundFalloffDistance` | `USnapUICoreSettings` | `float` | `2200.0f` | How far beyond WorldSoundInnerRadius, in centimeters, that sound fades out; past it, it isn't heard. |
| `WorldSoundInnerRadius` | `USnapUICoreSettings` | `float` | `300.0f` | A world container's sound (a chest opening) plays at full volume within this many centimeters of it. |
| `bPlaySounds` | `USnapUICoreSettings` | `bool` | `true` | Whether the plugin plays its sounds: items lifted, placed, collected, used, worn or dropped, containers opening, crafting actions, buttons and windows. |

## Spatial Grid

| Setting | Class | Type | Default | What it does |
|---|---|---|---|---|
| `bAllowItemRotation` | `USnapInventorySettings` | `bool` | `true` | Whether items may be turned in a grid, by every route: the rotate key, first-fit placement and swaps. |
| `bDrawSpatialItemsPerCell` | `USnapInventorySettings` | `bool` | `true` | Draw a multi-cell item in a spatial grid as one widget per cell. |
| `bFillSpatialItemCells` | `USnapInventorySettings` | `bool` | `true` | Show a multi-cell item's rarity background on every cell it covers, not only its top-left one. |

## Spoilage

| Setting | Class | Type | Default | What it does |
|---|---|---|---|---|
| `SpoilageMergePolicy` | `USnapItemsSettings` | `ESnapSpoilageMergePolicy` | `ESnapSpoilageMergePolicy::Average` | What merging two perishable stacks does to the result's clock (ESnapSpoilageMergePolicy). |
| `SpoilageMergeToleranceSeconds` | `USnapItemsSettings` | `float` | `1.f` | How close two remaining times must be, in seconds, for DoNotStack to treat them as one clock and allow the merge. |
| `SpoilageSweepInterval` | `USnapItemsSettings` | `float` | `5.f` | How often, in seconds, the server checks held, stored and worn perishables for an expired clock. |
| `bEnableSpoilage` | `USnapItemsSettings` | `bool` | `true` | Master switch for the whole spoilage feature. |
| `bSpoilageAffectsSellPrice` | `USnapItemsSettings` | `bool` | `true` | Scale a spoilable item's sell price by its freshness (never below 1). |

## Spoilage|Appearance

| Setting | Class | Type | Default | What it does |
|---|---|---|---|---|
| `SpoilageFreshTint` | `USnapItemsSettings` | `FLinearColor` | `FLinearColor(0.0765f, 0.4327f, 0.0850f, 1.f)` | Bar fill color above SpoilageYellowFraction. |
| `SpoilageRedFraction` | `USnapItemsSettings` | `float` | `0.2f` | Freshness fraction at or below which a slot's bar turns red. |
| `SpoilageRedTint` | `USnapItemsSettings` | `FLinearColor` | `FLinearColor(0.9050f, 0.0623f, 0.0413f, 1.f)` | Bar fill color at or below SpoilageRedFraction. |
| `SpoilageYellowFraction` | `USnapItemsSettings` | `float` | `0.5f` | Freshness fraction at or below which a slot's bar turns yellow (above SpoilageRedFraction). |
| `SpoilageYellowTint` | `USnapItemsSettings` | `FLinearColor` | `FLinearColor(1.0f, 0.5327f, 0.0025f, 1.f)` | Bar fill color between SpoilageRedFraction and SpoilageYellowFraction. |
| `SpoiledDimTint` | `USnapItemsSettings` | `FLinearColor` | `FLinearColor(0.35f, 0.32f, 0.28f, 1.f)` | Icon dim tint for a spoiled (inert) item, applied the way BrokenDimTint is. |

## Stack Count Badge

| Setting | Class | Type | Default | What it does |
|---|---|---|---|---|
| `bShowStackCountBackground` | `USnapUICoreSettings` | `bool` | `false` | Draw the filled plate behind a stack-count number. |

## Tooltip

| Setting | Class | Type | Default | What it does |
|---|---|---|---|---|
| `StatDisplayTable` | `USnapInventoryUISettings` | `TSoftObjectPtr<class USnapStatDisplayTable>` | `TSoftObjectPtr<class USnapStatDisplayTabl...` | The curated table that turns an item's GameplayEffects into tooltip stat lines. |
| `bCompareToEquipped` | `USnapInventoryUISettings` | `bool` | `true` | Whether a tooltip on an equippable item compares it against what is worn: one block per occupied slot it could go in, each naming its slot. |
| `bPlaceMouseTooltipBesideItem` | `USnapInventoryUISettings` | `bool` | `true` | With the mouse, every tooltip stands still beside what it explains instead of following the pointer: an item's card beside the item (in a grid, beside the whole item), and each small tooltip (a filter tab, a currency, a craft tile, a sort button) beside its control. |
| `bShowFocusedTooltipOnGamepad` | `USnapInventoryUISettings` | `bool` | `true` | On a gamepad, the focused item square shows its tooltip beside it, following each focus move (Slate itself shows tooltips only under the pointer). |
| `bShowTooltipShortcuts` | `USnapInventoryUISettings` | `bool` | `true` | Whether an item tooltip ends with a dim "Hold Alt for shortcuts" line. |

## Trade

| Setting | Class | Type | Default | What it does |
|---|---|---|---|---|
| `MaxTradeOfferEntries` | `USnapEconomySettings` | `int32` | `12` | How many items one player may offer at once, which is also how many squares each offer board draws (12: four by three). |
| `TradeDebugKey` | `USnapEconomySettings` | `FKey` | `EKeys::F7` | The key bBindTradeDebugKeyRaw binds. |
| `TradeMaxRange` | `USnapEconomySettings` | `float` | `800.f` | How far apart two players may be, in world units, and still hold a trade open; past it the server cancels the trade and returns both offers. |
| `TradeNoticeSeconds` | `USnapEconomySettings` | `float` | `4.f` | How long a trade notice stays on screen, in seconds. |
| `TradeRangeCheckInterval` | `USnapEconomySettings` | `float` | `0.5f` | How often, in seconds, the server re-checks TradeMaxRange while a trade is open. |
| `TradeScreenClass` | `USnapEconomySettings` | `TSoftClassPtr<UUserWidget>` | `TSoftClassPtr<UUserWidget>(FSoftObjectPat...` | The full-screen widget two players trade in. |
| `bBindTradeDebugKeyRaw` | `USnapEconomySettings` | `bool` | `false` | Development aid: TradeDebugKey invites the nearest other player in TradeMaxRange to a trade. |

## UI

| Setting | Class | Type | Default | What it does |
|---|---|---|---|---|
| `DefaultThemeUsage` | `USnapUICoreSettings` | `ESnapWidgetThemeUsage` | `ESnapWidgetThemeUsage::Authored` | Whether a picked UITheme recolors the colors a Widget Blueprint authors, for every widget and property that hasn't chosen. |
| `NoticePromptSeconds` | `USnapInventoryUISettings` | `float` | `3.f` | How long the white notice line stays before it fades, in seconds, for a one-line message. |
| `NoticeStyle` | `USnapInventoryUISettings` | `ESnapNoticeStyle` | `ESnapNoticeStyle::Prompt` | How the plugin tells a player why something failed (a full bag, stacks that will not combine, a junk sale's report). |
| `UITheme` | `USnapUICoreSettings` | `FSoftObjectPath` | `*(none)*` | The theme asset that supplies every color the plugin's native widgets paint. |
| `bCalibratePreviewExposure` | `USnapUICoreSettings` | `bool` | `true` | Let the bag's character preview calibrate its exposure to the figure it shows. |
| `bPersistWindowLayout` | `USnapUICoreSettings` | `bool` | `true` | Remember where the player dragged the bag, equipment and craft windows, across sessions. |

## UI|Gestures

| Setting | Class | Type | Default | What it does |
|---|---|---|---|---|
| `bEquipOnDoubleClick` | `USnapUICoreSettings` | `bool` | `true` | Double-clicking an equippable item in the bag or a pouch equips it, as Shift+click and the menu's Equip do. |

## Vendor

| Setting | Class | Type | Default | What it does |
|---|---|---|---|---|
| `DebugVendorStock` | `USnapEconomySettings` | `TSoftObjectPtr<class USnapVendorStockDataAsset>` | `*(none)*` | The stock bBindVendorKeyRaw's key opens. |
| `MaxTrackedVendorShelves` | `USnapEconomySettings` | `int32` | `8192` | The most vendor shelves (vendor and player pairs) the stock ledger holds; past it the least recently used is dropped. |
| `SellPriceFraction` | `USnapInventorySettings` | `float` | `0.25f` | What a vendor pays for an item, as a fraction of its buy price: 0.25 is a quarter. |
| `SellWindowClass` | `USnapEconomySettings` | `TSoftClassPtr<UUserWidget>` | `TSoftClassPtr<UUserWidget>(FSoftObjectPat...` | The sell window a shop sale asks in when bConfirmVendorSales is on. |
| `VendorBuybackSlots` | `USnapEconomySettings` | `int32` | `0` | How many recently sold things a shop remembers for the player to buy back. |
| `VendorCurrencyTag` | `USnapEconomySettings` | `FGameplayTag` | `*(none)*` | The currency a price uses when nothing more specific names one: the last step after the price's own and the shop's. |
| `VendorKey` | `USnapEconomySettings` | `FKey` | `EKeys::N` | The key bBindVendorKeyRaw binds. |
| `VendorScreenClass` | `USnapEconomySettings` | `TSoftClassPtr<UUserWidget>` | `TSoftClassPtr<UUserWidget>(FSoftObjectPat...` | The three-column vendor screen: shop list, buy panel and the player's inventory. |
| `VendorStockScope` | `USnapEconomySettings` | `ESnapVendorStockScope` | `ESnapVendorStockScope::PerPlayer` | Whose remaining count a limited vendor row has. |
| `bAffixesAffectSellPrice` | `USnapEconomySettings` | `bool` | `true` | Multiply a sell price by the item's rolled affixes' ValueMultiplier, so a well-rolled sword is worth more. |
| `bBindVendorKeyRaw` | `USnapEconomySettings` | `bool` | `false` | Development aid: a raw key (VendorKey) that opens a shop on DebugVendorStock, for a test map with no NPC yet. |
| `bConfirmVendorSales` | `USnapEconomySettings` | `bool` | `false` | Whether a shop sale asks first. |

## Vendor|Restock

| Setting | Class | Type | Default | What it does |
|---|---|---|---|---|
| `VendorRestockOverride` | `USnapEconomySettings` | `FSnapVendorRestockRule` | `*(none)*` | The restock rule every shop follows while bOverrideVendorRestock is on. |
| `bOverrideVendorRestock` | `USnapEconomySettings` | `bool` | `false` | Replace every shop's own restock rule with VendorRestockOverride, project-wide (for a game mode, a test or a demo). |

## Vendor|Upgrades

| Setting | Class | Type | Default | What it does |
|---|---|---|---|---|
| `BaseUpgradeCost` | `USnapEconomySettings` | `int32` | `250` | Price of one rarity upgrade in VendorCurrencyTag units: `BaseUpgradeCost * (CurrentRank + 1)`. |
| `bEnableEquipmentUpgrades` | `USnapEconomySettings` | `bool` | `true` | The vendor's equipment upgrade tab: pay to raise a worn item's rarity one tier. |

## Widget Classes

| Setting | Class | Type | Default | What it does |
|---|---|---|---|---|
| `ConfirmWindowWidgetClass` | `USnapUICoreSettings` | `TSoftClassPtr<USnapInventoryConfirmWindowWidget>` | `TSoftClassPtr<USnapInventoryConfirmWindow...` | Widget class for every confirmation window and notice window (NoticeStyle Window). |
| `DropOutlineWidgetClass` | `USnapUICoreSettings` | `TSoftClassPtr<USnapDropOutlineWidget>` | `TSoftClassPtr<USnapDropOutlineWidget>(FSo...` | Widget class for the green or red outline round the square under a dragged or carried item (USnapDropOutlineWidget). |
| `FocusRingWidgetClass` | `USnapUICoreSettings` | `TSoftClassPtr<USnapFocusRingWidget>` | `*(none)*` | Widget class for the outline round a focused button or currency on the bag screen (USnapFocusRingWidget). |
| `FocusedTooltipWidgetClass` | `USnapUICoreSettings` | `TSoftClassPtr<USnapFocusedTooltipWidget>` | `TSoftClassPtr<USnapFocusedTooltipWidget>(...` | Widget class for the layer that shows a pad-focused item square's tooltip beside it (USnapFocusedTooltipWidget). |
| `NoticePromptWidgetClass` | `USnapUICoreSettings` | `TSoftClassPtr<USnapInventoryNoticePromptWidget>` | `*(none)*` | Widget class for the white notice line (NoticeStyle Prompt). |
| `ThemedTooltipWidgetClass` | `USnapUICoreSettings` | `TSoftClassPtr<USnapInventoryThemedTooltipWidget>` | `TSoftClassPtr<USnapInventoryThemedTooltip...` | Widget class for the themed tooltip on the plugin's own buttons, glyphs and readouts (not the item tooltip). |
| `TradeNoticeWidgetClass` | `USnapUICoreSettings` | `TSoftClassPtr<USnapInventoryTradeNoticeWidget>` | `TSoftClassPtr<USnapInventoryTradeNoticeWi...` | Widget class for the short trade banner ("nobody is close enough", "they closed the trade"). |

## World Drops

| Setting | Class | Type | Default | What it does |
|---|---|---|---|---|
| `DropMotion` | `USnapInventorySettings` | `ESnapDropMotion` | `ESnapDropMotion::Auto` | Whether a world drop may physically eject what it spawns (ESnapDropMotion). |
| `DropTraceChannel` | `USnapInventorySettings` | `TEnumAsByte<ECollisionChannel>` | `ECC_WorldStatic` | The collision channel world drops trace on to find the floor and the room around it. |

## Fields of the structs the settings hold

Not settings of their own: each is a field of a row or a value that one of the settings above
holds, edited inside that setting on the page.

### `FSnapComponentAttachSpec`

| Field | Type | Default | What it does |
|---|---|---|---|
| `ComponentClass` | `FSoftClassPath` | `*(none)*` | The component to add. Must derive from UActorComponent. |
| `Host` | `ESnapAttachHost` | `ESnapAttachHost::PlayerState` | Which of the player's actors hosts this component on the manual path. |
| `ReceiverClass` | `FSoftClassPath` | `*(none)*` | The actor class (and every subclass of it) that receives the component automatically. |
| `bAddUnique` | `bool` | `true` | Skip an actor that already has a component of this class or a subclass of it. |
| `bEnabled` | `bool` | `true` | Untick to stop this component being attached, automatically or by EnsureSnapInventoryComponents. |

### `FSnapCurrencyAmount`

| Field | Type | Default | What it does |
|---|---|---|---|
| `CounterTag` | `FGameplayTag` | `*(none)*` | The currency these units are in, named by the counter tag that holds its balance. |
| `Units` | `int32` | `0` | The number of units: an int32, like FSnapInventoryCounter::Count. |

### `FSnapCurrencyDefinition`

| Field | Type | Default | What it does |
|---|---|---|---|
| `BaseUnitValue` | `int32` | `1` | What one unit of this currency is worth, in base units. |
| `CounterTag` | `FGameplayTag` | `*(none)*` | The slotless counter that holds this currency's balance: the tag your currency item routes to in Counter mode. |
| `DisplayName` | `FText` | `*(none)*` | Human-readable name, used as the row entry's tooltip. |
| `Icon` | `FSoftObjectPath` | `*(none)*` | Icon beside this currency's number in the wallet row, and beside every price in it (shop rows, sell lines, craft costs). |

### `FSnapHotbarBarSpec`

| Field | Type | Default | What it does |
|---|---|---|---|
| `Actions` | `TArray<TSoftObjectPtr<UInputAction>>` | `*(none)*` | The input actions that fire the bar's squares, in square order (element 0 fires square 0). |
| `AdmissionQuery` | `FGameplayTagQuery` | `*(none)*` | Which items the bar takes, as a tag query. |
| `BarTag` | `FGameplayTag` | `*(none)*` | The bar's identity: every hotbar API, placement and saved row names a bar by this tag. |
| `PanelSlotCount` | `int32` | `0` | How many of those squares the equipment panel draws. |
| `SlotCount` | `int32` | `0` | How many squares the bar has. 0 is a bar that exists and has no squares. |
| `bShowPageIndicator` | `bool` | `false` | Show which stored hotbar page is live ("2/3") beside the squares while HotbarPageCount is above 1. |

### `FSnapHotbarHUDPlacement`

| Field | Type | Default | What it does |
|---|---|---|---|
| `AlsoHideWidgetNames` | `TArray<FName>` | `*(none)*` | Other widgets in the same host that the bar replaces, hidden with the anchor (for example the second half of a two-part readout). |
| `AnchorWidgetName` | `FName` | `*(none)*` | The widget inside the host whose position the bar takes, and which is then hidden. |
| `BarTag` | `FGameplayTag` | `*(none)*` | Which bar this entry places, by its tag (see FSnapHotbarBarSpec). |
| `BelowAnchorGap` | `float` | `8.0f` | Clearance in Slate units between the anchor's bottom edge and the bar's top. |
| `BottomPadding` | `float` | `0.0f` | Clearance in Slate units below the bar, added to the anchor's own layout. |
| `FallbackAnchorHeight` | `float` | `96.0f` | The anchor height to assume until its real height is known (a widget has no size before its first paint). |
| `HostWidgetClass` | `FSoftClassPath` | `*(none)*` | The live HUD widget to look inside, found with GetAllWidgetsOfClass. |
| `bHideAnchorWidget` | `bool` | `true` | Hide the anchor once the bar has taken its place. |
| `bPlaceBelowAnchor` | `bool` | `false` | Sit below the anchor instead of taking its place, BelowAnchorGap under its bottom edge. |

### `FSnapHotbarSupersededWidget`

| Field | Type | Default | What it does |
|---|---|---|---|
| `HostWidgetClass` | `FSoftClassPath` | `*(none)*` | The live widget to look inside, found with GetAllWidgetsOfClass. |
| `WidgetName` | `FName` | `*(none)*` | The widget inside that host to collapse. |

### `FSnapInventoryContextActionEntry`

| Field | Type | Default | What it does |
|---|---|---|---|
| `ActionClass` | `TSoftClassPtr<USnapInventoryContextAction>` | `*(none)*` | The action. A concrete subclass of USnapInventoryContextAction. |
| `bEnabled` | `bool` | `true` | Untick to keep the row listed but never offered. |

### `FSnapInventoryHUDContainerRefreshBinding`

| Field | Type | Default | What it does |
|---|---|---|---|
| `ContainerTag` | `FGameplayTag` | `*(none)*` | The container whose changes trigger the call. |
| `FunctionName` | `FName` | `*(none)*` | A function with no parameters on that widget, usually its own "rebuild" function. |
| `HostWidgetClass` | `FSoftClassPath` | `*(none)*` | The HUD widget class to call. It's found in the viewport at runtime, so the plugin needs no reference to it. |

### `FSnapInventoryHUDEquipmentBinding`

| Field | Type | Default | What it does |
|---|---|---|---|
| `CountTextWidgetName` | `FName` | `*(none)*` | Text block for the stack count, shown for a stack of two or more. |
| `EmptyIconWidgetName` | `FName` | `*(none)*` | Image shown while the slot is empty (your own silhouette art; the plugin only shows or hides it). |
| `HostWidgetClass` | `FSoftClassPath` | `*(none)*` | The HUD widget class that hosts this readout. |
| `IconWidgetName` | `FName` | `*(none)*` | UImage painted with the equipped item's icon, shown while the slot is occupied. |
| `RarityColorWidgetName` | `FName` | `*(none)*` | Border tinted with the equipped item's rarity color. |
| `SlotTag` | `FGameplayTag` | `*(none)*` | Which equipment slot this readout mirrors. |
| `SlotWidgetName` | `FName` | `*(none)*` | The child widget for this slot when the host shows several slots. |

### `FSnapItemFilter`

| Field | Type | Default | What it does |
|---|---|---|---|
| `Icon` | `FSoftObjectPath` | `*(none)*` | Flat icon for the button, tinted by the FilterButton*Color settings. |
| `Label` | `FText` | `*(none)*` | The row's name: the button's tooltip, and its text when there is no Icon. |
| `MatchTags` | `FGameplayTagContainer` | `*(none)*` | The tags this row shows. An item matches when one of its tags is, or is a child of, one of these. |
| `bMatchesUnfilteredItems` | `bool` | `false` | Also show items that no tag row claims (a "Misc" catch-all). |

### `FSnapScreenOpenAbilityBinding`

| Field | Type | Default | What it does |
|---|---|---|---|
| `AbilityTagName` | `FName` | `*(none)*` | The tag on the ability to activate (for example "Ability.Type.Action.Jump"), not an input tag. |
| `Key` | `FKey` | `*(none)*` | The key that keeps working while a plugin screen is open. |
