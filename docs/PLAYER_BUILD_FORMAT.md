# Player Build Formats

## v1.11 release — Player configuration 4 and variant closure

Player configuration now writes **4**, with optional `saveGameId` only when project persistence
is enabled. A present identity must be a canonical lowercase UUID. Readers **1–3** remain
compatible with persistence disabled; malformed v4 identity is an error, not a display-name
fallback. The shared saved-content snapshot captures settings once for native and Web exports.
The project settings file itself is not required beside the Player. No saves, editor test state,
temporary storage files or user preferences enter a package.

The prefab closure includes exact variant and ordinary base bytes, plus dependencies of the
resolved effective records. Unique bases count toward 32 captured documents but not spawn depth.
The inherited source record alone does not load/package an overridden-away model or audio asset.
Both publishers consume the same captured snapshot; they do not reread live source during copy.
Native manifest 2, Web manifest 1 and distribution 7 remain unchanged.

## Declared prefab closure (introduced in v1.2.0)

Native configuration 3 and manifest 2 remain unchanged. The file-kind vocabulary adds `prefab`.
The shared saved-content snapshot follows every nonempty Prefab Spawner source, even disabled or
inactive, using the same bounded actor/graph validation as Play. It captures exact prefab bytes,
their models/materials/audio/scripts, further sources and required provenance/notices. Actor
templates do not need a standalone Camera. Missing/unsupported/cyclic/over-budget sources fail
before publication. Staging reparses the graph and compares captured prefab bytes; unrelated
prefabs stay excluded. Relocated Players prepare that same explicit project-local closure.

## v1.1.0 released static FBX and provenance closure

The shared saved-content snapshot accepts supported static FBX and valid project-owned
`asset-source.json` declarations. Used declarations/notices accompany exact importer dependencies;
unused originals listed in provenance are not runtime dependencies. Original hashes do not lock
edits, while local notice hashes remain mandatory. Fixed legacy pack checks cannot be overridden.
Every linked Player additionally carries `ThirdParty/ufbx/0.23.0/LICENSE` unchanged. Native manifest
2 adds the `provenance` file-kind token; its structure and Player configuration 3 do not change.
See [asset ownership and schema 1](ASSET_LIBRARY_AND_PROVENANCE.md). Publication evidence is tracked
in the implementation report, not implied by this reference.

## v1.0.0 additional Web target

Windows configuration 3 and manifest 2 are unchanged. CPU-only `GameContentSnapshot` supplies the
same saved Scene/dependency byte closure to independent Windows and Web publishers; native full
restoration/template/staging checks remain. Target choice is an export request, not a Build Settings
migration. **File > Build Game > Windows...** retains native behavior.

The experimental target uses Web manifest 1, the same configuration 3, browser runtime files and
Licenses; it excludes PE/GLEW and the preview helper. See [WEB_BUILD_FORMAT](WEB_BUILD_FORMAT.md)
for exact inventory, conservative upload/startup bounds, case rules, preview and itch.io guidance.

## v0.12.8 product and starter-pack notices

Every current Build Game package includes the full approved `Licenses/MoeMagicEngine.txt` once
as an ordinary notice, alongside the ten runtime dependency notices. Pinned bytes and contained
source are verified before publication. It permits game/runtime redistribution with required Moe
branding, not Editor mirrors. Game manifest/configuration formats do not change.

Exact Kenney Blocky Characters 2.0 and Quaternius Ultimate Animated Character (Nov 2019) roots now
have reviewed pinned notice mappings. Approved starter WAV packs include deduplicated adjacent
provenance and Interface/RPG derivative records. Unrelated valid project WAVs remain supported.
Missing/mismatched required notices reject; the starter shelf and source-only OGGs never enter
packages automatically. Older installations lacking these terms need the current complete
installation for the current Build Game contract.

## v0.12.1 project-scoped sample packages

Each selected immutable project supplies its saved Build Settings and exact dependency closure.
At the v0.12.1 release, Basic3D was **15 manifest entries / 16 physical files**, Ultimate Platformer
**31/32**, and Pot Pursuit **61/62**. These are release-time measurements, not count constraints on
later game-content commits; audit current inputs and the actual generated manifest.
Each sample declares `assets/scenes/main_menu.json` followed by
`assets/scenes/gameplay.json`; the Main Menu targets that same project-local gameplay identity.
There is no shared root/sibling fallback or repository-wide content scan. Project descriptors,
Library, Recent Projects, editor layout, and launcher state never enter these packages. Player
configuration 3 and Manifest 2 remain unchanged. Historical baselines below describe their releases.

