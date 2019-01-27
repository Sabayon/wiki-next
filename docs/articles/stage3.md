{{i18n| [[De:HOWTO: Install from an existing Linux system|de]] [[HOWTO: Install from an existing Linux system|en]] [[It:HOWTO: Installare da un sistema linux esistente|it]] [[Tr:HOWTO: Install from an existing Linux system|tr]]}}

;Overview

This HOWTO brings the gentoo installation method (as in the Handbook) to Sabayon.

Please note that "an existing Linux system" can be either an installed system (then you cannot use that partition for sabayon), or a live linux system.

The first part is about installing a very basic gentoo system in a chrooted partition; in the second part we will use <tt>portage</tt> to install <tt>entropy</tt> and <tt>equo</tt>. We will then use <tt>equo</tt> to install a full sabayon system in the third and last part.

== Part 1: Gentoo install ==

;Please refer to http://www.gentoo.org/doc/en/gentoo-x86-quickinstall.xml and http://www.gentoo.org/doc/en/handbook/index.xml for more in-depth details about gentoo installation.

=== Partitioning ===

Do your thing and partition your disk(s). You need at least a root partition (/).

Create filesystems for your partitions according to your likings. Remember that you will need to install the *progs packages for your filesystem(s).

From now on we will assume that your root filesystem is mounted under <tt>/mnt/sabayon</tt>.

Also if you have a <tt>/var</tt>, <tt>/boot</tt>, <tt>/usr</tt>, <tt>/home</tt> ... filesystems, create the mountpoints relative to /mnt/sabayon and mount them now.

=== Get Gentoo installation files ===

We will assume an x86_64 architecture here. If you're on x86 (32 bits), change "amd64" with "x86".

{{Console|<pre class="clear">export ARCH="amd64"</pre>}}

Pick up a mirror near you from this list: http://www.gentoo.org/main/en/mirrors.xml

{{Console|<pre class="clear">export MIRROR="ftp://distfiles.gentoo.org/pub/gentoo"</pre>}}

We will refer to your arch as <tt>$ARCH</tt> and to your chosen mirror as <tt>$MIRROR</tt> from now on.

==== Download and verify stage3 archive ====

{{Console|<pre class="clear">
wget -r $MIRROR/releases/$ARCH/current-stage3/
cat stage3-$ARCH-*.tar.bz2.DIGESTS && md5sum stage3-$ARCH-*.tar.bz2 && sha1sum stage3-$ARCH-*.tar.bz2
</pre>}}

==== Download and verify portage snapshot ====

{{Console|<pre class="clear">
wget $MIRROR/snapshots/portage-latest.tar.bz2
wget $MIRROR/snapshots/portage-latest.tar.bz2.md5sum
cat portage-latest.tar.bz2.DIGESTS && md5sum portage-latest.tar.bz2 && sha1sum portage-latest.tar.bz2
</pre>}}

=== Unpack stage3 and portage ===

{{Console|<pre class="clear">
tar xfpj stage3-$ARCH-*.tar.bz2 -C /mnt/sabayon
tar xfpj portage-latest.tar.bz2 -C /mnt/sabayon/usr
</pre>}}

Note the <tt>p</tt> option: it's for preserving file permissions and it's very important!

=== Chroot into your new Gentoo ===

Everything that follows is done while in <tt>/mnt/sabayon</tt>:

{{Console|<pre class="clear">cd /mnt/sabayon</pre>}}

==== Mounting filesystems ====

{{Console|<pre class="clear">
mount -o bind /dev ./dev
mount -t devpts none ./dev/pts
mount -t tmpfs none ./dev/shm
mount -t proc none ./proc
mount -t sysfs none ./sys
</pre>}}

==== Copying <tt>/etc/resolv.conf</tt> ====

{{Console|<pre class="clear">cp /etc/resolv.conf ./etc</pre>}}

==== Chroot! ====

{{Console|<pre class="clear">chroot /mnt/sabayon /bin/bash</pre>}}

Now you are on your Gentoo chroot!

==== Configure environnment ====

