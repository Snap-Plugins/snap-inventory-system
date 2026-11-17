<!-- GENERATED FILE -- do not edit by hand.
     Produced by Tools/generate_docs.py from the shipped headers. Tools/ is in the plugin's source
     repository, not in the packaged plugin.
     Re-run it after changing a declaration; `--check` fails when it is stale. -->

# Appendix: widget contracts

Each of these names a widget the plugin looks up **inside your Blueprint** with
`WidgetTree->FindWidget(Name)`. Give a widget that name and the plugin drives it; leave the
name blank and the plugin skips that feature entirely. Nothing is required.

**252 contracts across 39 widget classes and 3 headers of shared types.**

## `SnapHUDMirrorSettingsTypes.h`

| Property | Default widget name |
|---|---|
| `CountTextWidgetName` | `*(empty)*` |
| `EmptyIconWidgetName` | `*(empty)*` |
| `IconWidgetName` | `*(empty)*` |
| `RarityColorWidgetName` | `*(empty)*` |
| `SlotWidgetName` | `*(empty)*` |

## `SnapHotbarSettingsTypes.h`

| Property | Default widget name |
|---|---|
| `AnchorWidgetName` | `*(empty)*` |
| `WidgetName` | `*(empty)*` |

## `SnapUserWidget.h`

| Property | Default widget name |
|---|---|
| `WidgetName` | `*(empty)*` |

## `USnapActiveEffectSquareWidget`

| Property | Default widget name |
|---|---|
| `ElapsedWidgetName` | `FName("SnapEffectSquareElapsed")` |
| `FrameWidgetName` | `FName("SnapEffectSquareFrame")` |
| `GlyphWidgetName` | `FName("SnapEffectSquareGlyph")` |
| `IconWidgetName` | `FName("SnapEffectSquareIcon")` |
| `SizerWidgetName` | `FName("SnapEffectSquareSizer")` |
| `StacksTextWidgetName` | `FName("SnapEffectSquareStacks")` |
| `TimeTextWidgetName` | `FName("SnapEffectSquareTime")` |

## `USnapActiveEffectsWidget`

| Property | Default widget name |
|---|---|
| `RowsBoxWidgetName` | `FName("SnapEffectRows")` |

## `USnapCraftWaitingWidget`

| Property | Default widget name |
|---|---|
| `RowWidgetName` | `TEXT("SnapCraftWaitingRow")` |

## `USnapEquipmentCharacterPreviewWidget`

| Property | Default widget name |
|---|---|
| `PreviewImageWidgetName` | `FName("PreviewImage")` |

## `USnapEquipmentSlotWidget`

| Property | Default widget name |
|---|---|
| `DurabilityBarWidgetName` | `*(empty)*` |
| `GhostIconWidgetName` | `FName("GhostIcon")` |
| `HoverGlowWidgetName` | `FName("HoverGlow")` |
| `LockedGlyphWidgetName` | `*(empty)*` |
| `SlotLabelWidgetName` | `FName("SlotLabel")` |
| `StackTextWidgetName` | `*(empty)*` |

## `USnapFocusRingWidget`

| Property | Default widget name |
|---|---|
| `RingWidgetName` | `FName("SnapFocusRing")` |

## `USnapInventoryCollectFeedRowWidget`

| Property | Default widget name |
|---|---|
| `CountTextWidgetName` | `FName("SnapFeedRowCount")` |
| `IconHostWidgetName` | `FName("SnapFeedRowIconHost")` |
| `IconWidgetName` | `FName("SnapFeedRowIcon")` |
| `NameTextWidgetName` | `FName("SnapFeedRowName")` |
| `PlateWidgetName` | `FName("SnapFeedRowPlate")` |
| `RowSizerWidgetName` | `FName("SnapFeedRowSizer")` |

## `USnapInventoryCollectFeedWidget`

| Property | Default widget name |
|---|---|
| `RowContainerWidgetName` | `FName("SnapFeedRows")` |

## `USnapInventoryConfirmWindowWidget`

