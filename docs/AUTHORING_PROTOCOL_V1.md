# Bounded editor authoring protocol 1

Status: **RELEASED through v1.20.0**, with independent/owner/installed/public acceptance complete.
The v1.13.0 plan (see installed documentation/source reference)
records introduction; the v1.20 final report (see installed documentation/source reference) owns current
release evidence. v1.12 did not expose it.
This document owns the wire/descriptor contract; the plan must not maintain a competing copy.

Released revision 8 retains protocol/schema 1 and 50 complete descriptors. Its registry
exports 67 commands. `terrain.preview` includes palette edits, weighted paint disks/corridor
stamps, named corridor/exclusion upserts and removal. The new read-only `placement.inspect`
and guarded `placement.preview` / `placement.commit` use the same typed operation as the Terrain
panel. See the installed client recipe (see installed documentation/source reference)
for saved-terrain requirements, operand units, preview limits and the repair sequence. The
generated wire schema (see installed documentation/source reference) is the field authority.

The released v1.14 plan (see installed documentation/source reference)
adds `prefab.capture`, PrefabSource/FlightImage resource slots and24 complete records below.
That release used capabilityRevision2 and40 commands; v1.15 supersedes those counts below.
Early external/native/browser and independent installed E02/owner/public acceptance are complete.
Protocol/schema1 and launch/security semantics do not change.

The v1.15 release has38 complete descriptors,46 commands and capabilityRevision3. It adds typed
Material publication and existing navigation-bake exposure without changing Protocol1. Registry-derived
exports are authoritative for exact fields/defaults; the following additions supersede old counts.

The released v1.16 plan (see installed documentation/source reference) implements revision4:
43 complete descriptors,47 commands and copied bounds/motion inspection. The final report above
records completed acceptance. These counts supersede the historical v1.15 counts above.
The v1.17 release (see installed documentation/source reference) implements revision5
with46 complete descriptors/53 commands, typed input and copied rig/pose/vehicle inspection.
Those accepted exports are extended by the v1.18 release below.

The released v1.18 plan (see installed documentation/source reference) specifies
revision6 with49 complete descriptors/54 commands: Route/VehicleRouteDriver/VehicleBoost and copied
`play.progress.inspect`, plus additive motion status.
Protocol/schema1 and launch/security bounds remain unchanged; generated exports come from the registry.

**Released in v1.19.0:** capability revision 7 exports 50 complete descriptors and
64 commands. Discover the deployed revision before using `TerrainComponent` or the ten
`terrain.*` commands. The terrain recipe (see installed documentation/source reference)
shows the typed operation, revision, expected digest and short-lived preview token. No row arrays,
filesystem helper, Lua API, preview body or new transport are exposed. Publication of a terrain
file, the Scene reference and the navigation bake are separate operations with separate receipts.

**Released in v1.21.0:** the v1.21 execution plan, §6 (see installed documentation/source reference)
adds two complete component descriptors and read-only `play.combat.inspect` at capability
revision 9 (52 complete descriptors, 68 commands). Released clients still discover revision 8
(50/67); no change to the wire protocol or editor-only authority.

The revision-9 implementation accepts `play.combat.inspect {targets:[EditReference,...]}`
with a null guard in Playing/Paused, for 1–8 unique original Edit targets. It copies
runtimeGeneration, committedFrames, Scene identity and per-target resolved/view/firearm groups.
View data separates basePitch/baseYaw from kickPitch/kickYaw and names ownership/reason.
Firearm data includes controller/mode, ammo/capacity, cooldown/reloadRemaining, accepted revision,
tick, reason and local setback/tilt. A lastShot distinguishes eyeHit from actualHit, target,
incoming direction and acceptedDamage; absent hits/normals are null, not fabricated geometry.
Its references map back to Edit identities. Stop/reopen invalidates the mapping; reading never
advances simulation or grants capture. The installed release remains revision 8 until acceptance.

### v1.18 copied progress and rival authoring

RouteComponent, VehicleRouteDriverBehaviour and VehicleBoostComponent have complete typed fields,
defaults, ranges, dependencies, history and reference metadata. See Scene52 for native composition
constraints. No generic JSON writer or new pose authority is introduced.

`play.motion.inspect` adds `vehicleRoute` and `vehicleBoost`. Both have explicit available/reason.
Route driver copies state, mapped Route reference, distance/length/lateralOffset, desiredSpeed,
committed throttle/steering/brake, obstacleAvailable/obstacleDistance, stuckDuration/resetRevision.
Boost copies state, remaining, effectiveAcceleration/effectiveForwardSpeed and revision.

