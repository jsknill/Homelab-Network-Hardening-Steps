# Homelab-Network-Hardening-Steps
Network Hardening Steps

Updated password for root login to OMV to make it more secure before changing over from SSH password authentication to SSH keys.

I used:

ssh root@192.168.xxx.xxx

Authenticated with the existing root password, then ran:

passwd

and updated the root password to a new, stronger password.

Opened a new Command Prompt without closing the existing root session and ran:

ssh root@192.168.xxx.xxx

Confirmed the new password worked before closing the original session.

Created a Normal Linux Admin Account for SSH

Before creating a new account, I checked whether my existing OMV user Jordan was also a Linux user.

Initially:

id jordan

returned that the user did not exist. I discovered that Linux usernames are case-sensitive and my existing account was actually named Jordan.

I confirmed the account with:

getent passwd | grep -i jordan

The account existed as a Linux user, but further checks showed that it:

Only belonged to the standard users group.
Did not have sudo administrative privileges.
Was not authorized through OMV's _ssh group.
Was configured with /home/Jordan as its home directory, but that directory did not actually exist.
Was using /usr/bin/sh rather than Bash as its shell.

Instead of creating another user, I configured my existing Jordan account for Linux administration.

Created the missing home directory and assigned ownership and permissions:

mkdir -p /home/Jordan
chown Jordan:users /home/Jordan
chmod 750 /home/Jordan

Changed the user's interactive shell to Bash:

usermod -s /bin/bash Jordan

Added Jordan to the sudo group for administrative privileges and the _ssh group for SSH access:

usermod -aG _ssh,sudo Jordan

Verified the account configuration:

id Jordan
getent passwd Jordan

I then opened a new Command Prompt and tested SSH using the Jordan account:

ssh Jordan@192.168.xxx.xxx

After successfully logging in, I verified which user I was logged in as:

whoami

Output:

Jordan

Tested whether the account could elevate to administrator/root privileges:

sudo whoami

Output:

root

Finally, I verified that the account was using its newly created home directory:

pwd

Output:

/home/Jordan

At this point, I confirmed that Jordan can be used as my normal SSH administration account while still being able to temporarily elevate to root privileges with sudo when administrative access is required.

The actual root account has not been removed or disabled. The goal is to stop using direct root login for routine remote administration and move toward SSH key authentication.