| Property | Default widget name |
|---|---|
| `CancelButtonWidgetName` | `FName("SnapConfirmCancel")` |
| `CancelLabelWidgetName` | `FName("SnapConfirmCancelLabel")` |
| `ConfirmButtonWidgetName` | `FName("SnapConfirmAccept")` |
| `ConfirmLabelWidgetName` | `FName("SnapConfirmAcceptLabel")` |
| `MessageWidgetName` | `FName("SnapConfirmMessage")` |
| `PadHintAcceptGlyphWidgetName` | `FName("SnapConfirmPadAcceptGlyph")` |
| `PadHintAcceptTextWidgetName` | `FName("SnapConfirmPadAcceptText")` |
| `PadHintBackGlyphWidgetName` | `FName("SnapConfirmPadBackGlyph")` |
| `PadHintBackTextWidgetName` | `FName("SnapConfirmPadBackText")` |
| `PadHintRowWidgetName` | `FName("SnapConfirmPadHint")` |
| `PlateSizerWidgetName` | `FName("SnapConfirmSizer")` |
| `ScrimWidgetName` | `FName("SnapConfirmScrim")` |
| `TitleWidgetName` | `FName("SnapConfirmTitle")` |

## `USnapInventoryContextMenuWidget`

| Property | Default widget name |
|---|---|
| `PlateWidgetName` | `FName(TEXT("SnapContextMenuPlate"))` |
| `RowContainerWidgetName` | `FName(TEXT("SnapContextMenuRows"))` |

## `USnapInventoryCraftChipWidget`

| Property | Default widget name |
|---|---|
| `ChipBackgroundWidgetName` | `FName("SnapCraftChipBG")` |
| `ChipCountPillWidgetName` | `FName("SnapCraftChipCountPill")` |
| `ChipCountTextWidgetName` | `FName("SnapCraftChipCount")` |
| `ChipIconFitWidgetName` | `FName("SnapCraftChipIconFit")` |
| `ChipIconWidgetName` | `FName("SnapCraftChipIcon")` |
| `ChipSizerWidgetName` | `FName("SnapCraftChipSizer")` |

## `USnapInventoryCraftPanelWidget`

| Property | Default widget name |
|---|---|
| `CostTextWidgetName` | `FName("CraftDetailCost")` |
| `CraftButtonTextWidgetName` | `FName("CraftButtonText")` |
| `CraftButtonWidgetName` | `FName("CraftButton")` |
| `CraftTimeTextWidgetName` | `FName("CraftDetailTime")` |
| `DetailIconWidgetName` | `FName("CraftDetailIcon")` |
| `DetailNameWidgetName` | `FName("CraftDetailName")` |
| `DetailOutputCountTextWidgetName` | `FName("CraftDetailOutputCount")` |
| `DetailProgressWidgetName` | `FName("CraftDetailProgress")` |
| `DetailSubtitleWidgetName` | `FName("CraftDetailSubtitle")` |
| `DismantleButtonTextWidgetName` | `FName("DismantleButtonText")` |
| `DismantleButtonWidgetName` | `FName("DismantleButton")` |
| `IngredientHeadingWidgetName` | `FName("CraftIngredientHeading")` |
| `IngredientListSizerWidgetName` | `FName("CraftIngredientListSizer")` |
| `PanelBorderWidgetName` | `FName("CraftPanelBorder")` |
| `PanelHeadingWidgetName` | `FName("CraftPanelHeading")` |
| `QuantityWidgetName` | `FName("CraftQuantityStepper")` |
| `QueueBackdropWidgetName` | `FName("CraftQueueBackdrop")` |
| `QueueClearButtonTextWidgetName` | `FName("CraftQueueClearButtonText")` |
| `QueueClearButtonWidgetName` | `FName("CraftQueueClearButton")` |
| `QueueHeadingWidgetName` | `FName("CraftQueueHeading")` |
| `QueueListSizerWidgetName` | `FName("CraftQueueListSizer")` |
| `StatusTextWidgetName` | `FName("CraftDetailStatus")` |

## `USnapInventoryCraftProgressWidget`