{{Console|<pre class="clear">
export LANG=en_US
export LANGUAGE=${LANG}
export LC_ALL=${LANG}.UTF-8
env-update
eselect python set 1    #sets python 2.7 as default
source /etc/profile
</pre>}}

==== Some early configuration ====

===== Set up a root password =====
    
As I '''regulary''' forget to setup the root password, let's do that as first thing:

{{Console|<pre class="clear">passwd</pre>}}

===== Timezone =====

{{Console|<pre class="clear">        
cd /etc/
ln -sf ../usr/share/zoneinfo/<REGION>/<City> localtime
</pre>}}

Where <REGION> is your region and <City> the nearest big city.

===== Hostname =====
    
Be original and change "localhost" to something else. Update <tt>/etc/hosts</tt> accordingly.

{{Console|<pre class="clear">
nano -w /etc/conf.d/hostname
nano -w /etc/hosts
</pre>}}

===== Mount points =====

{{Console|<pre class="clear">nano -w /etc/fstab</pre>}}

== Part 2: Sabayon overlay setup and installation of <tt>entropy</tt>+<tt>equo</tt> ==

=== Layman ===

<tt>layman</tt> is a gentoo tool to manage portage overlays. We will use it to get our hands on the sabayon overlay.

{{Note|entropy and equo are in regular portage, hence: '''skip this step'''.}}

{{Console|<pre class="clear">
# emerge --sync                # make sure portage is fully up to date... or not. Up to you. portage snapshots are done daily, i see no need to sync.
# USE="git" emerge -avt layman # now equo is in regular portage!
# layman -a sabayon            # see above
</pre>}}

=== Entropy and Equo ===

{{Console|<pre class="clear">
emerge -avt equo --autounmask-write
etc-update
emerge -avt equo
</pre>}}

Now we have <tt>entropy</tt> and <tt>equo</tt> installed. To make use of it, we should first generate the entropy database ('''only the very first time!'''):

{{Console|<pre class="clear">equo rescue generate</pre>}}

Yes, you are sure about this. You should answer yes three times to make it happy.

