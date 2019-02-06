# Molecule
Molecule in a nut shell builds custom iso images.  If want to make your own custom Sabayon Linux, this is the tool you need.  You first need to install the package dev-util/molecule via entropy.  You can follow along with the molecule development via our git http://gitweb.sabayon.org/?p=molecule.git;a=summary molecule.git  You can also find sample files to view via the git repo.

## Installing Molecule
I assume if you are reading this that you are an advanced user and understand how entropy works. 
<pre class="clear">
     #equo update

     #equo install dev-util/molecule
</pre>
You now have molecule installed and ready to use.

## Understanding Molecule
What you are going to do is use an existing Sabayon Linux iso image for molecule to use.  Which iso image you choose is up to you.  Check our [http://www.sabayon.org/download/ mirrors] for an iso image to use.  Molecule is going to jump into that iso image, make the changes you tell it to via the spec file and other shell script files.  Once molecule has done it's changes that you requested, it will spin out your new custom iso image.  There is no gui for this, you run molecule from the command line once you have your spec file and shell scripts set up.  The spec file is the heart of your configurations. 

An [http://gitweb.sabayon.org/?p=molecule.git;a=blob;f=examples/specs/5-x86-g-remaster-add-games.spec;h=5623f42dee8595990663ad2e72697cafd9ce9143;hb=29d45a193430bc8afbda65d0cf19 example spec file] for you to view and use.  

You can use shell script files to further makes changes.  I would consider that an advanced topic as you need to know shell scripting and understand the guts of the existing Sabayon Linux iso.  I'm not gonna touch much on that subject, cause I'm not a programmer.  I will be using some shell script files in this example, very basic.

## Getting Started
### WARNING 
***If using an extra partition for these processes, ensure that it is capable of maintaining proper file attributes. FAT and NTFS are NOT capable, EXT2,3,4 ReiserFS, linux native file systems are. ***

### Setting Up Existing ISO Images Structure
I like to setup a structure and keep things organized so I can keep track of things.  In my home directoy is where I like to store the existing Sabayon iso images.  My home goes something like this:


This is where I keep my 64bit Sabayon dvd iso images:
     `~/isos/amd64`


### Setting up Molecule Structure 
I keep the molecule work out of my home directory and use the / for this.  So I do something like this:

I want to give my molecule stuff a home to hold my spec file(s) and shell script file(s), so I create:
     `/spin`

I need to give a home to my custom ISO images, so I have:
    `/spin/final`

Since I like to have my own background image added into the new custom ISO image, I create a folder for those image files:
     `/spin/background`

That covers everything I need to do.  I like to use the cache abilities of molecule, saves on downloading for each run.  Molecule will create the directories for that on the first run, so molecule will automatically create
     `/sabayon/remaster/pkgs/amd64/5`

## Spec File
Lets take a look at a [https://github.com/wolfden/Coding spec file], looks frightening, but lets go through it section by section.  Bare with me as I just wing it here.

Description and mission, nothing to touch here:
<pre class="clear">
     # Sabayon Linux 5 amd64 GNOME Molecule remaster spec file
     # The aim of this spec file is to add arbitrary applications & misc stuff
     # to an already built ISO image via scripting (providing hooks that call
     # user-defined scripts).
     # squashfs, mkisofs needed
</pre>

Some setting we don't need to bother changing, another words, leave it alone as is
<pre class="clear">
     # Define an alternative execution strategy, in this case, the value must be
     # "iso_remaster"
     execution_strategy: iso_remaster
</pre>

Straight forward, you want to tell molecule where your existing iso image is, use your directory structure that you created
<pre class="clear">
     # Path to source ISO file (MANDATORY)
     source_iso: /home/wolfden/isos/amd64/Sabayon_Linux_amd64_G.iso
</pre>

No idea, safe to ignore this part, nothing to touch, leave as default
<pre class="clear">
     # Error script command, executed when something went wrong and molecule has to terminate the execution
     # environment variables exported:
     # - CHROOT_DIR: path to chroot directory, if any
     # - CDROOT_DIR: path to livecd root directory, if any
     # - SOURCE_CHROOT_DIR: path from where chroot is copied for final handling
     # error_script: /path/to/script/to/be/executed/outside/after
</pre>

I use a shell script here to setup my cache, I will post that shell script further down.  If you do not have this shell script, simple add a # to comment out the line and molecule will ignore it.  You do not need this script, this is for further customization.
<pre class="clear">
     # Outer chroot script command, to be executed outside destination chroot before
     # before entering it (and before inner_chroot_script)
     outer_chroot_script: /spin/remaster_pre.sh
</pre>

No idea, I leave this default
<pre class="clear">
     # Inner chroot script command, to be executed inside destination chroot before packing it
     # - kmerge.sh - setup kernel bins
     #  inner_chroot_script:
</pre>

I use a shell script here to do some clean up after molecule has done it's package installs and removal, I will post that shell script further down.  If you do not have this shell script, simple add a # to comment out the line and molecule will ignore it.  You do not need this script, this is for further customization.
<pre class="clear">
     # Inner chroot script command, to be executed inside destination chroot after
     # packages installation and removal
     inner_chroot_script_after: /spin/inner_chroot_script_after.sh
</pre>

Another shell script I use to do my background image swap, equo cleanup and more cache work. I will post that shell script further down. If you do not have this shell script, simple add a # to comment out the line and molecule will ignore it.  You do not need this script, this is for further customization.
<pre class="clear">
    # Outer chroot script command, to be executed outside destination chroot before
    # before entering it (and AFTER inner_chroot_script)
    outer_chroot_script_after: /spin/remaster_post.sh
</pre>

Leave this area alone, leave as default
<pre class="clear">
    # Extra mkisofs parameters, perhaps something to include/use your bootloader
    extra_mkisofs_parameters: -b isolinux/isolinux.bin -c isolinux/boot.cat
</pre>

Leave this area alone, leave as default
<pre class="clear">
   # Pre-ISO building script. Hook to be able to copy kernel images in place, for example
   # pre_iso_script: /sabayon/scripts/cdroot.py
</pre>

This is where you tell molecule to save your custom made ISO image, use your directory structure.
<pre class="clear">
   # Destination directory for the ISO image path (MANDATORY)
   destination_iso_directory: /spin/final
</pre>

If you are making a cd ISO image, here is where you name it, give it the name and than remove the # in front of the line so molecule will act upon it
<pre class="clear">
   # Destination ISO image name, call whatever you want.iso, not mandatory
   # destination_iso_image_name: Sabayon_Linux_Customcd_amd64_G
</pre>

If you are making a DVD ISO image, give it's name here and remove the # in front of the line so molecule can act upon it.
<pre class="clear">
   # Output iso image title
   iso_title: Sabayon_Linux_CustomDVD_amd64_G
</pre>

No idea, leave as default
<pre class="clear">
   # Alternative ISO file mount command (default is: mount -o loop -t iso9660)
   # iso_mounter:
</pre>

No idea, leave as default
<pre class="clear">
   # Alternative ISO umounter command (default is: umount)
   # iso_umounter:
</pre>

No idea, leave as default
<pre class="clear">
   # Alternative squashfs file mount command (default is: mount -o loop -t squashfs)
   # squash_mounter:
</pre>

No idea, leave as default
<pre class="clear">
   # Alternative ISO squashfs umount command (default is: umount)
   # squash_umounter:
</pre>

No idea, leave as default
<pre class="clear">
   # Merge directory with destination LiveCD root
   # merge_livecd_root: /put/more/files/onto/CD/root
</pre>

This is self explanatory,  put the packages you want to remove here separated by a comma. Make sure no # is at the beginning of the line so molecule can act upon it.  I use foo as an example package name
<pre class="clear">
   # List of packages that would be removed from chrooted system (comma separated)
   packages_to_remove: foo, foo2, foo3, foo4, foo5
</pre>

No idea, leave as default
<pre class="clear">
   # Custom shell call to packages removal (default is: equo remove)
   # custom_packages_remove_cmd:
</pre>

This is self explanatory,  put the packages you want to add here separated by a comma. Make sure no # is at the beginning of the line so molecule can act upon it.  I use foo as an example package name
<pre class="clear">
   # List of packages that would be added from chrooted system (comma separated)
   packages_to_add: foo, foo2, foo3, foo4, foo5
</pre>

No idea, leave as default
<pre class="clear">
   # Custom shell call to packages add (default is: equo install)
   # custom_packages_add_cmd:
</pre>

No idea, leave as default
<pre class="clear">
   # Custom command for updating repositories (default is: equo update)
   # repositories_update_cmd:
</pre>

While molecule is running, entropy does an equo update to update it's database so you can get the latest packages.  You will want this set to yes, otherwise molecule will fail with entropy unable to determine packages since the database isn't updated.
<pre class="clear">
   # Determine whether repositories update should be run (if packages_to_add is set)
   # (default is: no), values are: yes, no.
   execute_repositories_update: yes
</pre>

No idea, leave as default
<pre class="clear">
   # Directories to remove completely (comma separated)
   # paths_to_remove:
</pre>

No idea, leave as default
<pre class="clear">
   # Directories to empty (comma separated)
   # paths_to_empty:
</pre>


## Shell Files
I want to stress that these are for the more advanced stuff and are not needed.  They need to be executable in order to run them.  You need an understanding of existing ISO image.  In my case I didn't realize that the root and user accounts were created on the fly, so if you try making preferences directly to the sabayonuser account, it won't work.  You need to access the skel files instead.  Knowledge is power I always say, sometimes we got to learn things the hard way.  Keep in mind that the first time you run molecule, you may see some error messages with these files, but will work ok and the next time it won't report errors.  Keep in mind also that these are tailored for my custom ISO image, but they give you an idea and you can adopt what you need.

### remaster_pre.sh 
What this does, it sets up my cache.  It will store the packages I add so I don't have to keep re-downloading them each time I want to make a spin.  A great time saver if you have large files to work with.
<pre class="clear">
     #!/bin/sh
     PKGS_DIR="/sabayon/remaster/pkgs"
     CHROOT_PKGS_DIR="${CHROOT_DIR}/var/lib/entropy/client/packages"
     [[ ! -d "${PKGS_DIR}" ]] && mkdir -p "${PKGS_DIR}"
     [[ ! -d "${CHROOT_PKGS_DIR}" ]] && mkdir -p "${CHROOT_PKGS_DIR}"
     echo "Mounting packages over"
     rm -rf "${CHROOT_PKGS_DIR}"/*
     cp ${PKGS_DIR}/* "${CHROOT_PKGS_DIR}"/ -Ra
     exit 0
</pre>
### remaster_post.sh
Again, this is working with the cache files again.  Basically the cache works like this, molecule downloads the packages via chroot, once done, it moves the packages to /sabayon/remaster/pkgs for local storage and the next time you run your spin, it copies the files over and back into the chroot so you don't have to re-download all the packages again.  I also use this file to change the desktop background to the one I created.  I also do an equo clean up to remove packages and reduce ISO image size.
<pre class="clear">
     #!/bin/sh
     PKGS_DIR="/sabayon/remaster/pkgs"
     CHROOT_PKGS_DIR="${CHROOT_DIR}/var/lib/entropy/client/packages"
     echo "Merging back packages"
     cp "${CHROOT_PKGS_DIR}"/* "${PKGS_DIR}"/ -Ra
     rm -rf "${CHROOT_PKGS_DIR}"{,-nonfree,-restricted}/*
     cp /spin/background/sabayon-forensic.png "${CHROOT_DIR}/usr/share/backgrounds/sabayonlinux.png"
     cp /spin/background/sabayon-forensic.jpg "${CHROOT_DIR}/usr/share/backgrounds/sabayonlinux.jpg"
     is_64=$(file "${CHROOT_DIR}"/bin/bash | grep "x86-64")
     if [ -n "${is_64}" ]; then
         echo "equo cleanup" | chroot "${CHROOT_DIR}"
     else
         echo "equo cleanup" | linux32 chroot "${CHROOT_DIR}"
     fi
</pre>
### inner_chroot_script_after.sh
I use this script to do some more clean up stuff.  It's pretty basic
<pre class="clear">
     #!/bin/bash
     #fix clamav freshclam
     touch /var/log/clamav/freshclam.log
     chown clamav:clamav /var/log/clamav/freshclam.log
     #remove desktop icons
     rm /etc/skel/Desktop/*
     #remove no longer needed folders/files
     rm -r /etc/skel/.fluxbox
     rm -r /etc/skel/.kde4
     rm -r /etc/skel/.mozilla
     rm -r /etc/skel/.emerald
     rm -r /etc/skel/.xchat2
     rm -r /etc/skel/.config/compiz
     rm -r /etc/skel/.config/lxpanel
     rm -r /etc/skel/.config/pcmanfm
     rm -r /etc/skel/.config/Thunar
     rm -r /etc/skel/.config/xfce4
     rm -r /etc/skel/.gconf/apps/compiz
     rm -r /etc/skel/.gconf/apps/gset-compiz
     rm /etc/skel/.config/menus/applications-kmenuedit.menu
     rm /etc/skel/.kderc
     emaint --fix world
</pre>
## Running Molecule
So you got your directory structure done, edited a spec file to suit your needs, possibly created some shell script files and ready to roll.  Open a terminal window and make sure you are root and issue
<pre class="clear">
    #molecule /spin/mycustom.spec
</pre>
Replace the mycustom.spec with the name of your spec file of course.  Go make some popcorn and grab a soda and watch the magic happen in your terminal
