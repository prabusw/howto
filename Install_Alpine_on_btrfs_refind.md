Installation Guide for Alpine Linux on a btrfs+refind system

Given below is the assumed disk partition scheme and it is also assumed that your other OS is already installed and working with refind as boot manager. If your setup differs, you need to take care of them.
```
dev/nvme0n1p1 -EFI appears as TYPE "vfat"
dev/nvme0n1p2 - swap appears as TYPE "swap"
dev/nvme0n1p3 - btrfs partition appears as TYPE "btrfs"
```
Boot your PC from the Alpine Linux USB.
Once booted, log in as root (no password required).

Identify your partitions by using the command

`# blkid`

Mount the btrfs partition to /mnt
`# mount /dev/nvme0n1p3 /mnt -t btrfs`

Run the setup script:
`# setup-alpine`
This will guide you through basic system configuration. Follow the prompts to:

    Select keyboard layout
    Set hostname
    Configure network (choose your WiFi interface and enter your WiFi credentials)
    Set root password
    Choose timezone
    Choose NTP client (chronyd)
    Choose a mirror for packages
    Create a new user (optional but recommended)
    After this accept the remaining choices.
Once basic setup is complete, prepare your disk for installation:

Create a new Btrfs subvolume for Alpine and you need btrfs-progs package 
`# apk add btrfs-progs`
Create a new Btrfs subvolume for Alpine:
```
# btrfs subvolume create /mnt/@alpine
# umount /mnt
```
Mount the new subvolume and other necessary partitions:
```
# mount /dev/nvme0n1p3 /mnt -t btrfs
# mkdir /mnt/os
# mount -o subvol=@alpine /dev/nvme0n1p3 /mnt/os
```
Install the base system:

`# setup-disk -m sys /mnt/os`

This will install the base system to the mounted subvolume.
Once the base system is installed, chroot into the new system:

`# chroot /mnt/os`

Edit /etc/fstab to use the correct subvolume. Ensure the root entry looks like this:

`/dev/nvme0n1p3 / btrfs subvol=@alpine,rw,relatime 0 1`

Also verify linux kernel and firmware are installed successfully.

`# apk list --installed |grep linux`

You may want to remove the grub and grub-efi packages

`# apk del grub grub-efi`


Exit the chroot:

`# exit`

Mount the EFI partition to edit the refind.conf file

```
# mkdir /mnt/os/boot/efi
# mount dev/nvme0n1p1 /mnt/os/boot/efi
```
Edit /mnt/os/boot/efi/EFI/refind/refind.conf to boot alpine:

Add an entry for Alpine:

    menuentry "Alpine Linux" {
        volume   "BTRFSVOL"
        loader   @alpine/boot/vmlinuz-lts
        initrd   @alpine/boot/initramfs-lts
        options  "root=UUID=823a3283-30a7-4fef-b50b-8a2230c71b5b rw rootflags=subvol=@alpine rootfstype=btrfs"
        # PARTUUID not working for alpine unlike arch
    }

Save and exit the editor.
    
Unmount everything:

`#umount -R /mnt`

Reboot your system. 

`#reboot`

You should now see an option to boot into Alpine Linux in the rEFInd boot menu.
