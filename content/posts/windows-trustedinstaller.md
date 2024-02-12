---
title: Overriding Windows TrustedInstaller Permissions
date: 2019-07-29
draft: false
description: How to bypass or override Windows TrustedInstaller to write files to System32
---

tl;dr  
For a single file, all you have to do is:

`takeown /F C:\FULL_PATH_TO_FILE`  
`icacls C:\FULL_PATH_TO_FILE /grant Administrators:F`

For an entire directory, it’s a bit longer:

`takeown /F C:\FULL_PATH_TO_FOLDER`  
`takeown /F C:\FULL_PATH_TO_FOLDER /r /d y`  
`icacls C:\FULL_PATH_TO_FOLDER /grant Administrators:F`  
`icacls C:\FULL_PATH_TO_FOLDER /grant Administrators:F /t`

Basically, this can be used to modify and overwrite Windows’s system files and other things you’re not supposed to touch. This can also be done from the GUI by assigning yourself to be the owner of the file, applying the change, closing the window, opening the permissions again, elevating to admin, and finally setting the permissions you would like.

[Credit to Micah](https://web.archive.org/web/20220520135811/https://merabheja.com/solved-failed-to-enumerate-objects-in-the-container-windows-10-error/) in the comments.