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

    # wget http://http.us.debian.org/debian/pool/main/h/hello/hello_2.8-4_armhf.deb
      (Fetch an ARM Hard-float package.)
    # mkdir rootfs
    # dpkg -x hello_2.8-4_armhf.deb rootfs/
      (Unpack your package in the target directory.)
    # ./rootfs/usr/bin/hello
      (Run your foreign package. Hello, world! should be displayed on your terminal.)

# Rootfs creation #

    # mkdir udoo
    # cd udoo
    # mkdir -p rootfs/usr/bin
    # cp -p /usr/bin/qemu-arm-static rootfs/usr/bin
    # /usr/sbin/debootstrap --foreign --arch armhf wheezy rootfs/ http://cdn.debian.net/debian/
    # DEBIAN_FRONTEND=noninteractive DEBCONF_NONINTERACTIVE_SEEN=true LC_ALL=C LANGUAGE=C LANG=C chroot rootfs/ /debootstrap/debootstrap --second-stage
    # DEBIAN_FRONTEND=noninteractive DEBCONF_NONINTERACTIVE_SEEN=true LC_ALL=C LANGUAGE=C LANG=C chroot rootfs/ dpkg --configure -a
    # mount -t proc proc rootfs/proc/
    # echo 'mxc1:12345:respawn:/sbin/getty 115200 ttymxc1' >> rootfs/etc/inittab
    # sed -i -e 's/^\([1-6]:23.*\)/#\1/g' rootfs/etc/inittab
    # chroot rootfs/ sh -c "cd /dev; mknod --mode=660 ttyS0 c 4 64 && chmod +t ttyS0 "
    # chroot rootfs/ sh -c "cd /dev; mknod --mode=660 ttymxc0 c 207 16 && chmod +t ttymxc0"
    # chroot rootfs/ sh -c "cd /dev; mknod --mode=660 ttymxc1 c 207 17 && chmod +t ttymxc1"
    # chroot rootfs/ sh -c "cd /dev; mknod --mode=660 ttymxc2 c 207 18 && chmod +t ttymxc2"
    # chroot rootfs/ sh -c "cd /dev; mknod --mode=660 ttymxc3 c 207 19 && chmod +t ttymxc3"
    # chroot rootfs/ sh -c "sed -i -e 's/\(localhost$\)/udoo-quad \1/g' /etc/hosts"
    # chroot rootfs/ sh -c "echo udoo-quad > /etc/hostname"
    # chroot rootfs/ passwd
    # chroot rootfs/ sh -c 'echo "deb http://cdn.debian.net/debian/ wheezy main contrib non-free\ndeb-src http://cdn.debian.net/debian/ wheezy main contrib non-free" >/etc/apt/sources.list'
    # chroot rootfs/ sh -c 'echo "deb http://security.debian.org/ wheezy/updates main\ndeb-src http://security.debian.org/ wheezy/updates main" >>/etc/apt/sources.list'
    # chroot rootfs/ apt-get update
    # LC_ALL=C LANGUAGE=C LANG=C chroot rootfs/ apt-get install openssh-server openssh-client -V
    # chroot rootfs/ sh -c "rm /etc/ssh/ssh_host*key*"
      (The above step and the subsequents are required if you plan to release your distribution to the general public. Each ssh host key should be locally generated and unique (ie: not coming from your kitchen).)
        # cp scripts/ssh-keys rootfs/etc/init.d/
        # chmod 0755 rootfs/etc/init.d/ssh-keys
        # LC_ALL=C LANGUAGE=C LANG=C chroot rootfs/ sh -c "update-rc.d ssh-keys defaults 2>/dev/null"
    # LC_ALL=C LANGUAGE=C LANG=C chroot rootfs/ /etc/init.d/ssh stop
    # LC_ALL=C LANGUAGE=C LANG=C chroot rootfs/ apt-get install ntp -V
    # LC_ALL=C LANGUAGE=C LANG=C chroot rootfs/ /etc/init.d/ntp stop
    # ...
    # umount rootfs/proc
    # LC_ALL=C LANGUAGE=C LANG=C chroot rootfs/ apt-get install wpasupplicant -V
    # cp conf/interfaces rootfs/etc/network/interfaces
      (Please refer to interfaces5 and wpa_supplicant.conf5 for further details on Wifi connection (3).)
    # chmod 0600 rootfs/etc/network/interfaces

