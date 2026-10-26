# Moe Magic Material Format

Reusable Material assets are deterministic UTF-8 JSON documents beneath `assets/materials/` with
the complete case-insensitive suffix `.material.json`. The current canonical writer emits strict
format 4 with marker `MoeMagicMaterial`; the reader accepts exact format 1–4 documents. Parsing
is transactional and rejects duplicate, missing, extra, malformed, non-finite, out-of-range,
oversized, unknown-model/mode, or future content before runtime publication changes.

```json
{
  "format": "MoeMagicMaterial",
  "version": 4,
  "name": "Example",
  "shadingModel": "MetallicRoughness",
  "baseColor": [1.0, 1.0, 1.0],
  "baseColorTexturePath": "",
  "specularColor": [1.0, 1.0, 1.0],
  "specularStrength": 0.5,
  "shininess": 32.0,
  "metallicFactor": 0.0,
  "roughnessFactor": 0.5,
  "emissiveColor": [0.0, 0.0, 0.0],
  "emissiveIntensity": 0.0,
  "surfaceMode": "Opaque",
  "opacity": 1.0
}
```

The exact format-4 fields are:

- `name`: valid UTF-8, 1–128 bytes, with no control characters;
- `shadingModel`: exact `LegacyBlinnPhong` or `MetallicRoughness`;
- `baseColor` and `specularColor`: three finite display-authored sRGB channels in `[0,1]`;
- `baseColorTexturePath`: empty, or a normalized project-relative existing PNG/JPG/JPEG identity
  contained canonically beneath `assets/textures/`;
- `specularStrength`: finite `[0,1]`;
- `shininess`: finite `[1,256]`;
- `metallicFactor` and `roughnessFactor`: finite `[0,1]`;
- `emissiveColor`: three finite display-authored sRGB channels in `[0,1]`;
- `emissiveIntensity`: finite `[0,16]`;
- `surfaceMode`: exact `Opaque` or `AlphaBlend`;
- `opacity`: finite `[0,1]`.

The record is deliberately fixed-shape. Legacy values remain present while
`MetallicRoughness` is active, and metallic/roughness values remain present while
`LegacyBlinnPhong` is active. The Materials panel disables but displays the inactive group, so
switching models is reversible without discarding authored values. New documents default to a
white dielectric `MetallicRoughness` surface with metallic `0`, roughness `0.5`, and dormant legacy
white/specular-strength `0.5`/shininess `32` values. Emission is active for either shading model;
new documents default to black emission at intensity `0`. New documents also default to `Opaque`
at opacity `1`. Opaque rendering deliberately ignores the stored opacity, but the panel retains and
disables the scalar so switching to `AlphaBlend` restores the authored value without loss.

Save emits the fields in the order shown, two-space indentation, and one trailing newline. Material
identities must remain lexically and canonically beneath `assets/materials`; case-colliding project
identities are rejected. The document limit is 262,144 bytes.

## Format-1, format-2, and format-3 read migration

The format-1 reader still requires its exact released fields: `format`, `version`, `name`,
`diffuseColor`, `specularColor`, `specularStrength`, `shininess`, and `diffuseTexturePath`. A
successful read migrates only the in-memory value:

- the model becomes `LegacyBlinnPhong`;
- `diffuseColor` becomes `baseColor`;
- `diffuseTexturePath` becomes `baseColorTexturePath`;
- legacy specular values retain their exact meaning;
- dormant metallic and roughness become `0` and `0.5`;
- emissive color and intensity become black and `0`;
- surface mode and opacity become `Opaque` and `1`.

The format-2 reader retains its exact released fixed-shape Legacy/PBR fields and likewise supplies
black/zero emission only in memory. Format 3 retains its exact emission fields. All three historical
formats supply `Opaque`/`1` only in memory. Merely opening them does not rewrite source bytes. Any
explicit Save or Save As writes canonical format 4, making the upgrade deliberate and deterministic.

## Runtime publication and rendering

Material-asset base and emissive colors are display-referred. `RuntimeResourceService` applies the
exact piecewise sRGB transfer once when preparing an immutable shared `Material`; optional
base-color images load with `WorldColor` texture semantics. Publication uses the released weak, case-folded Material cache
separate from importer-owned Model-section Materials. Explicit Save or Reload prepares a complete
replacement before publication and updates all stopped-Scene renderers with the same normalized
identity; importer-owned Materials are never mutated in place.

