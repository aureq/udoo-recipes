# Preparing your environment #

    # apt-get install git build-essential fakeroot kernel-package u-boot-tools zlib1g-dev libncurses5-dev dosfstools lzop bc file
    # echo 'deb http://ftp.uk.debian.org/emdebian/toolchains unstable main' > /etc/apt/sources.list.d/emdebian.list
    # apt-get install emdebian-archive-keyring
    # apt-get update
    You can safely ignore Failed to fetch http://www.emdebian.org/debian/dists/unstable/Release  Unable to find expected entry 'main/binary-armhf/Packages' in Release file (Wrong sources.list entry or malformed file). This is because of "dpkg --add-architecture armhf"
    # apt-get install -V gcc-4.7-arm-linux-gnueabihf
    # for x in /usr/bin/arm-linux-gnueabihf-*-4.7; do ln -s $x $(echo $x | sed -e 's/-4.7//g' | sed -e 's#/usr/bin/#/usr/local/bin/#g' ); done
     (This is required as some of the compilation tools end with -4.7. Make sure /usr/local/bin is in your PATH and in first or second position)

# Kernel compilation #

## Architecture ##

    # export TARGET_SYSTEM="wbquad"
    (If your board is a wandboard quad)
    # export TARGET_SYSTEM="udoo-quad"
    (If your board is a udoo-quad)

## For Udoo ##

`FIXME` revalidate these steps with latest kernel and `zImage` instead of `uImage`

Please note, this section uses `-j5`. You should change the number according to your number of available CPU+1.
Also, while many operations should be executed as root, you can compile your kernel using an unprivileged user.

    # git clone git://github.com/wolfgar/Kernel_Unico.git kernel-${TARGET_SYSTEM}
    # cp conf/UDOO_defconfig kernel-${TARGET_SYSTEM}/.config
    # cd kernel-${TARGET_SYSTEM}
    # make ARCH=arm menuconfig
      (This is optional, but you may want to check/change some of the options (for advanced users))
    # make EXTRAVERSION=-kitchenaid ARCH=arm CROSS_COMPILE=arm-linux-gnueabihf- -j5 uImage modules
    # make EXTRAVERSION=-kitchenaid ARCH=arm CROSS_COMPILE=arm-linux-gnueabihf- INSTALL_MOD_PATH=../rootfs/ modules_install
    # cd ..

## For Wandboard ##

So far, the wandboard-quad uses `zImage` instead of `uImage`. The kernel configuration is provided by the Wandboard team.

    # git clone https://github.com/wandboard-org/linux kernel-${TARGET_SYSTEM}
    # cd kernel-${TARGET_SYSTEM}
    # git checkout -b wandboard_imx_3.10.17_1.0.0_ga remotes/origin/wandboard_imx_3.10.17_1.0.0_ga
    # make ARCH=arm CROSS_COMPILE=arm-linux-gnueabihf- -j5 wandboard_defconfig
    # make ARCH=arm CROSS_COMPILE=arm-linux-gnueabihf- menuconfig
    (this is only needed if you wish to customize your kernel, see below)
    # make ARCH=arm CROSS_COMPILE=arm-linux-gnueabihf- -j5 zImage modules
    # make ARCH=arm CROSS_COMPILE=arm-linux-gnueabihf- imx6q-wandboard.dtb
    # make ARCH=arm CROSS_COMPILE=arm-linux-gnueabihf- INSTALL_MOD_PATH=../rootfs/ modules_install
    # cd ..

You may want to enable the following kernel settings via `make menuconfig`

1. Kernel timing info:  `Kernel hacking` -> `Show timing information on printks`
2. Wireless lan Broadcom 43xx: `Device Drivers` -> `Network device support` -> `Wireless LAN`
 * &lt;`M`&gt; `Broadcom 43xx wireless support (mac80211 stack)`
 * [`*`] `Broadcom 43xx SDIO device support`
 * [`*`] `Support for 802.11n (N-PHY) devices` (See kernel help)
 * [`*`] `Support for low-power (LP-PHY) devices` (See kernel help)
3. Hardware Random Number Generator (/dev/hwrng, see `rng-tools`): `Device Drivers` -> `Character devices`
 * [`*`] `Freescale RNG B/C Random Number Generator`

# linux-firmware #

    # git clone git://git.kernel.org/pub/scm/linux/kernel/git/firmware/linux-firmware.git
    # cd linux-firmware


# u-boot #

`FIXME` validate these steps using `udoo-quad` hardware

    # git clone git://git.denx.de/u-boot.git
    # cd u-boot
    # [ "$TARGET_SYSTEM" = "wbquad" ] && make wandboard_quad_config || make udoo_quad_config
    # make ARCH=arm CROSS_COMPILE=arm-linux-gnueabihf- -j5
    # cd ..

# Firmware installation #

`FIXME` revalidate these steps with latest firmware

    # mkdir tmp
    # cd tmp
    # wget http://www.freescale.com/lgfiles/NMG/MAD/YOCTO/firmware-imx-3.10.17-1.0.0.bin
    # sh firmware-imx-3.10.17-1.0.0.bin
    (You need to accept the End User Licence Agreement)
    # cd firmware-imx-3.10.17-1.0.0
    # cp -ri firmware/ar3k/ firmware/ath6k/ firmware/sdma/ firmware/vpu/ ../../rootfs/lib/firmware/
    # cd ../../

# Media preparation #

