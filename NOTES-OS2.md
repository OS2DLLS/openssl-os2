### OS/2 (EMX)

This source tree includes a project-specific `OS2-EMX` target in
`Configurations/90-os2.conf`. Upstream OpenSSL removed OS/2 support in 3.0, so
this target is not an upstream-supported platform. It builds the OpenSSL
libraries as OS/2 DLLs; the target currently disables the command-line
applications and test suite. It requires an EMX GCC toolchain with OMF support
(`-Zomf`, `-Zsym`, `-Zmap`), Perl with the OpenSSL build prerequisites, make,
and pthread headers/libraries.

The pthread installation prefix can be set with `OS2EMX_PTHREAD_ROOT` and must
contain `include/pthread.h` and `lib/`. If unset, the native OS/2 build uses
`/@unixroot/usr`; a Linux cross-build uses
`$OS2EMX_OPENSSL_BUILD_ROOT/deps/pthread/@unixroot/usr` when that environment
variable is set.

#### Build natively on OS/2

Run these commands from an OS/2 shell with the EMX compiler and an
OMF-compatible archiver installed. Adjust `SRC` and the pthread prefix for the
local installation; use a fresh, separate build directory.

```sh
SRC=/path/to/openssl-3.6.5
BUILD=/path/to/openssl-os2-build
export OS2EMX_PTHREAD_ROOT=/@unixroot/usr
mkdir -p "$BUILD"
cd "$BUILD"
perl "$SRC/Configure" OS2-EMX --prefix=/usr --openssldir=/etc/ssl
make -j2 AR="$OS2EMX_AR" ARFLAGS="-p512 qc" RANLIB=:
```

The native `ar` selected by the toolchain must create archives compatible with
OMF objects. If the default archiver is not OMF-aware, pass the native OMF
archiver as `AR` and a compatible no-op as `RANLIB` to `make`. The OS/2 target
does not enable `make test`; validate the resulting DLLs with a native OS/2
consumer instead.

#### Cross-compile from Linux

Enter the project's Nix shell (the `shell.nix` environment defines
`OS2EMX_PREFIX`, `OS2EMX_SYSROOT`, `OS2EMX_TARGET`, `OS2EMX_CC`, `OS2EMX_AR`,
`OS2EMX_RANLIB`, `OS2EMX_OPENSSL_SOURCE`, and
`OS2EMX_OPENSSL_BUILD_ROOT`). Use an empty build directory and keep host build
tools separate from the OS/2 compiler and target sysroot.

```sh
nix-shell /path/to/shell.nix
SRC="$OS2EMX_OPENSSL_SOURCE"
BUILD="$OS2EMX_OPENSSL_BUILD_ROOT"
mkdir -p "$BUILD"
cd "$BUILD"

perl "$SRC/Configure" OS2-EMX \
    --cross-compile-prefix="$OS2EMX_PREFIX/bin/$OS2EMX_TARGET-" \
    --prefix=/usr --openssldir=/etc/ssl

make -j2 AR="$OS2EMX_AR" ARFLAGS="-p512 -qc" RANLIB="$OS2EMX_RANLIB" DESTDIR="$OS2EMX_SYSROOT" install_sw
```

The cross-compile prefix selects the target GCC; `AR` and `RANLIB` are set
explicitly because EMX uses an OMF archiver and GNU `ranlib` must not process
its archives. `DESTDIR` stages the installation under the target sysroot
without installing into the Linux host. A successful Linux build or staged
install does not by itself prove that the DLLs load or run on OS/2; perform a
separate OS/2 runtime smoke test.

Specific notes on upgrading to OpenSSL 3.x from previous versions can be found
in the [ossl-guide-migration(7ossl)] manual page.

