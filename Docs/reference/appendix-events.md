<!-- GENERATED FILE -- do not edit by hand.
     Produced by Tools/generate_docs.py from the shipped headers. Tools/ is in the plugin's source
     repository, not in the packaged plugin.
     Re-run it after changing a declaration; `--check` fails when it is stale. -->

# Appendix: events

Every `BlueprintAssignable` delegate the plugin broadcasts. Bind these in Blueprint to react
to inventory changes without subclassing anything.

**53 delegates across 26 classes.**

## `USnapCraftingComponent`

| Event | Signature | Fires when |
|---|---|---|
| `OnCraftingChanged` | `FSnapOnCraftingChanged` | Fires when this player's crafting state changes: the queue or finished crafts waiting at a station (on the server and the owning client), the station session or the unlocked recipes (on the owning client only, as they arrive). |

## `USnapDamageWearComponent`

| Event | Signature | Fires when |
|---|---|---|
| `OnDamageWearChanged` | `FSnapOnDamageWearChanged` | Fires on the server after a hit wears at least one equipped item. |

## `USnapDeathDropComponent`

| Event | Signature | Fires when |
|---|---|---|
| `OnDeathDropChanged` | `FSnapOnDeathDropChanged` | Fires on the server when this player's death drop latch changes: a death drop ran for this life, or a respawn or a new pawn cleared it. |

## `USnapEncumbranceComponent`