The shared static/skinned GLSL 330 geometry fragment path chooses exactly one direct-light branch:

- `LegacyBlinnPhong` retains the released diffuse/specular calculation for procedural geometry,
  OBJ/MTL, migrated format-1 assets, and explicit Legacy assets;
- `MetallicRoughness` uses finite Cook-Torrance direct lighting with GGX distribution, Smith
  masking-shadowing, Schlick Fresnel, dielectric F0 `0.04`, energy-conserving diffuse, and a bounded
  internal roughness floor of `0.045`.

After either branch, the shared static/skinned shader adds
`linearEmissiveColor * emissiveIntensity` exactly once. This radiance is independent of base color,
textures, renderer tint, lights, shadows, view direction, metallic/roughness, and environment. It
does not illuminate other geometry; zero emission preserves the released surface result.

Opaque sections retain the released stable shadow/color order, write depth, and output alpha `1`.
AlphaBlend sections never cast the Directional shadow. After opaque geometry and the procedural sky,
the renderer sorts their copied current world-bounds centers by descending Camera-forward depth,
breaking ties by ascending GameObject ID then section index. Each section depth-tests without depth
writes, uses straight-alpha `FUNC_ADD` blending (`SRC_ALPHA`, `ONE_MINUS_SRC_ALPHA` for RGB and
`ONE`, `ONE_MINUS_SRC_ALPHA` for alpha), and draws culled back-facing surfaces before culled
front-facing surfaces. Lighting, shadow receiving, environment, and emission are evaluated before
the scalar opacity is composed in linear HDR.

The released lowest-ID Directional plus at most eight Point/Spot lights, Directional shadow
visibility, procedural sky, analytic PBR environment response, and compatibility ambient remain
shared. World radiance renders into fixed four-sample RGBA16F, resolves nearest to single-sample
RGBA16F, then one mandatory half-resolution bloom processor extracts at threshold `1`/soft knee
`0.5`, performs five fixed separable blur pairs, and composes at `0.08` before unchanged exposure
`1`, Reinhard mapping, and exact sRGB. Canvas, HUD, splash, Input Preferences, and ImGui remain
post-presentation and never bloom. Shadows modulate only the primary Directional direct term.

Supported static and skinned glTF sections publish `MetallicRoughness` Materials directly from
their already-linear base-color factor and scalar metallic/roughness values. A glTF primitive with
no Material uses the glTF defaults: white base color, metallic `1`, roughness `1`. Procedural and
OBJ/MTL sources remain Legacy. All importer-owned Materials remain `Opaque` at opacity `1`; glTF
alpha modes and base-color texture alpha remain unsupported.

Scene 32 and prefab 26 store only optional `materialPath` identity on `MeshRenderer`. A resolved
override applies to every static or skinned Model section; renderer tint and lighting still compose
normally. Geometry bounds, picking, framing, collision, animation, and gameplay ignore opacity.
Empty identity preserves each procedural or imported section Material exactly.

The dockable Materials panel edits one conflict-aware draft. New/Open/Save/Save As/Reload and
Save/Discard/Cancel switching are explicit; Play is read-only, and a dirty Material blocks Play,
Build Game, and exit until resolved. Inspector and Asset Browser assignment are undoable Scene
edits; editing Material content is an independent asset edit. Build Game copies only referenced
format-1/2/3/4 Material documents and optional base-color texture bytes, labels them with existing
Manifest-2 `material`/`texture` dependency kinds, then reparses the staged package through the
production runtime boundary.

Metallic-roughness, normal, occlusion, emissive, and other effect maps; glTF emission; tangents;
texture/glTF alpha, cutouts, refraction, transparent shadows, alternate blend modes, per-triangle
sorting/OIT, and particles; global illumination; physical light units; selectable bloom/exposure/
effect quality;
per-section overrides; imported-Material editing; previews; automatic hot reload; Material graphs;
runtime mutation commands; rename/delete; and GUID/meta files remain unsupported.
