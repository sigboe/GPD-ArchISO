# GPD-Arch-LUKS-LVM
Tutorial how to install Arch Linux with full disk encryption on GPD Pocket

### Booting

* Plug your USB stick with arch into the USB-A (regular usb port)
* Plug a USB-C hub or a dock or something to give you ethernet (its possible to do everything over wifi but out of scope for this step by step as its lots of extra work)

While you boot spam the Del key (if you have problems try without the charger)

Plug back your charger or ethernet to usb-C when you see the UEFI setup screen (formerly known as BIOS back in the 90's).

Hit left once to go to the menu containing save and exit, but find the USB stick as a temporary boot device.

You will very soon see systemd boot menu for arch usb, I press an arrow key to alt the boot process. On the main boot entry press E to edit it. Go to the end (hint the End key is Fn+ right arrow)

add `fbcon=rotate:1` to the end and hit enter.

First command you should write is one to make the text more readable

    setfont latarcyrheb-sun32

If you want to test if you have internet connection you may 

    ping google.com

### Optional

If you want to be able to copy paste the commands from another PC using ssh from a Linux or Mac machine (Putty, Kitty, or MobaXterm on Windows).

You just need to set a password for the root user on the live USB

    passwd

Then enable SSH

    systemctl start sshd

To learn your IP you may run (or check your routers config)

    ip addr show

Then from a nicer bigger PC or Mac with a webbrowser, use the command `ssh root@IP` or use Putty,Kitty,MobaXterm on Windows.

### Partitioning

**Warning, this does wipe your whole disk!!!!**

    gdisk /dev/mmcblk0
    # o ↵ to create a new empty GUID partition table (GPT)
    # y ↵ to confirm

    # n ↵ add a new partition
    # ↵ to select default partition number of 1
    # ↵ to select default start at first sector
    # +512M ↵ make that size partition for booting
    # ef00 ↵ Partition type EFI

    # n ↵ to add new partition
    # ↵ to select default partition number of 2
    # ↵ to select default start of sector
    # ↵ to select default end of sector
    # 8e00 ↵ to make partition type of LVM

    # p ↵ if you want to check the partition layout
    # w ↵ to write changes to disk
    # y ↵ to confirm

### Formatting

The EFI partition

    mkfs.fat -F32 /dev/mmcblk0p1

Encrypt the LVM partition with LUKS

    cryptsetup luksFormat -v -s 512 -h sha512 /dev/mmcblk0p2

Read the warning thourougly it says **you need to confirm writing YES in capital letters**. set a password.

Then open the partition.

    cryptsetup luksOpen /dev/mmcblk0p2 luks

Initialize a physical volume

    pvcreate /dev/mapper/luks

Create a volume group, we'll call it rootvg

    vgcreate rootvg /dev/mapper/luks

Create swap `-C` makes continuous data blocks

    lvcreate -n swap -L 8G -C y rootvg

Create a root partition (can resize later if you need, probably not)

    lvcreate -n root -L 30G rootvg

Create /home partition (**lower case** `-l` this time)

    lvcreate -n home -l 100%FREE rootvg

### Formatting continued

This formats home and root as ext4 filesystems. Then designates the swap partition as such, and enables the swap.

    mkfs.ext4 /dev/mapper/rootvg-home
    mkfs.ext4 /dev/mapper/rootvg-root
    mkswap /dev/mapper/rootvg-swap
    swapon /dev/mapper/rootvg-swap

### Mounting before install

Mount root to /mnt (this is the convention while installing)  

    mount /dev/mapper/rootvg-root /mnt

Make a boot and a home folder (two commands in one :) )

    mkdir /mnt/{home,boot}

Mount home and boot

    mount /dev/mapper/rootvg-home /mnt/home
    mount /dev/mmcblk0p1 /mnt/boot

### Installing

Install Arch some things needed for AUR repository and an easier text editor.

    pacstrap /mnt base base-devel nano zsh grml-zsh-config

Generate the fstab

    genfstab -pU /mnt > /mnt/etc/fstab

Remote control your installation (we need to make it boot)

    arch-chroot /mnt /usr/bin/zsh

### Install some of the GPD packages

Uninstall linux (yay)

    pacman -R linux

add GPD repostiory

    nano /etc/pacman.conf

