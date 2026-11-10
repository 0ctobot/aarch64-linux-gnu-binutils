# Maintainer: Anatol Pomozov <anatol.pomozov@gmail.com>
# Contributor: Adam W. Willis <return.of.octobot@gmail.com> (binutils trunk)

# binutils-gdb master, to go with Neutrino GCC. The source is a git archive of
# $_commit; make it with:
#   git -C ~/toolchains/src/binutils archive --format=tar \
#     --prefix=binutils-trunk-${_commit:0:8}/ $_commit | zstd -T4 -10 \
#     -o binutils-trunk-${_commit:0:8}.tar.zst
# Built by the Neutrino GCC host compiler in $_hostcc, linking libstdc++ and
# libgcc statically so that the package doesn't depend on it at run time.

_target=aarch64-linux-gnu
pkgname=$_target-binutils-trunk
_commit=892d98146139de38662c1f5b7d48fc3a606ec3f6
_basever=2.47.50
_datestamp=20261110
pkgver=$_basever.r$_datestamp.g${_commit:0:8}
_pkgver="${pkgver}"
pkgrel=1
pkgdesc='A set of programs to assemble and manipulate binary and object files for the ARM64 target (binutils trunk)'
arch=(x86_64)
url='https://sourceware.org/binutils/'
license=(GPL)
depends=(zlib libelf zstd)
provides=("$_target-binutils=$_basever")
conflicts=($_target-binutils)
source=(binutils-trunk-${_commit:0:8}.tar.zst)
sha256sums=('ee6d7ea5a5eeea9a1c957d4442d63f8d170936c52757a6b009e24adacf4cfd4d')

_basedir=binutils-trunk-${_commit:0:8}
_hostcc=$HOME/toolchains/gcc-17-host

prepare() {
  cd $_basedir

  grep -q "BFD_VERSION_DATE $_datestamp" bfd/version.h ||
    { echo "bfd/version.h doesn't match the PKGBUILD"; return 1; }
  "$_hostcc"/bin/gcc --version | head -1 | grep -q 'Neutrino GCC' ||
    { echo "$_hostcc isn't a Neutrino GCC host compiler"; return 1; }

  sed -i "/ac_cpp=/s/\$CPPFLAGS/\$CPPFLAGS -O2/" libiberty/configure
  # Turn off development mode (-Werror, gas run-time checks, date in sonames)
  sed -i '/^development=/s/true/false/' bfd/development.sh

}

build() {
  cd $_basedir

  export CC="$_hostcc"/bin/gcc CXX="$_hostcc"/bin/g++
  LDFLAGS+=" -static-libstdc++ -static-libgcc"

  # The git tree also holds gdb and its libraries, which the release
  # tarballs leave out
  ./configure --target=$_target \
              --with-sysroot=/usr/$_target \
              --prefix=/usr \
              --disable-gprofng \
              --disable-multilib \
              --with-gnu-as \
              --with-gnu-ld \
              --disable-nls \
              --enable-gold \
              --enable-ld=default \
              --enable-plugins \
              --enable-deterministic-archives \
              --enable-new-dtags \
              --with-system-zlib --with-zstd \
              --disable-gdb --disable-gdbserver --disable-sim \
              --disable-readline --disable-libdecnumber \
              --disable-werror \
              "--with-pkgversion=GNU Binutils trunk ${_commit:0:8}"

  make -j8
}

# No check(): the testsuite needs DejaGnu (runtest), which isn't installed

package() {
  cd $_basedir

  make DESTDIR="$pkgdir" install

  # The host compiler is only needed to build the package
  if find "$pkgdir"/usr/bin "$pkgdir"/usr/$_target/bin -type f -executable \
       -exec ldd '{}' \; 2>/dev/null | grep -E 'not found|gcc-17-host'; then
    echo "binaries depend on $_hostcc"; return 1
  fi

  # Remove file conflicting with host binutils and manpages for MS Windows tools
  rm "$pkgdir"/usr/share/man/man1/$_target-{dlltool,windres,windmc}*
  rm "$pkgdir"/usr/lib/bfd-plugins/libdep.so

  # Remove info documents that conflict with host version
  rm -r "$pkgdir"/usr/share/info
}
