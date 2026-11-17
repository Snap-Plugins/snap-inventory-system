# Make a grid item

A walk-through: an item that takes more than one square in the grid layout, from a new item definition to dragging it
round the demo's grid bag. Nothing here needs code.

- **Such an item:** A sword three squares tall, a two by two helmet, an L-shaped plank.
- **What the grid layout is and how a bag switches to it:** See [Inventory and containers](inventory-and-containers.md).
- **Every fragment named below:** See [Items and fragments](items-and-fragments.md).

## 1. Create the item definition

1. Create the definition: **Content Browser** > right-click > **Snap Inventory** > **Item Definition**.
   - Or duplicate an example item such as `ID_Sword_Iron`.
2. Give it a **UI Data** fragment with a name and an icon.
   - The shape editor draws that icon behind its squares, so you can see which part of the picture each square covers.

## 2. Add the Footprint fragment

In the definition's **Fragments** list, add the **Footprint** fragment (`USnapItemFragment_Footprint`).

- **Its rows:** `Width`, `Height`, `bCanRotate` and the shape editor below them.
- **Without it:** An item without this fragment always takes one square.

## 3. Pick a size, then paint the shape

- **Size presets.** Under the squares, one button per common size (width x height):
  - **1 x 1**, **1 x 2**, **2 x 1**, **2 x 2**, **1 x 3**, **2 x 3** and **3 x 2**.
  - A click makes the item that size and a full rectangle, as one step in the undo history.
  - One Undo puts the old size and shape back.
  - You can also set `Width` and `Height` yourself (the slider goes to 8).
- **Paint.** Click a square to uncover or cover that cell.
  - An L or a T is a rectangle with cells uncovered.
  - The squares must stay connected (validation refuses a split shape).
  - An empty top-left square is tinted orange: that is the cell the item is stored by.
- **Reset to rectangle** covers every cell again.
- **Turning.** Turn on `bCanRotate` and the player can turn the item between upright and sideways with `R` (or the
  gamepad's right trigger).
  - The small outlines beside the squares show both orientations.
  - A shape a turn cannot change (a 1 x 1, a full square) never turns, whatever the flag says.
- **The validity line** under the editor says exactly what asset validation will report for the shape, as you paint.

## 4. Draw its icon for the grid

The grid draws an item's picture over its whole box. Pick how on the spatial panel and the pouch window (`IconDrawing`):

- **Fit** (the default) draws the whole texture at its own shape, as large as fits the box.
- **Footprint Region** expects a square texture with the item drawn in its middle at the footprint's shape, and fills
  the box with that region.
  - That is how the example icons are made.
- **Stretch** draws the whole texture stretched over the box.
- **With Fit or Footprint Region** the picture is never stretched; whichever you pick, it turns with the item.

## 5. Try it in the demo

1. Open `L_SnapInventoryDemo`.
2. Press **Play**.
3. Step on the **BAG LAYOUT** pad to switch the bag to the grid.
   - A project's own bag switches with `USnapInventoryComponent::SetLayoutMode`.
4. Give yourself the item: drop it in the level, or call `AddItem` on the player's bag.
5. Open the bag. You should see the item drawn across the squares its shape covers.
   - Hovering, selecting, the tooltip and the right-click menu treat it as one thing from any of its squares.
6. Drag it. A green outline shows where it would land, a red one where it would not fit.
   - Press `R` while holding it to turn it.

A pouch in a grid bag is a grid too (its `StorageWidth`), so the item can be dragged into a pouch that has room for its
shape and its weight.

## Sizes that work well

| Item | Size | Turns |
|---|---|---|
| Ring, potion, gem, repair kit | 1 x 1 | Never |
| Dagger, wand, totem | 1 x 2 | Yes |
| Sword, axe | 1 x 3 | Yes |
| Greatsword, staff | 1 x 4 | Yes |
| Ingot | 2 x 1 | Yes |
| Helmet, gloves, boots, shield | 2 x 2 | Never |
| Chest armor | 2 x 3 | Yes |

Pouches and bags read best with whole rows:

- Give a storage item `StorageSlots` that is a whole number of rows of its `StorageWidth` (6 slots 3 wide, 12 slots 4
  wide).
- Validation warns otherwise.
