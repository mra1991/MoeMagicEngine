# Editor distribution format and layout

Released v1.22.0 publishes **format 9**, with an explicit optional Cinder slot. Its accepted public
payload leaves the slot empty and has two template choices. Use the current manifest and exact
maintained inventories for file counts/hashes. Earlier numbered sections describe historical
increments, not the current product version.

**Released v1.22:** the plan, §7 (see installed documentation/source reference)
implements format9 and one explicit optional Cinder sample slot. `optionalSamples: []` retains
exactly Basic 3D and Ultimate Platformer. The only nonempty form is the closed CinderRelay record
with the fixed project root and tutorial identities specified by the plan. Its exact
declared files, notices, terrain/navigation and saved dependency closure must verify before a
third choice appears. Partial/tampered/unlisted/escaping content rejects rather than becoming
an unavailable but trusted sample. Historical formats1–8 keep their original contracts.
Format9's exact ordered root is `format`, `engineVersion`, `platform`, `optionalSamples`, `files`;
the native serializer, parser and package publisher use the same order.

In v1.22, `package-editor.ps1 -IncludeCinderRelay` created the explicit portable fixture using
`installer/cinder-relay-files.txt` and generated `Documentation/cinder-relay.html`. Reuse the
canonical matching Release receipt; never install this fixture under the normal AppId. The
v1.22 public installer omitted the switch, Cinder files and tutorial. Copying an admitted optional sample
uses the ordinary confined New Project transaction and a fresh save identity; installation
sources stay immutable. No directory discovery or runtime Shared fallback.

**v1.23 candidate recipe, acceptance pending:** the plan, §6 (see installed documentation/source reference)
populates this same format9 slot for the accepted public mission. The existing installer wrapper's
`-IncludeCinderRelay` selection carries the exact reviewed project/tutorial inventory and yields
three choices after validation. This approval supersedes the former fixture-only product boundary,
not its historical evidence or confinement rules. No format10, new launcher discovery or changed
New Project transaction. Do not install an old optional fixture as the new public candidate.
V1230-T1 preserves the slot's intentionally empty image/texture category roots through explicit,
selection-guarded Inno `[Dirs]` records. The manifest remains an exact file inventory; installed
admission is not weakened and no placeholder file or format change is introduced.

## v1.1.0 introduction — format 6

Format 6 preserves all format-5 Web/template/preview roles and requires
`StarterLibrary/catalog.json` (`shared-catalog`) plus `ThirdParty/ufbx/0.23.0/LICENSE` (`notice`).
Static `.fbx` is admitted only in the model shelf/sample roles. Sample model/audio bundle
`asset-source.json` records use `asset-provenance`; New Project copies these alongside the sample's
other declared files. These are explicit inventory inputs, not directory discovery or a shared
runtime mount. The catalog/metadata overlay and pinned vendor input participate in publication
receipts. The 21 legacy core entries and existing size/count/path/hash limits remain unchanged.
Formats 1–5 retain their historical accepted engine versions (including format-5 1.0.0/1.0.1);
unknown future formats reject. See [Asset Library and Provenance](ASSET_LIBRARY_AND_PROVENANCE.md).

The following sections retain the historical format increments.

Released v0.12.8 (see installed documentation/source reference) specifies format 4 with
bounded starter-library/product-license roles and exact legacy readers. Format 4 is that historical
baseline; released v1.0.0 uses format 5. SourceAudio is explicitly non-runtime
content, not a project fallback. The selected shelf contains 1,062 entries / 155,071,525 bytes before its README.

The completed v1.0.0 milestone (see installed documentation/source reference) implements format 5.
It retains formats 1–4 at their exact historical read boundaries, including v0.12.8 format 4.
`web-preview` admits exactly `MoeMagicWebPreview.exe`. `web-template` admits exactly
`WebTemplate/{index.html,moe-player.js,moe-player.wasm,web-template.json}` and the seven declared
Emscripten/linked-runtime notice files under `WebTemplate/Licenses/Emscripten/` (LICENSE, AUTHORS,
musl-COPYRIGHT, libcxx/libcxxabi/compiler-rt/libunwind LICENSE.TXT files).
All twelve additions are required and hash/size/path verified. Project/native game formats and
the existing 2,048-entry/4 GiB distribution bounds remain unchanged. Older formats reject these
roles. [WEB_BUILD_FORMAT](WEB_BUILD_FORMAT.md) owns the inner template/package contracts.

The v1.0.1 release (see installed documentation/source reference) retained format 5 and its
payload roles. Archive generation is compiled editor functionality, not a new shipped helper or
SDK. That release published matching 1.0.1 templates and installer through the existing receipt
path; archive outcome/retry does not change the Web manifest or distribution schema.

