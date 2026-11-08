# Bounded agent gameplay test sessions

**Implemented v1.24.0 candidate contract, 2026-11-08; not an accepted release.**
This document specifies the input/step/capture addition to the existing
[v1.24 plan](exec-plans/v1.24.0-structured-mechanisms-and-two-genre-authorability.md).
It does not replace completed descriptor work or authorize v2. Protocol1 capability revision13
exposes61 complete descriptors and80 commands in the Editor: the existing73 plus the seven
`test.*` commands below. Scene56/prefab49/API28/distribution9 and saved project formats stay unchanged.
The implemented revision12/61/73 checkpoint remains valid historical evidence, not the new target.
Focused native/browser evidence and its simulation/host limitations are recorded in the
[implementation report](implementation-reports/v1.24.0-implementation.md). Tester owns acceptance.

## Purpose and authority

An external author can submit bounded virtual-device input, advance normal gameplay, inspect
committed state and retrieve actual rendered images without OS-wide input or owner key-holding.
This is an opt-in runtime input source and clock/capture adapter, not a private validation path,
AI provider, test planner, arbitrary script/eval service, new renderer or general network service.

Reuse PhysicalInputState, InputActionMap/authorization/latches, RuntimeUIController/overlays,
GameplayLookLease, PlaySession/BehaviourSystem scheduling, committed observation services and
the existing resolved Game rendering path. Never set a Character pose, joint angle, Health,
objective, UI action callback or success flag in order to simulate input. Project Lua remains
ordinary gameplay; the API does not run arbitrary client code or edit runtime objects.

## 1. Explicit host grants and isolation

### Windows Editor and Player

- Editor requires `--project <disposable-root> --authoring-stdio --authoring-test-session`.
  The new flag is not enabled by ordinary authoring or normal GUI launch. Retain existing launch
  confinement, redirected pipes, project/output grants and native document safeguards.
- Windows Player supports an explicit `--agent-test-stdio` mode with its existing content-root
  launch argument. It grants only `session.open/inspect/close` and the seven test commands, not
  Scene authoring, scripts, builds or filesystem access. Reject ordinary validation/interactive
  command combinations. Move/reuse only the narrow byte envelope and runtime dispatcher utilities;
  do not link Editor operations or COM dialogs into Player/runtime common code.
- Both require an explicitly identified disposable project/export, and `test.begin` must confirm
  its observed content identity. Reject known engine/source sample/install roots and owner profile/
  save roots. A `disposable:true` assertion is a confirmation of the granted path, not proof that
  an arbitrary user directory is disposable. The client must name the actual authorized copy.
- Establish storage isolation before runtime startup callbacks: use existing progress/input/
  graphics storage interfaces with session-local in-memory adapters. No read/write of owner saves,
  Windows profile preferences, browser IndexedDB/localStorage or `imgui.ini`. Runtime UI Apply/Reset
  uses the same validation/feedback but commits only to this ephemeral profile. Scene/settings on
  disk are not changed. Reset the ephemeral state when the test session ends; no persistence PASS
  is inferred. Installed/ordinary user modes retain their normal storage unchanged.

### Web: explicit local Preview, not a network control server

Use the existing verified Web export and shipped read-only loopback Preview helper. Add an explicit
helper `--agent-test` launch option; the maintained client can launch it directly without desktop
input. In that mode only, an unguessable session-prefix-scoped GET bootstrap resource provides an
ephemeral capability bound to the exact verified Web manifest digest and helper lifetime. Ordinary
Preview omits it. No POST control endpoint, new listener, WebSocket, remote origin, CORS wildcard,
filesystem operation, upload or mutation of published game files. Existing host/token/path checks
remain. Reject test bootstrap on non-loopback origins or mismatched content.

The Web shell exposes one bounded `MoeMagicTest.request(envelope)` bridge only after this grant.
It submits the same closed runtime subset to WASM on the main browser loop and returns bounded
receipts asynchronously. No eval/string execution, arbitrary export invocation or cross-origin
postMessage bridge. Export only the required fixed submit/drain functions through the pinned
Emscripten build; do not upgrade the SDK. The page subset advertises ten commands (three session
commands plus seven test commands), not the Editor's80 authoring commands. Same protocol/version,
IDs, sizes and request validation; no invented Edit objects in a Player.

