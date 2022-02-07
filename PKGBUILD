# Maintainer:  Dakkshesh <dakkshesh5@gmail.com>
# Contributor: Allan McRae <allan@archlinux.org>
# Contributor: Bartłomiej Piotrowski <bpiotrowski@archlinux.org>

# toolchain build order: linux-api-headers->glibc->binutils->gcc->binutils->glibc

pkgname=neutron-binutils
pkgver=2.37
pkgrel=1
pkgdesc='A set of programs to assemble and manipulate binary and object files'
arch=(x86_64)
url='https://www.gnu.org/software/binutils/'
license=(GPL)
groups=(base-devel)
depends=(glibc zlib elfutils)
makedepends=(elfutils git)
conflicts=(binutils-multilib)
replaces=(binutils-multilib)
provides=("binutils=${pkgver}")
options=(staticlibs !distcc !ccache)
source=(https://ftp.gnu.org/gnu/binutils/binutils-$pkgver.tar.xz)
sha256sums=('820d9724f020a3e69cb337893a0b63c2db161dadcb0e06fc11dc29eb1e84a32c')

prepare() {
  [[ ! -d binutils-gdb ]] && ln -s binutils-$pkgver binutils-gdb
  mkdir -p binutils-build

  cd binutils-gdb

  # Turn off development mode (-Werror, gas run-time checks, date in sonames)
  sed -i '/^development=/s/true/false/' bfd/development.sh
}

build() {
  cd binutils-build

  "$srcdir/binutils-gdb/configure" \
    CFLAGS="-flto -O3 -pipe -ffunction-sections -fdata-sections" \
    CXXFLAGS="-flto -O3 -pipe -ffunction-sections -fdata-sections" \
    --prefix=/usr \
    --with-lib-path=/usr/lib:/usr/local/lib \
    --enable-cet \
    --enable-deterministic-archives \
    --enable-gold \
    --disable-sim \
    --enable-ld=default \
    --enable-lto \
    --enable-plugins \
    --disable-gdbserver \
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

  make configure-host
  make tooldir=/usr
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
          check || true
}

package() {
  cd binutils-build
  make prefix="$pkgdir/usr" tooldir="$pkgdir/usr" install

  # Remove unwanted files
  rm -f "$pkgdir"/usr/share/man/man1/{dlltool,nlmconv,windres,windmc}*

  # No shared linking to these files outside binutils
  rm -f "$pkgdir"/usr/lib/lib{bfd,opcodes}.so
  echo 'INPUT( /usr/lib/libbfd.a -liberty -lz -ldl )' > "$pkgdir/usr/lib/libbfd.so"
  echo 'INPUT( /usr/lib/libopcodes.a -lbfd )' > "$pkgdir/usr/lib/libopcodes.so"
}