Format 4 retains caps of 2,048 records, 4 GiB total, 1 MiB manifest, 512-byte path and 256 MiB per
file. Required additions are `Licenses/MoeMagicEngine.txt` (`product-license`) and
`StarterLibrary/README.html` (`starter-library`). The license also has an independently pinned
approved hash. `starter-library` admits model/data/notice files under `StarterLibrary/assets/models/`
and WAV/provenance under `StarterLibrary/assets/audio/`. `source-audio` admits only OGG/provenance
beneath `StarterLibrary/SourceAudio/`, never an executable/runtime/project/template role. Publication
is further limited by the reviewed five-model/nine-audio-pack source inventory. Formats 1/2/3
reject these new roles. Format 3 retains exact version 0.12.6/0.12.7 reads alongside current reads.
Sizes/hashes/casing/containment/reparse checks and atomic publication remain in force.

The approved v0.12.6 plan (see installed documentation/source reference)
implements format 3 for verified named sample inventories and retains format-1/2 readers. The
released implementation adds `sample-project` exclusively beneath `Samples/UltimatePlatformer/` and
`Samples/PotPursuit/`. Both require project/build/input descriptors. Reviewed sources are listed
in `installer/sample-project-files.txt`; generated Library, test captures, tools and unsupported
model/audio alternatives are excluded. All entries retain canonical path/size/hash/casing/reparse
validation. Format 3 caps 2,048 records and 4 GiB total; the 1 MiB manifest, 512-byte path and
256 MiB per-file limits remain. Current sample selection adds 154 files to 28 core/docs/default
inputs; counts derive from actual inventory, not a format constant.

New Project revalidates sample sources around copying, retains existing owned-stage/destination
rules, changes only descriptor name through its codec and validates staged documents/dependencies
before publication. Legacy/source builds without declared sources give actionable feedback.
Format 2 retains released 0.12.5 reading; format 1 retains 0.12.1. Inno adds uninstall bookkeeping
outside the manifest. See INSTALLER (see installed documentation/source reference).

v0.12.5 emits `MoeMagicEditorDistribution.json` **format 2**. This is integrity inventory, not
a signature or public license. The strict ordered root fields remain `format`, `engineVersion`,
`platform`, `files`; each ordered record is `path`, `role`, positive `size`, lowercase `sha256`.
Current format 6 requires the running engine version and `windows-x64`.

The 21 fixed core entries retain their original roles. Format 2 additionally requires:

| Path | Role |
| --- | --- |
| `EditorDefaults/imgui.ini` | `default-layout` |
| `Documentation/index.html` | `documentation` |
| `Documentation/user-manual.html` | `documentation` |
| `Documentation/ultimate-platformer.html` | `documentation` |
| `Documentation/pot-pursuit.html` | `documentation` |

The v0.12.5 generation inventory adds CSS and branding PNG; that payload has 28 records / 29
physical files. Counts derive from inventory, not a frozen format total. Documentation-only
resources stay under `Documentation/`, with `.html`, `.css`, `.png`, `.jpg` or `.svg` extensions;
no project asset or executable role is admitted there. Format 2 caps all records at 256,
manifest bytes at 1 MiB, path UTF-8 bytes at 512 and each file at 256 MiB. Paths must be canonical,
unique including case-insensitive comparison, strictly ordinal-sorted and contained. Missing,
reparse-backed, case-ambiguous, escaped, size/hash-mismatched required inputs reject startup.
The seed must also pass the same bounded INI grammar used by preferences.

Format 1 retains exactly the legacy 21-file role/path contract. Its released 0.12.1 payload remains
readable, but is never accepted by current package verification as format 2. Legacy missing Help
is actionable and missing seed uses the warned defensive arrangement; no checkout/CWD search.

Repository mode explicitly resolves root `imgui.ini` and `build/Documentation/index.html`.
Distribution mode resolves only the trusted default/manual paths above. In both modes valid
`LocalAppData/MoeMagicEngine/Editor/imgui.ini` customization takes precedence; otherwise the
seed loads before docking construction. Reset loads the seed and saves customization. Malformed
or unreadable preferences are preserved and automatic replacement is suppressed until explicit
reset. ImGui's implicit relative-path I/O is disabled in both modes. The projectless launcher
loads/saves neither editor seed nor preferences. Layout is not Scene/Undo/project state.

Packaging preflights documentation tooling/output, invokes the canonical clean Release build,
copies only declared source inputs into one owned sibling stage, rechecks source/destination
hashes, emits/reparses the manifest and atomically publishes to a nonexistent outside-repo final.
An already published folder is never overwritten; only the verified invocation-owned stage is
cleaned on failure. No CWD, machine path or timestamp is emitted in the deterministic inventory.
Scene 37/prefab 31, project, Lua API 12 and game-package formats remain unchanged.
