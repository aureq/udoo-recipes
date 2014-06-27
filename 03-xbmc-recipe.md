# Preparing the compilation environment #

This part reuses the same technique to recreate a working armhf environment. It consists of a chroot'ed environment that will be used to compile our programs.

    # mkdir xbmc-build
    # mkdir -p xbmc-build/usr/bin
    # cp -p /usr/bin/qemu-arm-static xbmc-build/usr/bin
    # /usr/sbin/debootstrap --foreign --arch armhf wheezy xbmc-build/ http://cdn.debian.net/debian/
    # DEBIAN_FRONTEND=noninteractive DEBCONF_NONINTERACTIVE_SEEN=true LC_ALL=C LANGUAGE=C LANG=C chroot xbmc-build/ /debootstrap/debootstrap --second-stage
    # DEBIAN_FRONTEND=noninteractive DEBCONF_NONINTERACTIVE_SEEN=true LC_ALL=C LANGUAGE=C LANG=C chroot xbmc-build/ dpkg --configure -a
    # mount -t proc proc xbmc-build/proc/
    # mount -t devpts devpts xbmc-build/dev/pts
    # chroot xbmc-build/ sh -c 'echo "deb http://cdn.debian.net/debian/ wheezy main contrib non-free\ndeb-src http://cdn.debian.net/debian/ wheezy main contrib non-free" >/etc/apt/sources.list'
    # chroot xbmc-build/ sh -c 'echo "deb http://security.debian.org/ wheezy/updates main\ndeb-src http://security.debian.org/ wheezy/updates main" >>/etc/apt/sources.list'
    # LC_ALL=C LANGUAGE=C LANG=C chroot xbmc-build/ apt-get update
    # LC_ALL=C LANGUAGE=C LANG=C chroot xbmc-build/ apt-get install build-essential git subversion bison flex texinfo gettext autoconf libtool patch python file cmake
    # LC_ALL=C LANGUAGE=C LANG=C chroot xbmc-build/ apt-get install -V libsmbclient libsmbclient-dev libssh-4 libssh-dev libavahi-client3 libavahi-client-dev libmicrohttpd10 libmicrohttpd-dev libtinyxml2.6.2 libtinyxml2-dev libyajl2 libyajl-dev libbluetooth3 libbluetooth-dev liblzo2-2 liblzo2-dev libjpeg8 libjpeg8-dev libpython2.7 python-dev python-support python-imaging libfribidi0 libfribidi-dev libpcre3 libpcre3-dev libpcrecpp0 libfreetype6 libfreetype6-dev libasound2 libasound2-dev libdbus-1-3 libdbus-1-dev libmysqlclient-dev libmysqlclient18 libcdio-dev libcdio13 libudev-dev libass4 libass-dev libboost1.49-all-dev zip yasm libmodplug-dev libmodplug1 libbz2-dev libtiff4 libtiff4-dev libcwiid-dev libcwiid1 libtinyxml-dev libxslt1-dev libxslt1.1 curl swig2.0 swig openjdk-6-jre gawk gperf libvorbis-dev libcap-dev libmpeg2-4 libmpeg2-4-dev libmad0 libmad0-dev libsamplerate0 libsamplerate0-dev libcurl4-gnutls-dev libflac8 libflac-dev libsqlite3-0 libsqlite3-dev libpng12-0 libpng12-dev libjasper1 libjasper-dev libjasper1 libmp3lame-dev libmp3lame0 libiso9660-8 libiso9660-dev libusb-1.0-0 libusb-1.0-0-dev libhal-storage-dev libhal-storage1

For here on, all the subsequent work will be done in the chroot environment usint /root/work as the base directory.

# Xbmc compilation #

## Architecture ##

    # export TARGET_SYSTEM="wbquad"
    (If your board is a wandboard quad)
    # export TARGET_SYSTEM="udoo-quad"
    (If your board is a udoo-quad)

## Freescale librairies ##

This section assumes you're running the current shell in the chroot environment. Please refer to the first section of this page.

Freescale provides a number of librairies which are subject to a licence agreement (EULA) which you need to accept. If you plan on redistributing your distribution, you should make sure to include this licence and inform the final user about it.

    # mkdir xbmc-build/usr/src/linux
    # cd kernel-${TARGET_SYSTEM}
    # make ARCH=arm CROSS_COMPILE=arm-linux-gnueabihf- INSTALL_HDR_PATH=../xbmc-build/usr/src/linux headers_install
    # cd ..
    # LC_ALL=C LANGUAGE=C LANG=C chroot xbmc-build/ bash -l
    # cd
    # mkdir work
    # cd work
    # mkdir freescale
    # cd freescale

