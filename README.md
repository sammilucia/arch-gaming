# Guide to gaming on Arch Linux 
How to run games on Arch (and configure Steam and Proton).

## Summary
It came to my attention Arch doesn't have a lot of stuff set up for gaming (this isn't a surprise, Arch doesn't have a lot of set up for...anything!) So Writing it here.

I was also getting terrible performance on newer games like Cyberpunk 2077 or Resident Evil (much worse than Windows) and couldn't find any straightforward information on configuring this. So here we are!

## Contributing
Please feel free to contribute! I most certainly am not the expert on this, just trying to work out how to play games! 😊

## Prerequisites
This guide assumes you have the latest NVIDIA drivers installed and working correctly. Version 495.46 at the time of writing.

## Guide
1. Install the latest [Steam](https://archlinux.org/packages/multilib/x86_64/steam/), login and download your games
2. If you want Steam to start with certain command line options, create a file `~/.config/steam-runtime-flags.conf` and put the options in there. There are a few ways to list them, however I find one per line easy, e.g: ![Screenshot from 2021-12-27 22-48-14](https://user-images.githubusercontent.com/3295286/147529440-e8ae6eee-08f8-4a79-9ba9-b243125bcb0e.png)
3. In Steam > Settings > Steam Play > **Enable Steam Play for all other titles** > **Proton Experimental** ![Screenshot from 2021-12-27 22-48-14](https://user-images.githubusercontent.com/3295286/147529249-29eb8927-abf3-4fba-b092-6fa8a0f66c39.png)
4. Force the new game to run with Proton Experimental: Right click the game > Properties > Compatibility > **Force the use of a specific Steam Play compatibility tool** > **Proton Experimental** ![Screenshot from 2021-12-27 22-53-12](https://user-images.githubusercontent.com/3295286/147529496-fc4e14c3-1a7b-4ef8-9211-6cbf09e14236.png)
5. Change the Launch Options for the game to enable the newest Proton flags: Game > Properties > General > **Launch Options** > `vblank_mode=0 VKD3D_CONFIG=force_static_cbv PROTON_ENABLE_NVAPI=1 PROTON_HIDE_NVIDIA_GPU=0 %command%`![Screenshot from 2021-12-27 22-55-08](https://user-images.githubusercontent.com/3295286/147529599-6fbb0362-4527-4009-b6f7-13161e4df57c.png)
6. That should be it! Try the game and see how it goes.

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
- [Gaming - ArchWiki](https://wiki.archlinux.org/title/Gaming#Improving_performance) - of special interest, the section on `schedtool` and `schedtoold`
- [Xanmod kernel](https://github.com/arglebargle-arch/xanmod-rog-PKGBUILD) - including the Asus ROG patches from (Asus Linux)[https://asus-linux.org/]
