# Nearby Chests

Craft, build and fuel stations from the chests around you, and put everything away with one button.
Client-side only: nobody else on the server needs it.

## What it does

- **Craft and build from your chests.** Anything within 30 m counts as if you were carrying it.
- **Feed smelters, kilns and fires from your chests.** One press, one item, same as vanilla.
- **Stack puts everything away.** Press **Stack** on any chest and each item goes to the chest it
  already lives in. New items join similar things, or take an empty chest.
- **Tidy a messy chest.** The three-bar icon next to *Place stacks* sends what doesn't belong to
  the right chest, and pulls in what does.
- **Ignore a slot.** Middle-click a slot in your inventory and Stack leaves it alone.
- **Pull build materials.** **Ctrl + left click** with a piece selected on your hammer grabs its
  materials from nearby chests.

Your food, ammo, gear and hotbar stay with you. The Obliterator, gravestones, dungeon chests and
other players' open chests are never touched.

![Inventory with pinned slots](https://raw.githubusercontent.com/timothydodd/valheim-nearby-chests/main/docs/ignored-slots.png)

## Optional, off by default

Turn these on in `BepInEx\config\NearbyChests.cfg`:

- **`StackAndTidyKey`**: one key stacks your inventory and tidies every chest in range.
- **`OnlyUseMarkedChests`**: tick **Include in NearbyChests** in a chest window and only ticked
  chests are used.
- **`DontFillEmptyChests`**: Stack never starts a new chest on its own.
- **`FallbackToAnyChest`**: items with no matching chest go into any chest with room.

Every setting, and the item groups file that decides what counts as "similar", is explained in the
[full guide](https://robododd.com/nearby-chests/).

## Links

- [Full guide](https://robododd.com/nearby-chests/)
- [Source and issues on GitHub](https://github.com/timothydodd/valheim-nearby-chests)
- [Changelog](https://github.com/timothydodd/valheim-nearby-chests/blob/main/CHANGELOG.md)