Runtime modules may include the dormant adapter, but ordinary Player launch, ordinary Preview,
downloaded/itch-hosted games and normal installed users cannot activate it without the explicit
test host grant. A query/fragment flag alone is not a grant. No test capability or transcript is
embedded in a public ZIP/manifest. Normal publishing formats and file closure remain unchanged.
The helper's read-only bootstrap is the only new HTTP role; all test controls stay inside the
opted-in page/WASM instance. Parent/helper termination revokes the grant; browser polling/heartbeat
checks and the bounded lease below prevent abandoned input. Never log a reusable capability token.

### Physical versus virtual ownership

One exclusive virtual-device lease per runtime. It replaces, never merges with, physical gameplay
devices while active. Physical editor/window emergency Stop/Escape remains available and cancels
the lease; it must not become a simulated in-game key. Ordinary Scene/editor shortcuts receive no
synthetic events. OS focus is not stolen or claimed: the opted-in runtime surface has a distinct
simulated eligibility source, while real document modals, resource/context loss and runtime UI
consumption retain their guards. Releasing the lease clears pending input and restores ordinary
neutral/reacquisition rules; a physically held key cannot leak into a subsequent normal Play.

No Windows SendInput, cursor relocation, global keyboard hooks, DOM KeyboardEvent faking or hidden
direct gameplay mutation. Synthetic gamepad values are virtual engine controls, not evidence that
a controller is attached or its browser mapping works.

## 2. Command contract and limits

Reuse the protocol1 envelope, increasing decimal request IDs, session/state guards, bounded errors
and main-thread dispatch. Unknown fields/commands reject; no automatic retry after an unknown
mutation outcome. test.begin/run/end require the current state guard. Read-only inspect/artifact
requests use null guard. Cancel accepts the exact session/test/job identity even if the ordinary
authoring guard changed, so stale state cannot prevent release of that job's held controls.
Capture checks the exact committed boundary, not an old editor-selection guard.
Player session state uses the same opaque token discipline, with its runtime Scene epoch,
`dirty:false` and actual Play state; it has no Edit document/history authority. Lease begin/end
advance the token; stepping alone does not. While a test lease exists, reject authoring mutation,
build/bake and competing normal Play controls as busy. Reads remain available; physical/ordinary
Stop is an emergency end, never blocked by the lease. Editor document writes cannot race test data.

| Command | Closed operands and result |
| --- | --- |
| `test.begin` | `disposable:true`, `expectedContentDigest`, `viewport:{width,height}`. Requires stopped/clean runtime startup, no unresolved document decision or authoring/bake/build job. Starts through normal saved-content/preflight/Play startup, then returns a controlled-idle lease before simulation advances. Returns testId, content/host identity, initial boundary, limits, session-local storage and input-source disclosure. |
| `test.run` | `testId`, `expectedBoundary`, exactly one of `frames`/`fixedTicks`, `events`, optional `observe`, `captureAt`, `wallTimeoutMs`. Validates the entire schedule before changing any input; returns accepted job ticket, not completion. One active job, no queue chaining. |
| `test.inspect` | `testId`, optional `ticket`, `collection` = `status`, `objects`, `observations` or `diagnostics`; collection-specific `targets`, `groups`, `after`, `limit`, `cursor` only. Returns current/terminal receipts or bounded copied data. Does not advance time. |
| `test.cancel` | `testId`, `ticket`. Idempotently cancels that job, neutralizes devices/pending deltas and leaves controlled-idle, with completed counts and reason. It never rolls back already committed gameplay. |
| `test.capture` | `testId`, `expectedBoundary`. Queues one actual Game-frame render/readback at that idle boundary without simulation; returns an accepted job ticket. `test.inspect` reports the artifact only after render/encoding completes. No arbitrary image or path operand. |
| `test.artifact.read` | `testId`, `artifactId`, `offset`, `length` (raw bytes, max65536). Returns base64 chunk, total size/digest and next offset. Session-local known artifact IDs only; no path traversal or file reader. |
| `test.end` | `testId`. Cancels pending work, neutralizes input, stops test Play/runtime and disposes temporary profile/clock ownership. Editor returns to its unchanged Edit Scene; Player stops its test runtime. Never silently resumes physical gameplay. |

