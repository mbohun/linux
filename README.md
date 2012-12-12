linux
=====

my diff linux kernel configs are for now at: [http://users.on.net/~mbohun/linux](http://users.on.net/~mbohun/linux)

## grub2
```
martin@yobbo:~$ grub-fstest --version
grub-fstest (GRUB) 1.98
```
```
martin@yobbo:~$ grub-fstest --version
grub-fstest (GRUB) 1.99
```
```
# WARNING: grub 1.99; /dev/root; ln -s /dev/sda1 /dev/root
#          /usr/sbin/grub-probe: error: cannot stat `/dev/root'
#
# or add an udev rule: test /dev/root symlink; grub2 1.99 seem to get all excited about /dev/root
# SUBSYSTEMS=="block", KERNEL=="sda1", SYMLINK="root"
grub-install /dev/sda
```
```
# reboot=w required on my old laptop (DMI: TOSHIBA TECRA S3/Portable PC, BIOS Version 3.20 11/13/2007)
#          for PMU to work after reboot (works fine after shutdown/boot)
#          good: Performance Events: p6 PMU driver.
#           bad: Performance Events: Broken PMU hardware detected, using software events only.
#
martin@yobbo:~$ cat /etc/default/grub
GRUB_DEFAULT="saved"
GRUB_TIMEOUT=6
GRUB_DISABLE_LINUX_UUID="true"
# GRUB_CMDLINE_LINUX_DEFAULT="vmalloc=256M reboot=w pci=nommconf,use_crs"
```
```
# set the 3rd image (3rd grub menu entry) for the next boot
grub-reboot 2
```

## /sbin/installkernel
```
martin@yobbo:~/src/kernel$ cat /sbin/installkernel
#!/bin/sh
#
# Arguments:
#   $1 - kernel version
#   $2 - kernel image file
#   $3 - kernel map file
#   $4 - default install path (blank if root directory)

/usr/bin/cat $2 > $4/vmlinuz-$1
/usr/bin/cp  $3 $4/System-$1.map
/usr/sbin/grub-mkconfig -o /boot/grub/grub.cfg
```

## pat
```
martin@yobbo:~/src/kernel/linux$ uname -a
Linux yobbo 3.3.1-00 #1 PREEMPT Tue Apr 3 10:50:04 EST 2012 i686 Intel(R) Pentium(R) M processor 2.00GHz GenuineIntel GNU/Linux

root@yobbo:/home/martin/src/kernel/linux# cat /sys/kernel/debug/x86/pat_memtype_list
PAT memtype list:
write-back @ 0x7ff70000-0x7ff77000
write-back @ 0x7ff76000-0x7ff78000
uncached-minus @ 0x80100000-0x80104000
uncached-minus @ 0x80104000-0x80105000
uncached-minus @ 0x88000000-0x88001000
uncached-minus @ 0x8c002000-0x8c003000
uncached-minus @ 0xcdbff000-0xcdc00000
uncached-minus @ 0xcdc04000-0xcdc05000
uncached-minus @ 0xcdc04000-0xcdc05000
uncached-minus @ 0xcdc04000-0xcdc05000
uncached-minus @ 0xcdc04000-0xcdc05000
uncached-minus @ 0xcdffc000-0xce000000
uncached-minus @ 0xf0008000-0xf000c000
uncached-minus @ 0xf000b000-0xf000c000
uncached-minus @ 0xfed00000-0xfed01000
```

## zram
```
# do we have zram compiled into the kernel? yes
martin@yobbo:~/src/kernel$ dmesg|grep zram
zram: num_devices not specified. Using default: 1
zram: Creating 1 devices ...

# no - perhaps as a module?
martin@yobbo:~/src/kernel$ lsmod | grep zram
modprobe zram

# have a look what parameters are available to fiddle with
ls -laF /sys/block/zram0/

# let's make a 512mb zram
cat /sys/block/zram0/disksize
echo $((512*1024*1024)) > /sys/block/zram0/disksize
cat /sys/block/zram0/disksize

# example use as swap (set some high swap priority)
mkswap /dev/zram0
swapon -p 32766 /dev/zram0

# later that day someone turned the swapoff
swapoff /dev/zram0
```
