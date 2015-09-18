# Cheat Sheet - Linux-Unix - User Account Setup

By Jack Szwergold, September 16, 2015

Here are instructions on how to setup an account on a Linux server.

#### Create a user account.

Login to the server you want to create an account on and add the user—in this case `username`—like this:

    sudo adduser username

Use the password as follows for initial setup or change it to something else:

    new2015pass16

This is a temporary account so the user will have to change it on their first login. More on that later when we expire the user’s password to force creation of a new password.

#### Add a user to a group and set the user’s default group.

Add the user to the `www-readwrite` group:

    sudo adduser username www-readwrite

Set user’s default group to the `www-readwrite` group:

    sudo usermod -g www-readwrite username

#### Expire the password immediately to force the user to choose their own password.

First, let’s check the password status like this:

    sudo passwd -S username

Next, expire the password immediately:

    sudo chage -d 0 username

After that’s done, check the password status again to make sure it has been expired:

    sudo passwd -S username

#### Miscellaneous user account management items.

Here are some related commands that might need to be used depending on the situation.

***

Run this command to unexpire a user’s password:

    sudo chage -d 99999 username

Check the account aging information for a user with this command:

    sudo chage -l username

Lock a user’s password:

    sudo passwd -l username

Delete user from system and remove their home directory:

    sudo deluser --remove-home username

***

*Cheat Sheet - Linux-Unix - User Account Setup (c) by Jack Szwergold*

*This work is licensed under a Creative Commons Attribution-NonCommercial-ShareAlike 4.0 International License (CC-BY-NC-SA-4.0).*