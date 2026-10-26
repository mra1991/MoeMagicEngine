# Build Settings Format

v0.8.7 owns one project document at `ProjectSettings/build_settings.json`. It declares the exact
ordered set of Scenes included by Build Game and exactly one startup Scene. It is authoring data,
not Scene data, runtime state, a build profile, or a request to load multiple Scenes at runtime.

In v0.12.1 each project owns this file; the checkout root has none. Basic3D uses the singleton
example below. Both `Projects/UltimatePlatformer` and `Projects/PotPursuit` instead declare
`assets/scenes/main_menu.json` as startup, followed by `assets/scenes/gameplay.json`. The editor
uses saved startup only when no valid project Recent Scene loads; Build Game always uses the saved
declaration, independent of the open Edit Scene. Release-time v0.12.1 package baselines were
Basic3D **15/16**, Ultimate **31/32**, and Pot **61/62**. Later game-content commits change the
closure; current saved inputs and the generated manifest govern, not those historical totals.
Format 1 is unchanged. See game-development direction.

## Format 1

```json
{
  "format": "MoeMagicBuildSettings",
  "version": 1,
  "startupScene": "assets/scenes/default_scene.json",
  "includedScenes": [
    "assets/scenes/default_scene.json"
  ]
}
```

The object has exactly these four fields. `includedScenes` contains 1–64 unique normalized
project-relative `.json` identities beneath `assets/scenes`; Windows case aliases count as
duplicates. Each identity is at most 1024 UTF-8 bytes, contains no control character, remains
lexically and canonically inside the Scene root, and identifies an existing regular file whenever
the document is loaded, edited, saved, or used for Build Game. `startupScene` must appear exactly
once in the array. Authored array order is retained. The complete UTF-8 document is bounded to
256 KiB, writes two-space JSON with one trailing newline, and rejects unknown, missing, duplicate,
or mistyped fields and unsupported versions transactionally.

## Editor ownership

The Build Settings document owns the candidate value, exact disk baseline bytes, and its clean,
dirty, unavailable, or conflict state. The dockable **Window > Build Settings** panel submits Add,
Remove, Move Up, Move Down, Set Startup, Save, and confirmed Reload requests; it does not perform
filesystem or package work. Add uses the native Windows file picker and accepts only a valid Scene
inside the project. The startup Scene cannot be removed until another included Scene is designated.
Editing is read-only during Play.

Save reparses a canonical candidate and writes a unique sibling temporary with write-through before
atomically replacing the fixed document. The exact source bytes are checked immediately before
replacement. External change preserves the draft and enters conflict state; Reload requires
confirmation before discarding a dirty draft. Exit coordinates Scene, Input Actions, and Build
Settings in one Save/Discard/Cancel decision and closes only after every requested save succeeds.

### Implemented v0.11.7 editor recovery boundary

The v0.11.7 implementation does not revise authored format 1. While this fixed editor document is
dirty, an editor-only recovery service may embed its canonical format-1 text in the strict
generated record:

```text
Library/Recovery/active_build_settings_recovery.json
```

The record contains exact authored-source and embedded-document fingerprints and is bounded to
1 MiB encoded bytes; the embedded Build Settings document retains the 256 KiB limit. Restore
reparses through this production codec, rereads the fixed authored source, requires the recorded
source fingerprint to match, and publishes a dirty editor draft without writing source. Changed,
missing, unreadable, malformed, future, or hash-invalid state cannot restore as Untitled, merge,
force overwrite, or become Build Game authority. Successful ordinary Save reconciles the generated
record only after source publication. Exact architecture and release gates are in
`docs/exec-plans/v0.11.7-transactional-input-build-settings-recovery.md`.
Implementation evidence is in `docs/implementation-reports/v0.11.7-implementation.md`.

## Build and runtime boundary

Play continues to use the open editor Scene. Build Game instead rereads the saved Build Settings
bytes immediately before opening the output picker and again validates every declared disk Scene.
An unsaved open Scene blocks Build Game only when that Scene is included; an unrelated dirty Scene
does not replace disk authority. Build Game packages each declared Scene once plus the compatible,
deduplicated union of parser-proven dependencies and notices. Missing, invalid, non-startable,
changed, escaping, or conflicting source/staged content rolls back the complete publication.

Player configuration 3 and Build Manifest 2 record the same ordered included set and startup
identity. Player validates and retains that allowed set but v0.8.7 loads only the startup Scene.
Runtime Scene transition/loading APIs, a Main Menu, additive loading, and cross-Scene object
references are not implemented.
