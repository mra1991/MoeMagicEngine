# User documentation — authoring and review notes

These are maintenance/handoff notes, not user instructions or a release-gate report.

v1.13 product amendment: the current guide publishes the Manual and Ultimate Platformer only.
Pot Pursuit's historical tutorial/source observations below remain attributed to their original
baseline; the tutorial is no longer a shipped page or New Project choice. The old chooser figure
is explicitly labeled historical. Current support and acceptance are recorded in the v1.13 report.

## Baseline

- Engine tag and HEAD: **v0.12.3**, `f6fbca1` (`chore(release): establish v0.12.3 real-game stabilization baseline`).
- Authored content: the actual current `Projects/UltimatePlatformer` and `Projects/PotPursuit` files in that checkout, including newer owner-authored course/collider/Material changes carried into the tag.
- Basic 3D: `MoeMagicEngine/Templates/Basic3D`.
- Initial Git state: only owner-local root `imgui.ini` modified. Its SHA-256 was `459C82F0EC6F464B3B7495BC71FC92EBF21D6E77F3A70A9287163A5BA6A45D39` and remains the preservation baseline.
- No C++/Lua/game-content edits, engine builds, test harnesses, commits, tags, or pushes are part of this documentation task.

## Evidence used

Read the current-facing portions of README, BUILDING, PROJECT_FORMAT, GAME_DEVELOPMENT, ROADMAP, DEVLOG, ARCHITECTURE, SCENE_FORMAT, PREFAB_FORMAT, SCRIPTING_API, BUILD_SETTINGS_FORMAT, PLAYER_BUILD_FORMAT, MATERIAL_FORMAT, ASSET_DATABASE, Input Action documentation, and workflow/testing policy. Historical sections were treated as historical, not cumulative current limitations.

Parsed the four current game Scenes, Basic3D Scene, project/Build Settings/input documents, Material files, and attached Script Behaviour configuration. Read both gameplay controllers plus orbit, Kinematic route, item configuration, ambient/patrol/spin, repeated sparks, player/spike audio, result-action, and shared UI-feedback scripts as relevant. Verified UI terminology against ProjectLauncher, NewProjectDialog, EditorWorkspace, HierarchyPanel, InspectorPanel, MaterialsPanel, ViewportPanel, and RendererDiagnosticsPanel. Checked supported Lua method/handle registration, audio decoding limits, graphics presets, and reparent behavior where prose was insufficient or conflicted.

This establishes source/API/content-backed instructions. It is **not** an assertion that every reader procedure was exercised physically or synthetically in this pass.

## Desktop attempt and outstanding visual work

The existing Release editor was launched with `--project-launcher` from a fresh temporary working directory. Windows reported a responsive `Moe Magic - Project Launcher` process. Computer Use `list_windows` and refreshed `list_apps` exposed no Moe Magic target, only other applications. No guessed window handles or coordinates were used. The invocation-owned process was stopped after the bounded failed attempt.

One recovery attempt using the native Computer Use launch route returned **Computer Use app approval timed out**. No further desktop-input retries were chained. No editor interaction, New/Open/Recent workflow, sample gameplay, save, Build Game, packaged Player, physical input/audio check, or screenshot was completed. The earlier release's observations are not presented as new evidence.

**Screenshots created: none.** There are no fake editor mockups, reused historical captures, empty image links, or placeholders represented as finished screenshots. The text drafts are reviewable, but the requested hands-on/screenshot portion remains incomplete because of this tooling blocker.

### Bounded follow-up capture sessions

When desktop targeting is available, use a disposable learning project and preserve personal layout/Recent state. Save genuine captures beneath `docs/user-guide/images/` only after inspecting their contents.

| Session (stop/reassess within 10–15 minutes) | Purposeful outputs |
| --- | --- |
| Launcher and starter | Clean launcher/New Project, overall editor overview, Scene/Game distinction and gizmos. |
| Platformer authoring | Character/capsule/animation setup, selected Camera frustum, ferry root/deck overlay, HUD layout. |
| Pot authoring | Left-card menu plus rotating food, gameplay table, Script Behaviour property wiring, spatial Audio Source. |
| Shared workflow | Materials, Build Settings, Play/Pause/Step/Stop, Controls/Graphics and a genuine Build Game result. |

Crop/frame away machine paths, Recent Project names unrelated to the samples, desktop clutter, and diagnostics that do not teach the feature. Do not alter engine/game content to stage a capture. Add each image only where it answers a visual question, with alt text and a caption identifying the relevant UI.

## Product/documentation discrepancies found

No runtime product defect was established: the tool could not interact with the engine. These concrete **documentation mismatches** were found and avoided in the new guides:

| Existing text | Current authority / treatment |
| --- | --- |
| README's older Lua introduction says versions 1–10. | Current runtime registration and SCRIPTING_API support exact versions 1–12; the manual documents current supported calls and phases. |
| README contains old default five-Coin game, root pot demo, and native HUD descriptions. | Current independent projects have their own menus/gameplay; Ultimate has 27 Coins and authored heart/results UI; Pot has food-only project completion. |
| BUILD_SETTINGS_FORMAT ends with a historical statement that runtime transitions/Main Menu are not implemented. | Current declared transition behavior is implemented and used by both games; current Player Build/runtime/script contracts take precedence. |
| PROJECT_FORMAT historical limits say no launcher/relocation/automatic refresh. | Later current launch/distribution and asset Auto Reload sections supersede those introduction-time exclusions. |
| Historical gameplay notes describe circular ferries, missing repeatable particle API, native collection writers, and health-HUD recipes without Canvas ancestry. | Current Scene attachments and API 12 controllers use reversing targets, repeated finite bursts, project policy, and authored Canvas hearts. |
| Pot historical notes give radius 6.5 and fireplace distances 3.5/20. | Current menu authors radius 5.8 and distances 5/25. |
| Pot historical descriptions can suggest negative food or Health damage. | All current food scores are positive, utensils are negative, and no Health is authored; tutorial distinguishes score penalties from health damage. |
| Some project Lua comments still name native timed-item/damage consequences. | Attached current controllers own those consequences. No production comments/scripts were changed during this documentation task. |

The existing large historical documents were not rewritten wholesale. A root README link directs readers to the new coherent user-facing set while keeping useful technical/provenance material intact. Broader archival cleanup is a separate task, not silently folded into documentation authoring.

## Deliberate scope and brevity

- No future AI features, unsupported importer options, runtime spawning, orthographic Cameras, linked prefab workflows, full animation graphs, or nonexistent Lua methods are taught as available.
- General feature mechanics are explained once in the manual; tutorials link back and concentrate on construction, actual values, property references, and policy.
- Decorative object placement is representative rather than an exhaustive 141-object transcription. Essential route, collider, motor, ferry, objective, HUD, sound, and script configuration is explicit. Supplied assets/scripts and the reference Scenes provide exact remaining visual dressing.
- The tutorials reuse supplied scripts, explain their important sections, and link their full current source instead of duplicating every file and creating a second drifting codebase.
- No physical audio/balance guarantees are inferred from authored volume, speed, or lifetime values. Reader completion checklists are instructions, not past test results.
- Installation explains the existing authorized portable folder/source build, not an invented installer/download/licensing promise.

## Documentation validation

- Audited all five new Markdown files: **107 links**, no missing target files or Markdown anchors, and no image references. TOCs resolve to their intended headings.
- Checked balanced fenced code blocks, heading separation, and trailing whitespace. No failures were found.
- Compared the **23 distinct Lua method names called by the documented examples** with current runtime/handle registration; none were missing. Names, relevant signatures, and phase restrictions were also reviewed against the scripting contract and sample sources. This is static verification, not execution of those examples.
- Reviewed the three documents for UI names, Component/property terminology, construction order, duplicated explanations, unsupported features, and sample-specific versus engine-required behavior. Corrections included the exact **Open Recent** Scene menu name, root-only Canvas placement, actual Camera settings, and result-panel geometry.
- Approximate finished text: User Manual **10,000 words / 20 chapters**; Ultimate Platformer **4,500 words / 14 chapters**; Pot Pursuit **4,600 words / 13 chapters**. Counts include tables and code, not only prose.
- `git diff --check` passed. Because the new documents are untracked, each was also checked explicitly with `git diff --no-index --check -- NUL <file>`; none produced whitespace diagnostics (each returned the expected difference status 1 against an empty file). A separate text scan found no trailing whitespace. Git's LF-to-CRLF notices are line-ending normalization notices, not whitespace-check failures.
- The tracked documentation diff is only the root README navigation link/review notice. The five new files are confined to `docs/user-guide/`. No production/game file was edited; nothing was staged, committed, or pushed. Owner-local `imgui.ini` remains at its original SHA-256 above.

Runtime/screenshot coverage remains incomplete as explicitly bounded above. This is a documentation review, not a release test report; no previous build or test result is claimed as evidence produced by this task.

## Proposed commit (not executed)

```text
docs(user-guide): add engine manual and sample game tutorials

Explain current editor, asset, scripting, gameplay, and Player workflows.
Teach Ultimate Platformer and Pot Pursuit from their authored project content.
Record source verification and outstanding desktop/screenshot coverage.
Preserve production code, sample assets, and owner-local editor state.
```
