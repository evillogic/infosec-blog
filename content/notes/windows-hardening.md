---
title: Windows Hardening Stuff
date: 2019-10-15
draft: false
---
This is a random list of notes for Windows Hardening

Against Responder

“Turn off multicast name resolution” Enabled

Don’t need SMB? Turn it off

From Services.msc Disable and Stop “Server” AKA LanmanServer. Restart.

Need SMB but worried about Eternal Blue?

Set-SmbServerConfiguration -EnableSMB1Protocol $false

Fuck it why not just turn it all off for good measure?  
Set-SmbServerConfiguration -EnableSMB2Protocol $false