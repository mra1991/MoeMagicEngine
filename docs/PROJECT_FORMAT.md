# Moe Magic Project Descriptor

## v1.11 release — separate persistent-progress settings

Project descriptor **1** is unchanged. Optional `ProjectSettings/save_settings.json` is a separate
strict document, at most 4 KiB, with exactly `format: MoeMagicSaveSettings`, `version: 1`,
`enabled` (Boolean), and `gameId` (canonical lowercase UUID). Missing means disabled; a present
disabled record still validates its identity. Build Settings has a separate Persistent Progress
section with enable, read-only identity, confirmed new identity and Reset Editor Test Progress.
Save/Discard/Cancel protects its dirty draft before Play, Build, project operations and Exit.
These actions do not secretly save Scene or Build Settings documents.

Every New Project/template copy generates a fresh game identity; Basic 3D starts disabled.
Manual copying, rebuilding or relocating an existing game retains its authored identity. Changing
identity intentionally starts a separate save namespace. Editor Play uses an isolated namespace
that also includes a digest of the canonical project root; it cannot overwrite Player progress.
Confirmed Edit-only reset affects only that project's editor test slots, not Player saves.
Settings are captured once for Play/Build and cannot change during Play. Player output includes
only the enabled game identity, never save contents or editor test state.

## Status

This document records the unchanged **format-1** descriptor and released v0.12.1 launch contract, the
released v0.11.5 New/Open Project use of that unchanged format, and the released v0.11.6 global
Recent Projects generated-state boundary. The released
v0.11.0 editor did not parse this file; v0.11.1 requires and validates it before window
initialization. Released authority is
`exec-plans/v0.11.1-explicit-project-root-external-project-launch.md`; v0.11.5 authority is
`exec-plans/v0.11.5-transactional-new-open-project-workflow.md`; v0.11.6 authority is
`exec-plans/v0.11.6-global-recent-projects-and-reopen-continuity.md`.

## Identity and location

Released v0.12.6 adds three closed New Project choices without changing this format:
Basic 3D, Ultimate Platformer and Pot Pursuit. Format-3 installed samples are immutable sources;
creation copies only verified inventory entries through the existing sibling-stage transaction,
parses/re-serializes the descriptor with only the requested name changed, validates dependency
inputs, then publishes an outside-installation user project and launches a separate editor.
Child-launch failure preserves that published project. Existing destinations/originals are never
overwritten. Documents is suggested through the Windows Known Folder, not a hardcoded path.
Legacy/source-built installations without sample inventories keep Basic3D; source developers can
Open ordinary repository Projects. No arbitrary template path or shared sample mount is exposed.

One Moe Magic project root contains exactly one descriptor at:

```text
ProjectSettings/project.json
```

The descriptor identifies editor/project metadata only. It is not a Scene, prefab, Player
configuration, Build Game declaration, runtime dependency, recovery document, recent-list record,
or asset metadata file. It is not packaged into a Player.

The canonical project-root path is the v0.11.1 session identity. Format 1 intentionally contains no
project UUID, engine path, organization/company name, product name, template identity, recent state,
or asset GUID.

## Format 1

The file is strict ordered UTF-8 JSON:

```json
{
  "format": "MoeMagicProject",
  "version": 1,
  "name": "Ultimate Platformer"
}
```

Fields are exact and ordered:

1. `format` is the exact string `MoeMagicProject`;
2. `version` is the exact integer `1`; and
3. `name` is the exact project display name.

No missing, extra, duplicate, or reordered field is accepted. Wrong types, malformed JSON,
unsupported versions, invalid UTF-8, NUL/control content, an empty or whitespace-only name, and a
name longer than 128 encoded UTF-8 bytes reject the complete descriptor. The accepted name is
preserved for display; v0.11.1 performs no Unicode normalization or filename derivation from it.

The descriptor is read-only in v0.11.1. There is no editor draft, dirty state, Save/Reload panel,
undo, recovery, migration writer, or automatic repair. A failure is actionable before GLFW/editor
initialization and does not mutate the project.

## Launch contract

The implemented production editor syntax is:

```text
MoeMagicEngine.exe [--project <existing-directory>]
MoeMagicEngine.exe --project-launcher
MoeMagicEngine.exe --help
```

One explicit project directory is canonicalized and its contained descriptor is parsed before
window creation. With no arguments, the exact valid top Recent Project is reopened; empty,
malformed, or unavailable history presents the projectless launcher. `--project-launcher` forces
selection. Neither mode falls back to CWD, the checkout, or a second recent entry. Explicit
`--project` bypasses history and never falls back on failure. Missing values, duplicate arguments, unknown
options, extra positional values, invalid roots, and invalid descriptors reject before GLFW.

Resolving a project never changes the process working directory. Once bootstrap succeeds, the
immutable canonical project root—not ambient CWD—is the only editor project authority. Spaces and
valid non-ASCII Windows paths are supported subject to normal filesystem access and canonical
containment.

The launcher owns no Application, Scene, resource cache, project document, recovery lease, or
asset monitor. Its New/Open/Recent operations reuse validated new-process launching and global
Recent Projects storage. It exits after successful child launch and stays usable after failure.
Each editor process retains one immutable ProjectContext; in-editor New/Open/Recent never switches
dirty state in process. Portable projects must remain outside the installation.

Initial Edit Scene selection is the first valid project-local Recent Scene, then saved Build
Settings startup, then one unsaved starter with a diagnostic. Recovery resolves afterward in
Scene/Input/Build order. No engine-wide `default_scene.json` exists; Basic3D uses that filename only
as ordinary project content. The independent sample roots are `Projects/UltimatePlatformer` and
`Projects/PotPursuit`, each with Main Menu and gameplay plus its own exact asset/license closure.

## Path and ownership contract

The [asset-source schema 1](ASSET_LIBRARY_AND_PROVENANCE.md), introduced in v1.1.0, is independent of
Project 1. Source declarations belong to first-level project model/audio bundles; they are not
Scenes, prefabs, layout or global preferences. Saving a declaration changes no Scene/history.

Authored paths remain normalized project-relative identities. Absolute paths are transient I/O
values produced by resolving an identity against the immutable project root; they are never written
into Scenes, prefabs, materials, scripts, Build Settings, Input Actions, Player configurations,
manifests, or recovery source identities.

Project-owned inputs include:

- `assets/**` authored content and asset-specific redistribution notices;
- `ProjectSettings/project.json`;
- `ProjectSettings/build_settings.json`; and
- generated editor state beneath the project's ignored `Library` boundary.

Engine-owned Build Game inputs include the Release Player/GLEW templates, authoritative embedded-
splash source, vendored linked-runtime bytes, and mandatory linked-engine dependency notices. They
come from the validated repository-built editor layout, never from matching paths supplied by a
project.

## Released v0.11.3 generated Recent Scenes boundary

Released v0.11.3 adds one editor-generated, ignored project-local document at
`Library/RecentScenes/recent_scenes.json`. It is not part of `ProjectSettings/project.json` and does
not change project descriptor format 1. The strict Recent Scenes format 1 contains only an ordered
zero-to-ten array of normalized project-relative Scene identities. It stores no canonical project
root, timestamp, selection, Camera, object ID, or authored metadata.

The file is personal/generated workflow state. It never enters authored assets, Scene/prefab
documents, Build Settings, recovery payloads, asset catalogs, Player configuration, Build Manifest,
or Build Game packages. Missing state loads as empty. Malformed, future, or unreadable state warns,
loads an empty available list, and remains untouched for diagnosis; it cannot invalidate the
project or block editor startup. Updates publish atomically, retain at most ten identities, and
deduplicate identities case-insensitively. Recent projects, project switching, project UUIDs,
editable Project Settings, and global installed-editor history remain outside v0.11.3.

## Released v0.11.5 New/Open Project workflow

Released v0.11.5 keeps descriptor format 1 exact. File -> New Project accepts one bounded UTF-8
name that is also an exact valid Windows directory segment; it does not silently sanitize or store
a separate folder identity. The operation generates `ProjectSettings/project.json` through the
released codec inside an unpublished sibling stage, validates the staged root through
`ResolveProjectContext`, and publishes the project only through a final directory rename.

The generated Basic 3D project also contains exact Input Actions 3, Build Settings 1, one current
Scene-37 default Scene, `.gitignore`, and standard asset directories. It stores no absolute root,
engine path, template identity, timestamp, project UUID, or recent-project fact. The template Scene
uses procedural content only and the complete project is independent after publication.

File -> Open Project does not edit the descriptor. It canonicalizes one selected existing root,
validates it through the same released resolver, and launches a new editor process through explicit
`--project`. New Project uses that same launcher after publication. The current process retains its
original immutable project. In-process switching and Recent Projects remain excluded.

## Released v0.11.6 global Recent Projects generated state

v0.11.6 does not change project descriptor format 1. It adds one editor-generated machine-local
document outside every project at:

```text
<Windows Local App Data>/MoeMagicEngine/Editor/RecentProjects/recent_projects.json
<Windows Local App Data>/MoeMagicEngine/Editor/RecentProjects/recent_projects.lock
```

Its approved strict ordered format 1 is:

```json
{
  "format": "MoeMagicRecentProjects",
  "version": 1,
  "projects": [
    "C:/canonical/absolute/project/root"
  ]
}
```

The array contains zero to ten normalized absolute UTF-8 Windows roots in deterministic case-
insensitive MRU order. Each encoded root is bounded to 4,096 bytes and the complete document to
48 KiB. The document stores no project name, timestamp, project UUID, engine path,
template identity, availability flag, Scene state, authored metadata, or package data. Touch stores
only a canonical root already validated through `ResolveProjectContext`; display names and current
availability are derived afresh from the strict descriptor.

This absolute-path document is intentionally non-portable local editor history. It is not
`ProjectSettings/project.json`, project-local `Library`, an authored document, a recovery payload,
an asset catalog, Player configuration, Build Manifest, or Build Game input. Every mutating update
uses a bounded native exclusive lock, rereads latest state while locked, and publishes atomically
so concurrent parent/child editor processes do not overwrite one another's entries. Missing state
loads empty. Malformed/future/unreadable state warns and remains untouched except through explicit
Clear. History failure cannot invalidate a project or change a truthful Create/Open/launch result.

## Deliberate current limits

One project remains fixed for one editor process. v0.11.5 implements bounded Basic 3D generation
and validated new-process Open; v0.11.6 adds only global Recent Projects over that path.
Descriptor format 1 still does not imply in-process project switching, a pre-window launcher,
editable settings, project UUIDs, asset metadata, `.meta` files, import caches, automatic asset
refresh/hot reload, or installed-editor relocation. Those require separate approved plans.
