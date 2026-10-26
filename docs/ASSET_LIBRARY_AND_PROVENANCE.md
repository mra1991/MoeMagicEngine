# Asset library and project provenance

Current v1.12.0 supports OBJ, bounded JSON glTF, static FBX and the bounded skeletal FBX/TRS
path introduced in v1.7. Project-owned selected Lever/Door/Cannon and Worker_Male_Moe4 copies retain
their provenance/dependencies. Catalog/provenance remain version 1; current distribution is 7,
Scene47/prefab41/API23. Earlier version-specific paragraphs below describe their introduction.

## v1.7.0 selected skeletal FBX derivative

The reviewed Quaternius Worker_Male_Moe4 entry is Skinned / Animation Present, with 23 joints,
17 clips and six material sections. Add to Project copies its FBX, CC0 License, conversion README
and full JSON receipt; schema-1 `asset-source.json` records the original/derivative relationship.
The original over-four-influence FBX is retained in the repository, excluded from installed and
game closure. Exact inventory hashes and deterministic conversion are documented alongside it.
UltimatePlatformer uses its independent `assets/models/workshop-worker-moe4` copy. Catalog 1,
provenance 1 and distribution 6 are unchanged; no runtime library mount or automatic weight trim.

## v1.6.0 selected kart addition

v1.6.0 added the reviewed **Kenney Car Kit 3.1 — Rally selection**, not the full upstream
archive. The existing catalog receives three static art descriptors for extracted chassis, static
driver and one wheel reused four times; exact seven-file size/hash inventory remains authoritative.
`assets/models/kenney_car-kit-rally/DERIVATIVES.md` records the official archive URL/hash, selected
source hash, group extraction and ordinary art-node alignment. The catalog carries these derivation
facts and both retained notices into schema-1 project provenance. UltimatePlatformer owns its copy;
Players retain no shared-root dependency. Catalog/provenance 1 and distribution 6 are unchanged.

The following is the v1.1.0 accepted implementation reference. The completed
execution plan owns scope; completed
historical checkpoints are in the implementation report.

## Project-owned source records (schema 1)

Each new first-level bundle under `assets/models` or `assets/audio` owns `asset-source.json`.
The exact marker is `MoeMagicAssetSource`, version 1. Required fields are `origin`
(`kind`, `libraryId`, `assetId`, `revision`), `creator`, `package`, `sourceUrl`, `license`
(`identifier`, `url`, `attributionRequired`, `attributionText`), `notices`, `files`,
`derivations`, and `declaration` (`redistributionConfirmed`, `note`).

`SharedLibrary` records an origin, not a signature or legal certification. `UserDeclared`
has empty origin IDs/revision and requires the user's explicit redistribution confirmation.
Source/license URLs are informational HTTPS links, never fetched during import/build; an empty
source URL requires the declaration's explanation of original/local work. Attribution text is
required when its flag is true. At least one bundle-local notice is required.

Notice entries contain `path` and `sha256`; file entries contain `path`, `originalIdentity`,
and `originalSha256`. Original file hashes describe intake history, not a lock on project edits.
Notice hashes remain strict: changing a notice requires explicitly updating the declaration.
Derivations contain `description`, informational `inputs`, and bundle-relative `outputs`.
An input can describe an original not included in the runtime bundle.

The strict UTF-8 codec rejects unknown/duplicate fields, unsupported versions, wrong types and
escaping paths. Limits: 1 MiB record, 4,096 files, 64 notices/derivations, 4 KiB ordinary text,
1 KiB identities/paths. Serialization is deterministic, with sorted unordered record arrays and
no timestamps or machine paths. Required notice resolution checks exact case and rejects links/
reparse paths. Runtime model loading does not depend on this metadata.

## External declarations and publishing

Right-click a project model or WAV in the Asset Browser's folder listing and choose
**Asset Source / Provenance**. Supply creator, source, license/attribution, bundle-relative
notice paths (one per line), explanation and confirmation, then explicitly save. This writes
only the declaration; failure retains the prior record. It does not dirty a Scene or create a
Scene Undo entry. Editing is unavailable during Play.

