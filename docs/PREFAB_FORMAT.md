# Moe Magic Prefab Format

## v1.12 release — action gameplay records

Ordinary prefabs write **41** and read **1–41**. Scene47 shares the four new explicit records:
Projectile, Interactor, Interactable and Camera Framing Zone. All start disabled; settings/ranges
validate in disabled drafts too. See [Scene Format](SCENE_FORMAT.md#version-47--composable-action-gameplay)
and [Action Gameplay](ACTION_GAMEPLAY.md) for exact fields, defaults, capacities and requirements.

Only Projectile joins the prepared runtime-actor allowlist. Enabled use requires the validated
unit-scale root, no actor Collider/Rigid Body/Character, no competing root motion, and atomic API23
`fire_projectile`. Ordinary spawn rejects enabled Projectile. Interactor, Interactable and framing
zones are ordinary editor-prefab data only and remain rejected runtime records even disabled.
Framing Camera references use normal complete-subtree capture/remapping and None repair.

Variant wrapper1 remains unchanged. The new whitelist exposes authored enabled (default false)
and each safe scalar/vector setting, including bounded prompt/action-name strings, but **not** the
framing Camera reference. Whole-record validators still run after overrides. Existing Gem variants,
base IDs/digests, reference prohibitions and explicit refresh semantics remain unchanged. Descriptor
coverage across all engine types is partial, not universal reflection. The model descriptor names
only supported OBJ/glTF/FBX; it does not claim GLB import support. Historical formats do not gain
implicit new Components. Runtime launch/interaction/framing state is never prefab data.

## v1.11 release — single-base variants and typed asset properties

At the v1.11 release, ordinary prefabs wrote **40** and read **1–40**. A separate **MoeMagicPrefabVariant 1**
wrapper uses the same project-owned `.prefab.json` path family. It requires exactly `format`,
`version`, `name`, `basePrefab`, `baseDigest`, `overrides`. `basePrefab` is one canonical ordinary
project prefab, never another variant. `baseDigest` is lowercase SHA-256 of its exact authored
bytes. Wrapper limit: 256 KiB, 64 overrides, 64 resolved objects; ordinary base limit: 4 MiB.

Each override has exactly `objectId`, `componentType`, `property`, `value`. IDs refer to the base's
file-local objects. Component names are the existing serialized singleton names, or `GameObject`
and `Transform`. Duplicate keys, missing targets, unknown fields, type mismatches, nested variants,
external paths and stale base digests fail without changing the destination. Output ordering is
canonical by object/component/property. There is no hierarchy editing or linked Scene instance.

| Component | Editable descriptor IDs |
| --- | --- |
| GameObject | `name`, `active` |
| Transform | `localPosition`, `localRotationDegrees`, `localScale` |
| Existing optional component with authored enabled state | `enabled` |
| MeshRenderer | `model`, `materialPath`, `tint` |
| HealthComponent | `maximumHealth`, `startingHealth` (validated together) |
| RotatorBehaviour | `angularVelocity` (degrees/second) |
| AudioSourceComponent | `clipPath`, `volume` |
| ParticleBurstComponent | `particleCount`, `startColor`, `endColor` |
| ScriptBehaviour | `scriptProperty.<existingName>` for Boolean/Number/String/Vector3 only |

Vectors/colors are whole arrays. `model` contains the existing complete selection union:
`sourceType: Procedural` with `primitiveType`/`primitiveParameters`, or `sourceType: ImportedModel`
with `modelPath`. Other renderer fields are not model selection. Existing component ranges,
resource compatibility and internal references remain authoritative; references and script paths
cannot be overridden. No native asset/GL work occurs in the pure resolver.

Asset Browser context actions **Create Variant** and **Edit Prefab Properties / Inspect Base**
open a separate typed asset draft. The panel shows the read-only hierarchy, effective/inherited
values and override markers. Save/Discard/Cancel protects asset change/close, Play, Build and Exit;
asset edits do not enter Scene history. Save checks source bytes and uses atomic replacement.
An ordinary base edit preserves its IDs/topology. A changed base requires **Inspect / Refresh
Base**, review of the proposed effective properties, explicit acknowledgment, then **Save**.
Invalid targets can be removed explicitly; they are never guessed by object name.

Instantiation captures resolved independent data once. Undo/Redo does not reread the source.
Runtime actors retain their existing allowlist and 64-object/depth-eight limits. The 32-document
closure counts unique variants and ordinary bases; a base does not add a spawn level. Windows
and Web capture variant/base bytes and effective resources in the same saved-content snapshot.
Overridden-away base resources are not loaded or packaged unless independently referenced.
Implementation evidence and remaining checkpoints are in the v1.11 implementation report;
this section is not release acceptance.

Prefab assets are deterministic UTF-8 JSON files beneath `assets/prefabs/` with the complete
case-insensitive suffix `.prefab.json`. Format 40 uses marker `MoeMagicPrefab` and stores exactly one
authored GameObject subtree:

v1.10.0 saves **40** and reads **1–40**. Animation Constraint uses the exact tagged
[Scene 46 record](SCENE_FORMAT.md#version-46--authored-animation-constraints), following its
MeshRenderer/Animator. Formats 1–39 reject this record. Target/pole references use the existing
complete-subtree mapping and fresh IDs; external references reject capture, while None drafts
remain valid. Target deletion repairs None and Undo restores mapped identities. No pose, solver
state, status, override or resource pointer is prefab data. Runtime-spawnable actors reject the
Component even disabled; existing unconstrained animated actors remain supported.

v1.9.0 saves **39** and reads **1–39**. Physics Constraint, Ragdoll and Breakable `terminalAction`
share the [Scene 45 records](SCENE_FORMAT.md#version-45--physical-mechanisms-and-terminal-ragdolls).
Older formats reject these new records/field and migrate Breakable to Destroy. Internal body/visual
references remap to fresh IDs; external references still reject capture. World-anchor positions
remain world-space when an instance is moved. Only authored mapping/settings persist. Runtime
actors reject both new Components even disabled and reject Breakable Ragdoll mode. See the
[mechanism contract](PHYSICAL_MECHANISMS.md) for dependencies, capacities and terminal ownership.

v1.8.0 saves **38** and reads **1–38**. Navigation Surface, NavAgent, Perception and Agent State
Controller use the exact [Scene 44 records](SCENE_FORMAT.md#version-44--navigation-perception-and-agent-states).
Formats 1–37 reject these records. Ordinary instances remap internal Surface/target references;
external references must be explicitly cleared before capture under the existing prefab policy.
World-space bake geometry is not relocated with an instance: its saved source/profile fingerprint
must match before Play/Build. Runtime actors reject all four Components even disabled. No route,
observation, controller state, timer or command queue is prefab data.

v1.7.0 saves **37** and reads **1–37**. The inline Animation State Controller uses the exact
[Scene 43](SCENE_FORMAT.md#version-43--inline-animation-state-controller) authored record and
same-object Animator dependency. Ordinary instances copy definitions with fresh object IDs;
runtime parameters, pose/blend state and marker sequences never persist. Formats 1–36 reject the
new record. Runtime-spawnable actor templates reject controllers even disabled; ordinary animated
actors without controllers retain their existing support.

The v1.6.0 release saves **36** and reads **1–36**. It adds the vehicle/input
records, FollowCamera `targetOrientation` and Checkpoint `receiverLayerMask` documented in
[Scene 42](SCENE_FORMAT.md). Formats 1–35 migrate Upright and the old Checkpoint mask while rejecting
premature vehicle/new fields. Four wheel pivots use the common internal-reference mapping: None
drafts persist, outside-subtree references reject, and independent instances get their own IDs.
Controller/Input remain excluded from runtime actor templates even disabled. Ordinary editor
prefabs support configuration only, never live constraints, drivetrain/wheel state or control queues.

The v1.5.0 release saves **35** and reads **1–35**. It adds PathFollower/FollowCamera and the
three Motor basis fields described in [Scene 41](SCENE_FORMAT.md). Older prefabs migrate Motors
to Legacy/0/None and reject premature new records/fields. Ordinary capture/instantiate remaps
target and movement-Camera references; external references reject, deletion repairs None and Undo
restores them. PathFollower/FollowCamera and Character Motors are not runtime actor Components,
even when disabled. Runtime path/damping/control state is never captured.

The v1.4.0 release saves **34** and reads **1–34**. It adds Respawnable, Checkpoint, Objective
Counter and Conditional Reward with the exact fields/defaults/bounds in [Scene 40](SCENE_FORMAT.md).
Formats 1–33 migrate without them and reject injected progression records. Ordinary editor prefabs
support these Components and remap their ordered member/counter references to instantiated IDs.
Capture rejects references outside the captured subtree; None slots remain incomplete Edit state,
not external reference injection. Typed deletion repairs references and Undo restores them.
The four Components are excluded from runtime-spawnable actor templates even when disabled;
ordinary Green/Pink reward actors use the existing allowlist. No checkpoint/progress/result state
is captured or shared across instances.

The v1.3.0 release saves **33**, loads **1–33**, and adds Stompable, Contact Damage and Breakable
with the exact fields/defaults/dependencies in [Scene 39](SCENE_FORMAT.md). Formats 1–32 migrate
without them and reject injected interaction records. All three are eligible runtime actor
Components; they retain detached configuration only, never source contact/break state. Admission
and pending reservations share the session's 256 interaction-object bound, including dormant actors.
No broader actor factory or new prefab instancing semantics are introduced.

v1.1.0's supported static FBX uses the existing ImportedModel `modelPath`; prefab format 31
does not change. [Project provenance](ASSET_LIBRARY_AND_PROVENANCE.md) lives beside asset bundles,
not in prefab records. Instantiation retains existing shared resource ownership.

Released v1.2.0 saves **32**, loads **1–32**, and adds the exact Prefab Spawner
record documented in [Scene 38](SCENE_FORMAT.md). Formats 1–31 migrate without it and reject an
injected new record. Exact optional-Component order and internal Script reference remapping remain.

## Runtime actor sources (v1.2.0)

An authored spawner prepares a project-owned immutable actor before Play. The allowlist is
Transform, MeshRenderer, Animator, Script/Rotator/Mover/DestroyAfterTime/DestroyOnCollision,
Collider/RigidBody/KinematicMotion, Health/DamageOnTrigger, AudioSource, ParticleBurst and
PrefabSpawner; v1.3.0 additionally permits Stompable, ContactDamage and Breakable. Other types reject
even when disabled. In particular, Camera, Listener, Light/Sky,
Canvas/UI, Character/Planar/Pot controllers and deprecated collection policy are not runtime actors.
Ordinary editor prefab capture/instantiation retains its wider vocabulary.

Sources form an explicit acyclic graph: at most 32 distinct templates, depth 8 (first source = 1),
and 64 objects per template including inactive descendants. Enabled-empty references fail preflight;
all nonempty references are followed, including disabled ones. Models/audio/scripts/materials and
provenance are preloaded/packaged through the same project root. Templates need no primary Camera
or listener of their own; spatial sound uses the host Scene's listener. Root-only physics rules still
apply. Runtime copies use fresh IDs, remapped internal references, requested world root position/
Euler rotation and authored scale/child transforms; no external reference injection or runtime parent.
See [API 13](SCRIPTING_API.md) for request/result and instance limits.

The approved v0.12.2 plan
retains format 31 and existing typed Script reference remapping. Its runtime controls/UI state
are not prefab fields. Deprecated collection records remain readable; no automatic guessed Lua
conversion of unrelated prefabs is performed. The released implementation uses existing Script
properties for authored policy and existing Canvas Components for presentation; runtime text,
visibility, health projection and simulation state never enter a prefab. Inspector Planar Motor
retains the `PotPlayerBehaviour` token. See [SCRIPTING_API.md](SCRIPTING_API.md).

The current shared authored-object codec also includes the post-v0.12.1 optional Character Motor
`cameraRelativeMovement` and Collectible `healAmount` fields while retaining format 31.
Defaults, bounds, and tagged-binary compatibility are documented once in
[Scene format](SCENE_FORMAT.md#current-post-v0121-authored-additions). Historical format records
below describe their original introductions, not a promise that old binaries accept new fields.

## Format 31 — stable internal Lua GameObject references

Format 31 adds the fifth Script Behaviour property tag `gameObject`. Its exact ordered record uses
`name`, `type`, `value`; the value is JSON `null` for None or one positive integer object ID inside
the complete captured prefab subtree. Forward, backward, self, parent, child, and inactive internal
targets are valid. Zero, malformed/fractional values, missing targets, and nonnull references to an
object outside the captured subtree reject capture/save/load transactionally with an actionable
diagnostic; external Scene references are never silently cleared.

Capture retains file-local identity only after the complete subtree is known. Load validates the
complete object-ID set before publication. Every instantiation creates all destination objects,
then maps internal GameObject properties through that instance's complete fresh-ID map. Separate
instances therefore never share internal target identities. Undo/Redo restores references from
the in-memory snapshot through fresh command-target mappings and never copies obsolete runtime IDs.
Format 30 retains the exact four-type grammar below and rejects a premature `gameObject` tag.

## Format 30 — typed authored Lua Behaviour properties

Format 30 mirrors Scene 36's four-type `ScriptBehaviour` shape. Its exact ordered fields are `type`,
boolean `enabled`, string `scriptPath`, and required array `properties`. Each property has exact
ordered fields `name`, `type`, `value`; the four lowercase tags and value shapes are `boolean` /
Boolean, `number` / finite Number, `string` / valid UTF-8 without NUL up to 1,024 encoded bytes,
and `vector3` / an exact three-finite-number array. At most 32 ordered records are allowed, and
names are unique exact 1–64-byte ASCII identifiers matching `[A-Za-z_][A-Za-z0-9_]*`.

Format 29 path-only records migrate with an empty property list; formats 1–28 still migrate with
no Script Behaviour. Capture, load, instantiation, duplication, and history copy distinct complete
engine-owned values. Missing/extra/reordered fields, premature properties, invalid names, duplicate
names, unknown or mismatched types, nonfinite values, invalid strings, and exceeded bounds reject
the complete prefab transactionally. Lua/runtime state and source bytes never enter prefab JSON.

## Format 29 — bounded Lua Script Behaviour

Format 29 mirrors Scene 35's optional path-only `ScriptBehaviour` record with exact canonical
fields `type`, boolean `enabled`, and string `scriptPath`. The path may be empty for an incomplete
Edit-time Add/Assign state; a nonempty value must be a canonical project-relative `.lua` identity
strictly beneath `assets/scripts`. Persistence validates identity syntax without reading or
executing source. Component order is captured and restored exactly, and duplicate, malformed,
extra-field, reordered-field, invalid-path, or format-1-through-28 occurrences reject the complete
prefab transactionally.

Formats 1–28 migrate with no Script Behaviour. Format-29 load copies enabled/path data and supplies
an empty property list. Lua source bytes, environments, instance tables, registry
references, callback contexts, execution bridges, allocator/instruction state, Start completion,
and runtime Transform/input/destruction/log state never enter prefab bytes. Enabled incomplete or
unavailable sources are rejected later by Play/Player/Build preflight rather than executed during
prefab authoring.

## Format 28 — Graphics Preferences launcher

Format 28 mirrors Scene 34's optional field-free
`UIGraphicsPreferencesLauncherComponent` with exact fields `type` and boolean `enabled`. It follows
its required same-object Button, and canonical Component order places it after the Input
Preferences launcher and before Quit. An enabled instance requires the same eligible enabled Rect
Transform/visual/Button beneath an active Canvas as the Scene contract. A Button may enable at most
one Graphics, Input Preferences, Quit, or Scene Transition primary action; Audio Button remains an
allowed secondary behavior. Formats 1–27 migrate with no Graphics launcher. Active/pending quality,
overlay/controller state, renderer resources, storage/profile values, and host requests never enter
prefab bytes.

## Format 27 — authored world-space particle bursts

Format 27 mirrors Scene 33's optional fixed-shape `ParticleBurstComponent` with exact canonical
fields `type`, `enabled`, `particleCount`, `lifetimeSeconds`, `initialSpeed`,
`spreadAngleDegrees`, `gravity`, `startSize`, `endSize`, `startColor`, `endColor`,
`emissiveIntensity`, and unsigned 32-bit `seed`. It uses the same bounds and not-both-zero size
invariant documented in `docs/SCENE_FORMAT.md`; formats 1–26 migrate with no particle component.
Generated particles, runtime commands/sequences, frame views, and GPU resources never persist.

## Format 26 — authored procedural sky environment

The v0.9.5 implementation advanced the then-current writer/reader to format 26 and mirrored Scene 32's
optional fixed-shape `SkyEnvironmentComponent`. RGB channels are finite `[0,1]` and both independent
intensities are finite `[0,10]`. A complete prefab subtree may contain at most one total; formats
1–25 migrate with none. Instantiation into a Scene that already owns one rejects transactionally.
Transform, copied frame values, shaders, GPU state, and derived environment response never persist.

## Format 25 — reusable Material overrides

Format 25 mirrors Scene 31 by appending required string `materialPath` to every procedural or
imported `MeshRenderer`. The value is empty or a normalized project-relative `.material.json`
identity beneath `assets/materials`; formats 1–24 migrate it empty. Capture and instantiation retain
the identity and shared immutable override, while a missing or invalid Material or optional
contained diffuse texture rejects the complete transaction. Material values, decoded textures,
caches, and GPU resources never enter prefab bytes. See `docs/MATERIAL_FORMAT.md`.

The released v0.9.4 implementation does not advance prefab persistence. The same format-25 identity
may resolve a strict format-1 or current format-2 Material through the runtime boundary;
shading model and surface values remain external Material-asset content and never enter prefab
JSON.

## Format 24 — bounded primary-Directional shadows

Format 24 mirrors Scene 30 by appending required boolean `castsShadows` after `lightingEnabled` in
the fixed Light record. Formats 1–23 synthesize false. Point, Spot, and later-Directional true
values remain dormant authored data; only the complete Scene's primary Directional can activate a
runtime shadow pass. Shadow-map resources and derived projection/filter values never persist.

## Format 23 — bounded Point and Spot lights

Format 23 mirrors Scene 29 by adding required `lightType`, `range`, `innerConeAngleDegrees`, and
`outerConeAngleDegrees` fields to the fixed Light record. All light types retain the same exact
shape so dormant settings survive type changes. Range is finite in `[0.01,1000]`, inner full cone
is `[0,179]`, outer full cone is `[1,179]`, and inner cannot exceed outer. Formats 1–22 migrate
existing Light records to Directional with range `10`, inner `30`, and outer `45`. A complete
prefab may contain at most eight enabled, active, `lightingEnabled` Point/Spot records; the ninth
rejects transactionally, while multiple historical Directionals remain valid.

## Format 22 — initial Button focus and Quit

Format 22 mirrors Scene 28 by appending required boolean `initiallyFocused` to each current Button
record and adding optional field-free `UIQuitGameComponent` with exact fields `type`, `enabled`.
Formats 1–21 migrate Buttons with false initial focus and never synthesize Quit. A prefab may author
at most one enabled, active, interactable initially focused Button in its stored subtree. Exact
same-object Button dependencies, primary-action exclusivity, component order, transactional
capture/load/instantiate, and Audio Button secondary coexistence match the Scene contract.

## Format 21 — Scene Transition Button Behaviour

Format 21 mirrors Scene 27 `SceneTransitionButtonBehaviour` with exact fields `type`, `enabled`,
and `targetScene`. An enabled instance requires a preceding same-object Rect Transform, one UI
visual, and UI Button and cannot be on a Canvas root. The target identity is syntax-validated only;
declaration membership and file readiness belong to Play/Build hosts. Formats 1–20 migrate without
an implicit transition instance, and runtime requests/current Scene identity never persist.

## Format 20 — authored Health and damage hazards

Format 20 mirrors Scene 26 Health, Damage-on-trigger, and field-free Collection Game Defeat
records, including identical value limits and same-object dependency rules. Authored insertion
order is preserved and both dependencies must precede the defeat bridge. Formats 1–19 migrate
without implicit Health or hazard behavior; transient current health is never prefab data.

## Format 19 — generic Collection Game player identity

Format 19 mirrors Scene 25 with field-free `CollectionGamePlayerComponent` (`type`, `enabled`).
Authored insertion order is preserved; canonical migrated order places the marker immediately
after each legacy Pot Player. Formats 1–18 synthesize one enabled marker for each Pot Player and do
not infer markers from Movers or Character Motors. Runtime Collection Game state is never prefab
data.

## Format 18 — Character jump animation phases

Format 18 mirrors Scene 24 by adding optional exact `jumpTakeoffClipName` and `landingClipName`
values immediately after `airborneClipName` on `CharacterAnimationBehaviour`. Each value is empty
or a case-sensitive clip name of at most 128 bytes in the required direct-child rigged Model.
Format 17 drivers migrate both values to empty and preserve their released behavior. Event
revisions, copied playback status, logical phases, resolved targets, commands, playheads, and
terminal poses remain runtime-only.

## Format 17 — interruptible Character animation cross-fades

Format 17 mirrors Scene 23 by adding finite `transitionDurationSeconds` in `[0,10]` immediately
after `movementSpeedThreshold` on `CharacterAnimationBehaviour`. Format 16 drivers migrate that
value to `0.15`; formats 1–15 still migrate without an implicit driver. Sampled source poses, fade
timing, playheads, command queues, resolved targets, and final runtime poses never enter prefab
data.

## Format 16 — Character animation selection

Format 16 mirrors Scene 22 by adding optional `CharacterAnimationBehaviour` after its same-object
Character Motor. It stores exact enabled, three clip-name, and finite non-negative movement-speed
threshold values. The controller root must retain exactly one direct compatible Animator child;
editor duplicate/delete/reparent/component/model transactions preserve that relationship. Formats
1–15 migrate without an implicit driver, and resolved target/logical state/commands/physics state
never enter prefab data.

## Format 15 — authored Animator

Format 15 mirrors Scene 21 by adding an optional `AnimatorComponent` immediately after its required
same-object rigged imported-model `MeshRenderer`. It stores exact `enabled`, `clipName`,
`playOnStart`, `loop`, and finite `[0.01,8]` `playbackSpeed` values while preserving authored
Component order. Formats 1–14 migrate without an implicit Animator; runtime playheads, poses,
palettes, deformed geometry, and Model resources never enter prefab data.

## Format 14 — generic imported models

Format 14 mirrors Scene 20 by replacing released OBJ-specific `ImportedObj`/
`importedObjPath` values with strict generic `ImportedModel`/`modelPath` values for contained `.obj`
and supported static `.gltf` identities while preserving exact authored Component order. Formats
1–13 migrate released OBJ values exactly. No importer dependency graph, node hierarchy, cache/GPU
state, skin, clip, or runtime state becomes prefab data.

Format 13 implements the approved v0.7.2 persistence change by adding only the closed same-object
`AudioButtonBehaviour` record while preserving exact Component order. Runtime voices, playback
position, queued commands, detached-voice state, and backend ownership never persist.

Format 12 implements the approved v0.7.1 Scene-18
spatial Audio Source fields and field-free enabled Audio Listener while preserving exact authored
Component order. Format 11 sources migrate to TwoDimensional/minimum `1.0`/maximum `25.0` with no
implicit listener.

Format 11 supports the same strict `AudioSourceComponent` record as Scene 17 while preserving exact
authored Component order. Formats 1–10 migrate without an implicit source.

```json
{
  "format": "MoeMagicPrefab",
  "version": 31,
  "name": "Platform Section",
  "rootId": 1,
  "gameObjects": [{
    "id": 1,
    "name": "Platform Section",
    "active": true,
    "parentId": 0,
    "transform": {
      "position": [0.0, 0.0, 0.0],
      "rotation": [0.0, 0.0, 0.0],
      "scale": [1.0, 1.0, 1.0]
    },
    "components": []
  }]
}
```

`name` is a nonempty display value. `rootId` is one unique nonzero file-local ID and must identify
the only object whose `parentId` is zero. Every other parent must identify another object in the
same file. Outside parents, missing parents, multiple/no roots, duplicate/zero IDs, and cycles are
invalid. Object output is stable ascending capture order. The captured root retains its authored
local Transform; only its external parent link is removed.

The `components` array records exact optional-Component insertion order. Format 36 supports the
same persistent concrete types, fields, ordering, and relationship rules documented through Scene
version 42. Rigid Body must follow its required Collider; the Vehicle Controller follows its
enabled Dynamic body and Vehicle Input follows the Controller;
Kinematic Motion must follow a Kinematic Rigid Body; Character Controller must follow its enabled
nontrigger Capsule Collider and cannot coexist with a Rigid Body; Character Motor must follow its
Character Controller. Button must follow exactly one same-object UI visual and its preceding Rect
Transform, and cannot coexist with Canvas. Its identifier and four RGBA state tints use the strict
Scene-15 contract. Each field-free Input/Graphics launcher must follow that Button, with Graphics
canonically after Input and before Quit when those disabled records coexist.
Duplicate/unknown types, missing/malformed fields, non-finite or otherwise invalid authored values,
mismatched order/value entries, dependency violations, and invalid Transform scale are rejected.

Format 1 remains load-compatible. Its legacy `TriggerVolumeComponent` becomes a trigger Box
`ColliderComponent` with the same enabled/center/size, layer 0, and mask 65535. Formats 2–4 accept
only the historical `BoxColliderComponent` spelling and migrate it exactly to the Box shape.
Format 2 migrates without a Rigid Body; format 3 may contain only Static/Dynamic Rigid Body fields;
format 4 adds the Kinematic enum and Kinematic Motion's finite world linear and local degree-valued
angular velocity fields. Formats 1–3 migrate without Kinematic Motion and reject Kinematic content.

Format 5 accepts only `ColliderComponent` and requires the common enabled, center, trigger, layer,
and mask fields plus exactly one shape's active geometry: Box `size`, Sphere `radius`, or Capsule
`radius` and total `height`. Unknown shapes, inactive-shape geometry, missing fields, and extra
fields are transactional failures. Capsule height must be strictly greater than twice its radius.
A successful legacy load publishes only a complete older-version value; the next Save emits the
canonical format-31 representation.

Format 6 adds the two character records using the exact Scene-version-12 fields and validation.
The Controller owner must be the prefab root, upright, and uniformly scaled; must have an enabled
nontrigger Capsule Collider; must have no Rigid Body; and must not combine with an enabled Mover,
Rotator, Pot Player, or Kinematic Motion Transform authority. Its `mass` is finite and positive;
`maximumPushStrength`, `gravityFactor`, `maximumStepHeight`, and `floorStickDistance` are finite and
non-negative; and `maximumSlopeAngleDegrees` is finite, positive, and below 90. Motor
`moveSpeed`, `turnSpeedDegrees`, and `jumpSpeed` are finite and non-negative. Formats 1–5 reject
either character record and migrate without implicit character content.

Format 7 adds `DestroyOnCollisionBehaviour`. Its record contains exactly `type` and boolean
`enabled`; missing, extra, malformed, duplicate, or format-1-through-6 occurrences reject the
complete load transaction. Formats 1–6 migrate without an implicit instance. Because prefab
Component order is authored, the record may occupy any position that does not violate another
Component's dependency order.

Format 8 adds the five Scene-version-14 Canvas records and their exact dependency/exclusivity
rules. A Canvas prefab may contain its Canvas only on the single prefab root and instantiates as a
Scene root. Rect/UI-only prefabs remain reusable nonrendering subtrees until placed below a Canvas.
UI Image identity is persisted but its exact contained bytes and shared texture are restored
transactionally by `RuntimeResourceService`; failed restoration publishes no partial instance.

Format 9 adds the Scene-version-15 Button record. Rect/visual/Button order is preserved exactly;
missing dependencies, multiple visuals, Canvas coincidence, invalid action identifiers or tints,
and Button content marked as formats 1–8 reject the complete prefab. Formats 1–8 migrate without
Button state. Focus, capture, navigation, presentation, and pending actions are never prefab data.
At runtime, navigation is focus-only; Mouse Left, Enter, Space, and Face South share one held press
owner; only that source's valid release activates; and presentation priority is disabled, pressed,
focused, hovered, then normal. None of these transient facts alters format 9.

Format 10 adds the Scene-version-16 Input Preferences launcher record. It contains only `type` and
boolean `enabled`, must follow its same-object Button, and owns no path, profile, storage, map,
capture, overlay, or callback state. Formats 1–9 migrate without a launcher.

Format 11 adds the exact Scene-version-17 Audio Source record. It preserves contained WAV identity,
enabled/play-on-start/loop flags, volume, and pitch only. Formats 1–10 migrate without an Audio
Source; voice/device/cursor/backend state is never prefab data.

Format 12 adds Scene-version-18 `spatialMode`, `minimumDistance`, and `maximumDistance` fields to
every Audio Source and adds the field-free Audio Listener. Format-11 sources migrate to exact
`TwoDimensional`, `1.0`, and `25.0` values with no implicit Listener. A prefab stores at most eight
Listeners and preserves the authored source/listener insertion order; backend poses and voices are
never prefab data.

Format 13 adds the Scene-version-19 `AudioButtonBehaviour` record with exact canonical keys `type`,
boolean `enabled`, and case-sensitive `action` (`Play`, `PlayOneShot`, or `Stop`). It must follow a
valid same-object Rect Transform, UI visual, Button, and Audio Source; `PlayOneShot` requires that
source to be non-looping. Duplicate records, missing or reordered dependencies, missing/extra/
duplicate/reordered keys, malformed types, and unknown action spellings reject the complete
transaction. Formats 1–12 migrate without an implicit Audio Button Behaviour. No cross-object ID,
action-name filter, clip copy, command sequence, voice, or runtime toggle state is prefab data.

The v0.5.3 physics-query view and all requests/results remain callback/runtime-only values. Prefabs
do not persist query filters, hits, probes, CharacterVirtual/proxy state, ground/velocity state,
Jump edges, fixed requests, or other native/runtime state. Format 5 exists only to persist the
generic collider shape and its active geometry; format 6 adds only authored character settings;
format 7 adds only the authored Destroy On Collision enabled state; format 8 adds only authored
Canvas layout/visual state; format 9 adds only authored Button values; format 10 adds only the
authored launcher marker; format 11 adds legacy Audio Source values; format 12 adds spatial source
values and Audio Listener; format 13 adds only Audio Button authoring; format 14 adds generic
imported-model identity for `.obj` and supported static `.gltf`; format 15 adds the authored
Animator selection for a supported one-skin animated `.gltf`; format 16 adds only the authored
Character Animation Behaviour values and direct-child dependency; format 17 adds only its bounded
transition duration; format 18 adds only its optional takeoff and landing clip names; format 19
adds the generic Collection Game Player marker and deterministic legacy Pot migration; format 20
adds authored Health, Damage-on-trigger, and Collection Game Defeat records; format 21 adds the
authored Scene Transition Button target and enabled state; format 22 adds explicit initial focus
and the field-free Quit marker; format 23 adds bounded Directional/Point/Spot Light authoring; format
24 appends the dormant-preserving primary-Directional shadow toggle; format 25 appends the optional
renderer-wide Material identity; format 26 appends the optional singleton Sky Environment; format
27 appends optional authored Particle Burst data; format 28 appends only the field-free Graphics
Preferences launcher; format 29 appends the optional path-only Script Behaviour; format 30 appends
its ordered typed properties; format 31 adds the internal-or-None GameObject property value.
Collision
listener facts, pair
history, callback contexts, event phase, and pending runtime destruction never persist.

Procedural MeshRenderer state contains its primitive type, complete validated parameters, tint,
wireframe, enabled flag, and optional Material identity. Imported renderers contain one normalized
project-relative `.obj`, supported static/one-skin animated `.gltf`, or bounded static/skeletal
`.fbx` path plus that same optional
Material identity. The override applies to every Model section in color rendering but does not
change depth, picking, bounds, collision, or animation data.
Instantiation additionally applies the existing canonical containment gate beneath `assets/models`
and rejects the complete operation when the model or a required resource cannot be restored.

Prefab files never contain Scene Camera/selection/name/path/dirty/history state; live runtime IDs
or editor target aliases; Models, Meshes, Materials, Textures, metadata caches, pointers, GPU handles,
derived bounds, Behaviour or script runtime state, input, triggers, score, or PlaySession values. They also
contain no live instance link, overrides, Apply/Revert state, nesting, variants, watchers, or source
timestamps.

Save writes and flushes a sibling `.tmp`, preserves an existing destination as `.bak` during
replacement, restores the previous file on failure, and removes transient residue. Load parses and
validates a complete `PrefabData` value before any runtime object or resource is created. Failed
load never mutates the caller's previous output.

Instantiation creates ordinary independent Scene objects with fresh runtime IDs, exact internal
hierarchy/order/values, the stored root local Transform, and no parent. Existing instances never
change when the source file changes or disappears. Undo removes the instance subtree; Redo uses the
command's in-memory snapshot and never rereads this file. Saving a Scene containing instances
writes ordinary current-version Scene-46 objects and references, with no prefab source link.
