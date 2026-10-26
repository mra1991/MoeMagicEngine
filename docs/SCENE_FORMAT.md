# Moe Magic Scene Format

Moe Magic Engine scene files are UTF-8 JSON stored beneath `assets/scenes/`. The current save format
is version 47 with marker `MoeMagicScene`; the current loader accepts versions 1 through 47 and rejects
malformed or future versions rather than guessing. The engine uses the official modular
nlohmann/json 3.12.0 distribution.

## Version 47 — composable action gameplay

v1.12 adds four optional authored records, default disabled. Exact ordered fields after `type`:

| Component | Ordered fields |
| --- | --- |
| `ProjectileComponent` | enabled, speed, lifetimeSeconds, size, damageAmount, queryLayer, hitMask, damageTargetMask, playImpactAudio, emitImpactParticles |
| `InteractorComponent` | enabled, actionName, range, originOffset, queryLayer, occlusionMask |
| `InteractableComponent` | enabled, prompt, interactionOffset, initiallyAvailable, requireLineOfSight, oneShot, cooldownSeconds |
| `CameraFramingZoneComponent` | enabled, camera, priority, distance, height, lookAtHeight, blendSeconds, exitMargin |

All numeric fields validate even on disabled drafts. `size` is full Box dimensions, not half extents;
offsets use ordinary owner-local coordinates. The framing Camera reference remaps with clone/load/
history and repairs None on deletion; encode None as `null`, otherwise use a positive local ID.
[Action Gameplay](ACTION_GAMEPLAY.md) owns exact ranges,
dependencies, capacities and admission. An enabled Projectile belongs to a prepared prefab actor,
not placed Scene movement. The other three new types are not runtime-prefab actors even disabled.

Versions 1–46 reject these premature records and load with them absent. Ordinary prefab41 shares
the strict codecs and preserves authored insertion order; Scene output appends the new records
in canonical Projectile/Interactor/Interactable/Camera Framing Zone order. Runtime focus, cooldown,
availability, launch metadata, impacts, revisions and framing blends never serialize.

## Version 46 — authored animation constraints

v1.10.0 adds `AnimationConstraintComponent`, ordered after its same-object MeshRenderer/Animator.
Its exact ordered fields are `type`, `enabled`, `rigSignature`, `entries`. The Component defaults
disabled. The ordered list has at most eight entries per visual and 64 per Scene, including drafts.
Every entry begins with `key`, `kind`, `enabled`, `weight`, `target`, `targetOffset`, followed by:

| Kind | Remaining exact ordered fields |
| --- | --- |
| `LookAt` | bonePath, aimAxis, maxAngleDegrees |
| `TwoBoneIK` | rootBonePath, midBonePath, tipBonePath, poleTarget, poleOffset, minBendDegrees, maxBendDegrees, tipOrientationWeight, targetRotationOffsetDegrees |

References are positive same-Scene IDs or null; vectors are three finite numbers. Keys are unique
ASCII `[A-Za-z0-9_-]` (1–64 bytes). Weight is 0–1. LookAt's bone-local aim axis is normalized (+Z
default); angle is 0–175 degrees (60 default). IK uses a direct-parent three-joint chain, ordered
bend limits 0–175 (defaults 0/160, zero straight), and tip orientation weight 0–1 (zero default).
Offsets default zero. Targets use full world transforms; visual/chain frames must be positive,
uniform and nonsheared. Enabled configurations, including zero-weight ones, require the exact rig
signature and valid paths. None/inactive/deleted/terminal targets retain base pose with status.

Versions 1–45 reject this record and load without implicit constraints. Prefab 40 shares the same
strict tagged codec; unknown/missing/mode-inappropriate fields reject. Internal references remap;
target deletion repairs None. Runtime actors reject this Component even disabled. Only authored
settings persist, never resolved indices, runtime overrides, statuses, poses or palettes. Saved
Windows/Web closure and explicit reimport use the shared binding validator. See
[Animation Constraints](ANIMATION_CONSTRAINTS.md) for ownership, authoring and limitations.

## Version 45 — physical mechanisms and terminal ragdolls

v1.9.0 adds two optional singleton records. Following `type`, their exact ordered fields are:

| Component | Ordered fields |
| --- | --- |
| `PhysicsConstraintComponent` | enabled, mode, bodyA, bodyB, anchorA, anchorB, frameRotationA, frameRotationB, collideConnected, hinge, distance, breakForce, breakTorque |
| `RagdollComponent` | enabled, visual, rigSignature, lifetimeSeconds, environmentMask, parts |

Modes are `Hinge` / `DistanceSpring`; `hinge` persists limitsEnabled, minAngleDegrees,
maxAngleDegrees, frictionTorque, motorEnabled, motorSpeedDegrees, maxMotorTorque. `distance`
persists minDistance, maxDistance, frequency, damping. Both records persist even when inactive.
Object references are positive IDs or null; null Body B means World, not a missing automatic anchor.
Each Ragdoll part persists key, bonePath, parentKey (string or null), shape (`Box` / `Capsule`),
offset, rotation, halfExtents, radius, halfHeight, mass, swingNormalDegrees, swingPlaneDegrees,
twistMinDegrees, twistMaxDegrees, in that order. Breakable appends `terminalAction` (`Destroy`
default or `Ragdoll`) after its prior fields. Ragdoll follows required actor dependencies in
canonical Scene order. Limits, ownership and authoring are in [Physical Mechanisms](PHYSICAL_MECHANISMS.md).

Versions 1–44 reject premature new records/terminalAction and migrate Breakable to Destroy.
Prefab 39 shares these values. Only authored configuration persists: no pose, velocity, solver
IDs, runtime retirement, break state or lifetime age. Drafts save; enabled Play/Build requires
complete preparation. Body/visual references use the existing same-Scene visitor and fresh-ID
remapping; deleting Body B also disables its connection. Rig/model saved preflight is shared by
Windows/Web publishing. Runtime actor templates exclude the new records even disabled.

## Version 44 — navigation, perception and agent states

v1.8.0 adds four optional singleton records. Fields below follow `type`, then `enabled`, in
canonical order; vectors are three-number arrays, object references are positive IDs or `null`.

| Component | Remaining ordered fields |
| --- | --- |
| `NavigationSurfaceComponent` | boundsCenter, boundsSize, sourceLayerMask, agentRadius, agentHeight, maxSlopeDegrees, maxClimb, cellSize, cellHeight, navigationPath |
| `NavAgentComponent` | surface, speed, turnSpeedDegrees, arrivalTolerance, repathInterval |
| `PerceptionComponent` | targets, eyeOffset, targetOffset, range, fieldOfViewDegrees, queryLayer, occlusionMask, sampleInterval |
| `AgentStateControllerComponent` | patrolPoints, patrolPauseSeconds, leashDistance, lostSightSeconds |

NavAgent follows its Collider and Rigid Body records; Agent State Controller follows NavAgent and
Perception. Unknown, duplicate, missing or reordered fields/dependencies reject transactionally.
Versions 1–43 reject these records and migrate without implicit navigation Components. Prefab 38
shares these records. Surface and target references participate in the ordinary same-Scene/fresh-ID
reference visitor, deletion repair and Undo. Outside-subtree prefab references retain the existing
explicit rejection policy; set them to None before saving a standalone agent prefab.

Only authored configuration persists. Paths, query nodes, current destinations, observations,
last-seen ages, home/state, timers, budgets and queued controls are PlaySession-owned runtime values.
Missing navigation drafts can be saved; Play and saved-Scene Build preflight require a matching
bake for enabled agents. The [navigation contract](NAVIGATION.md) defines defaults, physical
requirements, bounds, format-1 bake data and stale/rebake behavior. New Components remain excluded
from runtime actor templates even when disabled.

## Version 43 — inline Animation State Controller

v1.7.0 adds an optional singleton after its required same-object Animator. Canonical field order:

```json
{"type":"AnimationStateControllerComponent","enabled":true,"defaultState":"","parameters":[],"states":[],"transitions":[]}
```

Parameter records are `key,type,defaultBool,defaultNumber`; type is Bool, Number or Trigger.
Defaults are false/0 and unused defaults must remain false/0. State records are
`key,clipName,loop,markers`; marker records are `key,normalizedTime` (finite 0–1).
Transition records are `source,destination,condition,parameter,boolValue,numberValue,comparison,crossfadeSeconds`.
Conditions: BoolEquals, Number, Trigger, ClipCompleted. Number comparisons: Equal, NotEqual,
Less, LessOrEqual, Greater, GreaterOrEqual. Crossfade is finite 0–10 seconds. ClipCompleted
requires a non-looping source and empty parameter; other conditions require the matching type.
Source/destination must exist and differ. Definition order is decision/equal-time marker order.

Limits: 16 parameters, 16 states, 32 transitions, 16 markers/state. Keys are unique in their
respective lists, valid non-control UTF-8 of 1–64 bytes; clip names are exact, at most 128 bytes.
Missing default/clip choices are saveable incomplete Edit drafts, not valid enabled Play/Build
configurations. All active state clips must resolve on the same rigged Model. Enabled ownership
excludes Character Animation's resolved child and direct Play/CrossFade/Stop on that Animator.

Only authored values persist. Runtime parameters, trigger pulses, state clocks, blend snapshots,
marker batches/sequences and Scene generation never serialize. Clone/ordinary prefab/history
copy the definitions; runtime-spawnable actor templates reject this Component even disabled.
Versions 1–42 reject this record and otherwise retain their existing migrations. No mass resave.
See [API 18](SCRIPTING_API.md) for exact update/event timing.

## Version 42 — arcade vehicle configuration and recovery receivers

The v1.6.0 release adds two optional singleton records. Canonical order/defaults:

```json
{"type":"ArcadeVehicleControllerComponent","enabled":true,"wheelPivots":[null,null,null,null],"wheelRadius":0.3,"wheelWidth":0.2,"suspensionMinLength":0.1,"suspensionMaxLength":0.35,"suspensionFrequency":2,"suspensionDamping":0.7,"acceleration":6,"forwardSpeed":18,"reverseSpeed":6,"braking":10,"maxSteerDegrees":30,"steerRateDegrees":120,"grip":1,"maxTiltDegrees":60}
{"type":"VehicleInputBehaviour","enabled":true,"moveAction":"Move","brakeAction":"Brake"}
```

All numeric values are finite. Radius is 0.1–1 m; width 0.05–1 m and at most twice radius;
suspension minimum 0.02–1 m, maximum 0.05–1.5 m and strictly greater than minimum;
frequency 0.5–10 Hz; damping 0–2; nominal acceleration 0.1–20 m/s²; forward speed 1–40 m/s;
reverse speed 0.5–15 m/s and no greater than forward; nominal braking 0.1–30 m/s²;
maximum steer 1–60 degrees, steer rate 1–720 degrees/s, grip 0.1–3 and maximum tilt 30–85 degrees.
Input names use the existing bounded action-name vocabulary. Move resolves to Axis2D and Brake
to Button at Play/Build preflight; there is no hard-coded vehicle keyboard polling.

An active controller requires an upright unit-scale Scene root, enabled Dynamic Rigid Body and
solid Box Collider, ordered before the controller; Vehicle Input follows it. Four distinct direct
children are referenced in front-left/front-right/rear-left/rear-right order. Their pivots have
identity rotation/unit scale, local centers within ±10 m at maximum suspension extension, left X
less than right X and front Z less than rear Z. Local -Z is forward and +X is right. Put mesh
orientation/scale beneath these pivots: pivot transforms belong to physics. Competing motion,
physics or animation writers reject. Limits: eight authored controllers including dormant ones,
one enabled Vehicle Input. Both Components reject in runtime-spawnable actor templates.

None wheel slots are valid incomplete Edit drafts, not playable configurations. Typed inspection,
property edits, history, duplication, deletion repair and ordinary prefab references share the
existing reference visitor. Remove Vehicle Input before its controller, then controller before
Rigid Body. Whole chassis deletion is safe; independently removing a live required wheel fails Play.

FollowCamera inserts required `targetOrientation` immediately after `target`: `Upright` or
`YawOnly`. Upright preserves v41 behavior; YawOnly projects target forward onto XZ with world +Y,
retaining the last valid heading near vertical (initial fallback is authored yaw). Invalid/singular
frames remain unavailable. Checkpoint renames only its `characterLayerMask` to `receiverLayerMask`;
Stompable's Character mask is unchanged. Respawnable now supports either a valid vehicle or
Character with enabled same-object Health under the existing aggregate 16-receiver limit.

Scenes 1–41 reject injected vehicle records/new fields, migrate FollowCamera to Upright and translate
the old Checkpoint mask name. Scene 42 requires the new names and rejects obsolete/dual fields.
Existing Scenes are not mass-resaved. Drive/control queues, Jolt state, wheel poses/spin, contacts,
recovery state and Camera damping never persist. See [API 17](SCRIPTING_API.md) for runtime phases.

## Version 41 — paths, follow Cameras and explicit movement basis

The v1.5.0 release adds these optional singleton records in canonical field order:

```json
{"type":"PathFollowerComponent","enabled":true,"points":[[0,0,0],[0,0,-5]],"speed":2,"traversal":"Once","playOnStart":true,"faceTravel":false}
{"type":"FollowCameraComponent","enabled":true,"target":null,"mode":"Orbit","headingDegrees":0,"distance":8,"height":3.4,"lookAtHeight":1,"turnSpeedDegrees":120,"positionHalfLife":0.08}
```

Path points are 2–32 finite local offsets in [-1000,1000], starting at the origin. World capture
requires a positive uniformly scaled root, segment lengths at least 0.01 and total length at most
4096. Speed is 0–50; traversal is Once, Loop or PingPong (Loop includes its closing segment).
Face Travel requires horizontal segments and matching upright initial yaw. Paths may be
nonphysical or use a solid Collider plus Kinematic Rigid Body. Conflicting pose writers reject.
The Play session allows 64 enabled paths / 2048 points including inactive owners.

Follow Camera requires an enabled same-object Camera on a unit-scale root and a valid non-self,
upright target. Orbit integrates Turn once before Motors; Trailing uses target yaw plus heading;
FixedHeading retains its heading. Distance is 0.1–100, height -50–50, look-at height -10–10,
turn speed 0–720 and position half-life 0–1. Heading is finite/wrapped. Eight enabled Cameras
including inactive owners are permitted. Chains/cycles and competing Camera pose writers reject.

CharacterMotorBehaviour appends required `movementBasis` (Legacy/World/FollowCamera),
`movementYawDegrees` (finite/wrapped, default 0) and `movementCamera` (Scene ID or null).
Legacy retains the prior checkbox behavior. World ignores Turn for movement; FollowCamera
requires an Orbit or FixedHeading Camera targeting that Character. Missing runtime availability
neutralizes movement/jump while gravity continues.

Versions 1–40 gain no new Components and migrate Motor fields to Legacy/0/null; premature fields
and records reject. Target and movement-Camera references use the existing remap/delete/Undo/prefab
visitor. Incomplete Edit references are saveable; Play/Build checks enabled dependencies.
No captured path, runtime distance/heading, reset status, damping state or recovery revision persists.
See [API 16](SCRIPTING_API.md) for command phases and presentation semantics.

## Version 40 — recovery, objectives and conditional rewards

The v1.4.0 release adds four optional singleton records. Each includes exact `type` and
Boolean `enabled`; canonical remaining fields/defaults are:

| Type | Fields and defaults |
| --- | --- |
| `RespawnableComponent` | `automaticOnDepletion: true`, `respawnDelay: 0.5`, `restoredHealth: 1` |
| `CheckpointComponent` | `characterLayerMask: 65535`, `respawnOffset: [0,1,0]`, `respawnYawDegrees: 0`, `playActivationAudio: false`, `emitActivationParticles: false` |
| `ObjectiveCounterComponent` | `completionSource: "Script"` (or `"Break"`), `members: []` |
| `ConditionalRewardComponent` | `requiredCounters: []`, `spawnOffset: [0,0,0]` |

Delay is finite 0–10 seconds; restored Health is integral 1–1,000,000 and cannot exceed the enabled
Health maximum. Offsets are finite local vectors with each coordinate in [-100,100]; yaw is finite
and wrapped. The Character mask is 0–65535. Members are an ordered list of at most 256 distinct
nonnull Scene-local IDs, with null None slots permitted during Edit. Required counters are at most
four distinct nonnull IDs, also with Edit-time None slots. Save/reopen, whole-Scene clone, subtree
duplication and prefab instantiate remap these references; delete repairs slots to None and Undo
restores them. Runtime IDs are not persistent global identities.

Structural parsing permits empty/incomplete lists and disabled configuration. Play/Build separately
rejects invalid enabled relationships, including inactive objects: Respawnable requires enabled
Health and Character Controller on an upright, uniformly scaled root capsule, and excludes enabled
Breakable/Destroy After Time/Destroy On Collision. Checkpoint requires an enabled trigger and
upright decomposable pose; requested activation feedback requires enabled same-object Audio Source
or Particle Burst. Objective Counter needs a nonempty fully assigned list; Break members require
enabled Health and Breakable. Conditional Reward needs 1–4 enabled counters and an enabled nonempty
same-object Prefab Spawner. The four Components are forbidden in runtime-spawnable actor templates,
even disabled; ordinary editor prefabs support them.

Per Scene limits are 16 Respawnables, 64 Checkpoints, 16 counters, 1,024 total memberships and 16
conditional rewards. Versions 1–39 migrate with none of these records and reject injected records.
Captured checkpoint poses, recovery delay/status, completed members, reward actors/tickets and
run-lifetime level results never serialize. See [API 15](SCRIPTING_API.md) for runtime ordering.

## Version 39 — reusable interactions

The v1.3.0 release adds three optional singleton Components. Canonical field order and defaults:

```json
{"type":"StompableComponent","enabled":true,"stompDamage":1,"bounceSpeed":4.0,"minimumClosingSpeed":0.5,"maximumTopAngleDegrees":45.0,"characterLayerMask":65535}
{"type":"ContactDamageComponent","enabled":true,"damageAmount":1,"targetLayerMask":65535}
{"type":"BreakableComponent","enabled":true,"playBreakAudio":false,"emitBreakParticles":false,"spawnContents":false,"contentsOffset":[0.0,0.0,0.0]}
```

Damage is integral 1–1,000,000; masks are integral 0–65535. Bounce speed is finite 0.1–100,
minimum closing speed 0.01–100, top angle 0–60 degrees, and each local contents-offset coordinate
is finite -100–100. Historical Scenes 1–38 migrate with no interactions and reject injected new
records. Exact Component order and detached clone/history behavior remain intact. Runtime contact
episodes, Health origins, break latches, feedback snapshots and tickets never serialize.

Enabled Stompable requires enabled same-object Health, solid Collider and Rigid Body. Enabled
Contact Damage requires the solid Collider and Rigid Body. Enabled Breakable requires enabled
Health; its enabled options require an enabled non-looping Audio Source, Particle Burst and
nonempty Prefab Spawner respectively. Incomplete Edit configuration remains authorable/saveable;
Play/Build preflight rejects missing enabled dependencies, including on inactive objects. Disabled
interactions do not execute. Prefab/template preparation applies the same rules. Solid contact,
not a trigger cap or object-centre heuristic, qualifies a stomp. See [API 14](SCRIPTING_API.md).

Released v1.1.0 accepts project-relative static `.fbx` through the existing ImportedModel
`modelPath` field alongside OBJ/glTF. This is an importer capability, not a Scene migration.
Skin/morph/animated FBX is rejected; [asset-source declarations](ASSET_LIBRARY_AND_PROVENANCE.md)
are separate bundle files and never Scene Components or serialized runtime state.

