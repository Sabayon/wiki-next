Fresh Install - what to do

So you just installed a fresh copy of Sabayon Lixux and are wondering what to do next.

Entropy is the package manager and will be vital to know. First thing you will want to do is sort your mirrors. All commands must be run as root, so su or sudo.

It is a good idea to optimize the sorting of mirrors so that all package upgrades will be downloaded as quickly as possible. The choice will depend on what repository you are using, choices are:

limbo(sabayon-limbo) - main(sabayonlinux.org) - weekly(sabayon-weekly) | to see which one your system is using simply do:

 equo status

Issue the proper command according to your status

equo repo mirrorsort sabayon-weekly
equo repo mirrorsort sabayonlinux.org
equo repo mirrorsort sabayon-limbo

Now update the package lists from the mirrors

equo update

If you run into problems with that then try:

equo update --force

Once you have that completed it is vital to get Entropy upgraded to the latest version before doing a full system upgrade. Upgrade will bring your system to current development.

equo install sys-apps/entropy rigo equo  --relaxed


equo conf update

Once the Entropy code is upgraded to the latest version, fully upgrade the rest of your system with these two commands:

equo update


equo upgrade --ask

Follow what is happening on the screen, as Entropy will show you what it is going to do and ask for confirmations.

The 'equo update' command will update the database on your PC with the latest information on packages available in the Entropy repositories;

The 'equo upgrade' command will download from the repositories the binary files for new versions of packages installed on your PC and then install the new versions of those packages.

Time of process depends on how many packages, bandwidth and hardware. After it is done, make sure to:

equo conf update

You will want to make your selection but you really should get to know your config files as they will change your system. More than likely most will select -5. I always look over the config files as I don't want some of my configs getting overwritten.

The final step is to run the following commands, checking for missing dependencies and stability:

equo deptest


equo libtest

After this has finished, reboot and enjoy your freshly installed fully updated Sabayon.

There is a graphical user interface(gui) that you can use instead called Rigo. Rigo will allow you to sort your mirrors, manage repositories, do system upgrades with a click of a button and deal with config files. Get to know how to use equo incase you can't access Rigo.

A Note on Repositories:

    Limbo - testing packages, things may break, advanced users

    Main - After limbo and made sure things are stable the package is moved to Main

    Weekly - The most stable as packages have been through limbo and main.

You can manage repositories with equo or Rigo

equo repo disable sabayon-limbo


equo repo enable sabayon-limbo

Now your set to use equo:

To install a package, the --ask amendment is optional but recommended.

equo install <package> --ask

Removing Packages

equo remove <package> --ask

Search for a Package

equo search <package>

Cleanup downloaded packages

Every now and then you will want to cleanup the packages that Entropy downloaded:

equo cleanup

See help for more equo commands

equo --help
