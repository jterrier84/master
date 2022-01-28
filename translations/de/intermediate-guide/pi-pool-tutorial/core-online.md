# Pi-Core

Create another Pi-Node from the guide or burn the Armada Alliance Pi-Node.img.gz to a new drive.

## Core server setup

We will be using Martin Lang's [ATADA](https://cardanoscan.io/pool/00000036d515e12e18cd3c88c74f09a67984c2c279a5296aa96efe89) StakePool Operator Scripts to manage our pool and interact with the blockchain. These scripts not only handle pool creation & operations. They can be used with a hardware wallet, send ada, create NFT's and more.

Unlike [CNTools](https://cardano-community.github.io/guild-operators/Scripts/cntools/) they do not hook into systemd and can be used in any directory under the users home folder. We already have gLiveView.sh and you can retreive the pools slot schedule [1.5 days before epoch boundary](https://github.com/asnakep/ScheduledBlocks) without CNCLI thanks to [SNAKE](https://cardanoscan.io/pool/342350284fd76ba9dbd7fd4ed579b2a2058d5ee558f8872b37817b28) pool!

Martin provides a [script](https://github.com/gitmachtl/scripts/tree/master/cardano/mainnet#import-your-existing-pool-from-cli-keys-or-tutorials) that can help you import your existing pool to work with SPOS.

{% embed url="https://github.com/gitmachtl/scripts" %}

Please visit and review the configuration, scriptfiles syntax & filenames to better familiarize yourself with the workflow and capabilities of this awesome toolset. All the instructions and commands needed to use these scripts are in the README files located in the stakepoolsripts folder.

### Online Core Installation

Refer to the Pi-Relay guide and update these settings accordingly for a Core server and reboot.
- Set hostname in /etc/hosts & /etc/hostname
- Static IP (192.168.1.150 or whatever works for you on your LAN)
- Port to 3000

```bash
sudo reboot
```

While your node is syncing back up, clone the Stakepool Operator Scripts repo into your home directory. Create the bin folder to hold the scripts and add them to your PATH.

```bash
cd; git clone https://github.com/gitmachtl/scripts.git $HOME/stakepoolscripts
mkdir -p $HOME/stakepoolscripts/bin; cd $_
echo "export PATH=\"$PWD:\$PATH\"" >> $HOME/.adaenv
export PATH="$PWD:$PATH"; . $HOME/.adaenv
```

By now you should have chosen and synced your node on Testnet or Mainnet. There are two sets of scripts respectively. If you are on Testnet you can run a core with all the keys on it in Online mode. With Mainnet we set up an online Core running a full node and an offline machine that runs the same version of cardano-cli as the online machine uses. All key generation takes place on the cold machine for mainnet.

The Cold machine does not run cardano-node. It is offline.

This offline or cold machine protects the nodes cold keys and the owners pledge keys. A json file with built transactions are transfered to the cold machine for signing and then moved back to the core for submission. Preventing node and wallet keys from ever being on a machine connected to the internet.

```bash
cd $HOME/stakepoolscripts
git fetch origin; git reset --hard origin/master
```

Confirm these scripts are in your PATH and check the integrity of the scripts with git.

Git has checksums baked right in.

Everything in Git is an object. Every object has an ID. The IDs are a checksum of the content and connections. If the content or connections change, the ID is no longer valid.

For example, a commit ID is basically a checksum of...

* The contents and permissions of all files (git calls them "blobs") at the point of that commit (which have their own IDs).
* The fields of the commit like author, date, log message, etc...
* The commit IDs of the parent commits.

```bash
git fsck --full
# silence is golden
git status
```

You will see that our new bin folder is untracked. everything else should be up to date.

Copy the latest versions of the scripts into the bin folder.

```bash
rsync -av $HOME/stakepoolscripts/cardano/${NODE_CONFIG}/* $HOME/stakepoolscripts/bin
```

Martin hosts checksums for his files as well. You can learn how in the README files in the stakpoolscripts/bin folder.

I am in the habit of pulling updates, running a check against the repo and gathering copies of any binaries needed for USB transfer to the cold machine.

These would include the latest $HOME/stakepoolscripts/bin folder and a copy of the cardano-cli binary in $HOME/.local/bin. the rsync backup we take further down in this guide will copy everything necessary and it can be used to update the cold machines environment to match the core machine.

#### Common.inc

Create a variable for testnet magic, Byron to Shelley epoch value and a variable to determine whether we are on mainnet or testnet. If on testnet we append the magic value onto our CONFIG_NET variable.

```bash
echo export MAGIC=$(cat ${NODE_FILES}/${NODE_CONFIG}-shelley-genesis.json | jq -r '.networkMagic') >> ${HOME}/.adaenv; . ${HOME}/.adaenv
if [[ ${NODE_CONFIG} = 'testnet' ]]; then echo export BYRON_SHELLEY_EPOCHS=74; else echo export BYRON_SHELLEY_EPOCHS=208; fi >> ${HOME}/.adaenv
if [[ ${NODE_CONFIG} = 'testnet' ]]; then echo export CONFIG_NET='testnet-magic\ "${MAGIC}"'; else echo export CONFIG_NET=mainnet; fi >> ${HOME}/.adaenv; . ${HOME}/.adaenv
```

Copy the top portion of the 00_common.sh file into a new file named common.inc. This will hold the variable paths needed to connect these scripts to our running node.

```bash
cd $HOME/stakepoolscripts/bin/
sed -n '1,69p' 00_common.sh >> common.inc
```

And edit the lines needed to get up and running. I would look in this file beforehand to get an idea of what I am changing from defaults.

```bash
sed -i common.inc \
    -e 's#socket="db-mainnet/node.socket"#socket="${NODE_HOME}/db/socket"#' \
    -e 's#genesisfile="configuration-mainnet/mainnet-shelley-genesis.json"#genesisfile="'${NODE_FILES}'/'${NODE_CONFIG}'-shelley-genesis.json"#' \
    -e 's#genesisfile_byron="configuration-mainnet/mainnet-byron-genesis.json"#genesisfile_byron="'${NODE_FILES}'/'${NODE_CONFIG}'-byron-genesis.json"#' \
    -e 's#cardanocli="./cardano-cli"#cardanocli="cardano-cli"#' \
    -e 's#cardanonode="./cardano-node"#cardanonode="cardano-node"#' \
    -e 's#offlineFile="./offlineTransfer.json"#offlineFile="${HOME}/usb-transfer/offlineTransfer.json"#' \
    -e 's#byronToShelleyEpochs=208#byronToShelleyEpochs='${BYRON_SHELLEY_EPOCHS}'#' \
    -e 's#magicparam="--mainnet"#magicparam="--${CONFIG_NET}"#' \
    -e 's#addrformat="--mainnet"#addrformat="--${CONFIG_NET}"#'
```

This gets us what we need to continue. Have a look in the file for more options and edits you may need to make depending on your task(like catalyst voting, minting tokens or setting up a hardware wallet).

#### Test installation

Let's test we have these scripts in our PATH and test they are working.

```bash
cd; 00_common.sh
```

Should see this on testnet or similar for mainnet. If something went wrong Martin presents you with a nice mushroom cloud ascii drawing and a hint as to what failed. If you are not synced to the tip of the chain it will warn you that the socket does not exist!

```bash
Version-Info: cli 1.33.1 / node 1.33.1      Scripts-Mode: online        Testnet-Magic: 1097911063
```

Martin ships a few binaries that are built for x86. These are useless on ARM64 so keep in mind that the token registration and catalyst registration scripts will not work until we can build these binaries for ARM. Lets delete them to save any confusion.

```bash
cd
rm stakepoolscripts/bin/catalyst-toolbox
rm stakepoolscripts/bin/jcli
rm stakepoolscripts/bin/token-metadata-creator
rm stakepoolscripts/bin/voter-registration
```

**You need a fully synced node to continue.**

Watch sync progress by following journalctl.

```bash
sudo journalctl --unit=cardano-node --follow
```

## Configure your USB transfer stick

Grab a USB stick and set it up with an ext4 partition owned by $USER that we can transfer between our two machines.

Create the mount point & set default ACL for files and folders with umask.

```bash
cd; mkdir $HOME/usb-transfer; umask 022 $HOME/usb-transfer
```

Attach the external drive into one of the USB2 ports and list all drives with fdisk. Some drive adapters eat a lot of power and you do not want to risk another USB device eating too much power on USB3 triggering a bus reset.

```bash
sudo fdisk -l
```

Example output:

```bash
Disk /dev/sdb: 57.66 GiB, 61907927040 bytes, 120913920 sectors
Disk model: Cruzer
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: gpt
Disk identifier: EECA81B9-3683-4A59-BC63-02EEDC04FD21
```

In my case it is /dev/sdb. Yours may be /dev/sdc, /dev/sdd or so on. /dev/sda is usually the system drive. **Do not format your system drive by accident**.

### Create a new GUID Partition Table (GPT)

**This will wipe the disk**

```bash
sudo gdisk /dev/sdb
```

Type ? to list options

```
Command (? for help): ?
b   back up GPT data to a file
c   change a partition's name
d   delete a partition
i   show detailed information on a partition
l   list known partition types
n   add a new partition
o   create a new empty GUID partition table (GPT)
p   print the partition table
q   quit without saving changes
r   recovery and transformation options (experts only)
s   sort partitions
t   change a partition's type code
v   verify disk
w   write table to disk and exit
x   extra functionality (experts only)
?   print this menu
```

1. Enter **o** for new GPT and agree to wipe the drive.
2. Enter **n** to add a new partition and accept defaults to create a partition that spans the entire disk.
3. Enter **w** and 'Y' to write changes to disk and exit gdisk.

Your new partition can be found at /dev/sdb1, the first partition on sdb.

#### Optionally Check the drive for bad blocks (takes a few hours)

```bash
badblocks -c 10240 -s -w -t random -v /dev/sdb
```

### Format the partition as ext4

We still need to create a new ext4 file system on the partition.

```bash
sudo mkfs.ext4 /dev/sdb1
```

Example output:

```bash
mke2fs 1.46.3 (27-Jul-2021)
Creating filesystem with 15113979 4k blocks and 3784704 inodes
Filesystem UUID: c2a8f8c7-3e7a-40f2-8dac-c2b16ab07f37
Superblock backups stored on blocks:
    32768, 98304, 163840, 229376, 294912, 819200, 884736, 1605632, 2654208,
    4096000, 7962624, 11239424

Allocating group tables: done
Writing inode tables: done
Creating journal (65536 blocks): done
Writing superblocks and filesystem accounting information: done
```

### Mount the drive at boot

Since it will be holding sensitive data we will mount it in a way where only root and the user cardano-node runs as can access.

Run blkid and pipe it through awk to get the UUID of the file system we just created.

```bash
sudo blkid /dev/sdb1 | awk -F'"' '{print $2}'
```

Example output:

```bash
c2a8f8c7-3e7a-40f2-8dac-c2b16ab07f37
```

For me the UUID=c2a8f8c7-3e7a-40f2-8dac-c2b16ab07f37

Add a mount entry to the bottom of fstab adding your UUID and the full system path to you usb-transfer folder.

```bash
sudo nano /etc/fstab
```

```bash
UUID=c2a8f8c7-3e7a-40f2-8dac-c2b16ab07f37 /home/ada/usb-transfer auto nosuid,nodev,nofail 0 1
```

> nofail allows the server to boot if the drive is not inserted.

#### Test your drive mounts

Mount the drive and confirm it mounted by locating the lost+found folder. If it is not present then your drive is not mounted.

```bash
sudo mount usb-transfer; ls $HOME/usb-transfer
```

Take ownership of the file system.

```bash
sudo chown -R $USER:$USER $HOME/usb-transfer
```

Now your stick will auto mount if it is left in the core machine and it is rebooted. We will repeat these steps on the offline cold machine. When you plug it into a running server just issue the same mount/check command.

We already set the location of our USB mount in the SPOS common.inc file. We can again test our installation by creating a new offlineTransfer.json file which we need for continuing in offline mode on our Cold machine.

```bash
01_workOffline.sh new
```

Lets copy this environment to the offline machine. We want the environment identicle and rsync is great for this.

### Grab jq on your way out

The Pi-Node has a static(portable) binary that can be transfered to the cold machine. Build instructions can be found in the environment section of the guide.

{% embed url="https://github.com/stedolan/jq" %}

Locate and copy the static jq binary we built earlier to our $HOME directory.

```bash
sudo cp /usr/local/bin/jq $HOME
```

Create an rsync-exclude.txt file so we can rip through and grab everything we need and skip the rest.

```bash
cd; nano exclude-list.txt
```

Add the following.

```bash
.bash_history
.bash_logout
.bashrc
.cache
.config
.local/bin/cardano-node
.local/bin/cardano-service
.profile
.selected_editor
.ssh
.sudo_as_admin_successful
.wget-hsts
git
tmp
pi-pool/db
pi-pool/scripts
pi-pool/logs
usb-transfer
exclude-list.txt
```

### Download the guide markdown files

Grab this guide so you can view it on the offline machine.

```bash
wget https://raw.githubusercontent.com/armada-alliance/master/master/docs/intermediate-guide/pi-pool-tutorial/core-online.md
wget https://raw.githubusercontent.com/armada-alliance/master/master/docs/intermediate-guide/pi-pool-tutorial/cold-offline.md
```

Optionally use VSCodium editor, the opensource VSCode to render markdown files on the offline machine. This makes Martins markdown easier to read. It has no Microsoft non free blobs like VSCode.

{% embed url="https://vscodium.com" %}

```bash
wget https://github.com/VSCodium/vscodium/releases/download/1.63.2/codium_1.63.2-1639700587_arm64.deb
```

Copy the files and folders to the USB stick.

```bash
rsync -av --exclude-from="exclude-list.txt" /home/ada /home/ada/usb-transfer
```

Unmount the drive before removing it.

```bash
cd; sudo umount usb-transfer
```

## Set up your cold machine.

For the cold machine I would use 64bit Raspberry Pi OS(Raspbian) with a desktop on a Raspi-400. It already has rng-tools by default. We want entropy on the offline machine that is generating all our keys.

A desktop allows for multiple windows, copy and paste and another way to see your keys. It will help you start figuring out the different keys/certificates and what they are used for. Raspberry Pi OS is in my opinion a more stable desktop. Gnome on Ubuntu tends to be a little sluggish and can freeze up at times. Totally fine if you would rather use Ubuntu. Just make sure you have a username(ada) with a UID of 1001 and GID of 1001. Allowing for smooth transfer between machines.

Having a built in keyboard is nice. The only way to get at these keys is physically stealing the drive or through inserting a badusb type root kit which is unlikely but possible. It is one less unknown device that has to be plugged in and you can put the whole thing in a safe quite nicely.
