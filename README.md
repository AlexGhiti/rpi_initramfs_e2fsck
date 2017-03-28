WHAT IS IT ?

I had problems with power failures, and everytime my raspberry would not boot
after such problem because my root partition on my external HDD was 'corrupted'.
The only way I found to fix that is to create a new kernel image that embeds
a small initramfs that contains e2fsck and that checks my root partition.

HOW DOES THAT EVEN WORK ?

The raspberry pi kernel is in a /boot partition, loaded in RAM by the
bootloader (cf
http://raspberrypi.stackexchange.com/questions/10489/how-does-raspberry-pi-boot
for further explanations on raspberry pi boot process): embedding an initramfs
into the kernel image ensures that even if the root partition is corrupted,
e2fsck is able to be launched and fix the root partition since everything is
in RAM and /boot partition should never be corrupted.

HOW TO COMPILE NEW KERNEL ?

1/ Use buildroot to generate kernel + initramfs
$ git clone git://git.busybox.net/buildroot

2/ a/ Copy rpi_initramfs_e2fsck_defconfig file into buildroot/configs
   b/ Copy busybox-rpi_initramfs_e2fsck_defconfig file into
      buildroot/package/busybox/
   c/ Copy overlay-rpi_initramfs_e2fsck_defconfig directory into
      buildroot/system/
      IMPORTANT: You must edit init file in
      overlay-rpi_initramfs_e2fsck_defconfig directory in order to give the right
      partition to e2fsck (ie the rootfs partition of your disk/sd card...).
      In my case, it is /dev/sda1, to check yours, log into your raspberry,
      type "df -h" and note which partition is mounted on "/".

3/ Go into buildroot/ directory
$ make O=build rpi_initramfs_e2fsck_defconfig

4/ A build/ directory has been generated, go into this directory
$ cd build/ && make

5/ a/ Into build/ directory, you will find the zImage kernel into images/zImage:
scp this image to your raspberrypi into /boot as kernel7.img.
$ scp images/zImage root@RASPBERRY_PI_IP:/boot/kernel7.img 

Note: Before doing this, you may want to save kernel7.img in case your raspberry
does not boot correctly.

   b/ You must also send all the newly built modules to /lib/modules/ of the
raspberry. This is important as the kernel version is likely to change and
modules are built against a certain kernel version.
$ scp -r build/target/lib/modules/KERNEL_VERSION/ root@RASPBERRY_PI_IP:/lib/modules/

6/ Finally, reboot, and you should see e2fsck checking your root partition
in dmesg :)

TODO

1/ Improve init script:
   a/ USB hdd might take some time to be discovered by the
   kernel, for the moment, I put a "sleep 10" to wait for it to appear...
   b/ Sometimes e2fsck might need human intervention: we could add dropbear
   so that user might be able to log into the raspberry and manually fix the
   issues.

2/ Minimize busybox size.

PROBLEMS

If any problem, please file an issue, I will update the README accordingly.

