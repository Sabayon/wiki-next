# Systemd Guide

`systemd` is a **System and Service Manager for Linux**, compatible with SysV and LSB init scripts.
systemd provides aggressive parallelization capabilities, uses socket and D-Bus activation for starting services,
offers on-demand starting of daemons, keeps track of processes using Linux control groups,
supports snapshotting and restoring of the system state, maintains mount and automount points
and implements an elaborate transactional dependency-based service control logic.


# systemctl usage

## the basics

### Verifying Bootup
As many of you know, systemd is the new init system.

Traditionally, when booting up a Linux system, you see a lot of little messages passing by on your screen, if they are shown at all, given we use graphical boot splash technology like Plymouth these days. 

Nonetheless the information of the boot screens was and still is very relevant, because it shows you for each service that is being started as part of bootup, whether it managed to start up successfully or failed (with those green or red [ OK ] or [ FAILED ] indicators).


systemd has feature that tracks and remembers for each service whether it started up successfully, whether it exited with a non-zero exit code, whether it timed out, or whether it terminated abnormally (by segfaulting or similar), both during start-up and runtime.  By simply typing systemctl in your shell you can query the state of all services, both systemd native and SysV/LSB services:

<pre class="clear">
 # root @ 15:51:25 ] bwg-inc # ]¬ systemctl
 # UNIT                        LOAD   ACTIVE SUB       DESCRIPTION
 # boot.automount              loaded active waiting   boot.automount
 # proc-sys...t_misc.automount loaded active waiting   Arbitrary Executable File Fo
 # sys-devi...und-card0.device loaded active plugged   NM10/ICH7 Family High Defini
 # sys-devi...-net-p1p1.device loaded active plugged   RTL8101E/RTL8102E PCI Expres
 # sys-devi...et-wlp3s0.device loaded active plugged   AR242x / AR542x Wireless Net
 # -.mount                     loaded active mounted   /
 # home2.mount                 loaded active mounted   /home2
 # tmp.mount                   loaded active mounted   /tmp
 # ntpd.service                loaded maintenance  maintenance    Network Time Service 
 # LOAD   = Reflects whether the unit definition was properly loaded.
 # ACTIVE = The high-level unit activation state, i.e. generalization of SUB.
 # SUB    = The low-level unit activation state, values depend on unit type.
 #
 # 99 loaded units listed. Pass --all to see loaded but inactive units, too.
 # To show all installed unit files use 'systemctl list-unit-files'.
</pre>

Look at the ACTIVE column, which shows you the high-level state of a service, whether it is active (i.e. running), inactive (i.e. not running) or in any other state.   If you look closely you'll see one item in the list that is marked maintenance and highlighted in red. This informs you about a service that failed to run or otherwise encountered a problem. 

In this case this is ntpd. Now, let's find out what actually happened to ntpd, with the systemctl status command:
<pre class="clear">
 # root @ 15:52:45 ] bwg-inc # ]¬ systemctl status ntpd.service
 # ntpd.service - Network Time Service
 #	  Loaded: loaded (/etc/systemd/system/ntpd.service)
 #	  Active: maintenance
 # 	    Main: 953 (code#exited, status#255)
 # 	  CGroup: name=systemd:/systemd-1/ntpd.service
</pre>

This shows us that NTP terminated during runtime (when it ran as PID 953), and tells us exactly the error condition: 
<pre class="clear">
 the process exited with an exit status of 255.
</pre>

### Killing Services
Killing a system daemon is easy, right? Or is it?

Sure, as long as your daemon persists only of a single process this might actually be somewhat true. 

You type killall rsyslogd and the syslog daemon is gone.

But here comes systemd to the rescue: With 'systemctl kill' you can easily send a signal to all processes of a service. Example:
<pre class="clear">
 # systemctl kill crond.service
</pre>

This will ensure that SIGTERM is delivered to all processes of the crond service, not just the main process.

Of course, you can also send a different signal if you wish. For example, if you are bad-ass you might want to go for SIGKILL right-away:
<pre class="clear">
 # systemctl kill -s SIGKILL crond.service
</pre>
And there you go, the service will be brutally slaughtered in its entirety, regardless how many times it forked, whether it tried to escape supervision by double forking or fork bombing.


Sometimes all you need is to send a specific signal to the main process of a service, maybe because you want to trigger a reload via SIGHUP. 

Instead of going via the PID file, here's an easier way to do this:
<pre class="clear">
 # systemctl kill -s HUP --kill-who=main crond.service
</pre>

How does this relate to systemctl stop? kill goes directly and sends a signal to every process in the group, however stop goes through the official configured way to shut down a service, i.e. invokes the stop command configured with `ExecStop=` in the service file. Usually stop should be sufficient. kill is the tougher version.


### stop, disable, or mask a service...   The Three Levels of "Off"
In systemd, there are three levels of turning off a service (or other unit). Let's have a look which those are:

1. You can stop a service. That simply terminates the running instance of the service and does little else.:
<pre class="clear">
 # systemctl stop ntpd.service
</pre>

2. You can disable a service. This unhooks a service from its activation triggers. That means, that depending on your service it will no longer be activated on boot, by socket or bus activation or by hardware plug (or any other trigger that applies to it). 

However, you can still start it manually if you wish. If there is already a started instance disabling a service will not have the effect of stopping it. :
<pre class="clear">
 # systemctl disable ntpd.service
</pre>

Disabling a service is a permanent change; until you undo it it will be kept, even across reboots.

3. You can mask a service. This is like disabling a service, but on steroids. It not only makes sure that service is not started automatically anymore, but even ensures that a service cannot even be started manually anymore. This is a bit of a hidden feature in systemd, 

since it is not commonly useful and might be confusing the user. But here's how you do it:
<pre class="clear">
 # systemctl mask ntpd.service
 # ln -s /dev/null /etc/systemd/system/ntpd.service
</pre>

By symlinking a service file to `/dev/null` you tell systemd to never start the service in question and completely block its execution. 

Unit files stored in `/etc/systemd/system` override those from `/usr/lib/systemd/system` that carry the same name. 

The former directory is administrator territory, the latter terroritory of your package manager. By installing your symlink in `/etc/systemd/system/ntpd.service` you make sure that systemd will never read the upstream shipped service file `/usr/lib/systemd/system/ntpd.service`.

## Processes and Common Admin Tools

### Which Service Owns Which Processes?
In systemd every process that is spawned is placed in a control group named after its service.

Control groups (or cgroups) are simply groups of processes that can be arranged in a hierarchy and labelled individually.

When processes spawn other processes, these children-processes are automatically made members of the parents cgroup.

Cgroups can be used as an effective way to label processes after the service they belong to and be sure that the service cannot escape from the label, Regardless how often it forks or renames itself. 

Here i discuss two commands you may use to relate systemd services and processes. `ps`  and `systemd-cgls`

### ps
<pre class="clear">
 # ps xawf -eo pid,user,cgroup,args
 #   PID USER     CGROUP                      COMMAND
 #   271 root     4:cpuacct,cpu:/system/crond /usr/sbin/crond -n
 #   272 root     4:cpuacct,cpu:/system/atd.s /usr/sbin/atd -f
 #   273 root     4:cpuacct,cpu:/system/kdm.s /usr/bin/kdm vt1
 #   281 root     4:cpuacct,cpu:/system/kdm.s  \_ /usr/bin/X :0 vt2 -background none -nolisten tcp -seat seat0 -auth /var/run/kdm/A:0-DZRMfb
 #   287 root     2:name=systemd:/user/1000.u  \_ -:0             
 #   351 apostee+ 2:name=systemd:/user/1000.u      \_ awesome
 #   376 apostee+ 2:name=systemd:/user/1000.u          \_ /usr/bin/ssh-agent /bin/sh -c exec -l /bin/bash -c "awesome"
 #   296 polkitd  4:cpuacct,cpu:/system/polki /usr/lib/polkit-1/polkitd --no-debug
 #   311 root     4:cpuacct,cpu:/system/dbus. /usr/sbin/modem-manager
 #   316 root     4:cpuacct,cpu:/system/bluet /usr/sbin/bluetoothd -n
 #   326 rpc      4:cpuacct,cpu:/system/rpcbi /sbin/rpcbind -w 
 #   339 root     4:cpuacct,cpu:/system/wpa_s /usr/sbin/wpa_supplicant -u -f /var/log/wpa_supplicant.log -c /etc/wpa_supplicant/wpa_supplicant.
 #   365 apostee+ 2:name=systemd:/user/1000.u dbus-launch --sh-syntax --exit-with-session
 #   366 apostee+ 2:name=systemd:/user/1000.u /bin/dbus-daemon --fork --print-pid 4 --print-address 6 --session
 #   422 apostee+ 2:name=systemd:/user/1000.u xscreensaver
 #   424 apostee+ 2:name=systemd:/user/1000.u conky
</pre>
In the third column you see the cgroup systemd assigned to each process.

If you want, you can set the shell alias psc (~/,bashrc) to the ps command line shown above:
<pre class="clear">
 # alias psc='ps xawf -eo pid,user,cgroup,args'
</pre>

### systemd-cgls
Another way to present the same information is the systemd-cgls tool which is shipped with systemd.

 It shows the cgroup hierarchy in a pretty tree. Its output looks like this:
 <pre class="clear">
 # systemd-cgls
 # │ ├─systemd-logind.service
 # │ ├─alsa-state.service
 # │ │ └─253 /usr/sbin/alsactl -s -n 19 -c -E ALSA_CONFIG_PATH#/etc/alsa/alsactl.conf --initfile#/lib/alsa/init/00main rdaemon
 # │ ├─systemd-udevd.service
 # │ │ └─163 /usr/lib/systemd/systemd-udevd
 # │ └─systemd-journald.service
 # │   └─147 /usr/lib/systemd/systemd-journald
 # └─user
 #   └─1000.user
 #     └─1.session
 #       ├─ 287 -:0             
 #       ├─ 351 awesome
 #       ├─ 365 dbus-launch --sh-syntax --exit-with-session
 #       ├─ 366 /bin/dbus-daemon --fork --print-pid 4 --print-address 6 --session
 #       ├─ 376 /usr/bin/ssh-agent /bin/sh -c exec -l /bin/bash -c "awesome"
 #       ├─ 422 xscreensaver
 #       ├─ 424 conky
 #       ├─ 464 /usr/libexec/at-spi-bus-launcher
 #       ├─ 469 /usr/libexec/gvfsd
 #       ├─ 497 /usr/lib/firefox/firefox
 #       ├─2267 /usr/bin/python /usr/bin/terminator
 #       ├─2275 gnome-pty-helper
 #       ├─2276 /bin/bash
 #       ├─2318 su
 #       ├─2326 bash
 #       ├─2480 systemd-cgls
 #       └─2481 less
</pre>
As you can see, this command shows the processes by their cgroup, as systemd labels the cgroups after the services.

If you look closely you will notice that a number of processes have been assigned to the cgroup /user. 

systemd does not only maintains services in cgroups, but user session processes as well.

# journalctl usage

let's start with some basics. To access the logs of the journal use the journalctl tool. 

To have a first look at the logs, just type in:
<pre class="clear">
 # journalctl
</pre>

If you run this as root you will see all logs generated on the system, from system components the same way as for logged in users.  The output you will get looks like a pixel-perfect copy of the traditional `/var/log/messages` format, but actually has a couple of improvements over it:

* Lines of error priority (and higher) will be highlighted red.
* Lines of notice/warning priority will be highlighted bold.
* The timestamps are converted into your local time-zone.
* The output is auto-paged with your pager of choice (defaults to less).
This will show all available data, including rotated logs.

### Access Control

Browsing logs this way is already pretty nice. 

But requiring to be root sucks of course, even administrators tend to do most of their work as unprivileged users these days.

By default, Journal users can only watch their own logs, unless they are root or in the adm group.

To make watching system logs more fun, you could add yourselve to adm:
<pre class="clear">
 # usermod -a -G adm yourusername
</pre>

After logging out and back in as yourusername you have access to the full journal of the system and all users:
<pre class="clear">
 $ journalctl
</pre>

### Live View

If invoked without parameters journalctl will show you the current log database. 

Sometimes one needs to watch logs as they grow, where one previously used `tail -f /var/log/messages`:
<pre class="clear">
 $ journalctl -f
</pre>

Yes, this does exactly what you expect it to do: it will show you the last ten logs lines, and then wait for changes and show them as they take place.


### Basic Filtering

When invoking journalctl without parameters you'll see the whole set of logs, beginning with the oldest message stored.

That of course, can be a lot of data. Much more useful is just viewing the logs of the current boot:
<pre class="clear">
 $ journalctl -b
 </pre>
 
This will show you only the logs of the current boot, with all the gimmicks mentioned. 

But sometimes even this is way too much data to process.

So let's just listing all the real issues to care about: all messages of priority levels ERRORS and worse, from the current boot:
<pre class="clear">
 $ journalctl -b -p err
