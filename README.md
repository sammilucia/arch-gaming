# Guide to gaming on Arch Linux 
How to run games on Arch (and configure Steam and Proton). Thanks to Leopard on the asus-linux Discord for your help, and to the creators of asus-linux.org for all their tireless work getting everything working on Asus notebooks 😊.

## Summary
It came to my attention it's really hard to find definitive information on how to cinfigure gaming on Linux. This guide is written for Arch (because I'm on Arch), however the concepts can be adapted to any _up-to-date_ Linux distro.*

I was also getting terrible performance on newer games like Cyberpunk 2077 or Resident Evil (much worse than Windows) and couldn't find any straightforward information on configuring this. So here we are!

_*Note: At the the time of writing, Fedora, and Arch are probably the only really up-to-date distros, though you may have success with some gaming-specific distros, I don't use them, so cannot confirm this._

## Contributing
Please feel free to contribute! I most certainly am not the expert on this topic, just trying to work out how to play games! 😊

## Prerequisites
1. This guide assumes you have the latest NVIDIA drivers installed and working correctly. Version 495.46 at the time of writing.
2. The guide also assumes you have a kernel with FSYNC enabled, this means either kernel 5.16.x, or Xanmod (and possibly some other modded kernels).

## Update your kernel
Xanmod enables FSYNC, and Intel Clear patches, and some other performance chages.

1. If your distro doesn't yet have the 5.16.x kernel (you can check with `uname -r`), you can install Xanmod by following the instructions [here](https://github.com/arglebargle-arch/xanmod-rog-PKGBUILD) for Arch, or it should be fairly easy to find a recent Xanmod built for your distro.
2. To enable vsync passthrough (depending on how well a game implements its vsync), add `nvidia-drm.modeset=1` to the `GRUB_CMDLINE_LINUX_DEFAULT` line in `/etc/default/grub`
3. You can also reduce mouse latency by adding `usbhid.mousepoll=1` to `GRUB_CMDLINE_LINUX_DEFAULT`. Note this is probably broken since kernel 4.4.x, however it shouldn't hurt to add it.
4. Regenerate grub by running `grub-mkconfig -o /boot/grub/grub.cfg`.
5. Reboot.

_Note: At the time of writing I believe there are some blocking issues from using 5.16.x kernels on Asus notebooks._