## Implemented v0.12.0 mandatory linked-code notice closure

Every Build Game package now adds exact TinyObjLoader, stb, and nlohmann JSON notices to the
existing GLFW, GLEW, GLM, Jolt, miniaudio, Lua, and cgltf set. These ten engine dependency notices
are mandatory even for procedural-only content because the corresponding code is compiled into or
required by every Player. ImGui and ImGuizmo remain editor-only and never enter Player packages.
No Player configuration, Build Manifest, Scene, prefab, Material, Input Actions, or Lua API format
changes. Exact generated, ordinary, and isolated package closure is **15/16**, **30/31**, and
**23/24** respectively, where the second number includes the self-excluding Manifest 2 file.

## Main Menu startup and host action closure (introduced in v0.8.9)

The strict format-1 Build Settings document now declares exactly two ordered Scenes:
`assets/scenes/main_menu.json` is startup and `assets/scenes/default_scene.json` is second. Build
Game continues to package only that declaration and its parser-proven dependency union; demos and
Showcases are not discovered or copied. Player configuration remains format 3 and Build Manifest
remains format 2, both retaining the exact order and the unchanged game name
`Ultimate Platformer Coin Run`.

After the unchanged embedded splash, Player restores the ordinary Scene-28 menu and seeds its one
authored initial-focus target. Play follows the released transition path and carries the same
effective Input Preferences profile into the level. The field-free Quit action is published only
after a successful runtime frame; Player consumes it as normal successful shutdown, while editor
Play consumes it only as Stop. No new package file, manifest kind, host protocol, or loose branding
asset is introduced. The startup-identity contribution to Player preference storage deliberately
creates one new pre-1.0 namespace.

## Implemented v0.8.8 runtime transition and branding closure

Configuration 3's normalized ordered `includedScenes` is now the immutable runtime allowed set.
Player loads startup normally, then a Button-authored Scene Transition request may select only an
exact member. The host validates membership before file access, prepares and starts a complete
candidate Scene/PlaySession with one active enabled primary Camera and the same effective input
profile/AudioService, and publishes it at a frame boundary. Content failure preserves the active
world; a genuine healthy-backend mutation failure remains fatal. Direct `--scene` launches retain
singleton authority and therefore support self reload but reject cross-Scene targets.

Build Game validates every enabled authored target against the exact declaration in both source
and staged Scenes. It packages only the already-declared Scene set and adds no transition-specific
file kind. Build Settings stays format 1, Player configuration stays 3, and Manifest stays 2.

The authoritative `MoeMagicPlayer/Resources/PoweredByMoeMagic.png` is 1,639,625-byte 1672x941 RGB
at SHA-256 `510219552E9C17991F44E6E46EE6912EC9A3F2400B102C64A0D9BF5007CBF64C`.
Debug/Release Player builds regenerate native RCDATA from that explicit input. Before staging,
Build Game bounds/decodes the current source and requires exact equality with extracted Release-
template RCDATA. Missing, malformed, stale, or mismatched branding publishes no package; no loose
startup-branding fallback exists.

## Implemented v0.8.7 multi-Scene declaration and package closure

Build Game reads the saved strict `ProjectSettings/build_settings.json` declaration and packages
each ordered included Scene exactly once. Every Scene is parsed and restored independently from
exact source and staged bytes and must contain exactly one active enabled primary Camera. The
physical inventory is the compatible case-insensitive deduplicated union of all parser-proven
Scene dependencies and mandatory notices; incompatible claims for one package identity fail the
whole transaction. Build Game never scans the Scene directory or substitutes the open editor Scene.

Player configuration advances to format 3 and Build Manifest advances to format 2. Both record the
same exact ordered `includedScenes` and `startupScene`; the Manifest also records
`playerConfigurationVersion: 3`. Configuration formats 1 and 2 remain readable as singleton
allowed sets. Player validates and retains the allowed set but loads only startup in v0.8.7. Direct
development `--scene` launch deliberately creates a singleton allowed set. No runtime Scene
transition, Main Menu, additive loading, or cross-Scene object reference is implied.

## Implemented v0.8.5 embedded Player-branding closure

