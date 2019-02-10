# LVM - Logical Volume Managment
LVM stands for Logical Volume Management. It is a system of managing logical volumes, or filesystems, that is much more advanced and flexible than the traditional method of partitioning a disk into one or more segments and formatting that partition with a filesystem.

## Why use LVM?
So why would you want to use LVM when you can normal partitions just fine? The answer is that lvm can do resizes, moves, and normal things you could do with typical partitions, but includes tools for easier mgmt and snapshots.

## The Basics
There are 3 things that LVM manages:

* **Volume Groups
* **Physical Volumes
* **Logical Volumes

A Volume Group is a named collection of physical and logical volumes. Typical systems only need one Volume Group to contain all of the physical and logical volumes on the system. Physical Volumes correspond to disks; they are block devices that provide the space to store logical volumes. Logical volumes correspond to partitions: they hold a filesystem. Unlike partitions though, logical volumes get names rather than numbers, they can span across multiple disks, and do not have to be physically contiguous.

## The Specifics
One of the biggest advantages LVM has is that most operations can be done on the fly, while the system is running. Most operations that you can do with gparted require that the partitions you are trying to manipulate not be in use at the time. You have to boot from a live image to perform the necessary resize, move, etc functions. LVM also circumvents the limits of the msdos partition table format with gparted, limiting you to only 4 primary partitions, and all logical partitions must be contained within one contiguous extended partition.

## Resizing Partitions
With gparted you can expand and shrink partitions, but only if they are not in use. LVM can expand a partition while it is mounted, if the filesystem used on it also supports that ( like the usual ext3/4 ). When expanding a partition, gparted can only expand it into adjacent free space, but LVM can use free space anywhere in the Volume Group, even on another disk. Using typical paritions often means that you must move other partitions around to make space to expand one, which is a very time consuming process that can result in massive data loss if it fails or is interrupted ( power loss ).

## Moving Partitions
Moving partitions with gparted is usually only necessary in the first place because of the requirement that partitions be physically contiguous. LVM isn't restricted in this way. LVM can move a partition while it is in use, and will not corrupt your data if it is interrupted. In the event that your system crashes or loses power during the move, you can simply restart it after rebooting and it will finish normally. One example would be installing a new SSD drive. Simply plug it in, boot it up, and ask lvm to move the running root filesystem to the new drive in the background while you continue working uninterrupted. Another reason you might want to move is to replace an old disk with a new, larger one. You can migrate the system to the new disk while using it, and then remove the old one later.

## Many Partitions
If you like to test various Linux distributions, or run a server with many services, cloud storage, file sharing, etc. you can quickly end up with quite a few partitions. With conventional msdos partitions, this becomes problematic due to its limitations. With LVM you can create as many Logical Volumes as you wish, and it is usually quite easy since you usually have plenty of free space left. Usually people allocate the entire drive to one partition when they first install, but since extending a partition is so easy with LVM, there is no reason to do this. It is better to allocate only what you think you will need, and leave the rest of the space free for future use. If you end up running out of the initial allocation, adding more space to that volume is just one command that completes immediately while the system is running normally.

## Snapshots
This is something you simply can not do without LVM or a filesystem using a snapshot function such as btrfs and zfs. It allows you to freeze an existing Logical Volume in time, at any moment, even while the system is running. You can continue to use the original volume normally, but the snapshot volume appears to be an image of the original, frozen in time at the moment you created it. You can use this to get a consistent filesystem image to back up, without shutting down the system. You can also use it to save the state of the system, so that you can later return to that state if you mess things up. You can even mount the snapshot volume and make changes to it, without affecting the original.

## So how do I start using LVM?
Sadly Calamares support for LVM is severely lacking. You'll need to create your Logical Volume (lv) using `parted`, `partitionmanager`, or `gparted` before you can install on top of it. First, you need to know if you're using UEFI or old BIOS. If your machine was built after 2010, you'll most likely be using UEFI BIOS. UEFI booting requires the use of an EFI partition "/boot/efi" the size of 100M-300M at the beginning of the drive (sda1). Next you'll need another physical boot parition the size of 300M to 500M for /boot (sda2). Some machines cannot read a boot partition past 100G into the hard drive so keeping it closer to the beginning of the drive is wise. Using LVM could accidently let you move your boot partition into an unbootable location. Afterwards, you can use the rest of the space for a single LVM partition (sda3).

Now there are a couple ways to proceed from here. Having an LVM partition does not immediately give you a Logical Volume. The Partition(s) which are Physical Volumes (pv) must be initialized with and added to a Volume Group (vg) and then the Volume Group (vg) acts as a "virtual hard drive" in which you must create partitions or Logical Volumes (lv). This can be done in KDE's `partitionmanager`, Gnome/MATE/XFCE's `gparted`, or in CLI. I'll assume the GUI methods can be self explanitory if you look around them, so I'll just show you the CLI way.

Once you have your LVM partition, you need to initialize it as a Physical Volume. Assuming this partition is /dev/sda3:

<pre class="clear">sudo pvcreate /dev/sda3</pre>

This writes the LVM header to the partition, which identifies it as a Physical Volume, and sets up a small area to hold the metadata describing everything about the Volume Group, and the the rest of the partition as unused Physical Extents. After that, you need to create a Volume Group and name it:

<pre class="clear">sudo vgcreate {Your VG Name Here} /dev/sda3</pre>

Now you have a named Volume Group. It contains only one Physical Volume. Now you'll want to create a Logical Volume from some of the free space in your volume group:

<pre class="clear">sudo lvcreate -n {Your LV Name Here} -L 500g {Your VG Name Here}</pre>

This creates a named Logical Volume in your named Volume Group using 500 GB of space. If you are installing, you probably want to create a Logical Volume like this to use as a root filesystem, and one for swap, and maybe one for /home. You can find the block device for this Logical Volume in '/dev/{Your VG Name Here}/{Your LV Name Here}' or 'dev/mapper/{Your VG Name Here}-{Your LV Name Here}'.

You might also want to try the lvs and pvs commands, which list the Logical Volumes and Physical Volumes respectively, and their more detailed variants; `lvdisplay` and `pvdisplay`.

If you are doing this from the live image, once you have created your Logical Volumes from the terminal, you can run the calamres installer, and use manual partitioning to select how to use each Logical Volume, and then install.

## Resizing Partitions
You can extend a Logical Volume with:

<pre class="clear">sudo lvextend -L +5g {Your VG Name Here}/{Your LV Name Here}</pre>

This will add 5 GB to the bar Logical Volume in the Volume Group. You can specify an absolute size if you want instead of a relative size by omitting the leading +. The space is allocated from any free space anywhere in the bar Volume Group. If you have multiple Physical Volumes you can add the names of one or more of them to the end of the command to limit which ones should be used to satisfy the request.

After extending the Logical Volume you need to expand the filesystem to use the new space. For ext 3/4, you simply run:

<pre class="clear">sudo resize2fs /dev/{Your VG Name Here}/{Your LV Name Here}</pre>

## Moving Partitions
If you only have one Physical Volume then you probably will not ever need to move, but if you add a new disk, you might want to. To move the Logical Volume bar off of Physical Volume /dev/sda3, you run:

 <pre class="clear">sudo pvmove -n bar /dev/sda3</pre> 

If you omit the -n bar argument, then all Logical Volumes on the /dev/sda3 Physical Volume will be moved. If you only have one other Physical Volume, then that is where it will be moved to, or you can add the name of one or more specific Physical Volumes that should be used to satisfy the request, instead of any Physical Volume in the Volume Group with free space. This process can be resumed safely if interrupted by a crash or power failure, and can be done while the Logical Volume(s) in question are in use. You can also add -b to perform the move in the background and return immediately, or -i s to have it print how much progress it has made every s seconds. If you background the move, you can check its progress with the lvs command.

## Snapshots
When you create a snapshot, you create a new Logical Volume to act as a clone of the original Logical Volume. The snapshot volume initially does not use any space, but as changes are made to the original volume, the changed blocks are copied to the snapshot volume before they are changed, in order to preserve them. This means that the more changes you make to the origin, the more space the snapshot needs. If the snapshot volume uses all of the space allocated to it, then the snapshot is broken and can not be used any more, leaving you only with the modified origin. The lvs command will tell you how much space has been used in a snapshot Logical Volume. If it starts to get full, you might want to extend it with the lvextend command. To create a snapshot of the bar Logical Volume and name it snap, run:

<pre class="clear">sudo lvcreate -s -n snap -L 5g {Your VG Name Here}/{Your LV Name Here}</pre>

This will create a snapshot named ***snap*** of the original Logical Volume bar and allocate 5 GB of space for it. Since the snapshot volume only stores the ares of the disk that have changed since it was created, it can be much smaller than the original volume.

While you have the snapshot, you can mount it if you wish and will see the original filesystem as it appeared when you made the snapshot. In the above example you would mount the /dev/{Your VG Name Here}/snap device. You can modify the snapshot without affecting the original, and the original without affecting the snapshot. If you take a snapshot of your root Logical Volume, and then upgrade some packages, or to the next whole distribution release, and then decide it isn't working out, you can merge the snapshot back into the origin volume, effectively reverting to the state at the time you made the snapshot. To do this, you simply run:

<pre class="clear">sudo lvconvert --merge {Your VG Name Here}/snap</pre>

If the origin volume of foo/snap is in use, it will inform you that the merge will take place the next time the volumes are activated. If this is the root volume, then you will need to reboot for this to happen. At the next boot, the volume will be activated and the merge will begin in the background, so your system will boot up as if you had never made the changes since the snapshot was created, and the actual data movement will take place in the background while you work.

## Mounting Additional LVMs Manually
Mounting an LVM may seem a bit intimidating, it's not. In reality, despite the length of this entry, you will only be running a handful of commands, and most of those just to get the needed information. Do not despair, this will be as painless as possible. All of these commands will be run as root from a terminal. Please also remember that your volume names may differ from the guide, please make sure to adjust accordingly.

<pre class="clear">root@localhost # pvs </pre>
This should give you an output similar to
<pre class="clear">root@localhost # pvs
  PV         VG         Fmt  Attr PSize PFree 
  /dev/sdb1  VolGroup00 lvm2 a-   7.88G 32.00M
</pre>
If we look closely we can see that /dev/sdb1 holds a lvm that is 7.88 gig in size. In this case, that's the one we want, as it is the only one.

So now we want to see what is actually in that lvm
<pre class="clear">
root@localhost # lvdisplay /dev/VolGroup00
  --- Logical volume ---
  LV Name                /dev/VolGroup00/LogVol00
  VG Name                VolGroup00
  LV UUID                SWp2V0-1xPU-0tOP-UnPs-snxF-THUl-pZMKb2
  LV Write Access        read/write
  LV Status              available
  # open                 0
  LV Size                6.88 GB
  Current LE             220
  Segments               1
  Allocation             inherit
  Read ahead sectors     auto
  - currently set to     256
  Block device           251:0
   
  --- Logical volume ---
  LV Name                /dev/VolGroup00/LogVol01
  VG Name                VolGroup00
  LV UUID                MGBeJP-ohrX-KLju-5V78-iJOi-pP3w-huaOmC
  LV Write Access        read/write
  LV Status              available
  # open                 0
  LV Size                992.00 MB
  Current LE             31
  Segments               1
  Allocation             inherit
  Read ahead sectors     auto
  - currently set to     256
  Block device           251:1
  </pre>
We are looking for two things out of that list: ***LV name*** and ***LV Size***. We have one that is ***6.88 GB*** and one that is ***992 MB***. We can safely assume that the smaller of the two is /swap so the larger must be our real filesystem. That one is named ***/dev/VolGroup00/LogVol00***.

So now we have all the information that we need. We need only to make a mount point and actually mount the volume.
<pre class="clear">
# cd /mnt
# mkdir lvm
# vgscan --mknodes
# lvchange -a y /dev/VolGroup00/LogVol00
# mount /dev/VolGroup00/LogVol00 /mnt/lvm
</pre>

If all went well then we can now get inside and look around, make changes, chroot in, or whatever caused us to want to mount the LVM in the first place.

## Mounting Additional LVMs Automatically using a script

The following is a script that will do all of that for you, in a slightly different way. You need only copy and paste the text in the box below to a text file that you make on your system. Once you copy and paste, save the file and go root, alternately you may download the script file [http://forum.sabayonlinux.org/viewtopic.php?f=86&t=17272&p=99187#p99187 this thread] from the forums. If you download from the forums, don't forget to rename the file lvm-mount.sh . Either way you do it, after the file is saved you need to go root and run the following commands:

<pre class="clear">
# chmod a+x lvm-mount.sh
# sh lvm-mount.sh 
</pre>

The script will then run and let you know where is mounted your LVM(s) for you. It will also print instructions on your screen as to how to chroot in to the LVM if that was what you were needing to do.

`lvm-mount.sh`
<pre class="clear">
#! /bin/bash
## This script is released under the GPL version 2
## copyright (2009) James Cook
## Thanks goes to Klaus Knopper who reminded me of something
## very simple that I had forgotten at the time, thanks bud.
## the author may be contacted at:
## azerthoth (at) gmail.com

## Check for user is root
## Thanks to micia for the suggestion
if [ $UID -ne 0 ]; then
   echo "You need to be root to run this script!"
   exit 1
fi

## get them all into /dev/mapper
modprobe dm-mod 2> /dev/null || :
vgscan --ignorelockingfailure --mknodes || :
vgchange -aly --ignorelockingfailure || return 2
clear
mkdir /LVM
cd /dev/mapper

## Create directories and mount
for FILE in *; do
test -b "$FILE" && mkdir /LVM/$FILE && mount /dev/mapper/$FILE /LVM/$FILE 2>/dev/null
done

## List good partitions
echo "Cleaning up LVMs that were swap partitions or with unsupported"
echo "File Systems from the list. This will not effect those partitions"
echo "There is just no need to list or parse them"
rmdir /LVM/Vol* 2>/dev/null"
echo " "
echo "The following LVM(s) were mounted for you and are ready to use"
echo " "
ls /LVM
echo " "
echo "You can find them in /LVM"
echo " "

## chroot instructions
echo "If you are rescuing/fixing a previous installation please issue"
echo "the following commands"
echo "cp -L /etc/resolv.conf /LVM/<your_lvm_name>/etc/resolv.conf"
echo "mount -t proc none /LVM/<your_lvm_name>/proc"
echo "mount -o bind /dev /LVM/<your_lvm_name>/dev"
echo "chroot /LVM/<your_lvm_name> /bin/bash"
echo "env-update"
echo "source /etc/profile"
## this process may also be interactively automated in the future
## more coffee and ambition is needed
</pre>