## Steam configuration
1. Install the latest [Steam](https://archlinux.org/packages/multilib/x86_64/steam/), login and download your games
2. If you want Steam to start with certain command line options, create a file `~/.config/steam-runtime-flags.conf` and put the options in there. (Note: if this doesn't work for you, you may need to modify the `Exec` in the `steam.desktop` shortcut. There are a few ways to list flags... I find one per line clearest, e.g:![Screenshot from 2021-12-27 22-51-42](https://user-images.githubusercontent.com/3295286/147530918-11996753-b1c4-443a-bda2-e18ed550c519.png)
4. In Steam > Settings > Steam Play > **Enable Steam Play for all other titles** > **Proton Experimental** ![Screenshot from 2021-12-27 22-48-14](https://user-images.githubusercontent.com/3295286/147529249-29eb8927-abf3-4fba-b092-6fa8a0f66c39.png)
5. Force the new game to run with Proton Experimental: Right click the game > Properties > Compatibility > **Force the use of a specific Steam Play compatibility tool** > **Proton Experimental** ![Screenshot from 2021-12-27 22-53-12](https://user-images.githubusercontent.com/3295286/147529496-fc4e14c3-1a7b-4ef8-9211-6cbf09e14236.png)
6. Change the Launch Options for the game to enable gamemode (see below), and force NVIDIA: Right-click game > Properties > General > **Launch Options** > `gamemoderun __NV_PRIME_RENDER_OFFLOAD=1 __GLX_VENDOR_LIBRARY_NAME=nvidia __VK_LAYER_NV_optimus=NVIDIA_only VK_ICD_FILENAMES=/usr/share/vulkan/icd.d/nvidia_icd.json %command%`![Screenshot from 2021-12-29 19-06-45](https://user-images.githubusercontent.com/3295286/147714429-727c07dc-8ade-4dd1-8a08-3a6791a4ddaa.png)

## Enable Game Mode scheduling
Game Mode configures the CPU governer, scheduler, and process niceness for maximum performance for games. To enable Game Mode:

1. First enable the _multilib_ repo if you haven't already. Edit `/etc/pacman.conf` to add

```
[multilib]
Include = /etc/pacman.d/mirrorlist
```
2. Run `pacman -Syy` to update your repositories.
3. Install `lib32-gamemode` with `pacman -S lib32-gamemode` [link to library](https://github.com/FeralInteractive/gamemode).
4. Install `gamemode` with `pacman -S gamemode`.
5. Add `gamemoderun` to the beginning of your Steam game Launch Options.
6. Create a new user group with `groupadd -r gamemode`.
7. Add yourself to the group with `usermod -a <yourUserName> -G gamemode`.
8. Edit system security limits for users `nano /etc/security/limits.conf` to add the line:

```
@gamemode       -       nice    10
```

This will enable user accounts that are members of the gamemode group to increase (or 'renice') processes up to a value of 10.

9. Now test everything is working by running `gamemoded -t` and fix any errors (hopefully none).

Note: you can check whether Game Mode is active with `gamemoded -s` from a command line. For more information on Game Mode and an example config (ini) file see the [Arch Wiki](https://wiki.archlinux.org/title/Gamemode).

## Enable saving the Shader Cache
By default the Shader Cache will generate every time a game is launched. To enable saving of the Shader Cache:
1. Right-click game > Properties > **Shader Pre-Caching** > Check **Enable Shader Pre-Caching** AND **Allow background processing of Vulkan shaders**:

![Screenshot from 2021-12-29 19-06-01](https://user-images.githubusercontent.com/3295286/147714558-717f9252-862e-4d2f-b300-8d47085286b4.png)

The Shader Cache should now start generating and saving the cache for that game.

## Fix laggy screen updates
For whatever reason (maybe me not understanding something) `vblank_mode=0` still leaves me with a weird laggy display and screen tearing. This fix shouldn't hurt and it works for me. Do:

1. Create a new config file with `nano /etc/drirc`.
2. Add the following to the file:

```
<driconf>
   <device>
       <application name="Default">
           <option name="vblank_mode" value="0" />
       </application>
   </device>
</driconf>
```
3. Reboot.

## Using environment variables instead
If you don't want to specify launch options for _every single game_ you can put _known safe_ options into `/etc/environment` so they are enabled during login. To do this:

1. Create and edit the file with `nano /etc/environment` if it doesn't already exist.
2. Add your desired options, for example:

```
#
# This file is parsed by pam_env module
#
# Syntax: simple "KEY=VAL" pairs on separate lines
#

# Configure AMD. Only required if you have AMD iGPU/dGPU
AMD_VULKAN_ICD="RADV"
QT_QPA_PLATFORMTHEME="qt5ct"
MOZ_ENABLE_WAYLAND=1

# Enable AMD ACO shader compiler threading
mesa_glthread=true

# Set video decode to AMD. This should be set automatically since 12.0.1 of
# libva-mesa-driver. This is only in here for testing.
#VDPAU_DRIVER="radeonsi"

# Enable rendering. Still in development, will probably crash!
#VKD3D_CONFIG=dxr11
#VKD3D_CONFIG=dxr

# Pascal speed hack for Vulkan for all games. This should make a big difference
# however _may_ cause problems in some games. If it does for you, move it from
# here to steam Launch Options.
VKD3D_CONFIG=force_static_cbv

# Enable DLSS. This is still fairly new.
PROTON_ENABLE_NVAPI=1
PROTON_HIDE_NVIDIA_GPU=0

# Compensate for compositing Window Managers:
# 1 = Display _previous_ rendered frame. Highest FPS, but higher latency
# 2 = Display _latest_ rendered frame, better latency, but _possibly_ lower FPS
PRIMUS_SYNC=2

# Enable shader disk cache per game instead of regenerating the cache at every
# game launch. The disk cache path should no longer be required, however if
# the shader cache is being regenerated each launch, try specifying it.
__GL_SHADER_DISK_CACHE=1
#__GL_SHADER_DISK_CACHE_PATH=/var/cache/shaders

# Enable threaded optomisation for OpenGL.
__GL_THREADED_OPTIMIZATION=1
```

3. In the above example you would still use the Launch Options above to force the use of NVIDIA. Using Launch Options this way enables you to still run some games on the iGPU, so you can play less demanding games on battery.

## Asus fan settings
If you are on a recent Asus notebook, see asus-linux.org for guides on how to properly configure NVIDIA drivers and support for configurable fan curves.

I use the following commands to set fan curves for the Performance power profile with [asusctl](https://gitlab.com/asus-linux/asusctl). You don't need `sudo` for these:

1. `asusctl fan-curve -m performance -f cpu -D 30c:80,40c:80,50c:80,60c:180,70c:180,80c:180,90c:220,100c:220` to set the CPU fan curve.
2. `asusctl fan-curve -m performance -f gpu -D 30c:80,40c:80,50c:80,60c:180,70c:180,80c:180,90c:220,100c:220` to set the GPU fan curve.

## Lutris installation
**Note 1: For me, installing Lutris made games run _worse_. I don't yet know why, however games became extremely choppy and unplayable. Removing Lutris fixed this,**

_Note 2: With all the above config, you actually shoudln't need Lutris.

You can install Lutris, which provides a single interface for all your games, and runs games with community optimised settings.

To install, do:
1. `pacman -S lutris python-magic`
2. Ensure your Steam profile and games list is set to _public_. This is how Lutris discovers your games list. (Everything else can be set to private.) 
![Screenshot from 2021-12-28 16-20-58](https://user-images.githubusercontent.com/3295286/147611722-11763405-64fc-4167-aeb0-e5b95e7b48fa.png)
3. Run a game, and run `gamemoded -s` to check gamemode is still active while launching a game from Lutris.

That should be it! Run Lutris to start and configure games from Lutris. If you have any problems, you can run `lutris -d` to see what might be going on.

## What do the flags do?
- `vblank_mode=0` prevents Proton for trying to wait for vblank (the next screen redraw), which depending on a number of factors (desktop environment, and your specific config), it may not even get. This can lead to a really weird tearing effect. If you don't see this problem, you do not need to enable this flag. This may vary by game, depending on how each game has implemented vsync (i.e: "well", or "poorly").
- `__NV_PRIME_RENDER_OFFLOAD=1` causes VK_LAYER_NV_optimus (Vulkan) to be loaded. I'm not sure this is needed any more.
- `__GLX_VENDOR_LIBRARY_NAME=nvidia` together with `__NV_PRIME_RENDER_OFFLOAD=1` forces OpenGL to choose the NVIDIA driver.
- `__VK_LAYER_NV_optimus=NVIDIA_only`together with `__NV_PRIME_RENDER_OFFLOAD=1` forces Vulkan to choose the NVIDIA driver.

## "Experimental" flags
- `VKD3D_CONFIG=dxr11` or `VKD3D_CONFIG=dxr` enables RTX/raytracing support. I haven't had any success with this, it's caused some games to not start up after enabling RTX, and manually having to disable RTX in the games `.ini`. Raytracing support is still in development, so proceed with caution.
- `__GL_THREADED_OPTIMIZATION=1` allows the NVIDIA driver to work multithreaded for OpenGL only (not Vulkan), _however_ be aware threading is not always good! As a rule of thumb if the GPU is under heavy burden, this may actually decrease FPS.

## Futher reading
- [Lutris docs](https://github.com/lutris/docs/blob/master/Performance-Tweaks.md) - a list of many launch flags.
- [Gaming - ArchWiki](https://wiki.archlinux.org/title/Gaming#Improving_performance) the Arch Wiki may have some other tweaks. Please note the section on `schedule` and `scheduled` should _not be required_ with Game Mode (unverified).
- [Xanmod kernel](https://github.com/arglebargle-arch/xanmod-rog-PKGBUILD) - including the Asus ROG patches from (Asus Linux)[https://asus-linux.org/]

## Changelog

29-Dec-21:
- Added changelog
- Enabled launching of some games on iGPU
- Added warning about Lutris
- Fixed fan curve typos

28-Dec-21:
- Added Lutris

27-Dec-21 #2:
- Added renice
- Added gamemode
- Added renice fix
- Added fan curves
- Added vblank fix

27-Dec-21 #1:
- Initial version
