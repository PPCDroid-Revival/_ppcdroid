# Setting up Linux build environment
## Hardware
To work with PPCDroid you will need the following:
* x86 Host Machine
* Supported target machine
  * Nintendo Wii
    * SD Card with Wii Linux boot files
    * Monitor or TV capable of display the Wii's video signal
    * (optional) USB Gecko serial console
  * Canyonlands development board
    * Serial cable (null-modem)
    * ATX Power supply
    * USB Hub
    * USB Ethernet adapter
    * USB Keyboard & Mouse
    * USB Flash stick
    * Ethernet cross cable or Ethernet Switch/Hub and 2 cables
    * Serial ATA HDD or IDE HDD with PCI/PCIe IDE controller (if you want to work with HDD)
    * PCIe Video adapter (Currently supported are XGI Volari z9s/z9m/z11)
    * VGA or DVI cable
    * Monitor with VGA or DVI support

 
## Software
### Common
#### OS Compatibility matrix
| OS                        | Donut | Gingerbread |
| ------------------------- | :---: | :---------: |
| Ubuntu 8.04 LTS (i386)    |   ❓  |     ❓      |
| Ubuntu 8.04 LTS (x86_64)  |   ❓  |     ❓      |
| Ubuntu 10.04 LTS (i386)   |   ❓  |     ❓      |
| Ubuntu 10.04 LTS (x86_64) |   ❓  |     ❓      |
| Ubuntu 12.04 LTS (i386)   |   ❓  |     ❓      |
| Ubuntu 12.04 LTS (x86_64) |   ❓  |     ✅      |
| Ubuntu 14.04 LTS (i386)   |   ❓  |     ❓      |
| Ubuntu 14.04 LTS (x86_64) |   ❓  |     🚫      |
| Non-Ubuntu OS             |   ❓  |     ❓      |
#### Git and repo setup

