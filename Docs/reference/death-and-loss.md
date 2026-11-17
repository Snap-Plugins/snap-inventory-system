# Death and loss

What a player loses when they die, where it goes, and how a project changes that. This page covers turning it on, what a
death does, the dead character as the body, the death drop policy, and deaths without the Gameplay Ability System.

- **Everything here runs on the server.** A client sees the result through replication.

**On this page:**

- [Turning it on](#turning-it-on)
- [What a death does](#what-a-death-does)
  - [The dead character as the body](#the-dead-character-as-the-body)
- [The policy](#the-policy)
- [Without the Gameplay Ability System](#without-the-gameplay-ability-system)
  - [Saying the player is dead](#saying-the-player-is-dead)
- [`DropEverythingToWorld`](#dropeverythingtoworld)

## Turning it on

**Nothing is lost on death until a project says so.** Three settings on `USnapInventoryComponent`
(`[/Script/SnapInventorySystem.SnapInventoryComponent]` in `DefaultGame.ini`):

| Setting | Default | What it does |
|---|---|---|
| `bDropEverythingOnDeath` | `false` | The switch. Off, a death costs nothing (unless the game calls `TriggerDeathDrop`, below). |
| `DeathTagName` | *(None)* | The gameplay tag your project raises while a player is dead — `Status.Death` for Lyra. Read through the player's Ability System Component. The project setting of the same name (**Snap Inventory System - Foundation → Gameplay Integration**) counts too wherever the plugin asks whether a player is dead — see [Saying the player is dead](#saying-the-player-is-dead). |
| `DeathDropPolicyClass` | *(None)* | What a death costs. Unset is total loss. See [The policy](#the-policy). |

- Only the **main bag** — the container with no container tag — runs the death drop.
- The Spells and Shards containers and a project's own tagged containers are emptied by it. They never run it themselves.

## What a death does

When the death tag appears (or a door below is called), the main bag runs one death drop, in this order:

1. **Once per death.** A latch is set first. A second report of the same death does nothing.
2. **The trade ends.**
   - An open trade is canceled and both sides' escrow goes home, so an item on the table does not survive its owner.
   - An invite this player sent is withdrawn, and one they were holding is declined.
3. **A lootable body** is left where the player died, holding their worn items and carried items.
   - A death below the world's KillZ uses the last walkable place instead (see
     [A death below the world's KillZ](#a-death-below-the-worlds-killz)).
   - It lands on the floor: a trace runs down from the pawn to `DropGroundTraceDistance` below its feet on
     `DropTraceChannel` (**Snap Inventory System - Inventory → World Drops**, `ECC_WorldStatic` by default).
   - The body is placed where it hits.
   - Nothing hit — a pit, mid-air — leaves it at the pawn.
4. **The rest scatters** into the world at the pawn's feet.
   - That is anything the body refused, and the currency, which no grid can hold.
   - A death below the world's KillZ scatters from the last walkable place instead (see
     [A death below the world's KillZ](#a-death-below-the-worlds-killz)).
   - A currency balance drops as one pile.

⚠ With nothing that will actually leave, **no body is left** — not an empty one.

- That covers a player whose items are all locked, all authored to survive death, or all kept by the policy.

⚠ Worn items and carried items each honor the item's own rules first.

- A **locked** item, and an item authored `bDropOnDeath = false`, stay with the player. See
  [Restrictions and marks](restrictions-and-marks.md).
- A **locked equipment slot** is different — it only refuses what goes in.
  - So a worn item in a locked slot drops like any other (see [Equipment](equipment.md)).

**The body's settings** (`DefaultBodyPresentation`, `LootableBodyClass` and `DropAllLootBagClass` are on **Snap Inventory System - Inventory
→ Loot**; `LootableBodyLifetimeSeconds`, `bRemoveEmptiedLootableBodies` and `bRagdollLootableBodies` on **Snap Inventory System - World →
Loot**):

- `DefaultBodyPresentation` chooses body or scatter for every death.
- `LootableBodyClass` and `LootableBodyLifetimeSeconds` set the body's class and how long a death's body stays (0: until
  something removes it).
  - When the lifetime runs out, the body scatters what it still holds and goes.
  - A body PLACED in a level never times out: it is level content, like a placed chest
    (`ASnapLootableBody::IsPlacedInLevel`).
- `LootableBodyClass` is also the drop-all loot bag's class unless `DropAllLootBagClass` names another (see
  [Loot and interaction](loot-and-interaction.md)).

**The `bRemoveEmptiedLootableBodies` setting** (off by default) removes a body once everything has been taken out of it and no player is left in loot range.

- It checks on the next tick after the body empties, and again every `ProximityCheckInterval` while a player can still
  loot it (an emptied loot window stays open on it).
- It applies only to a body spawned while the game runs — a death's body, or one world persistence restored.
- A body that never held anything is left alone.
- So is a body placed in a level (`ASnapLootableBody::IsPlacedInLevel`), which is level content like a placed chest.
- **In the demo:** The example demo turns it on with a lifetime of 0. So a death's body waits however long the player
  takes, and goes once they have taken everything back.

### A death below the world's KillZ

A player who falls out of the world dies below its KillZ (the world settings' KillZ, which the engine acts on only while the
world's bounds checks are on). A body or a scatter there would be out of every player's reach, and the engine destroys anything
down there that moves. So such a death drops at the **last place the character stood on walkable ground** instead:

- The server records that place while the character walks: at the end of each of its character movement updates that ends on a
  walkable floor found where it stands. A place it was only moved to, such as a teleport into the air, is never recorded.
- It is the place of the owner's current pawn: a new pawn starts a new record, and a pawn without character movement records
  nothing.
- The body is traced down from there as it would be from the pawn, and every scatter of the death (worn items, every container,
  the counters) starts from there. The pawn itself is not moved.
- The body keeps its own look: the copy of the character (`bRagdollLootableBodies`) would be posed where the character fell.
- With **no usable place recorded** (none, or one below the KillZ), nothing drops and the player keeps everything; one log
  line says why. The rest of the death (the latch, the trade, the access sessions and the invites) runs as usual.
- A death above the KillZ, or in a world with the bounds checks off, drops where the player died, as above. A direct
  `DropEverythingToWorld` (not a death) is unchanged.

### The dead character as the body

The `bRagdollLootableBodies` setting (**Snap Inventory System - World → Loot**, on by default) makes the character who died the body you
loot, instead of the body actor's own look:

- The body (`ASnapLootableBody`) shows its **own copy of the character**:
  - The skeletal mesh that drives its pose, and the materials it wore.
  - Plus a modular character's other meshes — its own skeletal mesh components, and those of actors attached to it by a
    child actor component (how Lyra draws its whole visible body on an invisible pose mesh).
  - The copy starts in the pose the character died in and falls limp under the mesh's physics asset.
- **Each machine simulates its own copy.**
  - Nothing about the fall is replicated, so two players can see it settle a little differently.
  - A machine that still has the character copies its current pose (on a client it may already be falling) and its own
    materials.
  - One that does not uses the pose taken at the death.
  - Each machine hides the character's own mesh as its copy appears and stops it colliding. The capsule is left alone.
- **Everything that was worn is drawn on the copy**, where it was worn.
  - It goes through the same spawn the character wore it with (`USnapEquipmentVisualComponent::SpawnEquipmentVisualFor`: its socket, offset and slot
    override, and nothing for an item worn Hidden, broken or spoiled).
  - It disappears from the copy the moment that item is taken out of the body.
  - Items that were only carried are not drawn.
  - The pieces do not collide, so they cannot push the copy around.
  - `ASnapLootableBody::AddWornVisual` is the door a project's own death path can call.
- **The prompt, the outline and the loot reach follow the copy.**
  - Each machine measures from its own copy.
  - A server allows `ASnapLootableBody::RagdollReachSlack` (150 cm) further, so a prompt a client was offered is
    never refused.
  - There is no slack in standalone, where there is one copy.
- When the body goes, the copy **sinks into the ground** over `RagdollFadeSeconds` (2 s), and looting stops as it starts.
  - The body goes when emptied (and nobody is in loot range) under `bRemoveEmptiedLootableBodies`, or at the end of
    `LootableBodyLifetimeSeconds` after it has scattered what is left.
  - A see-through fade would need the character's own materials to support one. A sink works for every character.
- **Fallback:** A character with no skeletal mesh, a mesh with no physics asset, or a `LootableBodyClass` that is not an
  `ASnapLootableBody` keeps the body actor's own look, unchanged. So does the body of a death below the world's KillZ
  ([above](#a-death-below-the-worlds-killz)).

⚠ **Your character's death code:** A character that ragdolls itself on death should ask
`ASnapLootableBody::IsTakenOverByRagdollBody` first.

- On a client, the death and the body can arrive in either order.
- A body that took the character over has already hidden its mesh.
- The body also re-asserts the takeover for five seconds.
- A game that **revives the same pawn** shows its mesh again (`SetHiddenInGame(false)` on it and its children) and
  restores the mesh's collision.

⚠ A body **restored by world persistence** comes back with the body actor's own look: the copy is not saved.

- The outline covers the copy, not the worn pieces drawn on it (separate actors, as on a living character).

⚠ A body holds what fits in its ledger — `USnapBodyLootInventoryComponent`, 40 slots unless its class or the instance says
otherwise (`SlotCountOverride`).

- Anything past that scatters.
- The 40 is the shipped bag (30 slots) plus the 10 shipped equipment slots, with nothing to spare.
- A project whose players carry more (a bag upgraded past 30 slots, more equipment slots, more containers) raises it.
  - For every body: `MaxNumSlots` under `[/Script/SnapInventorySystem.SnapBodyLootInventoryComponent]` in your
    `DefaultGame.ini`. 0 means unlimited.
  - For one body class: `SlotCountOverride` on its body component.
- The example body `BP_Example_LootableBody` sets `SlotCountOverride` to 64 on its component, enough for the demo's bag at
  its largest upgrade (48) and its 10 equipment slots.
  - The demo's Spells container is its spellbook (`BP_Example_DemoSpellsContainer`, 8 slots). It keeps what enters it,
    so a death leaves the spells with the player. See
    [Custom containers](custom-containers.md#a-container-that-keeps-what-enters-it-the-spellbook-example).

## The policy

`USnapDeathDropPolicy` is a Blueprintable class whose defaults are total loss.

**To use one:**

1. Make a Blueprint subclass.
2. Change its class defaults.
3. Name it in `DeathDropPolicyClass`.

A policy can only **keep** more:

- The item rules above are always asked first.
- Nothing a policy says makes an item leave that they would keep.

| Knob | Default | Keeps |
|---|---|---|
| `bDropEquipment` | True | False keeps every worn item worn |
| `bDropMainBag` | True | False keeps everything in the main bag (a pouch keeps its contents) |
| `KeptContainerTags` | *(Empty)* | The items of each tagged container listed, matched by exact tag |
| `CurrencyDropShare` | 1.0 | The rest of each balance. The share drops as one pile, rounded **down**: 7 at 0.6 drops 4 and keeps 3, and 1 at 0.5 drops nothing |
| `BodyPresentation` | Use Project Default | Overrides `DefaultBodyPresentation` for this policy's deaths |

**To keep particular items,** override **`ShouldDropItem`**: answer false and the item stays.

- It is asked only about items that would otherwise drop, never about a locked one.
- It must answer the same way every time it is asked about the same item — it is asked more than once per death.
- For a worn item, `Source` is the main bag and `WornSlot` is its slot.
- For a carried item, `Source` is its container and `WornSlot` is empty.

**To change the policy while the game runs,** construct one and pass it to `SetDeathDropPolicy` on the server (any
container forwards it to the main bag).

- Null restores total loss.

⚠ **A named class that cannot be used keeps everything.**

- This happens if `DeathDropPolicyClass` does not load, or is not a `USnapDeathDropPolicy`.
  - The likeliest cause of a failed load is a packaged build that never cooked it, because a class path is not a package
    reference.
- The player then keeps every item and coin on that death, and one Error per death names the class.
- **To avoid it,** list the class for cooking (the Asset Manager, or `DirectoriesToAlwaysCook`).

## Without the Gameplay Ability System

A project with no death tag calls one of three server-only functions on the player's main bag (any container forwards to
it):

| Function | Does |
|---|---|
| `NotifyOwnerDied()` | "This player died." Runs the death drop if `bDropEverythingOnDeath` is true. |
| `TriggerDeathDrop()` | Runs the death drop whatever `bDropEverythingOnDeath` says — for a per-death decision. |
| `NotifyOwnerRespawned()` | Clears the once-per-death latch. |

- Each death door returns true when it ran the drop.
- **Call them while the pawn is still possessed.** The body and the scatter are placed at the pawn (for a death below the
  world's KillZ, at the last walkable place recorded for it), and with no pawn nothing is dropped.
- The death-drop component beside the main bag (`USnapDeathDropComponent`, made on the server) has the same three doors, plus
  `SetDeathDropPolicy`, `GetDeathDropPolicy` and `DropEverythingToWorld` as Blueprint functions too; each acts for its owner's main bag.
- They are not remote procedure calls, so a client cannot run a death drop.

**The latch clears:**

- When the death tag is removed.
- On `NotifyOwnerRespawned`.
- When a PlayerState or Controller owner gets a **new** pawn.

An unpossess to no pawn, or the same pawn coming back, does not clear it.

The tag and a `NotifyOwnerDied` for the same death drop once, in whichever order they arrive.

### Saying the player is dead

Every "not while dead" rule in the plugin asks one function on the player's main bag,
`USnapInventoryComponent::IsOwnerDead`:

- The inventory screen, a vendor, a loot window and the trade screen refuse to open.
- The interaction prompt hides.
- A trade is refused.

It is a `BlueprintNativeEvent`, so a project with no death tag overrides it in a subclass of the component — Blueprint or
C++ — and all of those rules follow.

**Its default answer is the tag test.** The player is dead while the tag named by **either** the component's
`DeathTagName` **or** the project setting of the same name is on the player:

- On an actor's OWN tags (`IGameplayTagAssetInterface` on the component owner, its PlayerState or its pawn), in every build.
- Or on any ability system that could be the player's (the owner's, its PlayerState's and its pawn's), in a build with
  GameplayAbilities.

**So a project without GameplayAbilities** — or one that keeps its death outside an ability system — needs no subclass:

1. Carry the tag in the PlayerState's own replicated tags.
2. Report it through `IGameplayTagAssetInterface`.

The demo's PlayerState does exactly that on its native route.

⚠ Override it on the **main bag's** class: that is the component every rule asks.

⚠ It is asked on the owning client as well as the server, so answer from state both have — replicated state, or the tag
itself.

⚠ An inventory screen that is **already open** is closed on death by the death tag's event: `IsOwnerDead` is a question
nothing asks on a timer.

- Without the tag on an ability system, call `USnapInventoryHUDComponent::DismissScreensForDeath` (or
  `ISnapScreenHost::SnapCloseScreensForDeath`) on the dying player's own machine.
- It closes everything the tag closes — the vendor, the trade screen, a loot window, a station's craft window, the
  prompts, the notices, the bag — and hands the input state back.

## `DropEverythingToWorld`

- `DropEverythingToWorld()` scatters everything the policy lets go, with no body, no trade teardown and no latch.
- It is the scatter step of the death drop, callable on its own.
- A direct call leaves an item that is on the trade table in escrow.
- **For a death, use the doors above.**