Note: it's **very important** you set `DEVICE` correctly (/dev/sd?). Failing to do so may lead to **loss of data**.

    # export DEVICE=/dev/sdh
    # mkdir -p media/bootfs media/rootfs
    # dd if=/dev/zero of=/dev/sdg bs=1M count=30
    (This is usually required to have a clean sd-card and make sure nothig is left behind)
    # sfdisk $DEVICE < conf/layer.microsd
      (see the resource section)
    # mkdosfs -n ${TARGET_SYSTEM} ${DEVICE}1
    # mke2fs -t ext4 -L rootfs -m 0 ${DEVICE}2
    # tune2fs -e remount-ro -O dir_index ${DEVICE}2
    # mount ${DEVICE}1 media/bootfs
    # mount ${DEVICE}2 media/rootfs

# Preparing for system boot #

## for udoo-quad ##

`FIXME` revalidate these steps with latest kernel and latest u-boot

    # cd kernel-${TARGET_SYSTEM}
    # KERNEL_VERSION=$(make ARCH=arm kernelrelease | tail -n 1)
    # for x in arch/arm/boot/uImage System.map vmlinux; do cp -v $x ../rootfs/boot/$(basename $x-${KERNEL_VERSION}); done
    # cd ..
    # chroot rootfs/ sh -c "cd /boot; rm uImage; ln -s uImage-${KERNEL_VERSION} uImage"
    # umount rootfs/proc
    # cp -pdr rootfs/* media/rootfs/
    # cp kernel-${TARGET_SYSTEM}/arch/arm/boot/uImage media/bootfs/
    # mkimage -A arm -O linux -T script -n uImage -d conf/bootfs-${TARGET_SYSTEM}.txt media/bootfs/boot.scr

## for wandboard-quad ##

    # cd kernel-${TARGET_SYSTEM}
    # KERNEL_VERSION=$(make ARCH=arm kernelrelease | tail -n 1)
    # for x in arch/arm/boot/zImage System.map vmlinux; do cp -v $x ../rootfs/boot/$(basename $x-${KERNEL_VERSION}); done
    # cd ..
    # chroot rootfs/ sh -c "cd /boot; rm zImage; ln -s zImage-${KERNEL_VERSION} zImage"
    # umount rootfs/proc
    # cp -pdr rootfs/* media/rootfs/
    # cp kernel-${TARGET_SYSTEM}/arch/arm/boot/zImage kernel-${TARGET_SYSTEM}/arch/arm/boot/dts/imx6q-wandboard.dtb media/bootfs/

## u-boot ##

    # dd if=u-boot/u-boot.imx of=${DEVICE} bs=512 seek=2

# Final note #
At this point, your sd-card is ready to boot. You should connect your serial cable/usb cable to the consol port, insert the sd-card and power the board.
Using a terminal program (like `kermit` or `cu`) you should see the boot process, the kernel boot and the system boot.

# Resources #

- [http://udoo.org/download/files/Documents/UDOO_Starting_Manual_beta0.4_11_28_2013.pdf](http://udoo.org/download/files/Documents/UDOO_Starting_Manual_beta0.4_11_28_2013.pdf) (Section 2.4)
- [http://jas-hacks.blogspot.co.uk/2013/10/imx6-ubuntu-1304-with-gpu-acceleration.html](http://jas-hacks.blogspot.co.uk/2013/10/imx6-ubuntu-1304-with-gpu-acceleration.html)
- [http://www.udoo.org/forum/viewtopic.php?f=19&t=693](http://www.udoo.org/forum/viewtopic.php?f=19&t=693)
- [http://jas-hacks.blogspot.co.uk/2013/10/imx6-ubuntu-1304-debugging-gpu.html](http://jas-hacks.blogspot.co.uk/2013/10/imx6-ubuntu-1304-debugging-gpu.html)
- [http://jas-hacks.blogspot.co.uk/2013/12/imx6-debian-jessie-gpuvpu-3109-100.html](http://jas-hacks.blogspot.co.uk/2013/12/imx6-debian-jessie-gpuvpu-3109-100.html)
- [http://eewiki.net/display/linuxonarm/Wandboard](http://eewiki.net/display/linuxonarm/Wandboard)
- [https://romanrm.net/a10/cross-compile-kernel](https://romanrm.net/a10/cross-compile-kernel)
- [https://github.com/wolfgar/Kernel_Unico.git](https://github.com/wolfgar/Kernel_Unico.git) (Stéphan Rafin)
- [http://www.emdebian.org/crosstools.html](http://www.emdebian.org/crosstools.html)
- [http://wiki.wandboard.org/index.php/Wandboard_Linux_Kernel_3.10.17_Status](http://wiki.wandboard.org/index.php/Wandboard_Linux_Kernel_3.10.17_Status)

## Firware ##

- http://wireless.kernel.org/en/users/Drivers/brcm80211
- https://git.kernel.org/cgit/linux/kernel/git/firmware/linux-firmware.git
- https://android.googlesource.com/platform/hardware/broadcom/wlan

# Licence #

<a rel="license" href="http://creativecommons.org/licenses/by-sa/4.0/"><img alt="Creative Commons License" style="border-width:0" src="http://i.creativecommons.org/l/by-sa/4.0/88x31.png" /></a> <span xmlns:dct="http://purl.org/dc/terms/" href="http://purl.org/dc/dcmitype/Text" property="dct:title" rel="dct:type">Udoo recipes</span> by <a xmlns:cc="http://creativecommons.org/ns#" href="http://au.linkedin.com/in/aurelienrequiem/" property="cc:attributionName" rel="cc:attributionURL">Aurélien Requiem</a> is licensed under a <a rel="license" href="http://creativecommons.org/licenses/by-sa/4.0/">Creative Commons Attribution-ShareAlike 4.0 International License</a>. Based on a work at <a xmlns:dct="http://purl.org/dc/terms/" href="https://github.com/aureq/udoo-recipes" rel="dct:source">https://github.com/aureq/udoo-recipes</a>.
