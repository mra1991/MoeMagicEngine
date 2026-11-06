# Terrain resource formats 1 and 2

Terrain1 shipped in v1.19. Terrain2 is the v1.20 format; a v1.19 reader does not support it. Both are portable UTF-8 JSON files directly beneath `assets/terrain/`, with suffix `.terrain.json`. The local origin is the first sample, not the center. Placement and opaque Material belong to the referencing Component.

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

Placement is an editor operation on a **saved** terrain, not a third terrain resource format.
Generator 1 uses uint32 state `s = 1664525*s + 1013904223` modulo 2³². Every candidate,
including a rejection, consumes five draws: weighted source, local X, local Z, yaw, scale.
Map each draw to its inclusive integer interval by multiplying by the interval size and
taking the upper 32 bits. Source order is authored; mask names are canonicalized. The root
is upright at the sampled triangle height minus the rotated/scaled source's minimum Y.
After reserved slots, reject whole XZ bounds, masks, spacing, then terrain grade/height.
Round footprints outward to cm. Seed 7, weights [1,3], bounds [0,0,2000,2000], zero yaw,
scale 1000 and flat ground with a unit cube give the first four (source index,X,Z) triples:
(0,1827,1225), (1,722,106), (1,1063,904), (1,1560,836). Positions and bounds, not current
Scene IDs, participate in the digest. Neither Player runs this generator.

Terrain2 keeps the height fields and diagonal above. It adds exactly three fields, after `rows`:

- `palette`: one to three ordered objects `{"key":"base","srgb":[255,255,255]}`. Each key uses 1–32 ASCII letters, digits, hyphens or underscores. Channels are sRGB bytes. New slots append; only unused trailing slots may be removed. Stable keys cannot silently acquire another slot's paint.
- `weightRows`: as many canonical padded Base64 rows as height rows. The two **unsigned** bytes per vertex are slot 1 and slot 2; absent slots must be zero. Slot 0 is the remainder of 255. Reject sums over 255. Renderers decode each palette color to linear RGB once, blend by these weights, and use the normal opaque Material, texture, UVs and shadow geometry.
- `masks`: up to 16 unique masks, written in ascending key order. Each has `key`, `kind` and `pointsCm` (2–16 distinct local integer XZ pairs for a corridor, 3–16 for an exclusion). A `corridor` additionally requires positive `halfWidthCm` and has round ends. An `exclusion` is a simple, nonzero-area, closed polygon, without repeating the first point. Boundaries are included. The whole mask must fit inside the grid. Editing a mask does **not** repaint or move objects.

Terrain1 opens as one white base without overlays or masks. No-op saves preserve exact existing bytes; height-only edits retain Terrain1. Explicit Save of surface state writes compact Terrain2 JSON and one newline. Once saved as Terrain2, subsequent saves do not downgrade it. The limit remains 131,072 **encoded file bytes**.

Paint moves the chosen byte toward its target by the metric brush falloff and integer strength (nearest, half away from zero). Split the remaining weight in proportion to the previous other weights, floor the shares, then award the residual byte to the largest remainder, lower slot winning a tie. When previous other weights total zero, the lowest other active slot takes the remainder. A one-slot palette is always 255. Example: base/road/bank weights (85,170,0) painted to bank 128 become (42,85,128). Point sampling returns fractional byte weights interpolated on the same triangle as height, without rounding them back to bytes.

Source SHA guards the actual file; authored content SHA governs dirty state and history. Appearance SHA hashes the domain `MoeMagicTerrain/appearance/1`, zero byte, little-endian u16 sample count, each slot's one-byte key length + key + three sRGB bytes, zero terminator, then all decoded overlay byte pairs. Content SHA hashes ASCII `MoeMagicTerrain/content/1:`, the two lowercase hex geometry and appearance hashes, then compact canonical mask-array JSON. Neither mask edits nor paint change the released physical digest or require a nav bake. Palette edits affect cached colors throughout the grid; a modified border weight belongs to both neighboring render sections.

The panel labels integer centimetres; public commands accept local metres, rounded to the nearest centimetre (ties away from zero). Strength uses 10,000 integer steps. A stroke computes all falloffs against one pre-edit grid, not once per rendered frame. Smoothing uses the mean of existing 3×3 neighbours. A pad has XZ half-extents; a ramp interpolates height along a finite segment, including round end caps. Saturated samples are counted in the receipt.

Noise v1 hashes each feature-lattice corner as unsigned 32-bit arithmetic: XOR the seed with X × `9e3779b9` and Z × `85ebca6b` (hex); XOR-shift right 16, multiply by `7feb352d`, XOR-shift right 15, multiply by `846ca68b`, then XOR-shift right 16. Reduce modulo 20,001 and subtract 10,000. Interpolate in X then Z using fixed-point smoothstep. On a flat one-metre grid, seed 42, feature length 2 m, hard radius 4 m at the origin and amplitude 8 m produce −344, −162, +337 and −63 cm at XZ (0,0), (2,0), (0,2) and (1,1).

Preview never writes. Apply, Undo and Redo change the resource draft only; up to 32 transactions and 4 MiB of before/after deltas are retained. Save rechecks the original file digest (or absence), publishes one sibling stage atomically, and refreshes affected live owners. A conflicting edit leaves the draft and the other writer's bytes intact. Discard is explicit; unsaved terrain has no crash-recovery guarantee.
