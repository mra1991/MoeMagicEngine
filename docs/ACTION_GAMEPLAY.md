# Composable action gameplay

v1.12 adds four authored Components and Lua API23. These are reusable mechanics, not native
lever, door, cannon, quest or inventory policy. Scene47 / ordinary prefab41 store authored values;
variant1, Input Actions3 and the other v1.11 format authorities are unchanged.

## Interaction

An enabled **Interactor** is a finite, positive-uniform Scene root. Its named Button action
(`Interact` by default) must resolve before Play/Build. Range defaults to 2.5 metres [0.1,10],
owner-local origin offset to (0,1,0), query layer to 0 [0,15], and occlusion mask to 65535.
Offset components are bounded to [-10,10]. At most eight records, including disabled drafts.

An **Interactable** supplies prompt `Interact` (1–64 bytes in the existing Canvas alphabet),
local interaction offset (0,0,0), initially available true, line of sight true, one-shot false
and cooldown 0.25 seconds [0,60]. It needs a finite nonsheared world pose, but no Collider.
At most 64 records, including disabled drafts. Both Components start disabled when added.

The fixed boundary selects by world distance, then stable ID. Inactive, unavailable, spent,
pending/terminal or depleted-Health targets are excluded; recovery-suspended actors cannot act.
Cooling targets stay focused. LOS ignores triggers and the actor's own root body; reaching the
target or its descendants is visible, as are coincident points without a raycast. There is no
facing cone or screen reticle. Native press edges and Lua requests coalesce once per actor/tick.
Stable actor order arbitrates competing one-shot claims. Cooldowns pause with simulation.

API23 exposes copied `get_interactor_status` and `get_interactable_status` values, including
focus/activation revisions and safe optional last-participant handles. Project Lua consumes those
revisions to animate a lever, move a gate or request a reward. Project Canvas owns the prompt.
`request_interaction` and `set_interactable_available` are allowed only in running
FixedUpdate/Update/LateUpdate, returning result/reason. Availability changes precede selection,
do not reset spent/cooldown state, and reject competing writers. Limits: eight pending actor
intents, 64 distinct availability changes and 48 accepted activations per rendered frame.
Focus loss, modal suspension, Pause and session replacement discard stale pending intentions.

## Prepared projectiles

**Projectile** is a disabled-by-default, unit-scale prepared-actor root, not an ordinary placed
moving Scene object. The actor has no Collider, Rigid Body or Character; the root has no competing
motion controller. A visual child can scale an existing mesh independently. Only this new type
joins the runtime-prefab allowlist; Interactor, Interactable and framing zones remain forbidden
runtime actors even when disabled.

Defaults/ranges: speed 10 [0.1,50] metres/second; lifetime 3 [1/60,10] seconds; full sweep Box size
(0.3,0.3,0.3), each [0.02,2] metres; damage 1 [1,1000000]; query layer 0 [0,15]; hit/damage-target
masks 65535 [0,65535]. Optional same-root impact Audio/Particle feedback defaults false and needs
the corresponding enabled source. At most 32 live/reserved projectiles within existing shared
prefab limits. Disabled authored records also count toward the 32-record bound.

`fire_projectile(spawner, instigator_or_nil, position, direction)` accepts numeric-array Vector3s.
A finite nonzero direction is normalized; local -Z is launch forward. Launch metadata and the
ordinary spawn ticket reserve atomically. Ordinary `spawn_prefab` does not launch a projectile.
The optional instigator requires a live enabled root Collider and is ignored by the sweep.
Later source/instigator destruction does not cancel an accepted shot.

PlaySession admits the prepared actor at frame end. Start and movement occur on the next frame.
Each fixed step sweeps the full bounded segment against the committed physics-query snapshot,
excluding triggers. The nearest hit wins, including initial overlap; a final lifetime segment
can hit before expiry. The projectile latches terminal once, projects damage through existing
Health commands, captures optional feedback, emits API23 `projectile_hit`, then queues retirement.
Expiry has no impact event. Source, hit and projectile identities are copied, checked handles;
older APIs do not receive the new event. This is neither a new rigid-body system nor a direct
native-physics API. Scripts cannot overwrite the managed root pose.

## Camera framing zones

A **Camera Framing Zone** starts disabled and uses its enabled same-object Box trigger, no Rigid
Body, as an oriented volume. It requires a stationary root and an enabled same-Scene FollowCamera
reference before use. None remains a valid disabled draft. At most 32 zones and eight FollowCameras.

Defaults/ranges: priority 0 [-100,100]; distance 8 [0.1,100] metres; height 3.4 [-50,50] metres;
look-at height 1 [-10,10] metres; blend 0.35 [0,2] seconds; exit margin 0.25 [0,2] metres.
The FollowCamera target's world point selects highest priority then lowest stable ID. Only the
previous winner receives exit-margin hysteresis. Smoothstep blends from current effective framing
to the winner; leaving blends to captured authored base using the departing duration. Zero snaps.
Startup, explicit camera snap and recovery immediately select the applicable framing.

The existing FollowCamera service still publishes exactly one pose and owns damping/heading.
Zones never change mode, target, FOV, Orbit responsiveness or Character movement basis.
`get_camera_framing_status` reports copied zone, effective values, progress, revision and reason.
Unavailable targets retain the existing last-valid-pose behavior. Disable/delete returns to base
or another winner; Stop clears all transient blend state.

## Authoring and portability

The Inspector edits copied values through the same typed configuration/history operations used by
other editor callers. Clone, ordinary prefab capture, Delete/Undo/Redo and fresh-ID restoration
preserve/remap framing references. Safe scalar/vector properties use the existing typed variant
whitelist; object references are not variant overrides. Historical formats load without implicit
new Components. Runtime focus, cooldown, launch, impact and blend state never enter Edit data.

Windows and Web share these services and the existing declared project-content closure. Selected
media and notices are project-owned copies; Players never mount the shared library. See the
implementation report for current verification,
not a claim of independent acceptance. See pre-AI readiness for
composition limits and the deliberately partial descriptor inventory.
