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

Before creating a new account, I checked whether my existing OMV user (user x) was also a Linux user.

Initially:

id user x

returned that the user did not exist. I discovered that Linux usernames are case-sensitive and my existing account was actually named User x.

I confirmed the account with:

getent passwd | grep -i user x

The account existed as a Linux user, but further checks showed that it:

Only belonged to the standard users group.
Did not have sudo administrative privileges.
Was not authorized through OMV's _ssh group.
Was configured with /home/User x as its home directory, but that directory did not actually exist.
Was using /usr/bin/sh rather than Bash as its shell.

Instead of creating another user, I configured my existing User x account for Linux administration.

Created the missing home directory and assigned ownership and permissions:

mkdir -p /home/User x
chown User x:users /home/User x
chmod 750 /home/User x

Changed the user's interactive shell to Bash:

usermod -s /bin/bash User x

Added User x to the sudo group for administrative privileges and the _ssh group for SSH access:

usermod -aG _ssh,sudo User x

Verified the account configuration:

id User x
getent passwd User x

I then opened a new Command Prompt and tested SSH using the User x account:

ssh User x@192.168.xxx.xxx

After successfully logging in, I verified which user I was logged in as:

whoami

Output:

User x

Tested whether the account could elevate to administrator/root privileges:

sudo whoami

Output:

root

Finally, I verified that the account was using its newly created home directory:

pwd

Output:

/home/User x

At this point, I confirmed that User x can be used as my normal SSH administration account while still being able to temporarily elevate to root privileges with sudo when administrative access is required.

The actual root account has not been removed or disabled. The goal is to stop using direct root login for routine remote administration and move toward SSH key authentication.
