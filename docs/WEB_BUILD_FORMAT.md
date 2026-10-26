# Web Player and export

Web is an additional target, not a replacement for Windows. v1.1.0 removes the visible Experimental
label without changing the browser limits below. The early Ultimate
Platformer feasibility checkpoint passed with separately recorded owner interaction/audio evidence.
The implementation report records build/package
evidence; the final report records independent PASS and owner
physical confirmation. Actual itch.io hosting was not tested in that release.

## Runtime-prefab closure (introduced in v1.2.0)

Web uses the same prepared actor library and deferred runtime admission as Windows; there is no
browser gameplay fork or runtime file discovery. The shared publisher includes declared prefab
bytes and their recursive resources/scripts/provenance, reparses the staged graph and rejects
unsupported/over-budget actors before publication. Unrelated prefabs remain excluded. Web manifest
1, archive structure, preview and strict immutable verification are unchanged; `index.html` remains
the host-mutable wrapper. Current Scene47/prefab41/API23 do not advance these Web package formats.

## v1.0.1 archive publication

The completed plan adds an automatically generated
sibling itch ZIP, exact manifest-plus-files archive inventory, deterministic ZIP32 STORE metadata,
independent folder/archive outcomes, ZIP-only retry and a compact Build Game interface. Root
`index.html` has no enclosing directory. That release preserved the format/limits and then-current
Experimental label and required actual owner-assisted unchanged-ZIP itch.io acceptance. The workflow below
describes the accepted release. Owner Chrome/Incognito hosted acceptance passed after the narrow
host-mutable HTML correction; see the final report.

The editor-only `WebArchiveOperation` consumes the original exact manifest bytes and the verified
published folder. It enumerates only to reject missing/unexpected entries, never to discover new
payload. Each allowlisted file is read once into a bounded verified buffer, then written with the
manifest into a sibling `<GameName>-Web-itch.zip`. The folder publisher's source snapshots are
released before archive streaming. No compiler, native helper or project metadata enters the ZIP.

ZIP32 STORE has no compression: sorted UTF-8 paths/flag, CRC-32, fixed 1980-01-01 timestamp,
fixed attributes, no extra records/comments/directories, standard local/central/end records and
checked offsets. Identical bytes yield identical archives across source roots/timestamps. The
500 MB extracted limit includes the manifest; bounded ZIP headers are additional. ZIP64,
encryption and compression are unsupported. STORE uploads are larger than Deflate ZIPs by design.

Folder and archive are separate non-overwriting transactions. Known collisions reject before
folder staging. The archive stage is exclusively created, flushed/closed and renamed without
replacement. Failure removes only that invocation's stage, reports any cleanup path, and preserves
the valid folder. Retry rechecks original manifest identity, bytes, inventory and output-parent
boundary; it never recaptures the project or compiles. Changed/moved output requires a new build.

## User workflow

Save Scenes, Input Actions, Materials and Build Settings. Choose **File > Build Game...**, set
**Target: Web** and choose **Build Web**, then an existing output parent outside
the project and installation. Export
reads saved declarations, never the unsaved viewport or the whole asset directory. A matching
compiled template ships with the engine; no SDK, Node, Python, PowerShell or compiler is needed.

The panel reports separate folder and archive outcomes and both successful paths. **Open Output
Folder** and **Preview** remain available if ZIP creation fails; correct its error, then **Retry ZIP**.
**Show itch.io ZIP** opens the successful archive's parent with its filename identified. A new
cancelled/failed attempt does not reuse an earlier success. Result state is process-local, not history.

Choose **Preview** after a successful folder export. The installed native
helper opens the system browser at a private loopback URL. **Stop Preview** or closing the editor
stops that helper. Reopen a fresh preview after moving/changing output. Do not double-click
`index.html`: `file://` does not provide the required fetch/security behavior.

After loading, click **Start Game** to grant audio/input ownership. Fullscreen is an explicit button
and may be denied by the host. Focus/visibility loss pauses gameplay and audio; return to the canvas
to resume. Browser-reserved shortcuts remain browser-owned. Quit stops the runtime and asks for a
reload; it does not close the tab. Context loss fails closed with a reload message.

## Contract and bounds

v1.1.0 added supported static FBX and used project
[provenance records](ASSET_LIBRARY_AND_PROVENANCE.md) to the same saved-content closure.
Those records and their bundle-local notice identities are ordinary `content`; legacy notice
presentation copies remain under `Licenses`. The linked ufbx notice is always included.
Neither catalog nor shared-library paths are runtime inputs. Manifest 1 and its integrity contract,
including host-mutable HTML handling, are unchanged.

Output contains `index.html`, `moe-player.js`, `moe-player.wasm`, `web-manifest.json`,
`MoeMagicPlayer.json`, the exact declared `assets/` closure and `Licenses/`. No preview executable,
editor, SDK, source project metadata, undeclared Scene, library shelf or validation fixture ships.
Windows Player configuration **4** is the sole ordered included/startup Scene authority, with
the separate save-settings identity introduced in v1.11.0.
Windows Build Manifest **2**, Build Settings **1** and Project **1** remain unchanged.
Current v1.12.0 supports Scene **47**, ordinary prefab **41**, variant **1** and Lua API **23**.

`MoeMagicWebBuildManifest` **1** has exact fields `format`, `version`, `engineVersion`, `target`
(`web-experimental`), `runtime` (`emscripten-6.0.10-webgl2`), `files`. Records are strictly sorted by
UTF-8 identity with `path`, `kind`, positive `size`, lowercase `sha256`. Kinds are runtime,
configuration, content and notice. The manifest excludes itself. Case collisions, noncanonical
paths, case-mismatched references, reparse traversal, changed bytes and undeclared files reject.

`index.html` is the **host-mutable wrapper**, not part of the browser's immutable fetch inventory:
itch.io may inject its hosting script into the served page. The browser validates its manifest
record/schema and counts its exported size toward package bounds, but does not refetch/hash the
already-executing HTML against its pre-hosting bytes. Native template, folder, ZIP and preview
verification still cover the exact exported `index.html`; it remains at ZIP root. All other declared
files (JS/WASM, configuration, assets and notices) retain exact decoded-byte size and SHA-256 checks.
`web-manifest.json` retains its bounded fetch and strict schema/version/record validation; it does
not declare/hash itself. No transfer-size tolerance, host-specific byte allowance or global bypass
is used. Fetch checks decoded response bytes, not `Content-Length` or compressed transfer size.

Conservative upload bounds: at most **1,000 files including the manifest**, **240 UTF-8 path bytes**,
**200,000,000 bytes per file**, **500,000,000 bytes total**. Startup preloads at most **256 MiB of
content/configuration**; individual existing importer/resource limits still apply. Fetch is serial,
bounded and integrity-checked before shared synchronous runtime restoration. There is no network
fallback for undeclared assets. Memory starts at 128 MiB, grows to at most 512 MiB, with a 4 MiB
stack; these are bounds, not a promise that arbitrary content fits decoded memory.

Accelerated WebGL 2 and `EXT_color_buffer_float` are required. Actual float-target allocation
determines which existing Performance/Balanced/Quality presets are available. Unsupported saved
startup choices fall back to Performance; Quality's semantics are not weakened. Apply stages
resources and persistence before publication and preserves the previous generation on failure.
Wireframe authoring is explicitly rejected by Web export. No mobile, Safari, WebGL 1, threading,
SharedArrayBuffer or offline/file-URL support is claimed.

Standard-mapped browser controllers feed the existing logical backend/selector, deadzones and
neutral-rearm policies. Nonstandard controller maps are not guessed. Preferences use origin plus
package URL directory plus the existing profile identity. Denied/quota-limited storage is explicitly
session-only, never reported as saved. Another tab's conflicting persistent bytes are not overwritten.

## Preview confinement

`MoeMagicWebPreview.exe` binds only `127.0.0.1` on an ephemeral port with a random 128-bit path token.
It accepts only GET/HEAD with the exact Host, bounded headers and socket timeouts, and serves only
the verified manifest inventory plus the manifest. URL decoding cannot escape that allowlist.
No listing, upload, write, redirect, public bind, service, administrator reservation or shell command
is involved. WASM uses `application/wasm`. The editor's kill-on-close Job owns helper lifetime.

## itch.io

Upload the engine-generated **`<GameName>-Web-itch.zip` unchanged**; it already has `index.html`
at ZIP root. Create an HTML/browser game on itch.io, choose browser playback and click-to-play, and use a responsive
canvas with fullscreen optional. Preserve all assets and Licenses. A page embedded in a subdirectory
uses relative encoded URLs; no CDN or root-absolute project URL is emitted. Hosting may impose
additional current restrictions. Actual unchanged-ZIP itch.io upload is a mandatory v1.0.1
owner-assisted release step and is not implied by local validation or release Git authorization.
Use a draft/restricted page; no public visibility change or account automation is necessary.

## Developer publication and notices

`scripts/build.ps1 -BuildTarget Web -Configuration Release -Target Build` uses locally activated,
pinned Emscripten 6.0.10 and publishes `build/Web/Release/Template`. Its format-1
`MoeMagicWebTemplate` receipt fixes version, runtime, ten exact paths, hashes and sizes. A separate
same-input receipt permits canonical installer publication to reuse that optimized build. Installed
format-5 distribution integrity covers the receipt, all template bytes and the native preview helper.

Web retains the approved product/asset notices and linked GLM/Jolt/miniaudio/Lua/cgltf/TinyObjLoader/
stb/JSON notices. GLFW/GLEW notices are Windows-only. The template includes unmodified Emscripten
LICENSE/AUTHORS, musl COPYRIGHT and LLVM libc++/libc++abi/compiler-rt/libunwind licenses from the
pinned SDK. No vendor source is patched, shipped SDK or dependency upgrade is introduced.