`play.progress.inspect {targets:[EditReference,...]}` is read-only/null-guard, Playing/Paused only,
1–8 distinct original Edit references. It returns decimal runtimeGeneration/committedFrames,
runtimeScene, targets and currentLevel/lastLevel. Targets have sourceEditRef/resolved/reason and
explicitly available/unavailable Health (`current/maximum/depleted/revision`) and Objective
(`completed/total/complete/unresolved_missing/revision`) facts. Available level records contain
`scene_path/score/awards`; unavailable records give a reason. No private Lua state, full run ledger,
save storage or runtime mutation handle is returned. Stop or runtime Scene replacement invalidates
the original mapping; repeated Paused queries do not advance simulation. Existing1MiB response cap
and main-thread dispatch apply. The shipped client recognizes this read command without a state guard.

### v1.17 input and copied presentation contract

Complete additive descriptors are ArcadeVehicleControllerComponent, VehicleInputBehaviour and
AnimationConstraintComponent. The registry export owns every scalar/default/range, ordered four
wheel references, and tagged bounded LookAt/TwoBoneIK list. Ordinary typed operations preserve the
existing history, clone/prefab reference repair, resource and preflight authorities. No new runtime
mutation, generic JSON writer or importer authority is exposed.

- `input.inspect {}`: fixed project identity, available/dirty/conflict/status/diagnostic,
  sourceSha256, opaque documentRevision and typed actions/bindings.
- `input.edit {expectedDocumentRevision,operation}`: exactly one of AddAction, RenameAction,
  DeleteAction, MoveAction, ReplaceActionType, AddBinding, DeleteBinding, MoveBinding,
  ReplaceBinding or ReplaceControl. The generated closed request schema owns argument records.
  Indices are zero-based snapshot positions, moveDirection is -1/+1, and typed Digital controls use
  `{device,control}` on the wire (the native serializer owns disk format3).
- `input.save {expectedDocumentRevision}`: common-document source-conflict-checked atomic save.
- `input.reload {expectedDocumentRevision,discardChanges}`: false refuses dirty work; true is an
  explicit discard of the current agent-owned draft. A pre-existing human/recovery draft still
  surfaces DecisionRequired and the native decision UI before any adoption/save/discard.

The three input mutations require the current state guard and ordinary Edit/modal/publication
admission. GUI/recovery/save/reload changes invalidate document revisions and stale indices.
Consecutive agent-owned edits may share a draft; an intervening GUI edit loses that ownership.
Save is not Scene history; EOF preserves input recovery, never implicitly saving/discarding.
Limits remain64 actions,8 bindings/action,64-byte names,256KiB source and128 sampled controls.

`rig.inspect {target,collection:"clips"|"joints",limit?,cursor?}` is read-only/null-guard. It returns
available/reason, rigSignature, decimal resourceRevision, basis, items/total/nextCursor. Clips carry
exact name/durationSeconds; joints carry exact qualified path, nullable parentPath and
bindLocalTransform.matrixColumns. Pages default32/max64; cursor binds object/Scene/resource revision
and rejects reimport/replacement. This copies loaded bind data without incidental import or reload.

`play.pose.inspect {targets:[{target:EditReference,jointPaths?:[qualifiedPath,...]},...]}` requires
Playing/Paused,1–8 unique original Edit targets,0–8 unique exact paths per target and null guard.
It returns decimal runtimeGeneration/committedFrames, runtimeScene and copied target records:
sourceEditRef, resolved/available/reason, presentationRevision, nullable animation
(state/clip/playing/progress/completed/crossfading), constraints
(key/kind/state/reason/effectiveWeight/residual/poseRevision) and requested joints
(path/available/reason/nullable worldTransform.matrixColumns). Disabled, missing and retired values
report unavailability, not plausible zero transforms. Paused reads retain the committed snapshot;
Stop/restart/runtime-Scene replacement follows motion-inspection lifetime rules below. No read steps
animation, physics or Lua. Matrices are finite column-major values and total response remains1MiB.

`play.motion.inspect` additionally returns nullable `vehicle`: availability/reason, controlEnabled,
speed/forwardSpeed, groundedWheels, steeringDegrees, throttle/brake and four wheelPivots mapped back
to Edit references/None. These are committed physics facts, not queued input or a gameplay-control API.

### v1.16 copied inspection contract

`object.inspect` appends `worldTransform` with column-major matrixColumns, decomposable and nullable
position/rotationDegrees/scale, plus `bounds.modelLocal`, `bounds.modelWorld`, `bounds.colliderWorld`.
Each AABB reports availability/reason; model bounds are own static/bind geometry, not subtree or
animated deformation. Collider world bounds conservatively enclose the transformed own collider,
not exact contacts. Reads do not upload GPU resources or alter history.

