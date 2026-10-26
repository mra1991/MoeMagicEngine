# Project Asset Database and Dependency Diagnostics

`AssetDatabase` is the editor's sole owner of project asset discovery and the copied dependency
snapshot introduced in v0.11.8. It is an observational, process-memory service. It does not own
runtime resources, authored documents, package truth, or persistent asset metadata.

## Prefab-source edges and operations (introduced in v1.2.0)

`ProjectAssetIndex` exposes the `PrefabSource` relationship from Scene/prefab spawners to their
nonempty project-relative source, including disabled/inactive declarations. This remains copied
observational dependency data; Build Game independently captures and validates the authoritative
saved closure. There is no shared-library mount, scan-to-package or index-cache runtime loader.

`EditorObjectOperations` provides `AssignPrefabSource` / `ClearPrefabSource`; copied inspection
includes enabled, source and maximumLiveInstances. Add/Remove and enabled/limit changes use normal
Component commands and Undo/Redo. The Inspector uses these operations and the project prefab catalog;
it never parses/prepares a prefab or creates GPU resources. Clear is a permitted incomplete Edit
state; an enabled-empty spawner subsequently fails Play/Build preflight with assignment guidance.

## v1.1.0 released provenance relationship

The separate `EditorAssetSourceOperations` boundary reads or explicitly saves a bounded
[source declaration](ASSET_LIBRARY_AND_PROVENANCE.md) for a project-owned model/audio bundle.
AssetBrowserPanel only emits copied requests and displays results; Application executes them at
its safe point. The operation performs no GPU work or Scene/history mutation. A successful save
requests the ordinary project-index refresh. Unknown model bundles can report publishing eligibility
from a confirmed declaration with intact notices; this is not certification of legal ownership.
Existing fixed-pack notice requirements remain authoritative in Build Game.

`SharedAssetCatalog` is a separate editor-only snapshot, not a project index mount. Typed
`QueryProjectAssets` and shared query/inspection return copied, generation-labelled bounded pages.
`EditorAssetIntakeOperations` stages/validates a selected bundle and then invokes the normal index
refresh; it never inserts library paths into the project graph. Asset Browser's Project / Shared
Library views use this boundary. See the linked reference for tokens, duplicate/conflict rules,
and PublishedIndexRefreshFailed. All Player/runtime lookup remains project-owned.

## Released v0.11.10 notification relationship

The v0.11.10 release keeps `AssetDatabase` observational and adds a separate editor-only wide
Windows project notification monitor. Native events are change hints only. They are strictly
filtered, debounced, and passed as one bounded copied identity batch to the released
`EditorAssetOperations` transaction, which performs canonical Refresh and exact fingerprint/
candidate/source/target validation before any publication.

Normal idle notification does not call Refresh, hash/import the full repository, or alter
Catalogued/Uninspected state. Notification overflow may perform one exact bounded reconciliation
of only the currently reloadable source set; loss that cannot be reconciled pauses automation
instead of truncating silently. The monitor owns session pending/acknowledged evidence rather than
using an `AssetDatabase` generation as proof that live resources were updated.

Auto Reload does not change Scene/prefab/script lifecycles, Runtime/Player behavior, Build Game,
persistent metadata, formats, dependencies, or content. The exact implemented contract is
`docs/exec-plans/v0.11.10-debounced-referenced-asset-auto-reload.md`. Exactly **28/28** focused
Asset Auto Reload plus retained Asset Reload **30/30** and Asset Dependencies **30/30** markers
pass. Independent Tester acceptance also passes the real Unicode notification/resource workflow,
target exclusion, unchanged authored Scene proof, native-close exit 0, and exact owner-state
restoration. Exact evidence is in `docs/implementation-reports/v0.11.10-implementation.md` and
`docs/test-reports/v0.11.10-test-report.md`.

## Released v0.11.9 explicit reload relationship

The released v0.11.9 implementation keeps canonical `AssetDatabase::Refresh` observational. A
separate editor-only `EditorAssetOperations` boundary calls Refresh, reacquires one requested
identity by generation, consumes copied reverse dependencies, supplements them with live
Edit-Scene references, and coordinates a complete resource transaction through existing
production resource authorities.
Refresh itself must not begin loading or publishing resources.

The copied snapshot records exact lower-case SHA-256 fingerprints only for source files already
inspected or directly required by a supported explicit reload root. Catalogued/unreferenced Models
remain Uninspected and are not hashed or imported merely because Refresh ran. The operation
revalidates those fingerprints before all-or-nothing publication; they are process-memory evidence,
not GUIDs, `.meta` files, or a persistent import/hash database.

Supported approved fan-out covers Models and their MTL/texture/external-buffer support, reusable
Materials and textures, direct Canvas images, and AudioClips. Automatic watching, background
polling, reload-on-save, runtime/Player hot reload, and Scene/prefab/script migration remain outside
the approved milestone. The exact contract is
`docs/exec-plans/v0.11.9-dependency-aware-transactional-asset-reload.md`. Focused Asset Reload and
retained Asset Dependencies validation each pass **30/30**; exact implementation evidence is in
`docs/implementation-reports/v0.11.9-implementation.md`, and independent acceptance is in
`docs/test-reports/v0.11.9-test-report.md`.

## What is indexed