## imx-lib ##

    # wget http://www.freescale.com/lgfiles/NMG/MAD/YOCTO/imx-lib-3.10.17-1.0.0.tar.gz
    # tar -zxf imx-lib-3.10.17-1.0.0.tar.gz
    # cd imx-lib-3.10.17-1.0.0
    # rm -rf hdmi-cec
    (This seems to conflict with the *real* libcec when configuring xbmc)
    # make PLATFORM=IMX6Q C_INCLUDE_PATH=/usr/src/linux/include/ all
    # make PLATFORM=IMX6Q C_INCLUDE_PATH=/usr/src/linux/include/ install
    # cd ..

## imx-vpu ##

    # wget http://www.freescale.com/lgfiles/NMG/MAD/YOCTO/imx-vpu-3.10.17-1.0.0.bin
    # sh imx-vpu-3.10.17-1.0.0.bin
    # cd imx-vpu-3.10.17-1.0.0
    # make PLATFORM=IMX6Q
    # make PLATFORM=IMX6Q install
    # cd ..

## libfslvpuwrap ##

    # wget http://www.freescale.com/lgfiles/NMG/MAD/YOCTO/libfslvpuwrap-1.0.46.bin
    # sh cd libfslvpuwrap-1.0.46.bin
    # cd libfslvpuwrap-1.0.46
    # ./autogen.sh --prefix=/usr
    # make all
    # make install
    # cd ..

## gpu-viv-bin-mx6q ##

    # wget http://www.freescale.com/lgfiles/NMG/MAD/YOCTO/gpu-viv-bin-mx6q-3.10.17-1.0.0-hfp.bin
    # sh gpu-viv-bin-mx6q-3.10.17-1.0.0-hfp.bin
    # cd gpu-viv-bin-mx6q-3.10.17-1.0.0-hfp
    # cd usr/lib
    # rm libGAL.so libVIVANTE.so libEGL.so *-wl.so* *wayland* *-dfb.so* *-x11.so*
      (Freescale includes many different versions with different purposes for each of their library. I removed some of them as ldconfig creates by default symlinks on the X11 one and this crashes when XBMC starts.)
    # ln -s libEGL-fb.so libEGL.so
    # ln -s libGAL-fb.so libGAL.so
    # ln -s libVIVANTE-fb.so libVIVANTE.so
    # cp -pdr * /usr/local/lib/
    # ldconfig -p >/dev/null
    # cd ../include
    # cp -pdr * /usr/include/
    # cd ~/work/

## LibCEC compilation ##

This section assumes you're running the current shell in the chroot environment. Please refer to the first section of this page.

This work is in an early stage of support. A lot of credit goes back to Stéphan Rafin for his hard work.

    # git clone https://github.com/xbmc-imx6/libcec libcec-imx6
    # cd libcec-imx6
    # git checkout -b compile
    # libtoolize --force
    # aclocal
    # autoheader
    # automake --force-missing --add-missing
    # autoconf
    # ./configure --prefix=/usr --datadir=/usr/share --sysconfdir=/etc --sharedstatedir=/com --localstatedir=/var --libexecdir=/usr/local/lib/libexec --libdir=/usr/local/lib --includedir=/usr/include --oldincludedir=/usr/include --infodir=/usr/share/info --mandir=/usr/share/man --disable-silent-rules --disable-dependency-tracking --enable-imx6 --enable-shared --enable-static --enable-optimisation
    # make -j4
    # make install
    # ldconfig -vp >/dev/null
    # cd ..

## Xbmc compilation ##