Before you download the source, you will need [git](http://git-scm.com/) and [repo](http://source.android.com/source/git-repo.html) tools.

On an Ubuntu LTS (12.04) x86 host (the recommended one), you can do the following:

Getting `git`:
```
$ sudo apt-get install git-core
```

Getting other tools:
```
$ sudo apt-get install flex bison gperf build-essential zip curl python zlib1g-dev libncurses5-dev
```

Additionally on 64-bit hosts you should run:
- Ubuntu 10.04 and under:
  ```
  $ sudo apt-get install lib32ncurses5-dev ia32-libs lib32readline5-dev lib32z-dev
  ```
- Ubuntu 12.04:
  ```
  $ sudo apt-get install lib32ncurses5-dev ia32-libs lib32readline-gplv2-dev lib32z-dev g++-4.6-multilib
  ```
- Ubuntu 14.04 and over:
  ```
  $ sudo apt-get install [todo]
  ```
You'll also need a JDK (Java Development Kit) to build PPCDroid.  Assuming that you're building Gingerbread, you need JDK 6.  You can get a suitable JDK on most versions of Ubuntu by using:
```
$ sudo apt-get install openjdk-6-jdk
```

Getting `repo`:
You'll need an old version of repo for this to work, the latest version is too new for such an old copy of Ubuntu.  Here's how you do that (to get the latest 1.x version):
```
curl -L 'https://gerrit.googlesource.com/git-repo/+/refs/tags/v1.13.11/repo?format=TEXT'  | base64 -d > repo
chmod +x repo
```

You'll need to add this to your path.

#### PPCDroid source code download
Use the following commands to get source code:

```
$ mkdir ppcdroid
$ cd ppcdroid
$ repo init -u https://github.com/PPCDroid-Revival/manifest -m <manifest_name>
$ repo sync -j10
```

`<manifest_name>` can be the following:
   * ppcdroid-donut.xml - for Android 1.6 (Donut)
   * ppcdroid-gingerbread.xml - for Android 2.3 (Gingerbread)

### NFS root
To boot target board with NFS root you will need additional software:
* TFTP server
* NFS server
* DHPC server
* minicom
```
$ sudo apt-get install minicom atftpd uboot-mkimage nfs-kernel-server isc-dhcp-server
```

#### Configure network for Canyonlands
Ethernet adapter configuration:
```
$ sudo ifconfig ethX 10.10.10.1 netmask 255.255.255.0 up
```
DHCP server configuration:
`/etc/dhcp/dhcpd.conf`:
```
INTERFACES="ethX"; #

ddns-update-style none;
default-lease-time 600;
max-lease-time 7200;

subnet 10.10.10.0 netmask 255.255.255.0 {
  range 10.10.10.10 10.10.10.200;
  option routers 10.10.10.1;
  option broadcast-address 10.10.10.255;
  option subnet-mask 255.255.255.0;
}

host canyonlands {
  hardware ethernet 00:00:00:00:00:00; # Your device MAC
  fixed-address 10.10.10.36;
  option host-name "canyonlands";
  option root-path "/svr/nfs/canyonlands-root";
}
```
TFTP server configuration:
`/etc/default/atftpd`:
```
USE_INETD=true
OPTIONS="--tftpd-timeout 300 --retry-timeout 5 --mcast-port 1758 \
--mcast-addr 239.239.239.0-255 --mcast-ttl 1 --maxthread 100 --verbose=5 /srv/tftp"
```
NFS server configuration:
Adjust host configuration to allow target root filesystem to be exported on host. To do that add the following entry to `/etc/exports`
```
/srv/nfs/canyonlands-root    10.10.10.0/24(rw,no_root_squash,no_all_squash)
```

Re-export NFS directories and start NFS server:
```
$ sudo /usr/sbin/exportfs -rv
$ sudo /etc/init.d/nfs-kernel-server start
```

## Build
### Linux kernel (canyonlands)
To build kernel the Android compiler should be added into `PATH` variable:
```
$ cd <ppcdroid-repo-dir>
$ export PATH=$PATH:$PWD/prebuilt/linux-x86/toolchain/powerpc-android-linux-4.4.3/bin
```

The kernel source tree is located in root of Android sources in `kernel` folder:
```
$ make ARCH=powerpc 44x/canyonlands_android_defconfig
$ make ARCH=powerpc CROSS_COMPILE=powerpc-android-linux- uImage -j8
$ cd arch/powerpc/boot/dts && ../../../../scripts/dtc/dtc -I dts -O dtb -o canyonlands.dtb canyonlands.dts
```
uImage can be found in `arch/powerpc/boot` folder and device tree blob file in `arch/powerpc/boot/dts` folder.

### Linux kernel (Nintendo Wii)
To build the Linux kernel for the Nintendo Wii, a stock kernel can be used.  
Since it's based on Kernel 4.5, there's no need for additional changes to the source, normal Linux works fine given the right config.  
Simply follow the normal Wii Linux build process using it's [build-stack](https://github.com/Wii-Linux/build-stack), you should get `v4_5_0.krn` and `v4_5_0.ldr` in the `wiilinux` folder on the root of your SD Card.  No additional setup is required, the boot stack will detect PPCDroid if you follow the guide correctly.

### Android root filesystem

To build file system you need to specify for which board you want to build. 
This can either be `full-canyonlands` for the Canyonlands development board, or `wii` for the Nintendo Wii.

```
$ cd <ppcdroid-repo-dir>
$ source build/envsetup.sh
$ lunch <board>-eng
$ make -j8
```

The number after '-j' is amount of available CPUs on your host system. This can significantly reduce build time.

After build process finished you'll need rootfs for this do following:

```
cd out/target/product/<board>
mkdir ppcdroid_rootfs
cp -r root/* ppcdroid_rootfs
cp -r system ppcdroid_rootfs
sudo ../../../../build/tools/mktarball.sh ../../../host/linux-x86/bin/fs_get_stats ppcdroid_rootfs . target_rootfs ppcdroid-rootfs.tar.bz2
```

## Startup process
### Nintendo Wii
- Ensure that you have a kernel and loader in the `wiilinux` directory of your SD Card
- Ensure that you have Gumboot on your SD Card, and that you have BootMii, either as IOS, or boot2, installed.
- On either SD Card or USB, it doesn't matter which:
  - Create an `ext4` partition (at least 128MB or so) for PPCDroid's rootfs.  Unpack the rootfs tarball on it.
  - Create an `ext4` partition for the userdata.
- Start your Wii with all necessary storage media attached.
- The boot menu should detect PPCDroid.
- Launch it, and you should see Android starting up.
### Canyonlands
#### USB+HDD
Linux kernel and the device tree blob file can be loaded from USB stick and Android root filesystem can be located on HDD.
To prepare bootable USB stick and HDD with Android rootfs:
- Create single VFAT partition on the USB stick and put uImage and device tree blob file on it
- Create single ext3 partition on the HDD and unpack rootfs tarball on it
- Insert USB stick into target device and switch it on
- Get U-Boot command prompt
- Start USB subsystem in the U-boot:
```
# usb start
```
- Clarify USB stick device and partition numbers:
```
# usb storage
        Device 0: Vendor: USB 2.0  Rev: 0.00 Prod: USB Flash Drive
                  Type: Removable Hard Disk
                  Capacity: 3856.0 MB = 3.7 GB (7897088 x 512)
        Device 1: Vendor: Generic  Rev: 0.00 Prod: USB Flash Disk
                  Type: Removable Hard Disk
                  Capacity: 3856.0 MB = 3.7 GB (7897088 x 512)
```
- Say USB stick with the uImage and device tree blob is the Device 0:
```
# usb part 0
      print_part of 0

      Partition Map for USB device 0  --   Partition Type: DOS

      Partition     Start Sector     Num Sectors     Type
          1                   63         7886529       b 
```
- Load uImage and device tree blob from the USB stick:
```
# fatload usb 0:1 0x300000 uImage
# fatload usb 0:1 0x3000000 canyonlands.dtb
```
- Setup bootargs for the USB/IDE boot:
```
# setenv bootargs root=/dev/hde1 rw rootwait console=ttyS0,115200 init=/init
```
- Boot the kernel
```
# bootm 300000 - 3000000
```

#### Boot with NFS root
- Copy required filesystem tree:
```
$ cd <ppcdroid-repo-dir>/out/target/product/canyonlands
$ sudo cp -r root/* /svr/nfs/canyonlands-root
$ sudo cp -r system /svr/nfs/canyonlands-root
```
- Get U-Boot command prompt
- Setup bootargs for network boot:
```
setenv ethact ppc_4xx_eth1
setenv netdev eth1
setenv console_dev ttyS0
setenv kernel_image uImage-canyonlands
setenv fdt_file canyonlands.dtb
setenv fdt_addr 0x3000000
setenv loadaddr 0x300000
setenv rootfs_path /srv/nfs/canyonlands-root
setenv serverip 10.10.10.1
setenv ipadd 10.10.10.2
setenv netmask 255.255.255.0
setenv gatewayip 10.10.10.1
setenv bootargs \"root=/dev/nfs rw nfsroot=${serverip}:${rootfs_path} init=/init ip=dhcp panic=1 console=${console_dev},${baudrate}\"
setenv bootcmd \"setenv ethact ppc_4xx_eth1;tftp ${kernel_image};tftp ${fdtm_addr} ${fdt_file};bootm ${loadaddr} - ${fdtm_addr}\"
bootm
```

***Important note for Donut version:***

During start sequence the Android will bring up eth0 interface with ethmonitor service. So the NFS root should be loaded through second interface, otherwise ethmonitor must be disabled.

To do that comment line in init.rc file on target rootfs:
```
# service ethmonitor /system/bin/ethmonitor eth0
```
