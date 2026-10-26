# Moe Magic Engine — Getting started

This v1.12.0 Windows x64 release includes per-user setup, the complete portable payload and
independent sample sources. It needs a working OpenGL 3.3-capable Windows graphics driver.
Read the included Licenses/MoeMagicEngine.txt for approved free proprietary use and game/runtime
redistribution. This unsigned release is not clean-machine certification.

## Install and launch

Download `MoeMagicEngine-1.12.0-Setup.exe` and its adjacent `.sha256` from the
[official release](https://github.com/mra1991/MoeMagicEngine/releases/tag/v1.12.0).
Run the installer wizard. The default application location is
your Local App Data Programs folder. Choose an optional Desktop shortcut, then launch **Moe Magic
Engine** from the Start Menu. Game authoring needs no Visual Studio, PowerShell or C++ compilation.
Keep application files together: the Player template, defaults, notices and guide are required.

For an upgrade, close running editor/Player windows and retain the existing installation folder.
Setup replaces application files in place and updates installed-app metadata/shortcuts; it does
not upgrade independent game copies or erase external projects, preferences or saves. Do not
choose a new version-numbered test directory for a normal upgrade. After installation, About and
Windows Installed apps should both report 1.12.0. If a shortcut still opens an older copy, rerun
setup into the original installation folder and select the Desktop shortcut task if used.

No-argument launch reopens the exact valid most-recent project, otherwise the Project Launcher.
It never treats CWD or another recent entry as a fallback game. Source developers retain the
separate BUILDING reference and explicit `--project` / `--project-launcher` commands.

## Create a writable project

1. Choose **New Project...** in the launcher or **File > New Project...**.
2. Choose **Basic 3D**, **Ultimate Platformer**, or **Pot Pursuit**, then enter a project/folder name.
3. Accept the suggested Documents parent or Browse to another existing writable folder.
4. Check the displayed path, then **Create**. It must be a new folder outside the installation.

Basic 3D is the small starter. Named samples produce independent writable copies, retaining
unused intentional assets and notices. Only the project display name changes; Scenes and Build
Settings retain authored names. Never edit installed `Samples` originals. Older/source-built
editors without sample inventories show sample choices unavailable; source developers can Open
their repository Projects instead.

**Open Project...** / **Open Recent Project** open an existing user copy in a new process without
replacing current unsaved work. A published project survives child-launch failure; open its reported
folder again rather than recreate it.

## Play, build and learn

The first valid project Recent Scene opens, otherwise the saved Build Settings startup Scene.
Use Play/Stop for preview. Save changes and configure **Window > Build Settings**, then **File >
Build Game... > Target: Windows > Build Windows** to publish a non-overwriting dependency-closed Player folder independent of the
installation. Preserve asset notices and comply with applicable engine/game redistribution terms.

For browser output use **Build Game... > Target: Web > Build Web**, then **Preview**.
The installed native helper opens localhost; click **Start Game** after loading for audio/input.
No developer toolchain is required. Stop Preview or editor exit closes the helper. Use **Show itch.io
ZIP** and upload the generated `-Web-itch.zip` unchanged for HTML/browser playback. No manual ZIP
step is needed. If archive creation fails, the folder remains usable; resolve the error and **Retry ZIP**.
Keep all Licenses and assets;
see the offline manual for browser limits. Do not launch through `file://`.

**Help > User Manual** opens the offline manual and both tutorials; `Documentation/index.html`
also opens directly. **Help > About Moe Magic Engine** shows version/developer/ownership and GitHub.
**Support Moe Magic on Patreon** opens the owner-confirmed
https://patreon.com/MoeMagicEngine. External links open only on explicit action.

Use **Asset Browser > Shared Library** to search/filter, inspect provenance and **Plan Add to
Project > Add to Project**. Only the selected dependency/notices closure is copied into a new
project-owned bundle. Exact copies report AlreadyPresent; edited/missing copies report Conflict.
Choose an explicit different new-copy name; no overwrite or merge occurs. Assign the revealed
project asset separately. OBJ, bounded JSON glTF, static FBX and bounded skeletal FBX with baked
TRS clips are supported. Skins are limited to four nonzero influences per vertex; unsupported morphs,
constraints and arbitrary exporter features are rejected rather than silently simplified.
**Help > Starter Asset Library** retains guidance. External model/WAV folder rows offer
**Asset Source / Provenance** for explicit rights declarations; unknown models stay local-only.
SourceAudio OGGs require external WAV export; they are not directly assignable AudioClips.
The offline guide explains copying, permissions and decoder limits.

The setup is unsigned: Windows may report Unknown Publisher/unrecognized app or block it under
security policy. Do not disable protections. Compare the supplied .sha256 with the downloaded EXE;
that proves byte integrity, not publisher identity. Clean-host testing is unavailable and remains
pending for public readiness.

## User state and uninstall

Valid layout at `%LOCALAPPDATA%\MoeMagicEngine\Editor\imgui.ini` wins over shipped
`EditorDefaults/imgui.ini`; **Window > Reset Layout** explicitly reloads the seed. Malformed/
unreadable preferences are preserved until reset rather than overwritten automatically.

**Edit > Preferences > Editor Text Size** defaults to Comfortable (16); Compact (13) and Large
(18) are also available in the launcher. `Editor/preferences.json` stores that choice separately;
Reset Layout never resets font size. Invalid/conflicting/unwritable files produce session-only
warnings and retain their original bytes.

Close editor/Player windows before reinstall or uninstall. Setup asks rather than silently killing
unsaved work. Reinstall replaces application inputs, not your external projects or preferences.
Windows' installed-apps list or the Start Menu uninstall entry removes listed application files,
shortcuts and registration. Separately stored projects, layout and recent history remain.

The format-7 manifest lists exact application/docs/default/sample/license/catalog/shelf/Web inputs by size/SHA-256.
Installer bookkeeping is separate. CC0 art/audio retain provenance; the proprietary engine,
scripts and branding are not made CC0 by inclusion. The approved license permits free/commercial game creation, runtime distribution with games and
Moe-owned sample/template reuse; preserve the mandatory splash and notices. Editor redistribution
needs separate permission. No updater, file association, machine-wide PATH change or service is added.
