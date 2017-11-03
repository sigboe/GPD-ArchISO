# GPD Pocket Arch Linux Full disk encryption
Tutorial how to install Arch Linux with (almost) full disk encryption using LUKS and LVM on GPD Pocket

https://github.com/sigboe/GPD-Arch-LUKS-LVM/wiki

## Todo

- [x] Custom Arch ISO with WiFi and repository already set up
- [ ] Improve guide
- [ ] More improvements to the installation
- [ ] Install Gnome
- [ ] Install KDE
- [ ] Other optional software

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
- [ ] Audio over HDMI
- [ ] Displayport/HDMI over USB-C
- [ ] Hibernation

## Building the ISO

### Dependencies

    pacman -S arch-install-scripts dosfstools libisoburn lynx make squashfs-tools

If you want to build the ISO on another platform than Arch Linux I suggest doing it in a VM or a Docker running Arch. It should work in distros based on arch, but this is not supported.

### Procedure

The files are very sensitive to file permission changes, so unfortunately you need to do everything as root (even git clone). This is a wontfix issue even upstream.

    sudo su
    cd
    git clone https://github.com/sigboe/GPD-Arch-LUKS-LVM.git
    cd GPD-Arch-LUKS-LVM/archlive
    mkdir out
    ./build.sh -v
