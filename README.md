# How to RIGHT enable HW transcoding on Jellyfin with Intel QSV on unRaid with a 7/8/9th gen up Intel Desktop CPU

This just a quick guide, mostly for myself, and to share to other that have my same problem.

### The Scenario:

You have a Jellyfin Docker installed on your unRaid NAS/Server, HW transcoding doean't work at all.
You have a Intel CPU with iGPU from 7/8/9th gen up. 

### Summary:

1. Install the plugin "Intel-GPU-TOP" from the App manager of unRaid.
2. Install the plugin "GPU Statistics" from the App manager of unRaid.
3. Install the "linuxserver/Jellyfin" repository on the App manager of unRaid, and add some variables.
4. On Jellyfin use the right setup under the "Transcoding" TAB.
5. Done

##
**1 - Intel-GPU-TOP**

Nothing fancy to do, taking for granted you alredy have installed the `Community Applications` plugin on your unRaid istance, then go to "APP" and search for `Intel` on the search bar and install the `Intel-GPU-TOP` plugin from the "ich777" repository. (searching directly for Intel-GPU-TOP give me nothing)
Installing it, would done everything automatically.
<br>
<br>

Just to be totally sure about permission, i would follow those simple CLI command below: 
<br>
<br>


```
lspci -vnnn | perl -lne 'print if /^\d+\:.+(\[\S+\:\S+\])/' | grep VGA
```
That's to see if you have any GPU available and "on" on your system, the one actually selected as primary GPU, in case you have more than one, it is the one that end with `[VGA controller]`.
<br>
<br>

```
ls -la /dev/dri/
```
With that you can see the 3D Core of your GPU, from my understanding `renderD128` is a standard name and what matter for us. 
<br>
<br>

```
chown :video /dev/dri/
```
To make sure Jellyfin have the right permission to use this device. (On some system the `video` Group can be named `render`, in case it give you a "invalid group" error)
<br>
<br>

