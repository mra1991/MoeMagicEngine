# Navigation, perception and agent states

v1.8.0 uses editor-only Recast and shared runtime Detour 1.6.0, pinned to
`6dc1667f580357e8a2154c28b7867bea7e8ad3a7`. Scene 44/prefab 38 store authored Components; API 19
exposes copied runtime status and deferred controls. No Crowd, tile cache, runtime rebaking,
off-mesh jumping, dynamic carving or runtime actor spawning is included.

## Authoring contract

Create one enabled Navigation Surface on a root, unrotated, unit-scale object. Its local bounds
default to center `(0,0,0)`, size `(20,10,20)`, source mask `65535`. The bake includes enabled active
solid Static Rigid Body Box/Sphere/Capsule geometry intersecting these bounds and mask, with actual
world transforms. Render meshes, triggers, query-only Colliders, Characters and moving bodies do
not define walkability. Duplicate physical sources remain significant.

| Profile value | Default | Range |
| --- | ---: | --- |
| agentRadius | 0.5 | 0.05–5 |
| agentHeight | 2 | 0.1–10; at least twice radius |
| maxSlopeDegrees | 45 | 0–60 |
| maxClimb | 0.3 | 0–1; less than height |
| cellSize / cellHeight | 0.2 / 0.1 | 0.05–0.5 each |

NavAgent needs a root upright uniformly positive-scaled object, centered-XZ solid Box or local-Y
Capsule and enabled Kinematic Rigid Body. Its actual envelope must fit the baked profile and source
mask. Box radius is its XZ half-diagonal; the foot is the actual bottom center. Defaults/ranges:
speed 2 (0–10), turnSpeedDegrees 180 (0–720), arrivalTolerance 0.2 (0.05–1), repathInterval 0.5
(0.1–5). Assign the same Scene's Surface. Other pose writers cannot compete, even when the optional
state controller is disabled. Physical sweeps, support, slope/step and rotation clearance precede
one Jolt Kinematic target; a failed proof is Blocked, not teleportation. Zero-radius Boxes keep a
2 mm separation above their verified support to avoid degenerate coplanar sweep contacts; this
does not relax wall, overlap or rotation-envelope rejection. Capsule behavior is unchanged.
Box yaw clearance uses a cylindrical envelope with radius equal to the physical half-diagonal,
containing every intermediate yaw without the artificial corners of a square envelope. Leave
adequate baked-radius margin for voxel/contour approximation near obstacles; matching the physical
radius almost exactly can still produce a physically Blocked route at a simplified corner.

Perception has 0–8 unique optional target references, eyeOffset/targetOffset `(0,1,0)`, range 10
((0,100]), full fieldOfViewDegrees 120 ((0,360]), queryLayer 0 (0–15), occlusionMask 65535 and
sampleInterval 0.2 (0.05–5 seconds). Offsets are object-local. The world-forward full cone and real
committed physics LOS ignore self, triggers, query-only, inactive and pending-destroy objects.
Symmetric masks apply. Depleted Health/recovering targets are ineligible. Nearest visible wins;
authored order breaks ties. A target need not have a Collider. The sampled targetOffset point is
the pursued destination; choose a point near the walking surface, not an elevated mesh center.

Agent State Controller needs enabled NavAgent and Perception. It has 1–16 world-space patrol points,
patrolPauseSeconds 1 (0–30), leashDistance 10 ((0,100]), lostSightSeconds 1.5 (0–10). Patrol pauses
only on achieved arrival. Pursue uses completed observations, retaining the last observed point
through grace; no hidden target position is read. Lost sight, ineligible target or leash violation
causes Return to original grounded home. Return suppresses reacquisition until arrival. An
unreachable/blocked home remains truthful rather than snapping. It owns no damage, animation or
sample-specific rules; compose existing Components and project Lua for those.

## Bake transaction and freshness

Enter a canonical `assets/navigation/*.nav.json` path in the Surface Inspector, then Bake.
One detached CPU worker owns copied source values only. Cancel is checked between stages. The
main thread rechecks document identity/revision, source fingerprint and settings before flushing
and atomically replacing a sibling temporary file. Cancellation, stale inputs and failures retain
the old file and Scene. Reference assignment uses ordinary history; Undo/Redo changes the reference,
**not external bake bytes**. Save the Scene after success. Selected bounds/patrol/route overlays
appear only in Scene view, never Game/Player.

Fingerprints use a canonical geometry multiset plus settings: names/IDs/rendering do not matter;
geometry/profile changes and duplicate counts do. Scalars use exact `f32be1` IEEE-754 binary32 hex
with signed zero normalized, shared by Windows/Web. No tolerance hides changed geometry. Play
uses fresh edit-world resources; Build validates the saved Scene. A surface-only empty draft does
not block unrelated gameplay. An enabled agent needs a matching nonempty bake. Declared invalid
navigation files never ship. Retained source removal/change invalidates runtime navigation at the
next fixed boundary; there is no automatic rebake/new-source registration.

## Portable navigation asset 1

UTF-8 JSON stores the format/version, pinned bake implementation, origin/bounds, full profile,
source mask/fingerprint and bounded value arrays: voxel XYZ vertices, six vertex/six neighbor
indices per polygon, detail mesh ranges, world detail XYZ and triangle indices/edge flags. It never
stores native Detour pointers/struct bytes. Validation reconstructs Detour privately in shared
runtime code. Navigation is an explicit Asset Browser/dependency-index kind. Windows/Web consume
the same saved-Scene closure and include only declared navigation data and required notices.
Distribution 7 admits the exact `ThirdParty/RecastNavigation/1.6.0/License.txt` notice.

Limits: 256 sources, 65,536 source triangles, 512×512 voxel cells, 64 MiB bake allocation, 8 MiB JSON,
8,192 polygons, 65,534 base vertices; 32 MiB aggregate runtime navigation allocation, 32 agents and
32 sensors. Four path searches and 32 LOS rays per rendered frame use fair cursors across fixed
catch-up ticks; whole sensor samples reserve their ray budget. Queries cap 2,048 nodes, 256 corridor
polygons and 64 corners. Local projection is at most 0.25 XZ/0.5 Y; partial/far/stacked routes reject.

## Ownership / typed tooling seam

Components own configuration only. `EditorNavigationOperations` captures/cancels/publishes a bake;
ordinary `EditorObjectOperations` inspect/edit Components and history. `NavigationData` validates
portable arrays; `NavigationSource` owns canonical physical fingerprints. `RuntimeNavigationWorld`
owns per-Play route/observation/controller queues and budgets; `CollisionWorld` owns achieved pose.
`PlaySession` schedules prior-command retirement, sensing/decisions, callbacks, target preparation,
physics and committed snapshots. Lua reads copies. Asset indexing and `GameContentSnapshot` share
the same identity/schema/physical preflight, not a second navigation implementation.
