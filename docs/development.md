# Development

## Building

You need the .NET SDK (6 or newer) and a Valheim install to compile against. The game client or the
free dedicated server both work.

The game's DLLs aren't in this repo, so tell the build where Valheim is, one of:

- On the command line:
  ```
  dotnet build -c Release -p:ValheimDir="D:\Steam\steamapps\common\Valheim"
  ```
- In a git-ignored `ValheimDir.props` next to `NearbyChests.csproj`:
  ```xml
  <Project>
    <PropertyGroup Condition="'$(ValheimDir)' == ''">
      <ValheimDir>D:\Steam\steamapps\common\Valheim</ValheimDir>
    </PropertyGroup>
  </Project>
  ```
- Nothing, if Valheim is at the default Steam location
  `C:\Program Files (x86)\Steam\steamapps\common\Valheim`.

The DLL lands in `bin/Release/NearbyChests.dll`. Copy it over the installed one to test. To make the
release zips:

```
python3 build/package.py 1.0.0 path/to/BepInExPack_Valheim.zip
```

`lib/` holds `BepInEx.dll` (LGPL-2.1) and `0Harmony.dll` (MIT) from BepInExPack_Valheim so the
project builds without BepInEx installed. `assembly_valheim` is referenced with
[BepInEx.AssemblyPublicizer](https://github.com/BepInEx/BepInEx.AssemblyPublicizer), so private
game members are accessible without reflection. After a Valheim update, rebuild: if the game changed
something the mod relies on you get a build error rather than a mod that silently misbehaves.

## CI and releases

On every push and pull request, [the build workflow](../.github/workflows/build.yml) downloads the
Valheim dedicated server with SteamCMD, builds against its DLLs, and uploads the zips as artifacts.
The Thunderstore package is uploaded unpacked, so the artifact download is itself the zip
Thunderstore accepts.

To publish a release:

1. Bump `Version` in `src/Plugin.cs`.
2. Add a `## <version> - <date>` section to `CHANGELOG.md`. The workflow uses it as the release
   notes and fails if the section is missing.
3. Commit, push, then tag and push the tag:
   ```
   git tag v1.0.1 && git push origin v1.0.1
   ```

The workflow builds the tag, checks it matches `Plugin.Version`, and attaches the zips to a GitHub
release. Upload the Thunderstore artifact by hand.

## How it works

The mod is C# with [Harmony](https://github.com/pardeike/Harmony) patches. Each feature lives in
its own file under `src/`.

### Crafting from chests (`CraftPatches.cs`)

Nothing in Valheim's crafting logic is rewritten. The game already has methods that check a
recipe's requirements (`Player.HaveRequirementItems`, `Player.HaveRequirements(Piece, ...)`,
`Player.GetFirstRequiredItem`, `InventoryGui.SetupRequirement`) and methods that spend them
(`Player.ConsumeResources`, `InventoryGui.DoCrafting`). The mod marks when the game is inside one of
those with a "count scope" and a "consume scope". While a scope is active, calls on the local
player's inventory are extended to nearby chests:

- `Inventory.CountItems` / `HaveItem` add chest totals.
- `Inventory.GetItem` falls back to a chest item.
- `Inventory.RemoveItem(name, ...)` takes whatever the player is short of from chests, nearest first.

Outside those scopes the inventory behaves exactly like vanilla, which keeps the patches small and
means recipe changes in game updates mostly just work.

`ChestFinder.cs` tracks every loaded container and answers "which chests near the player can I
use?". Results are cached for a second and invalidated whenever a chest's contents change, because
the crafting and build menus ask many times per frame. It also owns the eligibility rules: range,
player-placed, not a disposal, access and ward checks, not open by someone else, and, with
`OnlyUseMarkedChests`, ticked.

### Feeding stations (`StationPatches.cs`)

