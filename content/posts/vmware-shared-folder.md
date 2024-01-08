---
title: Mounting a Shared Folder in Kali and VMware Workstation/Player 2019
date: 2019-09-14
draft: false
---
I hate that I have to write this, but a lot of the solutions out there are really old and misleading. I’ve spent hours before just trying to get this to work. This method works with Kali 2019.3, and the day of this writing is 9/13/2019.

First install the relevant tools:

`sudo apt install -y open_vm_tools open_vm_tools_desktop`

Then make sure you share your folder through VM -> Settings -> Options tab and then set it to “Always Enabled”

Then run the following command:

`sudo vmhgfs-fuse .host:/ /mnt/ -o allow_other -o uid=1000`

Then run:

`ls -la /mnt/`

And you should see your shared folders popping up!

**Alternatively**, I just found out while writing this that there is a mount-shared-folders.sh bash script on the desktop of the VMware distribution. That probably works too.

Credit to [con-f-use](https://web.archive.org/web/20220520135811/https://askubuntu.com/questions/29284/how-do-i-mount-shared-folders-in-ubuntu-using-vmware-tools) in one of the worst aging askubuntu questions ever.