Unknown model roots remain local-only until a valid declaration and notices permit publishing.
Legacy approved-root notice checks still apply and cannot be bypassed by a declaration. Existing
unrelated WAV policy is unchanged; newly declared WAV bundles contribute their notices.

Windows and Web share the saved-content closure. Only used models/audio, importer-discovered
dependencies, their provenance and notices are included—not every original file listed in a
record. Both linked runtimes carry the unmodified ufbx license even when no FBX is used. Web
retains declared notice identities beside provenance and its legacy Licenses presentation copies.
Scene 37, prefab 31, Project 1 and Lua API 12 are unchanged.

## Static FBX boundary

Pinned unmodified ufbx 0.23.0 feeds the existing CPU model-import result. ASCII/binary static
polygon meshes, instances, transforms, axes/units, material sections, normals, RGB/UV0 and
contained or embedded PNG/JPEG are supported. The later v1.7 path also supports bounded linear
skeletal deformation and baked TRS clips; morphs, constraints and over-limit skin weights remain rejected.
FBX does not construct a Scene hierarchy or introduce a separate gameplay/rendering path.
The adapter converts FBX UV V once for top-first decoded image rows. Web uploads RGB sRGB
textures as opaque RGBA so WebGL 2 can generate the existing mipmap chain; native uploads are
unchanged. The early native/Chrome checkpoint is recorded separately from package acceptance.

## Shared catalog and Add to Project

The editor owns one immutable `MoeMagicSharedAssetCatalog` version-1 snapshot. The explicit
shipping inventory and `installer/shared-library-metadata.json` generate it; no geometry corpus
import or ambient directory discovery populates search. Repository builds prepare
`build/SharedLibrary/catalog.json`; installed builds use `StarterLibrary/catalog.json`.
Missing/invalid catalogs disable only Shared Library and explain build/repair.

The catalog contains `libraryId`, `revision`, `packs`, `files` and `assets`. Packs carry factual
creator/source/license/notices/derivations. Files record source/shelf identities, exact size/hash
and role. Assets record a stable origin ID, revision, entry and dependency IDs, type/format,
category/tags, support reason and static/skinned/animation capability facts. Stable IDs do not
depend on installation path or byte hash. Descriptor/closure revisions are deterministic SHA-256;
the process generation advances independently on catalog publication. Names and art tags do not
claim gameplay mechanics. Source-only OGG remains inspectable, not an AudioClip or conversion action.

Limits: 8 MiB text, 1,800 files/assets, 64 packs, 32 tags of 64 bytes and 4,096 dependency references.
Queries accept up to 256 bytes and pages of 1–100 copied descriptors. Unknown references, escaping
or case-colliding paths and duplicate fields are rejected before snapshot publication.

Asset Browser defaults to **Project**. **Shared Library** offers keyword, type and pack filters,
details/provenance, known origin copies and **Plan Add to Project**. The plan displays the bundle,
count/bytes and notices before **Add to Project**. An optional explicit new-copy name resolves a
conflict without overwriting the existing copy. Adding reveals the new project asset; assignment
or instantiation remains a separate ordinary editor action.

`EditorAssetIntakeOperations` holds at most one opaque plan. New plans, Refresh, catalog/index
generation changes and consumption invalidate it; Add is blocked during Play. Selected files are
streamed/hash-verified into an invocation-owned `Library/AssetIntake` stage. The production CPU
model/audio importer validates that temporary project layout before a non-overwriting atomic
bundle rename. There is no GPU allocation, Scene dirtying, history operation or shared runtime mount.

AlreadyPresent requires current exact file, notice and declaration bytes—not a matching filename
or historical original hash. Edited/missing origin copies return Conflict. Failure before rename
cleans only the owned stage. Failure to refresh the index after publication returns
PublishedIndexRefreshFailed and preserves the complete bundle; use Project Refresh, not recopy.
Crash-left stages are ignored; there is no broad startup cleanup. Players load project content only.

Distribution format 6 introduced the catalog, static FBX, sample provenance and linked ufbx notice.
Current format 7 retains those records and includes the later navigation closure/notices.