Player configuration remains format 2 and Build Manifest remains format 1. Exact
`MoeMagicPlayer/Resources/PoweredByMoeMagic.png` bytes compile into Debug and Release Player as
numeric Windows RCDATA. Player copies and decodes the bounded resource from its current module and
must produce the released black-background, one-second opaque hold, and 0.4-second fade before
simulation/input begin. Missing, empty, oversized, inaccessible, undecodable, or unrenderable
mandatory branding returns distinct process result 7 and never falls through to gameplay.

Before stage creation, Build Game opens the selected Release Player template data-only, extracts
the exact same resource ID, and decodes it through the shared limits. Failure publishes nothing.
The Player executable remains manifest kind `player`; there is no automatic loose logo copy,
`startup-logo` manifest kind, path override, or runtime file fallback. A Scene-referenced image,
including `assets/images/logo.png`, remains an ordinary exact `canvas-image`. Adding, deleting, or
changing unrelated loose logo/splash files cannot affect Player startup authority.

## Historical pre-v0.8.7 multi-Scene direction

Earlier plans reserved an explicit declared Scene set. v0.8.7 implements its authoring, package
closure, and versioned metadata boundary. Runtime transition/loading and Main Menu authoring remain
future work; inclusion remains explicit and directory-wide Scene copying remains forbidden.

## Implemented v0.8.1 authored animation closure

v0.8.1 keeps Player configuration 2, Build Manifest 1, and the single selected startup
Scene. Animator values introduce no new file kind: Build Game requires the same-object rigged Model
and exact case-sensitive clip, then packages only that Model's parser-proven `model`, optional
`model-buffer`, optional `texture`, and cgltf-notice closure. After exact bytes are copied, every
staged Model is reparsed and its dependency, rig, and clip contract is compared with the source-side
candidate before configuration/manifest publication. Missing or changed clips/rigs and malformed
staged content roll back the existing transaction. No runtime playhead, pose, palette, FBX/GLB
variant, or unrelated animated corpus file is packaged.

## Implemented v0.8.2 runtime Character animation closure

The released v0.8.2 implementation keeps Player configuration 2, Build Manifest 1, the single startup
Scene, and every existing dependency kind. Build Game validates the Scene-22 Character Animation
Behaviour/controller/motor/direct-child Animator relationship and all three exact clips against
both source and staged rig data. Runtime command/state values add no file and no manifest entry;
missing, ambiguous, disabled, incompatible, changed, or tampered relationships fail the existing
transaction. Standalone Player uses the same PlaySession-owned command/state/Animation World path.

## Implemented v0.8.3 interruptible cross-fade closure

The released v0.8.3 implementation keeps Player configuration 2, Build Manifest 1, the single startup
Scene, and every existing dependency kind. Build Game reparses Scene 23 and validates the finite
`transitionDurationSeconds` value together with the released controller/motor/direct-child Animator
and exact-clip contract against source and staged rig data. Cross-fade commands, sampled source
poses, timing, playheads, and final poses add no file or manifest entry. Standalone Player uses the
same PlaySession-owned Animation World and local-pose composition path as editor Play.

## Implemented v0.8.4 Character jump-phase closure

Released v0.8.4 keeps Player configuration 2, Build Manifest 1, the single startup
Scene, and every existing v0.8.4 dependency kind. Build Game reparses Scene 24 and validates each nonempty
optional takeoff/landing clip together with the released controller/motor/direct-child Animator
relationship against both source and staged rig data. Empty optional names preserve v0.8.3 package
behavior. Fixed-step event revisions, copied playback status, logical phases, completion state,
commands, playheads, and terminal poses add no file or manifest entry. Standalone Player uses the
same PlaySession-owned status/command/Animation World path as editor Play.

## Implemented v0.8.0 static glTF package closure

v0.8.0 retains Player configuration 2, Build Manifest 1, and the current single startup Scene. A
selected supported static `.gltf` document uses existing
manifest kind `model`, each parser-proven contained external BIN will use new closed kind
`model-buffer`, and each parser-proven external PNG/JPG/JPEG base-color image will use existing
kind `texture`. Embedded data buffers have no separate package identity. The exact pinned cgltf
1.15 MIT notice becomes mandatory because the parser is compiled into Player.

