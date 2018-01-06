# GPD Pocket Arch Linux Full disk encryption
Tutorial how to install Arch Linux with (almost) full disk encryption using LUKS and LVM on GPD Pocket

https://github.com/sigboe/GPD-Arch-LUKS-LVM/wiki

## Todo

- [x] Custom Arch ISO with WiFi and repository already set up
- [ ] Integrate guide in the ISO
- [x] Improve guide
- [x] More improvements to the installation
- [x] Install Gnome
- [x] Install KDE
- [x] Other optional software

## What works?

Most things work, if not mentioned it is likely that it works.

- [x] Readable terminal shortly after bootloader
- [x] Multitouch 
- [x] Wifi
- [x] Bluetooth 
- [x] Speaker 
- [x] Headphones with autoswitch
- [x] Battery manager
- [x] Intel video driver
- [x] Sleep/wake
- [x] USB-C for data
- [x] HDMI port
- [x] Audio over HDMI
- [ ] Displayport/HDMI over USB-C
- [ ] Hibernation (Its possible to do it unreliably)

## Building the ISO

### Dependencies

    pacman -S archiso

If you want to build the ISO on another platform than Arch Linux I suggest doing it in a VM or a Docker running Arch. It should work in distros based on arch, but this is not supported.

### Procedure

The files are very sensitive to file permission changes, so unfortunately you need to do everything as root (even git clone). This may be fixed upstream later.

    sudo su
    cd
    git clone https://github.com/sigboe/GPD-ArchISO.git
    cd GPD-ArchISO/archlive
    mkdir out
    ./build.sh -v
    
#### Cleanup

You need to clean up after the script before running it again. You need to `rm -rf work`. **Make sure** you do this in the **correct directory** because you do run it with root permissions. 

If you exited the script during execution you may end up with files that are mounted inside work. This will prevent you from deleting work. Then you may run

    umount --recursive work

## Credits 

[joshskidmore]() lots of help, inspirations and awesome commits  
[emanuelduss](https://emanuelduss.ch/2016/03/arch-linux-installation-gpt-luks-lvm-i3/) as basis for partitioning, encryption and bootloader.  
[jwrdegoede](https://github.com/jwrdegoede/) for making the kernel  
[njkli](https://github.com/njkli/)For working on lots of the packages used after install  
[cawilliamson](https://github.com/cawilliamson) For working on lots of the packages used after install  
[microdou](https://github.com/microdou) for testing, finding bugs, and filing issues.  
