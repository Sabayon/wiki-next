# TEAM RED - AMD

<pre class="clear">
######### SED #########
In these instructions you'll notice the sed command. This may make some people leary as not 
everyone understands this command. "sed" stands for "stream editor". It can edit the edit file 
for you by substituting lines, words, strings, etc. In this case we are using it to remove a 
line in blacklist.conf and nomodeset command in grub config files.

If you don't feel fully comfortable about this command. 
Back up your file or use nano (vi etc.) to edit the file!
</pre>

## EXPERIMENTAL AMDGPU w/Display Code

<pre class="clear">
######### Not all AMD Display Code is not mainline! #########
This contains incomplete code that has yet not been adopted into the mainline kernel. 
Results may vary and this may not be the same as the finalized versions of code.
</pre>
Be aware, this is experimental and you're on your own. This is not a supported setup. It only allows you early access to AMDGPU upgrades.
You'll also want to be aware that you should still keep your current kernel in case anything goes wrong. That will allow you to continue booting into your system from the advanced options in GRUB by selecting the older kernel. 


Lets get started with prep work!
<pre class="clear">
localhost root # equo repo enable sabayon-limbo
localhost root # equo update; equo upgrade
localhost root # equo install v86d bison make cmake automake gcc genkernel-next dracut sabayon-dracut git 
</pre>

Now lets download a kernel with display code.
<pre class="clear">
localhost root # git clone -b amd-staging-drm-next --depth=1 git://people.freedesktop.org/~agd5f/linux    
localhost root # mv linux /usr/src/linux-5.0.0-rc1+
localhost root # cd /usr/src
localhost root # rm -rf linux
localhost root # ln -s linux-5.0.0-rc1+/ linux
localhost root # cd linux
</pre>

Now we finish configuring and building the kernel. Now's you chance to make any other config changes you want. If you use lvm or luks add the --lvm or --luks flag to the genkernel command.
You can also double check Device Drivers -> Graphics Support -> Display Engine Configuration -> AMD DC is enabled as well as Raven family if you plan on owning an APU based on Raven.
<pre class="clear">
localhost root # zcat /proc/config.gz > /usr/src/config
localhost root # genkernel --kernel-config=/usr/src/config --menuconfig --splash=sabayon kernel
localhost root # dracut -H -q -f -o systemd -o systemd-initrd -o systemd-networkd -o dracut-systemd --kver=5.0.0-rc1+ /boot/initramfs-genkernel-x86_64-5.0.0-rc1+
localhost root # grub2-mkconfig -o /boot/grub/grub.cfg 
</pre>

You can watch the git repo for updates/fixes/changes etc at https://cgit.freedesktop.org/~agd5f/linux/log/?h=amd-staging-drm-next to decide if you want to update the kernel again. If you do this, none of the sed commands are necessary for the config as /proc/config.gz will already have the updated config changes.

# <span style="color:green">TEAM GREEN - NVIDIA</span>

## Booting live/installer disc with vesa graphics on older GPUs
Can't seem to get the GUI on the installer disc and seem stuck at a black screen or terminal? We've got you covered. When selecting an option from the boot screen edit the boot line and include the following at the end of the line. 
<pre class="clear">
modprobe.blacklist=nvidia xdriver=vesa
</pre>
This should get you into the live session to perform an install. Afterwards, you'll need to either switch to Nouveau or a proprietary driver version that supports your card.

## Proprietary
Sabayon should already come with the latest nvidia-drivers, but if for some reason you don't have correct proprietary Nvidia drivers installed there are simple steps to follow.

### Prepare if returning from nouveau
Because we use the proprietary drivers by default, the opensource driver is usually already blacklisted; but if you switched to the opensource driver and are returning to the proprietary offerings, you must first blacklist the opensource driver.
<pre class="clear">
localhost root # sed -i s/'#blacklist nouveau'/'blacklist nouveau'/ /etc/modprobe.d/blacklist.conf
</pre>
### Available driver version

To get a list of all drivers for all kernels that are available:
<pre class="clear">
localhost root # equo search -qv nvidia-drivers
</pre>
List of all drivers versions available for currently running kernel:
<pre class="clear">
localhost root # equo search -qv nvidia-drivers#$(uname -r)
</pre>
### Newest drivers for currently running kernel
<pre class="clear">
localhost root # equo install --ask nvidia-driver#$(uname -r) nvidia-userspace
</pre>
Will install newest driver available for currently running kernel.

### Older drivers for currently running kernel

<pre class="clear">
######### Not supported or Guaranteed #########
It has been noticed that depending on the old GPU or GPU drivers version, they may or may not work. 
This seems pretty hit and miss.Mobile GPUs and non-discrete GPUs (integrated in the motherboard) 
appear to be affected by this. If you're using an GPU older than Fermi or an integrated GPU, 
we recommend using the open source nouveau driver.
</pre>

List Nvidia drivers for your current kernel (insturctions above), and install it along with corresponding '''nvidia-userspace''', e.g.:
<pre class="clear">
localhost root # equo install --ask nvidia-drivers-340.104#$(uname -r) nvidia-drivers-340.104
</pre>
now lets switch from opensource opengl libraries to proprietary nvidia opengl libraries
<pre class="clear">
localhost root # eselect opengl set nvidia
</pre>
## Open Source - Switching to Nouveau drivers
If Sabayon is installed you reboot and you find you're stuck at a login terminal or just wan to switch to open source drivers, this is a simple fix. 
Just login with your user you created. Then we'll remove the proprietary drivers and the blacklist for nouveau using the following:

### Unblock nouveau driver
Because we use the proprietary drivers by default, the opensource driver is blacklisted. You must first unblock the driver from being loaded.
<pre class="clear">
localhost root # sed -i s/'blacklist nouveau'/'#blacklist nouveau'/ /etc/modprobe.d/blacklist.conf
</pre>
You still must block or remove the proprietary driver. Now that both drivers have the ability to load, it will most likely lock up the machine
on boot or cause instability. Both drivers cannot be loaded at the same time.

### Removal of proprietary driver
You can blacklist the nvidia drivers OR remove them entirely. Removal is the simplest method.
<pre class="clear">
localhost root # equo update; equo remove nvidia-drivers nvidia-userspace
</pre>
Now lets make sure we're using the correct opengl libraries
<pre class="clear">
localhost root # eselect opengl set xorg-x11
</pre>
Now reboot and you'll be using nouveau. If you ever feel like returning to proprietary drivers, you'll need to blacklist nouveau again.