| Property | Default widget name |
|---|---|
| `CancelButtonTextWidgetName` | `FName("CraftDetailProgressCancelLabel")` |
| `CancelButtonWidgetName` | `FName("CraftDetailProgressCancel")` |
| `ProgressBarWidgetName` | `FName("CraftDetailProgressBar")` |
| `StateTextWidgetName` | `FName("CraftDetailProgressState")` |

## `USnapInventoryCraftQueueRowWidget`

| Property | Default widget name |
|---|---|
| `CancelButtonTextWidgetName` | `FName("CraftQueueRowCancelButtonText")` |
| `CancelButtonWidgetName` | `FName("CraftQueueRowCancelButton")` |
| `CountPillWidgetName` | `FName("SnapCraftQueueTileCountPill")` |
| `CountTextWidgetName` | `FName("CraftQueueRowCount")` |
| `IconWidgetName` | `FName("CraftQueueRowIcon")` |
| `NameTextWidgetName` | `FName("CraftQueueRowName")` |
| `ProgressBarWidgetName` | `FName("CraftQueueRowProgress")` |
| `StatusTextWidgetName` | `FName("CraftQueueRowStatus")` |

## `USnapInventoryDragVisualWidget`

| Property | Default widget name |
|---|---|
| `ActionBadgeWidgetName` | `*(empty)*` |
| `ActionGlyphWidgetName` | `*(empty)*` |
| `CountBadgeWidgetName` | `*(empty)*` |
| `IconWidgetName` | `*(empty)*` |
| `RarityWidgetName` | `*(empty)*` |
| `StackTextWidgetName` | `*(empty)*` |

## `USnapInventoryHotbarSlotWidget`

| Property | Default widget name |
|---|---|
| `ChargesWidgetName` | `NAME_None` |
| `CooldownTextWidgetName` | `NAME_None` |
| `CooldownWidgetName` | `NAME_None` |
| `CountTextWidgetName` | `FName("HotbarSlotCount")` |
| `FrameWidgetName` | `FName("HotbarSlotFrame")` |
| `HoverGlowWidgetName` | `FName("HoverGlow")` |
| `KeyGlyphWidgetName` | `FName("HotbarSlotKeyGlyph")` |
| `KeyTextWidgetName` | `FName("HotbarSlotKey")` |
| `LockedWidgetName` | `NAME_None` |

## `USnapInventoryHotbarWidget`

| Property | Default widget name |
|---|---|
| `PageIndicatorWidgetName` | `NAME_None` |
| `RowWidgetName` | `FName("HotbarRow")` |

## `USnapInventoryLootSlotWidget`

| Property | Default widget name |
|---|---|
| `StackTextWidgetName` | `FName("StackText")` |

## `USnapInventoryNoticePromptWidget`

| Property | Default widget name |
|---|---|
| `PromptTextWidgetName` | `FName("SnapNoticePromptText")` |

## `USnapInventoryPanel`

| Property | Default widget name |
|---|---|
| `ContainerNameTextWidgetName` | `*(empty)*` |
| `CounterIconWidgetName` | `*(empty)*` |
| `CounterValueTextWidgetName` | `*(empty)*` |
| `OccupancyTextWidgetName` | `*(empty)*` |
| `WeightReadoutAnchorWidgetName` | `FName("CapacityLabel")` |

## `USnapInventoryScreenWidget`

| Property | Default widget name |
|---|---|
| `BackdropWidgetName` | `FName("Backdrop")` |

## `USnapInventorySlotWidget`

| Property | Default widget name |
|---|---|
| `DurabilityBarWidgetName` | `*(empty)*` |
| `FavoriteGlyphWidgetName` | `*(empty)*` |
| `GhostIconWidgetName` | `*(empty)*` |
| `HoverGlowWidgetName` | `FName("HoverGlow")` |
| `JunkGlyphWidgetName` | `*(empty)*` |
| `LockGlyphWidgetName` | `*(empty)*` |

## `USnapInventorySortMenuWidget`

| Property | Default widget name |
|---|---|
| `RowContainerWidgetName` | `FName(TEXT("SnapSortMenuRows"))` |

## `USnapInventoryStackSplitWidget`

