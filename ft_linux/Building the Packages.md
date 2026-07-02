- [binutils](#binutils)
- [GCC](#gcc)
- [Linux API Headers](#linux-api-headers)
- [Libstdc++](#libstdc)
- [M4](#m4)
- [Ncurses](#ncurses)
- [Bash](#bash)
- [Coreutils](#coreutils)
- [Diffutils](#diffutils)
- [File](#file)
- [Findutils](#findutils)
- [Gawk](#gawk)
- [Grep](#grep)
- [Gzip](#gzip)
- [Make](#make)
- [Patch](#patch)
- [Sed](#sed)
- [Tar](#tar)
- [Xz](#xz)
- [Binutils (2nd Pass)](#binutils-2nd-pass)
- [GCC (2nd Pass)](#gcc-2nd-pass)

## binutils

Lets start with out old friend ```binutils```. Binutils is a crucial collection of command-line tools from the GNU Project, essential for creating, manipulating, and inspecting binary files (executables, object files, libraries) in software development, featuring core tools like the as assembler, ld linker, strip (removes symbols), objdump (displays info), nm (lists symbols), and objcopy (copies/translates files).

This configures Binutils to build a cross-assembler and linker.

```sh
tar -xf binutils-2.45.1.tar.xz # 'Unzip' the tarball
cd binutils-2.45.1 # Move into the directory
mkdir -v build # Create buld directory
cd build # Move into the build directory
../configure --prefix=$LFS/tools \
	--with-sysroot=$LFS \
	--target=$LFS_TGT \
	--disable-nls \
	--enable-gprofng=no \
	--disable-werror \
	--enable-new-dtags \
	--enable-default-hash-style=gnu
make
make install
cd ../..
rm -rf binutils-2.45.1
```

## GCC

GCC most commonly refers to the GNU Compiler Collection, a vital, free, open-source suite of compilers for languages like C, C++, Fortran, and Ada, essential for Linux/Unix development.

We will need this to compile our code

```sh
tar -xf gcc-15.2.0.tar.gz 
cd gcc-15.2.0

# GCC needs GMP, MPFR, and MPC - extract them into the gcc directory

tar -xf ../mpfr-4.2.2.tar.xz
mv -v mpfr-4.2.2 mpfr
tar -xf ../gmp-6.3.0.tar.xz
mv -v gmp-6.3.0 gmp
tar -xf ../mpc-1.3.1.tar.gz
mv -v mpc-1.3.1 mpc

# Why extract these inside GCC?
# - GCC needs these math libraries to compile
# - Putting them here makes GCC build them internally

# Now we have to configure GCC for cross compilation
case $(uname -m) in
  x86_64)
    sed -e '/m64=/s/lib64/lib/' \
        -i.orig gcc/config/i386/t-linux64
 ;;
esac

# Then the build directory
mkdir -v build
cd build

../configure                  \
    --target=$LFS_TGT         \
    --prefix=$LFS/tools       \
    --with-glibc-version=2.42 \
    --with-sysroot=$LFS       \
    --with-newlib             \
    --without-headers         \
    --enable-default-pie      \
    --enable-default-ssp      \
    --disable-nls             \
    --disable-shared          \
    --disable-multilib        \
    --disable-threads         \
    --disable-libatomic       \
    --disable-libgomp         \
    --disable-libquadmath     \
    --disable-libssp          \
    --disable-libvtv          \
    --disable-libstdcxx       \
    --enable-languages=c,c++

make

make install

# create a compatibility symlink
cd ..
cat gcc/limitx.h gcc/glimits.h gcc/limity.h > \
  `dirname $($LFS_TGT-gcc -print-libgcc-file-name)`/include/limits.h

# and again, we clean up
cd ..
rm -rf gcc-15-2-0
```

## Linux API Headers

The next thing we need to do is install the API headers. The Linux kernel needs to expose an Application Programming Interface (API) for the system's C library (Glibc in LFS) to use. This is done by way of sanitizing various C header files that are shipped in the Linux kernel source tarball.

```sh
tar -xf linux-6.17.9.tar.xz # Extract the next tarball
cd linux-6.17.9             # Move into the directory

make mrproper # Clean the source tree

# Install the headers
make headers
find usr/include -type f ! -name '*.h' -delete
cp -rv usr/include $LFS/usr

# and again, we clean up
cd ..
rm -rf linux-6.17.9
```

## Glibc

This ones a big one...

Glibc (GNU C Library) is the essential C standard library for GNU/Linux systems, providing fundamental functions (like printf, malloc, file I/O, networking) that act as a bridge between applications and the Linux kernel, ensuring software portability and standard compliance (POSIX, ISO C) across distributions, making it a core component for almost all Linux programs

What does this ```make``` do?

- Building the C library (the foundation of your LFS system)
- Creating essential system calls and functions
- Building locale support, time functions, threading, etc.

```sh
tar -xf glibc-2.42.tar.xz # Extract glibc

cd glibc-2.42   # Move into the directory

# This patch makes glibc compliant with the Filesystem Hierarchy Standard (FHS) and ensures libraries go in the correct locations
patch -Np1 -i ../glibc-2.42-fhs-1.patch # Patch the files

# Now we create our build directory
mkdir -v build
cd build

# Set the config parameters
echo "rootsbindir=/usr/sbin" > configparms 

# Run the configure command
../configure                             \
      --prefix=/usr                      \
      --host=$LFS_TGT                    \
      --build=$(../scripts/config.guess) \
      --disable-nscd                     \
      libc_cv_slibdir=/usr/lib           \
      --enable-kernel=5.4

# When finished simply run 
make # This may take a while (~10 mins)

# Install to $LFS
make DESTDIR=$LFS install 

# Fix hardcoded path in ldd
sed '/RTLDLIST=/s@/usr@@g' -i $LFS/usr/bin/ldd 

# Then we'll do a critical sanity check to verify your toolchain works:
echo 'int main(){}' | $LFS_TGT-gcc -x c - -v -Wl,--verbose &> dummy.log
readelf -l a.out | grep ': /lib'
# Should output: [Requesting program interpreter: /lib64/ld-linux-x86-64.so.2]

# Now make sure that we're set up to use the correct start files
grep -E -o "$LFS/lib.*/S?crt[1in].*succeeded" dummy.log


# Verify that the compiler is searching for the correct header files
grep -B3 "^ $LFS/usr/include" dummy.log


# Next, verify that the new linker is being used with the correct search paths
grep 'SEARCH.*/usr/lib' dummy.log |sed 's|; |\n|g'

# Check that we are using the correct libc
grep "/lib.*/libc.so.6 " dummy.log

#Make sure GCC is using the correct dynamic linker:
grep found dummy.log

# Cleanup
rm -v a.out dummy.log

cd ../..
rm -rf glibc-2.42
```

## Libstdc++

Libstdc++ is crucial for C++ programs compiled with the GNU Compiler Collection (GCC) (g++) on Linux and other systems, providing essential components like containers, algorithms, and I/O streams

For libstdc++, we need a clean gcc again

```sh
tar -xf gcc-15.2.0.tar.gz
cd gcc-15.2.0

# then build directories for libstdc++
mkdir -v build
cd build

# Configure libstdc++
../libstdc++-v3/configure      \
    --host=$LFS_TGT            \
    --build=$(../config.guess) \
    --prefix=/usr              \
    --disable-multilib         \
    --disable-nls              \
    --disable-libstdcxx-pch    \
    --with-gxx-include-dir=/tools/$LFS_TGT/include/c++/15.2.0

# Then make
make

# We install with
make DESTDIR=$LFS install

# After that we remove the libtool archive files
rm -v $LFS/usr/lib/lib{stdc++{,exp,fs},supc++}.la

# Don't forget to clean up after yourself
cd ../..
rm -rf gcc-15.2.0
```

## M4

M4 is a powerful, general-purpose macro processor (a text-replacement tool) available on Unix-like systems, used to expand macros in source code (like C, Autoconf, Bison) before compilation

```sh
tar -xf m4-1.4.20.tar.xz 
cd m4-1.4.20

# Configure m4
./configure --prefix=/usr   \
            --host=$LFS_TGT \
            --build=$(build-aux/config.guess)

# Then make
make

# We install with
make DESTDIR=$LFS install

# Don't forget to clean up after yourself
cd ..
rm -rf m4-1.4.20
```

##  Ncurses

Ncurses is a programming library for creating textual user interfaces (TUIs) that work across a wide variety of terminals

```sh
tar -xf ncurses-6.5-20250809.tgz 
cd ncurses-6.5-20250809

mkdir -v build
pushd build
  ../configure --prefix=$LFS/tools AWK=gawk
  make -C include
  make -C progs tic
  install progs/tic $LFS/tools/bin
popd

# Configure m4
./configure --prefix=/usr                \
            --host=$LFS_TGT              \
            --build=$(./config.guess)    \
            --mandir=/usr/share/man      \
            --with-manpage-format=normal \
            --with-shared                \
            --without-normal             \
            --with-cxx-shared            \
            --without-debug              \
            --without-ada                \
            --disable-stripping          \
            AWK=gawk

# Then make
make

# We install with
make DESTDIR=$LFS install
ln -sv libncursesw.so $LFS/usr/lib/libncurses.so
sed -e 's/^#if.*XOPEN.*$/#if 1/' \
    -i $LFS/usr/include/curses.h

# Don't forget to clean up after yourself
cd ..
rm -rf ncurses-6.5-20250809
```

## Bash

Bash, which stands for "Bourne Again Shell," is
a command-line interpreter and scripting language for Unix-like operating systems like Linux and macOS. We are defintley going to need this.

```sh
tar -xf bash-5.3.tar.gz
cd bash-5.3

# Configure bash
./configure --prefix=/usr                      \
            --build=$(sh support/config.guess) \
            --host=$LFS_TGT                    \
            --without-bash-malloc

# Then make
make

# We install with
make DESTDIR=$LFS install

# Make a link for the programs that use sh for a shell
ln -sv bash $LFS/bin/sh

# Don't forget to clean up after yourself
cd ..
rm -rf bash-5.3
```

## Coreutils

Coreutils (GNU Core Utilities) are the essential collection of basic command-line programs for Unix-like operating systems (like Linux and macOS), providing fundamental file, text, and shell manipulation tools such as ls (list files), cp (copy), mv (move), rm (remove), and cat (concatenate), forming the bedrock for system administration and scripting, ensuring consistent functionality across diverse systems

```sh
tar -xf coreutils-9.9.tar.xz 
cd coreutils-9.9

# Configure bash
./configure --prefix=/usr                     \
            --host=$LFS_TGT                   \
            --build=$(build-aux/config.guess) \
            --enable-install-program=hostname \
            --enable-no-install-program=kill,uptime

# Then make
make

# We install with
make DESTDIR=$LFS install

# Move programs to their final expected locations
mv -v $LFS/usr/bin/chroot              $LFS/usr/sbin
mkdir -pv $LFS/usr/share/man/man8
mv -v $LFS/usr/share/man/man1/chroot.1 $LFS/usr/share/man/man8/chroot.8
sed -i 's/"1"/"8"/'                    $LFS/usr/share/man/man8/chroot.8

# Don't forget to clean up after yourself
cd $LFS/sources
rm -rf coreutils-9.9
```

## Diffutils

Diffutils is a core GNU software package providing command-line tools like
diff, cmp, and diff3 for finding and displaying differences between files or directories, essential for version control and tracking changes in text-based data. It helps users see what's added, removed, or changed, generating output in various formats (like unified diffs) for patching or side-by-side viewing, crucial for developers and system administrators

```sh
tar -xf diffutils-3.12.tar.xz
cd diffutils-3.12

# Configure diffutils
./configure --prefix=/usr   \
            --host=$LFS_TGT \
            gl_cv_func_strcasecmp_works=y \
            --build=$(./build-aux/config.guess)

# Then make
make

# We install with
make DESTDIR=$LFS install

# Don't forget to clean up after yourself
cd $LFS/sources
rm -rf diffutils-3.12
```

## File

The ```file``` command in Linux is a vital utility for determining the type of a file. It identifies file types by examining their content rather than their file extensions, making it an indispensable tool for users who work with various file formats.

```sh
tar -xf file-5.46.tar.gz 
cd file-5.46

# Build step
mkdir -v build
pushd build
  ../configure --disable-bzlib      \
               --disable-libseccomp \
               --disable-xzlib      \
               --disable-zlib
  make
popd

# Prepare for compilation
./configure --prefix=/usr --host=$LFS_TGT --build=$(./config.guess)

# Then make
make FILE_COMPILE=$(pwd)/build/src/file

# We install with
make DESTDIR=$LFS install

# Remove the libtool archive file because it is harmful for cross compilation
rm -v $LFS/usr/lib/libmagic.la

# Don't forget to clean up after yourself
cd ../..
rm -rf file-5.46
```

## Findutils

Findutils is a core GNU software package providing essential command-line utilities for finding files and managing directories in Linux/Unix systems, primarily featuring the powerful find command (searches file hierarchies) and xargs (builds commands from input), along with locate and updatedb for fast, database-driven searches, offering versatile ways to locate and act on files

```sh
tar -xf findutils-4.10.0.tar.xz 
cd findutils-4.10.0

# Configure findutils
./configure --prefix=/usr                   \
            --localstatedir=/var/lib/locate \
            --host=$LFS_TGT                 \
            --build=$(build-aux/config.guess)

# Then make
make

# We install with
make DESTDIR=$LFS install

# Don't forget to clean up after yourself
cd $LFS/sources
rm -rf findutils-4.10.0
```

## Gawk

The gawk command in Linux is a pattern scanning and processing language. No compilation is required, and variables can be used along with numeric functions, string functions, and logical operators

```sh
tar -xf gawk-5.3.2.tar.xz 
cd gawk-5.3.2

# First, ensure some unneeded files are not installed
sed -i 's/extras//' Makefile.in

# Configure gawk
./configure --prefix=/usr   \
            --host=$LFS_TGT \
            --build=$(build-aux/config.guess)

# Then make
make

# We install with
make DESTDIR=$LFS install

# Don't forget to clean up after yourself
cd $LFS/sources
rm -rf gawk-5.3.2
```

## Grep

 Grep is the short form of 'global search for the regular expression'. The grep command is a filter that is used to search for lines matching a specified pattern and print the matching lines to standard output.

```sh
tar -xf grep-3.12.tar.xz 
cd grep-3.12

# Configure grep
./configure --prefix=/usr   \
            --host=$LFS_TGT \
            --build=$(./build-aux/config.guess)

# Then make
make

# We install with
make DESTDIR=$LFS install

# Don't forget to clean up after yourself
cd $LFS/sources
rm -rf grep-3.12
```

## Gzip

Gzip is a command-line utility and file format for lossless data compression, reducing file sizes for storage or faster transfer, creating .gz files by default, and often paired with tar for archiving directories into tar.gz (tarball) files, using the DEFLATE algorithm

```sh
tar -xf gzip-1.14.tar.xz 
cd gzip-1.14

# Configure gzip
./configure --prefix=/usr --host=$LFS_TGT

# Then make
make

# We install with
make DESTDIR=$LFS install

# Don't forget to clean up after yourself
cd $LFS/sources
rm -rf gzip-1.14
```

## Make

The make command for Linux is a very useful utility in the automation of software development and performing tasks in a Linux environment. It simply reads a special file, which is called a Makefile and this file describes how one's program is compiled and linked with another file or another program action.

```sh
tar -xf make-4.4.1.tar.gz
cd make-4.4.1

# Configure make
./configure --prefix=/usr   \
            --host=$LFS_TGT \
            --build=$(build-aux/config.guess)

# Then make
make

# We install with
make DESTDIR=$LFS install

# Don't forget to clean up after yourself
cd $LFS/sources
rm -rf make-4.4.1
```

## Patch

Patch is a shell command that updates text files according to instructions in a separate file, called a patch file

```sh
tar -xf patch-2.8.tar.xz
cd patch-2.8

# Configure patch
./configure --prefix=/usr   \
            --host=$LFS_TGT \
            --build=$(build-aux/config.guess)

# Then make
make

# We install with
make DESTDIR=$LFS install

# Don't forget to clean up after yourself
cd $LFS/sources
rm -rf patch-2.8
```

## Sed

The sed (Stream Editor) command in Linux is a powerful text processing tool used to perform basic text transformations on an input stream (a file or input from a pipeline). It allows you to search, replace, delete, and insert text, making it highly useful for automating text manipulation tasks.

```sh
tar -xf sed-4.9.tar.xz 
cd sed-4.9

# Configure sed
./configure --prefix=/usr   \
            --host=$LFS_TGT \
            --build=$(./build-aux/config.guess)

# Then make
make

# We install with
make DESTDIR=$LFS install

# Don't forget to clean up after yourself
cd ..
rm -rf sed-4.9
```

## Tar

tar (Tape Archive) in Linux is a command-line utility that bundles multiple files and directories into a single archive file (.tar), making them easier to back up, transfer, and manage, while preserving file permissions and structure; it can also work with compression tools like gzip or bzip2 to create smaller, compressed archives (e.g., .tar.gz, .tar.bz2).

```sh
tar -xf tar-1.35.tar.xz
cd tar-1.35

# Configure tar
./configure --prefix=/usr   \
            --host=$LFS_TGT \
            --build=$(build-aux/config.guess)

# Then make
make

# We install with
make DESTDIR=$LFS install

# Don't forget to clean up after yourself
cd ..
rm -rf tar-1.35
```

## Xz

XZ in Linux refers to the xz data compression tool and its .xz file format, known for high compression ratios using the LZMA2 algorithm, making it essential for shrinking Linux kernels, software packages, backups, and embedded system firmware, though it gained recent notoriety for a security backdoor found within its utilities.

```sh
tar -xf xz-5.8.1.tar.xz 
cd xz-5.8.1

# Configure xz
./configure --prefix=/usr                     \
            --host=$LFS_TGT                   \
            --build=$(build-aux/config.guess) \
            --disable-static                  \
            --docdir=/usr/share/doc/xz-5.8.1

# Then make
make

# We install with
make DESTDIR=$LFS install

#  Remove the libtool archive file because it is harmful for cross compilation
rm -v $LFS/usr/lib/liblzma.la

# Don't forget to clean up after yourself
cd $LFS/sources
rm -rf xz-5.8.1
```

## Binutils (2nd Pass)

Why a second pass?

We perform two passes on Binutils (and other core tools) to create a self-contained, host-independent toolchain, first building temporary versions with the host's compiler (Pass 1) and then rebuilding them with the new, nascent toolchain (Pass 2) to strip host dependencies, ensuring the final system is built purely from LFS components for maximum portability and isolation

```sh
tar -xf binutils-2.45.1.tar.xz 
cd binutils-2.45.1

# Binutils building system relies on an shipped libtool copy to link against internal static libraries, but the libiberty and zlib copies shipped in the package do not use libtool. This inconsistency may cause produced binaries mistakenly linked against libraries from the host distro. Work around this issue:
sed '6031s/$add_dir//' -i ltmain.sh

#Create a separate build directory again: 
mkdir -v build
cd build

# Prepare Binutils for compilation:
../configure                   \
    --prefix=/usr              \
    --build=$(../config.guess) \
    --host=$LFS_TGT            \
    --disable-nls              \
    --enable-shared            \
    --enable-gprofng=no        \
    --disable-werror           \
    --enable-64-bit-bfd        \
    --enable-new-dtags         \
    --enable-default-hash-style=gnu

# Then make
make

# We install with
make DESTDIR=$LFS install

#  Remove the libtool archive file because it is harmful for cross compilation
rm -v $LFS/usr/lib/lib{bfd,ctf,ctf-nobfd,opcodes,sframe}.{a,la}

# Don't forget to clean up after yourself
cd $LFS/sources
rm -rf binutils-2.45.1
```

## GCC (2nd Pass)

The same is true for our second pass of GCC. We want an isolated system that does not rely on the host machine

```sh
tar -xf gcc-15.2.0.tar.gz  
cd gcc-15.2.0

# As in the first build of GCC, the GMP, MPFR, and MPC packages are required. Unpack the tarballs and move them into the required directories:
tar -xf ../mpfr-4.2.2.tar.xz
mv -v mpfr-4.2.2 mpfr
tar -xf ../gmp-6.3.0.tar.xz
mv -v gmp-6.3.0 gmp
tar -xf ../mpc-1.3.1.tar.gz
mv -v mpc-1.3.1 mpc

# Change the default directory name for 64-bit libraries to “lib”: 
case $(uname -m) in
  x86_64)
    sed -e '/m64=/s/lib64/lib/' \
        -i.orig gcc/config/i386/t-linux64
  ;;
esac

# Override the build rules of the libgcc and libstdc++ headers to allow building these libraries with POSIX threads support: 
sed '/thread_header =/s/@.*@/gthr-posix.h/' \
    -i libgcc/Makefile.in libstdc++-v3/include/Makefile.in

# Create a separate build directory again
mkdir -v build
cd build

# Prepare GCC for compilation:
../configure                   \
    --build=$(../config.guess) \
    --host=$LFS_TGT            \
    --target=$LFS_TGT          \
    --prefix=/usr              \
    --with-build-sysroot=$LFS  \
    --enable-default-pie       \
    --enable-default-ssp       \
    --disable-nls              \
    --disable-multilib         \
    --disable-libatomic        \
    --disable-libgomp          \
    --disable-libquadmath      \
    --disable-libsanitizer     \
    --disable-libssp           \
    --disable-libvtv           \
    --enable-languages=c,c++   \
    LDFLAGS_FOR_TARGET=-L$PWD/$LFS_TGT/libgcc

# Then make
make

# We install with
make DESTDIR=$LFS install

# As a finishing touch, create a utility symlink. Many programs and scripts run cc instead of gcc, 
# which is used to keep programs generic and therefore usable on all kinds of UNIX systems where 
# the GNU C compiler is not always installed. Running cc leaves the system administrator free to 
# cleardecide which C compiler to install
ln -sv gcc $LFS/usr/bin/cc

# Don't forget to clean up after yourself
cd $LFS/sources
rm -rf gcc-15.2.0
```

# Chroot

This is a major milestone. We are now going to use the environment we just created.

First you'll have to ```exit``` out of the LFS user and run this command to
change ownership of everything to root,

```sh
sudo -i # Enter sudo

  

chown --from lfs -R root:root $LFS/{usr,var,etc,tools}

case $(uname -m) in

x86_64) chown --from lfs -R root:root $LFS/lib64 ;;

esac
```

Then we need to prepare the kernel file system (KFS)

```sh
# Create necessary directories
mkdir -pv $LFS/{dev,proc,sys,run}

# Create initial device nodes
mount -v --bind /dev $LFS/dev

mount -vt devpts devpts -o gid=5,mode=0620 $LFS/dev/pts
mount -vt proc proc $LFS/proc
mount -vt sysfs sysfs $LFS/sys
mount -vt tmpfs tmpfs $LFS/run

# If /dev/shm exists
if [ -h $LFS/dev/shm ]; then
  install -v -d -m 1777 $LFS$(realpath /dev/shm)
else
  mount -vt tmpfs -o nosuid,nodev tmpfs $LFS/dev/shm
fi

# Create symlink from /lib64 to the actual dynamic linker in /usr/lib
ln -sv ../usr/lib/ld-linux-x86-64.so.2 $LFS/lib64/ld-linux-x86-64.so.2

# Also ensure libc.so.6 is accessible
ln -sv ../usr/lib/libc.so.6 $LFS/lib64/libc.so.6

# Verify the symlinks
ls -la $LFS/lib64/
```

Drum-roll please.....

```sh
# Enter Chroot!
chroot "$LFS" /usr/bin/env -i   \
    HOME=/root                  \
    TERM="$TERM"                \
    PS1='(lfs chroot) \u:\w\$ ' \
    PATH=/usr/bin:/usr/sbin     \
    MAKEFLAGS="-j$(nproc)"      \
    TESTSUITEFLAGS="-j$(nproc)" \
    /bin/bash --login
```

### Congratulations, you are now in the new LFS system!!!!
Your prompt should now look like

```sh
(lfs chroot) I have no name!:/#
```

## Kicked Out!

If you need to get back into the system to add more packages or do upgrades, you'll need to remount the drives and re-enter chroot

```sh
# 1. Mount the LFS partition
export LFS=/mnt/lfs
sudo mkdir -pv $LFS
sudo mount -v -t ext4 /dev/sda3 $LFS

# 2. Activate swap (sda2)
sudo swapon /dev/sda2

# 3. Mount the virtual kernel file systems
sudo mount -v --bind /dev $LFS/dev
sudo mount -v --bind /dev/pts $LFS/dev/pts
sudo mount -vt proc proc $LFS/proc
sudo mount -vt sysfs sysfs $LFS/sys
sudo mount -vt tmpfs tmpfs $LFS/run

# 4. Enter the chroot environment
sudo chroot "$LFS" /usr/bin/env -i   \
    HOME=/root                  \
    TERM="$TERM"                \
    PS1='(lfs chroot) \u:\w\$ ' \
    PATH=/usr/bin:/usr/sbin     \
    /bin/bash --login
```

## Fix SSH

To fix the known hosts issue, just run this command on the thost machine

```sh
# Create the FHS directory structure
mkdir -pv /{boot,home,mnt,opt,srv}
mkdir -pv /etc/{opt,sysconfig}
mkdir -pv /lib/firmware
mkdir -pv /media/{floppy,cdrom}
mkdir -pv /usr/{,local/}{include,src}
mkdir -pv /usr/lib/locale
mkdir -pv /usr/local/{bin,lib,sbin}
mkdir -pv /usr/{,local/}share/{color,dict,doc,info,locale,man}
mkdir -pv /usr/{,local/}share/{misc,terminfo,zoneinfo}
mkdir -pv /usr/{,local/}share/man/man{1..8}
mkdir -pv /var/{cache,local,log,mail,opt,spool}
mkdir -pv /var/lib/{color,misc,locate}

# Create compatibility symlink
ln -sfv /run /var/run
ln -sfv /run/lock /var/lock

# Set permissions
install -dv -m 0750 /root
install -dv -m 1777 /tmp /var/tmp
```

## Create Essential Files and Symlinks
```sh
ln -sv /proc/self/mounts /etc/mtab

# Create /etc/hosts
cat > /etc/hosts << EOF
127.0.0.1  localhost $(hostname)
::1        localhost
EOF

# Create /etc/passwd
cat > /etc/passwd << "EOF"
root:x:0:0:root:/root:/bin/bash
bin:x:1:1:bin:/dev/null:/usr/bin/false
daemon:x:6:6:Daemon User:/dev/null:/usr/bin/false
messagebus:x:18:18:D-Bus Message Daemon User:/run/dbus:/usr/bin/false
uuidd:x:80:80:UUID Generation Daemon User:/dev/null:/usr/bin/false
nobody:x:65534:65534:Unprivileged User:/dev/null:/usr/bin/false
EOF

# Create /etc/group
cat > /etc/group << "EOF"
root:x:0:
bin:x:1:daemon
sys:x:2:
kmem:x:3:
tape:x:4:
tty:x:5:
daemon:x:6:
floppy:x:7:
disk:x:8:
lp:x:9:
dialout:x:10:
audio:x:11:
video:x:12:
utmp:x:13:
cdrom:x:15:
adm:x:16:
messagebus:x:18:
input:x:24:
mail:x:34:
kvm:x:61:
uuidd:x:80:
wheel:x:97:
users:x:999:
nogroup:x:65534:
EOF

# Add our own user
echo "mgeiger:x:101:101::/home/mgeiger:/bin/bash" >> /etc/passwd
echo "mgeiger:x:101:" >> /etc/group
install -o mgeiger -d /home/mgeiger

# Then login
exec /usr/bin/bash --login

# New prompt
#(lfs chroot) root:/# 

# Initialize log files
touch /var/log/{btmp,lastlog,faillog,wtmp}
chgrp -v utmp /var/log/lastlog
chmod -v 664  /var/log/lastlog
chmod -v 600  /var/log/btmp
```

From here (/), we can enter ```/sources``` and that is our old ```/mnt/lfs/sources```

````sh
cd /sources
pwd # /sources
ls
# Python-3.14.0.tar.xz               libcap-2.77.tar.xz
# XML-Parser-2.47.tar.gz             libffi-3.5.2.tar.gz
# acl-2.3.2.tar.xz                   libtool-2.5.4.tar.xz
# attr-2.5.2.tar.gz                  libxcrypt-4.5.2.tar.xz
# autoconf-2.72.tar.xz               linux-6.17.9
# .......
```