`play.motion.inspect {targets:[EditReference,...]}` is read-only/null-guard, requires1–8 unique
current-epoch references and Playing/Paused. It returns decimal-string runtimeGeneration and
committedFrames, runtimeScene and copied targets. Each has sourceEditRef, resolved/reason, nullable
worldPose (matrixColumns/position), path, camera, recovery and rolling. Path exposes achieved
distance/length/progress/direction/speed/state and endpoint/reset revisions; camera exposes mode,
heading, route projection and mapped Edit references; recovery exposes state/checkpoint/revisions/
committed position/yaw; rolling exposes availability, mapped source/visual, radius, accumulated
world quaternion XYZW and reset revision. Unavailable existing records include a bounded reason;
absent component groups are null and retired objects unresolved. No native/runtime handle escapes.
Snapshots use the original Edit-to-runtime map only: every Start gets a fresh generation, Stop
rejects, and runtime Scene replacement invalidates this mapping even when reopening the same path.
The shared response ceiling remains1MiB. Snapshot data is never a runtime mutation capability.

## 1. Authority and launch

Use the explicit launch form:

```text
MoeMagicEngine.exe --project <existing-project-root> --authoring-stdio
                   [--authoring-output-root <existing-output-parent>]
```

The existing ordinary launch/launcher behavior is unchanged. Reject duplicate/unknown switches,
missing pipe handles, invalid roots and combinations with launcher/validation modes before
project mutation. Authoring requires inherited redirected stdin/stdout, never console allocation,
socket, port, discovery service or automatic attachment to another editor. Bootstrap roots may be
absolute; subsequent authoring uses project-relative identities, returned references and opaque
tokens. Without an output grant, build requests return `OutputGrantRequired`.

One process owns one immutable ProjectContext. No protocol project switch/create operation in
version 1: ordinary New Project can prepare a disposable template copy first, and a new session
can open it. The launcher is not an agent-controlled arbitrary directory creator. Existing project
and output safety checks apply; an output grant cannot authorize writes to an installation, asset
root, user save directory or otherwise forbidden destination. No toolchain is required by this
installed-product authoring workflow.

Application owns Scene, resources, history, document continuations, Play and publishing. The
adapter owns transport, request validation, copied responses and tokens only. A bounded pipe
reader/writer may handle bytes off-thread; only the main thread resolves or mutates engine state.
Reuse EditorObjectOperations, EditorAssetIntakeOperations, EditorSceneDocumentOperations and
GameBuildOperation. Extract a narrow shared host coordinator where a UI handler currently owns
preflight; do not duplicate it or bypass dirty Material/Input/Build/Save Settings documents.

The adapter and its dependencies compile only into Editor, not MoeMagicRuntime/Windows Player/
Web Player. Runtime data produced through it uses the same published formats and shared runtime.
No provider, prompt execution, arbitrary C++/host eval, shell, network or unrestricted file API.

## 2. Framing, limits and lifetime

- UTF-8 JSON Lines; exactly one object per line, LF output, accept LF/CRLF input. Embedded
  newlines are JSON-escaped. No BOM, duplicate keys, arrays/batches, comments, NaN or Infinity.
- Maximum request: **262,144 UTF-8 bytes**, excluding newline; maximum JSON depth **16**.
  Overlong lines close the stream with a bounded error when possible, rather than buffering
  indefinitely. Malformed complete lines return `InvalidRequest` with null id and do not mutate.
- Maximum response: **1,048,576 bytes**. Lists default to 32 entries, maximum 64; opaque cursors
  bind to the queried snapshot/catalog generation. Stale cursors reject. Never silently truncate
  a single authored value; return `ResultTooLarge` or the documented script chunk interface.
- At most **eight queued requests**, one active mutation, one active publication. Backpressure
  stops pipe reads at the queue bound. One normal command is dispatched per editor frame; no
  unbounded loop starving the UI. Existing synchronous authoring/build work retains its cost.
- At most eight pending response frames; a client that stops reading suspends dispatch rather
  than blocking the UI thread or growing memory. Byte workers never wait while holding a Scene
  or editor-state lock. Broken pipes use the disconnect behavior below.
- Stdout is protocol-only; existing logs must route elsewhere. Stderr diagnostics are bounded,
  drained by the client, and are not the command result. No unsolicited stdout event stream.
- Client request ids are increasing canonical decimal strings in [1, 2^53-1]. Repeated/lower ids
  return `DuplicateRequest` and never execute again. No exactly-once promise across process loss:
  after an unknown outcome, inspect content/receipts; do not blindly replay mutations.
- Retain only eight completed build receipts and a 256-entry diagnostics ring (each entry at
  most 1024 bytes, with a truncation flag). Runtime diagnostics use a monotonic cursor and
  report dropped entries. No unbounded trace, screenshot capture or source-content log.
- EOF/disconnect disables dispatch and discards undispatched requests. Finish an already
  committed/synchronous operation; do not kill publication mid-transaction. Stop session-owned
  Play and leave unsaved Edit content available for normal recovery/UI; never auto-save,
  discard, reset progress or terminate a dirty editor. A clean explicit close uses ordinary
  shutdown. Broken output cancels pending transport, not already published data.
- A human UI mutation is allowed but invalidates the prior state token. Native dialogs, modal
  recovery/dirty continuations, continuous edits and exit retain their existing authority;
  commands report their blocked state instead of injecting clicks or dismissing them.