Released v1.2.0 adds Scene **38**.
The optional singleton record has exactly these ordered fields:

```json
{"type":"PrefabSpawnerComponent","enabled":true,"prefabPath":"assets/prefabs/reward_gem.prefab.json","maximumLiveInstances":16}
```

The source is empty or a canonical project-relative prefab identity; maximumLiveInstances is an
integer 1–128 (default 16). Empty is a valid incomplete Edit state, but enabled-empty rejects
Play/Build even on an inactive owner. Disabled-empty is allowed; every nonempty source is prepared
and published, including disabled/inactive declarations. Versions 1–37 migrate with no spawner and
reject an injected new record. Spawned objects, tickets, live counts and prepared resources never
write back into the edit Scene. The runtime actor subset is documented in [PREFAB_FORMAT](PREFAB_FORMAT.md).

Released by owner approval: v0.12.2
keeps Scene 37 and existing authored Script/Canvas data. API 12 is a script-source contract;
runtime UI, health projections and simulation control are not new persisted Scene fields.
Current samples no longer author legacy collection-policy Components; older records remain
readable through explicit deprecated compatibility. Existing Script properties carry score,
lifetime, group references, routes and sound mappings; Canvas data carries HUD/results. Runtime
values do not serialize. Inspector Planar Motor keeps the `PotPlayerBehaviour` token and mechanics.
See [SCRIPTING_API.md](SCRIPTING_API.md) for current API 15 and retained API 1–14 contracts.

## Current post-v0.12.1 authored additions

v0.12.7 retires the root showcase Scenes, chair and their unused support files. Current
product examples are the two independent sample projects and Basic3D. Scene 37 is unchanged.
Release-era example names, IDs and hashes below are historical records, not current files or
sample-content requirements. Reusable checks now generate validation-owned current fixtures.

The September 10 game-development commits retain Scene 37 and prefab 31 but extend the shared
authored-object codec. Current serialization emits CharacterMotorBehaviour's Boolean
`cameraRelativeMovement` (absent loads as false) and CollectibleBehaviour's integer `healAmount`
(absent loads as zero; accepted range 0–1,000,000). Positive healing pickups use the existing
Health command path and remain available when the other actor is full-health or depleted;
Trigger Stay can collect them after later damage. Zero retains ordinary pickup behavior.

These are current-main source capabilities, not a claim of forward compatibility with the
v0.12.1 tagged binary. Older builds may reject newly emitted fields despite the unchanged format
number. Use a binary matching the authored content's source revision. Historical schema examples
below describe their introduction-time records; this paragraph supplies the later optional fields.
See game-development direction.

## Released format history and ownership

The released v0.11.2 editor-operation milestone does not change this format. Its copied
inspection/property values reuse the current pointer-free authored schema, while structural,
Component, and asset operations retain existing commands, setters, resource policies, and
persistence. Scene/prefab remain 37/31; no operation-result, inspection, history, resource, or AI
state is serialized. Authority:
`docs/exec-plans/v0.11.2-shared-editor-object-operations-inspection.md`.

Material override and Script source remain optional serialized relationships and may be cleared.
Canvas Image, AudioClip, and Scene-transition target remain required while their corresponding
Component exists; operation-layer Clear returns typed `RequiredResource` rejection with Remove
Component guidance and writes no invalid Scene state. Scene/prefab therefore remain 37/31.

The released v0.11.1 project-root milestone does not change this format. It adds
only a separate
read-only `ProjectSettings/project.json` descriptor and explicit editor launch/path ownership.
Scene identities remain project-relative, Scene/prefab remain 37/31, and no project/engine absolute
root is serialized into a Scene. Authority:
`docs/exec-plans/v0.11.1-explicit-project-root-external-project-launch.md`.

The released v0.11.0 milestone does not change this format. Its ignored editor-only
`MoeMagicSceneRecovery` format-1 wrapper at
`Library/Recovery/active_scene_recovery.json` embeds the exact canonical version-37
`MoeMagicScene` text plus recovery metadata and hashes. It is not a Scene asset, is not accepted by
Scene path policy, never enters Build Settings or packages, and never overwrites an authored Scene.
Ordinary Scene file Save/Load and recovery share one canonical text codec; versions 1–37 and prefab
format 31 remain unchanged. Authority:
`docs/exec-plans/v0.11.0-transactional-scene-recovery-autosave.md`.

The wrapper is UTF-8 ordered JSON with exactly these seven fields, in this order:

```json
{
  "format": "MoeMagicSceneRecovery",
  "version": 1,
  "revision": 1,
  "sourceScene": "assets/scenes/example.json",
  "sourceSha256": "<64 lowercase hexadecimal characters>",
  "sceneSha256": "<64 lowercase hexadecimal characters>",
  "sceneDocument": "{ exact canonical MoeMagicScene JSON text }"
}
```

`revision` is a positive unsigned integer. `sourceScene` and `sourceSha256` are both null for an
Untitled recovery; a normalized project-relative `assets/scenes/*.json` source may carry a null
hash when no trustworthy successful Load/Save baseline exists, which forces an unverifiable/
Save-As restore. `sceneSha256` always fingerprints the exact embedded `sceneDocument` bytes. The
outer wrapper and embedded Scene text are each bounded by 64 MiB; missing, extra, duplicate,
reordered, wrong-type, malformed-path/hash, unsupported-version, noncurrent embedded Scene, and
hash-mismatched records reject without Scene publication or silent deletion.

The released v0.10.10 milestone does not change this format. Exact API version 11 and
`get_animation_playback_status` are Lua source/runtime capability changes only. The existing
`animate_showcase.lua` identity advances in place to completion-driven Jump playback; no Scene
record, ID, object/root count, Component, property, prefab record, or Build Settings entry changes.
The focused Scene remains byte-identical at eleven objects/eight roots, 16,937 bytes, SHA-256
`E30536CA2B39A634B86F452F285EECB18532DB62CDBEF9675553B8EAA6350E3F`, and format 37; prefab
remains format 31. Authority:
`docs/exec-plans/v0.10.10-bounded-lua-animation-playback-status-query.md`.

The released v0.10.9 milestone does not change this format. It composes only
existing Scene-37 Canvas, RectTransform, Panel, Button, Text, ScriptBehaviour, and GameObject-
property records in `scripting_showcase.json`. IDs 1–8 remain protected; root ID 9 Canvas,
child ID 10 `LuaNudge` Button/script, and child ID 11 label make the focused target eleven objects/
eight roots. API version 10 and `ui_button_activated` are source content, not Scene schema. Final
release bytes/hashes are recorded in the current showcase section below. Authority:
`docs/exec-plans/v0.10.9-bounded-lua-ui-button-activation-callback.md`.

The released v0.10.8 milestone does not change this format. It adds one root to the
isolated `scripting_showcase.json` using only the existing imported Mesh Renderer, Animator, and
ScriptBehaviour records; API version 9 is source content rather than a Scene schema field. The
current eight-object/seven-root baseline and exact hashes are recorded later in this document.
Authority: `docs/exec-plans/v0.10.8-bounded-lua-animation-playback-commands.md`.

The released v0.10.7 milestone likewise does not change this format. It composes only
existing version-37 Collider, RigidBody, ScriptBehaviour, and GameObject-property records in
`scripting_showcase.json`, preserving IDs 1–6 and adding only root ID 7 `Collision Drop`. The
seven-object content baseline, exact values, sizes, and hashes are recorded later in this document.
Independent release acceptance passes. Authority:
`docs/exec-plans/v0.10.7-bounded-lua-trigger-collision-callbacks.md`.

## Version 37 — stable authored Lua GameObject references

Version 37 adds exactly one fifth `ScriptBehaviour` property tag, `gameObject`. Its value is JSON
`null` for None or one positive integer ID declared by an object in the same Scene document:

```json
{
  "name": "target",
  "type": "gameObject",
  "value": 2
}
```

The property still uses exact ordered fields `name`, `type`, `value` and counts toward the existing
32-property limit. Zero, negative/fractional/out-of-range values, missing targets, and references
outside the complete Scene reject the document transactionally. Forward, backward, self, inactive,
parent, and child targets are valid. Parsing first establishes every file-local object ID, then
resolves all GameObject properties through the complete destination-ID map before publishing the
candidate Scene. Play cloning, subtree duplication, history restoration, and Edit deletion use
their corresponding complete remap/repair transactions; the property stores no pointer, object
name, global GUID, asset identity, or cross-Scene reference.

Version 36 retains the exact four-type grammar below and migrates unchanged in memory; a
`gameObject` tag is premature in Scene 36. Prefab format 31 provides the corresponding
internal-or-None prefab contract, while prefab format 30 remains four-type-only.

## Version 36 — typed authored Lua Behaviour properties

Version 36 extends the optional `ScriptBehaviour` record with required ordered `properties`:

```json
{
  "type": "ScriptBehaviour",
  "enabled": true,
  "scriptPath": "assets/scripts/coin_spin.lua",
  "properties": [{
    "name": "degrees_per_second",
    "type": "number",
    "value": 45.0
  }]
}
```

The exact property field order is `name`, `type`, `value`; the version-36 tags are `boolean`,
`number`, `string`, and `vector3`. Boolean, Number, and String use matching JSON scalars, while Vector3 is an exact
three-number array. A Component stores at most 32 ordered records. Names are unique case-sensitive
1–64-byte ASCII identifiers matching `[A-Za-z_][A-Za-z0-9_]*`; Number and Vector3 values are
finite; String is valid UTF-8 without embedded NUL and at most 1,024 encoded bytes. Empty arrays
are valid and required on current records. Missing/extra/reordered fields, invalid or duplicate
names, unknown tags, mismatched value shapes, nonfinite values, and exceeded bounds reject the
complete document transactionally.

