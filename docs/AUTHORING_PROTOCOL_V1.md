# Bounded editor authoring protocol 1

Status: **IMPLEMENTED v1.13.0 CANDIDATE, NOT RELEASE ACCEPTANCE** — owner approval 2026-11-04.
The v1.13.0 plan (see installed documentation/source reference)
owns scope, exercises and acceptance. Released v1.12.0 does not expose this protocol.
This document owns the wire/descriptor contract; the plan must not maintain a competing copy.

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
| `component.add` / `component.remove` | `target, type` | Initial fully covered optional types only; normal dependency/capacity/history rules |
| `component.patch` | `target, type, changes` | Nonempty descriptor-listed fields; one complete validated typed candidate, one history item |
| `resource.assign` | `target, slot, identity` | Slot Model/Material/CanvasImage/AudioClip/Script; existing resource assignment and path policy |
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

Version 1 deliberately omits project switching, generic Scene JSON/Material/prefab/Build Settings
file writes, navigation baking, runtime property mutation, input injection, arbitrary scripts on
the host, full schema reflection, compound batch transactions and external website uploads.
Existing projects/prefabs may contain uncovered components; inspect/copy/instantiate them through
normal operations without silently dropping fields. Direct writes/add/remove to uncovered types
return `UnsupportedCapability`. UI-specific operations remain available through the normal editor;
that limitation is advertised, never described as complete agent support.

### Documents, scripts and publication

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

## 5. Descriptor schema 1 and complete initial coverage

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
AssetIdentity, PrimitiveParameters and ScriptPropertyList. Resource identities are written only
through resource operations; mesh geometry/source only through `mesh.use_primitive`/Model assign.
Mixing these routes into component.patch rejects rather than partially applying ordinary fields.

The following **ten** type records have complete writable authored coverage. Transform is an
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

Initial complete coverage does not include UIPanel, system-menu marker types, animation rigs or
other gameplay Components. Preserve/duplicate existing authored buttons and markers where useful;
never retrofit all legacy types merely to claim universality. Broader coverage requires the next
approved release or an explicit amendment. If the ten-type exercise cannot work, it is a defect,
not permission to edit Scene JSON. A menu need for an uncovered new property must be surfaced
before expanding scope, not disguised as full-record ApplyAuthoredProperties transport.

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
