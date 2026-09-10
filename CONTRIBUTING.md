# Contributing

Thanks for helping improve HassControl! This document covers what you need to
know to build, test, and submit changes.

## Project layout

- The widget source lives in `widget/` (this is also the VS Code workspace root).
- `widget/source/hass/` - Home Assistant communication (client, OAuth, entities).
- `widget/source/Entities/` - the entity views (card and list style).
- `widget/source/Menu/` - settings menus.
- `widget/monkey.jungle` - build configuration, including the memory-tier split described below.
- `widget/AGENTS.md` - coding conventions (naming, comments, entity-type checklist). Please read it before making code changes.

## Building

Use the Connect IQ SDK (`monkeyc`). In VS Code the build task
`monkeyc: fenix7x` builds the default target; from the command line:

```sh
java -jar <SDK>/bin/monkeybrains.jar -o bin/widget.prg -f monkey.jungle \
  -y <developer_key> -d fenix7x -w -l 0
```

## Memory tiers: always test both

Garmin devices fall into two memory classes for this widget:

- **high-mem (fullmem)** - most modern watches (e.g. fenix7x, edge1040).
  Full feature set: MDI icons, list view, select/input_number editing.
- **low-mem** - devices with a 64 KB widget heap (e.g. instinct2x, fenix5, fr245, vivoactive3;
  see `monkey.jungle` for the full list). These build with `excludeAnnotations = fullmem`, which strips icons, the list view, and all editing UI at compile time.

**Every change must be built and smoke-tested on at least one device of each class.** A feature that compiles on fenix7x can still break the low-mem build (excluded symbols) or push it over the memory budget (launch failure or runtime Out Of Memory errors). Check the code+data size with `--build-stats 0`; the budget for low-mem widgets is 65536 bytes.

### Low-mem testing requires release mode

Low-mem devices can only be tested with a **release build** (`-r` flag / release configuration). A debug build carries extra code that pushes the binary past the 64 KB heap budget, so the app runs out of memory and won't even start in debug mode on these devices. Do not debug-test low-mem; use release builds and verify behavior there.

## Testing the storage upgrade path

The widget caches entities in `App.Storage` under a versioned key, and the persisted format has changed between releases. A bad migration makes the app show "No entities configured" after an update - this has happened before, so **test the upgrade path whenever you touch anything that is persisted**.

Simulator procedure:

1. Build the **old** release (e.g. `git checkout v2.1.0`), run it in the simulator, and let it import and store entities.
2. Stop the app, but keep the simulator running so its storage persists.
3. Build the **new** version, start it in the *same* simulator session, and confirm the previously stored entities are loaded (names and states intact), then that a refresh succeeds.
4. Repeat on a low-mem device (release build) - the storage format differs per memory tier, so a migration that works on fenix7x can still be wrong on instinct2x.

## Submitting

- Keep changes focused; match the existing code style (2-space indent, camelCase, `Utils.debugLog` instead of `System.println`).
- State in the pull request which devices you tested, and whether you exercised the storage upgrade path.
