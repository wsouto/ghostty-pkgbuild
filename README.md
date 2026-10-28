# ghostty-pkgbuild

Arch Linux PKGBUILD for [Ghostty](https://ghostty.org), built from the upstream
release tag.

> **Unofficial.** This packaging recipe is not affiliated with or endorsed by
> the Ghostty project or Arch Linux. It is not published to the AUR. Report
> Ghostty bugs upstream only if they also occur with an official build.

This repository builds and installs **Ghostty 1.4.0** from source on **Arch
Linux x86_64**. It downloads the upstream `v1.4.0` tag archive and verifies it
against a pinned SHA-256 checksum. Updates are manual; rebuilding does not
automatically select a newer release.

## Requirements

- An up-to-date Arch Linux x86_64 system.
- The `base-devel` package group and a regular user account with access to
  `sudo` for dependency installation.
- Zig **0.17.x** (`>=0.17.0` and `<0.18.0`), as required by this recipe.
- Internet access to download the source, build dependencies, and Zig packages.
- Disk space for source, dependency caches, build output, and package archives.

Install the standard packaging tools:

```sh
sudo pacman -Syu --needed base-devel
```

`makepkg --syncdeps` installs the dependencies declared in `PKGBUILD`. These
include Zig, Blueprint Compiler, Pandoc, gettext, ncurses, and pkgconf. If your
configured repositories do not provide a compatible Zig package, resolve that
version requirement before building.

Runtime dependencies include GTK 4.18 or newer, libadwaita 1.7 or newer, and
gtk4-layer-shell 1.0.4 or newer. Pacman checks these requirements when installing
the package.

## Build

Run these commands from this repository as a regular user:

```sh
makepkg --verifysource
makepkg --syncdeps --log
```

The first command downloads and checks the source archive. The second fetches
the Zig dependency cache, compiles Ghostty, and creates a package archive in
this directory. Do not run `makepkg` with `sudo`.

The recipe enables X11 and Wayland support, generates documentation, and uses
`ReleaseFast` optimization with a baseline CPU target. It stages files under
`/usr` and uses system libraries through Zig's `--system` build mode.

The main output is named `ghostty-<pkgver>-<pkgrel>-x86_64.pkg.tar.zst`, for
example:

```text
ghostty-1.4.0-3-x86_64.pkg.tar.zst
```

No debug package is produced: the `ReleaseFast` build carries no debug info, so
the recipe disables it. Build logs are written alongside the package when
`--log` is used.

## Install and run

List the output package paths and inspect the main package before installing:

```sh
makepkg --packagelist
pacman -Qip ./ghostty-1.4.0-3-x86_64.pkg.tar.zst
```

Adjust the filename if the version, package release, or compression format
changes. Building alone does not install Ghostty.

Install the package you built:

```sh
sudo pacman -U ./ghostty-1.4.0-3-x86_64.pkg.tar.zst
```

Arch's official packaging splits Ghostty into `ghostty`,
`ghostty-shell-integration`, `ghostty-terminfo`, and `ghostty-nautilus`. This
package contains all of them and declares them as conflicts, so if any are
installed, pacman offers to remove them during the install. Accept that; the
same files are provided by this package.

Alternatively, build and install in one step:

```sh
makepkg --syncdeps --install --log
```

Launch Ghostty from your desktop application menu or a terminal:

```sh
ghostty
```

Check the installed version with `ghostty --version`. The package includes
desktop integration, the `xterm-ghostty` terminfo entry (the plain
`ghostty` entry comes from `ncurses`), shell integration, completions, and manual
pages; use `man ghostty` for usage information. Install the optional
`nautilus-python` dependency if you want the included GNOME Files integration.

The package contains the Ghostty terminal only. Upstream's build also installs
libghostty-vt (shared and static library, headers, and pkg-config files), but
that library is unreleased and its API is unstable, so the recipe removes it.

## Update

To package a newer upstream release, following the
[Arch package guidelines](https://wiki.archlinux.org/title/Arch_package_guidelines):

1. Edit `PKGBUILD`: set `pkgver` to the new version and reset `pkgrel=1`. The
   source URL, archive name, and source directory are all derived from `pkgver`.
2. Read the release notes, especially "Changes for Package Maintainers", at
   `https://ghostty.org/docs/install/release-notes/<version>` (for example
   `1-4-0`). Adjust the Zig version bounds, library dependencies, and build
   options as needed. Check that the files removed in `package()` still exist.
3. Download and independently verify the new source archive, then replace
   `sha256sums` with its SHA-256 digest. Calculate the digest with
   `sha256sum ghostty-<version>.tar.gz`; a calculated digest alone does not
   establish that the source is authentic. Do not use `SKIP` or bypass checks.
4. Check the recipe syntax and generated metadata, verify the source, then
   rebuild from a clean source tree and install:

   ```sh
   bash -n PKGBUILD
   makepkg --printsrcinfo > .SRCINFO
   makepkg --verifysource
   makepkg --cleanbuild --syncdeps --install --log
   ```

For packaging changes to the same upstream version, increment `pkgrel` instead
of `pkgver`. Install the resulting archive with `pacman -U`, or use the combined
build-and-install command above. This repository does not configure a binary
package repository or an automatic update service.

Update system dependencies separately with `sudo pacman -Syu`. Rebuild this
local package when system library changes require it.

## Troubleshooting and cleanup

- **Source checksum mismatch:** stop and investigate the downloaded archive
  and its origin before changing the pinned checksum.
- **Zig version error:** check `zig version` and ensure the executable on your
  `PATH` matches the version bounds in `PKGBUILD`.
- **Build failure after an update:** retry with
  `makepkg --cleanbuild --syncdeps --log` and inspect the build log. A clean
  build recreates `src/`, including this recipe's Zig caches and staging
  directory. Preserve anything you need there before using `--cleanbuild`.

The generated `src/`, `pkg/`, source archives, package archives, and logs are
ignored by Git. Keep the package archive if you want to reinstall it without
recompiling. To uninstall Ghostty:

```sh
sudo pacman -Rns ghostty
```

Review pacman's removal transaction before confirming it.

## License

The packaging files in this repository (`PKGBUILD`, `.SRCINFO`, this README) are licensed
under the [0BSD](LICENSE) license. Ghostty itself is licensed separately by its
authors (MIT, with the GNOME Files extension under GPL-2.0-or-later); the built
package installs its license to `/usr/share/licenses/ghostty/`.