Version 35 records retain their exact path-only shape and migrate in memory with an empty property
list. Versions 1–34 still migrate with no Script Behaviour. Scene bytes own only authored values;
Lua tables, runtime instances, callback contexts, execution bridges, and Play-time mutation never
persist. Prefab format 30 mirrors this four-type record and prefab format 29 supplies the same empty-
property migration.

## Version 35 — bounded Lua Script Behaviour

Version 35 adds one optional path-only `ScriptBehaviour` in canonical Component order immediately
after `RotatorBehaviour` and before `MoverBehaviour`. Its complete historical record is:

```json
{
  "type": "ScriptBehaviour",
  "enabled": true,
  "scriptPath": "assets/scripts/coin_spin.lua"
}
```

The exact fields and their order are `type`, boolean `enabled`, and string `scriptPath`. The path may
be empty as an incomplete Edit-time Add/Assign state; a nonempty value must be a canonical forward-
slash, project-relative `.lua` identity strictly beneath `assets/scripts`. Persistence validates the
identity without reading or executing source. An enabled empty path, missing/invalid source, binary
chunk, or source outside the 1 MiB text limit instead rejects Play, Player startup, or Build Game
through their source-preflight boundaries. Missing, extra, malformed, reordered, duplicate, or
premature records reject the complete historical document transactionally. A `properties` field
is premature in version 35.

Versions 1–34 migrate with no Script Behaviour, and prefab format 29 mirrors the same record while
formats 1–28 migrate with none. Version-35 restoration supplies an empty property list before
ordinary clone/snapshot ownership. Lua values, source bytes, per-instance tables,
environments, registry references, allocator/hook state, callback contexts, execution bridges,
Start completion, and runtime Transform/input/destruction/log state never enter Scene or prefab
bytes.

## Version 34 — Graphics Preferences launcher

Version 34 adds optional field-free `UIGraphicsPreferencesLauncherComponent` with exact fields
`type` and boolean `enabled`. Canonical Scene output places it after
`UIInputPreferencesLauncherComponent` and before `UIQuitGameComponent`; versions 1–33 migrate with
no implicit launcher. Prefab format 28 mirrors the same record and formats 1–27 migrate with none.

An enabled Graphics launcher requires, on the same object and in dependency order, an enabled Rect
Transform, exactly one eligible enabled UI visual, and an enabled interactable Button beneath an
active Canvas. One Button may enable at most one Graphics launcher, Input Preferences launcher,
Quit marker, or Scene Transition primary action; Audio Button remains an allowed secondary
behavior. Malformed, duplicate, premature, reordered, dependency-invalid, or ambiguous records
reject the complete document transactionally. Runtime overlay state, active/pending quality,
renderer resources, Local App Data profile keys/paths/bytes, and host acknowledgements never
persist in Scene or prefab content.

## Version 33 — authored world-space particle bursts

Version 33 adds optional data-only `ParticleBurstComponent` in canonical order immediately after
`ScoredTimedItemBehaviour`. Its exact fields are `type`, `enabled`, `particleCount`,
`lifetimeSeconds`, `initialSpeed`, `spreadAngleDegrees`, `gravity`, `startSize`, `endSize`,
`startColor`, `endColor`, `emissiveIntensity`, and `seed`. `particleCount` is an integer in `1..64`;
the finite scalar ranges are lifetime `0.05..5`, speed `0..25`, world-+Y cone angle `0..180`,
downward gravity `0..50`, each size `0..10` with the pair not both zero, every RGBA channel `0..1`,
and emissive intensity `0..16`. `seed` is an unsigned 32-bit integer. Missing, duplicate, extra,
malformed, non-finite, out-of-range, wrong-type, or premature-version records reject the complete
document transactionally. Versions 1–32 migrate with no Particle Burst Component; prefab format 27
mirrors the record and formats 1–26 migrate with none.

The default and shipped Coin record is count `24`, lifetime `0.7`, initial speed `2.4`, spread
`80`, gravity `3.5`, start/end sizes `0.16`/`0.02`, start color `[1,0.55,0.08,1]`, end color
`[1,0.15,0.02,0]`, emissive intensity `3`, seed `9099`, and `enabled: true`.

The Component stores authoring data only. During Play, an accepted same-object scored collection
copies the enabled template and committed world origin into the bounded PlaySession particle world
before deferred owner destruction. Expiration and rejected/already-resolved collection paths do not
emit. Generated particles, age, runtime commands, frame views, renderer/GPU resources, and burst
sequence never persist. Continuous emission, textures, local-space simulation, collision, trails,
rotation, curves, alternate blend modes, and editor preview are not part of this format.

## Version 32 — authored procedural sky environment

The v0.9.5 implementation advanced the then-current writer/reader to version 32 and added optional
`SkyEnvironmentComponent` with exact fields `type`, `enabled`, `zenithColor`, `horizonColor`,
`groundColor`, `backgroundIntensity`, and `lightingIntensity`. RGB channels are finite `[0,1]`;
both intensities are finite `[0,10]`. A complete Scene may contain at most one such Component total,
enabled or disabled. Its owning Transform has no rendering meaning. Versions 1–31 migrate with no
environment and retain the released fixed background/PBR fallback. Runtime linear colors, Camera
rays, shaders, GPU state, and environment response never persist. Prefab format 26 mirrors the
record.

## Version 31 — reusable Material overrides

Version 31 appends required string `materialPath` to every procedural or imported `MeshRenderer`.
The value is either empty or one normalized project-relative identity beneath `assets/materials`
with the complete case-insensitive `.material.json` suffix. Versions 1–30 synthesize the empty
value, preserving their procedural defaults and imported section Materials. A nonempty current
value resolves transactionally through `RuntimeResourceService`; missing, invalid, conflicting, or
unrestorable Material/texture content rejects the complete runtime Scene replacement. The resolved
shared Material pointer, cache state, decoded texture, and GPU resources never persist. Prefab
format 25 mirrors this identity and formats 1–24 migrate it empty.

The released v0.9.4 implementation does not advance Scene persistence. The same version-31 identity
may resolve a strict format-1 or current format-2 Material through the runtime boundary;
shading model and surface values remain external Material-asset content and never enter Scene JSON.

## Version 30 — bounded primary-Directional shadows

Version 30 appends required boolean `castsShadows` after `lightingEnabled` in the fixed-shape Light
record. The value defaults false and survives type changes; only the enabled, effectively active,
lowest-stable-ID Directional can activate the shared shadow pass. Point, Spot, and later
Directional values are valid dormant data. Versions 1–29 synthesize false without rewriting their
source. Shadow projection, depth resources, filtering, bias, and runtime visibility never persist.
Prefab format 24 mirrors this field and formats 1–23 migrate it to false.

## Version 29 — bounded Point and Spot lights

Version 29 expands the existing Light record with required `lightType`, `range`,
`innerConeAngleDegrees`, and `outerConeAngleDegrees` fields. Current records retain one fixed shape
regardless of type so switching Directional/Point/Spot preserves dormant tuning. Range is finite in
`[0.01,1000]`; inner full cone is `[0,179]`; outer full cone is `[1,179]`; inner cannot exceed
outer. Versions 1–28 migrate every existing Light to Directional with range `10`, inner `30`, and
outer `45`, preserving the released lowest-stable-ID Directional behavior.

At most eight enabled, `lightingEnabled`, effectively active Point/Spot records may contribute in a
complete Scene. A ninth rejects actionably; multiple historical Directionals remain valid because
only the lowest stable ID contributes. Point/Spot frame state, attenuation results, shader
uniforms, and editor guide geometry remain derived runtime/editor data and never persist. Prefab
format 24 retains the matching v0.9.1 fields in addition to its v0.9.2 shadow toggle.

## Version 28 — initial Button focus and Quit

Version 28 appends required boolean `initiallyFocused` to the canonical `UIButtonComponent` record
after `disabledTint` and adds optional field-free `UIQuitGameComponent` with exact fields `type`,
`enabled`. Versions 1–27 migrate Buttons with `initiallyFocused = false` and never synthesize Quit.
A complete Scene may have at most one enabled, effectively active, interactable initially focused
Button. Each enabled Input Preferences, Quit, or Scene Transition primary action requires the same
ordered Rect Transform, one UI visual, and Button beneath a Canvas; one Button may enable at most
one of those primary actions. Audio Button remains an allowed secondary behavior. Scene 34 extends
that same primary-action set with the Graphics launcher documented above; current saves enforce
exact fields, component order, and the complete relationships.

## Version 27 — Scene Transition Button Behaviour

Version 27 adds optional `SceneTransitionButtonBehaviour` in exact field order `type`, `enabled`,
`targetScene`. The target is a bounded normalized project-relative `.json` identity beneath
`assets/scenes`; persistence validates syntax only and performs no file access. An enabled instance
requires a preceding same-object Rect Transform, one UI visual, and UI Button, and cannot be on a
Canvas root. Runtime/Build hosts separately require exact membership in the immutable Build
Settings declaration. Versions 1–26 migrate without an implicit transition instance.

## Version 26 — authored Health and damage hazards

Version 26 adds `HealthComponent` in canonical field order `type`, `enabled`, `maximumHealth`,
`startingHealth`; `DamageOnTriggerBehaviour` with `type`, `enabled`, `damageAmount`; and field-free
`CollectionGameDefeatBehaviour` with `type`, `enabled`. Health and damage integers are in
`[1,1000000]`, starting health cannot exceed maximum, and an enabled defeat bridge requires enabled
same-object Health and Collection Game Player records. Current health, damage queues, revisions,
and terminal observation latches are PlaySession-only. Versions 1–25 migrate without these records.

## Version 25 — generic Collection Game player identity

Version 25 adds optional field-free `CollectionGamePlayerComponent` with exact fields `type` and
boolean `enabled`, canonically immediately after `PotPlayerBehaviour` when both exist. An enabled
Collection Game requires exactly one enabled, effectively active marker and at least one scored
timed item; movement/controller type is irrelevant. Versions 1–24 synthesize one enabled marker
for every authored Pot Player and otherwise add none. Runtime score, remaining count, completion,
resolved items, and stable-ID mappings are never serialized.

