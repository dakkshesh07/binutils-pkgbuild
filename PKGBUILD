# Maintainer:  Dakkshesh <dakkshesh5@gmail.com>
# Contributor: Allan McRae <allan@archlinux.org>
# Contributor: Bartłomiej Piotrowski <bpiotrowski@archlinux.org>

# toolchain build order: linux-api-headers->glibc->binutils->gcc->glibc->binutils->gcc

pkgname=neutron-binutils
pkgver=2.38
pkgrel=5
pkgdesc='A set of programs to assemble and manipulate binary and object files'
arch=(x86_64)
url='https://www.gnu.org/software/binutils/'
license=(GPL)
groups=(base-devel)
depends=(glibc zlib libelf)
makedepends=(git)
checkdepends=(dejagnu debuginfod bc)
optdepends=('debuginfod: for debuginfod server/client functionality')
conflicts=(binutils-multilib binutils)
replaces=(binutils-multilib binutils)
provides=("binutils=${pkgver}")
options=(staticlibs !distcc !ccache)
source=("git+https://sourceware.org/git/binutils-gdb.git#branch=binutils-${pkgver/./_}-branch")
sha256sums=('SKIP')

prepare() {
  [[ ! -d binutils-gdb ]] && ln -s binutils-$pkgver binutils-gdb
  mkdir -p binutils-build

  cd binutils-gdb

  # Turn off development mode (-Werror, gas run-time checks, date in sonames)
  sed -i '/^development=/s/true/false/' bfd/development.sh
}

build() {
  cd binutils-build

  # Override makepkg.conf's flags
  LDFLAGS=${LDFLAGS/-O1/-O3}
  LDFLAGS=${LDFLAGS/-O2/-O3}

  CPPFLAGS=${CPPFLAGS/-O1/-O3}
  CPPFLAGS=${CPPFLAGS/-O2/-O3}

  CFLAGS=${CFLAGS/-O1/-O3}
  CFLAGS=${CFLAGS/-O2/-O3}

  CXXFLAGS=${CXXFLAGS/-O1/-O3}
  CXXFLAGS=${CXXFLAGS/-O2/-O3}

  "$srcdir/binutils-gdb/configure" \
    CFLAGS="-flto -O3 -pipe -ffunction-sections -fdata-sections -ffat-lto-objects" \
    CXXFLAGS="-flto -O3 -pipe -ffunction-sections -fdata-sections -ffat-lto-objects" \
    CPPFLAGS="-flto -O3 -pipe -ffunction-sections -fdata-sections -ffat-lto-objects" \
    LDFLAGS="-O3" \
    --prefix=/usr \
    --with-lib-path=/usr/lib:/usr/local/lib \
    --enable-cet \
    --enable-deterministic-archives \
    --enable-gold \
    --disable-sim \
    --enable-ld=default \
    --enable-lto \
    --enable-install-libiberty \
    --enable-plugins \
    --disable-gdbserver \
    --enable-pgo-build=lto \
    --disable-libdecnumber \
    --disable-readline \
    --enable-relro \
    --enable-targets=x86_64-pep \
    --enable-threads \
    --enable-shared \
    --disable-checking \
    --disable-gdb \
    --disable-werror \
    --with-debuginfod \
    --with-pic \
    --with-system-zlib \
    --with-pkgversion="Neutron Binutils"

  make -O -j$(nproc --all) configure-host
  make -O -j$(nproc --all) tooldir=/usr
}

check() {
  cd binutils-build

  # current testsuite failure in debuginfod (objdump)
  # https://sourceware.org/bugzilla/show_bug.cgi?id=28029
  sed -i '/test_fetch_debuglink $OBJDUMP/d' \
      $srcdir/binutils-gdb/binutils/testsuite/binutils-all/debuginfod.exp

  # Use minimal flags for testsuite
  # ld testsuite uses CFLAGS_FOR_TARGET and requires -g
  # gold testsuite requires CXXFLAGS/CFLAGS with default PIE/PIC disabled
  make -O CFLAGS_FOR_TARGET="-O3 -g" \
          CXXFLAGS="-O3 -no-pie -fno-PIC" \
          CFLAGS="-O3 -no-pie" \
          LDFLAGS="" \
          check -j$(nproc --all)
}

package() {
  cd binutils-build
  make prefix="$pkgdir/usr" tooldir="$pkgdir/usr" install -j$(nproc --all)

  # Remove unwanted files
  rm -f "$pkgdir"/usr/share/man/man1/{dlltool,nlmconv,windres,windmc}*

  # No shared linking to these files outside binutils
  rm -f "$pkgdir"/usr/lib/lib{bfd,opcodes}.so
  echo 'INPUT( /usr/lib/libbfd.a -liberty -lz -ldl )' > "$pkgdir/usr/lib/libbfd.so"
  echo 'INPUT( /usr/lib/libopcodes.a -lbfd )' > "$pkgdir/usr/lib/libopcodes.so"
}
