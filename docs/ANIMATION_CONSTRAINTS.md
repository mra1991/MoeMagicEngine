# Authored animation constraints

v1.10.0 introduced bounded LookAt and TwoBoneIK presentation on the existing rigged Model/Animator.
It does not change gameplay colliders, Character motion, imported assets or the four-influence
import limit. Windows and Web use the same implementation. The feature was introduced with
Scene 46, prefab 40 and Lua API 21; current v1.12.0 writes Scene 47/prefab 41 and supports API 23.

## Authoring

Add Animation Constraint to an object with MeshRenderer and Animator. It starts disabled.
Bind the current rig signature, add an explicitly named LookAt or TwoBoneIK entry, and choose
qualified joint paths from the imported rig. No bone name is guessed. Use ordinary Scene objects
as targets/poles, with target-local offsets. Configure the draft, then enable it. A missing target
is valid but unavailable; an invalid enabled rig/path/topology/transform is an admission error.

LookAt rotates one bone toward a target using its explicit bone-local aim axis (+Z default).
It preserves sampled roll through shortest-arc swing and limits the change from the current
sampled/blended pose (60 degrees default). Antiparallel fallback uses the sampled basis, not
previous-frame state. Coincident targets preserve the base pose with DegenerateTarget status.

TwoBoneIK needs a distinct direct-parent root/mid/tip chain and a pole defining the bend side.
It preserves current segment lengths, root translation and scales. Reach and bend limits clamp;
there is no stretching. Zero degrees is straight; the solver stays a scale-relative epsilon away
from singular inner/outer reach. Collinear poles or coincident goals preserve the base pose.
Tip orientation is optional: its weight multiplies the entry weight and uses the ordinary target
rotation plus Rz*Ry*Rx offset. All blending changes local rotations only.

The visual and participating chain require positive uniform nonsheared frames, including relevant
animated scale channels. Target frames may have positive nonuniform scale but cannot be singular,
reflected or sheared. Offsets use the full matrix. Enabled writer chains may not overlap or depend
on an ancestor written by another entry; separate head/arm branches can share an unwritten torso.
The whole IK chain participates in conflict checking even with zero tip-orientation weight.

Each visual permits eight entries, each Scene 64 including disabled drafts. Unique ASCII keys
are 1–64 bytes. Rename does not rewrite Lua source. The Inspector and
`EditorObjectOperations::ConfigureAnimationConstraints` submit complete copied configurations
through ordinary validation/history. Add/remove/reorder is atomic; continuous changes use one
undo command. Copied inspection supplies typed descriptors/defaults/ranges/units/dependencies.
Selected Scene guides show rest chains when stopped and committed poses during Play. They do not
enter picking, history, persistence, Game view or Player. Play status is copied and read-only.

## Ownership and lifecycle

Clip sampling and state/Character/direct crossfade ownership are unchanged. AnimationWorld applies
constraints after blending, using one ordinary-Transform target snapshot at the common end-frame
boundary. A candidate then composes the single palette, normal matrices, CPU deformation, bounds
and picking data. No render-only pose override or second animation service exists. Failed finite
math/preparation cannot publish a partial candidate. The imported shared Model remains immutable.

Initial Play solves authored values. Runtime API 21 weight/target overrides are transient and
deferred; explicit nil remains nil. Disabled/zero-weight entries preserve base pose. Deleted,
inactive, pending-destruction or terminal targets produce MissingTarget while animation continues.
Pause/Step, neutral admission and Scene replacement use the existing PlaySession boundaries.

Terminal ragdolls capture the last committed constrained pose. Physics then owns the only final
pose; constraints cannot overwrite or revive it. Retained status reports PhysicsOwned until
cleanup. Stop/restart/replacement creates fresh clocks, overrides and statuses. Ragdoll lifetime,
damage, contact, stomp/bounce and objective policy are unchanged.

Strict persistence stores authored values only. Clone/prefab/history remap internal target/pole
references and deletion repairs None. Outside-subtree references retain the ordinary rejection
policy. Explicit reimport preflights affected enabled bindings before generation publication;
saved Windows/Web closure performs the same rig/chain/transform validation. Runtime actor
templates reject Animation Constraint even disabled; unconstrained animated actors still work.

Full-body IK, terrain feet, kart hands, arbitrary bone scripting, ragdoll recovery and runtime
constraint spawning are not part of this release. Final owner acceptance concerns the restrained
attendant head attention and sentinel arm-ready-to-ragdoll appearance, not a new input test matrix.
