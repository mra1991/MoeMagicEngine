# Terrain resource format 1

This format is part of the **v1.19.0 release**. A terrain is a portable UTF-8 JSON file directly beneath `assets/terrain/`, with suffix `.terrain.json`. Its local origin is the first sample, not the center. Placement and opaque Material belong to the referencing Component.

| Field | Meaning |
| --- | --- |
| `format`, `version` | `"MoeMagicTerrain"`, integer `1` |
| `sampleCount` | 65 or 129 samples on each axis, not the number of cells |
| `spacingCm` | Integer 50–200; a 129-grid at 100 cm spans 128 metres |
| `encoding` | `"i16le-cm-base64-rows"` |
| `rows` | Exactly one canonical padded Base64 string per Z row, increasing X within each row. Each sample is a signed 16-bit little-endian height in centimetres, in [−3200, 3200]. |

The six fields above (counting format and version separately) form a closed schema. The writer emits them in table order and terminates with a newline; the reader does not require key order. Duplicate fields, future versions, malformed or noncanonical row encoding, out-of-range heights and files larger than 128 KiB are rejected. There is no hole sentinel.

The geometry digest is SHA-256 of the ASCII bytes `MoeMagicTerrain/1`, one zero byte, the sample count and spacing as unsigned little-endian 16-bit values, then all **decoded** row bytes. Consequently whitespace, translation, material and file name do not change the digest. The saved-file SHA hashes the exact source bytes instead: reformatting changes it. Neither digest nor the edit history belongs inside the asset.

A cell is split along its 00–11 diagonal: upward triangles **00–01–11** and **00–11–10**. Within a cell the former supplies height when fractional Z is at least fractional X. The outermost row and column are part of the surface; there is no land past them. Sampled slope uses the face normal; rendering uses neighboring faces to smooth lighting across section seams. The navigation asset, Jolt shape, material pixels and mesh sections are not alternative copies of the source heights.

## Editing units and determinism

The panel labels integer centimetres; public commands accept local metres, rounded to the nearest centimetre (ties away from zero). Strength uses 10,000 integer steps. A stroke computes all falloffs against one pre-edit grid, not once per rendered frame. Smoothing uses the mean of existing 3×3 neighbours. A pad has XZ half-extents; a ramp interpolates height along a finite segment, including round end caps. Saturated samples are counted in the receipt.

Noise v1 hashes each feature-lattice corner as unsigned 32-bit arithmetic: XOR the seed with X × `9e3779b9` and Z × `85ebca6b` (hex); XOR-shift right 16, multiply by `7feb352d`, XOR-shift right 15, multiply by `846ca68b`, then XOR-shift right 16. Reduce modulo 20,001 and subtract 10,000. Interpolate in X then Z using fixed-point smoothstep. On a flat one-metre grid, seed 42, feature length 2 m, hard radius 4 m at the origin and amplitude 8 m produce −344, −162, +337 and −63 cm at XZ (0,0), (2,0), (0,2) and (1,1).

Preview never writes. Apply, Undo and Redo change the resource draft only; up to 32 transactions and 4 MiB of before/after deltas are retained. Save rechecks the original file digest (or absence), publishes one sibling stage atomically, and refreshes affected live owners. A conflicting edit leaves the draft and the other writer's bytes intact. Discard is explicit; unsaved terrain has no crash-recovery guarantee.