Package construction must reparse through the same strict production policy, capture exact bytes,
and copy only selected-Scene dependency closure. It must exclude unrelated GLTF/GLB/FBX/BIN/images,
unsupported animated/skinned content outside the v0.8.1 one-skin subset, and pack siblings. Missing,
escaping, changed, colliding, unsupported, or
tampered model/buffer/image/notice input rolls back publication. Existing OBJ/audio/Canvas/input/
runtime roles remain exact. The compact unique sibling stage name deliberately omits the redundant
game name so deeply nested approved dependencies retain Windows path budget before atomic final
folder publication.

## Implemented v0.7.2 command/persistence closure

The v0.7.2 release accepts Scene 19 `AudioButtonBehaviour` data without adding any package
dependency or manifest kind. Player configuration remains 2, Build Manifest remains 1, and the
exact selected-Scene WAV/miniaudio closure remains authoritative. Every Audio Source referenced by
the selected Scene participates in closure even when it is used only by a runtime command. Detached
voices and queued runtime commands are session-only and never enter either JSON document or a
package. The expanded Audio Showcase packages exactly four unique WAV identities: startup click,
restart click, one-shot rollover, and the ambient fire loop.

## v0.7.1 spatial metadata closure

Build Game accepts Scene 18 spatial source/listener metadata without adding a package
dependency. Referenced WAVs remain exact `audio-clip` files, the fixed miniaudio notice remains
mandatory, Player configuration remains 2, Build Manifest remains 1, and the manifest vocabulary
does not change. The mono fireplace derivative is copied only when the selected startup Scene
references it; the unused stereo original remains excluded.

## v0.7.0 audio closure

The v0.7.0 release keeps Player configuration 2 and Build Manifest 1 while adding one closed
manifest kind, `audio-clip`. Build Game reparses every supported Scene through the shared current parser,
validates/deduplicates each referenced
contained WAV beneath `assets/audio`, captures and copies its exact bytes at the same identity, and
excludes unused WAV plus all unreferenced OGG/glTF/FBX content. Because miniaudio is statically compiled into the
Player, every package also contains its exact fixed-digest MIT-0 notice once. Missing, changed,
escaping, malformed, colliding audio or notice bytes roll back the existing staging transaction.
The exact miniaudio notice is `ThirdParty/miniaudio/0.11.25/LICENSE`.

## v0.6.8 runtime preference separation

Build Game accepts Scene 16, including the field-free Input Preferences launcher, through the
shared parser. Player configuration remains format 2 and Build Manifest remains format 1. The
package contains the exact validated authored Input Action bytes and never contains an
`input-preferences.json`, Local App Data profile directory, effective generated map, or new
manifest kind. Player derives its stable preference profile from validated game name, startup
Scene identity, and Input Action identity, so relocation and unrelated current directories do not
change the profile. Editor preview uses a separate namespace. See `INPUT_PREFERENCES_FORMAT.md`.

## v0.6.7 interactive Button closure

Build Game accepts strict Scene 15 and all released Scene versions through the shared transactional
parser. `UIButtonComponent` contributes no file dependency and no new manifest kind: only an
associated UI Image, if present, participates through the existing exact `canvas-image` closure.
Player configuration remains format 2, Build Manifest remains format 1, Input Action format remains
3, and the dependency-role vocabulary, staging transaction, relocation behavior, and exact runtime/
notice closure are unchanged. After the amended controller correction, the packaged Release Player
must own the same PlaySession Button state, per-control input authorization, and native Behaviour
activation path as editor Play.

The owner-approved v0.6.7 replacement `assets/images/logo.png` at SHA-256
`B621A200ADEB5AA379B3A3C83C13948EB3844C0F69A9A7079BF846E07FE6482F` is now the exact source for
the existing mandatory `startup-logo` role and any deduplicated Canvas image role. The separately
approved `assets/images/splash.png` remains reserved; the former `assets/images/icon.png` moves in
v0.12.5 to internal `MoeMagicEngine/Resources/Branding/MoeMagic.png` and supplies embedded executable/
window identity. These are not project dependencies. At the original release they were reserved assets,
not parser-proven dependencies; Build Game must not scan or package them merely because they exist.
No game-manifest asset kind or project dependency is added for them. The icon is carried inside
the current Player template; the original v0.6.7 milestone added no executable-resource integration.

## v0.6.5 exact authored Canvas image closure