## Version 24 — Character jump animation phases

Version 24 adds optional case-sensitive `jumpTakeoffClipName` and `landingClipName` strings
immediately after `airborneClipName` in each `CharacterAnimationBehaviour`. Each is either empty or
an exact clip name of at most 128 bytes in the required direct-child rigged Model. Scene 23 drivers
migrate both values to empty, preserving their released Idle/Run/Airborne policy. Successful fixed-
step revisions, accepted-jump and landing revisions, copied playback status, logical phases,
resolved targets, commands, playheads, and terminal poses remain PlaySession/runtime state and are
never serialized.

## Version 23 — interruptible Character animation cross-fades

Version 23 adds finite `transitionDurationSeconds` in `[0,10]` immediately after the existing
`movementSpeedThreshold` in each `CharacterAnimationBehaviour`. Scene 22 drivers migrate that
value to `0.15`; versions 1–21 still migrate without an implicit driver. Current saves require the
field and preserve the existing controller, motor, direct-child Animator, and exact-clip contract.
Resolved targets, sampled source poses, fade elapsed time, playheads, final poses, and command
queues remain PlaySession-owned runtime state and are never serialized.

## Version 22 — runtime Character animation selection

Version 22 adds one optional `CharacterAnimationBehaviour` after its same-object Character Motor.
Its exact canonical fields are `type`, boolean `enabled`, exact nonempty case-sensitive
`groundedIdleClipName`, `groundedMoveClipName`, and `airborneClipName` values (each at most 128
bytes), plus finite non-negative `movementSpeedThreshold`. An enabled instance requires the
same-object enabled Character Controller/Motor and exactly one direct child with enabled imported
rigged MeshRenderer before enabled Animator; all three names must resolve in that Model. Versions
1–21 migrate without an implicit driver. Resolved target IDs, logical state, command queues,
committed grounded/requested-speed state, playheads, and poses remain runtime-only.

## Version 21 — authored skeletal clip playback

Version 21 adds one optional same-object `AnimatorComponent` after a rigged imported-model
`MeshRenderer`. Its exact canonical fields are `type`, boolean `enabled`, exact case-sensitive
nonempty UTF-8 `clipName` (at most 128 bytes), boolean `playOnStart`, boolean `loop`, and finite
`playbackSpeed` in `[0.01,8]`. The referenced Model must contain one supported skin and the exact
named clip. Versions 1–20 migrate without an implicit Animator. Playheads, sampled poses, palettes,
deformed CPU geometry, and GPU state remain runtime-only and are never serialized.

## Version 20 — generic imported models

Version 20 replaces OBJ-specific `sourceType: "ImportedObj"` and `importedObjPath` with
strict generic `sourceType: "ImportedModel"` and `modelPath`, accepting contained `.obj` and the
supported static `.gltf` identities. Versions 1–19 migrate released OBJ values exactly;
version-20-and-later documents reject legacy names. Imported node graphs, skin data, clip data,
dependency lists, parser/cache/GPU state, and material graphs remain immutable Model resources;
only the version-21 Animator's authored selection values persist.

Version 19 implements the approved v0.7.2 persistence change by adding only the closed same-object
`AudioButtonBehaviour` record. Runtime voices, playback position, command queues, detached-voice
state, and backend ownership remain transient and never persist.

Version 18 implements the approved v0.7.1 spatial contract. Existing Scene-17
Audio Sources migrate to `TwoDimensional` with minimum distance `1.0` and maximum distance `25.0`;
versions 1–17 migrate without an implicit Audio Listener. Scene 18 adds exact `spatialMode`,
`minimumDistance`, and `maximumDistance` Audio Source fields plus a field-free enabled
`AudioListenerComponent`.

Version 17 adds one optional `AudioSourceComponent`. Its exact record contains `type`, `enabled`,
contained WAV `clipPath`, `playOnStart`, `loop`, finite volume `[0,1]`, and finite pitch
`[0.25,4]`; versions 1–16 migrate without an implicit source. A Scene has at most 128 Audio Sources,
and Save/Play/Build reject an editor-only source whose clip is still unassigned.

```json
{
  "format": "MoeMagicScene",
  "version": 35,
  "sceneName": "Pot Pursuit",
  "selectedObjectId": 2,
  "camera": {
    "position": [0.0, 0.0, 8.0], "yaw": -90.0, "pitch": 0.0,
    "fov": 45.0, "movementSpeed": 2.5
  },
  "gameObjects": [{
    "id": 1, "name": "Gameplay Sample", "active": true, "parentId": 0,
    "transform": { "position": [0,0,0], "rotation": [0,0,0], "scale": [1,1,1] },
    "components": [{
      "type": "RotatorBehaviour", "enabled": true,
      "angularVelocity": [0.0, 45.0, 0.0]
    }, {
      "type": "MoverBehaviour", "enabled": true,
      "movementSpeed": 2.0
    }, {
      "type": "DestroyAfterTimeBehaviour", "enabled": true,
      "delaySeconds": 4.0
    }, {
      "type": "DestroyOnCollisionBehaviour", "enabled": true
    }, {
      "type": "CameraComponent", "enabled": true, "primary": true,
      "fieldOfView": 45.0, "nearPlane": 0.1, "farPlane": 100.0
    }, {
      "type": "ColliderComponent", "enabled": true, "shape": "Box",
      "center": [0.0, 0.0, 0.0], "size": [1.0, 1.0, 1.0],
      "trigger": true, "layer": 0, "mask": 65535
    }, {
      "type": "RigidBodyComponent", "enabled": true,
      "motionType": "Kinematic", "mass": 1.0, "gravityFactor": 1.0,
      "friction": 0.5, "restitution": 0.0
    }, {
      "type": "KinematicMotionBehaviour", "enabled": true,
      "linearVelocity": [0.5, 0.0, 0.0],
      "angularVelocityDegrees": [0.0, 30.0, 0.0]
    }, {
      "type": "CollectibleBehaviour", "enabled": true
    }, {
      "type": "CollectionGameComponent", "enabled": true
    }, {
      "type": "PotPlayerBehaviour", "enabled": true,
      "movementSpeed": 4.0, "turnSpeedDegrees": 120.0,
      "minimumBounds": [-8.0, -8.0], "maximumBounds": [8.0, 8.0]
    }, {
      "type": "CollectionGamePlayerComponent", "enabled": true
    }, {
      "type": "ScoredTimedItemBehaviour", "enabled": true,
      "scoreValue": 100, "lifetimeSeconds": 60.0
    }]
  }]
}
```

IDs are stable references only inside one file. Load creates fresh nonzero runtime IDs, maps every
serialized ID, then restores parent and selection references. `parentId` zero denotes a root.
Transforms are local TRS values; parents are attached after every object exists with
preserve-world disabled.

`MeshRenderer` always contains `enabled`, RGB `tint`, `wireframe`, `sourceType`, and current-version
`materialPath`. A procedural
record additionally contains `primitiveType` and every validated `primitiveParameters` field; an
imported record instead contains one nonempty normalized project-relative `modelPath` ending in
`.obj` or supported static/one-skin animated `.gltf`. An empty `materialPath` retains the Model
section Material; a nonempty path overrides every section in static and skinned color passes while
remaining irrelevant to bounds, picking, and shadow depth. See `docs/MATERIAL_FORMAT.md`.
`LightComponent` contains `enabled`, RGB `color`, `intensity`,
`ambientStrength`, and `lightingEnabled`. Generated Models, Materials, Mesh buffers, textures,
pointers, OpenGL handles, shaders, and ImGui state are never serialized; Application reconstructs
them through existing caches.

Scene 28 continues storing Mesh Renderer `tint` and Light `color` as the exact display-authored RGB
values shown by existing editor controls. v0.9.0 converts those values to linear RGB only at the
world-render boundary; it does not rewrite Scene bytes or add exposure, tone-map, or color-space
fields. Imported material factors retain their existing linear internal interpretation, while
Canvas colors/images remain display-referred after the HDR resolve.

Version 2 adds `RotatorBehaviour`. Its component object requires `enabled` and finite
`angularVelocity` XYZ values expressed in local degrees per second. Duplicate Rotator components,
missing or malformed fields, non-finite values, and Rotator content marked as version 1 are rejected
transactionally.

Version 3 adds `MoverBehaviour`. Its component object requires `enabled` and a finite,
non-negative `movementSpeed` in local units per second. Duplicate Mover components, missing or
malformed fields, invalid speeds, and Mover content marked as version 1 or 2 are rejected
transactionally. Runtime Start completion, input transitions, fixed accumulator/remainder, tick
count, callback context/phase diagnostics, and Play-time Transform results are never serialized.

Version 4 adds `DestroyAfterTimeBehaviour`. Its component object requires `enabled` and a finite,
non-negative `delaySeconds`. Duplicate components, missing or malformed fields, invalid delays, and
DestroyAfterTime content marked as versions 1–3 are rejected transactionally. Fixed-clock elapsed
time, pending IDs, queued commands, commit results, and lifecycle state are runtime-only and never
saved.

