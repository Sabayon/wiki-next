{{Warning| '''If you have lost or do not remember the passphrase you set, your data is just gone and there is nothing that can be done about it. Dont even ask.'''}}


=Mounting Encrypted Volumes/Partitions=

*NOTE: Since the Anaconda installer has been added to Sabayon iso's disk and/or partition encryption is simply a matter of enabling a check box during installation and adding a password. It's quite easy even for new users. You may also wish to read the following link:
* [http://wiki.sabayon.org/index.php?title=En:Security#Hard_Disk_Encryption_.26_boot_options Hard Disk Encryption & boot options]

For those wishing or needing to perform this from the command line continue reading the following:

==Using LVM==
If you are also using LVM then all of the steps found in that how-to '''EXCEPT''' issuing the mount command must be done first. The LVM must be prepped prior to your decrypting it. A change should also be noted that with LVM instead of decrypting something like /dev/sda3 you instead will find the volume to decrypt in /dev/mapper.

You will have to do this manually, as the script that is in place in the LVM how-to automatically mounts the appropriate LVM volumes. If you use that script anyways and things dont work, dont say you haven't been warned.


==Mounting/Decrypting==
From the livecd you will need to go root, on the livecd the password for root is root.
{{Console|<pre class="clear">$ su 
Password: <type root here if on livecd></pre>}}
Next we will need a little information, namely which partition that the encrypted volume is on. If you have used encryption and LVM together you will have to mount the LVM first as documented elsewhere. For now, if you dont know or remember which partition you installed on we need to find out.
{{Console|<pre class="clear"># fdisk -lu </pre>}}
There is no real way to see which exactly is the encrypted partition. What we can do here is look to see which is the largest partition with a linux type. If you have a single linux install, then typically this will be the largest partition listed as linux in the table. In other words look at how many blocks each has.

Now that we have it we can set up to mount it.
{{Console|<pre class="clear"># cd /mnt
# mkdir enc </pre>}}
This made a mount point that we can use in a moment. Next we get to actually get the system to recognize and make the partition usable.
{{Console|<pre class="clear"># cryptsetup luksOpen /dev/<whateveryourdeviceis> encrypted
# <whateverpassphrase>  </pre>}}
*Note that the <> are not used in the actual command /dev/sda2 is an example of what you might put there. We also assigned a name to it, that will come in handy in a moment.*

Finally we will mount the volume with:
{{Console|<pre class="clear"># mount /dev/mapper/encrypted /mnt/enc</pre>}}
There we go, your volume is mounted. You now have access your data, make changes, chroot in, or whatever else that you needed to access the volume from the outside. Once you are done with whatever it was make sure to cleanly unmount the volume with :
{{Console|<pre class="clear">
# umount /mnt/enc
# cryptsetup luksClose /dev/mapper/encrypted</pre>}}

Have a great one
~Az
