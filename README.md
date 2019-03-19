# X250 Hackintosh (macOS 10.14.x)

This is a guide to set up the Lenovo Thinkpad X250 as a 99% working hackintosh.

## Introduction

Thinkpad X250 Hackintosh configuration. This repository contains the following folders:

- `EFI`: put this in your EFI partition in `EFI` folder, including `Boot` and `CLOVER` sub-folders,
- `Kexts`: kexts to install in `/Library/Extensions` on your local drive once macOS has been installed.

It's a `99.99%` working hackintosh. What doesn't work (yet):

- Trackpad
- Trackpad Buttons
- Wifi (use a USB Wifi Dongle or replace the Wifi Card with a DW1560)
- Card Reader

## Setup

### Bios Settings

The bios must be properly configured prior to installing MacOS.

In `Security` menu, set the following settings:

- `Security > Security` Chip: must be **Disabled**,
- `Memory Protection > Execution Prevention`: must be **Enabled**,
- `Virtualization > Intel Virtualization Technology`: must be **Disabled**,
- `Internal Device Access > Bottom Cover Tamper Detection`: must be **Disabled**,
- `Anti-Theft > Current Setting`: must be **Disabled**,
- `Anti-Theft > Computrace > Current Setting`: must be **Disabled**,
- `Secure Boot > Secure Boot`: must be **Disabled**.

In `Startup` menu, set the following options:

- `UEFI/Legacy Boot`: **Both**,
- `UEFI/Legacy Priority`: **UEFI First**,
- `CSM Support`: **Yes**.

### Requirements

