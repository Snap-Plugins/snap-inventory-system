# Networking

Server-authoritative throughout. A client predicts what it can, asks for the rest, and is told when it is refused.

**On this page:**

- [The shape](#the-shape)
- [Replication](#replication)
- [What a client may compute](#what-a-client-may-compute)
- [Refusals reaching a client](#refusals-reaching-a-client)
- [Multicasts on a listen server](#multicasts-on-a-listen-server)
- [Testing](#testing)
- [See also](#see-also)

## The shape

Every mutation has three parts:

```
bool CanDoX(...)                 a predicate BOTH sides can ask
ESnapRefusal DoX(...)            the mutator -- authority only
void Server_DoX(...)             the RPC a client sends
```

`Server_DoX_Implementation` **re-asks** the predicate rather than trusting the client.

- So an honest client never sees a refusal and a dishonest one cannot get past the gate.

Validation (`_Validate`) is deliberately shallow:

- It exists to reject a malformed call — a nonsense index — not to duplicate the gameplay gate.
- That is because a failing `_Validate` disconnects the client and must only ever fire on something no honest client can
  send.

## Replication

- `USnapInventoryComponent`'s contents are a `FFastArraySerializer`.
- Item instances are replicated **subobjects**, registered on the component's subobject list.
- Counters are a plain replicated array.
- Per-owner data uses `COND_OwnerOnly`: the bag's earned capacity (bonus slots, the upgrade tier), and the components
  beside it replicate their own per-owner state (the craft queue, the shop session) the same way.

### Who receives a container's contents

`USnapInventoryComponent::ReplicationScope` decides it, per instance:

| `ESnapReplicationScope` | Reaches | Use it for |
|---|---|---|
| `OwnerOnly` (the default) | The owning connection only | A player's own bag |
| `Everyone` | Every relevant connection | A lootable chest, a corpse, a shared stash |

It covers everything that describes what the container **holds**: `InventoryList`, `Counters`, `StoredItems`, the
per-instance capacity overrides, and the item instances themselves.

That last one matters: each item is a replicated subobject with a condition of its own, so scoping the properties alone
would still send every item to every client.

- It is registered `COND_Dynamic` and resolved per object, so a class default, a Blueprint subclass default and a placed
  actor's details panel all work.
  - `USnapLootInventoryComponent` states `Everyone` explicitly.
- ⚠ **Read once**, when the component first replicates. Set it in defaults, in the details panel, or in C++ before
  `FinishSpawning` — never at runtime.
- ⚠ **`OwnerOnly` on an actor nobody owns reaches nobody.** A chest has no owning connection, so an owner-only chest
  looks empty to every client.
  - The component logs a warning when it sees that.
- ⚠ **Trade escrow is not scoped.** An offered item is registered for everyone while it is on the table, so the partner
  sees it whatever the offerer's scope — and so is the **inside** of an offered container.
  - Its rows are copied into replicated state (`FSnapEscrowedContents`).
  - Every instance they name is registered for everyone until the container leaves the table, so the partner sees the
    real contents and their durability and freshness.
  - Nothing that is not on offer is copied. See [Trade](trade.md).
- The base class ships `OwnerOnly`: a player's bag, pouches and counters reach no other client.
  - If a UI of yours shows one player what is in another player's bag, set `Everyone` on that component class.

### Can this machine see inside a container?

`USnapInventoryComponent::GetStorageContentsAvailability(StorageItem)` answers it:

| `ESnapStorageContentsAvailability` | Meaning |
|---|---|
| `Live` | This machine reads the container's own contents — the server, the owning player, or a container whose contents reach everyone |
| `Copied` | A partner's offered container: the trade's copy is here and every item in it has arrived |
| `Arriving` | The copy is here, but at least one item in it has not reached this machine yet |
| `Unknown` | Nothing on this machine describes the contents |

⚠ **`Unknown` is not "empty".**

- The value accessors — `GetStorageOccupancy`, `GetStoredItems`, `GetStorageContentsWeightKilograms` — have no way to
  say "I cannot tell".
  - So on `Arriving` or `Unknown` they answer as for an empty container.
- Ask for availability before drawing a count, a weight or a window for a container you do not own. The trade screen
  does.

### Client doors that are shut by default

Each of these lets a client ask for something the server has no independent way to check. So each is off until a project
turns it on for a component class:

| Flag | Gates | Why |
|---|---|---|
| `bAllowClientRequestedAdd` | `Server_AddItem`, `Server_TryStackItem` | Mints items |
| `bAllowClientRequestedRecipeUnlock` | `Server_UnlockRecipe` | Unlocks any recipe by name |
| `bAllowClientVendorSession` | `Server_BeginVendorSession` with a stock asset | Lets a client pick which shop to open, from anywhere |
| `bAllowClientPricedPayment` | `Server_PayCurrencyPrice` | Debits a price the client states |

A shop in the world does not need `bAllowClientVendorSession`. It opens through `Server_BeginVendorSessionAt`, which:

- Takes the vendor **interactable**.
- Re-checks its verb, that it is switched on, and the caller's reach on the server.
- Reads the stock list off it.

That is the same contract `Server_BeginStationSession` holds for stations.

A container that is used only where it is kept (`bRequireAccessSession`, the reach rule) holds its
**access sessions on the server only** — nothing about them replicates.

- The server measures the player against the session's place at every request that moves an item into or out of the
  container.
  - So a modified client gains nothing by sending one from elsewhere.
- Your UI closes the window by its own distance test with the same range.
- See [custom containers](custom-containers.md#a-container-used-only-where-it-is-kept-the-reach-rule).

### Network limits

Every `_Validate` bound — an index ceiling, a batch size, a unit count — is a constant on `FSnapNetLimits`
(`SnapNetLimits.h`).

- Every widget that clamps a request before sending it reads the same constant.
- A failed `_Validate` disconnects the client, so these are shape checks only.
- A bound an honest client can reach is a bug, not a defense.

### The FastArray removal trap

`FFastArraySerializer` calls `PreReplicatedRemove` while the removed entries are **still in the array** and erases them
at the end of the same receive.

- Anything that responds by re-reading the list therefore sees the removed element as still present and repaints it.
- And nothing fires afterwards, so the stale paint is permanent.

`PostReplicatedReceive` is the only callback that runs after the erase, so that is where the resync belongs. Both
item lists (`FSnapInventoryList`, `FSnapEquipmentList`) implement it.

⚠ Re-broadcast the **re-read current value**, not a blanket null: a "removal" is not always an emptying.

- A move between two containers reaches the client as remove-then-add, so the slot may legitimately be occupied again.

## What a client may compute

A predicate both sides call must not read `UPROPERTY(NotReplicated)` state.

- On a client such a field arrives default-constructed, so the comparison fails **identically every time** — not
  intermittently.
- A UI gate built on one disables a legal action with no explanation.

The rule:

- Gate unreplicated terms on `HasAuthority()`.
- Let the client's answer be the **permissive** one.
  - A predictive UI may over-offer, because the server re-derives before mutating.
  - It must never under-offer.

## Refusals reaching a client

`Client_ReportRefusal` carries an `FSnapRefusal` back to the owning client, which raises `OnActionRefused`.

- It ships on the inventory, equipment and hotbar components.
- See [refusals](refusals.md).

⚠ `DispatchRefusal` returns early when the controller is not local.

- A refusal raised on the server *for* a remote client must not try to show that server a window.

## Multicasts on a listen server

A `NetMulticast` event invoked on the server runs its body **locally as well as remotely**.

- On a listen server — which is every ordinary Play-in-Editor session — that doubles anything the body does.
- Gate a multicast body on `HasAuthority()` and wire only the remote branch.

In an actor **component** Blueprint, `Switch Has Authority` cannot be used: it expands to `Actor::HasAuthority` and has
no Target to bind.

- Use `Get Owner → HasAuthority → Branch`.

## Testing

Ordinary Play-in-Editor is a **listen server**, so the host's UI runs on authority.

- A whole class of defect — a client-only gate, a FastArray removal desync, a refusal that never arrives — cannot appear
  there.
- Use **2 players, listen server** and drive the *non-hosting* client.
- Treat "verified in PIE" as covering server logic only.

The plugin's network suite checks three claims on this page automatically — owner-only contents, Everyone contents and a
trade's copy.

- It checks them against a listen server and, opted in, a dedicated server.
- It checks the removal resync above for the equipment list only.
  - Its stat-total test reads the total a client's listener heard last after an unequip.
  - A break that removed the equipment list's resync left that at the removed item's value.
- The inventory list's resync is not checked.
- See [Testing and diagnostics](testing-and-diagnostics.md).

⚠ `PIE_Client` net mode runs the server as a dedicated-server instance.

- In a project whose online subsystem cannot create one for a dedicated server (an EOS project without dedicated-server
  credentials, for example) it stops the editor when play starts.

## See also

- [Trade](trade.md) — the most network-sensitive feature
- [Persistence](persistence.md)
- [Refusals](refusals.md)