Uncomment the [multilib] repository by removing the #, its neear the bottom. Then add the following repo.

    [gpd-pocket] 
    SigLevel = Optional TrustAll 
    Server = https://github.com/njkli/$repo/releases/download/$arch

CTRL+x, then y then ↵ to save and exit

Update repository

    pacman -Suy

Install some packages related to the GPD

    pacman -Syyu --force gpd-pocket-support linux-jwrdegoede-docs linux-jwrdegoede-headers

### Configuring boot environment

    nano /etc/mkinitcpio.conf

Find the line starting with `HOOKS="` then make it say

    HOOKS="base consolefont udev autodetect keyboard keymap modconf block encrypt lvm2 filesystems fsck"

CTRL+x, then y then ↵ to save and exit  
The order here is important. Then we will set the consolefont.

    echo "FONT=latarcyrheb-sun32" >>  /etc/vconsole.conf

Then make the initramfs

    mkinitcpio -p linux-jwrdegoede

Install the bootloader

    bootctl install

Configure it to boot.  
First we want to write down the UUID of the partition with this weird command

    blkid | grep mmcblk0p2 | cut -f2 -d\" > /boot/loader/entries/arch.conf

Then edit the file

    nano /boot/loader/entries/arch.conf

You will only see the UUID. You can put it into nanos clipboard by pressing ctrl+k and paste it using ctrl+u

    title Arch Linux
    linux /vmlinuz-linux-jwrdegoede
    initrd /initramfs-linux-jwrdegoede.img
    options cryptdevice=UUID=xxxx-yyyy-zzzz-aaaa:luks root=/dev/mapper/rootvg-root quiet rw fbcon=rotate:1

paste the UUID in after `options cryptdevice=UUID=` and before `:luks root=/dev/mapper/rootvg-root quiet rw fbcon=rotate:1`  
CTRL+x, then y then ↵ to save and exit


Make this the default boot entry

    nano /boot/loader/loader.conf

Have the text be

    timeout 2
    default arch

CTRL+x, then y then ↵ to save and exit

### Do some basic config

Create your user

    useradd -m -g users -G wheel,storage,power -s /usr/bin/zsh USERNAMEHERE
    passwd USERNAMEHERE

Give this user sudo access

    EDITOR=nano visudo

uncomment the line `%wheel ALL=(ALL) ALL`

CTRL+x, then y then ↵ to save and exit

set hostname

    echo LOWERCASEHOSTNAMEHERE > /etc/hostname

set your timezone (use your own continent and city)

    ln -sf /usr/share/zoneinfo/Europe/Oslo /etc/localtime

Configure locale

    nano /etc/locale.gen

Uncomment the lines `en_US.UTF-8 UTF-8` and `en_US ISO-8859-1`  
CTRL+x, then y then ↵ to save and exit

generate locales

    locale-gen

set language

    echo "LANG=en_US.UTF-8" > /etc/locale.conf

Configure power management settings

    nano /etc/default/tlp

Change the line starting with `DISK_DEVICES="` to `DISK_DEVICES="mmcblk0"`  
and uncomment and change `#DISK_IOSCHED="` to `DISK_IOSCHED="deadline"`

CTRL+x, then y then ↵ to save and exit

Configure the fan

    cp /etc/default/gpd-fan.example /etc/default/gpd-fan

Uncomment all the lines that are settings.  
CTRL+x, then y then ↵ to save and exit

# Congrats, you may now reboot the system. You can install whatever desktop enviornment you like, now or after reboot. Or you can reboot to test if its working, then boot into the USB again and take a backup of your install using `clonezilla` which is included on the USB.

After the reboot you may want to install pacaur to get access to AUR. Dont do this before rebooting, and dont do it as root.

    git clone https://aur.archlinux.org/cower.git
    cd cower
    makepkg -si
    cd
    git clone https://aur.archlinux.org/pacaur.git
    cd pacaur
    makepkg -si
    cd
    rm -rf {cower,pacaur}

###Credits
[emanuelduss](https://emanuelduss.ch/2016/03/arch-linux-installation-gpt-luks-lvm-i3/) as basis for partitioning, encryption and bootloader.  
Hans de Goede for making the kernel  
[njkli](https://github.com/njkli/) for compiling the kernel and updating it  
u/sultanmvp for being awesome  

Edits: I fix anything as I find typos