Perfect. Now we need to setup sabayon repositories. As an example, we use '''sabayonlinux.org''' (i.e. updated daily), but you can use '''sabayon-weekly''' instead if you want to upgrade less often. See [[En:Entropy#Package_Repositories]].

{{Console|<pre class="clear">
cd /etc/entropy/repositories.conf.d
cp entropy_sabayonlinux.org.example entropy_sabayonlinux.org
cd -
</pre>}}

Let's use <tt>equo</tt> to populate the repo db (if you used sabayon-weekly, substitute "sabayonlinux.org" with sabayon-weekly here):

{{Console|<pre class="clear">    
equo update
equo repo mirrorsort sabayonlinux.org
</pre>}}

We're set! <tt>equo</tt> is now in a working state!

=== Optional: Entropy: the missing bits ===

'''Notice: if you are planning to exclusively use entropy, you can happily skip to [[HOWTO:_Install_from_an_existing_Linux_system#Part_3:_Finish_installation_using_equo|Part 3: Finish installation using equo]]'''.

----

So you will be using portage and/or want to have the same portage settings used to build your sabayon packages? Very well, here's how!

When you install entropy you'll miss some vital portage bits, namely the /etc/portage/* files used to build the sabayon packages.

Hopefully you can find those here: http://github.com/Sabayon/build

Let's make use of them!

{{Console|<pre class="clear">
# fetch the bits!
cd /opt
git clone git://github.com/Sabayon/build.git sabayon-build
cd /opt/sabayon-build/conf/intel/portage
# keep your specific stuff in "myconf" branch:
git checkout -b myconf
# symlink to your <arch>:
ln -sf make.conf.amd64 make.conf
ln -sf package.env.amd64 package.env
# add & commit
git add make.conf package.env
git config --global user.name "Your Name"
git config --global user.email "your@email"
git commit
# rename the gentoo /etc/make.conf and /etc/portage/:
cd /etc/
mv portage portage-gentoo
mv make.conf make.conf-gentoo
# symlink to sabayon /etc/make.conf /etc/portage/:
ln -sf /opt/sabayon-build/conf/intel/portage portage
</pre>}}

Of course, change <tt>make.conf.amd64</tt> and <tt>package.env.amd64</tt> to <tt>make.conf.x86</tt> and <tt>package.env.x86</tt> if you are not on x86_64.

If you want to change some USE flags on certain packages you should:
* update your /etc/portage/package.use
* mask the package in /etc/entropy/packages/package.mask

so that emerge will use your specified USE flags and entropy will not handle the specified package again.

i.e. you want ruby support for <tt>app-editors/vim</tt>:

{{Console|<pre class="clear">
cd /etc/portage
sed -i package.use -e "s,app-editors/vim vim-pager,app-editors/vim vim-pager ruby" # or just use VIM!
git add package.use
git commit -m "app-editors/vim +ruby"
equo mask app-editors/vim
</pre>}}

== Part 3: Finish installation using <tt>equo</tt> ==

=== eix ===

To make your life easier, let's install <tt>eix</tt> (a tool for portage searching) and also the sabayon-distribution overlay (i.e. the <tt>entropy</tt> packages as a portage overlay):

{{Console|<pre class="clear">
equo install eix
layman -a sabayon-distro
cd /etc/portage
echo "source /var/lib/layman/make.conf" >> make.conf
git commit -m "source layman/make.conf in /etc/make.conf"
eix-sync
</pre>}}

Now you can use <tt>eix</tt> to quickly search for installed/installable/upgradable packages and see their versions and use flags.

=== Filesystem progs ===

From gentoo quickinstall, code listing 2.27:

{{Console|<pre class="clear">
equo install xfsprogs # (if you use the XFS file system)
equo install jfsutils # (if you use the JFS file system)
</pre>}}

=== Network stuff ===

{{Console|<pre class="clear">
equo install dhcpcd         # (if you need a DHCP client)
equo install ppp            # (if you need PPPoE ADSL connectivity)
equo install wireless-tools # (if you need wireless connectivity)
equo install wpa_supplicant # (if you need WPA/WPA2 authentication)
equo install wicd           # (if you do like wicd)
</pre>}}

=== Kernel & bootloader ===

{{Console|<pre class="clear">
equo install linux-sabayon
equo install grub2
nano -w /etc/default/sabayon-grub
</pre>}}

/etc/default/sabayon-grub example:
<pre>
GRUB_CMDLINE_LINUX="<boot options specific to your system… root=... dolvm crypt real_root etc…> console=tty1 splash=silent,theme:sabayon quiet"
</pre>
Put boot options according to your system. It should at least be able to mount your rootfs (/).

Run grub2-mkconfig to generate /boot/grub/grub.cfg
{{Console|<pre class="clear">
mount /boot
grub2-mkconfig -o /boot/grub/grub.cfg
umount /boot
</pre>}}

=== System logger and Cron daemon ===

{{Console|<pre class="clear">
equo install syslog-ng vixie-cron
rc-update add syslog-ng default
rc-update add vixie-cron default
</pre>}}

This is just a suggestion: please go for the system logger and cron daemon of your choice!

=== Everything else ===

Install any other packages you want using equo:

{{Console|<pre class="clear">equo install <whatever></pre>}}

For example, if you want KDE, the base packages needed can be installed with <tt>equo install kdebase-meta</tt>

=== Exit & cleanup ===

==== Exit the chroot ====

{{Console|<pre class="clear">exit</pre>}}

==== Unmount filesystems ====

{{Console|<pre class="clear">
cd /mnt/sabayon
umount ./sys
umount ./proc
umount ./dev/shm
umount ./dev/pts
umount ./dev
</pre>}}

Unmount any other filesystem inside /mnt/sabayon.

Then unmount the root filesystem of your new sabayon system:

{{Console|<pre class="clear">umount /mnt/sabayon</pre>}}

=== Reboot ===

You are now ready to reboot (well, you were ready since the "setup grub" step, actually)

{{Console|<pre class="clear">reboot</pre>}}

Enjoy your new shiny Sabayon system installed like it should be! (please forgive the troll in me)
