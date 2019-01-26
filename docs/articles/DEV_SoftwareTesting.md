= '''Joining Software Testing Team''' = 

If you wish to assist with software testing for Sabayon Linux, assistance is always welcome.  We need testers to download ISO images and make sure they boot, install, no crashes/failures, and live on limbo repository. I wouldn't recommend this to someone who isn't familiar with virtual machines, partitioning, or is afraid of losing data on their machine. If you feel comfortable with those requirements, you may be a great candidate! Please contact someone in the #sabayon-dev IRC channel and let them know you're interested in joining the testing team. You will be contacted and request your email to be added to the testing group.


= '''What kind of skills do I need (or need to learn) to join testing?''' =

That actually depends on what skills you have and would like to learn. We're flexible and can teach you what you need to know. You just need to be prepared in case your machine loses its data. Backups! We plan to perform installs on virtual machines(BIOS/UEFI), bare metal machines(BIOS/UEFI), configure machines to our preferences, test graphics and wireless drivers/cards, check md5sums of ISO images, check the mirrors for up to date images, check that all hardware is recognized and working out of the box, run benchmarks, check for regressions, and look for solutions.  The point is to catch problems before they arise for normal users looking for a mainstream release. You'll be living on the Sabayon bleeding edge in the limbo repository getting package updates before the rest of our users. Do you use your linux machine in a way that many people don't? Even better! this gives us more thorough testing.

Even if you're fairly new to Linux, as long as you have the drive, patience, and willingness to learn. We can teach you what you need and/or want to know. IRC isn't always a quick communication method and tends to be used like old BBS message boards at times. You may not get an immediate response. We're all volunteers that have families and day jobs. So if you don't get an immediate response, leave the chat window open. You'll get a response sometime within the day. Some of us are even open to using alternative chat methods such as hangouts for direct contact.


= '''What if the issue isn't Sabayon Linux, but a piece of software?''' = 

We lookup where to report the bugs and we file them accordingly. KDE, Gnome, atom, libreoffice, VLC, etc. gdb will be useful.

They all have a place to report bugs. The more data on the bug and the more users reporting it, the easier it will be to get it fixed.

Get comfortable with some [https://wiki.sabayon.org/index.php?title=Debugging_Symbols_-_splitdebug debugging] if you can.

= '''I'm a Software Tester! Where do I start?''' =

'''First there's the CORE.''' Hardware support, Virtual Machine support, Booting, Installing, Pre-installed basics.

'''Next there's Spin dependent test.''' KDE->K3b working? QT5 working? systemsettings5 segfaulting?

'''What if I notice an issue?''' This issue could be limited by hardware or a fluke by corruption or easily fixed. Time to communicate, email testing@sabayonlinux.org with your findings. We can all test and search out a fix together or file a bug with developers and mark a bug as "confirmed" by having multiple people post findings in the bug. This raises validity and priority.

'''What if I'm looking at software that isn't installed by default?''' Same procedure! We can all look at the problem together to get updates, new software in the repository, patches, etc.

== '''CORE LIST''' ==

# Does sabayon greeter work? (GUI only)
# Does Anaconda work? GUI & Text install?
# Does Custom partitioning, LVM and/or encryption work?
# Does it boot after install? (UEFI and BIOS)
# Is your graphics card recognized? Does 3D acceleration work (OpenGL, Vulkan)?
# Does sound work?
# Does dmesg present any obvious errors?
# Does USB3.X and/or type-C work?
# Does wireless/bluetooth work? (Intel, Broadcom, Atheros, Realtek, etc)
# Check for Library issues? (sudo equo libtest)
# Sticky-bit/privilege issues? (allowed to ping as a user?)
# Test Rigo and Entropy?

== '''SPIN DEPENDENT''' ==
Many of these spin dependent softwares can be tested on a full running install with the limbo repository.
Using an already running install is no replacement for Live disc and Install testing, but can let you test for broken software before its released.

'''KDE''' - systemsettings5, gwenview, okular, k3b, dolphin, networkmanager, lockscreen, libreoffice, plasma5, MTP, steam, chrome.

'''Gnome''' - brasero, control center, lockscreen, MTP, nm-applet, steam, tweaktool, libreoffice, chrome.

'''MATE''' - Similar to Gnome package list + guake.

'''LXQt''' - Does not use Anaconda by default. Should have (experimental) Calamares. Please test Calamares installer. CORE LIST + LXQt desktop, settings, file manager.

'''Xfce''' - CORE LIST + Xfce desktop, settings, file manager.

'''Server & Minimal''' - CORE LIST CLI-Only (No GUI)

'''Forensics''' - This belongs to Wolfden. If any bugs/requests are filed, they should go to Wolfden specifically as well as testing group.


== '''LOOKING DEEPER''' ==
If you're curious and wish to look at performance changes and regressions. You're welcome to dig into newer software and kernels in search of fixes and performance boosts.

Some software that could help with that could be as simple as a few free steam games such as DOTA2 and benchmark tools like [https://benchmark.unigine.com/heaven Unigine Benchmarks], or [http://www.iozone.org/ iozone].

Check [https://www.kernel.org kernel.org] and browse the git tree for linux-next to track long awaited fixes.

Follow [https://www.phoronix.com forums and news] sites to track big changes within the linux community.

Use the testing Mailing List to discuss upcoming upgrades, patches, kernels, etc.