</pre>

But, if you reboot only seldom the -b makes little sense, filtering based on time is much more useful:
<pre class="clear">
 $ journalctl --since#yesterday
</pre>
And there you go, all log messages from the day before at 00:00 in the morning until right now. Awesome!

Of course, we can combine this with -p err or a similar match. But suppose, we are looking for something that happened on the 15th of October, or was it the 16th?
<pre class="clear">
 $ journalctl --since#2012-10-15 --until#"2011-10-16 23:59:59"
</pre>

And there we go, we found what we were looking for. But i noticed that some CGI script in Apache was acting up earlier today, let's see what Apache logged at that time:
<pre class="clear">
 $ journalctl -u httpd --since#00:00 --until#9:30
</pre>

There we found it. But... , wasn't there an issue with that disk /dev/sdc? Let's figure out what was going on there:
<pre class="clear">
 $ journalctl /dev/sdc
</pre>

Ouch ! a disk error! Hmm, maybe quickly replace the disk before we lose data.

Wait...  didn't I see that the vpnc binary was nagging? Let's check for that:
<pre class="clear">
 $ journalctl /usr/sbin/vpnc
</pre>

I don't get this, this seems to be some weird interaction with dhclient, let's see both outputs, interleaved:
<pre class="clear">
 $ journalctl /usr/sbin/vpnc /usr/sbin/dhclient
</pre>

As you can see here with the given examples, Journalctl is a pretty advanced tool, than can track down pretty much anything.

But we're not done yet. Journalctl has some more to offer, which will be showed in the section Advanced Usage.

## Advanced Administration
### Disable Paging
You can change and disable paging using the `$SYSTEMD_PAGER` environment variable. You may end up with truncated lines, if you set it to "" or `cat` as suggested in the manual. Try this:
<pre class="clear">
 export SYSTEMD_PAGER#"cat|cat"
</pre>

Add it to `~/.bashrc` or `/etc/profile.d/` to make it permanent.

### Advanced Filtering

Internally systemd stores each log entry with a set of implicit meta data.

This meta data looks a lot like an environment block, but actually is a bit more powerful.

This implicit meta data is collected for each and every log message, without user intervention. 

The data will be there, and wait to be used by you. Let's see how this looks:
<pre class="clear">
 $ journalctl -o verbose -n
 $ Fri, 2013-11-01 19:22:34 CET [s#ac9e9c423355411d87bf0ba1a9b424e8;i#4301;b#5335e9cf5d954633bb99aefc0ec38c25;m#882ee28d2;t#4ccc0f98326e6;x#f21e8b1b0994d7ee]
        PRIORITY=6
        SYSLOG_FACILITY=3
        _MACHINE_ID=a91663387a90b89f185d4e860000001a
        _HOSTNAME=epsilon
        _TRANSPORT=syslog
        SYSLOG_IDENTIFIER=avahi-daemon
        _COMM=avahi-daemon
        _EXE=/usr/sbin/avahi-daemon
        _SYSTEMD_CGROUP=/system/avahi-daemon.service
        _SYSTEMD_UNIT=avahi-daemon.service
        _SELINUX_CONTEXT=system_u:system_r:avahi_t:s0
        _UID=70
        _GID=70
        _CMDLINE=avahi-daemon: registering [epsilon.local]
        MESSAGE=Joining mDNS multicast group on interface wlan0.IPv4 with address 172.31.0.53.
        _BOOT_ID=5335e9cf5d954633bb99aefc0ec38c25
        _PID=27937
        SYSLOG_PID=27937
        _SOURCE_REALTIME_TIMESTAMP=1351029098747042
</pre>

(I cut out a lot here, I don't want to make this story overly long. Without the -n parameter it shows you the last 10 log entries, but I cut out all but the last.)

With the -o verbose switch we enabled verbose output. Instead of showing a pixel-perfect copy of classic 

`/var/log/messages` that only includes a minimimal subset of what is available, 

we now see all the details the journal has about each entry, but it's highly interesting: there is user credential information.

Now, as it turns out the journal database is indexed by all of these fields, out-of-the-box! Let's try this out:
<pre class="clear">
 $ journalctl _UID=70
</pre>

And there you go, this will show all log messages logged from Linux user ID 70. 

As it turns out you can easily combine these matches:
<pre class="clear">
 $ journalctl _UID=70 _UID=71
</pre>

Specifying two matches for the same field will result in a logical OR combination of the matches. 

All entries matching either will be shown, i.e. all messages from either UID 70 or 71


If you specify two matches for different field names, they will be combined with a logical AND. 

All entries matching both will be shown now, meaning that all messages from processes named avahi-daemon and host bwg-inc.
<pre class="clear">
 $ journalctl _HOSTNAME=bwg-inc _COMM=avahi-daemon
</pre>

But of course, that's not fancy enough for us. We must go deeper:
<pre class="clear">
 $ journalctl _HOSTNAME=bwg-inc _UID=70 + _HOSTNAME#epsilon _COMM=avahi-daemon
</pre>

The + is an explicit OR you can use in addition to the implied OR when you match the same field twice. 

The line above means: show me everything from host bwg-inc with UID 70, or of host epsilon with a process name of avahi-daemon.


### And now it becomes Magic

Who can remember all those values a field can take in the journal, I mean, who has that kind of photographic memory? 

Well, the journal has:
<pre class="clear">
 $ journalctl -F _SYSTEMD_UNIT
</pre>

This will show us all values the field `_SYSTEMD_UNIT` takes in the database, or in other words: 

the names of all systemd services which ever logged into the journal. This makes it super-easy to build nice matches.

# Systemd Timers

systemd is capable of taking on a significant subset of the functionality of Cron through built-in support for calendar time events as well as monotonic time events.

While we previously used Cron, systemd also provides a good structure to set up Cron-scheduling. 

## Running a single script
Let’s say you have a script `/usr/local/bin/myscript` that you want to run every hour.

* *service file*
First, create a service file, and put it in `/etc/systemd/system/`
<pre class="clear">
 # nano -w /etc/systemd/system/myscript.service
</pre>

with the following content:
<pre class="clear">
[Unit]
Description=MyScript

[Service]
Type=simple
ExecStart=/usr/local/bin/myscript
</pre>

Note that it is important to set the Type variable to be “simple”, not “oneshot”. 

Using “oneshot” makes it so that the script will be run the first time, and then systemd 

thinks that you don’t want to run it again, and will turn off the timer we make next.

* **timer file**
Next, create a timer file, and put it also in the same directory as the service file above.
<pre class="clear">
 # nano -w /etc/systemd/system/myscript.timer
</pre>

with the following content:
<pre class="clear">
[Unit]
Description=Runs myscript every hour

[Timer]
# Time to wait after booting before we run first time
OnBootSec=10min
# Time between running each consecutive time
OnUnitActiveSec=1h
Unit=myscript.service

[Install]
WantedBy=multi-user.target
</pre>


* **enable/start**
Rather than starting / enabling the service file, you use the timer.
<pre class="clear">
 # systemctl start myscript.timer
</pre>
and enable it with each boot:
<pre class="clear">
 # systemctl enable myscript.timer
</pre>

## Running Multiple Scripts on the Same Timer

Now let’s say there are a bunch of scripts you want to run, all at the same time. 

In this case, you will want make a couple changes on the above formula.

* **service files**
Create the service files to run your scripts as showed previously, but include the following section at the end of each service file:
<pre class="clear">
[Install]
WantedBy=mytimer.target
</pre>

If there is any ordering dependency in your service files, be sure you specify it with 

the *After=something.service* and/or *Before=whatever.service* parameters within the 

Description section.


* **timer file**
You only need a single timer file. Create mytimer.timer, as outlined above.


* **target file**
You can create the target that all these scripts depend upon.,
<pre class="clear">
 # nano -w /etc/systemd/system/mytimer.target
</pre>

with the following content:
<pre class="clear">
[Unit]
Description=Mytimer
# Lots more stuff could go here, but it's situational.
# Look at systemd.unit man page.
</pre>


* **enable/start**
You need to enable each of the service files, as well as the timer:
<pre class="clear">
systemctl enable script1.service
systemctl enable script2.service
...
systemctl enable mytimer.timer
systemctl start mytimer.service
</pre>

## Hourly, daily and weekly events

One strategy which can be used for creating this functionality is through timers which call in targets. All services which need to be run hourly can be called in as dependencies of these targets. 

First, the creation of a few directories is required:
<pre class="clear">
 # mkdir /etc/systemd/system/timer-{hourly,daily,weekly}.target.wants
</pre>

The following files will need to be created in the paths specified in order for this to work.

* **hourly events**
<pre class="clear">
 # nano -w /etc/systemd/system/timer-hourly.timer
 </pre>
 
with it's content:
<pre class="clear">
[Unit]
Description=Hourly Timer

[Timer]
OnBootSec=5min
OnUnitActiveSec=1h
Unit=timer-hourly.target

[Install]
WantedBy=basic.target
</pre>
<pre class="clear">
 # nano -w /etc/systemd/system/timer-hourly.target
</pre>

with it's content:
<pre class="clear">
[Unit]
Description=Hourly Timer Target
StopWhenUnneeded=yes
</pre>



* **daily events**
<pre class="clear">
 # nano -w /etc/systemd/system/timer-daily.timer
</pre>

content:
<pre class="clear">
[Unit]
Description=Daily Timer

[Timer]
OnBootSec=10min
OnUnitActiveSec=1d
Unit=timer-daily.target

[Install]
WantedBy=basic.target
</pre>

<pre class="clear">
 # nano -w /etc/systemd/system/timer-daily.target
</pre>

content:
<pre class="clear">
[Unit]
Description=Daily Timer Target
StopWhenUnneeded=yes
</pre>



* **weekly events**
<pre class="clear">
 # nano -w /etc/systemd/system/timer-weekly.timer
</pre>

content:
<pre class="clear">
[Unit]
Description=Weekly Timer

[Timer]
OnBootSec=15min
OnUnitActiveSec=1w
Unit=timer-weekly.target

[Install]
WantedBy=basic.target
</pre>

<pre class="clear">
 # nano -w /etc/systemd/system/timer-weekly.target
</pre>

content:
<pre class="clear">
[Unit]
Description=Weekly Timer Target
StopWhenUnneeded=yes
</pre>



* **adding events**
Adding events to these targets is as easy as dropping them into the correct wants folder. 

So if you wish for a particular event to take place daily, create a systemd service file and drop it into the relevant folder.

For example, if you wish to run mlocate-update.service daily (which runs mlocate), you would create the following file:
<pre class="clear">
 # nano -w /etc/systemd/system/timer-daily.target.wants/mlocate-update.service
</pre>

<pre class="clear">
[Unit]
Description=updates the mlocate database

[Service]
User=                                          # Add a user if you wish the service to be executes as a particular user, else delete this line
Type=                                          # Simple by default, change it if you know what you are doing, else delete this line
Nice=19
IOSchedulingClass=2
IOSchedulingPriority=7
ExecStart=/usr/bin/updatedb --option1 --option2     # More than one ExecStart can be used if required
</pre>


* **enable and start the timers**
<pre class="clear">
 # systemctl enable timer-{hourly,daily,weekly}.timer && systemctl start timer-{hourly,daily,weekly}.timer
</pre>

## Starting events according to the calendar
If you wish to start a service according to a calendar event and not a monotonic interval (i.e. you wish to replace the functionality of crontab), you will need to create a new timer and link your service file to that. An example would be:
<pre class="clear">
# nano -w /etc/systemd/system/foo.timer
</pre>

content:
<pre class="clear">
[Unit]
Description=foo timer

[Timer]
OnCalendar=Mon-Thu *-9-28 *:30:00 # To add a time of your choosing here, please refer to systemd.time manual page for the correct format
Unit=foo.service

[Install]
WantedBy=basic.target
</pre>

The service file may be created the same way as the events for monotonic clocks. However, take care to put them in the `/etc/systemd/system/` folder.

# timedatectl - - Control the system time and date.

systemd brings a new way of setting the system time and date, and making sure that time in system is correct.

## time, date, and timezone ##
The check status:
<pre class="clear">
 # timedatectl status
</pre>

gives you a very nice overview of the current settings of the system clock and RTC.
Example:
<pre class="clear">
      Local time: Sun 2014-02-02 15:56:40 CET
  Universal time: Sun 2014-02-02 14:56:40 UTC
        RTC time: Sun 2014-02-02 14:56:40
        Timezone: Europe/Amsterdam (CET, +0100)
     NTP enabled: n/a
NTP synchronized: no
 RTC in local TZ: no
      DST active: no
 Last DST change: DST ended at
                  Sun 2013-10-27 02:59:59 CEST
                  Sun 2013-10-27 02:00:00 CET
 Next DST change: DST begins (the clock jumps one hour forward) at
                  Sun 2014-03-30 01:59:59 CET
                  Sun 2014-03-30 03:00:00 CEST
</pre>

As you can see, it even shows at which date the clock will jump one hour forward/back.


If, for some reason the time or date is not right., you can change it. Synax:
<pre class="clear">
 # timedatectl set-time [time]
</pre>

The time may be specified in the format: "2013-11-30 18:17:16". Thus:
<pre class="clear">timedatectl set-time 2013-11-30 18:17:16</pre>


Your Timezone can also be specified, using `timedatectl set-timezone`.

Available time zones can be listed with:
<pre class="clear">
 timedatectl list-timezones
</pre>

Set your time zone of choice:
<pre class="clear">
 timedatectl set-timezone Europe/Amsterdam
</pre>

## time synchronization (NTP)

To enable NTP  (NTP based network time synchronization):
* Install NTP:
<pre class="clear">equo install net-misc/ntp</pre>


* Enable service to start at boot and start it now:
<pre class="clear">systemctl enable ntpd && systemctl start ntpd</pre>


* Enable time synchronization with NTP:
<pre class="clear">timedatectl set-ntp 1</pre>


Fore more info:
<pre class="clear">
 # timedatectl -h
 # man timedatectl
</pre>

# localectl - - Control the system locale and keyboard layout settings#

localectl may be used to query and change the system locale and keyboard layout settings.

*The system locale controls the language settings of system services and of the UI before

the user logs in, such as the display manager, as well as the default for users after login.


*The keyboard settings control the keyboard layout used on the text console and of the

graphical UI before the user logs in, such as the display manager, as well as the default for users after login.


The syntaxes are:
<pre class="clear">
 # localectl status       (shows your current locale settings:
 # localectl set-locale <variant>       ("localectl list-locales" will show all known locales)
 # localectl set-keymap <map> <map>       ("localectl list-keymaps" will show all known virtual console keyboard mappings)
 # localectl set-x11-keymap LAYOUT [MODEL] [VARIANT] [OPTIONS]
</pre>

for a complete listing of all available x11-keymap layouts, models, variants and options use: 
<pre class="clear">
 # localectl list-x11-keymap-layouts
 # localectl list-x11-keymap-models
 # localectl list-x11-keymap-variants
 # localectl list-x11-keymap-options
</pre>

Now, say you want to set the system locale to: "en_US.UTF-8", enter:
<pre class="clear">localectl set-locale LANG#en_US.UTF-8</pre>


To set the virtual console keyboard map to US, enter:
<pre class="clear">localectl set-keymap us it</pre> 
Note here that you can enter a *second* keymap to define a toggle keyboard mapping.


To set the system default keyboard mapping for X11, and define your model, variant, and some options, enter:
<pre class="clear">localectl set-x11-keymap us apple mac_nodeadkeys numpad:pc</pre>
So, in the above example the user has a Apple aluminum mac keyboard, no deadkeys, and a PC compatible numeric pad.


* --no-convert
If set-keymap or set-x11-keymap is invoked and this option is passed then the keymap will not be converted 

from the console to X11, or X11 to console, respectively.


It means that if this option is *NOT* passed to set-keymap, the selected setting is also applied to the default

keyboard mapping of X11, after converting it to the closest matching X11 keyboard mapping, and vice versa.


For a complete overview and list of all possible commands/options of localectl, please enter:
<pre class="clear">
 # localectl --help
</pre>
or
<pre class="clear">
 # man localectl
</pre>

# analyzing and performance

## analyzing the boot process 

*WARNING !!!*

Before actually trying to speedup the boot process, you really have to know what you are doing and how to revert your actions.

When moving, disabling or masking service files without knowing why, the boot process can take even longer than before.

 

systemd provides a tool called *systemd-analyze* that can be used to show timing details about the boot process, 

including an svg plot showing units waiting for their dependencies. You can see which unit files are causing your boot process 

to slow down. You can then optimize your system accordingly.


To see how much time was spent in kernelspace and userspace on boot, simply use:
<pre class="clear">systemd-analyze</pre>


To list the started unit files, sorted by the time each of them took to start up:
<pre class="clear">systemd-analyze blame</pre>


At some points of the boot process, things can not proceed until a given unit succeeds. 

To see which units find themselves at these critical points in the startup chain, do:
<pre class="clear">systemd-analyze critical-chain</pre>


You can also create a SVG file which describes your boot process graphically, similiar to Bootchart:
<pre class="clear">systemd-analyze plot > plot.svg</pre>
See *man systemd-analyze* for details.

## analyzing using bootchart 

## readahead

Systemd comes with its own readahead implementation, this should in principle improve boot time. 

However, depending on your kernel version and the type of your hard drive, your mileage may vary (i.e. it might be slower). 

To enable, do:
<pre class="clear">systemctl enable systemd-readahead-collect systemd-readahead-replay</pre>
In order for the readahead to work, you should reboot a couple of times, because it needs to adapt to the changed boot process.

## filesystem mounts

If btrfs is in use for the root filesystem, there is no need for a fsck on every boot like other filesystems. If this is the case,you may want to mask the systemd-fsck-root.service 

systemd will still fsck any relevant filesystems with the systemd-fsck@.service

You can also remove API filesystems from /etc/fstab, as systemd will mount them itself.

It is not uncommon for users to have a /tmp and/or /dev/shm entry carried over from sysvinit, but you may have noticed that systemd already takes care of this.  If you want to give /tmp a size, say: 4Gb., you can edit the file:
<pre class="clear">nano -w /usr/lib/systemd/system/tmp.mount</pre>
and add *size#4096M* to section: [Mount] :
<pre class="clear">
[Mount]
What#tmpfs
Where#/tmp
Type#tmpfs
Options#mode#1777,strictatime,size#4096M
</pre>


If on seperate partitions, other filesystems like /home and /boot can be mounted with custom mount units. 

Adding noauto,x-systemd.automount will buffer all access to that partition, and will fsck and mount it on first access, reducing the number of filesystems it must fsck/mount during the boot process.


*NOTE:* this will make your /home and /boot filesystem type autofs, which is ignored by mlocate by default. 

If you use mlocate, and want /home and/or /boot still to be indexed by mlocate., edit the file:
<pre class="clear">/etc/updatedb.conf</pre>}}
and remove the entries from the "PRUNEPATHS#" " :
<pre class="clear">
PRUNE_BIND_MOUNTS = "yes"
PRUNEFS = "9p afs anon_inodefs auto autofs bdev binfmt_misc cgroup cifs coda configfs cpuset cramfs debugfs devpts devtmpfs ecryptfs exofs ftpfs fuse fuse.enc$
PRUNENAMES = ".git .hg .svn"
PRUNEPATHS = "/afs /media /mnt /home/home2 /net /sfs /tmp /udev /var/cache /var/lib/pacman/local /var/lock /var/run /var/spool /var/tmp"
</pre>

