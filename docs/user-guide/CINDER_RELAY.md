# Cinder Relay — Outpost mission

Choose **New Project → Cinder Relay** to create an independent writable sample. Its three Scenes
form Main Menu → Outpost → Results. Start Mission begins a fresh attempt; there is no disk
Continue. Your copy receives its own save identity, with durable saving disabled by default.

## Route and recovery

Follow the amber road through **Switchback**, using real crate cover against the first Soldier.
Beyond the original pass, the exposed road and longer shallow ditch lead to **Pump Cut**.
Approach the lever and press **E / FaceNorth** when its prompt appears. Restoring the pump
unlocks the transmitter; reaching a checkpoint early cannot skip the pump objective.

The cyan flag behind the shelter secures the next checkpoint. In **Mast Yard**, use the shed,
crates and tower to separate encounters. Activate the transmitter beside the tower, then reach
the cyan extraction pad. Both relays are required, not every enemy defeated.

Player Health is six. Recovery restores six at the latest safe checkpoint, retaining restored
relays and defeated Soldiers. A Soldier takes three accepted rifle hits; living hits interrupt
its action, and defeat creates a short-lived physical body. Replay resets the attempt. Results
shows only an accepted current mission result; direct entry does not manufacture a victory.

## Controls, menus and browser use

Click Game to capture look. W/A/S/D moves, Space jumps, left mouse fires, R reloads and E interacts.
The capture click does not fire. Escape releases look; click again to resume. Gamepad uses left
and right sticks, FaceSouth jump, RightBumper fire, FaceWest reload, FaceNorth interact and Start
to resume. The six-round magazine, Health, objective and interaction prompt are on the HUD.
Interaction selects the nearest eligible terminal by distance/line of sight, not reticle facing.

Main Menu includes Guide, native Controls/Graphics overlays and Quit. Results offers Replay and
Main Menu. Native Player Quit closes its window. Web Quit stops and asks you to reload; it cannot
close the tab. Start Game is the browser audio gesture. Use standalone Chrome or a frame allowing
pointer lock; the embedded Codex tab is not a validated FPS input host.

## Edit and publish your copy

Use typed commands, the Inspector and ordinary Undo/Redo. Installed authoring support exposes
Panel, Controls launcher, Graphics launcher and Quit through complete component descriptors.
Preserve Canvas ownership and native Button/visual/marker order. Named Save As accepts an optional
validated Scene display name; publication derives the game name from the saved startup Scene.

Rifle and sentry prefabs are reusable disabled drafts. Bind external Camera/player/Navigation
Surface references before enabling them. Sentry presentation references are instance-local; do
not add a singleton lookup or another motion/fire/damage authority. Its actor must be a Scene
root before enabling physical terminal mapping. Disabled spare mappings also count against limits.

Terrain is the actual floor. Preserve walkable routes and clear checkpoints. After sculpting or
moving solid cover, save terrain, rebake Basin navigation, save the Scene, then Play. A stale bake
is a preflight error, not a reason to disable collision. The original pass and hand-edited scenery
remain; the mission adds a shallow ditch and deliberately authored static cover.

Save the entry/mission/results Build Settings declaration before Windows/Web publication.
Keep selected asset bundles and notices. Web includes the root-index itch ZIP and local Preview.
Relocate the whole Windows output folder, not just its executable.
See the [User Manual](USER_MANUAL.md) for project ownership, input and publication details.
