# Pi-Cold

## Cold server setup

This can be any 64 bit capable Raspi. Any Raspberry Pi 4, Raspberry Pi 3b+ or a Raspberry Pi-400. I would use a Pi-400. Keyboard is built in which is convenient and safer. The cold machine is only used to sign transactions and is left powered down for 99% of the time(if not more). The Pi-400 can just be powered off and in a safe. Also unlike an online node, the Cold machine can be run from the sdcard. Just make sure you have multiple copies of your keys just in case you get a bad sdcard. Or better yet clone this system onto 3 or 4 other bootable sdcards. Remember you can also boot from USB.

#### Ubuntu or Raspbian

Raspberry Pi OS desktop is a lot faster and more stable than the gnome desktop in Ubuntu. There is now a 64bit image you can install, it is not available in raspi-imager selection, IDK why. Check out the images in the link below grab the latest version. It is a zip file so we have to unzip it.

{% embed url="https://downloads.raspberrypi.org/raspios_arm64" %}

Unzip the img file and flash it with Raspi-imager.

### Log in & setup user

Disable the radios. It would just be foolish to leave these enabled..

```bash
sudo rfkill block wifi
sudo rfkill block bluetooth
```
I put a network jack without a wire into the NIC port to block it. Preventing any accidental insertion.

Open up Preferences/Raspberry Pi Configuration window(GUI) and disable auto login.

Create the $USER and add it to sudoers. ada in our case. We have to create a new user so the uid, gid and name match that of the core. It is far less error prone than trying to change the user id of the default user. With the user/group id of 1001 you will not run into issues with permissions transferring between systems.

```bash
sudo adduser ada; sudo adduser ada sudo
```

Reboot the server to stop the autologin pi user process running in the background. Log in as ada and delete the pi user.


```bash
sudo deluser --remove-home pi
```
{% hint style="Huomaa" %}
If you find your keyboard is not correctly printing the home symbol ~, you repair with, sudo locale-gen and reboot.
{% endhint %}

### USB transfer

Basically repeating the steps to setup an fstab entry from the Core guide. This is to mount the USB transfer disk at boot should you have it inserted when you power on. It also makes the mount command simpler. You can just sudo mount usb-transfer.

Create the mount point & set default ACL for files and folders with umask.

```bash
cd; mkdir $HOME/usb-transfer; umask 022 $HOME/usb-transfer
```

Attach the external drive and list all drives with fdisk.

```bash
sudo fdisk -l
```

If you are booting from the sdcard the first inserted disk is /dev/sda. The sdcard you are booting from will have the /dev/mmcblk0 designation.

Example output for my system:

```bash
Disk /dev/sda: 57.66 GiB, 61907927040 bytes, 120913920 sectors
Disk model: Cruzer
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: gpt
Disk identifier: 75B71A3E-9E1A-4659-94D0-E0C949A26740

Device     Start       End   Sectors  Size Type
/dev/sda1   2048 120913886 120911839 57.7G Linux filesystem
```

In my case it is /dev/sda because I am using the sdcard slot in the pi-400. Yours may be /dev/sdb, /dev/sdc, /dev/sdd or so on. /dev/sda is usually the system drive. Just be careful you are dealing with the correct drive.

Locate your drive and get the UUID for the partition we created earlier.

Run blkid and copy the UUID. In my case it is the UUID for /dev/sda1.

```bash
sudo blkid
```

Example output:

```bash
UUID="c2a8f8c7-3e7a-40f2-8dac-c2b16ab07f37"
```

Add a mount entry to the bottom of fstab adding your UUID and the full system path to you backup folder.

```bash
sudo nano /etc/fstab
```

```bash
UUID=c2a8f8c7-3e7a-40f2-8dac-c2b16ab07f37 /home/ada/usb-transfer auto nosuid,nodev,nofail 0 1
```

> nofail allows the server to boot if the drive is not inserted.

#### Test your drive mounts

Mount the drive and confirm it mounted by locating listing the contents. You should see the ada folder, offline-transfer and lost\&found. If they are not present then your drive is not mounted.

```bash
sudo mount usb-transfer; ls $HOME/usb-transfer
```

### Transfer contents of the USB stick.

The cold machine will never be online, these machines do not have any cmos battery to keep time. It should not pose too many issues if any. Just be aware that when you fire up the cold machine it's time is off.

```bash
cd; rsync -aP usb-transfer/ada/ ~/
```

Source the .adaenv file on login.

```bash
echo . ~/.adaenv >> ~/.bashrc
```

Switch the Stake Pool Operator scripts to 'offline mode'.

```bash
cd
sed -i stakepoolscripts/bin/common.inc \
    -e 's#offlineMode="no"#offlineMode="yes"#'
```

Move the jq binary into it's system PATH.

```bash
sudo cp usb-transfer/ada/jq /usr/local/bin
jq -V
```

Confirm SPOS is installed.

```bash
. .adaenv; 00_common.sh
```

### Install VSCodium

Right click on the VSCodium .deb file in the usb-transfer folder and choose install. You can then open the markdown files and use ctr+shift+V to render a preview.

{% embed url="https://vscodium.com" %}

### Pool Creation

That's it! you can now use Martins guide in the stakepoolscripts/bin file. Remember to always manually unmount your USB stick before unplugging it.

{% embed url="https://github.com/gitmachtl/scripts" %}

Thank you Martin for all your hard work!
