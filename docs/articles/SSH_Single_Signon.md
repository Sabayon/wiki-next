# SSH Single SignOn (SSO)

## Concept
If you work with multiple *nix-based machines via ssh, you are probably tired of constantly having to enter your password every time you want to access another box.  There is a secure way to allow you to access every machine, that you have ssh access to, without having to enter another password (other than the one you signed on with originally.)

This is actually quite simple to do, you basically just create a public/private key pair to authenticate yourself to your other machines, then have PAM spawn an agent to load your keys after you logon, providing a single signon solution to accessing all your remote machines.  This guide will walk you through setting this up.


## Equo Stuff
First of all, we are going to need the pam_ssh module.  You may already have this installed, if not equo it.  I'm going to assume you already have openssh installed, as every system should.
<pre class="clear"># equo install pam_ssh </pre>

## Create Key Pair
Now we need to create the key pair to authenticate yourself across the network.  To do so, run this as your **regular user**.
***WARNING: This should NOT be done as root, and you should never, ever ssh using the root account***
<pre class="clear">$ ssh-keygen -t dsa </pre>

This will ask where to save the file, just press enter as the default is what we want.  

After that it will ask for the passphrase you want to use.  It is important to ***set the passphrase to the exact same password as your normal logon password*** for this user.  This is the password for the user on the current machine, not the one for other machines, even if they differ.

When that is done, two files should of been created `~/.ssh/id_dsa` and `~/.ssh/id_dsa.pub` .  
***NOTE: It is very important to keep `~/.ssh/id_dsa private`, it should be only readable by your user, to make sure of this, run this command:***
<pre class="clear">$ chmod 600 ~/.ssh/id_dsa </pre>

## Distribute Public Key
Now we need to give all of our remote machines our public key so we can use it to authenticate.  This is very simple to do, run the following command as your user for each remote system you want to setup the passwordless authentication for.
<pre class="clear">$ ssh-copy-id -i ~/.ssh/id_dsa.pub username@remotehostname </pre>

## Configure PAM
We need to tell PAM to use our logon password to spawn an ssh-agent and load the rsa key we just made.  There are two lines for `pam_ssh.so` we need to add, so your system-auth should look something like this:
`/etc/pam.d/system-auth`
<pre class="clear">
auth       required     pam_env.so
# Add this line here
auth       sufficient   pam_ssh.so

auth       sufficient   pam_unix.so try_first_pass likeauth nullok
auth       required     pam_deny.so

account    required     pam_unix.so

password   required     pam_cracklib.so difok=2 minlen=8 dcredit=2 ocredit=2 try_first_pass retry=3
password   sufficient   pam_unix.so try_first_pass use_authtok nullok md5 shadow
password   required     pam_deny.so

session    required     pam_limits.so
session    required     pam_unix.so
# Add this line to the end
session    optional     pam_ssh.so
</pre>

## Summary
That's all we need to do, you should be able to logout and log back in with the ability to ssh to your remote hosts without a password. You may need to restart your login manager with `systemctl restart lightdm` or simply reboot.

***WARNING: This WILL remove some aspect of physical security on whatever machine you set this up on.  If physical security is a concern, please use a locking screensaver or logout whenever you leave your system unattended.  This is something you should do anyway to protect your local machine from malicious passersby***

## Troubleshooting
If you are having problems creating or distributing your keys, take a look at 
http://gentoo-wiki.com/SECURITY_SSH_without_a_password for more information on that task.

If you are still prompted for a password after completing this guide, there are a few files that you may need to check.  Make sure both of these entries are set to "yes" on your remote hosts. 
`/etc/ssh/sshd_config`
<pre class="clear">
# Allow Identity Auth for SSH1?
 RSAAuthentication yes
 
 # Allow Identity Auth for SSH2?
 PubkeyAuthentication yes
</pre>

Make sure these entries are in your local machine's config.
`/etc/ssh/sshd_config`
<pre class="clear">Host * 
Port 22
IdentityFile ~/.ssh/id_dsa
</pre>
