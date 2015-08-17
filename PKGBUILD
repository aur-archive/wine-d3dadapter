# Maintainer: smirky <smirky (at) smirky (dot) net>
# Based on Archlinux stock wine package with the d3dadapter patch

pkgname=wine-d3dadapter
_pkgname=wine
pkgver=1.7.41
pkgrel=1

_pkgbasever=${pkgver/rc/-rc}

source=(http://prdownloads.sourceforge.net/$_pkgname/$_pkgname-$_pkgbasever.tar.bz2{,.sign}
        30-win32-aliases.conf
	d3d9.patch)
sha1sums=('e31aceb246f3f75416f8bf5b59c668c02d221b17'
          'SKIP'
          '023a5c901c6a091c56e76b6a62d141d87cce9fdb'
	  '0340031f8165e9d06d229308d1b15e23d83eba41')
validpgpkeys=(5AC1A08B03BD7A313E0A955AF5E6E9EEB9461DD7)

pkgdesc="A compatibility layer for running Windows programs (with d3dadapter patch)"
url="http://www.winehq.com"
arch=(i686 x86_64)
options=(staticlibs)
license=(LGPL)
install=wine.install

_depends=(
  fontconfig      lib32-fontconfig
  libxcursor      lib32-libxcursor
  libxrandr       lib32-libxrandr
  libxdamage      lib32-libxdamage
  libxi           lib32-libxi
  gettext         lib32-gettext
  freetype2       lib32-freetype2
  glu             lib32-glu
  libsm           lib32-libsm
  gcc-libs        lib32-gcc-libs
  libpcap         lib32-libpcap
  desktop-file-utils
)

makedepends=(autoconf ncurses bison perl fontforge flex prelink
  'gcc>=4.5.0-2'  'gcc-multilib>=4.5.0-2'
  giflib          lib32-giflib
  libpng          lib32-libpng
  gnutls          lib32-gnutls
  libxinerama     lib32-libxinerama
  libxcomposite   lib32-libxcomposite
  libxmu          lib32-libxmu
  libxxf86vm      lib32-libxxf86vm
  libxml2         lib32-libxml2
  libldap         lib32-libldap
  lcms2           lib32-lcms2
  mpg123          lib32-mpg123
  openal          lib32-openal
  v4l-utils       lib32-v4l-utils
  alsa-lib        lib32-alsa-lib
  libxcomposite   lib32-libxcomposite
  mesa            lib32-mesa
  mesa-libgl      lib32-mesa-libgl
  libcl           lib32-libcl
  libxslt         lib32-libxslt
  samba
  opencl-headers
  dri2proto
)
  
optdepends=(
  giflib          lib32-giflib
  libpng          lib32-libpng
  libldap         lib32-libldap
  gnutls          lib32-gnutls
  lcms2           lib32-lcms2
  libxml2         lib32-libxml2
  mpg123          lib32-mpg123
  openal          lib32-openal
  v4l-utils       lib32-v4l-utils
  libpulse        lib32-libpulse
  alsa-plugins    lib32-alsa-plugins
  alsa-lib        lib32-alsa-lib
  libjpeg-turbo   lib32-libjpeg-turbo
  libxcomposite   lib32-libxcomposite
  libxinerama     lib32-libxinerama
  ncurses         lib32-ncurses
  libcl           lib32-libcl
  libxslt         lib32-libxslt
  cups
  samba           dosbox
)

if [[ $CARCH == i686 ]]; then
  # Strip lib32 etc. on i686
  _depends=(${_depends[@]/*32-*/})
  makedepends=(${makedepends[@]/*32-*/} ${_depends[@]})
  makedepends=(${makedepends[@]/*-multilib*/})
  optdepends=(${optdepends[@]/*32-*/})
else
  makedepends=(${makedepends[@]} ${_depends[@]})
  provides=("bin32-wine=$pkgver" "wine-wow64=$pkgver" "wine=$pkgver")
  conflicts=('bin32-wine' 'wine-wow64' 'wine')
  replaces=('bin32-wine')
fi

prepare() {
  cd $_pkgname-$_pkgbasever

  # This patch enables d3dadapter to use with the
  # gallium-nine state tracker. _DO_NOT_ replace 
  # mesa-libgl/lib32-mesa-libgl with nvidia's, it won't
  # build because of missing functions.  This patch is
  # accuired using the following:
  #
  # git clone git://github.com/ixit/wine.git
  # cd wine
  # git remote add upstream git://source.winehq.org/git/wine.git
  # git fetch upstream
  # git merge --no-edit upstream/master
  # git diff upstream/master > ../d3d9.patch

  patch -p1 -i "${srcdir}/d3d9.patch"
}

build() {
  cd "$srcdir"

  # Allow ccache to work
  mv $_pkgname-$_pkgbasever $_pkgname

  # Get rid of old build dirs
  rm -rf $_pkgname-{32,64}-build
  mkdir $_pkgname-32-build

  # These additional CPPFLAGS solve FS#27662 and FS#34195
  export CPPFLAGS="${CPPFLAGS/-D_FORTIFY_SOURCE=2/} -D_FORTIFY_SOURCE=0"

  if [[ $CARCH == x86_64 ]]; then
    msg2 "Building Wine-64..."

    mkdir $_pkgname-64-build
    cd "$srcdir/$_pkgname-64-build"
    ../$_pkgname/configure \
      --prefix=/usr \
      --libdir=/usr/lib \
      --with-x \
      --with-d3dadapter \
      --without-gstreamer \
      --enable-win64 \
      --disable-tests
    # Gstreamer was disabled for FS#33655

    make

    _wine32opts=(
      --libdir=/usr/lib32
      --with-wine64="$srcdir/$_pkgname-64-build"
    )

    export PKG_CONFIG_PATH="/usr/lib32/pkgconfig"
  fi

  msg2 "Building Wine-32..."
  cd "$srcdir/$_pkgname-32-build"
  ../$_pkgname/configure \
    --prefix=/usr \
    --with-x \
    --with-d3dadapter \
    --without-gstreamer \
    --disable-tests \
    "${_wine32opts[@]}"

  # These additional flags solve FS#23277
  make CFLAGS+="-mstackrealign -mincoming-stack-boundary=2" CXXFLAGS+="-mstackrealign -mincoming-stack-boundary=2"
}

package() {
  depends=(${_depends[@]})

  msg2 "Packaging Wine-32..."
  cd "$srcdir/$_pkgname-32-build"

  if [[ $CARCH == i686 ]]; then
    make prefix="$pkgdir/usr" install
  else
    make prefix="$pkgdir/usr" \
      libdir="$pkgdir/usr/lib32" \
      dlldir="$pkgdir/usr/lib32/wine" install

    msg2 "Packaging Wine-64..."
    cd "$srcdir/$_pkgname-64-build"
    make prefix="$pkgdir/usr" \
      libdir="$pkgdir/usr/lib" \
      dlldir="$pkgdir/usr/lib/wine" install
  fi

  # Font aliasing settings for Win32 applications
  install -d "$pkgdir"/etc/fonts/conf.{avail,d}
  install -m644 "$srcdir/30-win32-aliases.conf" "$pkgdir/etc/fonts/conf.avail"
  ln -s ../conf.avail/30-win32-aliases.conf "$pkgdir/etc/fonts/conf.d/30-win32-aliases.conf"
}

# vim:set ts=8 sts=2 sw=2 et:
