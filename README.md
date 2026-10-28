# ProjectAD Launcher

An unofficial Windows desktop launcher UI for Minecraft: Java Edition, built
with Electron. It loads stable version names from Mojang's public version
manifest.

(FOR DEVELOPER)

## Run on Windows

- Install a current Node.js LTS release.
- Open PowerShell in the workspace root and run:

```powershell
cd .\BlockLauncher
npm.cmd install
npm.cmd start
```

To run the focused tests:

```powershell
npm.cmd test
```

To build the Windows x64 installer:

```powershell
npm.cmd run dist:win
```

The NSIS installer is written to
`dist\ProjectAD-Launcher-<version>-Setup.exe`.
It installs per user, creates Start Menu and desktop shortcuts, and preserves
launcher settings and instances if the application is uninstalled.

## Current status

- Electron desktop shell with Home, Installations, Mojang News, Accounts, and Settings views
- Full-window background using the supplied Minecraft wallpaper
- Create isolated instances for vanilla, Forge, NeoForge, Fabric, and Quilt
- Browse Mojang's official Minecraft launcher news feed and open articles on Minecraft.net
- Keep saves, mods, settings, and version files separate per instance while reusing verified common assets and libraries
- Choose an AppData instances directory or select a custom folder in Settings
- Delete individual instances and their saves, mods, and game files
- Search Modrinth and CurseForge from **Installations > Download content** for compatible mods, resource packs, and shaders, with project icons loaded from each catalog; install verified files into an instance
- Filter content by Minecraft version, mod loader, category, source, and sort order
- Configure an encrypted local CurseForge API key in Settings
- Import Modrinth `.mrpack`, CurseForge `.zip`, and ProjectAD `.pjt` modpacks from Installations; export catalog-installed mods plus resource packs, shaders, and configuration overrides in those formats
- Sign in to Microsoft through the system browser with PKCE and Windows-encrypted token storage
- Configure Minecraft RAM, a GPU preference, and Automatic or custom Java runtime
- Choose English or Bahasa Indonesia as the launcher display language
- Connect an Ely.by account with two-factor authentication and Windows-encrypted session storage
- Launch installed Vanilla and modded instances with a connected Microsoft or Ely.by account
- Manage Ely.by and local profiles with a draggable 3D Minecraft skin preview
- Local profiles cannot launch the game

The default instance library is `%APPDATA%\ProjectAD Launcher\instances`.
Each new instance is stored in a folder named after the name set in the
launcher, keeping saves, mods, settings, and version files separate. Existing
UUID-named instance folders are renamed to the name saved in the launcher the
next time the launcher opens the instance list. Instance names must be unique.
Verified Minecraft assets and libraries are shared in a launcher-managed cache
under the instance library, so later instances can reuse files instead of
downloading them again. Minecraft
releases and loader metadata are checked against Mojang and the respective
loader's official Maven or metadata service.

Installations can download many gigabytes of game files. Forge's installer
may also require Java to be available on the system. Transient network failures
are retried per asset with bounded concurrency, so a failed asset does not
restart thousands of successful downloads. Asset indexes and objects are
verified against Mojang's SHA-1 metadata, and completed files are reused when
an install is retried. Failed new installations are cleaned up; existing
instances are left untouched.

Mod, resource-pack, and shader searches use the Modrinth and CurseForge APIs.
Modrinth search works without setup; CurseForge search requires an API key
configured in Settings. The key is encrypted with Windows secure storage and
is kept out of `settings.json`. The key is not embedded in the distributed
launcher because desktop application binaries can be inspected. Downloaded
files are checked against the catalog-provided checksum and placed in the
selected instance's `mods`, `resourcepacks`, or `shaderpacks` folder.

The Installations page can import Modrinth `.mrpack`, CurseForge `.zip`, and
ProjectAD `.pjt` archives. Mod files are fetched from the original catalog and
verified; Modrinth pack HTTPS mirrors are accepted only with their manifest
SHA-512 checksum, and CurseForge imports require your configured API key. Exports contain
catalog-managed mod references rather than redistributing mod binaries, and
include files from the instance's `resourcepacks`, `shaderpacks`, `config`,
`defaultconfigs`, `scripts`, and `kubejs` directories as overrides.
On import, every file and directory under the pack's override folder is merged
into the instance game directory, including root-level options and nested
configuration files.
Instances with manually copied `.jar` mods cannot be exported until those mods
are removed or installed through the launcher catalog.

Project image assets are stored in the repository's `assets` directory and are
packaged from there with the Windows installer.
Offline player previews use the bundled `assets/steve.png` skin texture rather
than requesting an online skin for a local profile.

Launch settings are stored in the launcher's `settings.json`. The RAM limit
leaves at least 1 GB for Windows and is capped at 32 GB. Automatic Java selects
a runtime compatible with the selected Minecraft version; a custom `java.exe`
or `javaw.exe` can also be selected.
The GPU option is a preference, not a VRAM allocator: Windows and the GPU
driver control adapter selection and Minecraft cannot reserve dedicated GPU
memory through a launcher setting. These settings are saved now and will be
used when the game launches.

Mojang News comes from `https://launchercontent.mojang.com/news.json`. The
launcher shows the feed's `Last-Modified` date because Mojang may not refresh
this legacy launcher feed as frequently as its website. Article links open on
the official Minecraft.net domain.

Ely.by sign-in uses its documented HTTPS authentication endpoint and supports
two-factor codes. The launcher never stores the account password; it encrypts
the returned session token using Electron's Windows secure storage. Ely.by
accounts launch through Ely.by's Yggdrasil authentication service. The
launcher downloads authlib-injector from its official distribution endpoint
and verifies its SHA-256 checksum before use. Unauthenticated offline/cracked
login is not supported.

Microsoft sign-in uses the system browser, OAuth authorization code with PKCE,
and the Microsoft/Xbox/Minecraft authentication services. Its refresh token is
encrypted with Windows secure storage. Ely.by uses its documented HTTPS
authentication endpoint, supports two-factor codes, and stores its session
token in Windows secure storage. Local profiles are only previews and cannot
access online services.
The 3D preview renderer is the locally installed MIT-licensed `skinview3d`
package; skin textures are fetched from Ely.by's documented skin service in
the main process, validated, and passed to the renderer as image data.

## Configure Microsoft sign-in

Create a public desktop app registration for Microsoft personal accounts:

1. Open the [Microsoft Entra admin center](https://entra.microsoft.com) and
   sign in.
2. Go to **Entra ID > App registrations > New registration**.
3. Set **Supported account types** to **Personal Microsoft accounts only**
   (or an option that also includes personal Microsoft accounts).
4. Select **Register** and copy the **Application (client) ID** from Overview.
5. Under **Manage > Authentication**, choose **Add a platform > Mobile and
   desktop applications**, and add the redirect URI `http://localhost`.
6. In ProjectAD Launcher **Settings**, paste the **Application (client) ID**.
   The sign-in flow opens the system browser and returns through a temporary
   loopback listener; it does not use an embedded WebView.

Do not create a client secret for a desktop app. Do not commit credentials or
access tokens. See Microsoft's
[desktop app configuration guide](https://learn.microsoft.com/en-us/entra/identity-platform/scenario-desktop-app-configuration).