Build Game parses the selected Scene 14 document and collects every explicit `UIImageComponent`
identity. Each identity must resolve to an existing regular PNG/JPG/JPEG file lexically and
canonically contained beneath `assets/images`; exact captured bytes must satisfy the shared 4 MiB,
4096-per-dimension, 16-megapixel RGB/RGBA decode contract. The validated bytes are copied at the
same normalized project-relative identity. Destination comparison and deduplication are
case-insensitive, with deterministic role priority `startup-logo` over `canvas-image` over material
`texture`. A non-logo Canvas image is listed as `canvas-image`; the Canvas Showcase reuses the
mandatory logo, so its one manifest entry correctly remains `startup-logo`.

Missing, unreadable, escaping, changed, oversized, corrupt, unsupported-channel, or colliding
Canvas dependencies fail the staging transaction and publish no final directory. Player
configuration remains format 2 and Build Manifest remains format 1; only the closed file-kind
vocabulary gains `canvas-image`.

## v0.6.4 exact format-3 input closure

Build Game accepts strict Input Action formats 1, 2, and 3 through the shared production parser,
then packages the exact validated disk bytes at the configured contained identity. The shipped
project default is canonical format 3 with Left Stick Move, Right X Turn, and Face South Jump after
the existing Keyboard bindings. Changed-source detection, rollback, and manifest kind
`input-actions` are unchanged. Player configuration remains format 2 and Build Manifest remains
format 1; no device identity, mapping database, controller DLL, or preference file is packaged.

## v0.6.3 fixed startup-logo closure

Every Build Game package now contains the exact validated bytes from the fixed engine identity
`assets/images/logo.png`. The source must be a contained regular RGB/RGBA image within the startup
decoder's fixed 4 MiB encoded-byte, 4096-per-dimension, and 16-megapixel bounds. Missing, escaping,
empty, oversized, malformed, unsupported-channel, or changed-after-validation source bytes fail the
transaction and publish no final directory. The manifest lists exactly one copy at that identity
with kind `startup-logo` and its exact byte size.

The logo is mandatory build-time Moe Magic branding, not Scene content or Player configuration.
Deleting or corrupting it after a valid package was published is treated as package tampering:
Player emits one presentation warning, skips only the splash, and starts an otherwise valid game.
Neither Player configuration format 2 nor Build Manifest format 1 changes in v0.6.3.

## v0.6.2 saved input-action authority

`MoeMagicPlayer.json` output is format 2 and contains exactly `format`, `version`, `gameName`,
`startupScene`, and `inputActions`. The reader continues to accept strict format 1; format 1
implicitly selects `assets/input/input_actions.json`, while format 2 requires a normalized path
beneath `assets/input`. Build Game always writes format 2, copies the validated default map at its
project-relative identity, and lists it once as `input-actions` in the deterministic manifest.
The editor requires its project-default Input Action document to be available and clean before it
opens the output-folder picker. Current Save publishes canonical format-3 source; Build Game then reparses
that exact disk source, retains the validated bytes, and preserves the released changed-source
rollback during publication. Invalid, dirty, unavailable, or externally changed input content
fails the existing transactional publication boundary.
The contained map may be strict Input Action format 1, 2, or 3; the shipped project default is
format 3 with tagged Keyboard and standard Gamepad controls. Player configuration remains format 2 and the Build Manifest
remains format 1.

Build Game emits two deterministic UTF-8 JSON files beside the packaged Windows x64 Player. Both
end with one newline, contain no absolute path, and remain independent of current Scene version 28
and prefab format 22.

v0.5.4 changes only the embedded engine version and supported startup-Scene collider content. It does
not change either Player JSON schema, their deterministic inventory rules, or package closure.

## `MoeMagicPlayer.json`

```json
{
  "format": "MoeMagicPlayerConfiguration",
  "version": 3,
  "gameName": "Ultimate Platformer Coin Run",
  "startupScene": "assets/scenes/main_menu.json",
  "includedScenes": [
    "assets/scenes/main_menu.json",
    "assets/scenes/default_scene.json"
  ],
  "inputActions": "assets/input/input_actions.json"
}
```

`format` and unsigned integer `version` are exact. `gameName` is nonempty validated UTF-8, at most
128 bytes, and contains no control characters. `startupScene` is a normalized forward-slash,
project-relative `.json` identity lexically and canonically beneath `assets/scenes`. Format 3
requires an ordered 1–64 `includedScenes` array under the same identity rules, unique under Windows
case folding, with startup present exactly once. It also requires `inputActions` as a normalized
`.json` identity lexically and canonically beneath `assets/input`. Strict formats 1 and 2 remain
readable and migrate to a singleton included set; format 1 supplies the project-default input
identity. Save emits fields in the illustrated order and only version 3. Loading is transactional.
The Player uses this file only for zero-argument launch; explicit CLI does not merge it, and a
missing file preserves the repository-CWD development fallback.

