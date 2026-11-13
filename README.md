# Kyro for Windows

Kyro for Windows is an Electron desktop client for [Kyro](https://kyro-chat.cyou). It opens the live Kyro service in a dedicated window and uses the existing account, messaging, community, voice, and screen-sharing features. An internet connection is required.

This repository contains the desktop application source and the Electron Builder configuration used to create the Windows installer. Dependencies and generated outputs are intentionally excluded.

## Requirements

- Windows 10 or 11 on x64
- Node.js 24 and npm
- Internet access when installing dependencies and when Electron Builder downloads its Electron and NSIS tooling

The installer is unsigned unless a maintainer supplies a code-signing certificate through Electron Builder's supported signing environment.

## Install dependencies

From the repository root, install the exact dependency tree recorded in `package-lock.json`:

```powershell
npm ci
```

Do not commit the generated `node_modules/` directory.

## Develop and test

Compile the TypeScript source:

```powershell
npm run build
```

Run the included security tests after compiling:

```powershell
npm test
```

Start the desktop client in development mode:

```powershell
npm start
```

The desktop client loads the live Kyro website. Starting it does not run a local copy of the Kyro web service.

## Build the Windows installer

On Windows, run:

```powershell
npm ci
npm run dist:win
```

The `dist:win` script compiles `src/` into `dist/`, then runs Electron Builder for the Windows x64 NSIS target. The installer is written to:

```text
release/Kyro-Setup-1.0.1-x64.exe
```

The build command includes `--publish never`, so it does not upload or publish the installer. To create an unpacked application directory instead, run:

```powershell
npm run pack
```

Both `dist/` and `release/` are generated build outputs and are excluded from source control. The committed lockfile provides a repeatable dependency installation; the project does not claim that separate installer builds are byte-for-byte identical.

The Electron Builder configuration in `package.json` creates a per-user, assisted NSIS installer with an installation-directory chooser and desktop and Start menu shortcuts. Its artifact name is `Kyro-Setup-${version}-${arch}.${ext}`.

## Security and behavior

- Only `https://kyro-chat.cyou` and `https://www.kyro-chat.cyou` can navigate the main window. External HTTP(S) and email links open in the default browser. File, executable, and custom OS protocols are blocked.
- The remote renderer is sandboxed, with context isolation, no Node integration, no webviews, and normal web security enabled.
- Microphone, camera, and notifications require native consent for the Kyro main frame. Screen sharing uses a separate local picker and a narrowly scoped IPC bridge.
- Credentials and browser storage live in Electron's per-user profile. No API keys or service credentials are required by this desktop source.
- Downloads show a Save dialog. The application does not execute downloaded files.
- Closing the window quits the application. There is no tray mode, startup registration, or automatic binary updater.

The 1.0.1 desktop client accepts both official HTTPS Kyro origins and provides local recovery screens for temporary connection and certificate failures. See [RELEASE_NOTES.md](RELEASE_NOTES.md) for the release details.

## Release handling

Never commit signing credentials, user profiles, environment files, `node_modules/`, `dist/`, `release/`, installer executables, or other generated artifacts. Generate and distribute checksums with release binaries through the project-approved release channel.
