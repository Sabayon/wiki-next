# KDE
### WARNING
Before removing a Desktop Environment like KDE, you have to keep a few things in mind;
* it will remove tons and tons of stuff, probably more than you want to be removed.
* Removing too much might leave your system unusable.
* Be sure to have an alternative DM installed. (gdm or lxdm or lightdm)
That said, unless you are running out of hard drive space or have limited bandwidth for updates,there is no real harm in having kde installed. If you don't use it, it won't hinder performance.

## Preparation
For this to work, the best way of doing this is to first install a alternative Desktop Environment.

In this tutorial KDE will be replaced with XFCE.

open a console/terminal and become root. (of course you can also do this with Rigo
install your alternative Desktop Environment:
<pre class="clear">#   equo install @xfce --ask</pre>
from Rigo: enter:***xfce4-meta*** in the searchbox, select the package, and click install,

or install directly using a shortcut in the searchbox: ***do:install xfce4-meta*** accept the licence agreement.
install a alternative LoginManager: (in this case, i choose lxdm)
<pre class="clear">#   equo install lxdm</pre>
while at the terminal, change the default LoginManager to ***lxdm***:
<pre class="clear">#   nano -w /etc/conf.d/xdm</pre>
and replace:
<pre class="clear">DISPLAYMANAGER="kdm"</pre>
with:
<pre class="clear">DISPLAYMANAGER="lxdm"</pre>
When done, save: [CTRL-O] and close: [CTRL-X]
logoff, and logon again into the fresh installed Environment. (NOT KDE)

## Removing KDE
So now we're logged in with XFCE.
open your favourite terminal, and become root.
enter following in the terminal:
<pre class="clear">equo remove kdelibs --ask --deep</pre>
check the package list before proceeding.
### NOTICE It might very well be possible, that Entropy wants to remove vital packages for XFCE as well.,

so chances are that after a reboot the system is unusable.

In that case, when KDE is removed., ***before*** rebooting, (just in case) simply reinstall your new DE, eg.: ***equo install @xfce***

## Final Steps
When done, a few final steps, enter ***equo conf update*** to check if configuration files needs to be updated, manually.

perform a ***equo deptest*** and ***equo libtest***.

and remove the .kde4 directory in your home folder...
<pre class="clear">rm -rf /home/your-username/.kde4</pre>
and ***.kderc***:
<pre class="clear">rm -rf /home/your-username/.kderc</pre>

# Gnome
### WARNING
***Before removing any Desktop Environment***
such as Gnome, you have to keep a few things in mind:
* Many libraries are shared with other DE's, so you don't want them to be removed.
* Removing too much might leave your system unusable.
* Be sure to have an alternative DM installed. (lxdm or lightdm)

That said, unless you are running out of hard drive space or have limited bandwidth for updates,
There is no real harm in having Gnome installed. If you don't use it, it won't hinder performance.


## Preparation
For this to work, the best way of doing this is to first install a alternative Desktop Environment.

In this tutorial Gnome will be replaced with XFCE.

open a console/terminal and become root. (of course you can also do this with Rigo
install your alternative Desktop Environment: this can be KDE (plasma-meta), mate, etc. We're using xfce for this demonstration.
<pre class="clear"># equo install @xfce --ask</pre>
from Rigo: enter:***xfce4-meta*** in the searchbox, select the package, and click install,

or install directly using a shortcut in the searchbox: ***do:install xfce4-meta*** accept the licence agreement.

***GDM*** will also be removed, so install a alternative LoginManager: (in this case, i choose lightdm, it works with everything)
<pre class="clear"># equo install lightdm</pre>

==Switching DMs==
Remove from boot entry starting GDM:
<pre class="clear"># systemctl disable gdm.service</pre>
Now enable new LoginManager to start at boot:
<pre class="clear"># systemctl enable lightdm.service</pre>
When done, logoff, and logon again into the fresh installed Environment. (NOT Gnome)

## Removing Gnome
### WARNING
***Don't forget --nodeps***

without the "--nodeps",  KDE, XFCE, important libraries, and even nvidia-drivers get pulled. This can wreck your system!


So now we're logged in with XFCE.
install and open your favorite terminal (except Gnome-terminal), and become root.
enter following in the terminal:
<pre class="clear">
equo install gentoolkit
equery list "*" | grep 'gnome\|evolution\|folks\|evince\|deja-dup\|brasero\|totem\|mutter\|shotwell\|eog\|cheese\|gedit\|libgdata\|geocode-glib\|gnote\|baobab\|caribou\|rhythmbox\|grilo\|libgweather\|alacarte' | xargs equo remove --nodeps
</pre>

### WARNING
***Do NOT Reboot!***
Do NOT reboot until you've performed the final steps!


## Final Steps

Now we need to check for possible broken dependencies and libraries that may pull something back in.

<pre class="clear">
# equo deptest
# equo libtest
# equo conf update
# reboot
</pre>

# MATE
### NOTICE
Before removing a Desktop Environment, you have to keep a few things in mind;
* Removing too much might leave your system unusable.
* Be sure to have an alternative DM installed. (gdm or lxdm or lightdm)
That said, unless you are running out of hard drive space or have limited bandwidth for updates,

there is no real harm in having Mate installed. If you don't use it, it won't hinder performance.

## Preparation
For this to work, the best way of doing this is to first install a alternative Desktop Environment.

In this tutorial Mate will be replaced with KDE.

open a console/terminal and become root. (of course you can also do this with Rigo
install your alternative Desktop Environment:
<pre class="clear">#   equo install kde-meta --ask</pre>
from Rigo: enter:***kde-meta*** in the searchbox, select the package, and click install,

or install directly using a shortcut in the searchbox: ***do:install kde-meta***.

## Removing Mate
So now we're logged in with KDE.
open your favourite terminal, and become root.
enter following in the terminal:
<pre class="clear">equo query installed mate | xargs equo remove --nodeps</pre>
***ignore all the warnings***
### NOTICE 
without the "--nodeps",   "sys-auth/pambase-20101024-r2" got pulled, and aborts the mission.

Also, ***don't*** use the ***--deep*** flag here

## Final Steps
### NOTICE 
***before*** rebooting, you ***must*** reinstall "sys-apps/dbus" and "x11-apps/scripts"

otherwise the system becomes unusable.
<pre class="clear">equo install sys-apps/dbus x11-apps/scripts</pre>
When done, a final step, enter ***equo conf update*** to check if configuration files needs to be updated, manually.

# XFCE
### NOTICE
Before removing a Desktop Environment like XFCE, you have to keep a few things in mind;
* Removing too much might leave your system unusable.
* Be sure to have an alternative DM installed. (gdm or lxdm or lightdm)
That said, unless you are running out of hard drive space or have limited bandwidth for updates,

there is no real harm in having XFCE installed. If you don't use it, it won't hinder performance.

## Preparation
For this to work, the best way of doing this is to first install a alternative Desktop Environment.

In this tutorial XFCE will be replaced with Mate.

open a console/terminal and become root. (of course you can also do this with Rigo
install your alternative Desktop Environment:
<pre class="clear">#   equo install mate --ask</pre>
from Rigo: enter:***mate*** in the searchbox, select the ***meta***package, and click install,

or install directly using a shortcut in the searchbox: ***do:install mate***.
With removing XFCE, lightdm stays untouched, so we don't have to install a alternative LoginManager.
When done, logoff, and logon again into the fresh installed Environment. (NOT XFCE)

## Removing XFCE
So now we're logged in with Mate.
open your favourite terminal, and become root.
enter following in the terminal:
<pre class="clear">equo remove libxfce4util xfconf --ask</pre>
check the package list before proceeding.
### NOTICE 
In contrary with removing KDE, ***DON'T*** use the "--deep" flag here.

With the "--deep" flag enabled, it removes also some system files, which deptest and libtest don't recover, so chances are that after a reboot the system is unusable.

### WARNING
The packages "mate-base/mate" and "virtual/notification-daemon" are going to be removed as well !!!

Before you reboot, you ***MUST first reinstall those packages.,*** please see Final steps.

## Final Steps
When done, a few final steps.

In the terminal, as root type: 
<pre class="clear">#   equo install mate</pre>
and:
<pre class="clear">#   equo install notification-daemon</pre>
from Rigo: enter:***mate*** in the searchbox, select the ***meta***package, and click install,

and repeat the same steps for the daemon, replacing ***mate*** with: ***notification-daemon***
or install directly using a shortcut in the searchbox: ***do:install mate*** / ***do:install notification-daemon***

Enter ***equo conf update*** in the terminal to check if configuration files needs to be updated, manually.

You could perform a ***equo deptest*** and ***equo libtest***., although not really needed.

and eventually you could remove the packages "gtk-engines-xfce" and "xfce4-taskmanager"
<pre class="clear">equo remove gtk-engines-xfce xfce4-taskmanager</pre>