Numeric bounds: viewport320–1280 by180–720 pixels, at most921600 pixels; default client request
960x540. Exactly1–600 frames or1–600 fixed ticks per job,1200 maximum attempted frames for a tick-target
job,128 ordered input events,32 observation boundaries and4 capture boundaries. dt is the existing
fixed quantum (currently1/60s), not a client-controlled float. `wallTimeoutMs` defaults10000, range
100–30000; no unbounded run/until condition. Requests retain262144-byte/depth16 and responses1MiB
limits. One host pump advances at most one controlled frame; process cancel/transport health first.
No tight600-step loop starving the editor/browser. Check deadline before each step; a currently
executing native frame is not preemptible, and a slow return records overrun rather than false
hard-real-time compliance.

Retain at most8 terminal jobs/capture receipts and8 PNG artifacts, max4MiB each/32MiB total, plus
one viewport-sized readback/encoding scratch set. Retained non-image samples/receipts have a2MiB
per-job and16MiB aggregate ceiling. Reject new evidence reservations at capacity, never evict a referenced
artifact during encoding/read or silently downscale. Artifacts/receipts remain readable after
test.end until session close/new test/explicit documented expiry; no files are written by capture.
The installed client may save fetched PNG/JSON bytes only to its explicitly authorized evidence
directory, with non-overwriting names. This is output collection, not a generic engine writer.

### Closed identity/discovery records

The production schema defines these records once and exports them for both transports:

- `Boundary = {runtimeGeneration, sceneEpoch, committedFrame, fixedTick}`; all four decimal strings,
  compared as one exact tuple. `expectedBoundary` must equal the idle committed boundary. A capture
  adds `renderSerial`, without changing that tuple. Jobs pin their initial tuple; normal progress
  inside the job is not a stale-guard failure.
- `expectedContentDigest` is the SHA-256 of the existing validated saved-content snapshot/closure
  selected for startup (Editor current saved Scene and dependencies; Player verified package
  manifest/closure). Return its scope and exact digest in session discovery. Recheck at begin and
  refuse changes during the lease, rather than hashing only the display name or trusting a caller.
- `TestObjectRef = {testId, runtimeGeneration, sceneEpoch, objectId}`; opaque IDs are issued only
  by `objects`. No raw object integer supplied by a caller can acquire authority.
- `observe` is an array of `{frame, targets, groups}`; targets1–8 from the current runtime and
  groups chosen from the existing `motion`, `progress`, `combat`, `pose` copied inspection families.
  Include copied world pose in each target result. Use the existing family field/null/reason limits,
  not raw runtime serialization. `captureAt` is an array of unique frame offsets. Both are bounded
  and strictly ordered; their frame offsets are after-update positions, like the event offsets.
  `frames` jobs reject offsets outside[0,N-1]; tick jobs use attempted-frame offsets[0,1199] and
  report unreached requests if the tick target completes sooner. No implicit extension to reach them.
- `objects` uses `after`/`limit`; `observations` uses `targets` and `groups`; `diagnostics` uses
  `cursor`/`limit`; `status` optionally uses `ticket`. Other collection fields reject. Each response
  states its actual boundary and collection availability; no unbounded accumulated transcript.
- In the opted-in runtime subset, `session.open/inspect` advertises host, revision, supported
  commands/limits, current content identity, input enum values, effective action bindings/dead zones,
  pointer convention and simulated-look eligibility. This is copied discovery, not a setter.
  Editor reports the same `testCapabilities` alongside ordinary authoring discovery. This lets a
  client choose real bound controls without guessing source IDs or needing Editor APIs in Player.
- Reuse existing protocol error categories where applicable; provide stable test-specific reasons
  for grant/content mismatch, busy, stale boundary/ref, invalid schedule, capacity, blocked clock,
  capture failure and revoked lease. Unknown outcomes require inspection, not automatic mutation retry.

Diagnostic/observation reservations must also fit the existing1MiB response/pagination limits.
Do not accumulate32 unbounded pose dumps; retain bounded samples and expose paged results with
explicit truncation/availability. An implementation must fail reservation before running input if
it cannot retain the promised evidence within its declared caps.

## 3. Virtual input schedules

Events use zero-based `frame` offsets and a closed `kind`. Sorted order is required; contradictory
same-frame writes to the same control reject atomically. A control retains its value until changed
within that job. All controls are neutral at job start and automatically neutral after completion,
failure or cancel. To hold, submit down/value and advance; release explicitly on a later frame or
use job cleanup. A tap is down and up on different frames, never a same-frame edge that disappears.

| kind | Fields / meaning |
| --- | --- |
| `button` | `frame`, `device` = Keyboard/Mouse/Gamepad, `control` from the existing closed enum, `held` Boolean. No platform scan codes or unrestricted names. |
| `axis` | `frame`, `control` from existing GamepadAxis, `value`; sticks[-1,1], triggers[0,1]. |
| `stick` | `frame`, `control` = LeftStick/RightStick, `value:[x,y]`, each[-1,1]; conflicting underlying axis writes in that frame reject. |
| `pointer` | `frame`, `position:[x,y]` normalized top-left [0,1], `inside` Boolean. Maps to the lease's actual render dimensions through the normal RuntimeUI pointer mapper. Mouse buttons are separate button events. |
| `look_delta` | `frame`, `delta:[dx,dy]`, finite relative virtual-mouse pixels, each[-512,512], consumed once and not multiplied by dt. Only under an eligible explicitly simulated look lease. |
| `cancel` | `frame`, `held` Boolean for the normal runtime Escape/Back cancellation path; separate from physical emergency cancellation. |

The client discovers Input Actions/bindings and chooses supported physical-control identities;
there is no direct `set_action_value`, UI button-ID activation, teleport or arbitrary action callback.
Movement/look analog sources go through existing dead zones, bindings, authorization and latches.
Pointer/UI press/release, keyboard/gamepad navigation, native Controls/Graphics and cancellation
run their ordinary eligibility/consumption code. A UI-owned control cannot also move/fire gameplay.
Out-of-surface pointer state releases/cancels according to the native UI rules. No synthetic input
is delivered to editor chrome, native dialogs, Windows shell or browser chrome.

First-person test look uses a distinct **simulated** lease provenance, never a fabricated OS cursor
capture or browser pointer-lock grant. Route deltas through the same first-person input ingress;
preserve base-look/recoil/neutralization and runtime modal suppression. Gamepad Start/right-stick
uses the ordinary pad path. A denied/unavailable simulated route is a reported contract failure,
not permission to set yaw directly. Receipts always identify `inputSource=Simulated`.

## 4. Clock, pause and exact ordering

test.begin initializes a fresh test runtime through ordinary preflight/Start. Startup rendering/
branding may complete normally; the first gameplay frame waits for test.run. Do not adopt an
unrelated owner-controlled live Play or reset the current disposable Edit Scene. Starting a new
test can reset that test attempt by explicit end/begin; it never rewinds an existing job silently.

Each controlled frame performs:

1. Check grant, cancellation, deadline, context and expected generation/Scene boundary.
2. Apply that frame's virtual device changes, transient pointer/look delta and neutral/ownership rules.
3. Compose/preprocess normal runtime UI/overlays and their gameplay consumption mask; evaluate the
   effective InputActionMap through the normal authorization/edge/latch machinery.
4. Advance PlaySession through its normal callback/fixed-step/commit/presentation scheduling with
   one fixed-quantum frame. Extend the existing SingleStep seam to accept authorized test input;
   do not call Pause/Resume for every step (those clear state), fabricate callback order, or bypass
   native game pause, recovery, terminal ownership or modal suppression.