![aaaattura](https://github.com/IlTossico/Jellyfin-unRaid-Intel_Transcoding/assets/77573228/f2f5a4da-2511-4a8e-b8d0-1fdbb68b29be)

##
**2 - GPU Statistics**

Another plugin to install, from the "APP" center, search for `GPU Statistics` and install it. Its only function is to add a Tab on the dashboard, to help us see if our GPU is working as intended. You can manage the plugin on the "Setting" tab and then from "GPU Statistics", just make sure you have selected the right GPU to show. 
<br>
<br>

Alternatively we can see the iGPU usage from CLI with the command: 
```
intel_gpu_top
```
##
**3 - Jellyfin**

That's the big step.
I was firstly using the official repository, and after tons of troubleshooting i start using the linuxserver's repository, so it's plausible that this solution work on the official repository too. 
On top of that, with this, i surely resolved my transcoding issue, or mostly, i enable the ability to have the igpu and jellyfin talk to each other, and i'm sure of that because some CLI command give me good resoult and because i can transcode video from my phone. But i still have problem, like with two TVs, both my 55" 1080p OLED and my 28" 720p IPS have problems playing some films or series, some media transcode fine, some give me an error and on the 55" most of the time the screen remain totally black, even so i can see the server transcoding fine in real time, or at least i see activity on the iGPU. I think the clients for WebOS, in my situation, have many problems.
<br>
<br>

So, from the "APP" center, search for `Jellyfin` and download the repository from `linuxserver`.
On the installation od the docker, we want to add two new Variable:

<br>

Click on `+ Add another Path, Port, Variable, Label or Device` and select it as `Device`:
```
Config Type:    Device
Name:           --device=/dev/dri
Value:          /dev/dri
Description:    Jellyfin see the iGPU
```
![aaaattura](https://github.com/IlTossico/Jellyfin-unRaid-Intel_Transcoding/assets/77573228/70389a3a-310d-4a27-900c-b48074ade6ae)

<br>
<br>

Click on `+ Add another Path, Port, Variable, Label or Device` and select it as `Variable`: 
```
Config Type:    Variable
Name:           OpenCLMod
Key:            DOCKER_MODS
Value:          linuxserver/mods:jellyfin-opencl-intel
Default Value:  
Description:    OpenCL Docker Mod from linuxserver for Transcoding HDR content. For transcoding HDR issue.
```
![b](https://github.com/IlTossico/Jellyfin-unRaid-Intel_Transcoding/assets/77573228/176fb8db-4275-466b-bea7-300216ed2e38)

This one is optional, for who have issue transcoding HDR content, but considering i've tried my istance of Jellyfin after adding this one too, i suggest to add it anyway. 
<br>
<br>

There is another option, for who need it, i have't tried it, so i don't know if it works fine.
The ability to use the RAM as transcoding folder, click on `+ Add another Path, Port, Variable, Label or Device` and select it as `Path`:
```
Config Type:      Path
Name:             Transcode to RAM
Container Path:   /config/data/transcodes
Host Path:        /dev/shm
Default Value:
Access Mode:      Read/Write
Description:
```
<br>
<br>

We are done with the installation of the docker. Next step.

##
**4 - Jellyfin Setup**

We can procede with the normal setup of Jellyfin, if you have problem with this step, there are plenty of tutorial online.
So, you have Jellyfin ready and working, you have setup your media Libray and what you need to have it working as you like, fine.
<br>
<br>

Go to `Control Panel > Playback > Transcoding` and select everything as the image below say:

![879071937_worksflawlessly PNG b22bcc01447fec613e098f667736e4da](https://github.com/IlTossico/Jellyfin-unRaid-Intel_Transcoding/assets/77573228/68dcf296-36d8-4a9f-9533-0ffffc17ee82)

- Hardware acceleration: `Intel QuickSync (QSV)`
- Enable hardware decoding: `with 7/8/9th you can transcode everything exept AV1 that is supported only on newer model of UHD GPU.` 

- ### **VERY IMPORTANT**
  **BOTH Intel Low-Power modes NEEDS TO BE DISABLE, OR THEY WOULD BROKE THE TRANSCODING.**

- I prefer disabling the transcoding in HEVC format, considering most devices prefer H264 and on streaming there is no much difference for bandwith, but it's a preference.

- Enable both `VPP Tone mapping` and `Tone mapping`. It works for me.
<br>
<br>

  **SAVE!!!**
<br>
<br>

  This setup would work fine for 7/8/9th Gen Intel Dektop CPU.

  Ok, we have done with the important setup.

##  
**5 - Testing**

Theorically our setup should work fine.
Before testing it directly, i would done some CLI testing, you can use those CLI line to test your actual setup, or in general to see if your Jellyfin istance is talking to your GPU.
<br>
<br>

```
docker exec -it jellyfin /usr/lib/jellyfin-ffmpeg/vainfo
```
![c](https://github.com/IlTossico/Jellyfin-unRaid-Intel_Transcoding/assets/77573228/6f4eae7f-ca29-4a0c-bf28-aea5865d9c3a)

Pay attention, the name of the container could be different from mine. This line make shure we have hardware acceleration working via VA-API.
<br>
<br>

```
docker exec -it jellyfin /usr/lib/jellyfin-ffmpeg/ffmpeg -v verbose -init_hw_device vaapi=va -init_hw_device opencl@va
```
![d](https://github.com/IlTossico/Jellyfin-unRaid-Intel_Transcoding/assets/77573228/ba29cb3b-7079-4378-a27d-58cba2f93519)

Pay attention, the name of the container could be different from mine. This line is good to see both VA-API, QSV and QSV with OpenCL.
<br>
<br>

If you get the same resoult as mine, you are done! Just do a real try, try with your smarphone or TV to play a bigger file than you can and watch first on your unRaid Dashboard for the GPU Stats; and then on your Jellyfin Control panel, click on the `i` icon to see what Jellyfin say. You should see something like this: (pardon the Italian on the Jellyfin pic)

![ggg](https://github.com/IlTossico/Jellyfin-unRaid-Intel_Transcoding/assets/77573228/77af7485-e1bf-4e43-9c0a-d46ad8b81923)
![ccc](https://github.com/IlTossico/Jellyfin-unRaid-Intel_Transcoding/assets/77573228/ca38323e-78ba-4735-8df3-2410b93d2352)

##
<br>
<br>

**Useful Link**

- https://github.com/linuxserver/docker-mods/tree/jellyfin-opencl-intel
- https://forum.jellyfin.org/t-solved-hardware-acceleration-on-unraid-issues
- https://forums.unraid.net/topic/144179-intel-n5105jasperlake-hw-transcode-with-binhex-jellyfin/
- https://forums.unraid.net/topic/145424-jellyfin-8700k-transcoding/
- https://www.reddit.com/r/unRAID/comments/166hpy7/please_help_me_getting_hw_transcoding_to_work/

