# Setup Partition
gpart show nda0
gpart add freebsd-zfs -l zfs0 nda0
gpart show nda0


# Install Loader & Boot Entry
mount -t msdosfs /dev/nda0p1 /mnt
cp /boot/loader.efi /mnt/EFI/
echo 'title    FreeBSD 15' > /mnt/loader/entries/freebsd.conf
echo 'efi      /EFI/loader.efi' >> /mnt/loader/entries/freebsd.conf
umount /mnt


# Create ZFS
mount -t tmpfs tmpfs /mnt
zpool create -o altroot=/mnt zroot nda0p8
zfs create -o mountpoint=none zroot/ROOT
zfs create -o mountpoint=/ -o canmount=noauto zroot/ROOT/default
zpool set bootfs=zroot/ROOT/default zroot
mount -t zfs zroot/ROOT/default /mnt


# Install Base & Kernel
mount /dev/da0s3 /media
tar -C /mnt -xvJf /media/freebsd-dist/base.txz
tar -C /mnt -xvJf /media/freebsd-dist/kernel.txz


# Setup fstab
echo '/dev/nda0p1    /boot/efi    msdosfs    rw    2  2' > /mnt/etc/fstab


# Setup loader.conf
echo 'beastie_disable="YES"' > /mnt/boot/loader.conf
echo 'autoboot_delay="-1"' >> /mnt/boot/loader.conf
echo 'zfs_load="YES"' >> /mnt/boot/loader.conf


# Setup rc.conf
echo 'rc_startmsgs="NO"' > /mnt/etc/rc.conf
echo 'update_motd="NO"' > /mnt/etc/rc.conf
echo 'zfs_load="YES"' > /mnt/etc/rc.conf


# Set root password
chroot /mnt
passwd

reboot
