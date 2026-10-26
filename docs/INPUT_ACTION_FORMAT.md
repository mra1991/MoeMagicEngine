# Input Action Format

Moe Magic Engine v0.6.4 uses one strict project input asset at
`assets/input/input_actions.json`. Editor Play and Build Game select that asset. Standalone Player
may select another contained asset with `--input-actions`. The editor panel changes only the fixed
project-default document through complete validated value requests.

## Current format 3

The UTF-8 JSON document is bounded to 262,144 bytes and has exactly these root fields:

```json
{
  "format": "MoeMagicInputActions",
  "version": 3,
  "actions": []
}
```

There must be 1–64 actions. Each action has exactly `name`, `type`, and `bindings`. Names are
case-sensitive ASCII identifiers, begin with a letter, use only letters, digits, `_`, `.`, or `-`,
are at most 64 bytes, and are unique. Types are `Button`, `Axis1D`, and `Axis2D`; each action has
1–8 bindings.

### Digital controls and composites

Every digital control is exactly `{ "device": "...", "code": "..." }`. Supported devices are:

- `Keyboard`: A–Z, 0–9, Left, Right, Up, Down, Space, Enter, Tab, Backspace, Insert, Delete, Home,
  End, PageUp, PageDown, left/right Shift, Control and Alt, and F1–F12. Escape remains reserved for
  Player shutdown.
- `Mouse`: Left, Right, Middle, and Button4–Button8.
- `Gamepad`: FaceSouth, FaceEast, FaceWest, FaceNorth, LeftBumper, RightBumper, Back, Start,
  LeftStickButton, RightStickButton, DPadUp, DPadRight, DPadDown, and DPadLeft. These are standard
  mapped positions; `Guide` and vendor labels are not supported.

Digital binding shapes remain:

```json
{ "key": { "device": "Gamepad", "code": "FaceSouth" } }
{ "negative": { "device": "Keyboard", "code": "Left" }, "positive": { "device": "Keyboard", "code": "Right" } }
{ "left": { "device": "Keyboard", "code": "A" }, "right": { "device": "Keyboard", "code": "D" }, "down": { "device": "Keyboard", "code": "S" }, "up": { "device": "Keyboard", "code": "W" } }
```

A composite may mix devices, but its controls must be distinct. Equivalent composites within one
action are rejected.

### Scalar Gamepad axes

Axis1D additionally accepts exactly `axis` and `deadZone`:

```json
{
  "axis": { "device": "Gamepad", "code": "RightX" },
  "deadZone": 0.2
}
```

Supported codes are LeftX, LeftY, RightX, RightY, LeftTrigger, and RightTrigger. Stick axes publish
`[-1,1]`, with positive X right and positive Y up/forward. GLFW Y is inverted at the platform edge.
Triggers are normalized from GLFW rest-to-full `[-1,1]` to `[0,1]` before processing.

### Two-dimensional Gamepad sticks

Axis2D additionally accepts exactly `stick` and `deadZone`:

```json
{
  "stick": { "device": "Gamepad", "code": "LeftStick" },
  "deadZone": 0.2
}
```

Supported codes are LeftStick and RightStick. Positive X is right and positive Y is up/forward. A
stick is one logical source for bounds, equivalence, authoring, and suppression.

### Dead zones and aggregation

Every analog binding owns one finite dead zone in inclusive `[0.0,0.95]`; new and shipped analog
bindings default to `0.20`. For signed scalar `v` and dead zone `d`:

```text
clamped = clamp(v, -1, 1)
output  = 0                                      when abs(clamped) <= d
output  = sign(clamped) * (abs(clamped)-d)/(1-d) otherwise
```

Triggers use the positive half of the same formula. Sticks first sanitize and clamp to the unit
circle while preserving direction, then apply that rescaling to magnitude. Magnitude at or below
`d` is `(0,0)`; no nonfinite value is published.

An Axis1D action may mix digital pairs and scalar axes. Axis2D may mix digital composites and
sticks. Bindings add in authored order and final components clamp to `[-1,1]`; analog magnitude is
not quantized. Button held is the OR of its bindings and pressed/released are transitions of that
aggregate. The same analog source cannot appear twice within one action, even with different dead
zones. At most 128 unique Keyboard, Mouse, Gamepad-button, scalar-axis, and stick sources may be
sampled.

