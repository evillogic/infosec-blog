---
title: Proxmox Notes
date: 2024-01-06
draft: false
---

Collection of proxmox notes

### MacOS virt-viewer (SPICE)

Unfortunately it doesn't look like virt-viewer is supported by default for macos, so you have to use a 3P homebrew formula to get this working. The following gist was helpful in setup:

https://gist.github.com/tomdaley92/789688fc68e77477d468f7b9e59af51c

Note that you no-longer need to install the tap as the formula is in homebrew proper now.

I had to chown a folder of icons used by this formula, which is likely a shared dependency with some other program I installed as root. I also had some issues getting the default application association to stick. I'd love to find a way to do this that's not in the "get info" pane, such as how file type associations in windows are stored in the registry and accessible in control panel.
