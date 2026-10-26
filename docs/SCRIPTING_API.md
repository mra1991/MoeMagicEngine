# Scripting API — runtime control

The v1.12.0 release supports exact **APIs 1–23**; API23 inherits APIs 1–22 and adds interaction,
projectiles and framing status. Earlier API examples remain valid. The final test report records
independent acceptance and separately attributed owner validation.

## API 23 — interaction, projectiles and camera framing

All arguments referring to objects are same-session/generation-bound handles. Reads are available
in read-capable callbacks including Start. Missing authored service records return nil; an existing
record can report unavailable with a bounded reason. Returned tables contain copied values only.

- `get_interactor_status(actor)` returns `available`, `target` or nil, `prompt`, `can_activate`,
  `revision`, `activation_revision`, `last_activated_target` or nil, and `reason`.
- `get_interactable_status(target)` returns `available`, `spent`, `cooldown_remaining`,
  `activation_revision`, `last_activator` or nil, and `reason`.
- `get_camera_framing_status(camera)` returns `available`, `zone` or nil, `distance`, `height`,
  `look_at_height`, `blend_progress`, `revision`, and `reason`.
- `request_interaction(actor)` coalesces an activation intent at the next fixed boundary.
- `set_interactable_available(target, boolean)` queues availability before selection. Repeated
  same-writer values coalesce; competing writers reject. It never resets cooldown or spent state.
- `fire_projectile(spawner, instigator_or_nil, position, direction)` atomically reserves launch
  metadata with the existing spawn ticket, returning ticket/nil-reason. Position/direction are
  finite three-number arrays; direction must be nonzero and is normalized. Read the ticket with
  the existing `get_spawn_result`. Ordinary `spawn_prefab` rejects enabled Projectile actors.

Interaction setters return `queued`, `no_effect`, `unavailable` or `capacity`, plus reason.
These three mutations require running FixedUpdate/Update/LateUpdate. Start, UI, collision/trigger
and semantic callbacks cannot mutate them. Variable-phase intent reaches the next fixed boundary;
failed-frame intent does not survive the failed session. Pause/focus/modal/Scene lifecycle clears
stale pending intentions. API1–22 do not acquire these operations or the new event kind.

API23 `gameplay_interaction` adds `kind = "projectile_hit"`: `subject` is the hit object, `other`
is the surviving optional instigator, `projectile` is the still-live impact actor, and `amount`
is actual projected damage (including zero). The copied point/feedback is captured before deferred
retirement. Existing participant deduplication applies; expiry emits no impact. Accepted activation
and launch are not proof that a project door/reward/game objective completed.

See [Action Gameplay](ACTION_GAMEPLAY.md) for exact bounds, selection/LOS, ownership, authoring,
framing and runtime-actor restrictions. Native mechanics draw no prompts and own no sample policy.

## API 22 — explicit persistent progress

`request_progress_load(slot)`, `request_progress_save(slot, schemaVersion, values)` and
`request_progress_reset(slot)` return `{accepted, requestId, error}`. Rejection has no request ID.
Admission is allowed only in Update, LateUpdate and ordinary UIButtonActivated callbacks. Start,
FixedUpdate, collision/trigger and observational UI callbacks reject these mutations. Values are
copied immediately. API1–21 do not acquire these methods.

Slots are integers 1–3. A save is a flat map of at most 128 ASCII keys, 1–64 characters from
`A–Z a–z 0–9 _ . -`, to Boolean, finite Number or valid UTF-8 String (at most 256 bytes, no NUL).
No nested tables, metatables, object handles or paths are admitted. Schema is 1–2147483647 and
the full envelope is at most 64 KiB. One admitted request is outstanding at a time; a second is
Busy. Mutations are limited to one per second per host service, with no hidden queue/retry.

`get_progress_status(slot)` returns a fresh copied `{state,lastRequestId,error,revision,
schemaVersion,values,canWrite}`. States: Disabled, NotLoaded, Loading, Ready, Missing, Saving,
Saved, Resetting, Reset, Conflict, Corrupt, Unavailable, Failed. Request admission is not durable
success: await the matching receipt and Saved/Reset. Errors include InvalidArgument, InvalidPhase,
Disabled, Busy, RateLimited, StaleObservation, Conflict, Corrupt, UnsupportedEnvelope, Unavailable,
Quota, Denied, StorageFailure and RevisionExhausted; successful requests report None.

Load first. Missing observes revision0 internally; writes compare the exact loaded observation
under platform update ownership and publish a new revision. Conflict requires a fresh Load.
Corrupt/future data is never silently Missing or overwritten. Explicit Reset requires a bounded
change-sensitive observation and writes an empty schema1 tombstone with the next valid revision.
Failed writes retain the previous valid loaded snapshot. Request IDs/revisions remain exactly
representable up to 9007199254740991; exhaustion fails instead of wrapping.

`clear_run_results()` returns `{accepted,error}` and clears only host-owned transient level results.
It has the same mutation phases. Clearing and completing a level in one frame conflict and neither
publishes; repeated clear is idempotent. It does not reset durable progress. Project New Game must
await a successful durable Reset before clearing run results and changing Scenes.

Requests stage until the successful whole-frame publication boundary; a failed callback/frame
starts no storage I/O and publishes no clear. The host service outlives Scene/Lua replacement.
Accepted storage I/O may complete after a transition or Stop; unpublished intent is discarded.
There are no Lua pointers in platform completion records.

The strict `MoeMagicSaveGame` envelope1 has exactly `format`, `version`, `gameId`, `slot`,
`revision`, `schemaVersion`, `values`; each value record has `type` (boolean/number/string) and
`value`. Windows uses Known-Folder LocalAppData with separate Player/editor namespaces, bounded
per-slot locking, flushed temporary files and atomic replacement. This is not a power-loss guarantee.
Web uses IndexedDB and reports success only at transaction completion. Origin/partition controls
storage: URL path does not. Reload of a running Preview URL retains identity; restarting its
random-port Preview server may change origin. Private mode, quota, denial, eviction or cleared
site data may prevent/remove saves. No cloud sync, cross-origin sharing or memory-only Saved claim.

The accepted v1.10.0 release supports **Lua API 21** under the
approved plan.
Exact `api_version = 21` inherits APIs 1–20. Older versions retain their released surfaces
and do not acquire the new methods. Preceding v1.9.0 is the API-20 release baseline.

v0.12.2 adds exact integer `api_version = 12`. APIs 1–11 keep their existing surfaces and
lifecycle contracts; they do not acquire the methods below. Scene 37 and prefab 31 are unchanged.
Script source resolution, copied authored properties, reference remapping, 16 MiB aggregate Lua
memory, 100,000 instructions per entry, and callback-lifetime contexts remain the authorities.
This is a bounded gameplay SDK, not reflection, filesystem access, or a service locator.

## API 21 — authored animation constraint controls

`set_animation_constraint_weight(visual, key, weight)` and
`set_animation_constraint_target(visual, key, target_or_nil)` accept only Update/LateUpdate while
simulation is enabled. Weight must be finite in [0,1]; key is an existing authored stable entry key.
Visual/target arguments are same-session, same-Scene handles. None is an explicit target override,
not a return to the authored target. Both setters return no value on success; wrong phase, handle,
key, value or capacity raises the existing script error. No definitions, poles or bones are mutable.

Writes share 64 accepted per frame and 64 outstanding through the existing animation queue, with
deterministic last-write-per-field ordering. They commit at the common animation boundary after
Update/LateUpdate, before final deformation. Reads in the submitting callback still see the last
committed pose, not queued intent. Terminal retirement wins over queued writes.

`get_animation_constraint_status(visual, key)` returns a copied table: `key`, `kind`, `state`,
`reason`, `effective_weight`, `pose_revision`, `residual`, `residual_unit`. States are `Applied`,
`Clamped`, `Disabled`, `MissingTarget`, `DegenerateTarget`, `PhysicsOwned`. Residual units are
`degrees` for LookAt and `model_distance` for IK. A cached same-generation visual handle may read
its retained PhysicsOwned status until corpse removal; it cannot write or revive the actor.
Invalid handles/keys are errors, not invented status. Pause does not advance the pose; Stop/Scene
replacement clears overrides and status with the owning PlaySession. See
[Animation Constraints](ANIMATION_CONSTRAINTS.md) for the authored configuration contract.

## API 20 — physical mechanisms and terminal status

These methods require callback-scoped same-Scene handles in FixedUpdate, Update or LateUpdate
while simulation is enabled. Start, contact/UI/gameplay-interaction events and suspended simulation
reject them. Invalid types, wrong Component/mode, stale handles and nonfinite values are errors.

| Method | Result |
| --- | --- |
| `get_constraint_status(target)` | Copied state, reason, mode, fixed_revision, optional angle_degrees/angular_speed_degrees_per_second or distance, optional force/torque, motor_enabled, motor_speed_degrees_per_second, max_motor_torque |
| `set_hinge_motor(target, enabled, speedDegrees)` | `queued`, `unavailable` or `capacity`, plus optional reason; Boolean enabled and finite speed [-720,720] |
| `break_constraint(target)` | Same result convention; terminal manual break at the next fixed boundary |
| `get_ragdoll_status(target)` | Copied state, reason, fixed_revision, simulated_age; cached same-generation handle may observe a sealed actor until destruction |

Constraint states are Inactive, Active, Broken, EndpointUnavailable. Ragdoll states are Inactive,
Ready, Pending, Simulating, Finished. Unavailable measurements are nil, not invented zeros. Reads
do not advance simulation or expose queued commands. Motor speed/enable commands preserve the
authored torque cap. Same-writer commands commit in order; a break cannot be undone by a later
motor write. Distinct writers cannot control one pending connection. Controls share 64 accepted
writes per rendered frame across catch-up ticks and at most 64 outstanding writes. Endpoint
destruction wins. Broken/terminal state clears only on restart/Scene replacement, never healing.

Terminal actors are renderable but unavailable for gameplay/motion/animation commands; their
ordinary handles cannot resurrect them. The status-only exception is not a new writable body API.
See [Physical Mechanisms](PHYSICAL_MECHANISMS.md) for units, authoring, corpse filtering and ownership.

## API 19 — bounded navigation and observations

These six methods require callback-scoped same-Scene opaque handles and **FixedUpdate, Update or
LateUpdate** while simulation is enabled. Start, events and suspended simulation cannot use them.

| Method | Result / ownership |
| --- | --- |
| `set_navigation_destination(target, {x,y,z})` | Queue a finite world-space destination as the existing indexed three-number vector |
| `stop_navigation(target)` | Queue route/destination clearing |
| `get_navigation_status(target)` | Copied state, reason, optional destination, actual_speed, fixed_revision |
| `get_perception_status(target)` | Copied visible_target, last_seen_target, last_seen_position, last_seen_age, observation_time, observation_revision |
| `set_agent_controller_enabled(target, boolean)` | Queue destination-policy ownership change, not Component removal |
| `get_agent_state_status(target)` | Copied state, home, one-based patrol_index, selected_target, fixed_revision |

Setters share 64 accepted submissions per rendered frame and the existing queued/unavailable/capacity
result convention. Wrong phase/type/handle/Component, nonfinite destination or competing destination
ownership is a script error. A controller owns destination policy while enabled. Queuing disable,
then a destination, in the same callback is supported; reads still return the last committed state.
Writes apply at the next fixed boundary, never immediately. Re-enable starts fresh Patrol while
retaining that Play session's original grounded home. Disable never transfers physical pose
authority: NavAgent still owns the Kinematic target, excluding direct/script/native competing writers.

Navigation states are `Idle`, `Pending`, `Moving`, `Arrived`, `Blocked`, `Unreachable` and
`InvalidNavigation`; controller states are `Disabled`, `Patrol`, `Pursue`, `Return`. Arrival and speed
come from achieved physics, not route intent. New destinations coalesce without delaying their
eligible query; Pending stops the old route. Observations are atomic completed samples, not live
target lookups. Only a new visible sample refreshes last-seen position/age. The resolved targetOffset
point is also the pursued point: author it near navigable ground when targets have elevated origins.
Ineligible targets clear visible selection; Return cannot reacquire before reaching home.

Pause freezes navigation/sensing/controller state; Step advances one fixed step. Stop/new Play and
Scene replacement retire all queues/status/handles. See [navigation authoring and limits](NAVIGATION.md).

## API 18 — authored animation states and committed presentation markers

These five callback-scoped methods use same-Scene live handles in **Update/LateUpdate only**:

| Method | Contract |
| --- | --- |
| `set_animation_bool(target, key, boolean)` | Queue a declared Bool value |
| `set_animation_number(target, key, number)` | Queue a declared finite Number value |
| `trigger_animation(target, key)` | Queue a declared Trigger pulse |
| `get_animation_state_status(target)` | Copied state_key, clip_name, playing, completed, crossfading, progress (0–1) |
| `get_animation_events(target)` | Copied ordered array of target, state_key, marker_key, clip_name, positive sequence |

Targets require enabled same-object Animator and Animation State Controller. Stale/wrong handles,
wrong phases/types/keys, nonfinite values and capacity are contract errors. Live but ineligible
targets return nil from the queries. Setters return no value and share the existing 256-command
frame queue. Writes apply in submission order; repeated Trigger pulses coalesce. The first matching
authored outgoing transition wins, at most one per positive animation update. All pending pulses
are cleared after that eligible decision, including unused pulses. Nonloop completion becomes
eligible on the following update. Zero-delta refresh neither decides nor consumes triggers.

State change resets the destination clip; interrupted crossfades capture the currently blended
pose. Controller ownership disables direct Play/CrossFade/Stop and conflicts with Character
Animation's resolved Animator. There is still one AnimationWorld clock/pose authority.

Markers cross `(previousTime,newTime]`; entry-time zero emits on the first positive update.
At loops, end precedes next zero; equal-time entries follow authored order. Crossfades emit only
destination markers. State, pose and at most 256 events commit atomically; overflow rejects the
candidate without partial publication. Sequences are monotonic within the Play session. Stop/new
Scene clears them. Repeated queries do not drain events: deduplicate by sequence in Lua. Read the
last successful batch in Update/LateUpdate, never use markers as authoritative damage/collision
or objective policy. Zero-delta actor admission does not replay or erase that batch.

The shipped `workshop_attendant.lua` is a small example: route/proximity remain project policy;
a committed `hello` marker plays existing audio once per sequence. No root-motion, retargeting,
blend tree, runtime controller creation or seek/playhead API is introduced.

## API 17 — bounded arcade vehicle controls

The three new callback-scoped methods use existing same-Scene generation-bound handles:

| Method | Contract |
| --- | --- |
| `get_vehicle_status(vehicle)` | Copied `available`, `control_enabled`, `speed`, `forward_speed`, `grounded_wheels`, `steering_degrees`, `throttle`, `brake`, optional unavailable/recovery `reason` |
| `set_vehicle_input(vehicle, throttle, steering, brake)` | FixedUpdate only; finite throttle/steering [-1,1], brake [0,1]; queues one tick's demand |
| `set_vehicle_control_enabled(vehicle, boolean)` | Start or running gameplay callbacks, including semantic GameplayInteraction; commits at next fixed boundary |

Status starts coherently at zero speed/contacts before the first step. Speed is magnitude in m/s;
forward speed is signed along chassis -Z; grounded wheel count is 0–4. It exposes no native body,
constraint, resource or mutable Component. Unavailable registered configuration returns a copied
unavailable table/reason; stale or nonvehicle handles are contract errors.

Drive admits at most 32 submissions per fixed tick. One writer may replace its request; competing
writers reject, including a Script on the same owner as an enabled native Vehicle Input. Native
input reserves the slot even when neutral. Accepted drive returns `queued, nil`; invalid range,
phase, producer or exhausted drive budget is a contract error. Missing input resets demand every
fixed tick, not velocity/gravity. Opposite throttle first brakes before reverse propulsion; speed
ceilings limit propulsion, not externally imparted motion.

Control returns queued/unavailable/capacity with the existing optional reason. Its independent
limits are 64 accepted requests per frame and 64 outstanding across zero-fixed frames. Last request
from the same writer wins; conflicting writers reject without replacing it. Disabled control uses
neutral demand and service brake. It never removes the Component, clears recovery suspension or
authorizes dead actors. UI callbacks and suspended simulation cannot mutate vehicle control.

API 17 extends `request_respawn` / `get_respawn_status` to vehicles, preserving their existing phases,
delay/checkpoint/fallback/retry and revision contracts. API 15/16 remain Character-only: vehicle
query returns nil and request returns unavailable. Recovery removes vehicle contact participation
while pending/blocked, checks the actual chassis shape/filter, and commits Health with an upright
zero-motion reset at the safe fixed boundary. Drivetrain, steering, spin and contact caches reset;
FollowCamera snaps through the existing recovery revision. Course progress remains project Lua.
No unrelated presentation, prefab or progression budget is enlarged.

## API 16 — path, follow Camera and presentation controls

All methods use callback-scoped `context:` and same-Scene generation-bound handles.

| Method | Copied result / operation |
| --- | --- |
| `get_path_status(path)` | state playing/paused/completed/blocked, distance, length, progress, direction, speed, endpoint_revision, reset_revision, optional reason |
| `set_path_playing(path, boolean)` | Pause/continue without rewinding |
| `set_path_speed(path, number)` | Finite 0–50 |
| `reverse_path(path)` | Reverse direction without teleporting |
| `reset_path(path, {progress=0, direction=1, playing=false})` | Explicit next-fixed-boundary reset; direction is -1/+1, progress 0–1 (Loop excludes 1) |
| `get_follow_camera_status(camera)` | available, mode orbit/trailing/fixed_heading, heading_degrees, target, revision, optional reason |
| `snap_follow_camera(camera)` | Snap current desired pose without changing authored settings |
| `set_ui_opacity(target, alpha)` | Finite 0–1; preserve RGB on existing Image/Text/Panel |
| `set_mesh_tint(target, {r=1,g=1,b=1})` | Each finite 0–1; per-renderer only, never shared Material |
| `set_light_intensity(target, intensity)` | Finite 0–10; preserve kind/color/range/shadow policy |

Motion queries are available in Start/UI. Path mutation is available in Start and ordinary running
gameplay callbacks; not UI, semantic-interaction or suspended simulation. Snap uses ordinary
running callbacks, not Start. Mutation returns queued/no_effect/unavailable/capacity plus a bounded
reason as applicable. Wrong arguments, stale handles and wrong phases are contract errors.
Path submission and outstanding queues are each capped at 256; the existing Kinematic target cap
is unchanged. One submitting owner has path control authority within a frame.

A physical reset checks the actual shape/filter against response bodies and Characters, ignores
its own body and nonresponse triggers, and permits resting floor contact. Clear reset zeroes
velocity and refreshes contact/ground history. Blocked reset retains pose/distance, stops the path
and waits for explicit retry; it never searches for an alternate location.

Opacity/tint/intensity use the existing presentation phases, including Start and suspended UI,
and share its 256-command atomic transaction with text/visibility. Last value wins per field.
No partial publication occurs after a failed batch. These setters do not edit resources or Scene
authoring state. Camera position damping runs after LateUpdate; startup/recovery/snap bypass it.
Camera loss freezes pose and reports unavailable; movement never derives from damped presentation.

## API 15 — recovery, objectives, rewards and run results

All calls are callback-scoped `context:` methods. Queries return copied tables or nil when an
applicable enabled service/result is unavailable; handles remain same-Scene, generation-bound.

| Method | Result |
| --- | --- |
| `get_respawn_status(character)` | `state` (alive/pending/blocked), `checkpoint` handle or nil, `checkpoint_revision`, `respawn_revision`, `respawn_position` {x,y,z}, `respawn_yaw_degrees`, optional `reason` |
| `request_respawn(character)` | Queue recovery or explicit blocked retry |
| `get_objective_status(counter)` | `completed`, fixed `total`, `complete`, `unresolved_missing`, `revision` |
| `complete_objective_member(counter, member)` | Resolve one configured Script member once |
| `get_conditional_reward_status(controller)` | `state` (locked/requested/spawned/claimed/failed/lost), `actor` or nil, `ticket` or nil, `revision`, optional `reason` |
| `claim_conditional_reward(actor)` | Claim an admitted reward root or descendant before its destruction |
| `retry_conditional_reward(controller)` | Retry failed only; lost is not retryable |
| `complete_level({score = integer, awards = {...}})` | Prepare one completion for this declared Scene attempt |
| `get_level_result(scenePath)` | `completed`, `last_score`, `best_score`, sorted unique union `awards` |
| `get_last_level_result()` | `scene_path`, `score`, `awards` from the most recent completion |

Mutation returns `result, reason`: queued/no_effect with nil reason, or unavailable/capacity with
a bounded reason. Invalid arguments, stale/foreign handles or wrong phases are contract errors,
not normal capacity failures. Queries are allowed in Start, UI and API-14 interaction callbacks.
Mutations require running simulation in FixedUpdate/Update/LateUpdate or trigger/collision callbacks;
`complete_level` is restricted further to Update/LateUpdate. Start, UI, semantic interaction and
suspended simulation cannot mutate progression.

