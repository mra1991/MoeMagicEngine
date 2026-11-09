# Moe Magic Engine user documentation

The v2.0.0 release adds [AI Designer Preview](USER_MANUAL.md#ai-designer-preview): optional
OpenAI inspection/planning, asset intake and selected reviewed Model assignment. Broader editing,
testing and publishing remain ordinary editor/structured-adapter features, not qualified live AI
operations. The guide covers user keys, consent, API allowances, review and offline use.

For the v1.20 development edition, see [Paint a terrain road and review scenery](USER_MANUAL.md#paint-a-terrain-road-and-review-scenery).
Group placement uses Scene Undo; palette, paint and masks use Terrain Undo. Neither replaces
an explicit navigation rebake after changing a solid obstruction.

The earlier v1.16 release added the curved three-act Boulder Run, PathGuided camera, Rolling Visual,
captured-route queries and copied authoring inspection. The accepted v1.15 six-act Orchard,
three Gem goals, sky/music and box/stack/TNT/300-unit Health policy remain unchanged. New procedures describe implemented behavior; historical screenshots
retain their original attribution. Independent installed E02 and owner acceptance are recorded
separately in the final test report.

Start here to learn the editor, author a game, and produce a Windows Player or Web game.

This **v2.0.0 edition** includes Basic 3D, Ultimate Platformer and Cinder Relay as the
three New Project choices. Cinder's Outpost mission has ordered relays, recovery and Results;
Platformer's compact Home/Level Select/Settings/Guide/Credits menu is unchanged. Optional
editor-only authoring support lives in the installed `Authoring` directory; normal users continue
with the graphical workflow. Independent acceptance is recorded separately.
Pot Pursuit is retired from the product/tutorials; existing user copies remain compatible.

The accepted **v1.12.0 release** adds reusable interaction, prepared swept projectiles and framing-only camera
zones. The manual explains ordinary Component/Lua composition; the Platformer tutorial describes
the optional utility-cache detour. Independent installed acceptance and all four owner judgments
passed; instructions are not a substitute for the separately recorded release evidence.

The v1.11 release adds explicit saved progress, Continue/New Game, separate game identity and
typed single-base prefab variants. The manual/tutorial explain graphical setup and project-owned
policy; no compiler/converter is needed for ordinary users. v1.10 pose constraints are accepted.
v1.11 independent PASS and owner acceptance are recorded in its final test report.

Use **Edit > Preferences > Editor Text Size** (Compact/Comfortable/Large)
and the **File > Build Game...** target/results panel. **Build Web** produces the previewable
folder plus an upload-ready itch.io ZIP, with **Preview**, output locations and **Retry ZIP**.
Start a sample through **New Project > Template > Ultimate Platformer > Create**;
these are complete independent writable game projects, not merely loose asset packs. The built-in
Character Controller and named Input Actions support keyboard and standard gamepads. The manual's
new Web section covers click-to-start audio, local preview, browser limitations and itch.io ZIPs.

| Document | Use it for |
| --- | --- |
| [Complete User Manual](USER_MANUAL.md) | The feature reference, editor procedures, scripting guide, and troubleshooting. |
| [Ultimate Platformer — Complete Game Tutorial](ULTIMATE_PLATFORMER.md) | Build a character-driven platformer with physics, moving platforms, hearts, hazards, sound, and menus. |
| [Cinder Relay — Outpost Mission](CINDER_RELAY.md) | Play and author the three-section FPS mission, relays, checkpoints, sentries and native menus. |

The original tutorial reference baseline is **v0.12.3**, repository commit `f6fbca1`, and the sample
content in that checkout. The games are independent projects and may evolve without an engine
version change. The manual includes accepted asset/runtime-prefab/interaction/progression,
path/Camera, vehicle/three-level, skeletal/state/NPC, navigation and physical-mechanism workflows,
plus v1.10 pose constraints, v1.11 progress/variants and v1.12 action gameplay. Independent Tester and owner
acceptance are complete. Prior unaffected itch.io/installed evidence carries forward; accepted v1.10
evidence is in its final report (see installed documentation/source reference). The owner-approved
[Free Proprietary License](../../LICENSE.txt), not this explanatory guide, governs engine use
and game/runtime distribution.

Read the manual's first three chapters, then choose a tutorial. General procedures live in the manual; tutorials supply the game-specific composition, settings, and scripts. The shipped projects are reference solutions, not files to overwrite while learning.

### Starter Asset Library

Open **Asset Browser > Shared Library**, search a name/tag such as Gem, and filter by type/pack.
Select an entry to inspect support, creator/source/license, dependencies and existing origin copies.
Choose **Plan Add to Project**, review the new bundle and notices, then **Add to Project**.
The revealed project copy can be assigned/instantiated separately. Add is blocked during Play.

An exact existing copy reports AlreadyPresent. Edited or incomplete copies report Conflict;
enter a different explicit new-copy name to keep both. Add never overwrites or merges folders,
changes a Scene or creates a Scene Undo entry. If indexing fails after publication, the complete
bundle remains: use Project Refresh rather than recopy. Missing catalogs explain build/repair;
the editor never searches CWD or another project. **Help > Starter Asset Library** retains shelf
guidance. Save your Scene and Build Game from your own project; only used dependencies and their
notices/provenance are packaged. Uninstall removes the shelf, not external project copies.

Right-click an external model/WAV in the Project folder listing for **Asset Source / Provenance**.
Declare creator/source, license, local notice paths and redistribution confirmation explicitly.
This is your declaration, not a legal certification by Moe Magic. Unknown model rights remain a
publishing block. `asset-source.json` original hashes record history; your project copy is editable.

Five selected CC0 model packs are supplied: KayKit Adventurers 2.0 FREE, Kenney Blocky Characters
2.0, Kenney Food Kit, Quaternius Ultimate Animated Character Pack (Nov 2019), and Ultimate Platformer
Pack (Dec 2021). Supported OBJ/MTL, JSON glTF and two official Kenney static FBX examples include complete dependencies.
FBX supports contained/embedded PNG/JPEG and material sections, plus bounded linear skeletal
skinning and baked TRS animation. Four nonzero influences per vertex are allowed; unsupported
morphs, constraints and over-limit assets are rejected, not silently discarded. The selected
Worker_Male_Moe4 derivative and its conversion receipt are supplied separately from its unsupported
original. Existing bounded animated glTF support remains available.
Unsupported GLB and the morph-bearing bow variant are excluded from this shelf, not silently converted.

The nine audio packs supply 75 decoder-validated WAVs. Spatial3D still requires mono.
`StarterLibrary/SourceAudio` separately retains the original Kenney OGGs with provenance.
They are **not AudioClips**: export selected sounds to a supported WAV using an external tool,
then copy the WAV and notices into project-owned `assets/audio`. Renaming the extension is not
conversion. Existing WAV derivative recipes are documented in the shelf README and adjacent
`WAV/DERIVATIVES.md` files.

### License and Windows security

Read the [Moe Magic Engine Free Proprietary License](../../LICENSE.txt), also shown by setup.
There are no engine license fees, royalties or revenue thresholds for the supplied version.
You own your original game. Free/commercial games may distribute the supplied Player/runtime
and required dependencies through Build Game; keep required notices and the built-in Moe Magic
splash/branding. Moe-owned sample/template scripts and content may be adapted into your games.
Editor redistribution, engine-source redistribution or modified-engine redistribution requires
separate permission. Third-party assets retain their independent terms; CC0 remains CC0.

This release is unsigned. Windows may show Unknown Publisher, an unrecognized-app warning,
or block it under Smart App Control/enterprise policy. Do not disable security protections.
Obtain the installer and checksum from the [official GitHub releases](https://github.com/mra1991/MoeMagicEngine/releases).
The adjacent SHA-256 checksum detects changed download bytes, not publisher identity or safety.
A 64-bit Windows computer with an OpenGL 3.3-capable GPU/driver is required; end users do not need
Visual Studio, source code, Pandoc or Inno Setup. The owner accepted installation and a complete
standalone sample playthrough. External clean-machine verification is unavailable and deferred
to real users after publication; it is not claimed as passed.

## Product status and evidence

v0.12.7 integrates three genuine v0.12.6 developer-host figures: the setup destination
page, New Project Template choices, and an installed sample copy with Help visible. These
unchanged captures are attributed beside the figures; they are not new v0.12.7, clean-host,
default-destination install, or completed-game evidence. The original captures and their limits
remain in the v0.12.6 evidence (see installed documentation/source reference).
The owner reports no clean test machine is available.

The owner approved the original textual edition for v0.12.5. Its source-checked reference
content and unavailable hands-on observations remain historical, as recorded in
[authoring notes](AUTHORING_NOTES.md). The installed product now includes per-user setup,
Basic 3D, both sample sources, and this offline HTML guide. Markdown remains maintained source;
PDF production and a new screenshot campaign are not required for this update.
Independent current-distribution Help/readability and sample-copy verification belongs to Tester.

## Maintaining these guides

- Update the manual when a supported feature or UI term changes; update tutorial settings when their project changes.
- Check the actual Scene and attached scripts before changing a tutorial. Historical project notes can describe superseded behavior.
- Keep links relative so the documentation works in a relocated checkout.
- Store future genuine captures in `images/`, with descriptive names and useful alt text. Do not use mockups as editor evidence.
- Record the release/content revision and interaction coverage in authoring notes. A screenshot's appearance alone is not proof of a gameplay or build result.
- Keep production code, authored game content, and personal editor state outside documentation changes.
