# Contributing

Bug reports, group fixes and features are all welcome. A few things that make them easy to take:

## Pull requests

- **One feature or fix per PR.** Small PRs get reviewed and merged quickly; a PR bundling three
  features gets asked to split.
- **Leave `Version`, `CHANGELOG.md` and the README to the maintainer.** They're done at release,
  so a PR that changes them conflicts with every other open PR. Do add or update the config
  description for any setting you add.
- **New behaviour goes behind a setting, off by default,** unless it's clearly a fix. The mod's
  defaults should keep working the same way for people who update without reading the notes.
- **Say what you tested in game.** CI only proves it compiles. Note whether you tried it in single
  player, on a server, or both, and what you didn't try.
- **Match the code around you.** Short methods, comments that explain why, no new dependencies
  without a reason.

## Building

See [Building](README.md#building) in the README. In short: .NET SDK and a Valheim install, then
`dotnet build -c Release -p:ValheimDir=...`. CI builds every PR against the free dedicated server.

## Reporting a bug

Use the bug report template. The BepInEx log (`BepInEx\LogOutput.log`) is the most useful thing
you can attach; most problems are visible there.
