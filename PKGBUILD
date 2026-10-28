# Maintainer: Walter Souto <357932+wsouto@users.noreply.github.com>

# Pinned-release build of the Ghostty terminal from an upstream release tag.
pkgname=ghostty
pkgver=1.4.0
pkgrel=3
pkgdesc='Fast, native, feature-rich terminal emulator'
arch=('x86_64')
url='https://ghostty.org'
license=('MIT' 'GPL-2.0-or-later')
# ReleaseFast leaves no debug info to split out, so a debug package would be empty.
options=('!debug')
# bzip2, libpng, pixman and zlib are built in by upstream, so they are not runtime deps.
depends=('fontconfig' 'libfontconfig.so'
         'freetype2' 'libfreetype.so'
         'glib2' 'libglib-2.0.so' 'libgio-2.0.so' 'libgobject-2.0.so'
         'glibc'
         'gtk4>=1:4.18' 'libgtk-4.so'
         'gtk4-layer-shell>=1.0.4'
         'harfbuzz' 'libharfbuzz.so'
         'hicolor-icon-theme'
         'libadwaita>=1:1.7' 'libadwaita-1.so'
         'libglvnd' 'libEGL.so'
         'libx11'
         'oniguruma' 'libonig.so'
         'wayland' 'libwayland-client.so')
makedepends=('zig>=0.17.0' 'zig<0.18.0' 'blueprint-compiler' 'pandoc-cli' 'gettext'
             'ncurses' 'pkgconf')
optdepends=('nautilus-python: Open in Ghostty from GNOME Files')
# Arch's official ghostty is split into these packages; this one ships them all in one.
provides=("ghostty-shell-integration=$pkgver" "ghostty-terminfo=$pkgver"
          "ghostty-nautilus=$pkgver")
conflicts=('ghostty-shell-integration' 'ghostty-terminfo' 'ghostty-nautilus')
source=("https://github.com/ghostty-org/ghostty/archive/v$pkgver/$pkgname-$pkgver.tar.gz")
sha256sums=('2e2a2516f9bea89a4991ad7c9401b77682eed366387a5b1d7c629472b5d11a43')

prepare() {
  cd "$srcdir/$pkgname-$pkgver"
  ZIG_GLOBAL_CACHE_DIR="$srcdir/zig-global-cache" \
    zig build --fetch=all --pkg-dir "$srcdir/zig-packages"
}

build() {
  cd "$srcdir/$pkgname-$pkgver"
  # A stale stage from an earlier build would leak removed files into the package.
  rm -rf "$srcdir/stage"
  ZIG_GLOBAL_CACHE_DIR="$srcdir/zig-global-cache" \
    DESTDIR="$srcdir/stage" \
    zig build \
      --summary all \
      --prefix /usr \
      --system "$srcdir/zig-packages" \
      --build-id=sha1 \
      -Doptimize=ReleaseFast \
      -Dcpu=baseline \
      -Dgtk-x11=true \
      -Dgtk-wayland=true \
      -Demit-docs=true \
      -Dinstalled-exe-path=/usr/bin/ghostty \
      -Dversion-string="$pkgver"
}

package() {
  cp -a "$srcdir/stage/." "$pkgdir/"
  # libghostty-vt is always installed by upstream but is an unreleased, unstable API;
  # this package ships the terminal only.
  rm -r "$pkgdir/usr/include/ghostty" "$pkgdir"/usr/lib/libghostty-vt.* \
    "$pkgdir"/usr/share/pkgconfig/libghostty-vt*.pc
  # ncurses >= 6.6 ships its own g/ghostty entry; TERM uses xterm-ghostty, which stays.
  rm "$pkgdir/usr/share/terminfo/g/ghostty"
  rmdir "$pkgdir/usr/share/terminfo/g"
  rmdir "$pkgdir/usr/include" "$pkgdir/usr/share/pkgconfig"
  install -Dm644 "$srcdir/$pkgname-$pkgver/LICENSE" \
    "$pkgdir/usr/share/licenses/$pkgname/LICENSE"
}
