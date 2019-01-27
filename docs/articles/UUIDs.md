{{i18n| [[HOWTO: UUID and Volumes|en]] [[Tr:HOWTO: UUID and Volumes|tr]]}}

=UUID=

===What is UUID?===
A Universally Unique Identifier (UUID) is an identifier standard used in software construction, standardized by the Open Software Foundation (OSF) as part of the Distributed Computing Environment (DCE).  With UUID Linux kernel should automatically find and map (read as mount to exact location) volumes to storage device. This saves lots of time and avoid /etc/fstab breaks.

===How do I find the UUID to my drive?===
Let's say I have /dev/sda5 and I want to find out the UUID, as root:
     # vol_id --uuid /dev/sda5
     62c289ef-ba8f-43bf-98bd-5150c7821ad8

===How do I list all my volumes?===
     #blkid
     /dev/sda1: UUID="F6EC3083EC304063" LABEL="winxp" TYPE="ntfs"
     /dev/sda5: UUID="62c289ef-ba8f-43bf-98bd-5150c7821ad8" TYPE="ext4dev"
     /dev/sdb1: UUID="14901BF4C42926A1" LABEL="Vista" TYPE="ntfs"
     /dev/sdb5: LABEL="tux2" UUID="8b3d5c13-e858-4a21-9aa7-491770d40d3b" SEC_TYPE="ext2" TYPE="ext3"
     /dev/sdb6: LABEL="tux3" UUID="943a2d7c-dbe9-45e1-9238-adbcf73f31c7" SEC_TYPE="ext2" TYPE="ext3"
     /dev/sdb7: LABEL="boot" UUID="6d993171-1fbf-4e13-a2f8-945da575eb62" SEC_TYPE="ext2" TYPE="ext3"
     /dev/sdb8: UUID="063b6f7d-e2e7-45ee-bd81-365039f42479" TYPE="ext4dev"
     /dev/sdc5: LABEL="STORAGE" UUID="421D-AF1D" TYPE="vfat"
     /dev/sdd5: LABEL="MOVIES" UUID="43F7-33C3" TYPE="vfat"
     /dev/sdd6: UUID="ddef8913-3dfd-4d8a-96da-997c500db8b5" SEC_TYPE="ext2" TYPE="ext3"
     /dev/sdd7: TYPE="swap" UUID="4a56fa8f-c951-4d4c-8d16-164cb4d3467b"
     /dev/sdd8: LABEL="Arch" UUID="f2d7c692-e8f0-483f-85ca-366da0759c63" SEC_TYPE="ext2" TYPE="ext3"
     /dev/sdd9: UUID="EC4EC2504EC212EE" LABEL="Misc" TYPE="ntfs"

=UUID and fstab=
===How do I add this to my fstab?===
You will need to make a directory first.  Lets say I have /dev/sdb5 as storage so I need to set that up for mounting
     # mkdir -p /media/storage
Now I can add the line to my /etc/fstab file
     #UUID=8b3d5c13-e858-4a21-9aa7-491770d40d3b  /media/storage      ext3    defaults,errors=remount-ro 0       1
Now every time I boot, it auto-mounts it for me

===I need to learn fstab===
I would suggest reading this page http://www.tuxfiles.org/linuxhelp/fstab.html

=Handy ls commands=
List devices:
===By volume label:===
    $ ls /dev/disk/by-label -lah
===By id:===
    $ ls /dev/disk/by-id -lah
===By uuid:===
    $ ls /dev/disk/by-uuid -lah

=Renaming a Label=
===e2label===
As time goes on you make changes and may need to change the name of a label.  Very simple with e2label:
     # e2label <dev> <label>
So, for example, if I want to change my sdb5 name from storage to music, I would use the command:
     # e2label /dev/sdb5 music
''Note'': will not work on FAT or NTFS volumes

[[Category:Hardware|UUID and Volumes]]