| Event | Signature | Fires when |
|---|---|---|
| `OnEncumbranceChanged` | `FSnapOnEncumbranceChanged` | Fires when this player's carry limit is changed (SetMaxCarryWeightKilograms), beside the main bag's OnInventoryChanged, which keeps firing: on the server when the limit is set, and on a client when the new limit reaches it (the owning client; every client when the bag's ReplicationScope is Everyone). |

## `USnapEquipmentComponent`

| Event | Signature | Fires when |
|---|---|---|
| `OnActionRefused` | `FSnapOnActionRefused` | Fires on the owning client when one of this component's player gestures is refused, with the reason. |
| `OnEquipmentChanged` | `FSnapOnEquipmentChanged` | Broadcast on every equip/replace/unequip. |
| `OnEquippedStatsChanged` | `FSnapOnEquippedStatsChanged` | Raised when GetEquippedStatTotal may have changed; it can fire more than once for one change. |
| `OnSlotLockChanged` | `FSnapOnSlotLockChanged` | Raised when a slot becomes admissible or stops being. |

## `USnapEquipmentVisualComponent`

| Event | Signature | Fires when |
|---|---|---|
| `OnEquipmentVisualChanged` | `FSnapOnEquipmentVisualChanged` | Fires on this machine when the animation layers linked for worn items, or the actor they are linked on, change. |

## `USnapHotbarComponent`

| Event | Signature | Fires when |
|---|---|---|
| `OnActionRefused` | `FSnapOnActionRefused` | Fires on the owning client when one of this component's gestures is refused. |
| `OnActivePageChanged` | `FSnapOnHotbarActivePageChanged` | When the live stored hotbar changes. |
| `OnHotbarChanged` | `FSnapOnHotbarChanged` | Raised after any slot change: an assign, a clear, a replicated update or a pruned stale reference. |
| `OnSelectionChanged` | `FSnapOnHotbarSelectionChanged` | When what is held from a selectable hotbar changes. |
| `OnSlotActivated` | `FSnapOnHotbarSlotActivated` | Once per activation attempt, refusals included. |

## `USnapHotbarSelectionComponent`

| Event | Signature | Fires when |
|---|---|---|
| `OnSelectionChanged` | `FSnapOnHotbarSelectionChanged` | Fires when what is held from a selectable hotbar changes. |

## `USnapInteractableComponent`

| Event | Signature | Fires when |
|---|---|---|
| `OnInteracted` | `FSnapOnInteracted` | Fired when a viewer interacted with this. |
| `OnInteractionOptionPerformed` | `FSnapOnInteractionOptionPerformed` | Fired alongside OnInteracted, carrying which option the press performed. |

## `USnapInteractionDetectorComponent`

| Event | Signature | Fires when |
|---|---|---|
| `OnFocusChanged` | `FSnapOnInteractionFocusChanged` | Fires on every change of focus, including to null; never for a re-scan that lands on the same target. |

## `USnapInventoryComponent`

| Event | Signature | Fires when |
|---|---|---|
| `OnActionRefused` | `FSnapOnActionRefused` | Fires on the owning client when one of this component's gestures is refused. |
| `OnCounterChanged` | `FSnapOnInventoryCounterChanged` | Fires on every counter change, on the server and on clients. |
| `OnCraftWaitingReported` | `FSnapOnCraftWaitingReported` | Fires when a timed craft finished away from its station and waits there (see FSnapOnCraftWaitingReported). |
| `OnDurabilityChanged` | `FSnapOnInventoryDurabilityChanged` | Fires on the server when an item this actor holds (grid, storage or worn) changes durability. |
| `OnFeedTextReported` | `FSnapOnFeedTextReported` | Fires when a plain text line is put on the player's feed (see FSnapOnFeedTextReported). |
| `OnInventoryChanged` | `FSnapOnInventoryChanged` | Fires on every add, remove, stack-count, move or merge made through this component (see FSnapOnInventoryChanged). |
| `OnItemActionPerformed` | `FSnapOnItemActionPerformed` | Fires when one of the player's item actions succeeded (see FSnapOnItemActionPerformed), on the player's main container, whichever container it happened in. |
| `OnItemBroke` | `FSnapOnInventoryItemBroke` | Fires once when an item's durability reaches 0 (see FSnapOnInventoryItemBroke). |
| `OnItemCollected` | `FSnapOnItemCollected` | Fires when items arrive from a collect (see FSnapOnItemCollected). |
| `OnItemCreated` | `FSnapOnItemCreated` | Fires when an item was made for the player (see FSnapOnItemCreated). |
| `OnItemDropped` | `FSnapOnItemDropped` | Fires on the server when a world drop puts an actor into the world (see FSnapOnItemDropped). |
| `OnItemUseEnded` | `FSnapOnItemUseEnded` | Fires when a timed use ends, landed or not (see FSnapOnItemUseEnded), on the player's main container on every machine. |
| `OnItemUseStarted` | `FSnapOnItemUseStarted` | Fires when a timed use starts (see FSnapOnItemUseStarted), on the player's main container on every machine. |
| `OnTradeChanged` | `FSnapOnInventoryTradeChanged` | Fires on both sides whenever anything about the trade session changes (see FSnapOnInventoryTradeChanged). |

## `USnapInventoryConfirmWindowWidget`

| Event | Signature | Fires when |
|---|---|---|
| `OnConfirmed` | `FSnapInventoryConfirmWindowConfirmed` | Fired once, when CONFIRM is pressed. |
| `OnDeclined` | `FSnapInventoryConfirmWindowDeclined` | Fires on a deliberate decline only: CANCEL or Escape, never on teardown. |

## `USnapInventoryHUDComponent`

| Event | Signature | Fires when |
|---|---|---|
| `OnItemLandedInSquare` | `FSnapOnItemLandedInSquare` | Fires when an item lands in one of this player's squares on screen, once per landing, as its picture starts to bounce. |

## `USnapInventoryLootWidget`

| Event | Signature | Fires when |
|---|---|---|
| `OnSelectionChanged` | `FSnapOnInventoryLootSelectionChanged` | — |

## `USnapInventoryPanel`

| Event | Signature | Fires when |
|---|---|---|
| `OnSelectionChanged` | `FSnapOnInventoryPanelSelectionChanged` | Fires whenever the selection changes, never on a mere repaint (see FSnapOnInventoryPanelSelectionChanged; a handler must not change the selection). |

## `USnapInventoryStorageWidget`

| Event | Signature | Fires when |
|---|---|---|
| `OnSelectionChanged` | `FSnapOnInventoryStorageSelectionChanged` | — |

## `USnapItemInstance`

| Event | Signature | Fires when |
|---|---|---|
| `OnInstanceChanged` | `FSnapOnInventoryInstanceChanged` | Fires when this instance's own state changes: its stack count, a memory or a tag stack. |

## `USnapItemUseComponent`

| Event | Signature | Fires when |
|---|---|---|
| `OnItemUseChanged` | `FSnapOnItemUseChanged` | Fires when this player's timed item use starts or ends. |

## `USnapLootInventoryComponent`

| Event | Signature | Fires when |
|---|---|---|
| `OnLootSlotCountChanged` | `FSnapOnLootSlotCountChanged` | Fires when GetLootSlotCount grows, on the server and on every client. |
| `OnOpenedChanged` | `FSnapOnLootContainerOpenedChanged` | Fires when the container is first opened (true) and when a respawn closes it again (false). |

## `USnapQuestGiverComponent`

| Event | Signature | Fires when |
|---|---|---|
| `OnQuestTurnedIn` | `FOnSnapQuestTurnedIn` | Fires on the server after a turn-in completes. |

## `USnapQuestLogComponent`

| Event | Signature | Fires when |
|---|---|---|
| `OnQuestAccepted` | `FOnSnapQuestLogEvent` | Fires on the server when a quest is accepted. |
| `OnQuestCompleted` | `FOnSnapQuestLogEvent` | Fires on the server when every objective of an active quest is met. |
| `OnQuestLogChanged` | `FOnSnapQuestLogChanged` | Fires when the turned-in list or the active quests change: on the server as they change, on the owning client as they arrive. |
| `OnQuestProgressed` | `FOnSnapQuestLogProgressed` | Fires on the server when an active quest's objective moves: the objective's index and its new progress. |
| `OnQuestTurnedIn` | `FOnSnapQuestLogEvent` | Fires on the server after a giver takes a quest in, accepted or not. |

## `USnapSetBonusComponent`

| Event | Signature | Fires when |
|---|---|---|
| `OnSetBonusChanged` | `FSnapOnSetBonusChanged` | Fires on the server when the set tiers applied to this player change. |

## `USnapTradeComponent`

| Event | Signature | Fires when |
|---|---|---|
| `OnTradeChanged` | `FSnapOnTradeChanged` | Fires when this player's trade changes: a session, an invite, the offer, a confirmation, or the escrowed items arriving. |

## `USnapTradePartnerComponent`

| Event | Signature | Fires when |
|---|---|---|
| `OnTradeCompleted` | `FOnSnapPartnerTradeCompleted` | Fires on the server after a trade with this partner completes. |

## `USnapVendorClientComponent`

| Event | Signature | Fires when |
|---|---|---|
| `OnVendorClientChanged` | `FSnapOnVendorClientChanged` | Fires when this player's shop session changes: on a client each time the session replicates (it opens or closes, or its stock or buyback rows change); on the server only after a sale that writes buyback rows, when another player's purchase changes the stock this shop shares (VendorStockScope PerVendor), or when an upgrade changes a bag item's rarity. |

## `USnapWalletComponent`

| Event | Signature | Fires when |
|---|---|---|
| `OnCounterChanged` | `FSnapOnInventoryCounterChanged` | Fires on every counter change, on the server and on clients. |
