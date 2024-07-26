# How to convert your (/) root partition to btrfs 

This how to describes the procedure to convert your linux root partition from ext4 to btrfs. 

Use Case: Over a period of time, we make multiple changes to the OS, customize multiple applications and lose track of all the changes. Reinstalling the OS is really out of question. If you rely on tools like Timeshift to restore your OS, converting the root filesystem to btrfs is impossible without beginning from fresh. 

The below steps allows you to continue using your OS with btrfs without the pain of re-installing all the software and customizing them all over again.

Requirements: 

1.Ability to work in command line and edit files with console text editors like nano, vi etc.  
2.Timeshift backup of your system on an external drive or on a partition different from /.   
3.Linux Mint LiveCD or USB boot image.  
4.Backup of all your data folder, home folder etc on an external drive.    

Tested with Linux Mint20.0, which was upgraded earlier from 19.3 and the original installation was 19.2. 

References: https://www.addictivetips.com/ubuntu-linux-tips/ubuntu-btrfs/  - To install with btrfs as root  

Time needed:  
The entire process may take a few hours, as you need to finish the OS installation once and do a Timeshift restoration once. 

Warning: 

Do not format other partitions like /boot/efi, /home/ and /data during installation. If your home and data are on your root partition, they need to be restored later. The entire root partition will be formatted and wiped clean. **If you do not understand the implication or do not have backup/restoration knowledge, do not proceed.**  

Remember to test your timeshift backup of the OS atleast once by restoring it to a different partition and do a test drive on the restored OS along with your data to avoid regrets later. 

Do remember to test your data/home backup and restoration procedure before you proceed further. Even though those partitions are untouched, it is better to test and understand the process of restoration once, incase something goes wrong.

The entire steps below depend on the above Timeshift backup and you are going to **wipe your entire OS installation** before you complete the conversion of root partition to btrfs. 

If your backup/restoration does not work, fix it first or learn to do it first before you proceed further.

Note: change x and y in the device names i.e /dev/sdxy to suit your needs like /dev/sda1.

Step1     
Install the OS from liveCD with /dev/sdxy as the root partition. Remember to set the (/) root partition filesystem to btrfs during installation. Complete the installation. Verify the working of OS once before you proceed to Step 2.

Step2     
Now once again boot from the liveCD, but this time, use timeshift to restore from the backup that you created and tested earlier. Once the restore is completed, reboot the system.

For Academic interest only. Skip and you can go to step3 directly  
When you try to use the system using regular boot up sequence, your system will not work. The reason being, the entries for (/) root partition in the restored /etc/fstab have your ext4 filesystem based values, but your (/) root partition is on btrfs now. The (/) root partition will be mounted in read only mode and  you’ve to work in the recovery mode only for now. Reboot the system again.

Step3  
Choose the recovery menu from the grub and drop in to the root shell. Perform the below steps as root user.

3.1 Create a temporary folder tmproot on /run, which is the only writable folder for you now. 

#mkdir /run/tmproot

3.2 Mount your  root partition on the folder as 

#mount /dev/sdxy /run/tmproot

3.3 Find your new device UUID by using the command lsblk -f

#lsblk -f

3.4 Edit your fstab file in /run/tmproot/etc/fstab and comment out the old entry for (/) root. Create a new entry by replacing the three options i.e filesystem UUID, type and options for (/) root.   
The new /etc/fstab entry for (/) root will look like below after you’ve made the changes..(a truncated output of fstab given below for clarity)  

#cat /run/tmproot/etc/fstab   
# <file system> <mount point> <type> <options> <dump> <pass> 
