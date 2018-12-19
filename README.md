# Raspberry PI 3 / yocto sumo 2.5.2 
### Yocto releases:
- thud: 2.6
- sumo: 2.5
---
## Bitbake Environment
**Ubuntu Server X64 16.04 LTS as a virtual machine (8 GB RAM)**

Note: Use ext3/ext4 filesystem. Bitbake currently did not build images on zfs.

Workaround:
Using zfs is possible if TMPDIR is mounted as ext4 onto a zfs zvol.

Use non root username yocto for bitbake!

### Steps
- Add secondary 100 gb vm-disk as sdb (system is on sda)
- Prepare ubuntu for bitbake 
- Infos: [Buildhost]( https://www.yoctoproject.org/docs/2.6/dev-manual/dev-manual.html#setting-up-a-native-linux-host) / [Ubuntu Distro](https://www.yoctoproject.org/docs/2.6/ref-manual/ref-manual.html#detailed-supported-distros)
>  ```
> sudo su
> apt-get update && apt-get -y upgrade
> reboot
>
> apt-get install -y gawk wget git-core diffstat unzip \
> texinfo gcc-multilib  build-essential chrpath socat \
> cpio python python3 python3-pip python3-pexpect \
> xz-utils debianutils iputils-ping
>
> # optional install kas from siemens
> apt-get install -y python3-pip
> pip3 install kas
>```

- create user yocto
> ```
> useradd -m yocto
> passwd yocto 
>```

- add yocto to /etc/sudoers - add line
>```
>visudo 
>
> yocto ALL=(ALL:ALL) NOPASSWD:ALL
>```

- make zfs
> ```
> apt-get install zfsutils-linux
> zpool create -f storage1 /dev/sdb
> zfs set compression=on storage1
>  
>  # filesystem for building
> zfs create storage1/yocto
> zfs set atime=off storage1/yocto
> chown yocto /storage1/yocto
>
>  # zvol for TMPDIR
> zfs create -s -V 60G storage1/zvol-yocto1
> zfs set compression=on storage1/zvol-yocto1
> mkfs.ext4 /dev/zvol/storage1/zvol-yocto1
> mkdir /zfstmp
> chown yocto /zfstmp
>
>  # mount ext4 at /zfstmp in /etc/fstab - add line:
>  /dev/zvol/storage1/zvol-yocto1   /zfstmp ext4 defaults 0 0
>
> # mount zfstmp
> mount -a
> ```

- install SDK (2.5 & 2.6)
> ```
> cd /storage1/yocto 
> wget "http://downloads.yoctoproject.org/releases/yocto/yocto-2.5/buildtools/x86_64-buildtools-nativesdk-standalone-2.5.sh"
> wget "http://downloads.yoctoproject.org/releases/yocto/yocto-2.6/buildtools/x86_64-buildtools-nativesdk-standalone-2.6.sh"> 
> chmod +x *.sh
>
>  # install to /opt/poky/2.5
> sh ./x86_64-buildtools-nativesdk-standalone-2.5.sh
>  # install to /opt/poky/2.6
> sh ./x86_64-buildtools-nativesdk-standalone-2.6.sh
> rm x86_64*.sh
> ```

---

## Prepare bitbake environment
- Work with user yocto
> ```
> su yocto
> bash
> mkdir -p /storage1/yocto/rpi
> cd /storage1/yocto/rpi
>```

- prepare poky
> ```
> cd /storage1/yocto/rpi
>  # clone poky
> git clone git://git.yoctoproject.org/poky poky
> cd poky
>  # checkout sumo commit 
>  # Author: Richard Purdie <richard.purdie@linuxfoundation.org>
>  # Date:   Mon Oct 29 13:49:24 2018 +0000
> git checkout 78020fb6395c73d9ea0ce8e3ad8767d6e55021ee
>  ```

- meta-oe
> ```
> cd /storage1/yocto/rpi
> git clone https://github.com/openembedded/meta-openembedded.git
> cd meta-openembedded
>  # checkout sumo commit 
>  # Author: Armin Kuster <akuster808@gmail.com>
>  # Date:   Mon Nov 26 07:55:17 2018 -0700
> git checkout 8760facba1bceb299b3613b8955621ddaa3d4c3f
>  ```

- meta-raspberrypi
> ```
> cd /storage1/yocto/rpi
> git clone https://github.com/agherzan/meta-raspberrypi
> cd meta-raspberrypi
> # meta-raspberrypi: (sumo commit)
> # Author: Andrei Gherzan <andrei@gherzan.com>
> # Date:   Fri Sep 7 15:38:53 2018 +0100
> git checkout  2d40b000021bc8a9ef7f329ed0ad410f8d227b97
> ```

---




