# Preparing your environment #

    # apt-get install fakeroot fakechroot debootstrap qemu binfmt-support qemu-user-static
    This is required as the kitchen is running on amd64 and the target is armhf (2).

    # update-binfmts --display
      (Ensure that qemu-arm is displayed and enabled.)
    # sysctl -w vm.mmap_min_addr=0
      (This allows to run the virtual machine as a non-root user. The change isn't permanent, you need to create a file in /etc/sysctl.d/qemu.conf.)
    # dpkg --add-architecture armhf
      (This will expose to your current environment the armhf architecture.)
    # apt-get update
      (This refreshes the list of available packages for all selected architectures.)
    # apt-get install libc6:armhf
      (We need to install this library so qemu can run dynamically linked armhf binaries.)
    # echo 'EXTRA_OPTS="-L /usr/arm-linux-gnueabihf"' > /etc/qemu-binfmt/armhf.conf
      (This will instruct qemu where to find the ELF interpreter for the given architecture.)

# Validating your environment #

    # mkdir /home/workspace
    # cd /home/workspace
    # wget http://http.us.debian.org/debian/pool/main/h/hello/hello_2.9-1_armhf.deb
      (Fetch an ARM Hard-float package.)
    # mkdir rootfs
    # dpkg -x hello_2.9-1_armhf.deb rootfs/
      (Unpack your package in the target directory.)
    # ./rootfs/usr/bin/hello
      (Run your foreign package. Hello, world! should be displayed on your terminal.)

# Rootfs creation #

## Architecture ##
There's few differences between udoo-quad and the wandboard-quad. You need to choose one depending on your own device.

    # export TARGET_SYSTEM="wbquad"
    (If your board is a wandboard quad)
    # export TARGET_SYSTEM="udoo-quad"
    (If your board is a udoo-quad)

## Commands ##

    # mkdir /home/workspace
    # cd /home/workspace
    # mkdir scripts conf
    # mkdir -p rootfs/usr/bin
    # cp -p /usr/bin/qemu-arm-static rootfs/usr/bin
    # /usr/sbin/debootstrap --foreign --arch armhf wheezy rootfs/ http://cdn.debian.net/debian/
    # DEBIAN_FRONTEND=noninteractive DEBCONF_NONINTERACTIVE_SEEN=true LC_ALL=C LANGUAGE=C LANG=C chroot rootfs/ /debootstrap/debootstrap --second-stage
    # DEBIAN_FRONTEND=noninteractive DEBCONF_NONINTERACTIVE_SEEN=true LC_ALL=C LANGUAGE=C LANG=C chroot rootfs/ dpkg --configure -a
    # mount -t proc proc rootfs/proc/
    # [ "$TARGET_SYSTEM" = "udoo-quad" ] && echo 'mxc1:12345:respawn:/sbin/getty 115200 ttymxc1' >> rootfs/etc/inittab
    # [ "$TARGET_SYSTEM" = "udoo-quad" ] && echo 'mxc0:12345:respawn:/sbin/getty 115200 ttymxc0' >> rootfs/etc/inittab
    # sed -i -e 's/^\([1-6]:23.*\)/#\1/g' rootfs/etc/inittab
    # chroot rootfs/ sh -c "cd /dev; mknod --mode=660 ttyS0 c 4 64 && chmod +t ttyS0 "
    # chroot rootfs/ sh -c "cd /dev; mknod --mode=660 ttymxc0 c 207 16 && chmod +t ttymxc0"
    # chroot rootfs/ sh -c "cd /dev; mknod --mode=660 ttymxc1 c 207 17 && chmod +t ttymxc1"
    # chroot rootfs/ sh -c "cd /dev; mknod --mode=660 ttymxc2 c 207 18 && chmod +t ttymxc2"
    # chroot rootfs/ sh -c "cd /dev; mknod --mode=660 ttymxc3 c 207 19 && chmod +t ttymxc3"
    # chroot rootfs/ sh -c "sed -i -e 's/\(localhost$\)/${TARGET_SYSTEM} \1/g' /etc/hosts"
    # chroot rootfs/ sh -c "echo ${TARGET_SYSTEM} > /etc/hostname"
    # chroot rootfs/ passwd
    # chroot rootfs/ sh -c 'echo "deb http://cdn.debian.net/debian/ wheezy main contrib non-free\ndeb-src http://cdn.debian.net/debian/ wheezy main contrib non-free" >/etc/apt/sources.list'
    # chroot rootfs/ sh -c 'echo "deb http://security.debian.org/ wheezy/updates main\ndeb-src http://security.debian.org/ wheezy/updates main" >>/etc/apt/sources.list'
    # chroot rootfs/ apt-get update
    # LC_ALL=C LANGUAGE=C LANG=C chroot rootfs/ apt-get install openssh-server openssh-client -V
    # chroot rootfs/ sh -c "rm /etc/ssh/ssh_host*key*"
      (The above step and the subsequents are required if you plan to release your distribution to the general public. Each ssh host key should be locally generated and unique (ie: not coming from your kitchen).)
        # cp script/ssh-keys rootfs/etc/init.d/
        # chmod 0755 rootfs/etc/init.d/ssh-keys
        # LC_ALL=C LANGUAGE=C LANG=C chroot rootfs/ sh -c "update-rc.d ssh-keys defaults 2>/dev/null"
    # LC_ALL=C LANGUAGE=C LANG=C chroot rootfs/ /etc/init.d/ssh stop
    # LC_ALL=C LANGUAGE=C LANG=C chroot rootfs/ apt-get install ntp -V
    # LC_ALL=C LANGUAGE=C LANG=C chroot rootfs/ /etc/init.d/ntp stop
    # LC_ALL=C LANGUAGE=C LANG=C chroot rootfs/ apt-get install file rng-tools -V
    (rng-tools requires the kernel to be compiled with Freescale Random Number Generator support enabled)
    # ...
    # umount rootfs/proc
    # LC_ALL=C LANGUAGE=C LANG=C chroot rootfs/ apt-get install wpasupplicant -V
    # cp conf/interfaces rootfs/etc/network/interfaces
      (Please refer to interfaces5 and wpa_supplicant.conf5 for further details on Wifi connection (3).)
    # chmod 0600 rootfs/etc/network/interfaces

# Resources #

- [https://wiki.debian.org/EmDebian/CrossDebootstrap](https://wiki.debian.org/EmDebian/CrossDebootstrap)
- [https://wiki.debian.org/QemuUserEmulation](https://wiki.debian.org/QemuUserEmulation)
- [http://ubuntuforums.org/showthread.php?t=263136](http://ubuntuforums.org/showthread.php?t=263136)

# Licence #

<a rel="license" href="http://creativecommons.org/licenses/by-sa/4.0/"><img alt="Creative Commons License" style="border-width:0" src="http://i.creativecommons.org/l/by-sa/4.0/88x31.png" /></a> <span xmlns:dct="http://purl.org/dc/terms/" href="http://purl.org/dc/dcmitype/Text" property="dct:title" rel="dct:type">Udoo recipes</span> by <a xmlns:cc="http://creativecommons.org/ns#" href="http://au.linkedin.com/in/aurelienrequiem/" property="cc:attributionName" rel="cc:attributionURL">Aurélien Requiem</a> is licensed under a <a rel="license" href="http://creativecommons.org/licenses/by-sa/4.0/">Creative Commons Attribution-ShareAlike 4.0 International License</a>. Based on a work at <a xmlns:dct="http://purl.org/dc/terms/" href="https://github.com/aureq/udoo-recipes" rel="dct:source">https://github.com/aureq/udoo-recipes</a>.
