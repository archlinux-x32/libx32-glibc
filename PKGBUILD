# $Id: PKGBUILD 163138 2016-02-22 12:25:28Z allan $
# Maintainer: Jan Alexander Steffens (heftig) <jan.steffens@gmail.com>
# Contributor: Jan de Groot <jgc@archlinux.org>
# Contributor: Allan McRae <allan@archlinux.org>

# toolchain build order: linux-api-headers->glibc->binutils->gcc->binutils->glibc
# NOTE: valgrind-multilib requires rebuild with each major glibc version

pkgname=lib32-glibc
pkgver=2.23
pkgrel=1
_commit=e742928c
pkgdesc="GNU C Library (32-bit)"
arch=('x86_64')
url="http://www.gnu.org/software/libc"
license=('GPL' 'LGPL')
groups=()
depends=()
makedepends=('gcc-multilib>=5.2' 'git')
backup=()


options=('!strip' 'staticlibs' '!emptydirs')

source=(git://sourceware.org/git/glibc.git#commit=${_commit}
        lib32-glibc.conf)

md5sums=('SKIP'
         '6e052f1cb693d5d3203f50f9d4e8c33b')

prepare() {
  mkdir glibc-build
}

build() {
  cd glibc-build

  #if [[ ${CARCH} = "i686" ]]; then
    # Hack to fix NPTL issues with Xen, only required on 32bit platforms
    # TODO: make separate glibc-xen package for i686
    export CFLAGS="${CFLAGS} -mno-tls-direct-seg-refs"
  #fi

  echo "slibdir=/usr/lib32" >> configparms
  echo "rtlddir=/usr/lib32" >> configparms
  echo "sbindir=/usr/bin" >> configparms
  echo "rootsbindir=/usr/bin" >> configparms

  export CC="gcc -m32"
  export CXX="g++ -m32"

  # remove hardening options for building libraries
  CFLAGS=${CFLAGS/-fstack-protector-strong/}
  CPPFLAGS=${CPPFLAGS/-D_FORTIFY_SOURCE=2/}

  ../glibc/configure --prefix=/usr \
      --libdir=/usr/lib32 --libexecdir=/usr/lib32 \
      --with-headers=/usr/include \
      --with-bugurl=https://bugs.archlinux.org/ \
      --enable-add-ons \
      --enable-obsolete-rpc \
      --enable-kernel=2.6.32 \
      --enable-bind-now --disable-profile \
      --enable-stackguard-randomization \
      --enable-lock-elision \
      --enable-multi-arch \
      --disable-werror \
      i686-unknown-linux-gnu

  # build libraries with hardening disabled
  echo "build-programs=no" >> configparms
  make

  # re-enable hardening for programs
  sed -i "/build-programs=/s#no#yes#" configparms
  echo "CC += -fstack-protector-strong -D_FORTIFY_SOURCE=2" >> configparms
  echo "CXX += -fstack-protector-strong -D_FORTIFY_SOURCE=2" >> configparms
  make

  # remove harding in preparation to run test-suite
  sed -i '/FORTIFY/d' configparms
}

check() {
  cd glibc-build

  # some failures are "expected"
  make check || true
}

package() {
  cd glibc-build

  make install_root=${pkgdir} install

  rm -rf ${pkgdir}/{etc,sbin,usr/{bin,sbin,share},var}

  # We need to keep 32 bit specific header files
  find ${pkgdir}/usr/include -type f -not -name '*-32.h' -delete

  # Dynamic linker
  mkdir ${pkgdir}/usr/lib
  ln -s ../lib32/ld-linux.so.2 ${pkgdir}/usr/lib/

  # Add lib32 paths to the default library search path
  install -Dm644 "$srcdir/lib32-glibc.conf" "$pkgdir/etc/ld.so.conf.d/lib32-glibc.conf"

  # Symlink /usr/lib32/locale to /usr/lib/locale
  ln -s ../lib/locale "$pkgdir/usr/lib32/locale"

  # remove the static libraries that have a shared counterpart
  # libc, libdl, libm and libpthread are required for toolchain testsuites
  # in addition libcrypt appears widely required
  rm $pkgdir/usr/lib32/lib{anl,BrokenLocale,nsl,resolv,rt,util}.a

  # Do not strip the following files for improved debugging support
  # ("improved" as in not breaking gdb and valgrind...):
  #   ld-${pkgver}.so
  #   libc-${pkgver}.so
  #   libpthread-${pkgver}.so
  #   libthread_db-1.0.so

  cd $pkgdir
  strip $STRIP_BINARIES \
                        \
                        \
                        usr/lib32/getconf/*




  strip $STRIP_STATIC usr/lib32/*.a

  strip $STRIP_SHARED usr/lib32/lib{anl,BrokenLocale,cidn,crypt}-*.so \
                      usr/lib32/libnss_{compat,db,dns,files,hesiod,nis,nisplus}-*.so \
                      usr/lib32/lib{dl,m,nsl,resolv,rt,util}-*.so \
                      usr/lib32/lib{memusage,pcprofile,SegFault}.so \
                      usr/lib32/{audit,gconv}/*.so || true



}
