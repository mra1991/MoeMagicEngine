# Persistent Progress and typed prefab authoring

Released in v1.11.0; acceptance/artifact status belongs to the implementation
and independent test reports. Scene46, ordinary prefab40 and Project1 are unchanged. Player4,
save settings1, save envelope1, prefab variant1 and exact Lua API22 are the new contracts.

## Game identity and authoring

Build Settings exposes a separate Save Settings document. Enable progress, save the document and
keep its generated game UUID stable across game rebuilds/relocation. New Project and installed
sample creation generate fresh UUIDs; Basic 3D starts disabled. A manual project copy retains its
identity. Explicit New Identity is a deliberate fork, not an automatic response to load failure.
Missing settings leave older projects disabled and playable. Invalid settings reject Play/Build.
Resolve Save/Discard/Cancel for dirty settings or prefab assets before Play, Build or project/exit
actions. These documents do not create Scene Undo entries.

## Explicit durable state

API22 offers Load, Save, Reset, copied Status and transient run-result clearing. Values are flat
Boolean/finite Number/UTF-8 String records, not objects, world snapshots or arbitrary files. Three
slots, 128 keys, 64-byte ASCII keys, 256-byte strings and 64-KiB envelopes bound each operation.
Load before mutation; await the matching receipt. Accepted means admitted, not saved. One request
is outstanding and mutations are rate limited to one per second. Failed frames publish no I/O.

Writes compare the exact loaded observation. Conflict requires a new Load and project-level
reconciliation; never blindly overwrite. Corrupt/future data is not Missing. Only explicit Reset
can replace such a bounded observation, and Reset writes an empty schema1 revision tombstone.
Project schema migration is Lua policy. Unknown newer schema must be left intact. Reset durable
data first, await success, then clear transient results if that is the game's New Game policy.
See [API22](SCRIPTING_API.md#api-22--explicit-persistent-progress) for exact fields and phases.

Windows uses Known-Folder LocalAppData with separate game and editor-test namespaces. Player
identity survives whole-folder relocation. Editor tests also include the project-root digest,
so Play cannot overwrite Player progress. No actual save enters Build Game or installer payloads.
Per-slot locking, flushed temporary files and atomic replacement preserve prior data on failure;
this is not a power-loss guarantee.

Browser storage is IndexedDB, origin/partition scoped, with success only on transaction completion.
Same-origin path/iframe reload can retain saves. A new random-port Preview may be a different
origin. Guest/private mode, denial, quota, eviction and clearing site data can remove/prevent saves.
There is no cloud sync, cross-origin transfer or memory-only Saved fallback. Normal gameplay may
continue in an explicitly labeled session-only mode.

## One-base prefab variants

In Asset Browser, right-click an ordinary prefab to Create Variant, or Edit Prefab Properties /
Inspect Base. The separate asset panel offers copied typed properties, inherited/effective values,
and override removal. Only the documented scalar/model/material/property whitelist is editable;
hierarchy, Component structure and references remain read-only. Save writes the asset, not existing
Scene instances; instantiated copies retain ordinary Undo/Redo semantics.

A variant names one project-owned ordinary base and its exact SHA-256. It cannot inherit another
variant. Changing base bytes makes the variant stale. Preview Refresh, inspect the effective values,
explicitly accept that digest, then Save. Nothing silently accepts a changed base. Stale/missing/
invalid assets block use before partial publication. Build Game captures the wrapper, base and
only effective used resources. Bases count toward the 32-document runtime closure limit, not an
extra spawn level. See [Prefab format](PREFAB_FORMAT.md) for descriptors, limits and encoding.

## Ultimate Platformer

Slot1/schema1 stores completed Sky Orchard/Boulder Run/Workshop Rally and the two Orchard Gems.
Results merge accepted completion, preserving previously earned Gems. Continue advances through
the three known levels, wrapping Rally to Orchard; all levels remain directly selectable. New Game
requires explicit confirmation and a successful reset receipt. Death/recovery/Restart stay attempt
local. Retry reloads and reconciles after failure; Session Only never claims a durable reset.
Main Menu and Results attach their progress policy to the owning Canvas ScriptBehaviour, which
also preserves the UI sound palette. Canvas-scoped interaction callbacks record actions; ordinary
Update performs progress/transition requests. Separate Scene-root scripts do not receive Canvas events.
The two existing objective Gem paths now share one base; course mechanics and art are unchanged.
