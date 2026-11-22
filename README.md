# VSCodium for OpenIndiana (illumos)

VSCodium 1.143.17232 (VS Code 1.143.1) for OpenIndiana Hipster, as three
[oi-userland](https://github.com/OpenIndiana/oi-userland) components:

- **`editor/vscodium`**: the desktop editor on our Electron 43 (oi-electron).
- **`editor/vscodium-server`**: the remote/web server (VS Code's REH target)
  on Node.js 24, with `vscodium-web` to open the editor in a browser.
- **`editor/vscodium/extension/vscode-amix`**
  (`components/editor/vscodium-amix`): the AMIX development extension as a
  built-in extension of the desktop editor, packaged from its release
  `.vsix` (amix-userland).

The editor and the server are built from the pristine VSCodium and VS Code release tarballs, the
same way FreeBSD's `editors/vscode` builds them.

## What works

- The desktop editor: integrated terminal (node-pty), file watching
  (@parcel/watcher with illumos FEN), search (ripgrep), extensions from
  Open VSX.
- The server, both for VSCodium's remote connection and in a browser
  (`vscodium-web`).

## Layout

| path | what |
|---|---|
| `components/editor/vscodium-server/patches/` | VS Code source patches: a sunos REH target, platform detection, the Microsoft authentication extension, webviews served locally, no git in postinstall, JSON printf, telemetry and system ripgrep |
| `components/editor/vscodium-server/npm-patches/` | patches to npm modules: node-pty (illumos PTYs), @parcel/watcher (FEN backend), kerberos, tree-sitter, @vscode/deviceid. They use patch-package's file naming and go onto every copy of that module version before its native code is compiled. |
| `components/editor/vscodium/patches/` | the same patch set (symbolic links to vscodium-server's), plus `17-desktop-sunos` for the desktop build |
| `components/editor/vscodium/npm-patches/` | the desktop's module patches: gulp-electron (our Electron from the download cache), native-keymap, policy-watcher |
| `components/editor/*/files/` | `vscodium-web` and its desktop file |
| `*.p5m` | the package manifests |

## Rebuilding on OpenIndiana

**For the desktop editor:**
1. Install `developer/electron-43` (oi-electron), `runtime/nodejs-24` and
   `text/ripgrep`.
2. Copy both components into an oi-userland tree.
3. Build `editor/vscodium-server` first: the desktop build reuses its
   patches.
4. Build `editor/vscodium`.

In each component directory, `gmake env-prep` installs the build
dependencies; then run `gmake publish`. Each build takes well under an hour,
mostly npm.

**Moving to another VSCodium version:**
- Change `COMPONENT_VERSION`, `VSCODE_VERSION`, `VSCODE_COMMIT` and the two
  archive hashes, in both Makefiles: a VSCodium client asks for a server of
  its own commit.
- Then rebase the patches.

## Upstreaming

These pieces are useful beyond this port:
- the FEN backend for @parcel/watcher;
- the illumos support in node-pty;
- the sunos target in VS Code's build.

## Licence

The build recipes and scripts in this repository (Makefiles, manifests,
shell and Python scripts) are under the CDDL 1.0, as oi-userland
components are: see `LICENSE`.

Exceptions:

- `patches/` and `npm-patches/` are offered under the licence of the project they patch: VS Code (MIT), and each npm module's own licence (node-pty MIT, @parcel/watcher MIT, …).

The software that gets built keeps its own licences: VSCodium and VS Code (MIT).
