# Maintainer: Eli Schwartz <eschwartz@archlinux.org>
# Contributor: Dave Reisner <d@falconindy.com>
# Contributor: Thomas Dziedzic < gostrc at gmail >
# Contributor: godane <slaxemulator@gmail.com.com>
# Contributor: Andres Perera <aepd87@gmail.com>

pkgname=pacman-git
pkgver=6.0.0.r0.g75eb3f4c
pkgrel=1
pkgdesc="A library-based package manager with dependency support"
arch=('i686' 'x86_64' 'arm' 'armv6h' 'armv7h' 'aarch64')
url="https://www.archlinux.org/pacman/"
license=('GPL')
depends=('archlinux-keyring' 'bash' 'curl' 'gpgme' 'libarchive'
         'pacman-mirrorlist')
optdepends=('pacman-contrib: various helper utilities'
            'perl-locale-gettext: translation support in makepkg-template')
makedepends=('git' 'asciidoc' 'doxygen' 'meson')
checkdepends=('python' 'fakechroot')
provides=("pacman=${pkgver%.*.*}" 'libalpm.so')
conflicts=('pacman')
backup=("etc/pacman.conf"
        "etc/makepkg.conf")
options=('emptydirs' 'strip' 'debug')
source=("git+https://git.archlinux.org/pacman.git"
        "pacman.conf.i686"
        "pacman.conf.x86_64"
        "pacman.conf.arm"
        "makepkg.conf")
sha256sums=('SKIP'
            '07b4e78745b9c9ecd93b703649b24b05803941f02ac2142ef62d50e36cb865a7'
            '4f349704aee808873bef4759a56d92e7985c3f9bffdd7b00bfaa988110124208'
            '6185dc65b18d1d085f65281c6bd1ce556466a64bf883d1f27c89b7e620570334'
            '806b40ba78eacedd96d733c1fa1eeae4b1f7398992976e22cf72e22563ab9c7a')

pkgver() {
  cd pacman

  git describe --long --tags | sed 's/^v//;s/\([^-]*-g\)/r\1/;s/-/./g'
}

prepare() {
  cd pacman
  ### Patching sources
  local src
  for src in "${source[@]}"; do
    src="${src%%::*}"
    src="${src##*/}"
    [[ $src = *.patch ]] || continue
    msg2 "Applying patch $src..."
    patch -Np1 < "../$src"
  done
}

build() {
  mkdir -p pacman/build
  cd pacman/build

  meson --prefix=/usr \
        --buildtype=plain \
        -Ddoc=enabled \
        -Ddoxygen=enabled \
        -Duse-git-version=true \
        -Dscriptlet-shell=/usr/bin/bash \
        -Dldconfig=/usr/bin/ldconfig \
        ..
  ninja
}

check() {
  cd pacman/build

  ninja test
}

package() {
  cd pacman/build

  DESTDIR="$pkgdir" ninja install

  # install Arch specific stuff
  install -dm755 "$pkgdir/etc"
  if [[ $CARCH =~ arm* || $CARCH = aarch64 ]]; then
    # $CARCH != uname -m
    sed -e "s|@CARCH[@]|$CARCH|g" "$srcdir/pacman.conf.arm" \
      | install -m644 /dev/stdin "$pkgdir/etc/pacman.conf"
  else
    install -m644 "$srcdir/pacman.conf.$CARCH" "$pkgdir/etc/pacman.conf"
  fi

  # set things correctly in the default conf file
  case $CARCH in
    i686)
      mychost="i686-pc-linux-gnu"
      myflags="-march=i686"
      ;;
    x86_64)
      mychost="x86_64-pc-linux-gnu"
      myflags="-march=x86-64"
      ;;
    arm)
      mychost="armv5tel-unknown-linux-gnueabi"
      myflags="-march=armv5te"
      ;;
    armv6h)
      mychost="armv6l-unknown-linux-gnueabihf"
      myflags="-march=armv6 -mfloat-abi=hard -mfpu=vfp"
      ;;
    armv7h)
      mychost="armv7l-unknown-linux-gnueabihf"
      myflags="-march=armv7-a -mfloat-abi=hard -mfpu=vfpv3-d16"
      ;;
    aarch64)
      mychost="aarch64-unknown-linux-gnu"
      myflags="-march=armv8-a"
      ;;
  esac

  # set things correctly in the default conf file
  install -m644 "$srcdir/makepkg.conf" "$pkgdir/etc"
  sed -i "$pkgdir/etc/makepkg.conf" \
    -e "s|@CARCH[@]|$CARCH|g" \
    -e "s|@CHOST[@]|$mychost|g" \
    -e "s|@CARCHFLAGS[@]|$myflags|g"
}

# vim: set ts=2 sw=2 et:
