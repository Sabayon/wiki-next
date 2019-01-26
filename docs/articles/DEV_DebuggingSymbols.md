Packages are, by default, installed without debugging symbols in Sabayon. This saves space, specially for users that are not developers and are not interested in debugging applications. But for those of us who are developers, finding the debug symbols of Entropy packages can come in handy. To do so, change the value of the <tt>splitdebug</tt> variable to enable in <tt>/etc/entropy/client.conf</tt>

 # Enable the installation of debug files
 # Also known as "splitdebug" support
 # Valid parameters: disable, enable, true, false, disabled, enabled, 0, 1
 # Default parameter if unset: disable
 splitdebug = enable
 # HOW SPLITDEBUG WORKS with Entropy
 # Once you enable the "splitdebug" feature
 # you just need to (re)install packages in order to
 # get /usr/lib/debug metadata files installed. That's it.
 # You can safely remove /usr/lib/debug without affecting
 # Operating System functionality, at any time.

After modifying this, next time you installa package, you will also get debug symbols installed inside <tt>/usr/lib/debug</tt>. For example, if you run <tt>equo install networkmanager</tt> you will get the following debugging files:

 >>  ### /usr/lib/debug
 >>  ### /usr/lib/debug/usr
 >>  ### /usr/lib/debug/usr/bin
 >>  ### /usr/lib/debug/usr/bin/nm-online.debug
 >>  ### /usr/lib/debug/usr/bin/nm-tool.debug
 >>  ### /usr/lib/debug/usr/bin/nmcli.debug
 >>  ### /usr/lib/debug/usr/lib
 >>  ### /usr/lib/debug/usr/lib/NetworkManager
 >>  ### /usr/lib/debug/usr/lib/NetworkManager/libnm-settings-plugin-ifnet.so.debug
 >>  ### /usr/lib/debug/usr/lib/libnm-glib-vpn.so.1.1.0.debug
 >>  ### /usr/lib/debug/usr/lib/libnm-glib.so.4.2.0.debug
 >>  ### /usr/lib/debug/usr/lib/libnm-util.so.2.1.0.debug
 >>  ### /usr/lib/debug/usr/lib/pppd
 >>  ### /usr/lib/debug/usr/lib/pppd/2.4.5
 >>  ### /usr/lib/debug/usr/lib/pppd/2.4.5/nm-pppd-plugin.so.debug
 >>  ### /usr/lib/debug/usr/libexec
 >>  ### /usr/lib/debug/usr/libexec/nm-avahi-autoipd.action.debug
 >>  ### /usr/lib/debug/usr/libexec/nm-crash-logger.debug
 >>  ### /usr/lib/debug/usr/libexec/nm-dhcp-client.action.debug
 >>  ### /usr/lib/debug/usr/libexec/nm-dispatcher.action.debug
 >>  ### /usr/lib/debug/usr/sbin
 >>  ### /usr/lib/debug/usr/sbin/NetworkManager.debug


===glibc missing debug info===

The following two problems may be resolved by adding debug symbols for the package sys-libs/glibc.


====GDB: libpthread library mismatch with libthread_db====

If there is no debug symbols for libpthread library, GDB will not directly say that there is no debugging symbols for libpthread, instead, whenever you start GDB, it will show below warning,

<pre>
warning: Unable to find libthread_db matching inferior's thread library, thread debugging will not be available.
</pre>

See this [http://bugs.sabayon.org/show_bug.cgi?id=3316 link] for more information.

====Valgrind: Needs a non-stripped LD====

When executing valgrind with no option in particular (for example: `valgrind /bin/true`) you may get the following error message:

<pre>
valgrind:  Fatal error at startup: a function redirection
valgrind:  which is mandatory for this platform-tool combination
valgrind:  cannot be set up.  Details of the redirection are:
valgrind:  
valgrind:  A must-be-redirected function
valgrind:  whose name matches the pattern:      strlen
valgrind:  in an object with soname matching:   ld-linux-x86-64.so.2
valgrind:  was not found whilst processing
valgrind:  symbols from the object with soname: ld-linux-x86-64.so.2
valgrind:  
valgrind:  Possible fixes: (1, short term): install glibc's debuginfo
valgrind:  package on this machine.  (2, longer term): ask the packagers
valgrind:  for your Linux distribution to please in future ship a non-
valgrind:  stripped ld.so (or whatever the dynamic linker .so is called)
valgrind:  that exports the above-named function using the standard
valgrind:  calling conventions for this platform.  The package you need
valgrind:  to install for fix (1) is called
valgrind:  
valgrind:    On Debian, Ubuntu:                 libc6-dbg
valgrind:    On SuSE, openSuSE, Fedora, RHEL:   glibc-debuginfo
valgrind:  
valgrind:  Note that if you are debugging a 32 bit process on a
valgrind:  64 bit system, you will need a corresponding 32 bit debuginfo
valgrind:  package (e.g. libc6-dbg:i386).
valgrind:  
valgrind:  Cannot continue -- exiting now.  Sorry.
</pre>


====Resolution====

To fix both these issues, just enable splitdebug inside /etc/entropy/client.conf and reinstall sys-libs/glibc package.

<pre>
# equo install sys-libs/glibc
</pre>

There is no need to reboot the system or restart anything.