## 3. Envelope and identity

The first request is `session.open` with `session: null`, `guard: null`, `args: {}`. It negotiates
only exact version 1 and returns the selected session/project identity, engine/version authorities,
current Scene state, output grant availability, capability revision and limits. Example shapes:

```json
{"protocol":"MoeMagicAuthoring","version":1,"id":"1","session":null,"command":"session.open","guard":null,"args":{}}
```

```json
{"protocol":"MoeMagicAuthoring","version":1,"id":"1","session":"opaque-session","status":"ok","code":"None","message":"","state":{"token":"opaque-state","sceneEpoch":"1","scene":"assets/scenes/main_menu.json","dirty":false,"play":"Edit"},"effects":{"authoredChanged":false,"historyRecorded":false,"filePublished":false,"sceneReplaced":false},"result":{"engineVersion":"1.13.0","projectName":"Ultimate Platformer","capabilityRevision":"1","outputGranted":true},"warnings":[]}
```

These examples abbreviate result contents, not the envelope. All seven request fields shown are
required, with no unknown fields; each command has a closed args schema. Response always includes
the fields shown; result may be null, warnings is an array of at most four bounded messages.
Before a valid session opens, session/state may be null; malformed input has null id. An open
receipt also identifies the selected canonical project root as read-only bootstrap information,
never as a replacement asset identity. Include every released format authority in its version map.
Status is `ok`, `no_change`, `accepted`, `decision_required`, `rejected`, `failed` or `cancelled`.
`accepted` means a job was admitted, not that a build succeeded. Errors may still report a primary
publication with a secondary warning/failure: preserve underlying typed-operation effects and
diagnostics. Never turn a failed archive into successful ZIP output or hide a recent-state warning.

Every post-open request supplies the returned session. Every state-changing command supplies
`guard: {"stateToken":"..."}` from its latest observation. Read-only commands use null guard.
Admission AND main-thread execution recheck the token. A rejected/no-change request does not
advance it. Successful authored/history/document changes, asset/script publication, Play start/
stop and human edits advance it; render ticks and selection-only changes do not. The token is
an opaque aggregate revision, not a raw pointer or a persistent project UUID. Reopening/replacing
a Scene advances `sceneEpoch`, including reopening the same path. Process restart changes session.

Object references are `{ "sceneEpoch": "1", "id": "42" }`, or null where None is permitted.
IDs are canonical positive decimal strings, resolved only in that Edit Scene/session. Serialized
IDs may remap on load: old references reject after reopen; rediscover by copied hierarchy/name
and verify uniqueness. Names are labels, not unique IDs. No auto-rebinding by name or array index.
Script GameObject properties use these references on the wire; only the typed native adapter
converts them to Scene-local values. Inspection distinguishes Edit identity from current runtime
Scene identity; runtime handles/addresses are never accepted as Edit targets.

Core error codes: `InvalidRequest`, `UnsupportedVersion`, `UnknownCommand`, `DuplicateRequest`,
`SessionMismatch`, `StaleState`, `StaleReference`, `MissingTarget`, `UnsupportedCapability`,
`InvalidValue`, `DependencyRejected`, `PathRejected`, `ResourceFailure`, `ExternalChange`,
`MutationUnavailable`, `DecisionRequired`, `Busy`, `BudgetExceeded`, `ResultTooLarge`,
`OutputGrantRequired`, `PublicationFailed`, `InternalFailure`. Preserve a copied native reason
inside result where useful. Clients branch on codes, never parse human text. Unknown command,
field, enum or component/property writes reject before mutation; no best-effort property skipping.

## 4. Initial commands

`?` marks optional arguments below; no hidden target from selection. Result values include normal
envelope state/effects plus the listed contract. All mutations are Edit-only except Play stop.
The machine-readable command schemas generated during implementation must match these shapes.