The speedup of automounting /home & /boot may not be more than a second or two, depending on your system, so this trick may not be worth it.
<pre class="clear">nano -w /etc/fstab

/dev/sda3	/               ext4    noatime,defaults        			1 1
/dev/sda1	/boot           ext4    noatime,noauto,x-systemd.automount 	        1 2
/dev/sda2	/home           ext4    noatime,noauto,x-systemd.automount		1 2
/dev/sda5	none            swap    sw						0 0
</pre>

# debugging

* [http://freedesktop.org/wiki/Software/systemd/Debugging/ *Debugging systemd Problems*)]


# systemd for Administrators


All links mentioned here, will lead you outside this Wiki...


* [http://0pointer.de/blog/projects/changing-roots.html *Changing Roots*)]

* [http://0pointer.de/blog/projects/the-new-configuration-files *The New Configuration Files*)]

* [http://0pointer.de/blog/projects/instances.html *Instantiated Services*)]

* [http://0pointer.de/blog/projects/inetd.html *Converting inetd Services*)]

* [http://0pointer.de/blog/projects/systemd-for-admins-3.html *Converting a SysVinit script into systemd service file*)]

* [http://0pointer.de/blog/projects/security.html *Securing Your Services*)]

* [http://0pointer.de/blog/projects/watchdog.html *Watchdogs*)]

* [http://0pointer.de/blog/projects/serial-console.html *Gettys on Serial Consoles)]

* [http://0pointer.de/blog/projects/resources.html *Managing Resources)]

* [http://0pointer.de/blog/projects/detect-virt.html *Detecting Virtualization*)]

* [http://www.freedesktop.org/wiki/Software/systemd/APIFileSystems/ *API File Systems*)]

# SysVinit to systemd cheatcheet

http://fedoraproject.org/wiki/SysVinit_to_Systemd_Cheatsheet

# systemd -index — - List of all manpages from the systemd project
http://www.freedesktop.org/software/systemd/man/

