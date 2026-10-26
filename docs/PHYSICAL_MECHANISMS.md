# Physical mechanisms and terminal ragdolls

v1.9.0 adds Physics Constraint and Ragdoll as ordinary authored Components on Windows and Web.
These mechanics were introduced with Scene45/prefab39/API20; v1.12.0 writes Scene47/prefab41
and supports APIs 1–23 without changing the mechanism contracts below. Navigation
asset 1 and distribution 7 are unchanged. Independent release acceptance is separate from this
contract; exact implementation evidence is in the v1.9.0 implementation report.

## Ownership and authoring

CollisionWorld owns the single pinned Jolt world, constraints, connected-pair filters and private
ragdoll bodies/joints. PlaySession owns bounded command/status and terminal-lifetime coordinators.
AnimationWorld remains the only final skin-palette, normal and CPU bounds publisher. No native
body IDs, resources or live solver pointers are exposed to Inspector or Lua.

In Inspector, add a Physics Constraint to a helper object, choose Hinge or Distance Spring, and
assign Body A. Body B=None explicitly means World. The helper's Transform does not move anchors.
Endpoints need enabled ordinary solid Collider/Rigid Body Components, at least one Dynamic;
Character, Vehicle and Ragdoll endpoints are excluded. Author anchors relative to each object
origin (not its collider centre/COM); a World B anchor is world-space. Frames use Transform Euler
degrees, local +X as hinge axis and +Y as reference. Selected Scene view shows anchors, axes and
hinge limits; Game/Player views never show authoring lines.

Hinge setup must agree within 0.01 m / one degree and start inside its limits. Distance endpoints
must start within their min/max range (0.01 m tolerance). Invalid execution reports the object and
reason instead of allowing a large initial correction. None bindings remain saveable drafts.
Enable Play or Build only with complete eligible bindings. Saved Build uses the same preparation
rules against packaged rig data, not an unsaved editor cache.

| Setting | Default / bound |
| --- | --- |
| Hinge limits | Enabled; -45 / +45 degrees; minimum [-180,0], maximum [0,180], min < max |
| Hinge friction / motor | 0 / disabled; speed 0 in [-720,720] degrees/s; maximum torque 100 N m; torque values [0,1,000,000] |
| Distance | Min=max=1 m, each [0.01,100]; frequency 2 Hz [0,20], damping 1 [0,1]; zero frequency is rigid |
| Break force / torque | 0 disables; otherwise [0,10,000,000] N / N m; Distance Spring requires torque=0 |
| Frames | Anchors within +/-100,000; finite wrapped Euler angles; supported nonsingular, nonsheared body transforms |
| Capacity | 64 authored constraints including disabled drafts; 64 accepted API writes/frame and 64 outstanding |

Connected collision is off by default and pair suppression is reference-counted. Enabling it never
bypasses ordinary symmetric masks. Solver reaction measurements use completed linear/angular impulse
magnitudes divided by fixed dt; hinge angular reaction includes limit and motor contributions.
Strictly exceeding an enabled threshold retires the connection after the solve. Explicit break
uses the same terminal path without inventing a force sample. Broken persists until restart.
Disable/reactivate revalidates current frames; missing endpoints publish EndpointUnavailable.
Deleting authored Body B clears it **and disables the constraint**, preventing an accidental World
connection. Undo restores its prior binding/enabled state through fresh-ID history mapping.

## Ragdoll mapping and terminal lifetime

Start with a root solid Kinematic/Dynamic body with Health and Breakable. Add Ragdoll, select an
enabled imported MeshRenderer/Animator in its subtree, then explicitly bind the copied rig
signature. Add 2–12 parts using the exact qualified bone choices, stable unique ASCII keys and
one physical root. Each parent must map to an ancestor bone; skipped helper bones are allowed.
There is no automatic body fitting. Inspector reports missing/incompatible bindings, and selected
Scene view shows reference-pose shapes, joint axes and swing boundaries.

Parts use Box or local-Y Capsule, bone-local offset/rotation and positive uniform visual scale.
Offsets are bounded to +/-5 m; half-extents/radius [0.01,2] m, capsule half-height [0,2] m;
mass [0.1,50] kg, total <=200 kg; swing angles [0,90], twist min [-90,0] / max [0,90]. The joint
anchor is the child bone reference origin; local +Y is twist and +X/+Z are the swing references.
There are at most four authored mappings (including disabled drafts), 48 reserved bodies and 44
internal joints. Preparation reserves capacity before gameplay; no corpse allocation occurs in a
contact callback. Reimport validates signature and paths before replacing the usable resource.

Set Breakable's Terminal Action to Ragdoll. Destroy remains the default and historical migration.
The accepted lethal interaction still emits feedback, contents and Broken/objective credit once.
Then the actor subtree is sealed against gameplay, without changing authored active/enabled flags.
At the next fixed boundary, physics takes the last committed animated pose and inherited body
velocity. Animation/markers, scripts, damage/healing, navigation/path/vehicle motion and new
spawning cannot compete with terminal ownership. Already-pending destruction wins.

Private corpse bodies collide only with eligible Static environment under the environment mask.
They are not public query/contact targets, cannot push Characters or Dynamic actors, and do not
collide with each other. They do not count as another kill or pickup. The actor/visual frame stays
fixed while the shared final-pose pipeline publishes physical bones and captured unmapped helper
transforms. Pause freezes age; Step advances one fixed tick. Lifetime defaults to 6 seconds,
bounded [0.1,30], then normal deferred subtree destruction removes joints before bodies. Restart,
Stop and Scene replacement clear runtime ownership; there is no resurrection/get-up API.

Characters, Vehicles, Respawnable actors, nested Ragdolls, primary Camera, Audio Listener and
Canvas/UI subtrees cannot enter this terminal contract. Missing drafts are editable, not playable.
Remove/disable dependent Components deliberately; removal is not a hidden clear or retarget.
Ordinary prefabs remap internal body/visual references; external references reject capture.
World anchors remain world coordinates when placing a prefab. Runtime Prefab Spawner actors
exclude both new Components even disabled, and exclude Breakable Ragdoll mode.

See [Scene fields](SCENE_FORMAT.md#version-45--physical-mechanisms-and-terminal-ragdolls),
[prefab format](PREFAB_FORMAT.md), and [Lua API 20](SCRIPTING_API.md#api-20--physical-mechanisms-and-terminal-status).
The optional Workshop mechanism section in Sky Orchard is project-owned policy, not native game
logic. Its permanent bypass matters: checkpoint recovery does not repair broken mechanisms.
