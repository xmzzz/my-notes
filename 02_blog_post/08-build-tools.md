# install rpm without root

```
$ yum search libXXX
$ mkdir ~/rpm && cd ~/rpm
$ yumdownloader --destdir ~/rpm/ --resolve XXX
$ for i in *.rpm; do rpm2cpio $i | cpio -idv; done
```

> [非root安装软件](https://www.cnblogs.com/Basasuya/p/11535055.html) 
> https://stackoverflow.com/questions/36651091/how-to-install-packages-in-linux-centos-without-root-user-with-automatic-depen

# openssl 构建安装

```
$ cd ~/work/
$ git clone git://git.openssl.org/openssl.git
$ cd openssl/
$ mkdir build
$ mkdir -p ~/install/openssl/ssl
$ ../Configure --prefix=$HOME/install/openssl --openssldir=$HOME/install/openssl/ssl
$ make
$ make test
$ make install
$ export LD_LIBRARY_PATH=$HOME/install/openssl/lib64
$ export PATH=$HOME/install/openssl/bin:$PATH
```

> https://github.com/openssl/openssl/blob/master/INSTALL.md

# cmake 构建安装

```
$ cd ~/work/
$ wget https://github.com/Kitware/CMake/releases/download/v3.26.4/cmake-3.26.4.tar.gz
$ tar -xf cmake-3.26.4.tar.gz
$ cd cmake-3.26.4
$ mkdir build
$ cd build/
$ mkdir ~/install/cmake
$ ../bootstrap --prefix=$HOME/install/cmake
$ make
$ make install
$ export PATH=$HOME/install/cmake/bin:$PATH
$ cmake --version
```

> https://cmake.org/download/

# llvm/clang 构建安装

## llvm-project

```
$ cd ~/work/
$ git clone https://github.com/llvm/llvm-project.git
$ cd llvm-project/
$ # git tag -l | less
$ # git checkout llvmorg-16.0.6
$ mkdir build
$ cd build
$ mkdir ~/install/llvm
$ pip3.7 install --user -U recommonmark
$ cmake -DLLVM_ENABLE_PROJECTS="clang;clang-tools-extra" -DLLVM_TARGETS_TO_BUILD="X86;RISCV;BPF" -DCMAKE_INSTALL_PREFIX=$HOME/install/llvm -DCMAKE_BUILD_TYPE=Debug -DLLVM_BUILD_DOCS=ON -DLLVM_ENABLE_SPHINX=ON -DSPHINX_WARNINGS_AS_ERRORS=OFF -DCMAKE_C_COMPILER=$HOME/install/gcc20230621/bin/gcc -DCMAKE_CXX_COMPILER=$HOME/install/gcc20230621/bin/g++ -DLLVM_PARALLEL_LINK_JOBS=1 -Wno-dev -G Ninja ../llvm
$ ninja
$ # ninja docs-llvm-html  docs-llvm-man
$ # ninja docs-clang-html docs-clang-man
$ ninja install
$ vim ~/.bashrc
$ source ~/.bashrc
$ clang -v
```

## llvm lld

```
$ mkdir build_lld && cd build_lld
$ cmake -DCMAKE_BUILD_TYPE=Debug -DLLVM_ENABLE_PROJECTS=lld -DCMAKE_INSTALL_PREFIX=$HOME/install/llvm -DCMAKE_C_COMPILER=$HOME/install/gcc20230621/bin/gcc -DCMAKE_CXX_COMPILER=$HOME/install/gcc20230621/bin/g++ -DLLVM_PARALLEL_LINK_JOBS=1 -G Ninja ../llvm
$ ninja
$ ninja install
```

> https://llvm.org/docs/GettingStarted.html#checkout
> https://www.linuxfromscratch.org/blfs/view/svn/general/llvm.html
> https://lld.llvm.org/
> https://discourse.llvm.org/t/parallel-build-fails-for-buffer-overflow-detected-error/52861
> https://discourse.llvm.org/t/host-compiler-does-not-support-fuse-ld-lld-error-when-running-cmake-command-that-used-to-work/68008/2
> https://clang.llvm.org/get_started.html
> https://blog.csdn.net/RubiaC/article/details/124821717

# qemu 8.0 构建安装

```
$ cd ~/work/
$ # wget https://download.qemu.org/qemu-8.0.2.tar.xz
$ # tar xvJf qemu-8.0.2.tar.xz
$ git clone https://gitlab.com/qemu-project/qemu.git
$ cd qemu/
$ git submodule init
$ git submodule update --recursive
$ mkdir build
$ cd build/
$ ../configure --help
$ mkdir ~/install/qemu
$ pip3.7 install --user -U sphinx sphinx-rtd-theme ninja
$ #../configure --enable-slirp --enable-virtfs --enable-bpf --enable-kvm --enable-libssh --enable-libusb --enable-numa --enable-opengl --enable-selinux --enable-vde --enable-vhost-kernel --enable-vhost-user --enable-vhost-crypto --enable-debug --enable-sdl --prefix=$HOME/install/qemu --target-list=riscv32-softmmu,riscv64-softmmu,arm-softmmu,aarch64-softmmu,x86_64-softmmu,aarch64-linux-user,arm-linux-user,riscv32-linux-user,riscv64-linux-user,x86_64-linux-user --cc=$HOME/install/llvm/bin/clang --cxx=$HOME/install/llvm/bin/clang++ --ninja=NINJA
$ #../configure --enable-slirp --enable-virtfs --extra-ldflags="-fuse-ld=lld" --prefix=$HOME/install/qemu --target-list=riscv32-softmmu,riscv64-softmmu,arm-softmmu,aarch64-softmmu,x86_64-softmmu,aarch64-linux-user,arm-linux-user,riscv32-linux-user,riscv64-linux-user,x86_64-linux-user --cc=$HOME/install/llvm10/bin/clang --cxx=$HOME/install/llvm10/bin/clang++ --ninja=$HOME/work/ninja/ninja > LOG_bash 2>&1
$ ../configure --enable-slirp --enable-kvm --extra-ldflags="-fuse-ld=lld" --prefix=$HOME/install/qemu-8.0.2 --target-list=riscv32-softmmu,riscv64-softmmu,arm-softmmu,aarch64-softmmu,x86_64-softmmu,aarch64-linux-user,arm-linux-user,riscv32-linux-user,riscv64-linux-user,x86_64-linux-user --cc=$HOME/install/llvm10/bin/clang --cxx=$HOME/install/llvm10/bin/clang++ --ninja=$HOME/work/ninja/ninja > LOG_configure 2>&1
$ # meson 0.61.5 版本可正常编译qemu，1.1.99高版本会报错
$ ../configure --enable-slirp --enable-kvm --audio-drv-list=jack,oss,pa,sdl --enable-gcrypt --enable-gtk --enable-opengl --enable-sdl --enable-selinux --enable-vnc --enable-pa --extra-cflags="-I$HOME/rpm/usr/include/json-c -I$HOME/rpm/usr/include/gtk-1.2 -I$HOME/rpm/usr/include/spice-server" --extra-ldflags="-L$HOME/rpm/usr/lib64 -fuse-ld=lld" --prefix=$HOME/install/qemu-8.0.2 --target-list=riscv32-softmmu,riscv64-softmmu,arm-softmmu,aarch64-softmmu,x86_64-softmmu,aarch64-linux-user,arm-linux-user,riscv32-linux-user,riscv64-linux-user,x86_64-linux-user --cc=$HOME/install/llvm10/bin/clang --cxx=$HOME/install/llvm10/bin/clang++ --ninja=$HOME/work/ninja/ninja --enable-dbus-display --enable-curl --enable-gnutls --enable-gtk-clipboard --enable-libnfs --enable-libusb --enable-numa --enable-rbd --enable-rdma --enable-sdl-image --enable-vdi --enable-vfio-user-server --enable-virtfs --enable-vnc --enable-vte --enable-zstd --enable-modules --enable-vde > LOG_configure 2>&1
$ ../configure --enable-slirp --enable-kvm --audio-drv-list=jack,oss,pa,sdl --enable-gcrypt \
--enable-gtk --enable-opengl --enable-sdl --enable-selinux --enable-vnc --enable-pa \
--enable-dbus-display --enable-curl --enable-gnutls --enable-gtk-clipboard --enable-libnfs \
--enable-libusb --enable-numa --enable-rbd --enable-rdma --enable-sdl-image --enable-vdi \
--enable-vfio-user-server --enable-virtfs --enable-vnc --enable-vte --enable-zstd --enable-vde --enable-glusterfs \
--extra-cflags="-I$HOME/rpm/usr/include/json-c -I$HOME/rpm/usr/include/fuse3 -I$HOME/rpm/usr/include -I$HOME/rpm/usr/include/gtk-3.0 -I$HOME/rpm/usr/include/spice-server -I$HOME/rpm/usr/include/vte-2.91 -I$HOME/rpm/usr/include/virgl -I$HOME/rpm/usr/include/pango-1.0 -I$HOME/rpm/usr/include/cacard -I$HOME/rpm/usr/include/cairo -I$HOME/rpm/usr/include/gdk-pixbuf-2.0 -I$HOME/rpm/usr/include/atk-1.0" \
--extra-ldflags="-L$HOME/rpm/usr/lib64 -fuse-ld=lld" --prefix=$HOME/install/qemu-8.0.2 \
--target-list=riscv32-softmmu,riscv64-softmmu,arm-softmmu,aarch64-softmmu,x86_64-softmmu,aarch64-linux-user,arm-linux-user,riscv32-linux-user,riscv64-linux-user,x86_64-linux-user \
--cc=$HOME/install/llvm10/bin/clang --cxx=$HOME/install/llvm10/bin/clang++ --ninja=$HOME/work/ninja/ninja > ../LOG_configure 2>&1
```

- 依赖

```
gtk+-devel libXft-devel.x86_64 cairo-devel.x86_64 gdk-pixbuf2-devel.x86_64 \
libtiff-devel libXinerama-devel libXi-devel libXrandr-devel libXcomposite-devel \
wayland-devel wayland-protocols-devel libxkbcommon-devel atk-devel at-spi2-atk-devel egl-wayland egl-wayland-devel \
at-spi2-core-devel dbus-devel dbus-libs libXtst-devel \
acpica-tools libaio-devel libnfs-devel libseccomp-devel \
alsa-tools-devel jack-audio-connection-kit-devel spice-protocol spice-server libiscsi-devel zstd-devel \
virglrenderer virglrenderer-devel curl-devel qt5-qtwayland qt5-qtwayland-devel \
python3-pyudev libvirt-daemon-driver-storage-mpath ncurses-devel libvirt-devel libvirt-libs SDL2-devel librados-devel librbd-devel \
glusterfs-devel libzip-devel libzip mesa-libgbm-devel gnutls-devel vte291-devel openjpeg-devel openjpeg libjpeg-turbo-devel \
cyrus-sasl-devel pam-devel snappy-devel lzo-devel numactl-devel rdma-core-devel libcacard-devel libusb-devel \
usbredir-devel usbredir daxctl daxctl-devel keyutils-libs-devel fuse3-devel fuse3 libdwarf-devel \
librados-devel librbd-devel nettle-devel libtasn1-devel gnutls-devel numactl-libs p11-kit-devel rdma-core \
libusb systemd-devel systemd-libs libcap-ng-devel libattr-devel json-c-devel elfutils-devel snappy pam cyrus-sasl valgrind-devel \
valgrind blktrace libaio ocaml-brlapi glusterfs-devel libtasn1 rpcgen libnfs vte291
```

> https://github.com/qemu/qemu
> https://www.qemu.org/download/
> https://www.linuxfromscratch.org/blfs/view/svn/postlfs/qemu.html
> https://www.sphinx-doc.org/en/master/usage/installation.html
> https://ninja-build.org/

# binutils

> https://www.gnu.org/software/binutils/
> https://mirrors.sjtug.sjtu.edu.cn/gnu/binutils/

# gcc 构建安装

```
$ cd ~/work/
$ #git clone git://gcc.gnu.org/git/gcc.git
$ git clone git@github.com:gcc-mirror/gcc.git
$ mkdir gcc-build && cd gcc-build
$ mkdir ~/install/gcc20230621
$ unset LIBRARY_PATH
$ ../configure --with-pkgversion=20230621 --host=x86_64-linux-gnu --prefix=$HOME/install/gcc20230621/ --disable-multilib --with-gmp=$HOME/install/gmp/ --with-mpfr=$HOME/install/mpfr/ --with-mpc=$HOME/install/mpc/ --enable-languages=c,c++
$ make
$ make install
$ vim ~/.bashrc
$ g++ -v
```

> https://gcc.gnu.org/install/
> https://marc.info/?l=gcc-fortran&m=128924831902835&w=2
> https://gcc.gnu.org/bugzilla/show_bug.cgi?id=91902
> https://gcc.gnu.org/onlinedocs/cpp/Environment-Variables.html

# GMP 构建安装

```
$ cd ~/work/
$ mkdir GMP && cd GMP
$ wget https://gmplib.org/download/gmp/gmp-6.2.1.tar.xz
$ tar -xf gmp-6.2.1.tar.xz
$ cd gmp-6.2.1
$ mkdir build && cd build
$ mkdir ~/install/gmp
$ ../configure --prefix=$HOME/install/gmp/
$ make
$ make check
$ make install
$ vim ~/.bashrc
```

> https://gcc.gnu.org/install/prerequisites.html
> https://gmplib.org/

# MPFR 构建安装

```
$ cd ~/work/
$ mkdir MPFR && cd MPFR
$ wget https://www.mpfr.org/mpfr-current/mpfr-4.2.0.tar.xz
$ tar -xf mpfr-4.2.0.tar.xz
$ cd mpfr-4.2.0
$ wget --no-config https://www.mpfr.org/mpfr-4.2.0/allpatches
$ patch -N -Z -p1 < ./allpatches
$ mkdir build && cd build/
$ mkdir ~/install/mpfr
$ ../configure --prefix=$HOME/install/mpfr/ --with-gmp=$HOME/install/gmp/
$ make
$ make check
$ make install
```

> https://gcc.gnu.org/install/prerequisites.html
> https://www.mpfr.org/

# MPC 构建安装

```
$ cd ~/work/
$ mkdir MPC && cd MPC
$ wget https://ftp.gnu.org/gnu/mpc/mpc-1.3.1.tar.gz
$ tar -xf mpc-1.3.1.tar.gz
$ cd mpc-1.3.1
$ mkdir build && cd build
$ unset $LD_LIBRARY_PATH
$ mkdir ~/install/mpc
$ ../configure --prefix=$HOME/install/mpc/ --with-gmp=$HOME/install/gmp/ --with-mpfr=$HOME/install/mpfr/
$ make
$ make check
$ make install

```

> https://gcc.gnu.org/install/prerequisites.html
> https://www.multiprecision.org/mpc/

# isl

```

```

> https://gcc.gnu.org/install/prerequisites.html
> https://gcc.gnu.org/pub/gcc/infrastructure/

# binutils + gcc for riscv

```
$ cd binutils-2.38
$ mkdir build && cd build
$ ../configure --target=riscv64-linux --enable-targets=all --prefix=/home/mingzheng/install/binutils-cross --enable-ld
$ make -j25 > LOG_make
$ make install
$ cd ~/work/gcc
$ git checkout releases/gcc-12.3.0
$ mkdir ../build-gcc-cross && cd ../build-gcc-cross
$ ../gcc/configure --target=riscv64-linux --prefix=/home/mingzheng/install/binutils-cross --disable-multilib --with-gmp=/home/mingzheng/install/gmp/ --with-mpfr=/home/mingzheng/install/mpfr/ --with-mpc=/home/mingzheng/install/mpc/ --with-isl=/home/mingzheng/install/isl/ --enable-languages=c --disable-bootstrap --disable-nls --disable-threads --disable-shared --disable-libmudflap --disable-libssp --disable-libgomp --disable-decimal-float --disable-libquadmath --disable-libatomic --disable-libcc1 --disable-libmpx
$ make -j25 > LOG_make 2>&1
$ make install
```

- binutils-gdb 只编译 binutils

```
$ # configure时增加参数： --disable-gdb --disable-libdecnumber --disable-readline --disable-sim
$ ../configure --target=riscv64-linux --enable-targets=all --prefix=/home/mingzheng/install/binutils-cross --enable-ld --disable-gdb --disable-libdecnumber --disable-readline --disable-sim
```

> https://inbox.sourceware.org/binutils/CAMe9rOoO1Cf-RCCXKiWATJt-z8g5=rRbx2i6sq6sSbdnhyRspg@mail.gmail.com/T/

# gdb-13.2

```

```

> https://sourceware.org/gdb/
> https://sourceware.org/gdb/onlinedocs/gdb/Configure-Options.html

# Ninja 构建安装

```
$ cd ~/work/
$ git clone git://github.com/ninja-build/ninja.git && cd ninja
$ ./configure.py -h
$ ./configure.py --bootstrap
$ ./ninja --version
$ vim ~/.bashrc
$ source ~/.bashrc

```

> https://ninja-build.org/
> https://github.com/ninja-build/ninja

# re2c 构建安装

```
$ cd ~/work/
$ git clone git@github.com:skvadrik/re2c.git && cd re2c
$ autoreconf -i -W all
$ mkdir cmakebuild && cd cmakebuild
$ mkdir $HOME/install/re2c
$ cmake -DCMAKE_INSTALL_PREFIX=$HOME/install/re2c/ -DCMAKE_BUILD_TYPE=Release ..
$ cmake --build .
$ cmake --build . --target install
$ vim ~/.bashrc
$ source ~/.bashrc

```

> https://re2c.org/build/build.html
> https://github.com/skvadrik/re2c
> https://blog.csdn.net/yetyongjin/article/details/108050490

# glib 构建安装

```
$ cd ~/work/
$ git clone git@github.com:GNOME/glib.git
$ cd glib/
$ vim INSTALL.md
$ mkdir ~/install/glib
$ pkg-config --modversion libpcre2-8
$ export PKG_CONFIG_PATH="$HOME/install/pcre2/lib/pkgconfig:$PKG_CONFIG_PATH"
$ CC=clang CXX=clang++ meson _build_clang --prefix=$HOME/install/glib/
$ ninja -C _build_clang/
$ ninja -C _build_clang/ install
$ vim ~/.bashrc
$ export PATH="$HOME/install/glib/bin:$PATH"
$ export LD_LIBRARY_PATH="$HOME/install/glib/lib64:$LD_LIBRARY_PATH"
$ export C_INCLUDE_PATH="$HOME/install/glib/include:$C_INCLUDE_PATH"

```

> https://github.com/GNOME/glib
> https://github.com/mesonbuild/meson/issues/1752
> https://docs.gtk.org/glib/

# meson 安装

```
$ cd ~/work/
$ git clone git@github.com:mesonbuild/meson.git
$ cd meson
$ ./meson.py -v
$ ./packaging/create_zipapp.py --outfile meson --interpreter '/usr/bin/env python3.7'
$ ./meson -v
$ mkdir build
$ mv meson build/
$ vim ~/.bashrc
$ export PATH="$HOME/work/meson/build:$PATH"
```

> https://mesonbuild.com/Getting-meson_zh.html
> [meson构建工程示例](https://blog.csdn.net/weixin_43360707/article/details/124585293)
> [meson使用](https://blog.csdn.net/linggang_123/article/details/118573164)

# pcre2 构建安装

```
$ cd ~/work
$ git clone https://github.com/PhilipHazel/pcre2.git && cd pcre2
$ ./autogen.sh
$ mkdir build && cd build
$ mkdir ~/install/pcre2
$ CFLAGS='-O2 -Wall' ../configure --prefix=$HOME/install/pcre2/ --enable-pcre2-16 --enable-pcre2-32
$ make
$ make install
$ vim ~/.bashrc
$ export PATH="$HOME/install/pcre2/bin:$PATH"
$ export LD_LIBRARY_PATH="$HOME/install/pcre2/lib:$LD_LIBRARY_PATH"
$ export C_INCLUDE_PATH="$HOME/install/pcre2/include:$C_INCLUDE_PATH"
$ pkg-config --modversion libpcre2-8
$ export PKG_CONFIG_PATH="$HOME/install/pcre2/lib/pkgconfig:$PKG_CONFIG_PATH"
```

> http://www.pcre.org/
> https://www.cnblogs.com/chuck-study/p/15247627.html
> https://askubuntu.com/questions/881513/error-while-install-vte-eg-no-package-libpcre2-8-found

# selinux 构建安装

```
$ cd ~/work/
$ git clone git@github.com:SELinuxProject/selinux.git
$ cd selinux
$ vim README.md
$ make clean distclean
$ mkdir ~/install/selinux
$ CFLAGS="-Wno-error" LDFLAGS="-L$HOME/install/bzip2/lib -L$HOME/install/audit-userspace/lib" cc=$HOME/install/gcc20230621/bin/gcc make DESTDIR=~/install/selinux/ install install-rubywrap install-pywrap
```

> https://askubuntu.com/questions/386315/how-to-add-libraries-path-to-the-configure-command
> https://github.com/SELinuxProject/selinux
> https://gcc.gnu.org/onlinedocs/gcc-4.1.2/gcc/Warning-Options.html

# bzip2 构建安装

```
$ cd ~/work/
$ git clone git://sourceware.org/git/bzip2.git
$ cd bzip2
$ vim README
$ mkdir ~/install/bzip2
$ make
$ make install PREFIX=$HOME/install/bzip2/
$ ~/install/bzip2/bin/bzip2 --version
$ vim ~/.bashrc
$ export PATH="$HOME/install/bzip2/bin:$PATH"
$ export LD_LIBRARY_PATH="$HOME/install/bzip2/lib:$LD_LIBRARY_PATH"
$ export C_INCLUDE_PATH="$HOME/install/bzip2/include:$C_INCLUDE_PATH"
$ make distclean
$ vim README
$ make -f Makefile-libbz2_so
$ cp libbz2.so.1.0* ~/install/bzip2/lib/
```

> https://sourceware.org/bzip2/downloads.html
> https://askubuntu.com/questions/498082/what-to-do-with-recompile-with-fpic-message

# audit-userspace

```
$ cd ~/work/
$ git clone git@github.com:linux-audit/audit-userspace.git
$ cd audit-userspace
$ vim INSTALL.tmp
$ ./autogen.sh
$ mkdir build && cd build
$ mkdir ~/install/audit-userspace
$ LDFLAGS=-L$HOME/install/openldap/lib ../configure --prefix=$HOME/install/audit-userspace/
$ make
$ make install
$ vim ~/.bashrc
$ export PATH_AUDIT_USERSPACE="$HOME/install/audit-userspace"
$ export PATH="$PATH_AUDIT_USERSPACE/bin:$PATH"
$ export LD_LIBRARY_PATH="$PATH_AUDIT_USERSPACE/lib:$LD_LIBRARY_PATH"
$ export C_INCLUDE_PATH="$PATH_AUDIT_USERSPACE/include:$C_INCLUDE_PATH"
$ export PKG_CONFIG_PATH="$PATH_AUDIT_USERSPACE/lib/pkgconfig:$PKG_CONFIG_PATH" 
```

> https://github.com/linux-audit/audit-userspace/tree/master

# openldap

```
$ cd ~/work/
$ mkdir openldap && cd openldap
$ wget https://www.openldap.org/software/download/OpenLDAP/openldap-release/openldap-2.5.14.tgz
$ gunzip openldap-2.5.14.tgz
$ tar -xf openldap-2.5.14.tar
$ cd openldap-2.5.14
$ vim INSTALL
$ ./configure --help
$ mkdir ~/install/openldap
$ mkdir mybuild && cd mybuild
$ ../configure --prefix=$HOME/install/openldap/
$ make depend
$ make
$ make test > LOG_maketest 2>&1
$ make install
$ vim ~/.bashrc
$ export PATH="$HOME/install/openldap/bin:$PATH"
$ export LD_LIBRARY_PATH="$HOME/install/openldap/lib:$LD_LIBRARY_PATH"
$ export C_INCLUDE_PATH="$HOME/install/openldap/include:$C_INCLUDE_PATH"
$ export PKG_CONFIG_PATH="$HOME/install/openldap/lib/pkgconfig:$PKG_CONFIG_PATH"
```

> https://www.openldap.org/
> https://www.openldap.org/software/download/

# swig

```
$ cd ~/work/
$ mkdir swig && cd swig
$ wget https://nchc.dl.sourceforge.net/project/swig/swig/swig-4.1.1/swig-4.1.1.tar.gz
$ tar -xf swig-4.1.1.tar.gz
$ cd swig-4.1.1
$ mkdir build && cd build
$ mkdir ~/install/swig
$ ../configure --prefix=$HOME/install/swig/
$ make
$ make install
$ vim ~/.bashrc
$ export PATH_SWIG="$HOME/install/swig"
$ export PATH="$PATH_SWIG/bin:$PATH"
```

> https://www.swig.org/download.html
> https://www.dev2qa.com/how-to-install-swig-on-macos-linux-and-windows/#google_vignette

# xmlto

```
$ cd ~/work/
$ mkdir xmlto && cd xmlto
$ wget https://releases.pagure.org/xmlto/xmlto-0.0.28.tar.bz2
$ tar -xvjf xmlto-0.0.28.tar.bz2 
$ cd xmlto-0.0.28
$ mkdir ~/install/xmlto
$ mkdir build && cd build
$ ../configure --prefix=$HOME/install/xmlto/
$ make
$ make install
$ vim ~/.bashrc
$ export PATH_XMLTO="$HOME/install/xmlto"
$ export PATH="$PATH_XMLTO/bin:$PATH"

```

> https://www.linuxfromscratch.org/blfs/view/svn/pst/xmlto.html

# pixman 构建安装

```
$ cd ~/work/
$ git clone https://gitlab.freedesktop.org/pixman/pixman.git
$ cd pixman/
$ mkdir build
$ ./autogen.sh
$ make distclean
$ cd build
$ ../configure --prefix=$HOME/install/pixman/
$ make
$ make install
$ vim ~/.bashrc
$ export LD_LIBRARY_PATH="$HOME/install/pixman/lib:$LD_LIBRARY_PATH"
$ export PKG_CONFIG_PATH="$HOME/install/pixman/lib/pkgconfig:$PKG_CONFIG_PATH"
$ export C_INCLUDE_PATH="$HOME/install/pixman/include:$C_INCLUDE_PATH"
```

> https://gitlab.freedesktop.org/pixman/pixman

# libssh 构建安装

```
$ cd ~/work/
$ git clone https://git.libssh.org/projects/libssh.git
$ cd libssh/
$ mkdir build
$ mkdir ~/install/libssh
$ cd build
$ export PKG_CONFIG_PATH="$HOME/install/openssl/lib64/pkgconfig:$PKG_CONFIG_PATH"  # vim ~/.bashrc
$ export CMAKE_PREFIX_PATH=$HOME/install/cmocka
$ cmake -DUNIT_TESTING=ON -DCMAKE_INSTALL_PREFIX=$HOME/install/libssh/ -DCMAKE_BUILD_TYPE=Debug -DWITH_ZLIB=OFF ..
$ make
$ make install
$ vim ~/.bashrc
$ export LD_LIBRARY_PATH="$HOME/install/libssh/lib64:$LD_LIBRARY_PATH"
$ export PKG_CONFIG_PATH="$HOME/install/libssh/lib64/pkgconfig:$PKG_CONFIG_PATH"
$ export C_INCLUDE_PATH="$HOME/install/libssh/include:$C_INCLUDE_PATH"
```

> https://www.libssh.org/get-it/
> https://github.com/build-trust/ockam/issues/330

# cmocka 构建安装

```
$ cd ~/work/
$ mkdir cmocka && cd cmocka
$ wget https://cmocka.org/files/1.1/cmocka-1.1.7.tar.xz
$ tar -xf cmocka-1.1.7.tar.xz
$ cd cmocka-1.1.7
$ mkdir ~/install/cmocka
$ mkdir build && cd build
$ cmake -DCMAKE_INSTALL_PREFIX=$HOME/install/cmocka/ -DCMAKE_BUILD_TYPE=Debug ..
$ make
$ make install
$ vim ~/.bashrc
$ export LD_LIBRARY_PATH="$HOME/install/cmocka/lib:$LD_LIBRARY_PATH"
$ export PKG_CONFIG_PATH="$HOME/install/cmocka/lib/pkgconfig:$PKG_CONFIG_PATH"
$ export C_INCLUDE_PATH="$HOME/install/cmocka/include:$C_INCLUDE_PATH"
```

> https://cmocka.org/
> https://cmocka.org/files/1.1/
> https://github.com/build-trust/ockam/issues/330

# libepoxy 构建安装

```
$ cd ~/work/
$ git clone git@github.com:anholt/libepoxy.git
$ cd libepoxy
$ mkdir ~/install/libepoxy
$ CC=clang CXX=clang++ meson _build_clang --prefix=$HOME/install/libepoxy/
$ cd _build_clang/
$ ninja
$ ninja install
$ vim ~/.bashrc
$ export LD_LIBRARY_PATH="$HOME/install/libepoxy/lib64:$LD_LIBRARY_PATH"
$ export C_INCLUDE_PATH="$HOME/install/libepoxy/include:$C_INCLUDE_PATH"
$ export PKG_CONFIG_PATH="$HOME/install/libepoxy/lib64/pkgconfig:$PKG_CONFIG_PATH"
```

> https://github.com/anholt/libepoxy
> https://github.com/mesonbuild/meson/issues/1752

# X11 构建安装

```
$ cd ~/work/
$ git clone git@github.com:mirror/libX11.git
$ cd libX11/
$ ./autogen.sh
$ mkdir ~/install/libX11
$ ./configure --prefix=$HOME/install/libX11/
$ make
$ make install
$ vim ~/.bashrc 
$ export LD_LIBRARY_PATH="$HOME/install/libX11/lib:$LD_LIBRARY_PATH"
$ export C_INCLUDE_PATH="$HOME/install/libX11/include:$C_INCLUDE_PATH"
$ export PKG_CONFIG_PATH="$HOME/install/libX11/lib/pkgconfig:$PKG_CONFIG_PATH"

```

> https://github.com/mirror/libX11

# autoconf 构建安装

```
$ cd ~/work/
$ git clone git://git.sv.gnu.org/autoconf
$ cd autoconf/
$ ./bootstrap
$ mkdir build && cd build
$ mkdir ~/install/autoconf
$ ../configure --prefix=$HOME/install/autoconf/
$ make
$ make install
$ vim ~/.bashrc
$ export PATH="$HOME/install/autoconf/bin:$PATH"
$ which autoconf
```

> https://www.gnu.org/software/autoconf/
> https://www.gnu.org/software/automake/manual/html_node/Macro-Search-Path.html
> [LIBTOOL is undefined 解决方法](https://blog.csdn.net/cynic_liu/article/details/81533064)

# help2man 构建安装

```
$ cd ~/work/
$ mkdir help2man && cd help2man
$ wget https://mirrors.nju.edu.cn/gnu/help2man/help2man-1.49.3.tar.xz
$ tar -xf help2man-1.49.3.tar.xz
$ cd help2man-1.49.3
$ mkdir ~/install/help2man
$ mkdir build && cd build
$ ../configure --prefix=$HOME/install/help2man/
$ make
$ make install
$ vim ~/.bashrc
$ export PATH="$HOME/install/help2man/bin:$PATH"
$ help2man --version
```

> https://www.gnu.org/software/help2man/#Availability

# xorg-macros

```
$ cd ~/work/
$ git clone https://gitlab.freedesktop.org/xorg/util/macros.git
$ cd macros
$ ./autogen.sh
$ make distclean
$ make build && cd build
$ mkdir ~/install/xorg-macros
$ ../configure --prefix=$HOME/install/xorg-macros/
$ make
$ make install
$ vim ~/.bashrc
$ export PKG_CONFIG_PATH="$HOME/install/xorg-macros/share/pkgconfig:$PKG_CONFIG_PATH"
$ export ACLOCAL_PATH=$HOME/install/xorg-macros/share/aclocal:$ACLOCAL_PATH
```

> https://github.com/mirror/libX11/issues/5
> https://gitlab.freedesktop.org/xorg/util/macros

# xtrans

```
$ cd ~/work/
$ git clone git@github.com:deepin-community/xtrans.git
$ cd xtrans
$ mkdir ~/install/xtrans
$ mkdir build && cd build
$ which autoreconf
$ autoreconf -f -i ../
$ ../configure --prefix=$HOME/install/xtrans/
$ make
$ make install
$ vim ~/.bashrc
$ export PKG_CONFIG_PATH="$HOME/install/xtrans/share/pkgconfig:$PKG_CONFIG_PATH"
$ export ACLOCAL_PATH="$HOME/install/xtrans/share/aclocal:$ACLOCAL_PATH"
```

> https://github.com/deepin-community/xtrans/tree/master

# xorgproto | xproto | xextproto | inputproto

```
$ cd ~/work/
$ # git clone https://gitlab.freedesktop.org/xorg/proto/xproto.git # 过时
$ git clone git://anongit.freedesktop.org/git/xorg/proto/xorgproto
$ cd xorgproto/
$ ./autogen.sh
$ mkdir ~/install/xorgproto
$ ./configure --prefix=$HOME/install/xorgproto/
$ make
$ make install
$ vim ~/.bashrc
$ export PKG_CONFIG_PATH="$HOME/install/xorgproto/share/pkgconfig:$PKG_CONFIG_PATH"
```

# xcb

```
$ cd ~/work/
$ mkdir xcb && cd xcb
$ wget https://xcb.freedesktop.org/dist/xcb-proto-1.15.2.tar.xz
$ tar -xf xcb-proto-1.15.2.tar.xz 
$ cd xcb-proto-1.15.2
$ mkdir build && cd build
$ mkdir ~/install/xcb
$ ../configure --prefix=$HOME/install/xcb/
$ make
$ make install
$ export PKG_CONFIG_PATH="$HOME/install/xcb/share/pkgconfig:$PKG_CONFIG_PATH"

```

> https://xcb.freedesktop.org/dist/
> https://xcb.freedesktop.org/

# libxcb

```
$ cd ~/work/xcb/
$ wget https://xcb.freedesktop.org/dist/libxcb-1.15.tar.xz
$ tar -xf libxcb-1.15.tar.xz
$ cd libxcb-1.15
$ mkdir build && cd build
$ mkdir ~/install/libxcb
$ ../configure --prefix=$HOME/install/libxcb/
$ make
$ make install
$ vim ~/.bashrc
$ export LD_LIBRARY_PATH="$HOME/install/libxcb/lib:$LD_LIBRARY_PATH"
$ export C_INCLUDE_PATH="$HOME/install/libxcb/include:$C_INCLUDE_PATH"
$ export PKG_CONFIG_PATH="$HOME/install/libxcb/lib/pkgconfig:$PKG_CONFIG_PATH"

```

# xau

```
$ cd ~/work/
$ git clone https://gitlab.freedesktop.org/xorg/lib/libxau.git
$ cd libxau/
$ ./autogen.sh 
$ mkdir ~/install/libxau
$ ./configure --prefix=$HOME/install/libxau
$ make
$ make install
$ vim ~/.bashrc
$ export PKG_CONFIG_PATH="$HOME/install/libxau/lib/pkgconfig:$PKG_CONFIG_PATH"

```

> https://gitlab.freedesktop.org/xorg/lib/libxau

# EGL-Registry

```
$ cd ~/work/
$ git clone git@github.com:KhronosGroup/EGL-Registry.git
$ cd EGL-Registry/api/EGL
$ ls
eglext.h       egl.h          eglplatform.h
$ vim ~/.bashrc
$ export C_INCLUDE_PATH="$HOME/work/EGL-Registry/api:$C_INCLUDE_PATH"
```

> https://github.com/KhronosGroup/EGL-Registry/tree/main
> https://github.com/KhronosGroup/EGL-Registry/blob/main/api/EGL/eglplatform.h

# dockbook-xml

```
$ cd ~/work/
$ mkdir docbook-xml && cd docbook-xml/
$ wget https://www.docbook.org/xml/4.5/docbook-xml-4.5.zip
$ # unzip docbook-xml-4.5.zip 
$ wget https://docbook.org/xml/5.0/docbook-5.0.zip
$ unzip docbook-5.0.zip
$ cd docbook-5.0
$ ls
$ cd ..
$ wget https://github.com/docbook/xslt10-stylesheets/releases/download/release/1.79.2/docbook-xsl-1.79.2.zip

```

> https://github.com/docbook/xslt10-stylesheets
> https://tdg.docbook.org/tdg/5.0/appa.html#s.stylesheetinstall
> http://www.oasis-open.org/docbook/
> https://stackoverflow.com/questions/13519203/git-compiling-documentation-git-add-xml-does-not-validate/18028336#18028336
> http://www.sagehill.net/docbookxsl/ToolsSetup.html
> https://github.com/docbook/xslt10-stylesheets/blob/master/building.md

# libxslt

```
$ cd ~/work/
$ git clone https://gitlab.gnome.org/GNOME/libxslt.git
$ cd libxslt/
$ mkdir ~/install/libxslt
$ aclocal
$ autoheader
$ autoconf
$ automake
$ ls /usr/lib64/pkgconfig/
libcrypt.pc          python-2.7-debug.pc  python2-debug.pc     python-debug.pc      
libxcrypt.pc         python-2.7.pc        python2.pc           python.pc
$ export PKG_CONFIG_PATH="/usr/lib64/pkgconfig:/usr/share/pkgconfig:$PKG_CONFIG_PATH"
$ vim ~/.bashrc
$ ./autogen.sh --prefix=$HOME/install/libxslt/
$ ./configure --prefix=$HOME/install/libxslt/ > LOG_configure2  2>&1
$ make
$ make install
$ vim ~/.bashrc 
$ export PATH_XSLT="$HOME/install/libxslt"
$ export PATH="$PATH_XSLT/bin:$PATH"
$ export ACLOCAL_PATH="$PATH_XSLT/share/aclocal:$ACLOCAL_PATH"
$ export PKG_CONFIG_PATH="$PATH_XSLT/lib/pkgconfig:$PKG_CONFIG_PATH"
$ export LD_LIBRARY_PATH="$PATH_XSLT/lib:$LD_LIBRARY_PATH"
$ export C_INCLUDE_PATH="$PATH_XSLT/include:$C_INCLUDE_PATH"
```

> https://linuxfromscratch.org/blfs/view/svn/general/libxslt.html
> https://gnome.pages.gitlab.gnome.org/libxslt/xsltproc.html
> https://gitlab.gnome.org/GNOME/libxslt

# automake

```
$ cd ~/work/
$ mkdir automake && cd automake
$ wget https://ftp.gnu.org/gnu/automake/automake-1.16.5.tar.gz
$ tar -xf automake-1.16.5.tar.gz
$ mkdir build && cd build
$ mkdir ~/install/automake
$ ../configure --prefix=$HOME/install/automake/
$ make
$ make install
$ vim ~/.bashrc 
$ export PATH_AUTOMAKE="$HOME/install/automake"
$ export PATH="$PATH_AUTOMAKE/bin:$PATH"
$ # aclocal --print-ac-dir
```

# pkg-config

```
$ cd ~/work/
$ git clone https://gitlab.freedesktop.org/pkg-config/pkg-config.git
$ cd pkg-config
$ ./autogen.sh
$ mkdir ~/install/pkg-config
$ ./configure --prefix=$HOME/install/pkg-config/
$ make
$ make install
$ vim ~/.bashrc
$ # ...
```

> https://unix.stackexchange.com/questions/405986/what-is-the-efficient-way-to-solve-no-version-information-available-when-an-ol
> https://askubuntu.com/questions/210210/pkg-config-path-environment-variable
> https://www.freedesktop.org/wiki/Software/pkg-config/

# libgcrypt

```
$ cd ~/work/
$ git clone git@github.com:gpg/libgcrypt.git
$ cd libgcrypt/
$ git tag | less
$ git checkout libgcrypt-1.9.4
$ make distclean
$ mkdir ~/install/libgcrypt
$ ./autogen.sh
$ ./configure --enable-maintainer-mode --prefix=$HOME/install/libgcrypt/ > LOG_configure194 2>&1
$ cd doc
$ make stamp-vti
$ cd ..
$ make
$ make install

```

> https://www.linuxfromscratch.org/blfs/view/8.0/general/libgcrypt.html
> https://www.linuxfromscratch.org/blfs/view/svn/general/libgcrypt.html
> https://github.com/gpg/libgcrypt
> https://github.com/rocky/remake/issues/16

# libgpg-error

```
$ cd ~/work/
$ git clone git clone git://git.gnupg.org/libgpg-error.git
$ cd libgpg-error/
$ ./autogen.sh 
$ mkdir ~/install/libgpg-error
$ ./configure --prefix=$HOME/install/libgpg-error/
$ cd doc/
$ make stamp-vti
$ cd ..
$ make > LOG_make 2>&1
$ make install > LOG_makeinstall 2>&1
$ vim ~/.bashrc 
$ export PATH_LIBGPGERROR="$HOME/install/libgpg-error"
$ export PATH="$PATH_LIBGPGERROR/bin:$PATH"
$ export LD_LIBRARY_PATH="$PATH_LIBGPGERROR/lib:$LD_LIBRARY_PATH"
$ export PKG_CONFIG_PATH="$PATH_LIBGPGERROR/lib/pkgconfig:$PKG_CONFIG_PATH"

```

> https://www.gnupg.org/software/libgpg-error/index.html
> https://github.com/rocky/remake/issues/16

# fig2dev

```
$ cd ~/work/
$ git clone git@github.com:deepin-community/fig2dev.git
$ cd fig2dev/
$ mkdir ~/install/fig2dev
$ autoreconf -i > LOG_autoreconf-i 2>&1
$ ./configure --prefix=$HOME/install/fig2dev/
$ make > LOG_make 2>&1
$ make install > LOG_makeinstall 2>&1
$ vim ~/.bashrc
$ export PATH_FIG2DEV="$HOME/install/fig2dev"
$ export PATH="$PATH_FIG2DEV/bin:$PATH"
```

> https://github.com/deepin-community/fig2dev

# Ghostscript

```
$ cd ~/work/
$ mkdir Ghostscript && cd Ghostscript
$ wget https://github.com/ArtifexSoftware/ghostpdl-downloads/releases/download/gs10012/ghostscript-10.01.2.tar.gz
$ tar -xf ghostscript-10.01.2.tar.gz 
$ cd ghostscript-10.01.2
$ mkdir ~/install/ghostscript
$ ./configure --prefix=$HOME/install/ghostscript/
$ make >> LOG_make 2>&1
$ make install >> LOG_makeinstall 2>&1
$ vim ~/.bashrc 
$ export PATH_GHOSTSCRIPT="$HOME/install/ghostscript"
$ export PATH="$PATH_GHOSTSCRIPT/bin:$PATH"

```

> https://www.ghostscript.com/
> https://github.com/ArtifexSoftware/ghostpdl-downloads/releases/tag/gs10012

# libxml2

```
$ cd ~/work/
$ git clone git@github.com:GNOME/libxml2.git
$ cd libxml2/
$ mkdir ~/install/libxml2
$ ./autogen.sh --prefix=$HOME/install/libxml2/
$ make
$ make install
$ vim ~/.bashrc
$ export PATH_XML2="$HOME/install/libxml2"
$ export PATH="$PATH_XML2/bin:$PATH"
$ export ACLOCAL_PATH="$PATH_XML2/share/aclocal:$ACLOCAL_PATH"
$ export PKG_CONFIG_PATH="$PATH_XML2/lib/pkgconfig:$PKG_CONFIG_PATH"
$ export LD_LIBRARY_PATH="$PATH_XML2/lib:$LD_LIBRARY_PATH"
```

> https://github.com/GNOME/libxml2

# libslirp

```
$ cd ~/work/
$ git clone https://gitlab.freedesktop.org/slirp/libslirp.git
$ cd libslirp
$ mkdir ~/install/libslirp
$ meson build --prefix=$HOME/install/libslirp/
$ ninja -C build install
$ vim ~/.bashrc
$ export LD_LIBRARY_PATH="$HOME/install/libslirp/lib64:$LD_LIBRARY_PATH"
$ export PKG_CONFIG_PATH="$HOME/install/libslirp/lib64/pkgconfig:$PKG_CONFIG_PATH"
```

> https://gitlab.freedesktop.org/slirp/libslirp

# libxi

> https://gitlab.freedesktop.org/xorg/lib/libxi

# libxtst

> https://gitlab.freedesktop.org/xorg/lib/libxtst

# libsm

> https://gitlab.freedesktop.org/xorg/lib/libsm

# libice

> https://gitlab.freedesktop.org/xorg/lib/libice

# zstd

> https://github.com/facebook/zstd

# libtool

> https://www.gnu.org/software/libtool/

# libsndfile

> https://github.com/libsndfile/libsndfile

# PulseAudio

> https://www.freedesktop.org/wiki/Software/PulseAudio/Download/

# libtdb

> https://www.samba.org/ftp/tdb/
> https://www.freedesktop.org/wiki/Software/PulseAudio/Download/

# gnutls

> https://gnutls.org/download.html

# libudev

> https://github.com/systemd/systemd/tree/main/src/libudev

# jackAudio

> https://jackaudio.org/downloads/

# sdl_image 2.0

> https://libsdl.org/projects/old/SDL_image/
> https://github.com/libsdl-org/SDL_image

# spice-protocol

> https://gitlab.freedesktop.org/spice/spice-protocol

# libvdeplug

> https://altlinux.pkgs.org/sisyphus/classic-x86_64/libvdeplug3-2.3.3-alt1.x86_64.rpm.html
> https://pkgs.org/download/libvdeplug3 

# libusb

> https://github.com/libusb/libusb
> https://github.com/libusb/libusb/wiki/FAQ

# dwarves

```
$ cmake -D__LIB=lib -DCMAKE_INSTALL_PREFIX=$HOME/install/dwarves -DCMAKE_C_COMPILER=$HOME/install/gcc20230621/bin/gcc -DCMAKE_BUILD_TYPE=Release -DCMAKE_LIBRARY_PATH=$HOME/rpm/usr/lib64 -DCMAKE_INCLUDE_PATH=$HOME/rpm/usr/include ..
```

> https://github.com/acmel/dwarves