One successful scan publishes a deterministically ordered `ProjectAssetIndexSnapshot` containing:

- Scenes beneath `assets/scenes`;
- prefabs beneath `assets/prefabs`;
- supported OBJ, static/one-skin animated glTF and bounded static/skeletal FBX models beneath `assets/models`;
- model support documents reported by the production importer, including MTL and external buffers;
- reusable Materials beneath `assets/materials`;
- PNG/JPG/JPEG images beneath `assets/images`, `assets/textures`, or a reachable model pack;
- WAV AudioClips beneath `assets/audio`; and
- Lua scripts beneath `assets/scripts`.

Project, Input Actions, Build Settings, recovery, Recent state, `Library`, notices, editor/Player
binaries, and generated package files are not asset nodes. Saved Build Settings is read only to
classify valid Scenes as Startup + Build Included, Build Included, or Editor Only. Every valid
prefab is an Editor Authoring Root.

The released model, prefab, Material, audio, and script catalog/search APIs remain available and
are published in the same transaction as the dependency snapshot.

## Reference and inspection flow

The dependency graph is derived only from complete production-validated values:

```text
SceneSerializer / PrefabSerializer
          |
          v
ProjectAssetReferenceCollector
          |
          +--> model / Material / Canvas image / AudioClip / Script / Scene target

ParseMaterialAsset -------------------------------> Material texture

reachable model --> ModelImporter ----------------> MTL / texture / external buffer
```

Edges point from the referencing asset to the referenced asset. Repeated relationships are
deduplicated by case-insensitive source identity, target identity, and relationship kind. The
edge retains the exact usage count and at most eight deterministic copied contexts. Reverse edges
are derived from the finalized direct set and retain the same usage/context values.

All supported model identities are catalogued cheaply. Only models reached from a valid Scene or
prefab are passed to `ModelImporter`. An unreferenced model is reported as
`Catalogued — dependency closure not inspected`; it does not claim a healthy dependency closure
or redistribution approval. Reachable models also receive the shared CPU-only
`ModelRedistributionPolicy` decision. Build Game does not trust the snapshot: it rereads current
sources and notices and reconstructs package closure independently.

## Refresh transaction

Scan and Refresh build nodes, parsed roots, roles, dependencies, reverse edges, diagnostics, and
the retained typed catalogs off to the side. A complete candidate publishes once and advances the
generation once.

Ordinary asset problems publish a complete unhealthy generation so they remain inspectable. This
includes malformed or unsupported documents, missing dependencies, wrong kinds, path escapes,
and models outside approved redistribution roots. Missing safe identities receive placeholder
nodes so reverse dependents remain visible.

Structural failures preserve the entire previous generation and every typed catalog. Structural
failure includes an unavailable/aliased category root, incomplete directory enumeration,
case-insensitive identity collision, graph capacity exhaustion, direct/reverse inconsistency, or
an unexpected construction failure.

The hard graph limits are:

- 16,384 nodes;
- 65,536 unique directed edges;
- 8 copied contexts per edge;
- 1,024 UTF-8 bytes per diagnostic or context; and
- 1,024 snapshot warnings.

Existing stricter serializer, importer, image, audio, and script limits still apply.

## Editor workflow

Open **Window -> Asset Dependencies** to inspect the current generation. The panel supports
case-insensitive identity search, kind/role/health/inspection filters, direct dependencies,
reverse dependents, usage contexts, and diagnostics. Panel selection is presentation-only and is
reacquired by copied identity after a generation changes.

The Asset Browser and Asset Dependencies panel submit Refresh through the same typed
`EditorObjectOperations::RefreshAssetCatalog` operation. Asset Browser folder rows update only
after that transaction succeeds.

Refresh reads the current project files for diagnostics, but deliberately does **not** reload live
models, Materials, textures, audio, or scripts. It does not reconstruct the active Scene or alter
selection, Camera, command history, dirty/source baseline, recovery, Play state, an unsaved Build
Settings draft, or any authored file. Resource reload remains an explicit resource operation.

For a selected supported identity, **Reload Affected Resources** submits a copied identity and
expected generation to `EditorAssetOperations`. That distinct explicit transaction refreshes and
reacquires the identity, resolves reverse roots plus current live references, prepares all Model,
Material/texture, Canvas image, AudioClip, and matching clean Material-document replacements
offside, revalidates exact source fingerprints and targets, then publishes all or nothing. The
Inspector Model and Material actions use the same boundary. Dirty/conflicted documents, unsafe
editor state, invalid or changing sources, and incompatible Animators reject without mutating the
prior live generation.

## Ownership and exclusions

- `AssetDatabase` owns the current copied generation and retained typed catalogs.
- `ProjectAssetReferenceCollector` only walks already validated Scene/prefab values.
- production serializers, path policies, decoders, and `ModelImporter` retain validation authority.
- `ModelRedistributionPolicy` owns only the exact root-to-notice mapping.
- Build Game retains package enumeration, source/notice reads, hashes, staging, and publication.
- `Application` adapts UI intent; panels own only presentation state.
- Runtime, Player, and packages exclude the editor index, collector, panel, and validation source.

v0.11.9 adds no `.meta` files, GUID database, watcher, background worker, automatic hot reload,
asset mutation, runtime/Player observation or reload, package format, or public tools/AI surface.