| Command | Args | Result/authority |
| --- | --- | --- |
| `session.open` | `{}` | Identity, versions, limits, grants, current state; once per connection |
| `session.inspect` | `{}` | Fresh project/document/dirty/recovery/Play state and token |
| `session.close` | `{}` | Normal close if clean and idle; otherwise decision/rejection, no implicit discard |
| `capabilities.list` | `cursor?, limit?` | Stable type IDs, coverage and supported operations, including explicit unavailable types |
| `capabilities.describe` | `type` | Complete descriptor or honest read-only limitation |
| `scene.inspect` | `{}` | Scene identity, hierarchy counts, epoch, document status; not raw Scene JSON |
| `objects.list` | `parent?, nameContains?, cursor?, limit?` | Copied refs/names/parents/active/component IDs; omitted parent means all, null means roots |
| `object.inspect` | `target` | Copied supported fields, resource facts, readonly facts, world bounds/pose when already available |
| `assets.search` | `scope: Project or Shared, text?, type?, pack?, supportStatus?, cursor?, limit?` | Existing catalog results; IDs/generation, relative identities and support facts |
| `assets.inspect` | `scope, assetId, generation` | Dependencies, provenance, format/support facts; no filesystem authority |
| `assets.plan_add` | `assetId, generation, slug?` | Existing one pending Shared intake plan/token, selected files/provenance and destination identities |
| `assets.commit_add` | `planToken` | Revalidated Add to Project receipt; no arbitrary source/destination list |
| `object.create` | `name` | Existing CreateRoot outcome/ref; required/default components are disclosed |
| `object.duplicate` | `target` | Existing subtree clone/ref remap, history receipt |
| `object.rename` | `target, name` | Existing Rename |
| `object.reparent` | `target, parent` | Existing reparent or clear-parent when null, preserving existing transform semantics |
| `object.delete` | `target` | Existing subtree deletion/reference repair and Undo, never file deletion |
| `object.patch` | `target, changes` | Closed subset `active, position, rotationDegrees, scale`; one typed atomic candidate |
| `prefab.instantiate` | `identity` | Existing ordinary/variant instantiate and reference remap; no prefab file writer |
| `prefab.capture` | `root, identity, expectedSha256` | v1.14: confined ordinary prefab publication from current Edit state; null hash is create-only |
| `component.add` / `component.remove` | `target, type` | Initial fully covered optional types only; normal dependency/capacity/history rules |
| `component.patch` | `target, type, changes` | Nonempty descriptor-listed fields; one complete validated typed candidate, one history item |
| `resource.assign` | `target, slot, identity` | Slot Model/Material/CanvasImage/AudioClip/Script/PrefabSource/FlightImage/SkyImage/NavigationData; existing typed resource/path authority |
| `resource.clear` | `target, slot` | Existing clear semantics; Model clearing unsupported (use procedural operation) |
| `mesh.use_primitive` | `target, kind, parameters` | Existing procedural generator; typed closed parameters, no geometry arrays |
| `script.read` | `identity, offset?, length?` | Bounded UTF-8 source plus whole-file SHA-256; offset/length in bytes, do not split UTF-8 sequences |
| `script.write` | `identity, expectedSha256, source` | Confined Lua create/update, null hash means require absent; atomic file receipt |
| `scene.open` | `identity` | Existing Open document operation; dirty continuation if needed |
| `scene.save` | `{}` | Existing Save; PathRequired if untitled, no secret Save As |
| `scene.save_as` | `identity` | Existing Save As and overwrite/conflict policy |
| `scene.decide` | `decisionToken, decision: Save or Discard or Cancel` | Submit exact pending document continuation; stale tokens reject |
| `history.undo` / `history.redo` | `{}` | Existing Scene history, including ordinary availability rules |
| `play.start` / `play.stop` | `{}` | Existing preflight/PlaySession transaction and Stop restoration; no skip-validation flags |
| `play.inspect` | `{}` | Edit/Starting/Playing/Paused/Failed state, current runtime Scene, committed frame count and diagnostics summary |
| `diagnostics.read` | `after?, limit?` | Bounded copied diagnostics, severity/source, next cursor and dropped count |
| `build.request` | `target: Windows or Web` | One job ticket using granted output parent, saved Build Settings and compatible installed templates |
| `build.inspect` | `ticket` | queued/running/succeeded/failed, copied publisher result, output and ZIP receipts |
| `material.inspect` | `identity` | Copied typed Material values, digest/bytes and dirty/conflict relation |
| `material.create` | `identity, properties` | Native defaults plus typed fields; destination must be absent |
| `material.patch` | `identity, expectedSha256, properties` | Nonempty typed patch; exact lowercase digest, complete candidate validation and atomic publication |
| `navigation.bake.request` | `target, identity` | Guarded Surface and confined navigation identity; admitted job is not publication |
| `navigation.bake.inspect` | `job` | Copied Working/Published/Cancelled/Rejected/Failed receipt, effects and published digest/bytes |
| `navigation.bake.cancel` | `job` | Session job cancellation; terminal no-change does not delete/undo published bytes |

Version 1 deliberately omits project switching, generic Scene JSON/Material/prefab/Build Settings
file writes, runtime property mutation, input injection, arbitrary scripts on
the host, full schema reflection, compound batch transactions and external website uploads.
Existing projects/prefabs may contain uncovered components; inspect/copy/instantiate them through
normal operations without silently dropping fields. Direct writes/add/remove to uncovered types
return `UnsupportedCapability`. UI-specific operations remain available through the normal editor;
that limitation is advertised, never described as complete agent support.

### Documents, scripts and publication

Material commands confine identities to `assets/materials/*.material.json`, use the current13
MaterialAssetData fields/format4, and reject dirty GUI drafts or changed source bytes. Texture paths
are typed confined Material fields, not an arbitrary filesystem channel. Native publication
rechecks conflicts/containment; prepublication failure preserves old bytes. A postpublication
refresh warning is not rollback. File effects advance session tokens but are not Scene Undo.

