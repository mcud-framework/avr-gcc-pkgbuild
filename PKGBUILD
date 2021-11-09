# Maintainer: schuay <jakob.gruber@gmail.com>
# Contributor: Brad Fanella <bradfanella@archlinux.us>
# Contributor: Corrado Primier <bardo@aur.archlinux.org>
# Contributor: danst0 <danst0@west.de>

# Build order: avr-binutils -> avr-gcc -> avr-libc

_target=avr
pkgname=$_target-gcc
pkgver=12.0.0
_islver=0.24
pkgrel=1
_snapshot=e5a67458da39de7db8971cc5ec017fa6199a08c1
pkgdesc='The GNU AVR Compiler Collection'
arch=(x86_64)
license=(GPL LGPL FDL custom)
url='http://gcc.gnu.org/'
depends=($_target-binutils gcc-libs libmpc)
optdepends=('avr-libc: Standard C library for Atmel AVR development')
options=(!emptydirs !strip)
source=(
	#https://ftp.gnu.org/gnu/gcc/gcc-$pkgver/gcc-$pkgver.tar.xz{,.sig}
	gcc-$pkgver.zip::https://github.com/D-Programming-GDC/gcc/archive/$_snapshot.zip
	#ftp://gcc.gnu.org/pub/gcc/snapshots/${_snapshot}/gcc-${_snapshot}.tar.xz
	https://libisl.sourceforge.io/isl-${_islver}.tar.bz2
)
sha256sums=('993e74560e89bd7a1ca19d52ef8b8c50d3fe2db160c275d779f3e04401627e1d'
            'fcf78dd9656c10eb8cf9fbd5f59a0b6b01386205fe1934b3b287a0a1898145c0')
validpgpkeys=(D3A93CAD751C2AF4F8C7AD516C35B99309B5FA62  # Jakub Jelinek <jakub@redhat.com>
              33C235A34C46AA3FFB293709A328C3A2C3C45C06  # Jakub Jelinek <jakub@redhat.com>
              13975A70E63C361C73AE69EF6EEB81F8981C74C7) # Richard Guenther <richard.guenther@gmail.com>

if [ -n "${_snapshot}" ]; then
  _basedir=gcc-${_snapshot}
else
  _basedir=gcc-${pkgver}
fi

prepare() {
    cd ${_basedir}
}

build() {
    CFLAGS=${CFLAGS/-Wformat/}
    CFLAGS=${CFLAGS/-Werror=format-security/}
    CXXFLAGS=${CXXFLAGS/-Wformat/}
    CXXFLAGS=${CXXFLAGS/-Werror=format-security/}
    cd "${srcdir}"/${_basedir} 

    # link isl for in-tree build
    ln -s ../isl-${_islver} isl

    # https://bugs.archlinux.org/task/34629
    # hack! - some configure tests for header files using "$CPP $CPPFLAGS"
    sed -i "/ac_cpp=/s/\$CPPFLAGS/\$CPPFLAGS -O2/" {libiberty,gcc}/configure

    echo ${pkgver} > gcc/BASE-VER

    cd "${srcdir}"
    mkdir gcc-build && cd gcc-build

    export CFLAGS_FOR_TARGET='-O2 -pipe'
    export CXXFLAGS_FOR_TARGET='-O2 -pipe'

    # --disable-linker-build-id   https://bugs.archlinux.org/task/34902
    # --disable-__cxa_atexit   https://bugs.archlinux.org/task/50848
    "${srcdir}"/${_basedir}/configure \
                --disable-install-libiberty \
                --disable-libssp \
                --disable-libstdcxx-pch \
                --disable-libunwind-exceptions \
                --disable-linker-build-id \
                --disable-nls \
                --disable-werror \
                --disable-__cxa_atexit \
                --enable-checking=release \
                --enable-clocale=gnu \
                --enable-gnu-unique-object \
                --enable-gold \
                --enable-languages=c,c++,d \
                --enable-ld=default \
                --enable-lto \
                --enable-plugin \
                --enable-shared \
                --infodir=/usr/share/info \
                --libdir=/usr/lib \
                --libexecdir=/usr/lib \
                --mandir=/usr/share/man \
                --prefix=/usr \
                --target=$_target \
                --with-as=/usr/bin/$_target-as \
                --with-gnu-as \
                --with-gnu-ld \
                --with-ld=/usr/bin/$_target-ld \
                --with-plugin-ld=ld.gold \
                --with-system-zlib \
                --with-isl \
                --enable-gnu-indirect-function

    make
}

package() {
    cd "${srcdir}"/gcc-build

    make -j1 DESTDIR="${pkgdir}" install

    # Strip debug symbols from libraries; without this, the package size balloons to ~500MB.
    find "${pkgdir}"/usr/lib -type f -name "*.a" \
        -exec /usr/bin/$_target-strip --strip-debug '{}' \;

    # Install Runtime Library Exception
    install -Dm644 "${srcdir}"/${_basedir}/COPYING.RUNTIME \
        "${pkgdir}"/usr/share/licenses/$_target-gcc/RUNTIME.LIBRARY.EXCEPTION

    rm -r "${pkgdir}"/usr/share/man/man7
    rm -r "${pkgdir}"/usr/share/info
    rm "${pkgdir}"/usr/lib/libcc1.*
}
