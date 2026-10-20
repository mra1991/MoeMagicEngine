<p align="center">
  <img src="media/icon.png" alt="Moe Magic Engine icon" width="120">
</p>

<p align="center">
  <img src="media/banner.png" alt="Moe Magic Engine banner">
</p>

<h1 align="center">Moe Magic Engine</h1>

<p align="center">
  A lightweight Windows-first 3D game engine and visual editor with a Unity-familiar Scene/GameObject/Component workflow, Lua gameplay scripting, physics, animation, audio, UI, particles, standalone Windows builds, and WebGL2/WebAssembly browser builds.
</p>

<p align="center">
  <a href="https://github.com/mra1991/MoeMagicEngine/releases/latest"><strong>Download the latest release</strong></a>
  ·
  <a href="https://patreon.com/MoeMagicEngine"><strong>Support on Patreon</strong></a>
</p>

---

## 🚀 Quick Start

**Install Moe Magic → Launch → New Project → choose Basic 3D / Ultimate Platformer / Pot Pursuit → Create → Play**

1. [Download the latest installer](https://github.com/mra1991/MoeMagicEngine/releases/latest), install it, and launch **Moe Magic Engine**.
2. Choose **New Project...** in the launcher, or **File → New Project...** in the editor.
3. Pick a template: **Basic 3D** is the small starter; **Ultimate Platformer** and **Pot Pursuit** are complete playable project templates.
4. Enter a project name, choose a writable location outside the installation, and click **Create**. Moe Magic creates your own writable project copy—there are no demo folders to locate manually.
5. For an instant playable example, choose **Ultimate Platformer** or **Pot Pursuit**, then press **Play** in the editor.

Edit your copy freely, then use **File → Build Game...** to make Windows or Web builds from the same project.

**[Play Ultimate Platformer in your browser](https://mra1991.itch.io/ultimate-platformer-coin-run)** — an actual Moe Magic Web build of the included Ultimate Platformer sample.

## ✨ What is Moe Magic?

**Moe Magic Engine** is an independently developed 3D game engine for Windows.

It is designed around a straightforward workflow:

**Create a project → build scenes visually → add reusable Components → write game logic in Lua → Play → Build Game.**

You do **not** need the engine source code, Visual Studio, or C++ to make a game with Moe Magic. Normal gameplay scripting is done in **Lua**.

The engine itself is proprietary, but it is **free to use with no royalties or revenue threshold** for personal, educational, free, and commercial games.

## 🎮 Highlights

- Visual Scene editor with Hierarchy, Inspector, transform gizmos, Scene/Game views, camera guides, and viewport orientation tools
- Component-based GameObjects
- OBJ and supported JSON glTF model import
- Skeletal animation
- Materials, PBR lighting, shadows, HDR, bloom, transparency, and graphics-quality presets
- Static, Dynamic, and Kinematic physics
- Capsule Character Controller with keyboard/gamepad support and moving platforms
- Keyboard and gamepad Input Actions with runtime rebinding
- 2D and spatial 3D audio
- Canvas UI, menus, HUDs, buttons, and graphical UI images
- Particle bursts and scripted effects
- Project-local Lua gameplay scripting
- Play / Pause / Step / Stop workflow
- Multi-scene projects and scene transitions
- Build Game for standalone Windows players and WebGL2/WASM browser games (**Web (Experimental)**)
- itch.io-ready Web ZIP packaging and built-in local browser preview
- Basic 3D, Ultimate Platformer, and Pot Pursuit templates in New Project
- Configurable editor text size: Compact, Comfortable, or Large
- Built-in offline User Manual and two complete game tutorials
- Bundled CC0 starter asset library

### Editor readability

Introduced in v1 in response to real user feedback, **Edit → Preferences → Editor Text Size** offers **Compact (13)**, **Comfortable (16, default)**, and **Large (18)**. Comfortable gives new users a more readable default on normal displays; Compact keeps the previous denser presentation for larger or multi-monitor setups; Large provides bigger text.

The choice applies without restarting, is also available in the launcher, and persists between sessions. It is separate from layout and game UI text; if preferences cannot be saved, the editor reports that the choice is session-only.

## 🕹 Included sample projects

Both samples are available directly from **New Project** as writable project copies, ready to **Play**.

### Ultimate Platformer

A third-person platforming game demonstrating:

- character movement and camera-relative controls
- animation
- physics and pushable objects
- moving ferries
- coins and health pickups
- enemies and hazards
- particles and spatial audio
- Lua-driven gameplay
- menus, HUDs, and standalone builds

### Pot Pursuit

A timed collection game demonstrating:

- project-owned scoring and objectives
- moving hazards
- timed collectibles
- spatial and 2D audio
- animated menu presentation
- Canvas UI
- Lua gameplay logic
- complete menu → gameplay → result flow

## 🌐 Web Builds and itch.io

Moe Magic v1.0.1 exports working **WebGL2/WebAssembly (WASM)** browser games with an **itch.io-ready ZIP**. The released editor and offline manual label this target **Web (Experimental)**.

**File → Build Game... → Target: Web (Experimental) → Build Web → Show itch.io ZIP → upload the ZIP unchanged to itch.io as an HTML/browser game**

Save your project and configure **Window → Build Settings** first, using the same Scenes and dependencies as your Windows build. Choose an output location outside the project and installation. Build Web produces a `<game-name>-Web` folder and a `<game-name>-Web-itch.zip` archive, with `index.html` already at the archive root alongside the required assets and notices. On itch.io, enable browser playback/click-to-play; no unpacking or repacking is needed.

The compatible Web runtime and local preview helper ship with Moe Magic. Users do **not** need Node, Python, Emscripten, Visual Studio, or a source checkout to export a Web game.

Choose **Preview** to open the build through localhost, wait for loading, and click **Start Game**. Browser audio requires this normal initial user interaction. **Stop Preview** or closing the editor stops the preview server. If ZIP creation fails, the output folder can still be previewed; fix the reported issue and use **Retry ZIP**.

**[Play Ultimate Platformer in your browser](https://mra1991.itch.io/ultimate-platformer-coin-run)** to try the included sample as a published Moe Magic Web game.

### Current browser support and limits

The v1.0.1 offline manual documents these limits:

- Use hardware-accelerated desktop **Chrome or Edge** with **WebGL2** and floating-point render-target support. Mobile and Safari are not guaranteed.
- Keyboard and standard gamepads are supported where exposed by the browser. The Character Controller and sample projects support movement, jumping, and runtime menus with these inputs.
- Serve builds through local Preview or a web host; `file://` is unsupported. Wireframe, arbitrary network asset discovery, and automatic graphics-context restoration are unsupported; context loss requires a reload.
- Gameplay and audio pause when the page is hidden or unfocused. Fullscreen requires an explicit action and may be denied by browser policy.
- Browser settings are scoped to the game's origin and URL directory. If browser storage is unavailable, settings are session-only.
- Graphics-quality options reflect browser support; unsupported startup presets fall back to **Performance**.
- Export limits are **1,000 files**, **240-byte relative paths**, **200 MB per file**, and **500 MB total**, with a stricter **256 MiB** startup-content budget.
- Generated ZIPs use uncompressed entries, so they can be larger than compressed ZIPs.

## 📦 Installation

Moe Magic currently targets **64-bit Windows 10/11**.

1. Open the [latest GitHub Release](https://github.com/mra1991/MoeMagicEngine/releases/latest).
2. Download `MoeMagicEngine-1.0.1-Setup.exe`.
3. Run the installer.
4. Launch **Moe Magic Engine** from the Start Menu or optional Desktop shortcut.
5. Choose **New Project**, select **Basic 3D**, **Ultimate Platformer**, or **Pot Pursuit**, and click **Create**. Press **Play** to try either sample immediately.

The installer includes the Editor, compatible Windows Player and Web runtime, local Web preview helper, documentation, project templates, and the selected CC0 starter library.

> Windows may show an **Unknown Publisher / SmartScreen** warning while the installer is unsigned. Always verify the published SHA-256 checksum if you want to confirm the download.

## 📚 Documentation

Moe Magic includes offline documentation accessible directly from:

**Help → User Manual**

The documentation includes:

- the complete engine User Manual
- an Ultimate Platformer tutorial
- a Pot Pursuit tutorial
- Lua scripting guidance
- authoring, physics, animation, audio, UI, particles, input, and Build Game instructions
- New Project templates, editor text-size preferences, local Web preview, itch.io publishing, and current browser limits

## 🧰 Starter asset library

The installer includes a curated CC0 starter library built from compatible assets by **Kenney**, **Quaternius**, and other CC0 audio creators.

The starter library is intentionally separate from your projects. Copy the assets you want into your own project's `assets/` folder so projects remain self-contained and Build Game packages only their actual dependencies.

Third-party assets retain their original licenses and provenance notices.

## 💜 Support Moe Magic

Moe Magic is independently developed by **Mohammadreza (Moe) Abolhassani**.

If you enjoy the engine and want to support continued development, documentation, examples, and future versions:

### ⭐ [Support Moe Magic on Patreon](https://patreon.com/MoeMagicEngine)

There are no royalties or revenue thresholds for games made with Moe Magic.

## 📜 License

Moe Magic Engine is **free proprietary software**, not open source.

In summary:

- You may use the supplied engine binaries for personal, educational, free, or commercial games.
- You keep ownership of your own game content and code.
- You may distribute the required Moe Magic Player/runtime with games built using Moe Magic.
- The built-in Moe Magic startup branding and required notices must remain intact.
- The Editor and engine source may not be redistributed or rebranded as another engine product without permission.
- Bundled third-party assets remain governed by their own licenses.

See [`LICENSE.txt`](LICENSE.txt) for the full terms.

## 💻 Current release

**v1.0.1 — Current Stable Public Release**

The v1.0.1 installer is **60,185,030 bytes (approximately 60.2 MB)** and includes the Windows editor, Windows Player and Web runtime, local Web preview helper, sample project templates, offline documentation, and curated CC0 starter library.

This release brings itch.io-ready browser publishing through **Web (Experimental)**, a straightforward New Project → Play workflow, and configurable editor text sizing.

SHA-256 checksums are published alongside release downloads.

## 🪄 Development history

Moe Magic has been built incrementally, with each version family establishing another part of the engine:

| Family | Main focus |
| --- | --- |
| **v0.0.x** | Core architecture, rendering, GameObjects, assets, lighting, hierarchy, and Scene persistence |
| **v0.1.x** | Interactive editor workflow, viewport tools, undo/redo, and docked workspace |
| **v0.2.x** | Play Mode, runtime isolation, native Behaviours, input, destruction, cameras, and triggers |
| **v0.3.x** | Early complete gameplay workflows, licensed model library, Scene usability, and prefabs |
| **v0.4.x** | Standalone Player foundation and portable Windows development builds |
| **v0.5.x** | Rigid-body physics, collider shapes, Kinematic motion, spatial queries, and Character Controller |
| **v0.6.x** | Input Actions, gamepad support, Canvas UI, fullscreen Player presentation, and rebinding |
| **v0.7.x** | 2D audio, spatial audio, listener support, and gameplay-driven sound |
| **v0.8.x** | glTF import, skeletal animation, Scene transitions, and authored Main Menu flow |
| **v0.9.x** | Rendering and presentation: lighting, shadows, PBR, HDR, bloom, particles, and quality presets |
| **v0.10.x** | Embedded Lua gameplay scripting and progressively richer user-side APIs |
| **v0.11.x** | Project workflows, recovery, project-scoped operations, dependency diagnostics, and asset reload |
| **v0.12.x** | Project-owned gameplay, multi-project workflow, user documentation, installer, sample projects, and public-distribution readiness |
| **v1.0.x** | Stable conventional engine baseline: Windows distribution, experimental Web builds, itch.io packaging, improved editor readability, and sample templates with a straightforward New Project workflow |

---

<p align="center">
  <strong>Moe Magic Engine</strong><br>
  Developed by Mohammadreza (Moe) Abolhassani
</p>