* A minimum of 4GB USB drive
* Python 2.7 or greater:
  * For Windows, get it from [https://www.python.org/downloads/windows/](https://www.python.org/downloads/windows/) and make sure you enable "add to PATH" in the install
  * For linux users, install it if you dont have it following your distro's tools
  * For macOS users, you already have 2.7+ version installed, no need for extra tools
* gibMacOS: a sweet tool from /u/corpnewt [https://github.com/corpnewt/gibMacOS](https://github.com/corpnewt/gibMacOS)
  * press `Clone or Download` button and download as Zip, extract it somewhere
* Other software requirements will be downloaded thorough the guide \(OS specific\)

### Downloading the Recovery HD image...

After downloading `gibMacOS` \(that will be refereed as gOS later on\), you'll have to open the the folder where it is \(or `cd` to it if you clone it with git\). If you're using macOS or Linux, you should know how to `cd` to it through your preferred Terminal emulator. If you're a **Windows** user, follow these instructions to get to the command line inside the folder.

* Open Command Prompt/PowerShell and `cd` to where it is.
* OR open the folder where it is, press `Files` &gt; `Open Windows PowerShell`

Now that you're inside the command prompt/powershell/terminal, we'll use gibMacOS to download the Recovery HD to be restored:

1. run `python gibMacOS.command -r -v <macos version>` and replace `<macos version>` with your target version:
   1. `10.14` or `mojave` for macOS Mojave release
   2. `10.13` or `"high sierra"` \(with quotes\) for macOS High Sierra release
   3. `10.11` or `"el capitan"` \(with quotes\) for OS X El Capitan release
   4. `10.10` or `yosemite` for OS X Yosemite release
   5. `10.9` or `mavericks` for OS X Mavericks release
   6. You may be asking where is Sierra, well ever since Sierra, apple started removing links from the AppStore to that release and thus from the release catalog, you can manually download from [here](http://swcdn.apple.com/content/downloads/01/53/031-86778/pnekzincp6rkf5iu91onj1bm5mw1gotnwg/RecoveryHDUpdate.pkg). \(thanks cvad for providing backup links in his BDU tool\) I think the same will happen to High Sierra and Mojave, ¯\\_\(ツ\)\_/¯.
2. The software will download the recovery image, meanwhile:
   1. Plug your USB device \(USB 2.0 drive are recommended\)
   2. Backup any data \*from\* it
   3. Format it \(it will be formatted anyways later on\)

Now that's downloaded, let's move on to the USB drive making...

### Preparing your installer media...

After the download, you'll find a new folder named `macOS downloads` where the program we used downloaded macOS recovery image for your macOS version. For this part of the setup. I'll be separating it to the 3 main OSes and how to make it.

#### Windows

Inside the same folder of gibMacOS, you'll find a cute `MakeInstall.bat` , double click it, accept UAC admin user, you'll be greeted by a CMD screen. The software will download software needed \(`dd` port for windows and 7-zip\). Then you'll be greeted with a list of drives:

* **Carefully** choose your drive \(look for the size, name or other features that points to your USB device\).
* type in its number
* press enter and follow the rest of the instructions

#### Linux

This section will target making the necessary partitions in the USB device. You can use your favorite program be it `gdisk` `fdisk` `parted` `gparted` or `gnome-disks`. I will focus on `gdisk` as it's nice ¯\\_\(ツ\)\_/¯ and can change the partition type later on, as we need it so that macOS Recovery HD can boot. \[the distro used here is Ubuntu 18.04, other versions or distros may work\]

In terminal:

1. run `lsblk` and determine your USB device block
2. run `sudo gdisk /dev/<your USB block>`
   1. send `p` to print your block's partitions \(and verify it's the one needed\)
   2. send `o` to clear the partition table and make a new GPT one
      1. confirm with `y`
   3. send `n`
      1. partition number: keep blank for default
      2. first sector: keep blank for default
      3. last sector: `+200M` to create a 200MB partition that will be named later on CLOVER
      4. Hex code or GUID: `0700` for Microsoft basic data partition type
   4. send `n`
      1. partition number: keep blank for default
      2. first sector: keep blank for default
      3. last sector: keep black for default \(or you can make it `+3G` if you want to partition further the rest of the USB\)
      4. Hex code or GUID: `af00` for Apple HFS/HFS+ partition type
   5. send `w`
      1. Confirm with `y`
   6. Close `gdisk` by sending `q`
3. Use `lsblk` again to determine the 200MB drive and the other partition
4. run `mkfs.vfat -F 32 -n "CLOVER" /dev/<your 200MB partition block>` to format the 200MB partition to FAT32, named CLOVER
5. then `cd` to `macOS downloads` and keep going until you get to a `pkg` file
   1. download `p7zip-full` \(depending on your distro tools\)
      1. for ubuntu/ubuntu-based run `sudo apt install p7zip-full`
      2. for arch/arch-based run `sudo pacman -S p7zip`
      3. for the rest of you, you should know ¯\\_\(ツ\)\_/¯
   2. run this `7z e -txar *.pkg *.dmg; 7z e *.dmg */Base*; 7z e -tdmg Base*.dmg *.hfs` this will extract the recovery from the pkg through extracting the recovery update package then extracting the recovery dmg then the hfs image from it.
   3. then when you get `4.hfs` or `3.hfs` \(depending on the macOS version used\) run `dd if=*.hfs of=/dev/<your USB's second partition block> bs=8M --progress` \(you may change the input file `if` and the block size to match your needs
6. after that, you're ready to go for the next step of the guide...

## crackOS \(macOS\)

After the download, open `macOS downloads/.../...` until you find RecoveryHDUpdate.pkg or RecoveryHDMetaDmg.pkg \(enable file name extensions in Finder under Finder &gt; Preferences &gt; Advanced\).

1. locate the pkg
2. do not open it
3. rename the extension from `.pkg` to `.dmg`
4. open the image \(it will be mounted\)
5. find `Basesystem.dmg`
6. double click to mount it
7. open `Disk Utility` application \(Launchpad &gt; Other &gt;\)
   1. Select View &gt; Show All devices \(above the drive list\)
   2. Select your USB device \(not the partition\)
   3. Select Partition
      1. if you do not see partition:
         1. Select Erase
         2. Name: whatever Format: MacOS Extended Journaled Scheme: GUID Partition Table
   4. Make:
      1. a minimum of 200MB partition Name: CLOVER Format: MSDOS
      2. the rest Name: whatever you want Format: MacOS Extended Journaled
   5. Apply
   6. Select the MacOS Extended Journaled partition
   7. Select Restore
   8. Choose BaseSystem from the list \(it should be the image you mounted earlier\)
   9. Let it restore
8. When done you can continue this guide...

### Copy EFI Folder to USB

Copy the content of the `EFI` folder provided in my repo onto your USB flash drive `EFI` partition. The EFI partition is usually hidden. Use [Clover Configurator](https://mackie100projects.altervista.org/download-clover-configurator/) to mount the EFI partition of your flash drive on your mac (it appears as a disk on the desktop once done). On Windows or Linux it shouldn't be hidden.

### Install macOS

Install macOS by booting on the USB key. It takes about 30min. The computer will restart multiple times. Make sure to select `Install macOS ...` each time. Once installed, choose to boot from local drive in Clover boot menu.

### What's next?

To finish the setup, you need to make your local drive bootable (right now you are still using the Clover boot from your USB installer to boot your system:

- **Copy EFI** folder from USB flash drive to local drive `EFI` partition (like you did for the USB installer). Use [Clover Configurator](https://mackie100projects.altervista.org/download-clover-configurator/) to mount it.
- **Install Kexts**: install kernel extensions provided by `Kexts` from this repository into `/Library/Extensions`. Use this guide to [properly install the kexts.](https://www.tonymacx86.com/threads/guide-installing-3rd-party-kexts-el-capitan-sierra-high-sierra-mojave.268964/)]

You're almost done! Reboot and enjoy macOS on your Thinkpad X250.

## Miscellaneous

### SSD Enable Trim

If you Sata ssd hasn't trim enabled, run the following command from the *Terminal* to enable it:

```
sudo trimforce enable
```