| Property | Default widget name |
|---|---|
| `AmountSliderWidgetName` | `FName("SplitAmountSlider")` |
| `AmountTextWidgetName` | `FName("SplitAmountText")` |
| `BodyWidgetName` | `FName("SplitBody")` |
| `CancelButtonWidgetName` | `FName("SplitCancelButton")` |
| `ConfirmButtonWidgetName` | `FName("SplitConfirmButton")` |
| `DecrementButtonWidgetName` | `FName("SplitMinusButton")` |
| `IncrementButtonWidgetName` | `FName("SplitPlusButton")` |
| `ItemDescriptionWidgetName` | `FName("SplitItemDescription")` |
| `ItemIconWidgetName` | `FName("SplitItemIcon")` |
| `ItemNameWidgetName` | `FName("SplitItemName")` |
| `ScrimWidgetName` | `FName("SplitScrim")` |

## `USnapInventoryStorageSlotWidget`

| Property | Default widget name |
|---|---|
| `DurabilityBarWidgetName` | `*(empty)*` |
| `StackTextWidgetName` | `FName("StackText")` |

## `USnapInventoryThemedTooltipWidget`

| Property | Default widget name |
|---|---|
| `LabelWidgetName` | `FName("SnapThemedTooltipText")` |
| `PlateWidgetName` | `FName("SnapThemedTooltipPlate")` |

## `USnapInventoryTooltipWidget`

| Property | Default widget name |
|---|---|
| `FlavorTextWidgetName` | `FName("FlavorText")` |
| `GrantsTextWidgetName` | `FName("GrantsText")` |
| `ShortcutRowsWidgetName` | `FName("ShortcutRows")` |
| `ShortcutTextWidgetName` | `FName("ShortcutText")` |
| `StatusTextWidgetName` | `FName("StatusText")` |
| `TitleTextWidgetName` | `FName("TitleText")` |
| `TypeTextWidgetName` | `FName("TypeText")` |
| `WidthBoxWidgetName` | `FName("TipSizer")` |

## `USnapInventoryTradeNoticeWidget`

| Property | Default widget name |
|---|---|
| `NoticeTextWidgetName` | `FName("SnapTradeNoticeText")` |

## `USnapInventoryTradeOfferSlotWidget`

| Property | Default widget name |
|---|---|
| `SpoilageBarWidgetName` | `FName("SnapTradeOfferSpoilageBar")` |

## `USnapInventoryTradeScreenWidget`

| Property | Default widget name |
|---|---|
| `BagPanelHostWidgetName` | `FName("TradeBagPanels")` |
| `CancelButtonWidgetName` | `FName("TradeMyCancelButton")` |
| `ConfirmButtonWidgetName` | `FName("TradeMyConfirmButton")` |
| `ConfirmLabelWidgetName` | `FName("TradeMyConfirmLabel")` |
| `CurrencyMinusWidgetName` | `FName("TradeMyCurrencyMinus")` |
| `CurrencyPlusWidgetName` | `FName("TradeMyCurrencyPlus")` |
| `CurrencySliderWidgetName` | `FName("TradeMyCurrencySlider")` |
| `MyActionRowWidgetName` | `FName("TradeMyActionRow")` |
| `MyConfirmBandWidgetName` | `FName("TradeMyConfirmBand")` |
| `MyConfirmTextWidgetName` | `FName("TradeMyConfirmText")` |
| `MyCurrencyBlockWidgetName` | `FName("TradeMyCurrencyBlock")` |
| `MyCurrencyCoinBoxWidgetName` | `FName("TradeMyCurrencyCoinBox")` |
| `MyCurrencyCoinWidgetName` | `FName("TradeMyCurrencyCoin")` |
| `MyCurrencyReadoutWidgetName` | `FName("TradeMyCurrencyReadout")` |
| `MyCurrencyRowWidgetName` | `FName("TradeMyCurrencyRow")` |
| `MyOfferListWidgetName` | `FName("TradeMyOfferList")` |
| `StatusTextWidgetName` | `FName("TradeMyStatusText")` |
| `TheirActionRowWidgetName` | `FName("TradeTheirActionRow")` |
| `TheirConfirmBandWidgetName` | `FName("TradeTheirConfirmBand")` |
| `TheirConfirmTextWidgetName` | `FName("TradeTheirConfirmText")` |
| `TheirCurrencyBlockWidgetName` | `FName("TradeTheirCurrencyBlock")` |
| `TheirCurrencyCoinBoxWidgetName` | `FName("TradeTheirCurrencyCoinBox")` |
| `TheirCurrencyCoinWidgetName` | `FName("TradeTheirCurrencyCoin")` |
| `TheirCurrencyReadoutWidgetName` | `FName("TradeTheirCurrencyReadout")` |
| `TheirCurrencyRowWidgetName` | `FName("TradeTheirCurrencyRow")` |
| `TheirOfferListWidgetName` | `FName("TradeTheirOfferList")` |
| `TitleWidgetName` | `FName("TradeTitle")` |
| `WindowOverlayWidgetName` | `FName("TradeWindowOverlay")` |