Loading version 1 migrates in memory with no Rotator, Mover, or DestroyAfterTime components. Loading
version 2 preserves Rotator and migrates without Mover or DestroyAfterTime. Version 3 preserves
Rotator and Mover and migrates without DestroyAfterTime. Version 4 preserves DestroyAfterTime and
migrates without CameraComponent. Version 5 preserves CameraComponent and migrates without trigger
or collectible components. Version 6 preserves those components and migrates without the v0.3.0
game marker, pot controller, or scored/timed item types. Version 7 preserves the Pot Pursuit types
and migrates its legacy trigger records to Box Colliders.
Version 8 preserves current Box Colliders and migrates without Rigid Bodies. Version 9 preserves
Static/Dynamic Rigid Bodies and migrates without Kinematic content. Version 10 preserves Kinematic
content and migrates without generic Sphere/Capsule geometry. Version 11 preserves generic
Collider geometry and migrates without character content. Version 12 preserves character content
and migrates without DestroyOnCollisionBehaviour. Version 13 preserves deterministic collision
events and migrates with no Canvas state. Version 14 preserves authored Canvas layout and visuals
and migrates with no Button state. Version 15 preserves Button state and migrates without an Input
Preferences launcher. Version 16 preserves the launcher and migrates without an Audio Source.
Version 17 preserves its legacy Audio Sources as 2D and migrates without a Listener. No Load
rewrites its source file. Version 18 preserves spatial Audio Sources and Listeners and migrates
without an Audio Button Behaviour. Version 19 preserves that Behaviour and migrates legacy
`ImportedObj`/`importedObjPath` values. Version 20 preserves generic imported-model identities and
migrates without an Animator. Version 21 preserves Animator values and migrates without a Character
Animation Behaviour. Version 22 drivers migrate a cross-fade duration of `0.15`; version 23 drivers
migrate empty takeoff and landing clip names. Version 24 preserves those names; version 25 adds
generic Collection Game player identity; version 26 adds authored Health/damage/defeat records;
version 27 adds the authored Scene Transition Button record; version 28 adds explicit initial focus
and the field-free Quit marker; version 29 expands the fixed Light record for bounded Point/Spot
authoring; version 30 appends the shadow toggle; version 31 appends the Mesh Renderer Material
identity; version 32 appends the optional singleton Sky Environment; version 33 appends optional
authored Particle Burst data; version 34 appends the field-free Graphics Preferences launcher;
version 35 appends the optional path-only Script Behaviour; version 36 appends its required ordered
typed property list; version 37 adds the optional Scene-local GameObject property value;
version 38 adds Prefab Spawner; version 39 adds the three interaction Components; version 40 adds
Respawnable, Checkpoint, Objective Counter and Conditional Reward. Version 41 adds paths/follow Cameras and Motor basis fields. Version 42 adds vehicles, target orientation and the receiver-mask spelling. Every subsequent Save emits version 42.
All accepted versions retain the same file-local
ID remapping, hierarchy restoration, Camera, Transform, MeshRenderer, and LightComponent contracts.

Optional Component absence is ordinary current-version state. Removing a Component in the editor causes
the next Save to omit that Component object; Undo reinserts its detached configuration at the exact
original runtime position. Scene version 42 emits canonical type order and does not persist
arbitrary optional-Component insertion order; prefab format 36 separately records
exact authored order. Changing this Scene contract requires a future explicit version bump.
Transform remains the mandatory embedded `transform` object and is never part of optional Component
mutation. Editor command history and logical target remaps are session-only and never serialized.

v0.5.3 physics-query requests, filters, hits, overlap result IDs, query adapters, and native Jolt
collector state are callback/runtime values only. They add no authored field or Component, so the
query state is never saved or loaded. v0.5.4 advances the format only for generic Collider shapes;
v0.5.5 advances it for authored character configuration; v0.5.6 advances it only for the authored
DestroyOnCollisionBehaviour enabled state.

Version 14 adds the authored runtime Canvas family. A root-only `CanvasComponent` stores `enabled`,
finite `referenceWidth`/`referenceHeight` in `[1,8192]`, and signed `renderOrder` in
`[-32768,32767]`. `RectTransformComponent` stores enabled state, normalized ordered anchor pairs,
normalized pivot, bounded finite anchored position/size delta, and render order. Exactly one of
`UIPanelComponent`, `UITextComponent`, or `UIImageComponent` may share an object, and it requires a
preceding Rect Transform. Panels store RGBA color; Text stores one printable 1–128-byte ASCII line,
RGBA color, font size `[1,256]`, and `Left`/`Center`/`Right` alignment; Image stores a normalized
PNG/JPG/JPEG identity beneath `assets/images`, RGBA tint, and `Stretch`/`AspectFit`. Canvas is
mutually exclusive with Rect/UI on its root, a Scene has at most eight Canvases and 256 visuals,
and all failures reject the complete parse or save transaction. GPU textures, resolved rectangles,
draw commands, and diagnostics remain runtime-only.

Version 15 adds `UIButtonComponent` as an interaction decorator after exactly one
`UIPanelComponent`, `UITextComponent`, or `UIImageComponent` on the same object. The exact order is
Rect Transform, visual, then Button; Button conflicts with Canvas and cannot exist without both
dependencies. Its record contains boolean `enabled`, boolean `interactable`, a 1–64-byte ASCII
`actionName` beginning with a letter and continuing with letters, digits, `_`, `.`, or `-`, plus
RGBA `[0,1]` `hoverTint`, `pressedTint`, `focusedTint`, and `disabledTint` arrays. Duplicate,
missing, extra, reordered, malformed, nonfinite, out-of-range, or invalid-identifier content rejects
the complete candidate. Versions 1–14 migrate without an implicit Button.

Button focus, hover, pointer capture, resolved/clipped rectangles, draw rank, sampled input edges,
suppression, presentation state, and pending action are PlaySession-only and never serialized. The
visual remains the sole draw command; Button state only multiplies its color. Activation dispatches
the bounded action to eligible native Behaviours on the same runtime object after Start. Navigation
is focus-only. Mouse Left, Enter, Space, and Face South share a source-owned held press, and only
the initiating source's valid release activates. Presentation priority is disabled, pressed,
focused, hovered, then normal. Callback targets and runtime-command results are not persistent data.

Version 16 adds the field-free `UIInputPreferencesLauncherComponent` marker. Its record contains
exactly `type` and boolean `enabled`, must follow a same-object `UIButtonComponent`, and therefore
inherits the Button's Rect/visual dependencies. An enabled eligible launcher consumes the Button's
release activation to open the PlaySession-owned controls modal; absent, disabled, or ineligible
launchers preserve version-15 Behaviour activation. Profile keys, paths, preference bytes,
effective maps, generations, modal/capture state, and generated overlay commands are runtime-only.
Versions 1–15 migrate without an implicit launcher.

Version 17 adds `AudioSourceComponent` in canonical type order. Its record has exactly the seven
fields below. `clipPath` is a normalized forward-slash `.wav` identity beneath `assets/audio` and
is resolved transactionally to immutable decoded PCM. Runtime voices, cursors, device/backend
state, callbacks, warnings, and playback progress are never serialized.

```json
{
  "type": "AudioSourceComponent",
  "enabled": true,
  "clipPath": "assets/audio/kenney_ui-audio/WAV/click1.wav",
  "playOnStart": true,
  "loop": false,
  "volume": 1.0,
  "pitch": 1.0
}
```

Version 18 extends that record with exact persistent spellings `TwoDimensional` or `Spatial3D`,
finite `minimumDistance` in `[0.01,1000]`, and finite `maximumDistance` greater than minimum and at
most `10000`. Legacy version-17 sources receive `TwoDimensional`, `1.0`, and `25.0` in memory.
Version 18 also adds a field-free `AudioListenerComponent` record containing exactly `type` then
boolean `enabled`; a Scene may store at most eight. Runtime listener/source poses, normalized
basis, voice state, and attenuation results never persist.

```json
{
  "type": "AudioSourceComponent",
  "enabled": true,
  "clipPath": "assets/audio/fireplace/fire-mono.wav",
  "playOnStart": true,
  "loop": true,
  "volume": 0.22,
  "pitch": 1.0,
  "spatialMode": "Spatial3D",
  "minimumDistance": 1.0,
  "maximumDistance": 18.0
}
```

```json
{
  "type": "AudioListenerComponent",
  "enabled": true
}
```

Version 19 adds `AudioButtonBehaviour` after a valid same-object Rect Transform, UI visual, Button,
and Audio Source. Its record contains exactly the canonical keys `type`, boolean `enabled`, and
case-sensitive `action` (`Play`, `PlayOneShot`, or `Stop`). `PlayOneShot` requires the same object's
Audio Source to be non-looping. Missing dependencies, dependency reordering, duplicate records,
missing/extra/duplicate/reordered keys, malformed types, or unknown action spellings reject the
complete load transaction. Versions 1–18 migrate without an implicit Audio Button Behaviour.

```json
{
  "type": "AudioButtonBehaviour",
  "enabled": true,
  "action": "PlayOneShot"
}
```

The Behaviour always targets its own runtime object's Audio Source when its same-object Button
activates. It persists no action-name filter, cross-object ID, clip copy, voice, command sequence,
or toggle state.

Version 5 adds `CameraComponent`. It stores `enabled`, `primary`, and finite perspective values:
field of view from 20 through 90 degrees, a positive near plane, and a far plane greater than near.
Pose is never serialized in the component because its owner's Transform and hierarchy exclusively
supply world pose. Versions 1–4 migrate without CameraComponent and reject camera content marked as
an older version. Multiple primary flags may persist, but Play transactionally rejects ambiguity.

Version 6 adds `TriggerVolumeComponent` with `enabled`, finite local `center`, and finite strictly
positive local `size`, plus enabled-only `CollectibleBehaviour`. Duplicate components, missing or
malformed fields, non-finite centers/sizes, zero/negative sizes, and either component marked as
versions 1–5 reject transactionally. Trigger world bounds, overlap pairs, entry events, pending
destruction, and collection state are runtime-derived and never serialized.

Version 7 adds enabled-only `CollectionGameComponent`; `PotPlayerBehaviour` with finite
non-negative `movementSpeed`, finite non-negative `turnSpeedDegrees`, and finite strictly ordered
two-number XZ `minimumBounds`/`maximumBounds`; and `ScoredTimedItemBehaviour` with a signed integer
`scoreValue` in `[-1000000, 1000000]` plus finite strictly positive `lifetimeSeconds`. Duplicate,
missing, malformed, invalid, or version-1-through-6 occurrences reject transactionally. Score,
remaining count, registered runtime IDs, resolved flags, elapsed lifetimes, completion, and HUD
state are runtime-only and never serialized.

Version 8 replaces authored `TriggerVolumeComponent` records with `BoxColliderComponent`.
`center` must be finite, every `size` axis must be finite and strictly positive, `trigger` is a
boolean, `layer` is an unsigned integer from 0 through 15, and `mask` is an unsigned 16-bit value.
Versions 6 and 7 accept only the legacy trigger spelling and migrate it in memory to a trigger Box
Collider on layer 0 with mask 65535. Version 8 accepts only the current spelling. Mixed, duplicate,
malformed, or out-of-range collider records reject the complete load transaction. Jolt bodies,
shapes, overlap/event history, and derived world poses are runtime-only.

Version 9 adds `RigidBodyComponent`. It requires a preceding `BoxColliderComponent` on the same
GameObject and stores explicit `enabled`, `motionType` (`Static` or `Dynamic`), finite positive
`mass`, finite non-negative `gravityFactor`, and finite `friction`/`restitution` in `[0, 1]`.
Missing or reordered collider dependencies, duplicate bodies, unknown motion types, missing fields,
non-finite values, and out-of-range values reject the complete load transaction. Versions 1–8
migrate without a Rigid Body. Jolt BodyIDs, velocities, activation/sleep state, contacts,
accumulators, allocator state, physics time, and simulated Transforms are runtime-only.

Version 10 extends `RigidBodyComponent.motionType` with `Kinematic` and adds
`KinematicMotionBehaviour`. The Behaviour must follow a Box Collider and a Kinematic Rigid Body on
the same object. It stores `enabled`, a finite signed XYZ world `linearVelocity` in units per
second, and a finite signed XYZ local `angularVelocityDegrees` in degrees per second. Duplicate,
reordered, missing-dependency, non-Kinematic dependency, malformed/non-finite, or version-1-through-9
Kinematic content rejects the complete transaction. Versions 1–9 migrate without Kinematic Motion.
Jolt targets, realized runtime poses, native velocities, activation, contacts, sleep state, and
fixed-clock state are never serialized.

Version 11 replaces the live box-only spelling with one strict `ColliderComponent`. Common fields
are `enabled`, finite local `center`, `trigger`, `layer` 0..15, and 16-bit `mask`. `shape` is exactly
`Box`, `Sphere`, or `Capsule`; Box stores only positive finite XYZ `size`, Sphere stores only a
positive finite `radius`, and local-Y Capsule stores only positive finite `radius` plus total
tip-to-tip `height` strictly greater than twice the radius. Missing, extra, mixed, unknown, or
invalid current fields reject the complete transaction. Scene versions 8–10 continue accepting
only their historical `BoxColliderComponent` schema and migrate it exactly to Box. Scene versions
6–7 retain their historical trigger migration. Jolt shapes, resolved scale, world pose, contacts,
query results, and simulated values remain runtime-only.

Version 12 adds `CharacterControllerComponent` and `CharacterMotorBehaviour` after the Collider/
Rigid Body/Kinematic entries and before collectible/gameplay Behaviours in canonical Scene output.
The Controller requires a preceding enabled, nontrigger Capsule `ColliderComponent`, a root owner,
finite upright local rotation with X/Z zero, finite positive uniform local scale, no
`RigidBodyComponent`, and no enabled Mover, Rotator, Pot Player, or Kinematic Motion Transform
authority. It stores `enabled`; finite positive `mass`; finite non-negative
`maximumPushStrength` and `gravityFactor`; finite `maximumSlopeAngleDegrees` strictly between zero
and 90; and finite non-negative `maximumStepHeight` and `floorStickDistance`. The Motor requires a
preceding Controller and stores `enabled` plus finite non-negative `moveSpeed`,
`turnSpeedDegrees`, and `jumpSpeed`. Missing, extra, duplicate, reordered, malformed, non-finite,
out-of-range, or dependency-invalid records reject the complete load transaction. Versions 1–11
migrate without implicit Controller or Motor content.

The strict character portion of a valid root object's `components` array is:

```json
[
  {
    "type": "ColliderComponent", "enabled": true, "shape": "Capsule",
    "center": [0.0, 0.0, 0.0], "radius": 0.5, "height": 2.0,
    "trigger": false, "layer": 0, "mask": 65535
  },
  {
    "type": "CharacterControllerComponent", "enabled": true,
    "mass": 70.0, "maximumPushStrength": 100.0, "gravityFactor": 1.0,
    "maximumSlopeAngleDegrees": 50.0, "maximumStepHeight": 0.4,
    "floorStickDistance": 0.5
  },
  {
    "type": "CharacterMotorBehaviour", "enabled": true,
    "moveSpeed": 4.0, "turnSpeedDegrees": 120.0, "jumpSpeed": 5.0
  }
]
```

CharacterVirtual instances, proxy bodies, ground/contact state, vertical or supporting-body
velocity, fixed motion requests, Jump edges, contact hits, native capacity, and simulated
Transforms are PlaySession-only and never serialized. Character runtime state cannot be restored
by changing a file version or adding undeclared fields.

Version 13 adds `DestroyOnCollisionBehaviour` in canonical Scene order immediately after
`DestroyAfterTimeBehaviour`. Its component object contains exactly `type` and boolean `enabled`;
missing, extra, malformed, duplicate, or version-1-through-12 occurrences reject the complete load
transaction. Versions 1–12 migrate without an implicit instance. Collision listener facts,
normalized current/previous pair sets, Enter/Stay/Exit values, callback contexts, pending
destruction, Jolt identities, sleep/activation state, and committed query snapshots are
PlaySession-only and never serialized.

### Historical v0.12.1 migration snapshot

This snapshot does not freeze later sample development. At v0.12.1 the projects owned separate `assets/scenes/main_menu.json` and
`assets/scenes/gameplay.json` files. Each Scene-34, 15-object Main Menu has one primary Camera,
one Canvas, Play as its sole initial-focus Button, Controls, Graphics, and Quit. Play targets
`assets/scenes/gameplay.json` within that project; presentation names differ for Ultimate and Pot.
The only menu image is its project-owned `assets/images/logo.png`.

Ultimate gameplay is the exact owner-edited Scene-37, 29-object Coin Run (48,249 bytes, SHA-256
`F2F8E29BF7CCB081553133F554302E841628427F0016C3E5922229BEFBB73083`). It contains seven
scored/timed trigger Coins, character health/animation, three spike hazards, local lighting and
sky. The world-synchronized moving Coin remains a root Kinematic object to avoid parent shear.
Object IDs and authored values come from this preserved Scene, not the retired synchronized demo.
The coin Material preserves SHA-256
`FE4C8BF8A954850A05F6EC4728CBA2DA74D02FCE1F3BA7EAE87434B27C3C3F1D`.
Pot gameplay preserves the exact Scene-12, 35-object source (SHA-256
`9ACDDE6FE261BC027804D375A20ED0CB7FDEEF0FE3442966C2B8AB167E4655EC`).
The root game-role Scenes and synchronization script are retired. Technical
`editor_showcase.json` and the other root showcases were subsequently retired in v0.12.7.

Initial Edit Scene ownership is project-local: first valid Recent Scene, otherwise saved Build
Settings startup, otherwise an unsaved starter with a diagnostic. Recovery decisions follow base
selection. This changes neither Scene 37 nor historical migration; Basic3D's `default_scene.json`
is just a project-local filename, not an engine default.
### Historical showcase demonstrations — retired in v0.12.7

The released v0.10.9 milestone kept `scripting_showcase.json` at Scene 37, expanded
it to exactly eleven objects/eight roots, and leaves it excluded from default Build Settings. IDs
1–8 retain the released v0.10.8 hierarchy, component order, and authored values. ID 1 `Showcase
Ground` remains a tinted Plane at scale `[12,1,10]` and now has canonical component order Mesh
Renderer -> Script Behaviour -> Collider -> Rigid Body. Its script references
`assets/scripts/destroy_collision_drop.lua` with exact GameObject property `drop = 7`. Its enabled
nontrigger Box Collider retains center `[0,-0.05,0]`, local size `[1,0.1,1]`, layer `0`, and mask
`65535`; its new enabled Static Rigid Body uses mass `1`, gravity factor `1`, friction `0.5`, and
restitution `0`. ID 2 `Moving Target` remains a tinted Cube at `[-2,0.5,0]` with canonical
component order Mesh Renderer -> Audio Source -> Collider. Its Audio Source is enabled, references
`assets/audio/kenney_ui-audio/WAV/click1.wav`, has `playOnStart = false`, Loop false, volume `0.8`,
pitch `1.0`, `TwoDimensional` mode, and distances `1.0`/`25.0`. Its retained enabled nontrigger
query-only Box Collider has center `[0,0,0]`, local size `[1,1,1]`, layer `0`, and mask `65535`.
The target has no Rigid Body.

ID 3 `Target Marker` remains the target's child UV Sphere at local `[0.75,1,0]` and uniform scale
`0.35`. ID 4 `Lua Follower` remains a tinted Cylinder at `[-2,1.5,2]`, uniform scale `0.75`, and now
has canonical component order Mesh Renderer -> Script Behaviour -> Audio Source -> Collider. Its
four exact Script Behaviour properties remain GameObject `target = 2`, Vector3
`follow_offset = [0,0,2]`,
Number `movement_speed = 2`, then Number `turn_speed_degrees = 90`. Its enabled Audio Source
references `assets/audio/fireplace/fire.wav`, has `playOnStart = false`, Loop true, volume `0.25`,
pitch `1.0`, `TwoDimensional` mode, and distances `1.0`/`25.0`. Its new enabled Trigger Box is
centered at `[0,-0.5,0]`, has local size `[7,1.5,7]`, layer `0`, and mask `65535`. ID 5 remains the
primary 55-degree
Camera at `[0,5,11]`, rotation `[-20,0,0]`, near `0.1`, far `100`; ID 6 remains the Directional
Light at rotation `[-50,-35,0]`, intensity `1.25`, ambient `0.25`, with shadows enabled.

