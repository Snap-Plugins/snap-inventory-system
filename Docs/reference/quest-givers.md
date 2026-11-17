# Quest givers

A quest giver is any actor — an NPC, a statue, a mailbox — that takes in items a player brings and gives something back:
other items, and equipment slots unlocked for good. Nothing here needs code.

- **The "!":** While a player carries everything one of its quests asks for, a "!" floats over it for that player,
  spinning and bobbing.
- **The prompt:** Walking up to it offers **Turn in: <the quest's title>**.

## Quest givers in three steps

1. **Make the quest.** **Content Browser** > right-click > **Snap Inventory** > **Quest**. Fill in:
   - `Title` — the quest's name, shown in the prompt ("Turn in: Deliver the Sealed Letter").
   - `Objectives` — what the player must do before the turn-in, every one of them: **Collect Items** (carried; counted,
     not taken), **Deliver Items** (handed over at the turn-in) or **Talk to Giver**.
   - `Rewards` — what the player receives: **Items**, **Currency** or **Unlock Slot** (an equipment slot unlocked for
     good, such as `Equipment.Slot.Ring.2`).
   - `Repeat` — **Once** (the default: once per player) or **Repeatable**.
   - `bRequiresAccept` (off by default) — the player must accept the quest first
     (`USnapQuestLogComponent::AcceptQuest`); a giver then takes it in only while it is among that player's active
     quests.
2. **Make an actor a giver.** Add a **Snap Quest Giver** component (`USnapQuestGiverComponent`) to any actor and list
   the quest in its `Quests`.
   - The component is an interactable, so the actor needs no other interaction component.
3. **Place the actor** in your level.

The example is `BP_Example_QuestGiverNPC` — the Courier — with `DA_ExampleQuest_SealedLetter`:

- Bring the Sealed Letter.
- Receive a Ring of Vitality, and the second ring slot unlocks.

## What a player sees

- **No quest ready: nothing.** A giver none of whose quests a player can turn in offers that player no interaction at
  all — no outline, no prompt.
  - Unless the component has options of its own (`Options`), which keep working beside the quest.
- **A quest ready: the "!" and the prompt.** The "!" is drawn on each player's own machine, from that player's own bag.
  - So two players near one giver each see their own answer.
  - It is shown or hidden when the bag changes, not every frame.

### The turn-in

The press goes to the server (`USnapQuestLogComponent::Server_TurnInQuest`). It checks reach the way it does for a
pickup and then turns the quest in — **all or nothing**:

1. The rewards are checked for room as things stand, before anything is taken.
   - Each is checked in the container it belongs in, as a pickup or a chest take routes it: the main bag for an ordinary
     item, the container its routing tag names for a routed one.
   - A counter item such as a coin goes to its counter, which is never full.
   - The rewards bound for one container are counted together, so two of them cannot each be promised its last free
     square (`USnapInventoryComponent::CanAddItemsWithReason`).
2. The items of its **Deliver Items** objectives are taken from the main bag (**Collect Items** are only counted).
3. "Quest complete: <the quest's title>" goes on that player's collect feed.
   - Then each reward is added where it belongs, each shown on the feed just below that line.
4. Its **Unlock Slot** rewards are unlocked (`USnapEquipmentComponent::UnlockSlot` — replicated and saved with the
   player).
5. The quest is remembered on the player (`USnapQuestLogComponent::CompletedQuestIds`), so a **Once** quest is not
   offered again.

A turn-in that cannot complete takes nothing and says why:

- `QuestNotReady`: "You do not have everything this quest asks for."
- When the main bag has no room for its rewards: "Your inventory is full." (`BagFull`).
- When another container (a routed item's own) has no room: the reward named ("<Reward> -- There is no room for that.").
- A reward that would pass its carry limit: "<Reward> -- You already hold as many of those as you are allowed."
  (`UnitCapReached`).
- A reward nothing on the player takes: `WrongContainer`.

In code, `USnapQuestGiverComponent::TurnInFor` returns the code.

- `TurnInForWithReason` also gives the notice, phrased by the container that refused.
- See [Refusals](refusals.md).

Two things to know:

- ⚠ **Item objectives are counted in (and Deliver Items taken from) the player's MAIN bag — not its pouches.**
  - The "!" and the turn-in ask the same question of the same container, so they cannot disagree.
  - A letter carried in a pouch does not count until it is taken out.
- ⚠ **The room check is cautious.** A reward that would fit only in the room the delivered items leave behind is refused.
  The player makes room and tries again.
  - It can never leave a player with the quest items gone and no reward.

## Saving

- A player's finished and active quests are saved with their profile, in the quest log's section of their record
  (`FSnapQuestLogRecord`; see [Persistence](persistence.md)).
- A quest is remembered by `USnapQuestDefinition::GetQuestId`: its `QuestId`, or the asset's name when that is empty.
  - ⚠ Changing the `QuestId` (or renaming the asset while it is empty) makes a **Once** quest new again for every player
    who finished it.
- Slots a quest unlocked are saved with the player's equipment.

## The "!"

The component draws it with two world-space widget components, made on every machine that has a player (never on a
dedicated server).

- They are two one-sided faces back to back, each with its own widget, so the mark reads the same from behind.
- A single two-sided world widget draws its back as a dark silhouette.

Its settings, under **Quest Giver | Marker**:

| Setting | Default | What it does |
|---|---|---|
| `bShowMarker` | On | Draw it at all |
| `MarkerWidgetClass` | `WBP_ExampleQuestMarker` | The widget drawn: a `USnapQuestMarkerWidget`, a bold "!" (`QuestMarkText`) in a gold yellow — the theme's `InteractionOutline` scaled by (1, 0.85, 0.2) — with a thin dark amber outline. Restyle the example or name a Widget Blueprint of your own. |
| `MarkerHeightAboveTop` | 16 cm | The widget's bottom edge above the top of the giver's collision. The glyph's dot sits about 40 cm above that edge; the demo's Courier uses -6. |
| `MarkerSpinDegreesPerSecond` | 90 | 0 = it does not turn |
| `MarkerBobHeight` / `MarkerBobSeconds` | 4 cm / 1.6 s | 0 = it does not bob |
| `MarkerScale` | 1 | One slate unit is one centimeter at 1 |
| `MarkerBrightness` | 1.6 | Tints the unlit world widget: above 1 is brighter than the widget's own colors (a little bloom) |

- The motion is read from the game clock, so it is the same at any frame rate.
- With several local players (split screen) the "!" follows the first one.

## Blueprint and C++

- `USnapQuestGiverComponent::FindQuestToTurnIn(Player)` — the first quest in `Quests` the player can turn in now.
- `USnapQuestGiverComponent::CanPlayerTurnIn(Quest, Player)` — the one question the "!", the prompt and the server ask.
- `USnapQuestGiverComponent::TurnInFor(Player)` — the server's turn-in, callable from a server-side script.
- `OnQuestTurnedIn` — fires on the server after a turn-in.
- `USnapQuestLogComponent::HasCompletedQuest(QuestId)` / `GetCompletedQuestIds()` — what a player has done, for a
  quest log of your own.
- `TurnInPromptText` — the prompt's words; `{Quest}` is the quest's title.

A subclass of `USnapInteractableComponent` can add options of its own the same way the quest giver adds its "Turn in":
see [Loot and interaction](loot-and-interaction.md#a-subclasss-own-options).

## See also

- [Loot and interaction](loot-and-interaction.md) — the interaction system the giver is part of
- [Equipment](equipment.md) — slot locks, `UnlockSlot` and `UnlockTag`
- [Persistence](persistence.md) — what a profile saves