Recovery observes final projected Health after LateUpdate; lethal damage defeats same-frame
checkpoint activation. Delay advances at later fixed boundaries. Actual Character shape/filter
clearance tries the captured latest pose, then initial pose once; blocked retains pose and Health
until explicit retry. A destroyed checkpoint leaves its captured pose valid but its handle nil.
Recovery resets physical motion/contact/motor/animation state without replaying Start; the next
LateUpdate camera follows the restored pose. It does not rewind the world, heal unrelated objects,
grant invulnerability or search for alternative ground.

Objective queries expose committed state. Script completion and accepted native Break completion
are prepared before destruction; disappearance alone never succeeds or shrinks the denominator.
Shared members may count for several counters; dynamic spawns do not join. Completed counters
make AND rewards eligible on the next normal frame. Existing prefab admission occurs at frame end,
then actor Start occurs next frame. Claim before Destroy wins, repeat live claim is no_effect,
unclaimed root destruction is lost, and retiring the controller does not destroy the actor.

Budgets: 16 recovery Characters, 64 checkpoints, 16 counters, 256 members each/1,024 total,
16 rewards, 1–4 required counters, 256 checkpoint candidates and 256 combined completions per
frame. Recovery requests coalesce per Character; claim/retry work coalesces per reward. Reasons
are at most 512 bytes. Existing prefab capacity still applies; requests are not successful admission.

The host owns one run store, never a global or file save. Up to 32 declared Scene results retain
last/best scores (integral [-1,000,000,1,000,000]) and at most eight unique awards. Keys use 1–64
ASCII letters/digits/underscore/dot/hyphen; overflowing the accumulated union rejects without
mutation or eviction. Identical same-frame completion coalesces; conflicting completion preserves
the first request; later completion in the same attempt has no effect. Publication follows all
successful frame work and precedes host transition consumption, so failed destination startup
does not erase a valid completion. Replay creates fresh Scene-local progress but retains this
run's results. Stop, exit or a new run clears them. Unsaved/undeclared Scenes have no result identity;
ordinary untitled Play still works. Direct-entry Results should handle nil.

## API 14 — semantic gameplay interactions (v1.3.0 released)

Exact `api_version = 14` inherits API 13 and adds `gameplay_interaction(self, context, event)`.
APIs 1–13 retain their existing methods/dispatch; they do not receive this callback. It reports
completed native outcomes, not collision manifolds or an invitation to apply the damage again:

| Copied field | Meaning |
| --- | --- |
| `kind` | `stomp`, `contact_damage`, or `broken` |
| `subject` | Same-session handle for the affected target/broken object |
| `other` | Originating Character/damage source, or nil when no live origin exists |
| `amount` | Actual clamped damage, or zero for `broken` |
| `world_position` | Copied `{x, y, z}` table |
| `reward_ticket`, `reward_reason` | Optional ordinary prefab ticket or bounded rejection reason (at most 512 bytes); never a promise that insertion already succeeded |

Only eligible, Started Script Behaviours on the distinct subject/origin participants receive the
event, in stable-ID order; there is no global broadcast. At most 512 events per rendered frame
and two participant receivers per event. Stomps/Enter damage dispatch after complete fixed-step
arbitration and before collision callbacks. Breaks dispatch after LateUpdate/final projected Health,
before deferred source destruction. Explicit destruction and expiry are not break events.

This phase permits copied authored properties, ordinary safe handle reads, committed/projected
Health reads, spawn-result reads, Canvas UI feedback, one-shot audio, particles and ordinary prefab
requests while simulation runs. It exposes no input or delta time and rejects Health mutation,
Transform writes, destruction, Kinematic targets, simulation/Scene-transition requests and recursive
event dispatch. Keep policy reactions here, and do not duplicate native damage/break mechanics.
Store a reward ticket only when needed, then use ordinary result reads in a later gameplay phase.
See [Scene component configuration](SCENE_FORMAT.md) and the manual.

## API 13 — bounded runtime prefabs

Released v1.2.0 adds exact `api_version = 13`, inheriting API 12 and adding only:

```lua
ticket, reason = context:spawn_prefab(spawner, x, y, z, rx, ry, rz)
status, object, reason = context:get_spawn_result(ticket)
```

