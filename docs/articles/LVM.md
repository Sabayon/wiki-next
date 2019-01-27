=Mounting LVMs=

==Manually==
Mounting an LVM may seem a bit intimidating, it's not. In reality, despite the length of this entry, you will only be running a handful of commands, and most of those just to get the needed information. Do not despair, this will be as painless as possible. All of these commands will be run as root from a terminal. Please also remember that your volume names may differ from the guide, please make sure to adjust accordingly.

{{Console|<pre class="clear"> # pvs </pre>}}
This should give you an output similar to
<pre>sabayonx86 sabayonuser # pvs
  PV         VG         Fmt  Attr PSize PFree 
  /dev/sda2  VolGroup00 lvm2 a-   7.88G 32.00M
</pre>
If we look closely we can see that /dev/sda2 holds a lvm that is 7.88 gig in size. In this case, that's the one we want, as it is the only one.

So now we want to see what is actually in that lvm
{{Console|<pre class="clear"> # lvdisplay /dev/VolGroup00 </pre>}}
<pre>sabayonx86 sabayonuser # lvdisplay /dev/VolGroup00
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
We are looking for two things out of that list: '''LV name''' and '''LV Size'''. We have one that is '''6.88 GB''' and one that is '''992 MB'''. We can safely assume that the smaller of the two is /swap so the larger must be our real filesystem. That one is named '''/dev/VolGroup00/LogVol00'''.

So now we have all the information that we need. We need only to make a mount point and actually mount the volume.
{{Console|<pre class="clear"># cd /mnt
# mkdir lvm
# vgscan --mknodes
# lvchange -a y /dev/VolGroup00/LogVol00
# mount /dev/VolGroup00/LogVol00 /mnt/lvm
</pre>}}

If all went well then we can now get inside and look around, make changes, chroot in, or whatever caused us to want to mount the LVM in the first place.

==Automatically using a script==

The following is a script that will do all of that for you, in a slightly different way. You need only copy and paste the text in the box below to a text file that you make on your system. Once you copy and paste, save the file and go root, alternately you may download the script file [http://forum.sabayonlinux.org/viewtopic.php?f=86&t=17272&p=99187#p99187 this thread] from the forums. If you download from the forums, don't forget to rename the file lvm-mount.sh . Either way you do it, after the file is saved you need to go root and run the following commands:

{{Console|<pre class="clear">
# chmod a+x lvm-mount.sh
# sh lvm-mount.sh 
</pre>}}

The script will then run and let you know where is mounted your LVM(s) for you. It will also print instructions on your screen as to how to chroot in to the LVM if that was what you were needing to do.

lvm-mount.sh
{{File| lvm-mount.sh | <pre class="clear">
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
</pre>}}

==Conclusion==

See that wasn't so hard now was it.

~Az

[[Category:Filesystems|Mount LVM]]