## `MoeMagicBuildManifest.json`

```json
{
  "format": "MoeMagicBuildManifest",
  "version": 2,
  "engineVersion": "0.9.0",
  "platform": "Windows-x64",
  "configuration": "Release",
  "gameName": "Ultimate Platformer Coin Run",
  "startupScene": "assets/scenes/main_menu.json",
  "includedScenes": [
    "assets/scenes/main_menu.json",
    "assets/scenes/default_scene.json"
  ],
  "playerConfigurationVersion": 3,
  "files": [
    { "path": "Ultimate Platformer Coin Run.exe", "kind": "player", "size": 1044992 }
  ]
}
```

`includedScenes` exactly matches configuration 3 in authored order, and startup appears once.
`playerConfigurationVersion` is exactly 3. `files` is sorted by case-insensitive Windows path
identity with stable original-path ties. Paths
are normalized package-relative identities, unique case-insensitively, and use one of `player`,
`runtimeDependency`, `scene`, `input-actions`, `canvas-image`, `audio-clip`, `model`, `model-buffer`, `material`,
`texture`, or `notice`. `size` is the exact
unsigned byte count after copy. The manifest intentionally excludes itself to avoid recursive
content; complete inventory is the manifest plus every listed entry. It contains no timestamp,
hash, random stage suffix, machine/user path, pointer, or duration.

v0.9.0 does not change configuration 3 or Manifest 2. Direct and packaged Players still request
OpenGL 3.3 Core, validate copied actual capability facts before GPU resource publication, and print
one bounded vendor/renderer/OpenGL/GLSL/profile/limit summary to the existing console diagnostic
stream. Unsupported graphics startup remains a nonzero platform-initialization failure; HDR target
or resolve failure during gameplay remains a nonzero runtime failure with orderly cleanup. No GPU
fingerprint, quality setting, exposure value, or graphics diagnostic is persisted in either JSON
document or added to the package inventory.

## Package layout and publication

The package contains only the renamed Release Player, the exact repository-vendored
`ThirdParty/GLEW/bin/windows-x64/glew32.dll`, these two JSON files, every declared Scene at its
project-relative path, the union of parser-derived used model/material/buffer/texture dependencies,
the validated input-action map at its exact project-relative `assets/input` identity, required
notices for approved used model roots, and the exact ten mandatory GLFW, GLEW, GLM, Jolt,
miniaudio, Lua, cgltf, TinyObjLoader, stb, and JSON notices at
`ThirdParty/GLFW/LICENSE.md`, `ThirdParty/GLEW/LICENSE.txt`, `ThirdParty/GLM/copying.txt`, and
`ThirdParty/Jolt/LICENSE`, `ThirdParty/miniaudio/0.11.25/LICENSE`, and
`ThirdParty/Lua/5.5.1/LICENSE.html`, `ThirdParty/cgltf/1.15/LICENSE`,
`ThirdParty/TinyObjLoader/LICENSE`, `ThirdParty/Stb/LICENSE.txt`, and
`ThirdParty/Json/LICENSE.MIT`. Each mandatory dependency notice is present and listed exactly once as
`notice` in every package, including procedural-only packages: GLFW and Jolt are statically linked,
GLM is compiled into the Player, and GLEW is shipped dynamically. Each complete notice must match
its fixed approved SHA-256; identifying text alone is insufficient. Staged publication reopens these
validated source bytes and the Release GLEW template before copying, so same-operation changes
roll back without publishing a partial package. The package excludes undeclared Scenes and unrelated models,
unreferenced prefabs, headers, static/import libraries, provenance READMEs, editor/source/build files, debug
symbols, personal layout, and ignored unaudited content.

Build Game creates a unique sibling stage under the selected existing parent, verifies the complete
regular-file/non-reparse inventory, and renames the whole stage to a nonexistent Scene-name-derived
final directory. It never overwrites, merges, repairs, increments, archives, installs, signs, or
deletes an existing final. This format describes a portable development folder, not an installer or
general project/build-profile system.
