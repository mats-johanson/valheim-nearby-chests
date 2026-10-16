# Nearby Chests

[![Build](https://github.com/timothydodd/valheim-nearby-chests/actions/workflows/build.yml/badge.svg)](https://github.com/timothydodd/valheim-nearby-chests/actions/workflows/build.yml)

A client-side [BepInEx](https://github.com/BepInEx/BepInEx) mod for Valheim. Craft, build and fuel
stations from the chests around you, and put everything away with one button. Nobody else on the
server needs it.

## What it does

- **Craft and build from nearby chests.** Recipes, upgrades and hammer builds count materials in
  chests within 30 m. What you carry is spent first, then the closest chests. The crafting panel and
  build menu show what you have next to what's needed (`312/5`), chests included.
- **Feed stations from chests.** Coal and ore into smelters and kilns, wood into fires, fuel into
  cooking stations and ovens. One press adds one item, the same as vanilla. Food and mead stay
  manual, because the mod would be picking your dinner.
- **Stack puts everything away.** Press **Stack** with a chest open, or hold **E** on a chest, and
  each item goes to every chest within 15 m that already holds it. A new item joins the chest with
  the most similar things (metals with metals, hides with hides, same biome), or takes an empty
  chest, or shares a chest holding one related group. Anything with nowhere to go stays with you,
  unless `FallbackToAnyChest` is on. Chests that received items get merged and sorted.
- **Tidy a messy chest.** The three-bar icon left of *Place stacks* works out what the open chest is
  mostly for, sends everything else to its proper chest, and pulls in strays that belong here.
- **Ignore a slot.** Middle-click a slot in your inventory and Stack leaves it alone. A small pin
  marks it; middle-click again to clear it. Marks are saved with your character.

  ![Inventory with pinned slots](docs/ignored-slots.png)
- **Pull build materials.** With a piece selected on your hammer, **Ctrl + left click** pulls its
  materials from nearby chests into your inventory, for building out of range. Each press adds one
  more piece's worth.
- **Stack and tidy with one key.** Bind `StackAndTidyKey` and a press stacks your inventory and
  tidies every chest in range, with no chest open. Off by default.
- **Choose which chests it uses.** Turn on `OnlyUseMarkedChests` and each chest window gets an
  **Include in NearbyChests** checkbox. Only ticked chests count, for crafting and stacking alike.
  The tick is saved on the chest, so other players with the mod see it too. Off by default.
- **Keep empty chests empty.** Turn on `DontFillEmptyChests` and Stack never starts a new chest on
  its own. Off by default.

**Stays with you:** food, meads and potions (raw ingredients still get put away), ammo, equipment,
your hotbar and anything equipped. Each has its own setting.

**Never touched:** the Obliterator, gravestones, dungeon loot chests, chests another player has
open, chests you can't open yourself, and unticked chests when `OnlyUseMarkedChests` is on. Stack
on the Obliterator or a gravestone does the vanilla stack, and the Tidy button is hidden there.

## Install

**With a mod manager:** install [Nearby Chests from Thunderstore](https://thunderstore.io/c/valheim/p/TeamRobo/Nearby_Chests/)
with r2modman or Thunderstore Mod Manager. BepInEx comes along as a dependency.

**By hand:** download a zip from [Releases](https://github.com/timothydodd/valheim-nearby-chests/releases).
`NearbyChests-x.y.z-with-BepInEx.zip` includes BepInEx; `NearbyChests-x.y.z.zip` is the mod alone.
Extract it straight into your Valheim folder, the one with `valheim.exe` in it, so that
`winhttp.dll` and the `BepInEx` folder sit right next to the exe. One folder too deep and nothing
loads. To check, look for `Nearby Chests x.y.z loaded` in `BepInEx\LogOutput.log`.

To uninstall, delete `BepInEx\plugins\NearbyChests`.

## Settings

After the first launch, settings are in `BepInEx\config\NearbyChests.cfg`.

| Section  | Setting              | Default          | What it does |
|----------|----------------------|------------------|--------------|
| General  | OnlyUseMarkedChests  | false            | Only use chests you've ticked **Include in NearbyChests** on. Applies to crafting, building, station feeding, Stack and Tidy. |
| General  | IncludeCartsAndShips | false            | Also use cart and ship storage. |
| Crafting | CraftingRange        | 30               | How far a chest can be for crafting, upgrading and building (3–60 m). |
| Crafting | CraftFromChests      | true             | Use chest materials at crafting stations. |
| Crafting | BuildFromChests      | true             | Use chest materials when building. |
| Crafting | StationsFromChests   | true             | Take fuel and ore from chests when feeding smelters, kilns, fires and cooking stations. |
| Crafting | ShowAvailableCounts  | true             | Show materials as have/needed in the crafting panel and build menu. |
| Stacking | StackingRange        | 15               | How far a chest can be for Stack, Tidy and sorting (3–60 m). |
| Stacking | StackAndTidyKey      | None             | Key that stacks your inventory and tidies every chest in range, with no chest open. Re-sorts every chest in range. |
| Stacking | StackToNearby        | true             | Stack sends items to every nearby chest. Off gives you the vanilla button back. |
| Stacking | KeepHotbar           | true             | Never stack items from the top row. |
| Stacking | ExcludeFood          | true             | Never stack food, meads or potions. Raw ingredients still stack. |
| Stacking | ExcludeAmmo          | true             | Never stack arrows, bolts, bait or other ammo. |
| Stacking | ExcludeEquipment     | true             | Never stack weapons, armor, shields, tools, torches, utility items or trinkets. |
| Stacking | DontFillEmptyChests  | false            | Never put items into a completely empty chest automatically. |
| Stacking | PlaceUnassignedItems | true             | Items with no home go to a chest of similar items, or an empty chest. |
| Stacking | UnassignedItemTypes  | Material,Trophy  | Item types placed even when not in the groups file. Options: Material, Trophy, Consumable, Ammo, AmmoNonEquipable, Fish, Misc. |
| Stacking | FallbackToOpenChest  | false            | With no similar or empty chest, use the open chest instead of leaving items with you. |
| Stacking | FallbackToAnyChest   | false            | If an item still has nowhere to go, put it in any nearby chest with room. |
| Stacking | SortAfterStack       | true             | Sort and merge chests that received items. |
| Stacking | ShareChests          | true             | Let a one-group chest take a second, related group when there's no empty chest. |
| Stacking | TidyButton           | true             | Show the Tidy button in the chest window (restart to apply). |
| Stacking | TidyGathers          | true             | Tidy also pulls stray items of the open chest's category in from other chests. |
| Stacking | IgnoreSlots          | true             | Middle-click a slot to mark it ignored; Stack never moves what's in it. |
| Building | PullBuildMaterials   | true             | With a piece selected, the key below pulls its materials from nearby chests. |
| Building | PullBuildMaterialsKey | Mouse0 + LeftControl | The key that pulls materials. Default is Ctrl + left click. |
| Building | PullBuildSets        | 1                | How many pieces' worth of materials each press adds (1–50). |

### Item groups

Which items count as "similar" lives in `BepInEx\config\NearbyChests.groups.txt`, one line per
group:

```
Metals = CopperOre, Copper, CopperScrap, TinOre, Tin, Bronze, ...
Cooked = Cooked*, BakedPoteitr, BoarJerky, Bread, ...
Trophies = Trophy*
```

Names are item prefab names (the ones the `spawn` console command uses) and aren't case sensitive.
A trailing `*` matches anything starting with that text. If an item is listed twice, the first group
wins. Edits apply the next time you stack. Delete the file to get the latest defaults back.

## Multiplayer

Only you need the mod. Before changing a chest it takes network ownership, the same way the game's
own *Take All* does, and it skips any chest another player has open. If another player changes a
chest in the same moment you craft from it or stack into it, one of the changes can be lost.

## Contributing

Bug reports and PRs are welcome. See [CONTRIBUTING.md](CONTRIBUTING.md) for the ground rules and
[docs/development.md](docs/development.md) for building, releasing and how the mod works inside.

## License

[MIT](LICENSE)