## `USnapInventoryVendorBuyPanelWidget`

| Property | Default widget name |
|---|---|
| `ActionButtonLabelWidgetName` | `FName("BuyButtonLabel")` |
| `BuyButtonWidgetName` | `FName("BuyButton")` |
| `BuyStackButtonWidgetName` | `FName("BuyStackButton")` |
| `BuyStackLabelTextWidgetName` | `FName("BuyStackButtonLabel")` |
| `BuyStackTotalTextWidgetName` | `FName("BuyStackButtonTotal")` |
| `BuyTotalTextWidgetName` | `FName("BuyButtonTotal")` |
| `CancelButtonWidgetName` | `FName("BuyCancelButton")` |
| `ContentRootWidgetName` | `FName("BuyPanelContent")` |
| `CostLabelWidgetName` | `FName("BuyCostLabel")` |
| `CostRowWidgetName` | `FName("BuyCostRow")` |
| `DecrementGlyphWidgetName` | `FName("BuyMinusGlyph")` |
| `IncrementGlyphWidgetName` | `FName("BuyPlusGlyph")` |
| `ItemDescriptionWidgetName` | `FName("BuyItemDescription")` |
| `ItemIconContainerWidgetName` | `FName("BuyIconSizer")` |
| `ItemIconWidgetName` | `FName("BuyItemIcon")` |
| `ItemNameWidgetName` | `FName("BuyItemName")` |
| `ItemSubtitleWidgetName` | `FName("BuyItemSubtitle")` |
| `MintInfoWidgetName` | `FName("BuyMintInfo")` |
| `QuantityCaptionWidgetName` | `FName("BuyQtyLabel")` |
| `QuantityRowWidgetName` | `FName("BuyQtyRow")` |
| `UnitPriceIconWidgetName` | `FName("BuyUnitPriceIcon")` |
| `UnitPriceTextWidgetName` | `FName("BuyUnitPrice")` |

## `USnapInventoryVendorCraftRowWidget`

| Property | Default widget name |
|---|---|
| `ChipStripWidgetName` | `FName("CraftRowChipStrip")` |
| `IconWidgetName` | `FName("CraftRowIcon")` |
| `NameTextWidgetName` | `FName("CraftRowName")` |
| `OutputCountTextWidgetName` | `FName("CraftRowOutputCount")` |
| `RowBackgroundWidgetName` | `FName("CraftRowBackground")` |
| `SubtitleTextWidgetName` | `FName("CraftRowSubtitle")` |

## `USnapInventoryVendorQuantityWidget`

| Property | Default widget name |
|---|---|
| `AmountSliderWidgetName` | `FName("VendorAmountSlider")` |
| `AmountTextWidgetName` | `FName("VendorAmountText")` |
| `DecrementButtonWidgetName` | `FName("VendorMinusButton")` |
| `IncrementButtonWidgetName` | `FName("VendorPlusButton")` |
| `QuantityBlockWidgetName` | `FName("VendorQuantityBlock")` |

## `USnapInventoryVendorRowWidget`