5. Finish deferred mutation, achieved physics pose, camera/animation/UI and any ordinary Scene
   transition transaction. Copy observations only after that committed boundary.
6. When requested, render the actual Game result and UI overlays, then read its resolved pixels.
   Complete that boundary's capture before advancing to another requested boundary.

Between jobs the test clock is **controlled-idle**: no simulation, Lua timer or input advancement.
test.run is bounded resume; completion/cancel is bounded pause with automatic device neutralization.
Ordinary manual Play Step remains neutral-input and unchanged outside this grant. Existing ordinary
fixed accumulator semantics are not changed to serve tests; the controlled seam uses the existing
single-fixed-step mode, preserving one normal frame of Update/LateUpdate/presentation per step.

`frames:N` means N controlled rendered-game update frames, possibly zero fixed ticks while native
game pause/UI is active. `fixedTicks:N` means N actually completed physics ticks, not attempted
updates. Stop as `blocked` if game pause/modal prevents tick progress; preserve partial counts and
allow a later frame-mode UI schedule. Never pretend N frames imply N physics ticks. Capturing an
idle frame adds a render serial, not a simulation frame or fixed tick. This controlled schedule
supports repeatable observations; it does not promise cross-host bit-identical physics/PNG pixels.

Scene transition ends the current job as `boundary_changed`, drops remaining inputs and returns
new Scene/runtime identity. Client rediscovers targets and submits a new schedule deliberately.
No held Fire/Interact crosses a menu/replay transition. Failed preflight/transition, resource loss,
runtime fault or Quit yields a terminal receipt with the actual partial outcome, never false success.
Quit must allow its terminal receipt to drain before test-host teardown; ordinary Quit is unchanged.

## 5. Observations, rendered output and receipts

Every job/sample/capture identifies session, testId, runtime generation, Scene epoch/identity,
content/manifest digest, input-map generation, committed frame, fixed tick and render serial where
applicable. Counters are decimal strings. Frames/ticks are actual commits, not requested totals.
Receipt states: accepted, running, completed, cancelled, timed_out, blocked, boundary_changed or failed;
record requested/completed counts, simulated seconds, wall time/overrun, stop reason, control lease,
released-input acknowledgement, diagnostics cursor/drop counts and artifact IDs/digests.
Completed input execution is **not** a gameplay PASS; callers must evaluate achieved observations.

`test.inspect(objects)` copies paged names, hierarchy, component types and opaque TestObjectRefs
for the current runtime (default32/max64, existing bounded cursor rules). References contain test/
generation/Scene identity; they cannot mutate or survive Scene replacement. `observations` accepts
1–8 such targets and uses existing motion/progress/combat/pose query authorities with copied world
poses. No invented original-Edit mapping in exported Players. Factor shared copy helpers if needed;
do not create a second simulation/status authority. The Editor's old play.* inspections remain
available under their original mapping limits. `observe`/`captureAt` on test.run identify frame
offsets after which to copy/render; target validation and output reservation happen before execution.
Diagnostics use the existing bounded ring/size/truncation disclosures, never arbitrary log files.

The motion copy also exposes the existing CharacterStateView's committed `grounded`,
`requestedHorizontalSpeed`, `successfulFixedRevision`, `lastAcceptedJumpRevision`,
`lastAcceptedBounceRevision` and `lastLandingRevision`, with availability/reason and nullable values.
This is a copied existing native state, not a new grounding algorithm. Requested speed is not
achieved displacement. The current query does not identify the support body: prove on-deck/landing
placement through copied world poses, collider bounds and actual frames, and never invent a support
identity. No broad physics contact API is required for this amendment.

