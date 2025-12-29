# Setup Partition
gpart show nda0<br>
gpart add freebsd-zfs -l zfs0 nda0<br>
gpart show nda0<br>


# Install Loader & Boot Entry
mount -t msdosfs /dev/nda0p1 /mnt<br>
cp /boot/loader.efi /mnt/EFI/<br>
echo 'title    FreeBSD 15' > /mnt/loader/entries/freebsd.conf<br>
echo 'efi      /EFI/loader.efi' >> /mnt/loader/entries/freebsd.conf<br>
umount /mnt<br>


# Create ZFS
mount -t tmpfs tmpfs /mnt<br>
zpool create -o altroot=/mnt zroot nda0p8<br>
zfs create -o mountpoint=none zroot/ROOT<br>
zfs create -o mountpoint=/ -o canmount=noauto zroot/ROOT/default<br>
zpool set bootfs=zroot/ROOT/default zroot<br>
mount -t zfs zroot/ROOT/default /mnt<br>


# Install Base & Kernel
mount /dev/da0s3 /media<br>
tar -C /mnt -xvJf /media/freebsd-dist/base.txz<br>
tar -C /mnt -xvJf /media/freebsd-dist/kernel.txz<br>


# Setup fstab
echo '/dev/nda0p1    /boot/efi    msdosfs    rw    2  2' > /mnt/etc/fstab<br>


# Setup loader.conf
echo 'beastie_disable="YES"' > /mnt/boot/loader.conf<br>
echo 'autoboot_delay="-1"' >> /mnt/boot/loader.conf<br>
echo 'zfs_load="YES"' >> /mnt/boot/loader.conf<br>


# Setup rc.conf
echo 'rc_startmsgs="NO"' > /mnt/etc/rc.conf<br>
echo 'update_motd="NO"' > /mnt/etc/rc.conf<br>
echo 'zfs_load="YES"' > /mnt/etc/rc.conf<br>


# Set root password
chroot /mnt<br>
passwd<br>
<br>
reboot<br>
