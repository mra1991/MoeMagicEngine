# Input Preferences Format

Moe Magic Engine v0.6.8 stores optional player-local rebinding deltas as strict UTF-8 JSON. The
authored `MoeMagicInputActions` format-3 asset remains the immutable base and is never rewritten by
runtime controls. A preferences file is at most 262,144 bytes, has no BOM, uses two-space JSON
indentation, and ends with one newline.

## Format 1

The root has exactly four fields:

```json
{
  "format": "MoeMagicInputPreferences",
  "version": 1,
  "baseMapSha256": "64 lowercase hexadecimal characters",
  "overrides": []
}
```

There are at most 2,048 overrides. Each override has exactly `action`, `type`,
`bindingFingerprint`, `slot`, and `replacement`. `type` is `Button`, `Axis1D`, or `Axis2D`.
`bindingFingerprint` is SHA-256 over the centralized canonical semantic representation of the
original authored binding, including action type, binding shape, ordered logical sources, and the
exact finite dead zone. The closed slot vocabulary is `Key`, `Negative`, `Positive`, `Left`,
`Right`, `Down`, `Up`, `Axis`, and `Stick`, as allowed by the original binding shape.

`replacement` has exactly `device` and `code`. Digital slots accept every released Keyboard key
except Escape, all eight Mouse buttons, and all released standard-Gamepad buttons. `Axis` accepts
only a complete standard-Gamepad scalar axis; `Stick` accepts only a complete standard-Gamepad
stick. Runtime preferences cannot add/delete/reorder actions or bindings and cannot edit dead
zones, processors, or inversion.

Canonical output orders overrides by authored action order, binding order, then slot order. A
generated document is reparsed before storage. Matching base digests require every override to
resolve uniquely and the complete effective map to pass `InputActionMapSerializer::Build`. When
the base digest changed, only uniquely compatible action/type/fingerprint/slot matches are retained
and the retained set is validated as one candidate. Invalid candidates leave the authored base
active. Loading never rewrites the preferences file.

## Profile identity and location

The Player profile digest hashes these UTF-8 fields, each prefixed by an unsigned fixed-width
eight-byte big-endian byte length:

1. `MoeMagicInputPreferencesProfileV1`;
2. validated game name;
3. normalized project-relative startup Scene identity; and
4. normalized project-relative Input Action identity.

Editor preview uses the distinct `MoeMagicInputPreferencesEditorPreviewProfileV1` domain with the
canonical project root identity and fixed project Input Action identity. Only the lowercase
64-character digest enters a filesystem path. Windows resolves `FOLDERID_LocalAppData` through the
Known Folder API:

```text
%LOCALAPPDATA%/MoeMagicEngine/InputPreferences/Player/<digest>/input-preferences.json
%LOCALAPPDATA%/MoeMagicEngine/InputPreferences/EditorPreview/<digest>/input-preferences.json
```

Mutation validates the complete effective map and canonical preference bytes, verifies the exact
previous file bytes or absence, writes and flushes an invocation-owned sibling, then performs an
atomic write-through replacement. The immutable effective map and generation publish only after
storage succeeds. External changes, write/flush/replace failure, unavailable Local App Data, or
invalid persisted data cannot produce an unsaved session-only map. Reset All writes a valid empty
override document; it does not recursively delete a profile directory.

Build Game packages the exact authored Input Action bytes and never copies this file or its profile
directories. Relocating a package does not change its profile identity.
