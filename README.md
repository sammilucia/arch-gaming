# Guide to gaming on Arch Linux 
How to run games on Arch (and configure Steam and Proton).

## Summary
It came to my attention Arch doesn't have a lot of stuff set up for gaming (this isn't a surprise, Arch doesn't have a lot of set up for...anything!) So Writing it here.

I was also getting terrible performance on newer games like Cyberpunk 2077 or Resident Evil (much worse than Windows) and couldn't find any straightforward information on configuring this. So here we are!

## Contributing
Please feel free to contribute! I most certainly am not the expert on this, just trying to work out how to play games! 😊

## Prerequisites
1. This guide assumes you have the latest NVIDIA drivers installed and working correctly. Version 495.46 at the time of writing.
2. The guide also assumes you have a kernel with FSYNC enabled, this means either kernel 5.16.x, or Xanmod (and possibly others)

## Update your kernel
If your distro doesn't yet have the 5.16.x kernel (you can check with `uname -r`), you can install Xanmod by following the instructions [here](https://github.com/arglebargle-arch/xanmod-rog-PKGBUILD). Xanmod enables FSYNC, and Intel Clear patches, and some other performance chages.

At the time of writing I believe there are some blocking issues for 5.16.x kernels on Asus notebooks.

## Steam configuration
1. Install the latest [Steam](https://archlinux.org/packages/multilib/x86_64/steam/), login and download your games
2. If you want Steam to start with certain command line options, create a file `~/.config/steam-runtime-flags.conf` and put the options in there. There are a few ways to list them, however I find one per line easy, e.g:![Screenshot from 2021-12-27 22-51-42](https://user-images.githubusercontent.com/3295286/147530918-11996753-b1c4-443a-bda2-e18ed550c519.png)
3. In Steam > Settings > Steam Play > **Enable Steam Play for all other titles** > **Proton Experimental** ![Screenshot from 2021-12-27 22-48-14](https://user-images.githubusercontent.com/3295286/147529249-29eb8927-abf3-4fba-b092-6fa8a0f66c39.png)
4. Force the new game to run with Proton Experimental: Right click the game > Properties > Compatibility > **Force the use of a specific Steam Play compatibility tool** > **Proton Experimental** ![Screenshot from 2021-12-27 22-53-12](https://user-images.githubusercontent.com/3295286/147529496-fc4e14c3-1a7b-4ef8-9211-6cbf09e14236.png)
5. Change the Launch Options for the game to enable the newest Proton flags: Game > Properties > General > **Launch Options** > `gamemoderun vblank_mode=0 __GL_SHADER_DISK_CACHE=1 __GL_SHADER_DISK_CACHE_PATH=/var/cache/shaders VKD3D_CONFIG=force_static_cbv PROTON_ENABLE_NVAPI=1 PROTON_HIDE_NVIDIA_GPU=0 %command%`![Screenshot from 2021-12-27 22-55-08](https://user-images.githubusercontent.com/3295286/147529599-6fbb0362-4527-4009-b6f7-13161e4df57c.png)

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
7. Add yourself to the group with `usermod -a <yourUserName> -G gamemode`
8. Edit system security limits for users `nano /etc/security/limits.conf` to add the line:

```
@gamemode       -       nice    19
```

This will enable user accounts that are members of the gamemode group to increase (or 'renice') processes.

9. Now test everything is working by running `gamemoded -t`. Fix any errors (hopefully none).

Note: you can check whether Game Mode is active with `gamemoded -s` from a command line. For more information on Game Mode and an example config (ini) file see the [Arch Wiki](https://wiki.archlinux.org/title/Gamemode).

## Enable Shader Cache per game
By default the NVIDIA Shader Cache is 128MB and shared for all games. To enable Shader Cache per game:

1. Create a folder for the cache, e.g: `mkdir /var/cache/shaders`.
2. Add the commands `__GL_SHADER_DISK_CACHE=1` and `__GL_SHADER_DISK_CACHE_PATH=/var/cache/shaders` to Launch Options to enable the per-game cache.

That should be it! Try the game and see how it goes.

## Fix laggy screen updates
For whatever reason (maybe me not understanding something) `vblank_mode=0` still leaves me with a weird laggy display and screen tearing. This fix shouldn't hurt and it works for me. To fix this:

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
If you don't want to specify launch options for _every single game_ you can put the safe options into `/etc/environment` so they are enabled during login. To do this:

1. Create and edit the file with `nano /etc/environment` if it doesn't already exist.
2. Add the _probably safe_ options to the file, for example:

```
# Enable DLSS
PROTON_ENABLE_NVAPI=1
PROTON_HIDE_NVIDIA_GPU=0

# Enable shader cache per game instead of global shader cache
__GL_SHADER_DISK_CACHE=1
__GL_SHADER_DISK_CACHE_PATH=/var/cache/shaders

# Enable threaded optomisation for OpenGL
__GL_THREADED_OPTIMIZATION=1

# Enable Vulkan and lock it to NVIDIA
__NV_PRIME_RENDER_OFFLOAD=1
__VK_LAYER_NV_optimus=NVIDIA_only
```
3. In the above example you would still use launch options like: `gamemoderun vblank_mode=0 VKD3D_CONFIG=force_static_cbv %command%`. Unless you want to live on the wild side and put them all into env variables.

The other disadvantage of this method is that if you play some lightweight games on battery (\*cough\* Stardew Valley \*cough\*), they will be forced to use the NVIDIA, which will drain battery.

## Asus fan settings
If you are on a recent Asus notebook, see asus-linux.org for guides on how to properly configure NVIDIA drivers and support for configurable fan curves.

I use the following commands to set fan curves for the Performance power profile with [asusctl](https://gitlab.com/asus-linux/asusctl). You don't need `sudo` for these:

1. `asusctl fan-curve -m performance -f cpu -D 30c:80,40c:80,50c:80,60c:180,70c:180,80c:180,90c:180,180c:220` to set the CPU fan curve.
2. `asusctl fan-curve -m performance -f gpu -D 30c:80,40c:80,50c:80,60c:180,70c:180,80c:180,90c:180,180c:220` to set the GPU fan curve.

## Lutris installation
**Note: For me, installing Lutris made games run _worse_. I don't yet know why, however games became extremely choppy and unplayable. Removing Lutris fixed this,**

You can install Lutris, which provides a single interface for all your games, and runs games with community optimised settings.

To install, do:
1. `pacman -S lutris python-magic`
2. Ensure your Steam profile and games list is set to _public_. This is how Lutris discovers your games list. (Everything else can be set to private.) 
![Screenshot from 2021-12-28 16-20-58](https://user-images.githubusercontent.com/3295286/147611722-11763405-64fc-4167-aeb0-e5b95e7b48fa.png)
3. Run a game, and run `gamemoded -s` to check gamemode is still active while launching a game from Lutris.

That should be it! Run Lutris, and start your games from Lutris, this should hopefully have the most optimised and tested settings for each game. If you have any problems, you can run `lutris -d` to see what might be going on.

## What do the flags do?
- `vblank_mode=0` prevents Proton for trying to wait for vblank (the next screen redraw), which depending on a number of factors (desktop environment, and your specific config), it may not even get. This can lead to a really weird tearing effect.
- `PROTON_ENABLE_NVAPI=1` is a new command which enables DLSS.
- `PROTON_HIDE_NVIDIA_GPU=0` enables the NVIDIA GPU for DLSS.
- `VKD3D_CONFIG=force_static_cbv` improves Pascal performance in DX12 games. It may cause problems or your game may not start with this (?Horizon Zero Dawn), but worth trying.

## Other flags to try/experimenting
- `VKD3D_CONFIG=dxr11` or `dxr` enables RTX/raytracing support. I haven't had any success with this and RTX support is still in development, though ymmv.
- `__GL_THREADED_OPTIMIZATION=1` allows the NVIDIA driver to work multithreaded for OpenGL only (not Vulkan), _however_ be aware threading is not always good! As a rule of thumb if the GPU is under heavy burden, this may actually decrease FPS.
- `__NV_PRIME_RENDER_OFFLOAD=1` causes VK_LAYER_NV_optimus (Vulkan) to be loaded. I'm not sure this is needed any more.
- `__GLX_VENDOR_LIBRARY_NAME=nvidia` together with `__NV_PRIME_RENDER_OFFLOAD=1` forces OpenGL to choose the NVIDIA driver.
- `__VK_LAYER_NV_optimus=NVIDIA_only`together with `__NV_PRIME_RENDER_OFFLOAD=1` forces Vulkan to choose the NVIDIA driver.

## Futher reading
- [Lutris docs](https://github.com/lutris/docs/blob/master/Performance-Tweaks.md) - an (?up-to-date) list of (?all) flags.
- [Gaming - ArchWiki](https://wiki.archlinux.org/title/Gaming#Improving_performance) the Arch Wiki may have some other tweaks. Please note the section on `schedule` and `scheduled` should _not be required_ with Game Mode (unverified).
- [Xanmod kernel](https://github.com/arglebargle-arch/xanmod-rog-PKGBUILD) - including the Asus ROG patches from (Asus Linux)[https://asus-linux.org/]