# Resources #

##  conf/interfaces  ##
	# interfaces(5) file used by ifup(8) and ifdown(8)
	auto lo
	iface lo inet loopback
	 
	auto eth0
	iface eth0 inet dhcp
	 
	auto wlan0
	iface wlan0 inet dhcp
	        wpa-conf /etc/wpa_supplicant/wpa_supplicant.conf

## scripts/ssh-keys ##
	#!/bin/sh -e
	### BEGIN INIT INFO
	# Provides:             ssh-keys
	# Required-Start:       urandom
	# Required-Stop:
	# X-Start-Before:
	# Default-Start:        S
	# Default-Stop:         0 1 6
	# Short-Description:    OpenBSD Secure Shell server (Host Keys)
	### END INIT INFO
	 
	. /usr/share/debconf/confmodule
	db_version 2.0
	 
	umask 022
	 
	get_config_option() {
	        option="$1"
	 
	        [ -f /etc/ssh/sshd_config ] || return
	 
	        # TODO: actually only one '=' allowed after option
	        perl -lne 's/\s+/ /g; print if s/^\s*'"$option"'[[:space:]=]+//i' \
	           /etc/ssh/sshd_config
	}
	 
	host_keys_required() {
	        hostkeys="$(get_config_option HostKey)"
	        if [ "$hostkeys" ]; then
	                echo "$hostkeys"
	        else
	                # No HostKey directives at all, so the server picks some
	                # defaults depending on the setting of Protocol.
	                protocol="$(get_config_option Protocol)"
	                [ "$protocol" ] || protocol=1,2
	                if echo "$protocol" | grep 1 >/dev/null; then
	                        echo /etc/ssh/ssh_host_key
	                fi
	                if echo "$protocol" | grep 2 >/dev/null; then
	                        echo /etc/ssh/ssh_host_rsa_key
	                        echo /etc/ssh/ssh_host_dsa_key
	                        echo /etc/ssh/ssh_host_ecdsa_key
	                fi
	        fi
	}
	 
	create_key() {
	        msg="$1"
	        shift
	        hostkeys="$1"
	        shift
	        file="$1"
	        shift
	 
	        if echo "$hostkeys" | grep -x "$file" >/dev/null && \
	           [ ! -f "$file" ] ; then
	                echo -n $msg
	                ssh-keygen -q -f "$file" -N '' "$@"
	                echo
	                if which restorecon >/dev/null 2>&1; then
	                        restorecon "$file.pub"
	                fi
	        fi
	}
	 
	create_keys() {
	        hostkeys="$(host_keys_required)"
	 
	        create_key "Creating SSH1 key; this may take some time ..." \
	                "$hostkeys" /etc/ssh/ssh_host_key -t rsa1
	 
	        create_key "Creating SSH2 RSA key; this may take some time ..." \
	                "$hostkeys" /etc/ssh/ssh_host_rsa_key -t rsa
	        create_key "Creating SSH2 DSA key; this may take some time ..." \
	                "$hostkeys" /etc/ssh/ssh_host_dsa_key -t dsa
	        create_key "Creating SSH2 ECDSA key; this may take some time ..." \
	                "$hostkeys" /etc/ssh/ssh_host_ecdsa_key -t ecdsa
	}
	case "$1" in
	  start)
	        create_keys
	  ;;
	  *)
	        exit 0
	esac
	 
	exit 0


## External resources ##

- [https://wiki.debian.org/EmDebian/CrossDebootstrap](https://wiki.debian.org/EmDebian/CrossDebootstrap)
- [https://wiki.debian.org/QemuUserEmulation](https://wiki.debian.org/QemuUserEmulation)
- [http://ubuntuforums.org/showthread.php?t=263136](http://ubuntuforums.org/showthread.php?t=263136)
