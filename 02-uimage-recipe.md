# Preparing your environment #

    # apt-get install git build-essential fakeroot kernel-package u-boot-tools zlib1g-dev libncurses5-dev dosfstools lzop
    # echo 'deb http://www.emdebian.org/debian/ unstable main' > /etc/apt/sources.list.d/emdebian.list
    You can safely ignore Failed to fetch http://www.emdebian.org/debian/dists/unstable/Release  Unable to find expected entry 'main/binary-armhf/Packages' in Release file (Wrong sources.list entry or malformed file)
    # for x in /usr/bin/arm-linux-gnueabihf-*-4.7; do ln -s $x $(echo $x | sed -e 's/-4.7//g' | sed -e 's#/usr/bin/#/usr/local/bin/#g' ); done
     (This is required as some of the compilation tools end with -4.7. Make sure /usr/local/bin is in your PATH.)

# Kernel compilation #

Please note, this section uses `-j5`. You should change the number according to your number of available CPU+1.
Also, while many operations should be executed as root, you can compile your kernel using an unprivileged user.

    # git clone git://github.com/wolfgar/Kernel_Unico.git kernel
    # cp conf/UDOO_defconfig kernel/.config
    # cd kernel
    # KERNEL_VERSION=$(make EXTRAVERSION=-kitchenaid ARCH=arm kernelrelease | tail -n 1)
    # make ARCH=arm menuconfig
      (This is optional, but you may want to check/change some of the options (for advanced users))
    # make EXTRAVERSION=-kitchenaid ARCH=arm CROSS_COMPILE=arm-linux-gnueabihf- -j5 uImage modules
    # make EXTRAVERSION=-kitchenaid ARCH=arm CROSS_COMPILE=arm-linux-gnueabihf- INSTALL_MOD_PATH=../rootfs/ modules_install
    # cd ..

# Firmware installation #

    # mkdir tmp
    # cd tmp
    # wget http://www.freescale.com/lgfiles/NMG/MAD/YOCTO/firmware-imx-3.5.7-1.0.0.bin
    # sh firmware-imx-3.5.7-1.0.0.bin
    # cd firmware-imx-3.5.7-1.0.0
    # cp -ri firmware/ar3k/ firmware/ath6k/ firmware/sdma/ firmware/vpu/ ../../rootfs/lib/firmware/
    # cd ../../

# Media preparation #

Note: it's very important you set `DEVICE` correctly (/dev/sd?). Failing to do so may lead to loss of data.

    # export DEVICE=/dev/sdh
    # mkdir -p media/bootfs media/rootfs
    # sfdisk $DEVICE < conf/layer.microsd
      (see the resource section)
    # mkdosfs -n udoo-quad ${DEVICE}1
    # mke2fs -t ext3 -L rootfs -m 0 ${DEVICE}2
    # tune2fs -e remount-ro -O dir_index ${DEVICE}2
    # mount ${DEVICE}1 media/bootfs
    # mount ${DEVICE}2 media/rootfs

# Preparing for system boot #

    # for x in arch/arm/boot/uImage System.map vmlinux; do cp -v kernel/$x rootfs/boot/$(basename $x-${KERNEL_VERSION}); done
    # chroot rootfs/ sh -c "cd /boot; ln -s uImage-${KERNEL_VERSION} uImage"
    # cp -pdr rootfs/* media/rootfs/
    # cp kernel/arch/arm/boot/uImage media/bootfs/
    # mkimage -A arm -O linux -T script -n uImage -d conf/bootfs.txt media/bootfs/boot.scr

# Resources #

## conf/layer.microsd ##
	# partition table of /dev/sdh
	unit: sectors
	
	/dev/sdh1 : start=     8192, size=    16384, Id= c
	/dev/sdh2 : start=    24576, size=  2343808, Id=83
	/dev/sdh3 : start=  2368384, size=  1621120, Id=83
	/dev/sdh4 : start=        0, size=        0, Id= 0

## # External resources # ##

- [http://udoo.org/download/files/Documents/UDOO_Starting_Manual_beta0.4_11_28_2013.pdf](http://udoo.org/download/files/Documents/UDOO_Starting_Manual_beta0.4_11_28_2013.pdf) (Section 2.4)
- [http://jas-hacks.blogspot.co.uk/2013/10/imx6-ubuntu-1304-with-gpu-acceleration.html](http://jas-hacks.blogspot.co.uk/2013/10/imx6-ubuntu-1304-with-gpu-acceleration.html)
- [http://www.udoo.org/forum/viewtopic.php?f=19&t=693](http://www.udoo.org/forum/viewtopic.php?f=19&t=693)
- [http://jas-hacks.blogspot.co.uk/2013/10/imx6-ubuntu-1304-debugging-gpu.html](http://jas-hacks.blogspot.co.uk/2013/10/imx6-ubuntu-1304-debugging-gpu.html)
- [http://jas-hacks.blogspot.co.uk/2013/12/imx6-debian-jessie-gpuvpu-3109-100.html](http://jas-hacks.blogspot.co.uk/2013/12/imx6-debian-jessie-gpuvpu-3109-100.html)
- [http://eewiki.net/display/linuxonarm/Wandboard](http://eewiki.net/display/linuxonarm/Wandboard)
- [https://romanrm.net/a10/cross-compile-kernel](https://romanrm.net/a10/cross-compile-kernel)
- [https://github.com/wolfgar/Kernel_Unico.git](https://github.com/wolfgar/Kernel_Unico.git) (Stéphan Rafin)