ID 7 `Collision Drop` is one root-level tinted purple procedural Cube at `[3,4,0]`, unit rotation
and scale. Its canonical order is Mesh Renderer -> Collider -> Rigid Body. The enabled nontrigger
Box Collider uses center `[0,0,0]`, local size `[1,1,1]`, layer `0`, and mask `65535`; the enabled
Dynamic Rigid Body uses mass `1`, gravity factor `1`, friction `0.5`, and restitution `0`.

ID 8 `Lua Animated Character` is a root at local position
`[3,-0.0310804843902588,-2]`, zero rotation, and uniform scale `[0.72,0.72,0.72]`. Its canonical
component order is Mesh Renderer -> Animator -> Script Behaviour. The enabled imported Mesh
Renderer references
`assets/models/Ultimate Platformer Pack - Dec 2021/Character/glTF/Character.gltf`, uses white tint,
and disables wireframe. The enabled Animator authors exact clip `Idle`, `playOnStart = false`,
`loop = true`, and playback speed `1`. The enabled Script Behaviour references
`assets/scripts/animate_showcase.lua` and has no properties.

ID 9 `Lua UI Canvas` is a new root with unit Transform and one enabled Canvas Component using
reference resolution 1280x720 and render order 0. ID 10 `Lua Nudge Button` is its child with unit
Transform and canonical order Script Behaviour -> Rect Transform -> Panel -> Button. The enabled
Script Behaviour references `assets/scripts/ui_button_showcase.lua` and owns exactly one GameObject
property `target = 8`. The Rect Transform anchors and pivots at bottom-right `[1,0]`, uses anchored
position `[-28,28]`, size `[240,64]`, and render order 10. Its enabled blue Panel and enabled,
interactable Button author action `LuaNudge`, distinct hover/pressed/focused/disabled tints, and
`initiallyFocused = false`. ID 11 `Lua Nudge Label` is ID 10's child with centered Rect Transform,
size `[216,34]`, render order 11, and enabled centered white Text `LUA NUDGE` at font size 20.

API-version-8 `assets/scripts/follow_target.lua` retains its authored target, marker, ground, owner,
configuration, and fresh-session audio flags during Start. First Update queues the quiet follower
ambient loop once; first meaningful WASD movement stops it once before retained normalized target
movement; arrows retain target Y rotation. Trigger Enter/Stay compares only the authored target
handle and opens/maintains a bounded gate; matching Trigger Exit closes it. The gate controls only
LateUpdate follower tracking, while the exact ground raycast, oriented target overlap, follower-to-
target box sweep, movement, rotation, audio, and Jump behavior remain functional. Jump stops
ambient if needed, queues one target click one-shot, then requests existing target-subtree
destruction and performs no later target access. API-version-8
`assets/scripts/destroy_collision_drop.lua` retains the authored drop handle at Start and destroys
only that matching counterpart on its first Collision Enter, accepting the existing
`accepted`/`already_pending` results without polling or repeated work.

API-version-9 `assets/scripts/animate_showcase.lua` retains its owner and private intended-state
flags. Start performs setup only. First Update requests looping `Idle`; later meaningful Move
transitions request one 0.15-second cross-fade to looping `Run` or back to `Idle`; Jump/Space
requests Stop once while held; and release cross-fades once to the current movement state. The
script submits at most one animation command for ID 8 in a frame, never moves the character, and
never reads or mutates Animator/Model/playback state.

API-version-10 `assets/scripts/ui_button_showcase.lua` retains authored GameObject property target
ID 8 and its exact authored local position during Start. Exact action `LuaNudge` alternates the
private toggle and submits the released target-position command with X 4.5 or the authored X 3.0;
every other action returns without mutation. The script logs only accepted activation and owns no
UI state, input polling, queue, global subscription, or Scene transition. Stop/new Play restores
the authored position and fresh private toggle.

Exact released v0.10.9 content:

| Asset | Bytes | SHA-256 |
|---|---:|---|
| `assets/scenes/scripting_showcase.json` | 16,937 | `E30536CA2B39A634B86F452F285EECB18532DB62CDBEF9675553B8EAA6350E3F` |
| `assets/scripts/follow_target.lua` | 7,829 | `292A93029AB68B698BB9ACB98DE39436B520A0ACA8A3D35E40A198CC010FE30B` |
| `assets/scripts/destroy_collision_drop.lua` | 671 | `D2E4B39EA880B13A4647234F34086AE5C8034E5999E75976A251B53908C05402` |
| `assets/scripts/animate_showcase.lua` | 2,155 | `652D792440B8CE361F1FACA305E7B1CB7BB251C8CB8B3D92626512F4EC0010C0` |
| `assets/scripts/ui_button_showcase.lua` | 871 | `A0BDDB4D99A018EA091BB61D1E493F4882D5ECDC68EE620A3D12EA2DF92AB136` |
| `assets/models/Ultimate Platformer Pack - Dec 2021/Character/glTF/Character.gltf` | 1,035,242 | `5913C61C972D752CDCE20DB1281E2FE48FDB55A5BAF8B298FC432C8C30B11232` |

This release uses only existing Scene-37 Components and does not change the Scene format. The
ordinary package target stays 27 manifest entries/28 physical files and excludes the isolated
Scene, its four scripts, and two WAVs while retaining its already-shipped Character model. The
isolated package target is 20 manifest entries/21 physical files through the exact four scripts,
two WAVs, self-contained model, seven dependency notices, and required Quaternius CC0 model-root
notice, with no external glTF resource. Focused Scripting proves ordinary closure/exclusions and
tampered rejection plus exact isolated closure, dependency rollback, relocation, and automated
healthy packaged-Player liveness; all of those gates pass at
**246/246**, including Player exit code 0. Independent real Release editor, native Build Game,
direct/relocated Player, synthetic interaction, package, and direct visual/state gates pass. At
released tag v0.10.8, the
same Scene had eight objects/seven roots and was 13,101 bytes at SHA-256
`BA630B2107BFBEB8901B790810594B9B76657697409BDE3525A2EB2F1F0DED97`. At released tag v0.10.7,
the same Scene had seven objects,
was 11,934 bytes at SHA-256
`0820C516DF1C8245D4D92532E7063B1F18C5E819CAAE446C2CFD22AD2CD4A5B0`. At released tag
v0.10.6, the same Scene had six objects, was 9,194
bytes at SHA-256
`DDAFD01A21653ABE5911A6A45F8BE9AD97622D395852DEFB2DED73C60DEAB094`, and its API-version-7 script
was 6,617 bytes at SHA-256
`09E99EC97CD4482CDE0301F54102F19B70828B9ED6143D2738266C20D82846CB`. At released tag v0.10.5,
the same Scene was
8,454 bytes at SHA-256 `EDCAF3BCAFEC61A96A13AA4B95ADC4963F560484A6BA6C5E511472F9F85DCF8E`
and its API-version-6 script was 5,897 bytes at SHA-256
`BD917136C2329A255739CBC160B55494E5BB2A4ACDFC0F9C4D33662CF2FEDE12`.
`physics_showcase.json` is a separate procedural nineteen-object sample with a primary Camera,
directional light, Static ground/ramp, seven Dynamic cubes, visible Dynamic Sphere and composite
Capsule drops, a translating Kinematic platform, and a
rotating Kinematic obstacle demonstrating stack, mass, restitution, friction, carrying, pushing,
and deflection without imported assets. Its isolated `Collision Event Disappearing Drop` is a
Dynamic procedural Cube with DestroyOnCollisionBehaviour; it queues itself on its first accepted
response collision and is restored when Play stops.
`character_controller_showcase.json` is a separate procedural eighteen-object sample with one
Capsule character and child primary Camera, walkable/too-steep ramps, low/high steps, a downward
floor transition, moving Kinematic platform, Dynamic push boxes, authored trigger collectible,
and course boundaries.
### Current imported-resource restoration

Imported paths may reference the approved Quaternius
package, including nested directories containing spaces, but Application accepts them only after
the shared model-path policy proves an existing canonical `.obj`, `.gltf` or `.fbx` below `assets/models`
and the common importer validates the format's supported content.
Serialization continues to store only the normalized project-relative path; catalog search and
cache metadata are editor/runtime-derived and never serialized.
The former proprietary Moe chair and its root directory were retired in v0.12.7. Assets outside explicitly audited
package/asset roots remain ignored and local-only. If any imported model or required resource
needed during restoration is unavailable or invalid, Application
rejects the complete replacement and preserves the active Scene, selection, Camera, current path,
command history, and dirty state.

The loader accepts versions 1 through 46; every accepted version rejects unknown components,
missing required fields, missing parents, duplicate/zero IDs, invalid scales, cycles, and
unavailable imported models. Versions above 45 reject as future.
Instantiated prefabs require no Scene schema: they save as ordinary GameObjects and Components with
no source link or override metadata. Prefab asset files use the separate contract in
`docs/PREFAB_FORMAT.md`. Script-declared schemas/general reflection, modules, hot reload, general
Scene enumeration, raw-ID exposure, global/cross-Scene object references, and broader other-object
script mutation beyond API-v5 local position/rotation and deferred subtree destruction remain
unsupported. API-v6 physics access is limited to callback-lifetime read-only closest raycast,
oriented Box overlap, and closest Box sweep over copied values and opaque handles. Additional
collider shapes beyond Box/Sphere/Capsule, constraints beyond the bounded Hinge/DistanceSpring
and terminal ragdoll contract, general animation
graphs beyond the bounded inline controller, blend trees/layers, gameplay-authoritative animation
events, root-motion application, editor layout,
unknown-component preservation, runtime lifecycle state, input state, and undo history also remain
unsupported.