| Property | Default widget name |
|---|---|
| `ItemIconWidgetName` | `FName("VendorRowIcon")` |
| `ItemNameWidgetName` | `FName("VendorRowName")` |
| `ItemSubtitleWidgetName` | `FName("VendorRowSubtitle")` |
| `MagicTagWidgetName` | `FName("VendorRowMagicTag")` |
| `PriceIconWidgetName` | `FName("VendorRowPriceIcon")` |
| `PriceTextWidgetName` | `FName("VendorRowPrice")` |
| `SelectionBorderWidgetName` | `FName("VendorRowSelection")` |
| `StockContainerWidgetName` | `FName("VendorRowStockBox")` |
| `StockTextWidgetName` | `FName("VendorRowStock")` |

## `USnapInventoryVendorScreenWidget`

| Property | Default widget name |
|---|---|
| `BuyPanelWidgetName` | `FName("VendorBuyPanel")` |
| `RestockLineWidgetName` | `FName("VendorRestockLine")` |
| `UpgradeItemDescriptionWidgetName` | `FName("VendorUpgradeItemDescription")` |
| `UpgradeItemIconWidgetName` | `FName("VendorUpgradeItemIcon")` |
| `UpgradeItemNameWidgetName` | `FName("VendorUpgradeItemName")` |
| `UpgradeItemSubtitleWidgetName` | `FName("VendorUpgradeItemSubtitle")` |

## `USnapInventoryVendorSellWidget`

| Property | Default widget name |
|---|---|
| `BodyWidgetName` | `FName("SellBody")` |
| `CancelButtonWidgetName` | `FName("SellCancelButton")` |
| `ItemDescriptionWidgetName` | `FName("SellItemDescription")` |
| `ItemIconWidgetName` | `FName("SellItemIcon")` |
| `ItemNameWidgetName` | `FName("SellItemName")` |
| `ScrimWidgetName` | `FName("SellScrim")` |
| `SellButtonWidgetName` | `FName("SellConfirmButton")` |
| `SellTotalTextWidgetName` | `FName("SellConfirmTotal")` |
| `UnitPriceTextWidgetName` | `FName("SellUnitPrice")` |

## `USnapInventoryVendorUpgradeRowWidget`

| Property | Default widget name |
|---|---|
| `GhostIconWidgetName` | `FName("UpgradeRowGhost")` |
| `GhostOutlineWidgetName` | `FName("UpgradeRowGhostFrame")` |
| `ItemIconWidgetName` | `FName("UpgradeRowIcon")` |
| `ItemNameWidgetName` | `FName("UpgradeRowName")` |
| `PriceIconWidgetName` | `FName("UpgradeRowPriceIcon")` |
| `PriceTextWidgetName` | `FName("UpgradeRowPrice")` |
| `RepairPriceBoxWidgetName` | `FName("UpgradeRowRepairBox")` |
| `RepairPriceIconWidgetName` | `FName("UpgradeRowRepairPriceIcon")` |
| `RepairPriceLabelWidgetName` | `FName("UpgradeRowRepairLabel")` |
| `RepairPriceTextWidgetName` | `FName("UpgradeRowRepairPrice")` |
| `SelectionBorderWidgetName` | `FName("UpgradeRowSelection")` |
| `SlotNameWidgetName` | `FName("UpgradeRowSlot")` |
| `TierWidgetName` | `FName("UpgradeRowTiers")` |
| `UpgradePriceLabelWidgetName` | `FName("UpgradeRowPriceLabel")` |

## `USnapItemSlotWidget`

| Property | Default widget name |
|---|---|
| `FocusedGlowWidgetName` | `FName("FocusedGlow")` |
| `IconWidgetName` | `*(empty)*` |
| `RarityGlowWidgetName` | `NAME_None` |
| `SelectionGlowWidgetName` | `FName("SelectionGlow")` |
| `SpoilageBarWidgetName` | `*(empty)*` |

## `USnapOnePanelWindowWidget`

| Property | Default widget name |
|---|---|
| `FooterWidgetName` | `FName("Footer")` |
| `FrameWidgetName` | `FName("PanelRoot")` |
| `HeaderRuleWidgetName` | `FName("HeaderRule")` |
| `TitleWidgetName` | `FName("Title")` |
