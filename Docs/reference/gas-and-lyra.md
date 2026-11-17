# GAS and Lyra

How the plugin uses the Gameplay Ability System (GAS), what a build without it keeps and loses, and how the plugin works
with Lyra.

**On this page:**

- [Gameplay Ability System](#gameplay-ability-system)
  - [Resolving the ability system component](#resolving-the-ability-system-component)
  - [What equipment grants](#what-equipment-grants)
  - [Consumables](#consumables)
  - [Equip requirements](#equip-requirements)
  - [Two GAS traps worth knowing](#two-gas-traps-worth-knowing)
  - [Without GameplayAbilities](#without-gameplayabilities)
- [Lyra](#lyra)
- [See also](#see-also)

## Gameplay Ability System

`GameplayAbilities` is **optional**.

- **The plugin uses it for:**
  - What equipment grants.
  - Consumables that run an ability or an effect.
  - Everything that reads a gameplay tag or an attribute on the player's ability system.
- **It never turns it on for you** — the `.uplugin` does not name it. So enable it in your own `.uproject`.
- The runtime module reads that choice, defines `WITH_SNAP_GAS` as `1` or `0`, and compiles
  `Source/SnapInventorySystem_RequiresGAS/` only when it is `1`.
- What a build without it keeps and loses is under [Without GameplayAbilities](#without-gameplayabilities).

⚠ The choice is read when the plugin is **compiled for your project**.

- A copy precompiled by `RunUAT BuildPlugin` (the editor's *Package...* button) was compiled in a host project that enables
  only this plugin.
  - So it is the build without `GameplayAbilities`, whatever your `.uproject` says.
  - A storefront's *Install to Engine* copy is expected to be one.
  - An installed engine never recompiles an engine-installed copy.

**For the features on this page:**

1. Put the plugin in your project's own `Plugins/` folder, where it is compiled with your settings.
   - In a Blueprint-only project, it is compiled by the editor when the project opens.
2. Compile it with `GameplayAbilities` enabled.
   - After you enable it, a C++ project's next build from your IDE recompiles the plugin. Restarting the editor does not.
   - In a Blueprint-only project, delete the plugin's `Binaries` and `Intermediate` folders and reopen.

**A copy compiled without it says what it lacks:**

- Its `Build.cs` answers `WITH_SNAP_GAS=0` for a precompiled copy.
- A C++ project's build prints a warning when the project enables `GameplayAbilities`.
- Every project that enables it gets one `Warning` at startup and a Setup Check note (`DependencyNotCompiled`).

The README's **Known limitations** section has what was measured.

### Resolving the ability system component

```cpp
UAbilitySystemGlobals::GetAbilitySystemComponentFromActor(Owner)
```

That checks `IAbilitySystemInterface` on the actor and falls back to a component search. The plugin never casts to a
game-specific type, so:

- Any actor implementing the interface works.
- An actor with no ability system component still equips items, and the grants are skipped with a warning.

⚠ **Owner convention.** The equipment component is designed to sit on the actor that owns the ASC (the ability system
component) — commonly the PlayerState.

- A PlayerState has no world transform.
- So a spawned equipment visual is attached to the **controlled pawn's** mesh instead, resolved through
  `APlayerState::GetPawn()`.

### What equipment grants

- The `GrantedAbilitySet` fragment lists abilities and gameplay effects applied on equip and removed on unequip.
- Plus the rarity tier's own effect, where the fragment scales with rank.

**The first thing to test in your own project is symmetry:**

1. Equip and unequip the same item twice.
2. Check the ability and effect counts return to where they started.

**Rules for an equip-time effect:**

- **It must be HasDuration or Infinite, never Instant.** An Instant effect writes the attribute's base value and returns
  no handle.
  - So unequipping removes nothing, and the stat drifts with every equip.
  - Equipping logs a Warning naming the effect.
- **`ScaledEquipEffects`** (`FSnapItemScaledEffect`) applies a SetByCaller effect whose magnitude is `BaseMagnitude`
  plus `MagnitudePerRarityRank` times the instance's rarity rank.
  - `SetByCallerTag` must match the effect's modifier tag.
  - With a non-zero per-rank value, an item whose rarity has no usable rank applies nothing (logged at Verbose).
- **Leave `EffectAssetTag` unset unless something reads it.** Code of yours that removes effects by tag would remove the
  item's effect while it is still worn, and nothing is logged. The plugin itself removes by handle.

**Other grants that work the same way:**

- An ability granted this way is how the demo's weapons take over a click.
  - The iron sword grants `USnapDemoAttack_OneHanded`, which answers for the hand its item is worn in.
  - See [A weapon that changes what a click does](equipment.md#a-weapon-that-changes-what-a-click-does).
- An item set's tier `Effects` are applied the same way, on the same ability system, and removed by handle when the tier
  stops being active — see [Item sets](item-sets.md).
- An item's `StatContributions` rows are applied the same way too, each through the effect the **Stat Effect Map** binds
  to its stat and unit — see [Equipment](equipment.md#stats-an-item-declares-itself).
  - The map must bind an authored effect.
  - A `GameplayEffect` built at runtime replicates to a client with no definition, and the client's attribute falls
    behind the server's by that effect's modifier.

### Consumables

- The same fragment carries `ConsumeAbilities` and `ConsumeEffectClasses` — the one-shot payload spent when a unit is
  consumed.
- `ConsumeAbilityTag` (**Snap Inventory System - Inventory → Consume**) names the ability your project activates.

**How the payload runs:**

- `ConsumeAbilities` run first, then `ConsumeEffectClasses`.
- If the abilities loaded and none of them activated, the consume is refused: no effect applies and no unit is spent.
- If none of the abilities could be loaded, that is read as a packaging fault, not a refusal: the effects still apply,
  and a Warning says so.
- Each consume ability must be server-runnable: `GiveAbilityAndActivateOnce` refuses a NonInstanced or LocalOnly
  ability.
- The spec's `SourceObject` is the item **definition**, not the instance (an equip-time grant gets the instance). So cast
  `GetCurrentSourceObject()` to `USnapItemDefinition` in a consume ability.

⚠ Consuming is the one place the plugin cannot supply a default: the ability it activates is **yours**.

- `bBindConsumeKeyRaw` (**Snap Inventory System - UI Core → Input → Keys**) ships off for that reason.

**A consumable does not have to use the ability system at all.**

- A fragment of your own can declare and perform the consume itself through `DeclaresConsumeGrant`,
  `CanApplyConsumeGrantTo` and `ApplyConsumeGrantTo`
  ([items and fragments](items-and-fragments.md#a-fragment-that-makes-its-item-a-consumable)).
- With both, the effect lands first and the fragment's `ApplyConsumeGrantTo` runs after it.

### Equip requirements

`USnapItemFragment_Requirements` reads the owner's ability system:

- Its gameplay tags.
- Its attribute values.
- The attribute named by `RequirementLevelAttributeName`, as a level.

**About `RequirementLevelAttributeName`** (**Snap Inventory System - Items → Equipment → Requirements**)**:**

- That setting holds an attribute's property name.
- It is resolved on the live ability system each time it is asked, because a settings object cannot hold a reference to a
  game module's attribute set.
- It ships empty, which ignores `MinLevel`.

**Everything it reads must replicate.** The evaluator answers on the client as well as the server — that is what lets the
equipment UI refuse before the server does.

- Grant required and blocked tags with a **gameplay effect**.
  - A loose tag added on the server stays on the server.
  - The client then offers or refuses an equip the server answers differently.
- Attributes replicate with their set, as usual.

A worn item is never re-checked, so an effect expiring does not take anything off.

**Without `GameplayAbilities`,** there is nothing to read, and no term is in force.

- An item with requirements equips freely, and one Warning per owner and item says so.
- See [Equipment](equipment.md#requirements).

### Two GAS traps worth knowing

- **A duration-based effect moves an attribute through the aggregator**, which does not run `PostGameplayEffectExecute`.
  - A health set that only broadcasts from there misses every equipment change.
  - The delegate that does fire is `ASC->GetGameplayAttributeValueChangeDelegate(Attribute)`.
- **Applying an effect's class-default object sets the level and nothing else.**
  - An effect whose magnitude is `SetByCaller` applied that way yields a valid handle and a modifier of **zero**.
  - Build a spec, set the named magnitude, then apply.

### Without GameplayAbilities

**A project that leaves `GameplayAbilities` off still builds.** That is because everything that names an ability system type
is behind `WITH_SNAP_GAS` or in the `_RequiresGAS` folder.

**Kept, unchanged:**

- The bag and its sibling containers.
- Equipment slots.
- The vendor, trading, crafting and loot.
- Storage items and the spatial grid.
- Saving and loading.
- Interaction.
- Durability.
- Carry weight and its direct speed penalty.
- The death drop.
- The hotbar's `Consumable` and `Event` squares.

The crafting, interaction, persistence, vendor and world-lifecycle source has no `WITH_SNAP_GAS` in it at all.

An item fragment's `OnEquipped` and `OnUnequipped` hooks are there either way. That is how the demo's weapons change a
click with no abilities at all (`USnapDemoWeaponFragment`, see
[Equipment](equipment.md#a-weapon-that-changes-what-a-click-does)).

**What changes:**

| Feature | Without `GameplayAbilities` | Use instead |
|---|---|---|
| What equipment grants | Abilities and effects are not applied on equip or removed on unequip: `USnapItemFragment_GrantedAbilitySet` is not compiled | — |
| A rarity tier's effect | `FSnapItemRarityEntry::GrantedEffect` is not applied; the tier's color and name are unaffected | — |
| An item set's bonuses | No tier effect is applied; the worn pieces are still counted | `USnapEquipmentComponent::GetWornPieceCountForSet`, for a reward of your own |
| A consume payload | `ConsumeEffectClasses` and `ConsumeAbilities` do not exist, so an item whose only payload they were is not offered Consume | A fragment's `USnapItemFragment::DeclaresConsumeGrant`, `CanApplyConsumeGrant` and `OnConsumed` ([items and fragments](items-and-fragments.md#a-fragment-that-makes-its-item-a-consumable)) |
| Consuming through `ConsumeAbilityTag` | The bag spends the unit directly, with no ability to present it | — |
| Abilities while held | A custom container's `bActivateHeldItemAbilities` grants nothing, and the shipped Spells container is not attached. See [Abilities while held, without GameplayAbilities](#abilities-while-held-without-gameplayabilities) | — |
| A hotbar `Ability` square | Pressing it does nothing | An `Event` item and `USnapItemFragment_Hotbar::OnHotbarEventActivated` |
| Hotbar cooldown and charges | Both defaults answer zero, so no overlay draws | Override `USnapHotbarComponent::GetSlotCooldown` and `USnapHotbarComponent::GetSlotCharges` |
| Equip requirements | No term is in force — tags, attribute minimums and `MinLevel` are all read off an ability system — so a gated item equips freely, with one Warning per owner and item | — |
| A slot locked by `UnlockTag` | Stays locked | `USnapEquipmentComponent::UnlockSlot` |
| Wear from damage | Nothing listens for a fall in `DamageWearHealthAttributeName` | Call `USnapDamageWearComponent::ApplyDamageWear` from your own damage code |
| The overweight gameplay effect | `OverweightGameplayEffect` does nothing, silently; it ships empty, and the direct speed scale is unaffected | The direct route ([carry weight](condition-and-weight.md#carry-weight)) |
| The death tag | Not watched, and `USnapInventoryComponent::IsOwnerDead` answers no by default; an inventory screen that is already open is not closed on death | `NotifyOwnerDied` / `TriggerDeathDrop`; override `IsOwnerDead`; `USnapInventoryHUDComponent::SetInventoryVisible` ([death and loss](death-and-loss.md#without-the-gameplay-ability-system)) |
| `ScreenAbilityBlockMode` `LooseTag` | Stops nothing, with one Warning per player; `AbilitiesUsableWhileScreenOpen` does nothing | `ConsumeClicksAtScreen`, or `None` |
| Tooltip stat and grant lines read from effects | Not drawn: both are read from the granted-ability fragment. Affix lines are unaffected | A `StatContributions` fragment: its rows draw stat lines and feed `GetEquippedStatTotal` in every build ([equipment](equipment.md#stats-an-item-declares-itself)) |
| The demo | Its classes are compiled either way and read one stats layer, `USnapDemoStatsComponent`. See [The demo, without GameplayAbilities](#the-demo-without-gameplayabilities) | [The demo's two routes](demo-map.md#the-demos-two-routes) |

#### Abilities while held, without GameplayAbilities

- **A custom container's `bActivateHeldItemAbilities` grants nothing** (one Log line says so).
  - It only makes the container one the Ability bar draws from.
  - `USnapContainerHeldAbilities` is not compiled.
- **The shipped Spells container is not attached:** The settings' attachment rows leave it out.
  - The shipped container registry, one asset for both builds, still names it, and that row attaches nothing.
  - Automatic attachment and `EnsureSnapInventoryComponents` say so at Log each time they meet that row, never as a
    Warning.
  - The registry still validates clean, and the Setup Check does not report it.
  - Only a class on `USnapFoundationSettings::GetGameplayAbilitiesOnlyComponentClasses` is excused this way.
- **The example screens bind their Spells panel to that missing container,** so the panel hides itself rather than drawing
  an empty `0/0` grid ([inventory and containers](inventory-and-containers.md)).

#### The demo, without GameplayAbilities

- **Its classes are compiled either way** and read one stats layer, `USnapDemoStatsComponent`.
- **So the whole map plays on the layer's native route:**
  - Health, damage, death and respawn.
  - The HUD, the pads, `K` and the training dummies.
  - Attacks through the attack component.
  - Worn items' stats and the iron set's bonuses.
  - The potions, and the ring's and amulet's passives.
  - The five spells (the demo's spell caster; a Spells container of the demo's own).
- **The DEMO RUNS ON pad** says the route is fixed.
- **Not compiled:** `USnapDemoAttributeSet`, the attack abilities, the spells' abilities and effects, and the passive
  ability and effect the jewelry named before (no example item names them now).
- **Opening the map logs five expected load warnings** — the ability system and attribute set that the demo's player
  state and training dummies make in a build with it.

#### How far this was checked

- A project generated from this plugin was built for the editor with `GameplayAbilities` off and then on.
- All four modules compiled and linked.
- The automation suite found **544** tests without it and **683** with it, and the network suite fourteen without it and
  sixteen with it.
  - With the test item under `Content/Tests` copied in and `Snap.Tests.AllowDedicatedServerPIE` at 1, every one of them
    ran and passed, the dedicated-server ones included.
  - As generated, the tests that need the item or the variable warn that they skipped, as in any packaged copy.
- The door tests passed in both: `SnapInventory.Death.AProjectCanSayItsPlayerIsDead`, `.TheScreenAsksTheSameDoor`,
  `SnapInventory.Consume.AFragmentOfYourOwnMakesAConsumable`, `.OnlyADeclaringFragmentIsOffered` and
  `.ARefusedGrantCostsNothing`.
- A packaged Development and Shipping game was also made without it and played with real input, from the demo map run by
  copies of the demo classes with their ability system taken out — see **Known limitations** in the plugin README.
- No dedicated-server build was made without it.

---

## Lyra

- The plugin works with Lyra out of the box and depends on **no Lyra module**.
- What Lyra gives you is the Modular Gameplay receiver classes.

### What "works out of the box" means

- The seven shipped attachment rows (six without `GameplayAbilities`) name `ModularPlayerState`, `ModularPlayerController`
  and `ModularCharacter` **by class path**.
  - That is because the plugin must not link against `ModularGameplayActors` — a sample plugin most projects do not have.
- `UGameFrameworkComponentManager` matches receivers by path, so:
  - A project that has those classes gets the components automatically.
  - A project that does not gets inert rows and **one Warning at first login** saying so.

See [QUICKSTART](../QUICKSTART.md) path B for what to do in the second case.

### Death and dropping

- The `bDropEverythingOnDeath` setting drops the bag on death.
- It needs the bag's own `DeathTagName` (under `[/Script/SnapInventorySystem.SnapInventoryComponent]` in `DefaultGame.ini`, beside `bDropEverythingOnDeath`) set to the tag your project raises while dead — for Lyra that is `Status.Death`. The Foundation page's `DeathTagName` does not arm the death drop.
- With the flag on and the tag unset, nothing drops and the plugin logs a warning naming the setting.

**What a death costs is a policy class.**

- A project without the Gameplay Ability System runs the same death drop through `NotifyOwnerDied` and says who is dead by
  overriding `IsOwnerDead`.
- See [Death and loss](death-and-loss.md).

### Things Lyra does *not* give you

- The consume ability. That is your project's, named by `ConsumeAbilityTag`.
- The currency.
  - `CounterTag` and `VendorCurrencyTag` both default to the plugin's own `SnapInventorySystem.Counter.Currency`.
  - If your project has its own, set **both**.
- Any of the example content. It is there to be replaced.

### Gameplay tags

- The plugin ships its own tag source at `Config/Tags/SnapInventorySystemTags.ini`.
- It is registered through a search path the module adds at startup — a plugin's own `Config/Tags` is not auto-scanned.

⚠ If you edit that file, write the key **bare**: `GameplayTagList=(...)`, never `+GameplayTagList=(...)`.

- A loose tag file is loaded standalone rather than combined into a config branch.
- So a `+` prefix makes every tag in the file silently fail to register — with no error and no warning.

## See also

- [Equipment](equipment.md)
- [Items and fragments](items-and-fragments.md) — `GrantedAbilitySet`
- [QUICKSTART](../QUICKSTART.md)
