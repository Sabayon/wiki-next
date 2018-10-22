## Add a Users

To create a user that is a member of the wheel, users, and audio groups.  As root:

    useradd -m -G users,wheel,audio -s /bin/bash UserName

This will also create a shell and home directory for the UserName.  After that, assign a password to the user account. As root:

    passwd UserName

So to add the user wolfden I would run

    useradd -m -G users,wheel,audio -s /bin/bash wolfden
    passwd wolfden

Type the password, as you type, it will not show any characters, retype password when it asks.

To see what groups you belong to:

    groups

or

    groups UserName

## Delete a User

To remove a user along with home directory and mail spool, as root:

    userdel -r UserName

## Add an existing user to a group

As root, run the command:

    gpasswd -a UserName wheel

You can add a user to multiple groups at same time by running

    usermod -aG wheel,audio UserName

Warning on the usermod, do not forget the -a or you will remove yourself from all existing groups and only belong to wheel and audio

## Remove user from group

As root, run the command:

    gpasswd -d UserName audio

## Add a user to sudo

As root, run the command

    visudo

Than you need to uncomment a line 

    ## Uncomment to allow members of group wheel to execute any command
    # %wheel ALL=(ALL) ALL

Change it so it looks like 

    ## Uncomment to allow members of group wheel to execute any command
    %wheel ALL=(ALL) ALL

Save the file