## Strict format-1 and format-2 compatibility

Readers accept strict versions 1, 2, and 3. Format 1 stores scalar Keyboard tokens such as
`{ "key": "Space" }`; format 2 stores tagged Keyboard/Mouse digital controls. Both migrate in
memory without changing action, binding, control-position, or sign order. They cannot name analog
or Gamepad sources. Loading does not rewrite disk. The next explicit successful Save emits format
3.

## Runtime ownership and suppression

Each executable owns one transient standard-mapped Gamepad selector. It chooses the lowest readable
mapped GLFW ID initially, keeps it while valid, and chooses an ascending replacement only after an
observable neutral frame. No device states are merged and no ID, name, GUID, platform index, or
GLFW type enters maps, Behaviours, Scenes, or persistence.

Gamepad authorization is independent of Keyboard and Mouse. Editor authorizes it only for a
running, focused, visible, nonzero Game view with focused application and no blocking document
decision. Player authorizes it only after startup presentation while focused, visible,
non-iconified, and nonzero. Scene view and ImGui navigation never receive this gameplay path.

Authorization loss, disconnect, or replacement emits neutral/release once and suppresses already
held/nonneutral sources. Digital suppression clears after release; scalar/stick suppression clears
after the dead-zone-processed value becomes neutral. When different actions use one analog source,
the smallest authored dead zone controls common neutral detection.

Successful device acquisition emits one informational lifecycle diagnostic. Disconnection or
standard-mapping loss and state-read loss each emit one warning on the transition to neutral.
Steady connected, absent, disabled, or unscanned frames do not repeat diagnostics. Messages contain
no transient joystick ID, name, GUID, or vendor identity.

## Canonical authoring, save, and package closure

The writer always emits strict format 3 with deterministic root/action/binding/control field order,
two-space indentation, no BOM, and one final newline. Action and binding order are preserved.
Editor operations include Gamepad digital selection, Digital/Analog kind replacement, scalar/stick
selection, bounded dead-zone editing, and existing add/delete/reorder/type operations. A failed
candidate preserves the prior valid definitions, order, dirty/conflict state, and source bytes.

Save serializes and production-reparses the whole draft, verifies exact baseline bytes, writes and
flushes one invocation-owned sibling, then performs a Windows write-through replacement. Reload
validates before publication and requires confirmation for a dirty draft. Dirty, unavailable, or
externally changed input blocks Play and Build Game. Build Game packages the exact validated source
bytes; it does not canonicalize a separate copy.

Duplicate JSON members, unknown/extra/missing fields, wrong types, invalid device/code pairs,
invalid binding shapes, repeated composite sources, equivalent bindings, nonfinite/out-of-range
dead zones, exceeded bounds, and unsupported versions fail transactionally.

Runtime preferences never alter this authored format-3 document. v0.6.8 may replace one existing
logical source slot in an immutable effective runtime map through semantic deltas documented in
`INPUT_PREFERENCES_FORMAT.md`; action/binding structure, dead zones, and the exact project/package
bytes remain unchanged. Multiple players or device assignment, raw joysticks, cursor/delta/wheel
actions, analog-to-Button thresholds, chords, processors beyond dead zone, response curves,
inversion preferences, haptics, and glyph selection remain outside format 3 and preferences.

## Implemented v0.11.7 editor recovery boundary

The v0.11.7 implementation does not revise this authored format. While the fixed editor document is
dirty, an editor-only recovery service may embed its canonical current format-3 text in the strict
generated record:

```text
Library/Recovery/active_input_actions_recovery.json
```

The record contains exact authored-source and embedded-document fingerprints and is bounded to
1 MiB encoded bytes; the embedded Input Actions document retains this format's 262,144-byte limit.
Restore reparses through this production codec, rereads the fixed authored source, requires the
recorded source fingerprint to match, and publishes a dirty editor draft without writing source.
Changed, missing, unreadable, malformed, future, or hash-invalid state cannot restore as Untitled,
merge, force overwrite, affect runtime, or enter Build Game. Successful ordinary Save reconciles
the generated record only after source publication. Exact architecture and release gates are in
`docs/exec-plans/v0.11.7-transactional-input-build-settings-recovery.md`.
Implementation evidence is in `docs/implementation-reports/v0.11.7-implementation.md`.
