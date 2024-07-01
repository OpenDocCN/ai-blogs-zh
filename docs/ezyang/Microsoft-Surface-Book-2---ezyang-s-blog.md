<!--yml
category: 未分类
date: 2024-07-01 18:16:53
-->

# Microsoft Surface Book 2 : ezyang’s blog

> 来源：[http://blog.ezyang.com/2019/03/microsoft-surface-book-2/](http://blog.ezyang.com/2019/03/microsoft-surface-book-2/)

## Microsoft Surface Book 2

Long time readers of mine may be aware that I used a ThinkPad X61T for the past decade. After the hinge on my second instance of the machine, I decided it was finally time to get a new laptop. And I had one particular model on my eye, after Simon Peyton Jones showed me his new laptop at the last Haskell Implementor's Workshop: the Microsoft Surface Book 2\. It fits my primary requirement for a laptop: it's a convertible laptop into tablet mode with a digitizer pen. The pen is not Wacom branded but it has an eraser end and can magnetically attach to the laptop (no enclosure for the pen, but I think that for modern hardware that constraint is unsatisfiable.) Furthermore, there is a [Linux enthusiast community](https://github.com/jakeday/linux-surface/) around the device, which made me feel that it would be more likely I could get Linux to work. So a few weeks ago, I took the plunge, and laid down three grand for my own copy. It has worked out well, but in the classic Linux style, not without a little bit of elbow grease.

### A quick review

The good:

1.  I've managed to get all of the "important" functionality to work. That's Xournal with XInput pen and hibernate (though with some caveats.)
2.  Linux support for other random features has pleasantly surprised me: I managed to get a working CUDA install and drivers (for PyTorch development), ability to boot my Linux partition bare metal as well as from a VM in Windows and I can even detach the screen while booted into Linux.
3.  The keyboard is nice; not as good as a classic Thinkpad keyboard but having actual function keys, but it has real function keys (unlike the Macbook Pro I use at work.)
4.  Two standard USB ports as well as a USB-C port means I don't need dongles for most usage (unlike my Macbook Pro, which only has USB-C ports.)

The bad:

1.  (Updated on March 19, 2019) Suspend is really slow. Although jakeday's setup.sh suggests that suspend is not working, *something* is working, in the sense that if I close my laptop lid, the laptop goes into a low power state of some sort. But it takes quite a long time to suspend, an even longer time to restart, and you still have to click past the bootloader (which makes me seriously wonder if we are actually suspending).
2.  The laptop un-hibernates itself sometimes when I put it in my backpack. My current hypothesis is that the power button is getting pushed (unlike most laptops, the power button is unprotected on the top of the screen). Probably some fucking around with my ACPI settings might help but I haven't looked closely into it yet.
3.  It's a high DPI screen. There's nothing wrong with this per se (and you basically can't buy a non-high DPI laptop these days), but any software that doesn't understand how to do high DPI (VMWare and Xournal, I'm looking at you) looks bad. The support of Ubuntu Unity for high DPI has gotten much better since the last time I've attempted anything like it, however; if I stick to the terminal and browser, things look reasonable.
4.  The function key is hardwired to toggle fn-lock. This is somewhat irritating because you have to remember which setting it's on to decide if you should hold it to get the other toggle. I'm also feeling the loss of dedicated page-up/page-down key.
5.  Apparently, the NVIDIA GPU downthrottles itself due to thermal sensor shenanigans (something something the fan is on the motherboard and not the GPU so the driver thinks the fan is broken and throttles? Mumble.)
6.  The speakers are... OK. Not great, just OK.
7.  It's too bad Microsoft opted for some custom charger for the Surface Book 2.

### Linux setup

I did a stock install of the latest Ubuntu LTS (18.04) dual boot with Windows (1TB hard drive helps!), and then installed jakeday's [custom Linux kernel and drivers.](https://github.com/jakeday/linux-surface) Some notes about the process:

*   I spent a while scratching my head as to why I couldn't install Linux dual-boot. Some Googling suggested that the problem was that Windows hadn't really shutdown; it had just hibernated (for quick startup). I didn't manage to disable this, so I just resized the Windows partition from inside Windows and then installed Linux on that partition.

*   Don't forget to allocate a dedicated swap partition for Linux; you won't be able to hibernate without it.

*   The Surface Book 2 has secure boot enabled. You must follow the instructions in [SIGNING.md](https://github.com/jakeday/linux-surface/blob/master/SIGNING.md) to get signed kernels.

*   One consequence of generating signed kernels, is that if you have both the unsigned and signed kernels installed `update-initramfs -u` will update the initrd for your *unsigned* kernel, meaning that you won't see your changes unless you copy the initrd over! This flummoxed me a lot about the next step...

*   If you want to use the NVIDIA drivers for your shiny NVIDIA GPU, you need to blacklist nouveau. There are plenty of instructions on the internet but I can personally vouch for [remingtonlang's instructions](https://github.com/jakeday/linux-surface/issues/264#issuecomment-427452156). Make sure you are updating the correct initrd; see my bullet point above. Once this was fixed, a standard invocation of the CUDA installer got me working `nvidia-smi`. Note that I manually signed the NVIDIA using the [instructions here](https://askubuntu.com/questions/1023036/how-to-install-nvidia-driver-with-secure-boot-enabled) since I already had generated a private key, and it seemed silly to generate another one because NVIDIA's installer asked me to.

*   Once you install the NVIDIA drivers, you have to be careful about the opposite problem: Xorg deciding it wants to do all its rendering on the NVIDIA card! The usual symptom when this occurs is that your mouse input to Linux is very laggy. If you have working `nvidia-smi`, you can also tell because Xorg will be a running process on your GPU. In any case, this is bad: you do NOT want to use the dGPU for plain old desktop rendering; you want the integrated one. I found that uncommenting the sample Intel config in `/etc/X11/xorg.conf.d` fixes the problem:

    ```
    Section "Device"
        Identifier  "Intel Graphics"
        Driver      "intel"
    EndSection

    ```

    But this doesn't play too nicely with VMWare; more on this below.

*   Sound did not work (it was too soft, or the right speaker wasn't working) until I upgraded to Linux 5.0.1.

*   After enabling XInput on [my fork of Xournal](https://github.com/ezyang/xournal), it did not start working until I restarted Xournal. Eraser worked right out of the box.

*   Don't forget to make a swap partition (Ubuntu default installer didn't prompt me to make one, probably because I was installing as dual-boot); otherwise, hibernate will not work.

*   Sometimes, when waking up from hibernate, networking doesn't work. Mercifully, this can be fixed by manually reloading the WiFi kernel module: `modprobe mwifiex_pcie` and `systemctl restart NetworkManager.service`. More discussion on [this issue.](https://github.com/jakeday/linux-surface/issues/431)

*   Sometimes, when waking up from hibernate/suspend, I get a big thermometer icon. When I reboot again it goes away but I have lost my hibernate/suspend. Perplexing! I don't know why this happens.

### Boot via VM

The sad truth of life is that the Windows tablet experience is much better than the Linux experience--to the point where many would just install Windows and then boot Linux from a virtual machine (or Windows Subsystem for Linux). This was a non-starter for me: a bare metal boot of Linux was necessary to get the best pen input experience. However, why not also make it possible to boot the Linux partition from VMWare running on Windows? This setup is [explicitly supported by VMWare](https://www.vmware.com/support/ws5/doc/disks_dualmult_ws.html), but it took a few days of fiddling to get it to actually work.

*   First, you need VMWare Workstation Pro to actually configure a VM that accesses raw disk (although the resulting VM image can be run from the free VMWare Player). You can sign up for the thirty-day trial to get it configured, and then use Player from then on, if you like. VMWare will offer the raw disk as an option when setting up disk; pick that and select the Linux partitions on your machine.
*   The primary challenge of setting up this system is that a standard install of Linux on the Surface Book 2 doesn't have a traditional Linux boot partition; instead, it has an EFI partition. Most notably, this partition is *permanently mounted* by Windows on boot up, so you can't remount it for your VM. Your regular partition doesn't have a bootloader, which is why when you turn on your VM, you get kicked into network boot via PXE. The workaround I ended up applying is to make a new, fake disk (vmdk-backed) and install the boot partition onto that (you don't actually need any of the kernels or initrds, since they live on your root filesystem; only `/boot/efi` is mounted from the EFI partition). Of course, you have to actually setup this boot partition; the way I did it was to chroot into my partition on a rescue CD and then run `grub-install /dev/sda1`. In the course of fiddling, I also accidentally ran `update-grub` which blew away my Windows boot option, but re-running this command when booted into Linux bare-metal fixed the problem (because the real `/boot/efi` will mount and thus Grub will find the Windows boot option.)
*   Some documentation about dual-boot is specific to VMWare Fusion. This is OS X specific, so not relevant to the Microsoft Surface Book 2.
*   Get yourself a bootable Linux CD (I personally use [SystemRescueCd](http://www.system-rescue-cd.org/)) to help debug problems in the installation process.
*   Make sure all of your `/etc/fstab` entries correspond to real disks, or your Ubuntu startup process will spend a while waiting for a disk that is never going to show up. I had this problem with the `/boot/efi` mount, because the mount was UUID based; I "fixed" it by changing the mount to be LABEL based and labeling my vmdk accordingly (I suppose it might also have been possible to change the UUID of my vmdk, but I couldn't find any reasonable instructions for doing so on Windows). Note that the volume doesn't actually have to successfully mount (mine doesn't, because I forgot to format it vfat); it just has to exist so system doesn't wait to see if it connects at some later point in time.
*   I don't really understand how Unity decides to provide scaling options, but although it offers magnification on a bare metal boot, those options are not available when run under a VM. I get something tolerably sized (with only slight blurriness) by setting the resolution to 1680 x 1050; play around a bit with it. I have "Stretch Mode" enabled in VMWare.
*   Whether or not you can log into your account depends on your X11 configuration; so if you're like me and uncommented the Intel configuration, I found this bricks my login (and you can unbrick it by commenting out again.) How do make both work? Don't ask me; I'm still figuring it out.

### Window manager

I haven't gotten around to setting up xmonad; this is no small part due to the fact that Unity appears to support a very rudimentary form of tiling: Windows-left and Windows-right will move Windows to fill the left/right half of the display, while Windows-up will full screen a Window. I might still try to get xmonad setup on 18.04, but for now it is nice not having to fight with trayer to get the standard icons.

### What's next

My two top priorities for improving the state of Linux on the Surface Book 2:

1.  Rewrite Xournal with support for hDPI (how hard could it be lol)
2.  Figure out how to make suspend/hibernate work more reliably

Otherwise, I am very happy with this new laptop. One thing in particular is how much faster my mail client (still sup) runs; previously, scanning for new mail would be a crawl, but on this laptop they stream in like a flash. Just goes to show how much an upgrade going from a 1.6GHz processor to a 4.2GHz processor is :3