Navigation exposes the existing detached native bake, one active job/eight bounded session receipts.
The main thread validates unchanged document/source/destination before publishing
`assets/navigation/*.nav.json`; inspect distinguishes admission, cancellation and actual publication.
Mutations/Play/build/close respect the existing active-publication guards; EOF cancels safely.
Assigning a different NavigationData reference uses Scene history; overwriting bake bytes does not.
SkyImage clear requires a valid remaining record (Gradient or disabled draft); stored nonempty
images are always validated. NavigationData uses native reference/lifecycle checks.

All asset/Scene/script identities are canonical project-relative strings. Existing policies still
reject escapes, absolute/UNC/device paths, alternate streams, invalid extensions, reparse escapes
and writes to protected roots. Recheck final targets at publication; do not rely only on string
prefixes. Catalog IDs and intake tokens cannot authorize an outside asset or a different generation.

`script.write` is limited to `assets/scripts/*.lua` and subdirectories, at most **65,536 UTF-8
source bytes**; read returns at most 16,384 bytes per chunk. The command envelope remains bounded
after escaping. Reject binary/NUL/invalid UTF-8, syntax failure and changed expected hashes.
Compile text for syntax only through the existing Lua dependency; do not execute top-level code
or callbacks during validation. API/callback/runtime preflight still happens through Play/Build.
Stage and atomically publish a file, retaining the original on pre-publication failure. Refresh
the existing catalog/source cache; no hot reload into running Play. A syntax-valid source may
still fail runtime preflight, which must be reported honestly. This file operation has no Scene
Undo; its receipt says so. It cannot write `.json`, manifests, project settings or engine source.

The build adapter uses the saved Build Settings and existing dirty-document guards. It cannot
replace the declared Scene closure, choose runtime binaries, accept untrusted templates, bypass
preflight or overwrite an existing output. `accepted` returns before one normal publisher runs
at a safe main-thread boundary; no new concurrent publisher is required. Polling can wait while
the existing synchronous publisher runs. The client must not infer failure from that delay.
If Web folder publication succeeds but ZIP fails, return both facts; do not erase the valid folder.
No remote upload, preview server or shell-opening operation is part of protocol 1.

`prefab.capture` accepts a guarded root reference and canonical `assets/prefabs/*.prefab.json`
identity. Null `expectedSha256` requires an absent destination; a lowercase SHA-256 authorizes
replacement of only those exact ordinary prefab bytes, never a variant. Capture is limited to64
objects/4 MiB by the adapter, not by unrelated native GUI capture. It uses the shared typed operation,
reference validation, serializer and atomic saver. Recheck path/reparse/sidecar safety and digest at
publication. A conflict or prepublication failure preserves the destination. The result contains
identity, sha256, bytes, objectCount, filePublished, catalogRefreshed and diagnostics. A failed
catalog refresh after publication is a warning, not a claimed rollback. Publication advances the
state token but has no Scene Undo. `assets.inspect` supplies the current project-file digest.

PrefabSource permits an incomplete empty Edit assignment, with existing enabled Play requirements.
FlightImage may clear only on a disabled emitter; enabled emitters require a resolved image and
same-Canvas target. Resource paths cannot be written through component.patch. CanvasImage and
AudioClip retain their required-resource rejection semantics; Material/Script retain optional clears.

## 5. Descriptor schema 1 and complete coverage

v1.16 adds CharacterControllerComponent, CharacterMotorBehaviour, CharacterAnimationBehaviour,
AudioListenerComponent and RollingVisualComponent, for43 complete records. CharacterAnimation's
visual binding remains derived from its sole eligible direct-child Animator, not a writable ref.
FollowCamera adds PathGuided, pathSource, pathLookAheadDistance and pathHeadingHalfLife. Discover
exact native defaults/ranges via descriptors; field/reference changes still use typed history,
whole-record validation, clone/remap and repair. Rolling leases only its specified direct child;
the generic Scene-root pose authority is not relaxed. No runtime-spawning eligibility is implied.

v1.15 adds complete SkyEnvironment, Light, Checkpoint, PathFollower, FollowCamera, ContactDamage,
NavigationSurface, NavAgent, Perception, AgentStateController, Animator, AnimationStateController,
Interactor and Interactable records to the preceding24. Structured points/states/markers/targets
retain native cardinality/reference/remapping and sole-pose-authority rules. For example a menu
PathFollower parent can move an Animator child; they cannot compete on the same owner. Authoring
support does not imply eligibility for runtime spawning. Generated descriptors advertise exact
fields/defaults/limits/dependencies and honest unsupported operations; no runtime/derived state is writable.

Every component/type descriptor includes `schemaVersion: 1`, stable `type`, `displayName`,
`coverage: complete or read_only`, `operations`, `defaults`, `fields`, `dependencies`,
`constraints`, `limits`, `platforms` (authoring Windows; produced content Windows/Web),
`cloneAndReferences`, `history`, `runtimeActorSupport` and `limitations`.
Each field has stable name, kind, default, units, finite numeric bounds where applicable,
enum vocabulary/length/item limits, nullable/reference requirements and `writeCommand`.
Cross-field constraints have stable IDs and readable explanations backed by native validation.
Required object/component dependencies and Scene-wide restrictions must be discoverable, not
learned only from rejection. No pointer, GPU/native-body handle, object address or implementation
enum integer is exposed. Inspect values are copies, not writable serialized Scene records.

Kinds: Boolean, Number, Integer, String, Enum, Vector2, Vector3, Color3, Color4, ObjectReference,
AssetIdentity, PrimitiveParameters, ScriptPropertyList and bounded typed structured lists. Resource identities are written only
through resource operations; mesh geometry/source only through `mesh.use_primitive`/Model assign.
Mixing these routes into component.patch rejects rather than partially applying ordinary fields.

The following **initial ten** type records have complete writable authored coverage. Transform is an
inherent object record, not an addable/removable Component. All nine optional Components include
their native `enabled` field (default true). Type IDs match existing persisted names exactly.

| Type | All fields/defaults and constraints |
| --- | --- |
| `Transform` | position (0,0,0) local metres, rotationDegrees (0,0,0) Euler degrees, scale (1,1,1) ratio with each component >=0.1. Finite native-float values; preserve current hierarchy/decomposition validation and T * Rz * Ry * Rx * S convention. Reject values that would silently clamp. Object name/active are separate object properties. |
| `MeshRenderer` | tint (1,1,1) Color3, wireframe false; sourceType Procedural/ImportedModel, primitiveType Cube, primitiveParameters, modelPath empty, materialPath empty. Model/Material resource routes preserve existing shaders/import contracts. Tint uses current finite color validation. |
| `CameraComponent` | primary false, fieldOfView 45 degrees [20,90], nearPlane 0.1 metres >0, farPlane 100 metres >nearPlane, finite. Perspective only; report existing primary-camera composition rule. No new projection mode. |
| `AudioSourceComponent` | clipPath empty (resource route), playOnStart true, loop false, volume 1 [0,1], pitch 1 [0.25,4], spatialMode TwoDimensional/Spatial3D default TwoDimensional, minimumDistance 1 [0.01,1000] metres, maximumDistance 25 >minimum and <=10000. WAV policy/mono spatial requirement and listener dependency remain unchanged. |
| `CanvasComponent` | referenceWidth 1280, referenceHeight 720 reference pixels, each [1,8192]; renderOrder 0 integer [-32768,32767]. Screen-space existing Canvas; max eight Canvases/256 UI visuals per Scene. |
| `RectTransformComponent` | anchorMinimum/anchorMaximum/pivot (0.5,0.5), normalized [0,1], minimum<=maximum; anchoredPosition (0,0), sizeDelta (200,100) reference pixels, finite components [-100000,100000]; renderOrder 0 [-32768,32767]. Existing layout and ancestor Canvas rules. |
| `UIImageComponent` | imagePath empty (resource route), tint (1,1,1,1) [0,1], scaleMode Stretch/AspectFit default Stretch. Existing image formats/RectTransform/Canvas requirement. |
| `UITextComponent` | text Text, color (1,1,1,1) [0,1], fontSize 24 [1,256] reference pixels, alignment Left/Center/Right default Left. Text 1..128 bytes; current glyph vocabulary: ASCII letters/digits/space/colon/hyphen/period only. No silent unsupported glyph replacement. |
| `UIButtonComponent` | interactable true, actionName ButtonAction (1..64 ASCII bytes, first letter then letters/digits/underscore/period/hyphen), hoverTint (0.90,0.95,1,1), pressedTint (0.70,0.80,1,1), focusedTint (1,0.90,0.60,1), disabledTint (0.50,0.50,0.50,0.65), colors [0,1], initiallyFocused false. Preceding eligible same-object visual/RectTransform/Canvas and unique eligible initial focus follow production validation. |
| `ScriptBehaviour` | scriptPath empty (resource route), properties empty. Full typed list of max32 unique valid names (max64 bytes): Boolean, Number, String (max1024 bytes/current string rules), Vector3, GameObject. Finite numeric values and valid current-Scene/null references. Replace list atomically, not arbitrary table/JSON merge; retain existing Lua sandbox/API/version/preflight limits. |

Primitive kinds are Cube, Plane, UVSphere, Cylinder, Cone. Parameters/defaults: cubeSize1;
planeWidth1/planeDepth1/planeRows1/planeColumns1; sphereRadius0.5/segments32/rings16;
cylinderRadius0.5/height1/segments32/capped true; coneRadius0.5/height1/segments32/capped true.
Wire parameter keys are the exact `PrimitiveParameters` names: sphereSegments/sphereRings,
cylinderHeight/cylinderSegments/cylinderCapped, coneHeight/coneSegments/coneCapped and the
fully named size/radius/plane keys above, not abbreviated segments/height/capped aliases.
Dimensions are [0.01,100] metres, rows/columns [1,256], segments [3,256], sphere rings [2,256].
Reject out-of-range agent values before the existing clamping generator: never silently report
the requested value as applied. Descriptor enum spellings use stable tokens (`UVSphere` on wire).

