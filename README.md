# shakti-debian

### Prerequisites

This setup is tested on a Ubuntu 18.04 host machine. Other linux distros should
work but not been validated.

This project makes use of ISAR for building debian images. For more information
visit https://github.com/ilbers/isar

Install the following packages.
```
sudo apt-get install binfmt-support \
    debootstrap \
    dosfstools \
    dpkg-dev \
    gettext-base \
    git \
    mtools \
    parted \
    python3 \
    python3-distutils \
    qemu \
    qemu-user-static \
    reprepro \
    sudo
```

### Cloning the repository
```
git clone --recurse-submodules git@github.com:vj-kumar/shakti-debian.git
```

### Setup sudo rights

Follow the instructions available [here](https://github.com/ilbers/isar/blob/master/doc/user_manual.md#setup-sudo) to setup sudo rights.

### Install qemu-riscv64-static
```
$ git clone https://git.qemu.org/git/qemu.git
$ cd qemu
$ ./configure --static --disable-system --target-list=riscv64-linux-user
$ make
$ sudo cp riscv64-linux-user/qemu-riscv64 /usr/bin/qemu-riscv64-static
```

Create a binfmt-support config file and register it:

```
$ cat >/tmp/qemu-riscv64 <<EOF
package qemu-user-static
type magic
offset 0
magic \x7f\x45\x4c\x46\x02\x01\x01\x00\x00\x00\x00\x00\x00\x00\x00\x00\x02\x00\xf3\x00
mask \xff\xff\xff\xff\xff\xff\xff\x00\xff\xff\xff\xff\xff\xff\xff\xff\xfe\xff\xff\xff
interpreter /usr/bin/qemu-riscv64-static
EOF
$ sudo update-binfmts --import /tmp/qemu-riscv64
```

### Patch qemu-debootstrap

Take a back up of qemu-debootstrap
```
$ cp /usr/bin/qemu-debootstrap /usr/bin/qemu-debootstrap.bak
```
Edit /usr/sbin/qemubootstrap and add riscv64 in the list of architectures.

```
--- /usr/sbin/qemu-debootstrap.bak      2019-06-13 11:38:33.000000000 +0530
+++ /usr/sbin/qemu-debootstrap  2019-09-20 17:31:12.410143638 +0530
@@ -133,9 +133,9 @@
 fi

 qemu_arch=""
 case "$deb_arch" in
-  alpha|arm|armeb|i386|m68k|mips|mipsel|mips64el|ppc64|sh4|sh4eb|sparc|sparc64|s390x)
+  alpha|arm|armeb|i386|m68k|mips|mipsel|mips64el|ppc64|sh4|sh4eb|sparc|sparc64|s390x|riscv64)
     qemu_arch="$deb_arch"
   ;;
   amd64)
     qemu_arch="x86_64"

```

### Building Debian Image
```
. ./setup-debian-build ./build
bitbake multiconfig:qemuriscv64-bullseye-ports:isar-image-base
```
### Booting image using Qemu

The final Debian image will be present in

```
tmp/deploy/images/qemuriscv64/isar-image-base-debian-bullseye-ports-qemuriscv64.ext4.img
```
Test the image by running
```
./run_debian_qemu.sh <path to image>
For ex:
./run_debian_qemu.sh build/tmp/deploy/images/qemuriscv64/isar-image-base-debian-bullseye-ports-qemuriscv64.ext4.img
```
