# Entropy - Package Manager

## About


Entropy is the name of the Sabayon Linux binary package management system. This is the name for the complete infrastructure, composed by Equo client (textual), Sulfur and Rigo client (graphical).

Sabayon is based on Gentoo's testing branch, which is about on par with Debian Sid releases. Entropy takes packages from Gentoo testing and they are pre-compiled, then offered to you in binary form. There is a time delay from when Sabayon compiles these packages for Entropy and when you receive them. It is recommended to only use 1 of the package managers(either Entropy or Portage) to avoid any possible conflicts as a result of the time delay. Generally, Entropy packages will be slightly more stable because they will have already been released in Gentoo testing for a period of time(exact amount of time varies), prior to being released in Entropy.

**Some highlights:**

* Gentoo Linux compatible (caution, mixing entropy and portage is for advanced users)
* Takes the best from Portage, Yum and APT
* Fast as lightning
* SQLite-powered (embedded)
* Smart and User-centric
* Powerful Packages: multiple packages inside one single archive (Smart Packages)
* Supports self-contained applications (Smart Applications)
* Backward Compatible Packages: they can be used in Gentoo Linux after a quick conversion
* Multiple branches support (each branch is a release version)
* Database corruptions aware: rescue and system health scanning tools included
* Easy to deploy and use in a Network Environment
* Multiple repositories aware: everyone can create one
* Extensible and Human Understandable API
* Strongest Artificial Intelligence (Entropy has a brain)
* Great sense of humour, and much more...
    
## Basic Usage

Equo is the command line client-side application for the Entropy package management system, and should be always performed as Root. It is capable of installing, removing and updating packages, resolving dependences, reverse dependency handling and configuration file handling and that's just to start with.
Basic Usage

There are several options you can use when using Equo, a few of the basic commands are shown below.

Searching for a package can be accomplished by running the equo search command:

    equo search foo

To install a package use the install function, the --ask amendment is optional but recommended.

    equo install foo --ask

To remove a package use the remove function as shown below:

    equo remove foo --ask

To upgrade all your packages to the latest versions use this command:

    equo upgrade --ask

As you can see from the examples above, the "--ask" amendment is optional, but highly recommended, as it not only gives you more information about the packages being installed, but also the dependencies that may come with them, giving you more control about what is going to be installed, followed by a confirmation/abortion of the command. 

You can always use help to see a complete list of commands

     equo --help
     
## Rigo

Rigo is the graphical user interface for package management.

**Features**

* "google search" like interface
* very simple and straight forward
* Rigo is faster and more responsive
* append the various packages by browsing
* Easy manage repositories
* Show list of pending configuration files to update
* Very detailed Package information
* List of Installed Applications
* "Activity" button (bottom notification box), that shows the current app management queue.
* Send votes and add comments to Packages
* and much more...
    
[![Click to make Bigger](http://photosbykjs.us/sabayon/rigo1-1.jpg)](http://photosbykjs.us/sabayon/rigo1.png)
[![Click to make Bigger](http://photosbykjs.us/sabayon/rigo2-2.jpg)](http://photosbykjs.us/sabayon/rigo2.png)

## Kernel-Switcher

kernel-switcher is an easy-to-use tool to simplify upgrading the kernel in Sabayon Linux. Remember, doing regular upgrades will not upgrade major version of the kernel, only minor (I.e. 3.12.3 → 3.12.4: YES || 3.12.3 → 3.13.0: NO). To upgrade major version of the kernel, you need to invoke a kernel change. 

    # kernel-switcher --help
 
     >> kernel-switcher - Sabayon Linux Kernel Switcher BETA
     >>   switch kernel:     kernel-switcher switch <kernel package>
     >>   list kernels:      kernel-switcher list
     >>   this help:         kernel-switcher help
     
The kernel-switcher list command is a nice feature, but can be overwhelming as it lists all kernels currently available in the repository. You may prefer to use **equo search linux-sabayon** as linux-sabayon is the Sabayon kernel package. With equo search linux-sabayon you can see if any newer kernels exist. For example, if you find that linux-sabayon-3.8.0 is available as an upgrade, you would upgrade to it as follows:

     # kernel-switcher switch linux-sabayon-3.8.0
     >>  @@ Calculating dependencies …
     >>  ## [U] [sabayonlinux.org] sys-kernel/linux-firmwares-3.8.0|0   [3.8.0|0]
     >>  ## [N] [sabayonlinux.org] sys-kernel/linux-sabayon-3.8.0|0
     >>  ## [N] [sabayonlinux.org] net-wireless/broadcom-sta-5.100.82.38-r1#3.8.0-sabayon|0
     >>  ## [N] [sabayonlinux.org] x11-drivers/nvidia-drivers-260.19.29#3.8.0-sabayon|0

