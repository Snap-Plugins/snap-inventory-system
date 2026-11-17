# Design notes

Why the plugin is built the way it is. These are for someone extending or modifying it.

A buyer who just wants an inventory on screen wants [QUICKSTART](../QUICKSTART.md) instead.

**On this page:**

- [Principles the code actually follows](#principles-the-code-actually-follows)
- [Where the archaeology went](#where-the-archaeology-went)
- [Documentation that cannot quietly go wrong](#documentation-that-cannot-quietly-go-wrong)
- [Content that opens outside Lyra](#content-that-opens-outside-lyra)
- [Content text spelled the American way](#content-text-spelled-the-american-way)
- [Content saved by an engine with a changelist](#content-saved-by-an-engine-with-a-changelist)
- [Generated documentation](#generated-documentation)

## Principles the code actually follows

**One question, one answer.** Wherever a UI grays a button and a server refuses an action, both ask the same planner.

- `DescribeCraft` and `CraftRecipe`, `PlanSellBundle` and `SellToVendor`, `AcceptsItemWithReason` and `AddItem` are each
  one decision with two callers.
- Two paths that can disagree about the same question is the defect this plugin has paid for most often.

**The server decides; the client may over-offer.**

- A predictive UI that shows an action the server then refuses is recoverable — the player sees a reason.
- A UI that hides an action the player is entitled to is not: there is nothing to click and nothing to explain.
- So where a predicate cannot be computed on a client, the client's answer is the permissive one.

**Absence is the refusal.**

- An item with no `Hotbar` fragment cannot go on a bar.
- An item with no `Equippable` fragment cannot be worn.
- A widget contract left blank switches that feature off.
- Nothing has to be listed anywhere to be excluded, and no exclusion list can go stale.

**A transition is not a state.** The `bDestroyEmptiedStorageItems` option destroys a pouch that *held something and now
holds nothing*, never one that is merely empty.

- Testing the state instead would delete a pouch the instant it was picked up.

**When two failures are not symmetric, default to the recoverable one.**

- The cost of `bDestroyEmptiedStorageItems` shipping off is clutter.
- The cost of it shipping on was silent deletion.

**Name things by role, not by class.** The main bag is the container whose `bIsMainInventory` is true, not the one whose
class matches exactly and not the first subclass found.

- A marker can be moved; a class test cannot.
- An `IsA` test silently picks whichever component is first.

**An unexplained gap is a lead.** Where a measurement disagrees with an arithmetic prediction, the gap is the defect,
not a rounding note.

## Where the archaeology went

The plugin was built across many numbered development sessions, and the shipped comments used to carry those numbers,
dates, and references to a private engineering-notes file.

- That is history about *how* the plugin was made, and a buyer reads the source to learn what it *does*.
- The reasoning stayed in the source.
- The markers moved to `Docs/design-notes/provenance/` in the repository, one file per subsystem, each line at the
  number it held.
- The full annotated history is also in the repository's own commit history.

The same goes for design-decision bookkeeping: numbered decision ids, who decided a thing, section references into the
private notes, review bookkeeping and the name of the game the plugin was built inside.

- Each comment keeps the decision and its reason, in plain words, and drops the paperwork.
- `Tools/check_header_hygiene.py` fails if a marker or a decision note comes back.
- `Tools/` is in the plugin's source repository, not in the packaged plugin.

## Documentation that cannot quietly go wrong

Three more checks sit beside it, because the ways documentation rots are all mechanical:

- `Tools/check_doc_links.py` resolves every `/SnapInventorySystem/...` path and every link between documents.
- `Tools/check_doc_symbols.py` resolves every backticked class, member and asset name against `Source/` and `Content/`.
  - It found a QUICKSTART step naming a setting that never existed, and a class documented under its FILE name rather
    than its own.
- `Tools/generate_docs.py --check` fails when an appendix is stale.

`Tools/install-hygiene-hook.ps1` wires these four, the Lyra import check and the content text check below, the size
limits (`check_sizes.py`), the layer map (`check_layers.py --enforce`), the UICore signatures
(`check_uicore_signatures.py`) and gate 6 (`check_gate6.py`: no CoreRedirects, no shipped content naming an old name or
importing a retired class's function, no unreadable package) to a pre-commit hook: ten checks.

- It is local to your clone.
- `-Remove` uninstalls it and `git commit --no-verify` bypasses it once.

## Content that opens outside Lyra

The plugin is developed inside a Lyra-based project, and its example content has to open in a project that is not.

- An asset saved here can pick up a reference to Lyra's game module without anything here noticing, because here that
  module is always loaded.
- The demo map picked one up when a save gave it Lyra's world settings, and a project without the module could not
  resolve it.

`Tools/check_content_imports.py` searches the bytes of every file under `Content/` for `/Script/LyraGame` and fails on a
hit.

- `Content/Tests/` is skipped, because `Config/FilterPlugin.ini` keeps it out of the packaged plugin.
- Its allow-list is empty and must only ever shrink: an entry that no longer hits is an error too, so a fixed asset
  cannot leave a stale exemption behind.
- A hit is fixed by removing the reference, not by allow-listing it.
  - For the demo map that meant re-saving it in a project without Lyra, because a save in this one puts Lyra's world
    settings back.

## Content text spelled the American way

`Tools/check_header_hygiene.py` holds the source, config and documents to American spelling. But an asset's own text —
an item's description, a widget's label, a material node's comment, an actor's label in a map — lives inside its
`.uasset` or `.umap`.

`Tools/check_content_text.py` reads those bytes:

- It reads the printable text runs of every file under `Content/` (plain and UTF-16).
- Each word is looked up in the same British list (a CamelCase name split into its words).
- `Content/Tests/` is skipped.
- The engine's own names that it spells the British way (`bWasCancelled`, and the ability task pin of the same words)
  are cut out first.
- A hit is fixed in the asset, saved on the launcher engine.

## Content saved by an engine with a changelist

This repository's engine is built from source and has no changelist.

- An engine without one writes an EMPTY engine version into every package it saves.
- Every Epic Games Launcher and Fab install has a changelist, and warns once for each such package it loads in the
  editor or a cook: `Asset has been saved with empty engine version` (LinkerLoad.cpp).
- So the shipped content is saved by a launcher engine of the plugin's own minor version — 1.0's by 5.8.2, which marks
  each package compatible with 5.8.0.
- **Any later save of a shipped asset in this repository puts the empty version back for that asset.**
- Content is edited here as usual; the last step before a release is to re-save every shipped package this repository
  saved since, on a launcher engine.

`Tools/check_content_saved_version.py` reads each package header under `Content/` (skipping `Content/Tests/`, which is
not shipped). It fails on:

- A package saved without a changelist.
- One marked compatible only with an engine newer than the `.uplugin`'s `EngineVersion`, which an older engine refuses
  to load.

It is a release check, not a commit hook: content is edited here first and re-saved afterwards.

**The re-save that works:**

1. Use a C++ project on the launcher engine with the plugin copied into its `Plugins/`, and `GameplayAbilities` and
   `CommonUI` enabled.
   - So the plugin is compiled from this tree and every class a package names exists.
2. Load every listed package there first and read the log.
   - A package whose class could not load would lose that export when it is saved.
3. Run the engine's own `UnrealEditor-Cmd <project> -run=ResavePackages -FILE=<list of package names> -OnlyUnversioned`.
   - `-OnlyUnversioned` touches only packages saved without a changelist.

What else a re-save changes, measured on 1.0's 130 packages:

- The Blueprint debug instructions the editor regenerates on load are not written.
- Member variables and default subobjects added in C++ since the last save are written with their defaults.
- A value an editor sets while loading (an actor Blueprint's editor-only hidden flag) is written into its class
  defaults.
- Empty placeholder thumbnails are dropped (every thumbnail with an image is kept).
- A map's static mesh components gain an empty per-LOD entry.

Apart from those LOD entries, none of these reads back differently when loaded.

## Generated documentation

Three appendices are generated from the headers by `Tools/generate_docs.py`:

- [Appendix: settings](../reference/appendix-settings.md)
- [Appendix: widget contracts](../reference/appendix-widgets.md)
- [Appendix: events](../reference/appendix-events.md)

A hand-written table drifts by the next change; a generated one cannot.

- The command `generate_docs.py --check` exits non-zero when they are stale, and prints how many entries each one holds.
- A count written here would drift the same way a hand-written table does.

The `*WidgetName` count is the number of `UPROPERTY` `FName` properties whose name ends in `WidgetName`, across
`Source/SnapInventorySystem/Public`.

- Counting textual occurrences instead gives a much larger number and counts function parameters and prose.