Adding fuel or ore is the game asking the player's inventory "do you have this?" then "take one",
the same calls the crafting scope already extends. The mod marks the station interaction
(`Switch.Interact`/`UseItem`, which covers smelters, kilns, windmills, spinning wheels, eitr
refineries, shield generators and the cooking station's fuel switch, plus `Fireplace` directly) and
lets those patches do the rest. The cooking station's add-food switch is skipped, and
`CookingStation` and `Fermenter` aren't patched, so food and mead stay manual. Ore is the exception:
the game finds it with `Inventory.GetItem(name)` and spends it with `RemoveItem(item, 1)`. When the
fallback hands back an item living in a chest, a patch on the instance overloads removes it from the
chest it came from.

### Stacking (`Stacker.cs`)

Replaces the game's stack action where it's triggered: `InventoryGui.OnStackAll` (the button) and
`Container.RPC_StackResponse` (holding Use). For each eligible item:

1. fill chests that already contain it,
2. fill the chest holding the most items from the same group (`ItemGroups.cs`), or, for items in no
   group, the catch-all chest (the one that's mostly ungrouped items),
3. fill an empty chest, which becomes that group's chest from then on,
4. share a chest that holds only one other group (`ShareChests`),
5. optionally fall back to the open chest (`FallbackToOpenChest`), then any chest with room
   (`FallbackToAnyChest`).

With the Obliterator or a gravestone open, both patches step aside and the vanilla stack runs.
Chests that received items are then merged and sorted by type, group order, name, quality and stack
size.

`DontFillEmptyChests` is a single guard in `MoveInto`, so it covers every path above and Tidy too.

### Tidy (`Tidier.cs`)

The button is a clone of the chest window's Stack button, made in an `InventoryGui.Awake` postfix,
resized to a square with a runtime-drawn icon (`TidyIcon`). Its `UIGamePad` shortcut is removed so a
controller press doesn't fire both buttons. It's positioned left of Stack the first time the window
is shown, using world-space corners so it lines up at any resolution or UI scale.

A chest's category is its most common group by stack count. Uncategorized items count as "junk",
and ties go to the group listed first. Items in the open chest that don't match its category move
to: chests of their category (dedicated first, then two-group chests holding it), an empty chest, a
one-group chest that becomes shared (closest group in file order first), then junk chests.

Then it gathers: items of the chest's own category are pulled out of every nearby chest whose own
category differs. Another chest of the same category is drained too, unless it holds more of the
category than the open one, so a split group merges into one chest and can't bounce back on the next
tidy. The mirror rule applies first: if another chest holds more of the open chest's category, the
open chest's share moves there.

A chest holding exactly two groups keeps its second group when there's no better home, so shared
chests don't bounce items back and forth. A junk chest never passes items on to another junk chest,
and uncategorized items never become a shared chest's second group.

### Food detection

An item counts as food when it's a `Consumable` that the game can produce: the output of an
`ObjectDB` recipe or of a `CookingStation`, `Fermenter` or `Smelter` conversion. Raw edibles aren't
produced by anything, so they stay ingredients. This works with modded recipes too. "Isn't a recipe
ingredient" was tried first and fails, because cooked dishes like deer stew are feast ingredients.

### Ignored slots (`IgnoredSlots.cs`)

A postfix on `InventoryGrid.UpdateGui` adds a pin image to each slot element the first time it sees
it and hooks the element's `UIInputHandler.m_onMiddleClick`, which the game leaves unused. The slot
list is kept in the character's custom data (`Player.m_customData`), so it saves with the character,
and `Stacker` skips any item whose grid position is marked.

### Pulling build materials (`BuildPull.cs`)

A prefix on `Player.UpdatePlacement`. When the shortcut is pressed in build mode it reads the
selected piece's requirements, works out how many pieces you could already build from what you
carry, and moves the shortfall for one more from the nearest chests. It then clears the frame's input
flag so the click doesn't also place the piece.

### Marked chests (`ChestSelection.cs`)

The tick is a boolean in the chest's own ZDO, so it saves with the world and syncs to other players
through the game's normal networking with no server mod. It's only written while the chest is open,
when the game has already given you ownership. The checkbox is added to the chest window in an
`InventoryGui.Awake` postfix and reread only when the open chest changes, not every frame. With the
option on, `ChestFinder` skips unticked chests and the open chest is checked before Stack or Tidy
runs.

### Stack-and-tidy key (`Plugin.cs`)

Runs from the plugin's `Update` while the player can take input (no menu, chat or chest open). It
stacks with no open chest, so the open-chest fallback doesn't apply, then tidies each chest in
stacking range nearest-first. Valheim's input layer ignores keycodes above 349, so F16 to F24 are
read through Unity's input system directly, for this key and `PullBuildMaterialsKey`.

## Multiplayer

The mod is client-side. Before changing a chest it claims network ownership, the same way the game's
own "Take All" does, and skips any chest another player has open. There is a small timing window: if
another player changes a chest in the same moment you craft from it or stack into it, one of the
changes can be lost.
