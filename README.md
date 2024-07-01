# Jellyfin-unRaid-Intel_Transcoding
How to RIGHT enable HW transcoding on Jellyfin with Intel QSV on unRaid with a 7th gen up Intel CPU

This just a quick guide, mostly for myself, and to share to other that have my same problem.

The Scenario:

You have a Jellyfin Docker installed on your unRaid NAS/Server, HW transcoding doean't work at all.
You have a Intel CPU with iGPU from 7th gen up. 

Summary:

1 - Install the plugin "Intel-GPU-TOP" from the App manager of unRaid.
2 - Install the plugin "GPU Statistics" from the App manager of unRaid.
3 - Install the "linuxserver/Jellyfin" repository on the App manager of unRaid, and add some variables.
4 - On Jellyfin use the right setup under the "Transcoding" TAB.
5 - Done


1 - Intel-GPU-TOP

Nothing fancy to do, taking for granted you alredy have installed the "Community Applications" plugin on your unRaid istance, then go to "APP" and search for "Intel" on the search bar and install the "Intel-GPU-TOP" plugin from the "ich777" repository. (searching directly for Intel-GPU-TOP give me nothing)
Installing it, would done everything automatically.

Just to be totally sure about permission, i would follow those simple CLI command below: 

lspci -vnnn | perl -lne 'print if /^\d+\:.+(\[\S+\:\S+\])/' | grep VGA
That's to see if you have any GPU available and "on" on your system, the one actually selected as primary GPU, in case you have more than one, it is the one that end with "[VGA controller]"

ls -la /dev/dri/
With that you can see the 3D Core of your GPU, from my understanding "renderD128" is a standard name and what matter for us. 

chown :video /dev/dri/
To make sure Jellyfin have the right permission to use this device. (On some system the "video" Group can be named "render", in case it give you a "invalid group" error)

![aaaattura](https://github.com/IlTossico/Jellyfin-unRaid-Intel_Transcoding/assets/77573228/f2f5a4da-2511-4a8e-b8d0-1fdbb68b29be)