v1.14 retains those ten records and adds14 complete records, for24 total. Exact field defaults,
numeric bounds, enum tokens, reference cardinalities and write routes are exported from the native
registry in `initial-descriptors-v1.json` (the filename is retained for compatibility):

| Added type | Complete authored fields after enabled |
| --- | --- |
| ColliderComponent | shape, center, boxSize, radius, capsuleHeight, trigger, layer, mask; active shape geometry validates natively |
| RigidBodyComponent | motionType, mass, gravityFactor, friction, restitution; non-Static runtime actor bodies remain roots |
| HealthComponent | maximumHealth, startingHealth; starting cannot exceed maximum |
| RespawnableComponent | automaticOnDepletion, respawnDelay, restoredHealth |
| StompableComponent | stompDamage, bounceSpeed, minimumClosingSpeed, maximumTopAngleDegrees, characterLayerMask |
| BreakableComponent | playBreakAudio, emitBreakParticles, spawnContents, contentsOffset, terminalAction |
| PrefabSpawnerComponent | prefabPath via PrefabSource, maximumLiveInstances |
| ParticleBurstComponent | particleCount, lifetimeSeconds, initialSpeed, spreadAngleDegrees, gravity, startSize, endSize, startColor, endColor, emissiveIntensity, seed |
| DestroyAfterTimeBehaviour | delaySeconds |
| ObjectiveCounterComponent | completionSource, members (up to256 guarded references) |
| ConditionalRewardComponent | requiredCounters (up to4 guarded references), spawnOffset |
| BounceSurfaceComponent | bounceSpeed6, minimumClosingSpeed0.5, maximumTopAngleDegrees45, characterLayerMask65535 |
| TimedExplosiveComponent | fuseSeconds3, blastRadius2.5, damageAmount50, damageTargetMask65535, chainTargetMask65535, chainDelaySeconds0.2 |
| UIFlightEmitterComponent | target null, imagePath empty via FlightImage, tint[1,1,1,1], iconSize32, durationSeconds0.55, arcHeight64 |

The three new mechanics/presentation Components default disabled; all supplied values still
validate. Bounce requires solid Collider/Rigid Body and excludes enabled Stompable. Explosive
requires Health, solid Collider/Rigid Body and Destroy-mode Breakable, with no enabled Ragdoll.
Flight requires an owning Canvas, resolved image and same-Canvas descendant RectTransform target.
List references retain native uniqueness/None-draft rules and clone/history/removal repair.
UIFlightEmitter, Respawnable, ObjectiveCounter and ConditionalReward are not prepared runtime
actor types, including disabled records. New scalar/vector variant overrides use the existing
whitelist; reference/image identity overrides are not introduced.

Coverage still excludes writable UIPanel, system-menu markers, Character controllers,
animation rigs and other undiscovered types. Preserve/duplicate existing authored objects where
useful; do not claim universal reflection. Missing required authoring capability is a defect or
reported limitation, never permission for Scene JSON/manual-rescue or generic full-record transport.

## 6. Maintained artifacts and evidence

Protocol phase rules are command-specific: read-only calls may inspect unavailable/Play state;
authoring/history/document/resource/script mutations require Edit; play.start requires its normal
Edit preflight; play.stop is the sole Play-state mutation. Session open/close and build admission
also have their lifecycle/idle requirements above. No command silently forces Stop to gain access.

Implementor produces a matching machine-readable closed command/envelope schema and descriptor
export from the authoritative registry, under `docs/authoring/` (protocol-v1.schema.json and
initial-descriptors-v1.json). They are reference artifacts, not a second parser/persistence system.
Defaults/limits/enum checks compare them with production validators in focused tests.

The repository skill lives at `skills/moe-magic-game-authoring/SKILL.md`; the plan specifies its
small reference set and shipped `Authoring/` placement. Do not auto-install it into a user's
Codex configuration, create a network integration or change tool permissions. Its transport recipe
must demonstrate a real persistent redirected process using an available agent process tool.
If Windows GUI process piping requires a helper, ship one small dependency-free Windows
PowerShell/.NET pipe adapter; it only forwards framed requests/responses, never reads/writes
project content or invents operations. It is not required in the ordinary graphical user journey.

The actual external early exercise and later installed exercise are mandatory. A C++ test invoking
the dispatcher, a transcript replay with hardcoded internal IDs or manual source/Scene rescue is
not equivalent. Record client/skill/descriptor identity, request/response receipts, discovered asset
IDs and output identities without embedding machine-specific paths into authored content.