This section assumes you're running the current shell in the chroot environment. Please refer to the first section of this page.

    # echo 3 > /proc/sys/vm/drop_caches
    (Sometimes, this is necessary if you don't have much free memory available.)

    # git clone git://github.com/wolfgar/xbmc.git xbmc-imx6
    # cd xbmc-imx6
    # git remote add xbmc https://github.com/xbmc/xbmc.git
    # git fetch xbmc -a
    # git checkout -b merge-13.1-Gotham
    # git merge 13.1-Gotham
    # make -C lib/taglib
    # make -C lib/taglib install
    # ln -s /usr/src/linux/include/linux/ipu.h /usr/include/linux/
    # ./bootstrap
    # CFLAGS='-I/usr/src/linux/include' ./configure --prefix=/imx6/xbmc --disable-x11 --disable-sdl \
                  --disable-xrandr --disable-gl --disable-vdpau \
                  --disable-vaapi --disable-openmax --enable-neon \
                  --enable-gles --enable-udev --enable-codec=imxvpu \
                  --enable-libcec --disable-debug --disable-texturepacker \
                  --disable-airplay --disable-airtunes --disable-crystalhd
    # make -j4
      (the option -j4 enables parallel compilation. Make sure you adjust this value depending on your current configuration)
    # make install

# Rootfs finalisation #

    # cp -pdr xbmc-build/usr/local/lib/* rootfs/usr/local/lib/
    # cp -dp xbmc-build/usr/lib/libfslvpuwrap* rootfs/usr/local/lib/
    # cp -dp xbmc-build/usr/lib/libvpu* xbmc-build/usr/lib/libcec* xbmc-build/usr/lib/libpxp* rootfs/usr/local/lib/
    # LC_ALL=C LANGUAGE=C LANG=C chroot rootfs/ apt-get install -V fbset libcurl3-gnutls libsmbclient libssh-4 libavahi-client3 libmicrohttpd10 libtinyxml2.6.2 libyajl2 libbluetooth3 liblzo2-2 libjpeg8 libpython2.7 python-support python-imaging libfribidi0 libpcre3 libpcrecpp0 libfreetype6 libasound2 libdbus-1-3 libmysqlclient18 libcdio13 libass4 zip libmodplug1 libtiff4 libcwiid1 libxslt1.1 libjasper1 libpng12-0 libsamplerate0 libhal1 libhal-storage1 libvorbisenc2 libcap2 libmad0 libflac8 libsqlite3-0 libmp3lame0 libiso9660-8 libusb-1.0-0 libhal-storage1
    # LC_ALL=C LANGUAGE=C LANG=C chroot rootfs/ sh -c "ldconfig"
    # cp -pdr xbmc-build/imx6 rootfs/
    # cp script/xbmc-watchdog rootfs/imx6/xbmc/bin/
    # chmod 0755 rootfs/imx6/xbmc/bin/xbmc-watchdog
    # chown root:root rootfs/imx6/xbmc/bin/xbmc-watchdog
    # sed -i 's|^exit.*|/imx6/xbmc/bin/xbmc-watchdog|' rootfs/etc/rc.local
    # echo -e "LABEL=xbmcdata  /.xbmc   ext4   defaults,errors=remount-ro,noatime,nodiratime   0   2" >> rootfs/etc/fstab
    # mkdir rootfs/.xbmc
    # ln -s /.xbmc rootfs/root/.xbmc

Please note that if you refreshed your rootfs/ without recompiling your kernel, you may need to run the following commands again

`FIXME` redundant ?

    # cd kernel-${TARGET_SYSTEM}
    # make EXTRAVERSION=-kitchenaid ARCH=arm CROSS_COMPILE=arm-linux-gnueabihf- INSTALL_MOD_PATH=../rootfs/ modules_install
    # cd ..
    # cd firmware-imx-3.5.7-1.0.0
    # cp -ri firmware/ar3k/ firmware/ath6k/ firmware/sdma/ firmware/vpu/ ../../rootfs/lib/firmware/
    # cd ../../

# Media preparation #

This part is the same as the recipe for uImage/zImage (kernel).

Note: it is very important you set DEVICE to the right device (/dev/sd?). Failing to do so may lead to loss of data.

    # export DEVICE=/dev/sdh
    # mkdir -p media/bootfs media/rootfs
    # sfdisk $DEVICE < conf/layer.microsd
    # mkdosfs -n udoo-quad ${DEVICE}1
    # mke2fs -t ext4 -L rootfs -O dir_index -m 0 ${DEVICE}2
    # tune2fs -e remount-ro ${DEVICE}2
    # mke2fs -t ext4 -L xbmcdata -O dir_index -m 0 ${DEVICE}3
      (This partition may contain your Xbmc settings, make sure you have a backup before creating the filesystem.)
    # tune2fs -e remount-ro ${DEVICE}3

`FIXME` needed !

    # mount ${DEVICE}1 media/bootfs
    # mount ${DEVICE}2 media/rootfs

# Preparing for system boot #

This part is the same as the recipe for uImage (kernel).

    # for x in arch/arm/boot/uImage System.map vmlinux; do cp -v kernel/$x rootfs/boot/$(basename $x-${KERNEL_VERSION}); done
    # chroot rootfs/ sh -c "cd /boot; ln -s uImage-${KERNEL_VERSION} uImage"
    # cp -pdr rootfs/* rootfs/.xbmc media/rootfs/
    # cp kernel/arch/arm/boot/uImage media/bootfs/

`FIXME`

    # mkimage -A arm -O linux -T script -n uImage -d conf/bootfs.txt media/bootfs/boot.scr
    # umount media/rootfs/ 
    # umount media/bootfs/
    # eject ${DEVICE}

Alternatively, if you know your system is already booting correctly, you may only want to refresh (or erase and copy) your `media/rootfs/`

# Xbmc debugging #

- If you already have a MySQL database configured and if Xbmc requires a database upgrade, the 1st start may take longer than usual depending on your database size. 
- `terminate called after throwing an instance of 'dbiplus::DbErrors'`: This issue is related to MySQL support and Xbmc. This is linked to your database schema not to up to date with your current xbmc build. Details here and here. Basically, Xbmc create a database named xbmcXX where XX represents the schema version. during the first startup and if you had a previous database, Xbmc will migrate the content from one database to another. If you kill, or if xbmc crashes, the database is left inconsistent (missing tables and data). At the next startup xbmc will throw this error 
- message. To fix, simply stop Xbmc, delete the database with the most recent version number and start Xbmc again. Make sure if has enough time to complete the operation.

# External resources #

- [https://github.com/wolfgar/xbmc.git](https://github.com/wolfgar/xbmc.git), [http://stephan-rafin.net/blog/](http://stephan-rafin.net/blog/) (Stéphan Rafin)
- [https://github.com/xbmc-imx6/](https://github.com/xbmc-imx6/) (Stéphan Rafin and the team)
- [http://wiki.xbmc.org/index.php?title=HOW-TO:Compile_XBMC_for_Linux](http://wiki.xbmc.org/index.php?title=HOW-TO:Compile_XBMC_for_Linux)
- [http://www.raspbian.org/RaspbianXBMC](http://www.raspbian.org/RaspbianXBMC)
- [http://linux-sunxi.org/XBMC](http://linux-sunxi.org/XBMC)
- [http://www.solid-run.com/mw/index.php/Building_XBMC](http://www.solid-run.com/mw/index.php/Building_XBMC)
- [http://wiki.openelec.tv/index.php/Compile_from_source](http://wiki.openelec.tv/index.php/Compile_from_source)
- [http://www.raspbmc.com/wiki/technical/building-xbmc/](http://www.raspbmc.com/wiki/technical/building-xbmc/)
- [https://github.com/huceke/buildroot-rbp](https://github.com/huceke/buildroot-rbp)
- [http://elinux.org/RPi_XBMC](http://elinux.org/RPi_XBMC)
- [http://boundarydevices.com/mx6-video-acceleration-raring-debian/](http://boundarydevices.com/mx6-video-acceleration-raring-debian/)

# To-Do #

- Check Freescale libs installation
- Remove HAL support durign xbmc compilation
- Add nfs support to xbmc


# Licence #

<a rel="license" href="http://creativecommons.org/licenses/by-sa/4.0/"><img alt="Creative Commons License" style="border-width:0" src="http://i.creativecommons.org/l/by-sa/4.0/88x31.png" /></a> <span xmlns:dct="http://purl.org/dc/terms/" href="http://purl.org/dc/dcmitype/Text" property="dct:title" rel="dct:type">Udoo recipes</span> by <a xmlns:cc="http://creativecommons.org/ns#" href="http://au.linkedin.com/in/aurelienrequiem/" property="cc:attributionName" rel="cc:attributionURL">Aurélien Requiem</a> is licensed under a <a rel="license" href="http://creativecommons.org/licenses/by-sa/4.0/">Creative Commons Attribution-ShareAlike 4.0 International License</a>. Based on a work at <a xmlns:dct="http://purl.org/dc/terms/" href="https://github.com/aureq/udoo-recipes" rel="dct:source">https://github.com/aureq/udoo-recipes</a>.