`spawner` is an ordinary same-session object handle with an authored, enabled Prefab Spawner.
Position and Euler-degree rotation are six finite numbers; scale remains authored. No paths,
property overrides or parent argument are accepted. Requests are allowed in FixedUpdate, Update,
LateUpdate and trigger/collision callbacks while simulation runs, never Start or UI callbacks.
Result reads are allowed in gameplay callbacks including Start, never UI. API 1–12 surfaces are
unchanged; they do not acquire these methods. Use a v1.2.0 binary for API 13.

Accepted calls return an opaque session ticket and nil reason; normal unavailability/capacity
returns nil and a bounded explanation. Malformed arguments, wrong phases, stale contexts and
foreign/stale tickets are contract errors. Results are `pending, nil, nil`,
`spawned, rootHandle, nil`, or `rejected, nil, reason`. A spawned result stays spawned after
destruction; its ordinary handle becomes invalid. Dropping a pending ticket does not cancel it;
garbage collection frees retained result slots. Do not accumulate tickets indefinitely.

Bounds: 32 requests/frame, 256 live spawned roots, 1,024 spawned objects, 1–128 roots per source
(authored default 16), 256 retained results, plus existing stricter subsystem limits. Reservations
include pending requests; destruction frees capacity only after commit. No lifetime spawn quota.
Accepted requests survive later source destruction; already-pending sources reject. Complete
actors publish after callbacks/destruction; they first run Start on the next scheduled frame.
Stop, failed session/Scene replacement and teardown retire session-local actors/results. Pause
queues nothing; Step performs one ordinary transaction. No runtime files, GPU creation or Edit writes.
Source limits/allowlist are in [PREFAB_FORMAT.md](PREFAB_FORMAT.md#runtime-actor-sources-v120).

## Handles and configuration

The five `get_object_*_property(target, name)` methods read the target ScriptBehaviour's copied
authored Boolean (`bool`), Number (`number`), String (`string`), Vector3 (`vector3`), or optional
GameObject (`game_object`) property. Vector3 returns three numbers, not a table. GameObject returns
nil or the existing opaque same-session handle. Reads do not depend on the target's Start callback.
Missing/wrong-kind properties and foreign, stale, destroyed or pending handles reject actionably.
There is no raw ID, property mutation, arbitrary Component access, or Edit writeback.

## API 12 operations

| Method | Result / authority |
| --- | --- |
| `set_kinematic_target(target, {x,y,z}, {rx,ry,rz})` | `queued`; world/root position and Euler degrees, finite and bounded; existing CollisionWorld target transaction. Scale unchanged. |
| `get_health_status(target)` | Copied `{current, maximum, depleted, revision, projected=false}` from committed HealthWorld. |
| `get_pending_health_status(target)` | Same shape with `projected=true`, evaluating already queued ordered commands through the commit implementation; revision still identifies the committed base. |
| `damage(target, amount)`, `heal(target, amount)` | `queued`, `no_effect`, or `capacity`. Amount is integral 1..1,000,000. Clamp and no-resurrection rules remain native. Full healing and depleted targets return `no_effect`. |
| `emit_particles(target, count?)` | `accepted`, `already_queued`, or `capacity`; omitted count uses authored count, override integral 1..64. Invalid source/arguments are errors, not capacity. |
| `set_ui_text(target, text)` | `queued` / `capacity`; existing UIText, 1..128 bytes using letters, digits, space, colon, hyphen and period. No newline or exclamation glyph. |
| `set_ui_visible(target, visible)` | `queued` / `capacity`; existing Canvas or Text/Image/Panel visual. It does not change GameObject activity, physics or resources. |
| `set_simulation_enabled(enabled)` | `queued`; committed at the successful frame boundary. No native score, result or terminal reason. |
| `request_scene_transition(identity)` | `accepted`, `already_requested`, or `conflict`; malformed identities error. Host declaration/containment/resource checks still precede transactional switch. |

Ordinary target/phase errors fail the script entry through the existing protected callback policy;
Editor discards the runtime session without damaging Edit, while Player follows its nonzero error
policy. A projected health read is not a committed effect. A failed later transaction never turns it
into published state. UI changes preflight the whole bounded list before publication.

## Phase and capacity boundaries

| Capability | Allowed boundary |
| --- | --- |
| Copied properties / health observations | Valid callback context and live target. |
| UI text / visibility | Start and later valid callbacks, including suspended UI. |
| Kinematic target | FixedUpdate only; live active enabled Kinematic root + Collider, no competing native KinematicMotionBehaviour. One script writer per target per tick. |
| Damage / healing / new particle bursts | Gameplay callbacks after Start while simulation runs; not Button or feedback callbacks. |
| Simulation / transition request | After Start; UI remains available while simulation is suspended. |
| `ui_interaction` | Shared semantic UI dispatch outside controller/native enumeration; no input, delta, physics or health-writing capability. |

Kinematic targets: at most 128 per tick; repeated same-writer requests replace that target, different
writers reject. The native batch preflights before MoveKinematic and writeback. No request holds the
current pose rather than replaying previous velocity. Raw Transform writes to physics owners remain
invalid. Health commands retain their existing 256-command bound. UI publication accepts at most 256
commands per frame. Feedback dispatch accepts at most 64 events per frame, scoped to the runtime.

Particles retain 128 authored enabled sources, 128 commands per frame and 512 live particles. A
source can emit again next frame, but a second same-source request this frame is `already_queued`.
Admission snapshots the source's configuration and world position before deferred destruction and
reserves a whole burst against live occupancy. Optional feedback capacity must not erase gameplay
collection. No continuous-emitter, GPU-simulation or arbitrary particle-resource API is added.

API 12 also rejects inherited Transform/destruction mutations in Start, observational feedback,
or suspended UI. This does not change the Start contract of APIs 1–11.

Simulation suspension stops gameplay callbacks, fixed physics and gameplay timers. UI and existing
particle draining remain responsive. Both hosts neutralize gameplay input, and fresh acquisition
prevents held-edge replay. Editor Pause is separate; Step cannot bypass the game's gate. Stop or a
new Play/transition creates fresh runtime values and restores the untouched authored presentation.

## Shared UI feedback

An API 12 ScriptBehaviour on the owning Canvas may define:

```lua
ui_interaction = function(self, context, event)
    if event.kind == "selection" then
        context:play_audio_one_shot(self.selection_sound)
    end
end
```

`event` contains `surface` (`canvas`, `controls`, `graphics`), `kind` (`selection`, `activation`,
`cancel`, `confirmation`, `error`), a stable `action` token, and `source` (authored Button handle or
nil for generated controls). Generated IDs never become fake Scene handles. Opening a modal captures
the originating Canvas and runtime generation; unscoped host opens are silent. Stop/transition
invalidates old scope. Programmatic teardown is not a user Cancel.

Mouse entry and keyboard/gamepad focus movement share selection semantics. Stationary hover,
initial focus, recomposition and neutral reacquisition are silent. Accepted press/release activates
once. Close/Escape/Back use Cancel; capture cancellation has `CancelCapture`, distinct from closing.
Rebind reports actual capture success/error. Graphics Apply reports activation followed by actual
host-acknowledged confirmation/error, never success merely because Apply was pressed. A project
may sound only confirmation for Apply/Rebind to avoid duplicate feedback. Handlers cannot veto or
recursively advance a native modal. Quit may cut its feedback clip short; exit is not delayed.

## Small project-owned examples

```lua
-- FixedUpdate: the project computes/clamps/reverses its route; physics owns the pose.
context:set_kinematic_target(self.platform, {x,y,z}, {rx,ry,rz})

-- Gameplay callback: optional whole-burst feedback is not collection authority.
local effect = context:emit_particles(self.pickup)
context:destroy_object(self.pickup)

-- LateUpdate: all contact policy has already queued health changes.
local health = context:get_pending_health_status(self.player)
if health.depleted then
    context:set_ui_text(self.result_text, "RUN ENDED")
    context:set_ui_visible(self.result_canvas, true)
    context:set_simulation_enabled(false)
end
```

`Projects/PotPursuit/assets/scripts/game_controller.lua` owns food/utensil categories, signed score,
expiry and food-only completion. Ultimate's corresponding controller owns coins, healing, damage,
hearts and defeat-before-completion policy. Ultimate's `kinematic_route.lua` and `repeated_sparks.lua`
show authored route/effect changes without C++. `ui_feedback.lua` chooses project Audio Sources.

The deprecated collection Components/System and native AudioButton/KinematicMotion remain for old
content. They are not current sample policy owners. Inspector **Planar Motor** retains the serialized
`PotPlayerBehaviour` token and mechanics. Removing this compatibility needs a separately approved
migration. See the implementation report for
evidence and the testing policy for role ownership.