Capture is the actual resolved LDR **Game surface after world, Canvas and runtime overlays**, not
Scene View, a reconstructed drawing, just a world-only buffer or the desktop. Use the existing Game
FBO/presentation resolver and overlay path at the explicit test viewport, restoring ordinary render
sizing on test.end without editing layout. Web reads its composed canvas at the same pre-swap/frame
boundary; preserve the framebuffer/context rather than enabling permanent preserveDrawingBuffer.
Normalize captured rows to top-left, encode PNG with dimensions/color interpretation/digest, and
bind metadata to that exact committed observation. A stale last-frame image must not satisfy a new
capture; render/encode failure reports failure. No screenshot of editor panels, credentials or other apps.

Use platform PNG facilities behind the narrow host adapter: native Windows WIC and browser PNG
encoding of the captured pixel buffer. No hand-written image codec, new downloaded dependency or
renderer redesign. Keep WIC/COM out of the Web/shared simulation build. Async encoding holds the
boundary/reservation until completion or failure; no unbounded stream/video. Raw buffers and GL state
are released/restored even on failure. A second pending capture/job rejects as busy.

## 6. Cleanup and what simulation cannot prove

Cancel, deadline, EOF/broken pipe, test.end, session.close, explicit host stop and runtime/context
failure immediately revoke device intent, queued look/pointer edges, schedules and capture work at
the next safe main-thread boundary. Watchdog expiry is30s without a valid client request (active
jobs also have their own wall deadline). Browser pagehide/visibility suspension/helper revocation
neutralizes and cancels; if its loop is suspended, neutralize before any resumed update. A crashed
external JS client has no detectable pipe EOF, so the watchdog is mandatory, not an assumed disconnect.
No additional simulated frame may run to flush inputs after cancellation; clear native latches/UI
press capture without advancing gameplay. Preserve already committed observations and partial receipts.
An explicit job cancel leaves the test controlled-idle for inspection. EOF, session close, expired
lease or revoked host grant also stops test runtime and disposes its ephemeral storage; capture
receipts remain only for the remaining explicitly live session lifetime. A disconnected client
cannot leave a hidden paused test owning the next normal user's input or profile.

Do not request or infer real Windows foreground activation, browser user activation, pointer lock,
fullscreen permission, audio autoplay unlock, physical-controller mapping or audible sound from these
commands. Runtime audio may remain suspended/muted in controlled mode without modifying preferences;
report this. Engine UI/shot/audio requests can be observed, but audible-output confirmation is separate.
The synthetic start/eligibility route is explicitly marked and cannot be cited as a host-permission PASS.
Normal pointer-lock denial, physical focus-loss/reacquisition and ordinary user controls remain intact.
Owner involvement is limited to genuine subjective judgment, required browser/OS permission or missing
physical hardware. Routine lever, movement, traversal and camera-route input is the agent's job.

## 7. Installed guidance and implementation boundary

During implementation update the maintained skill/client with begin/run/poll/cancel/capture/read/end
recipes and host limitations. Add one focused `test-input-and-capture.md` reference if necessary and
include it in the exact authoring-support inventory. Export revision13/80 Editor schema and runtime
subset availability from live definitions. No ordinary end user must install a compiler, Node/Python
or SDK. The existing installed PowerShell pipe helper and the bounded browser bridge are sufficient
for an external agent; no AI provider or scheduling framework is part of this work.

Relevant reuse points: Core GameplayInput/GameplayLookLease/PlaySession/BehaviourSystem and copied
inspection; Editor Authoring protocol/session/launch and Application Game render path; Windows Player
launch/frame input; WebPlayerApplication/shell/Emscripten exports; Preview's existing read-only grant
and lifetime; platform PNG adapters; existing progress/preferences storage interfaces. Persistence
and publishing formats do not advance. The plan owns the focused checks, actual workshop correction,
carry-forward evidence, publication hold and Tester freeze/handoff.

Platform references (implementation must use pinned local APIs, not upgrade to match latest docs):
[Emscripten JS/WASM exports](https://emscripten.org/docs/porting/connecting_cpp_and_javascript/Interacting-with-code.html),
[Windows native PNG codec](https://learn.microsoft.com/en-us/windows/win32/wic/png-format-overview),
[browser pointer-lock activation](https://developer.mozilla.org/en-US/docs/Web/API/Element/requestPointerLock).
