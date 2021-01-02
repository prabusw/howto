# Howto on converting root partition to btrfs 

This how to describes the procedure to convert your linux root partition from ext4 to btrfs. 

Use Case: Over a period of time, we make multiple changes to the OS, customize multiple applications and lose track of all the changes. Reinstalling the OS is really out of question. If you rely on tools like Timeshift to restore your OS, converting the root filesystem to btrfs is impossible without beginning from fresh. 

The below steps allows you to continue using your OS with btrfs without the pain of re-installing all the software and customizing them all over again.

Requirements: 

1.Ability to work in command line and edit files with console text editors like nano, vi etc.
2.Timeshift backup of your system on an external drive or on a partition different from /. 
3.LiveCD or USB disk with liveCD image in it.
4.Backup of all your data folder, home folder etc on an external drive.

Tested with Linux Mint20.0, which was upgraded earlier from 19.3. The original installation was 19.2. The entire process may take a few hours, as you need to finish the OS installation once and do a Timeshift restoration once. 

Warning: Remember to test your timeshift backup atleast once by restoring them to a different partition and do a test drive on the restored OS/data to avoid regrets later. Also remember to test your data/home folder backup before you proceed further. Even though those partitions are untouched, it is better to test and understand the process of restoration once, if something goes wrong.

The entire steps below depend on the above Timeshift backup and you are going to wipe your entire OS installation before you complete the conversion of root partition to btrfs. 

If your backup/restoration does not work, fix it first or learn to do it first before you proceed further.

Note: change x in the device name to suit your needs.

Step1. 
Install the OS from liveCD on with /dev/sdax as the root partition. Remember to set the / partition filesystem to btrfs during installation. Complete the installation.

Step2. 
Now once again boot from the liveCD, but this time, use timeshift to restore from the backup that you created and tested earlier. Once the restore is completed, reboot the system.

Step3. 
When you try to use the system, your system won’t boot into the OS. The reason being, the entries for / in the restored /etc/fstab have your ext4 filesystem based values, but your / is on btrfs now. So you’ll be able to boot into your GUI and you’ve to work in the recovery mode.


Step4. 
Choose the recovery menu from the grub and drop in to the root shell. Perform the below steps as root user.

4.1 Create a temporary folder tmproot on /run, which is the only writable folder for you now. 

#mkdir /run/tmproot

4.2 Mount your  root partition on the folder as 

#mount /dev/sdax /run/tmproot

4.3 Find your new device UUID by using the command lsblk -f

#lsblk -f

4.4 Edit your /etc/fstab file and comment out the old entry for /. Create a new entry by replacing the three options i.e filesystem UUID, type and options for (/) root. 
The new /etc/fstab entry for (/) root will look like below after you’ve made the changes..(a truncated output of /etc/fstab given below for clarity)

#cat /etc/fstab 
# <file system> <mount point> <type> <options> <dump> <pass>
  
#UUID=1f16d419-121a-4b71-83e8-f6e38d969dbd   /    ext4  errors=remount-ro   0  1

UUID=7c23009f-f0b0-4561-b576-031771763a32    /   btrfs  defaults,subvol=@   0  0


Reboot your OS. 

Now you have successfully converted your ext4 root partition to btrfs filesystem.
