---
description: Flash image
---

# Raspi-Node Guide

![](../../.gitbook/assets/download-10- (1) (3).jpeg)

## Miksi tämä opas?

This guide is intended for people who wants to get a Raspberry-pi 4 with full desktop Raspberry Pi OS installed along with all the required software to get a Cardano Node up and running on the blockchain. This can be a nice setup for those seeking to just do some lightweight develerpment on the blockchain like making NFTs for example.

{% hint style="Huomaa" %}
**You'll need a monitor (at least for initial setup as SSH is disabled and ufw is up) and you must use the Raspberry Pi 4 with 8GB of RAM!**
{% endhint %}

## Lataa & Polta

### Install Raspi-Imager

Lataa, asenna & avaa [Raspberry Pi Imager](https://github.com/raspberrypi/rpi-imager/releases/latest). Kytke USB-asema laittteeseen.

**64 bit Raspberry Pi OS desktop**

There is now a 64bit image you can install, it is not available in raspi-imager selection, IDK why. Check out the images in the link below grab the latest version. It is a zip file so we have to unzip it before flashing.

{% embed url="https://downloads.raspberrypi.org/raspios_arm64" %}

Unzip the img file and flash it with Raspi-imager. Plug it into your Raspberry Pi 4 and go through the initial setup. Default username=`pi` and the password=`raspberrypi`

You can find documentation here [https://www.raspberrypi.com/documentation/](https://www.raspberrypi.com/documentation/)

Aseta SSD yhteen sinisestä usb3-porteista. Then insert the HDMI, Keyboard, Mouse, Ethernet, and power supply.

{% hint style="Huomaa" %}
The first Pi4's to ship did not boot from USB3 by default, nowadays they do. Jos imagesi ei käynnisty kaksi yleisintä ongelmaa ovat vanhemmat laiteohjelmistot Pi:ssäsi tai yhteensopimaton USB3 sovitin.
{% endhint %}

![](../../.gitbook/assets/pi4.jpeg)

{% hint style="info" %}
All we really need to do here is disable auto-login & create the ada user with sudo privileges. After we log back in we will delete the default Pi user and configure the server & environment for cardano-node & cardano-cli.
{% endhint %}

![Avaa Raspberry Pi Configuration apuohjelma.](../../.gitbook/assets/raspberrypi-configuration.png)

![Aseta automaattinen kirjautuminen pois käytöstä](../../.gitbook/assets/disable-auto-login.png)

### Luo ada käyttäjä

This guide strives to be user agnostic so you can choose a different username and you should be ok. When creating the systemd services however you will have to edit the user. **Pay attention!**

Open a terminal then create a new user and add it to the sudo group.

```bash
sudo adduser ada; sudo adduser ada sudo
```

Update Raspbery Pi OS and reboot the server to make sure you are on the latest kernel. Reboot and log in as your new user.

```bash
sudo apt update; sudo apt upgrade
```

#### Vaihda salasana

You can change the users password at anytime with the following command.

```bash
passwd
```

{% hint style="Huomaa" %}
Ole huolellinen milloin käytät sudo-komentoa. Esimerkiksi 'sudo passwd' muuttaisi root -käyttäjän salasanan. Tämä tuntuu olevan kohta, jossa uudet käyttäjät hämmentyvät helposti.
{% endhint %}

#### Delete the pi user

The pi user is set to auto login and does not require a password for sudo commands. Best to just trash it to avoid any potential security issues.

```bash
sudo deluser --remove-home pi
```

### Server setup

### Konfiguroi Laitteisto

Säästetään hieman energiaa, nostetaan CPU:n hallintaa pikkuisen ja asetetaan GPU:n RAM mahdollisimman alas.

Tässä muutamia linkkejä ylikellotukseen ja aseman nopeuksien testaamiseen. Jos sinulla on lämpöaltaita, voit turvallisesti asettaa 2000. Kiinnitä huomiota yli-voltitus suosituksiin jotta nekin sopivat valitsemaasi kellotusnopeuteen.

* [https://www.raspberrypi.org/documentation/configuration/config-txt/overclocking.md](https://www.raspberrypi.org/documentation/configuration/config-txt/overclocking.md)
* [https://www.seeedstudio.com/blog/2020/02/12/how-to-safely-overclock-your-raspberry-pi-4-to-2-147ghz/](https://www.seeedstudio.com/blog/2020/02/12/how-to-safely-overclock-your-raspberry-pi-4-to-2-147ghz/)
* [https://www.tomshardware.com/how-to/raspberry-pi-4-23-ghz-overclock](https://www.tomshardware.com/how-to/raspberry-pi-4-23-ghz-overclock)
* [https://dopedesi.com/2020/11/24/upgrade-your-raspberry-pi-4-with-a-nvme-boot-drive-by-alex-ellis-nov-2020/](https://dopedesi.com/2020/11/24/upgrade-your-raspberry-pi-4-with-a-nvme-boot-drive-by-alex-ellis-nov-2020/)
* [Legendary Technology: New Raspberry Pi 4 Bootloader USB](https://jamesachambers.com/new-raspberry-pi-4-bootloader-usb-network-boot-guide/)

**Ylikellotus, muisti & radiot**

Edit /boot/config.txt.

```bash
sudo nano /boot/firmware/config.txt
```
Just paste the Pi Pool additions in at the bottom.

```bash
## Pi Pool ##
over_voltage=6
arm_freq=2000
```
use `CTRL + x` to save and `y` to confirm and exit.

Tallenna ja käynnistä uudelleen.

```
sudo reboot
```

### Configure Raspbian

#### Poista root käyttäjä käytöstä

```
sudo passwd -l root
```

#### Suojaa jaettu muisti

Mount shared memory as read only. Avaa /etc/fstab.

```
sudo nano /etc/fstab
```

Lisää seuraava tiedoston loppuun omalle riville, tallenna & sulje nano.

```
tmpfs    /run/shm    tmpfs    ro,noexec,nosuid    0 0
```

#### Increase open file limit for $USER

Add a couple lines to the bottom of /etc/security/limits.conf

```bash
sudo bash -c "echo -e '${USER} soft nofile 800000\n${USER} hard nofile 1048576\n' >> /etc/security/limits.conf"
```

Confirm it was added to the bottom.

```bash
cat /etc/security/limits.conf
```

#### Optimoi suorituskyky & tietoturva

{% hint style="info" %}
[https://gist.github.com/lokhman/cc716d2e2d373dd696b2d9264c0287a3](https://gist.github.com/lokhman/cc716d2e2d373dd696b2d9264c0287a3)
{% endhint %}

{% hint style="Huomaa" %}
If you would like to disable ipv6 or turn on forwarding you can below.
{% endhint %}

Lisää seuraava /etc/sysctl.conf tiedoston loppuun. Tallenna ja sulje.

```bash
sudo nano /etc/sysctl.conf
```

```
## Pi Node ##

fs.file-max = 10000000
fs.nr_open = 10000000

# enable forwarding if using wireguard
net.ipv4.ip_forward=0

# ignore ICMP redirects
net.ipv4.conf.all.send_redirects = 0
net.ipv4.conf.default.send_redirects = 0
net.ipv4.conf.all.accept_redirects = 0
net.ipv4.conf.default.accept_redirects = 0

net.ipv4.icmp_ignore_bogus_error_responses = 1

# disable IPv6
#net.ipv6.conf.all.disable_ipv6 = 1
#net.ipv6.conf.default.disable_ipv6 = 1

# block SYN attacks
net.ipv4.tcp_syncookies = 1
net.ipv4.tcp_max_syn_backlog = 2048
net.ipv4.tcp_synack_retries = 3
net.ipv4.netfilter.ip_conntrack_tcp_timeout_syn_recv=45

# in progress tasks
net.ipv4.tcp_keepalive_time = 240
net.ipv4.tcp_keepalive_intvl = 4
net.ipv4.tcp_keepalive_probes = 5

# reboot if we run out of memory
vm.panic_on_oom = 1
kernel.panic = 10

# Use Google's congestion control algorithm
net.core.default_qdisc = fq
net.ipv4.tcp_congestion_control = bbr
```

**Muutoksemme otetaan käyttöön uudelleen käynnistyksen jälkeen**

Luo uusi tiedosto. Liitä seuraava, tallenna & sulje.

```
sudo nano /etc/rc.local
```

```bash
#
# rc.local
#
# This script is executed at the end of each multiuser runlevel.
# Make sure that the script will "exit 0" on success or any other
# value on error.
#
# In order to enable or disable this script just change the execution
# bits.
#
# By default this script does nothing.

# Print the IP address
_IP=$(hostname -I) || true
if [ "$_IP" ]; then
  printf "My IP address is %s\n" "$_IP"
fi
# Give CPU startup routines time to settle.
sleep 120

sysctl -p /etc/sysctl.conf

exit 0
```

#### Chrony

Meidän täytyy saada aikamme synkronoitua niin tarkasti kuin mahdollista. Avaa /etc/security/chrony.conf

```
sudo apt install chrony
```

```bash
sudo nano /etc/security/chrony.conf
```

Korvaa tiedoston sisältö alla olevalla tekstillä, Tallenna ja poistu.

```bash
pool time.google.com       iburst minpoll 2 maxpoll 2 maxsources 3 maxdelay 0.3
pool time.euro.apple.com   iburst minpoll 2 maxpoll 2 maxsources 3 maxdelay 0.3
pool time.apple.com        iburst minpoll 2 maxpoll 2 maxsources 3 maxdelay 0.3
pool ntp.ubuntu.com        iburst minpoll 2 maxpoll 2 maxsources 3 maxdelay 0.3

# This directive specify the location of the file containing ID/key pairs for
# NTP authentication.
keyfile /etc/chrony/chrony.keys

# This directive specify the file into which chronyd will store the rate
# information.
driftfile /var/lib/chrony/chrony.drift

# Uncomment the following line to turn logging on.
#log tracking measurements statistics

# Log files location.
logdir /var/log/chrony

# Stop bad estimates upsetting machine clock.
maxupdateskew 5.0

# This directive enables kernel synchronisation (every 11 minutes) of the
# real-time clock. Note that it can’t be used along with the 'rtcfile' directive.
rtcsync

# Step the system clock instead of slewing it if the adjustment is larger than
# one second, but only in the first three clock updates.
makestep 0.1 -1

# Get TAI-UTC offset and leap seconds from the system tz database
leapsectz right/UTC

# Serve time even if not synchronized to a time source.
local stratum 10
```

Tallenna & poistu.

```bash
sudo service chrony restart
```

#### Zram swap

{% hint style="info" %}
Olemme havainneet, että kardano-node voi turvallisesti käyttää tätä pakattua swapia RAM:ina periaatteessatämä antaa meille noin 20 gb RAM:ia. Olemme jo asettaneet ytimen parametrit zram:ia varten /etc/sysctl.conf tiedostossa
{% endhint %}

Swapping to disk is slow, swapping to compressed ram space is faster and gives us some overhead before out of memory (oom).

{% embed url="https://haydenjames.io/raspberry-pi-performance-add-zram-kernel-parameters/" %}

{% embed url="https://lists.ubuntu.com/archives/lubuntu-users/2013-October/005831.html" %}

Poista Raspbian swapfile käytöstä.

```
sudo systemctl disable dphys-swapfile.service
```

```
sudo apt install zram-config
```

```bash
sudo nano /etc/default/zramswap
```

```bash
# Compression algorithm selection
# speed: lz4 > zstd > lzo
# compression: zstd > lzo > lz4
# This is not inclusive of all that is available in latest kernels
# See /sys/block/zram0/comp_algorithm (when zram module is loaded) to see
# what is currently set and available for your kernel[1]
# [1]  https://github.com/torvalds/linux/blob/master/Documentation/blockdev/zram.txt#L86
#ALGO=lz4

# Specifies the amount of RAM that should be used for zram
# based on a percentage the total amount of available memory
# This takes precedence and overrides SIZE below
PERCENT=150

# Specifies a static amount of RAM that should be used for
# the ZRAM devices, this is in MiB
#SIZE=256

# Specifies the priority for the swap devices, see swapon(2)
# for more details. Higher number = higher priority
# This should probably be higher than hdd/ssd swaps.
#PRIORITY=100
```

Tallenna ja käynnistä uudelleen.

```bash
sudo reboot
```

### Asenna paketit

Asennetaan tarvittavat paketit.

```bash
sudo apt install build-essential libssl-dev tcptraceroute python3-pip \
         make automake unzip net-tools nginx ssl-cert pkg-config jq \
         libffi-dev libgmp-dev libssl-dev libtinfo-dev libsystemd-dev \
         zlib1g-dev g++ libncursesw5 libtool autoconf unattended-upgrades -y
```

```
sudo reboot
```

### Automatic security updates

Enable automatic security updates.

```bash
sudo dpkg-reconfigure -plow unattended-upgrades
```

## Ympäristön Asetukset

***

### description: Configure the environment for Cardano Node

### Choose testnet or mainnet.

{% hint style="danger" %}
There is a 500 ₳ Registration deposit and another 5 ₳ in registration costs to start a pool on mainnet. First time users are strongly reccomended to use testnet. You can get tada (test ada) from the testnet faucet. [tada faucet link](https://testnets.cardano.org/en/testnets/cardano/tools/faucet/)
{% endhint %}

Create the directories for our project.

```bash
mkdir -p ${HOME}/.local/bin
mkdir -p ${HOME}/pi-pool/files
mkdir -p ${HOME}/pi-pool/scripts
mkdir -p ${HOME}/pi-pool/logs
mkdir ${HOME}/git
mkdir ${HOME}/tmp
```

Create an .adaenv file, choose which network you want to be on and source the file. This file will hold the variables/settings for operating a Pi-Node. /home/ada/.adaenv

```shell
echo -e NODE_CONFIG=testnet >> ${HOME}/.adaenv; source ${HOME}/.adaenv
```

#### Create bash variables & add \~/.local/bin to our $PATH 🏃

{% hint style="info" %}
[Ympäristömuuttujat Linux/Unix](https://askubuntu.com/questions/247738/why-is-etc-profile-not-invoked-for-non-login-shells/247769#247769).
{% endhint %}

{% hint style="Huomaa" %}
You must reload environment files after updating them. Same goes for cardano-node, changes to the topology or config files require a cardano-service restart.
{% endhint %}

```bash
echo . ~/.adaenv >> ${HOME}/.bashrc
cd .local/bin; echo "export PATH=\"$PWD:\$PATH\"" >> $HOME/.adaenv
echo export NODE_HOME=${HOME}/pi-pool >> ${HOME}/.adaenv
echo export NODE_PORT=3003 >> ${HOME}/.adaenv
echo export NODE_FILES=${HOME}/pi-pool/files >> ${HOME}/.adaenv
echo export TOPOLOGY='${NODE_FILES}'/'${NODE_CONFIG}'-topology.json >> ${HOME}/.adaenv
echo export DB_PATH='${NODE_HOME}'/db >> ${HOME}/.adaenv
echo export CONFIG='${NODE_FILES}'/'${NODE_CONFIG}'-config.json >> ${HOME}/.adaenv
echo export NODE_BUILD_NUM=$(curl https://hydra.iohk.io/job/Cardano/iohk-nix/cardano-deployment/latest-finished/download/1/index.html | grep -e "build" | sed 's/.*build\/\([0-9]*\)\/download.*/\1/g') >> ${HOME}/.adaenv
echo export CARDANO_NODE_SOCKET_PATH="${HOME}/pi-pool/db/socket" >> ${HOME}/.adaenv
source ${HOME}/.bashrc; source ${HOME}/.adaenv
```

### Build Libsodium

This is IOHK's fork of Libsodium. It is needed for the dynamic build binary of cardano-node.

```bash
cd; cd git/
git clone https://github.com/input-output-hk/libsodium
cd libsodium
git checkout 66f017f1
./autogen.sh
./configure
make
sudo make install
```

Echo library paths .bashrc file and source it.

```bash
echo "export LD_LIBRARY_PATH="/usr/local/lib:$LD_LIBRARY_PATH"" >> ~/.bashrc
echo "export PKG_CONFIG_PATH="/usr/local/lib/pkgconfig:$PKG_CONFIG_PATH"" >> ~/.bashrc
. ~/.bashrc
```

Update link cache for shared libraries and confirm.

```bash
sudo ldconfig; ldconfig -p | grep libsodium
```

#### Nouda palvelintiedostot

```bash
cd $NODE_FILES
wget -N https://hydra.iohk.io/build/${NODE_BUILD_NUM}/download/1/${NODE_CONFIG}-config.json
wget -N https://hydra.iohk.io/build/${NODE_BUILD_NUM}/download/1/${NODE_CONFIG}-byron-genesis.json
wget -N https://hydra.iohk.io/build/${NODE_BUILD_NUM}/download/1/${NODE_CONFIG}-shelley-genesis.json
wget -N https://hydra.iohk.io/build/${NODE_BUILD_NUM}/download/1/${NODE_CONFIG}-alonzo-genesis.json
wget -N https://hydra.iohk.io/build/${NODE_BUILD_NUM}/download/1/${NODE_CONFIG}-topology.json
wget -N https://raw.githubusercontent.com/input-output-hk/cardano-node/master/cardano-submit-api/config/tx-submit-mainnet-config.yaml
```

Run the following to modify ${NODE\_CONFIG}-config.json and update TraceBlockFetchDecisions to "true" & listen on all interfaces with Prometheus Node Exporter.

```bash
sed -i ${NODE_CONFIG}-config.json \
    -e "s/TraceBlockFetchDecisions\": false/TraceBlockFetchDecisions\": true/g" \
    -e "s/127.0.0.1/0.0.0.0/g"
```

{% hint style="info" %}
**Tip for relay nodes**: It's possible to reduce memory and cpu usage by setting "TraceMemPool" to "false" in **{NODE\_CONFIG}-config.json.** This will turn off mempool data in Grafana and gLiveView.sh.
{% endhint %}

#### Retrieve aarch64 1.33.1 and cardano-submit-api binaries

{% hint style="info" %}
The **unofficial** cardano-node, cardano-cli and cardano-submit-api binaries available to us are being built by an IOHK engineer in his **spare time**. Consider delegating to zw3rk pool to support mobile Haskel development.
{% endhint %}

```bash
cd ${HOME}/tmp
wget https://ci.zw3rk.com/build/430108/download/1/aarch64-unknown-linux-musl-cardano-node-1.33.1.zip
unzip *.zip
mv cardano-node/cardano-* ${HOME}/.local/bin
rm -r *
cd ${HOME}
```

{% hint style="Huomaa" %}
If binaries already exist (if updating) you will have to confirm overwriting the old ones.
{% endhint %}

Confirm binaries are in $USER's $PATH.

```bash
cardano-node version
cardano-cli version
which cardano-submit-api
```

#### Systemd unit startup scripts

Create the systemd unit file and startup script so systemd can manage cardano-node.

```bash
nano ${HOME}/.local/bin/cardano-service
```

Liitä seuraavat, tallenna & sulje nano.

```bash
#!/bin/bash
. /home/ada/.adaenv

## +RTS -N4 -RTS = Multicore(4)
cardano-node run +RTS -N4 -RTS \
  --topology ${TOPOLOGY} \
  --database-path ${DB_PATH} \
  --socket-path ${CARDANO_NODE_SOCKET_PATH} \
  --port ${NODE_PORT} \
  --config ${CONFIG}
```

Allow execution of our new cardano-node service file.

```bash
chmod +x ${HOME}/.local/bin/cardano-service
```

Avaa /etc/systemd/system/cardano-node.service.

```bash
sudo nano /etc/systemd/system/cardano-node.service
```

Paste the following, You will need to edit the username here if you chose to not use ada. Tallenna & poistu.

```bash
# The Cardano Node Service (part of systemd)
# file: /etc/systemd/system/cardano-node.service

[Unit]
Description     = Cardano node service
Wants           = network-online.target
After           = network-online.target

[Service]
User            = ada
Type            = simple
WorkingDirectory= /home/ada/pi-pool
ExecStart       = /bin/bash -c "PATH=/home/ada/.local/bin:$PATH exec /home/ada/.local/bin/cardano-service"
KillSignal=SIGINT
RestartKillSignal=SIGINT
TimeoutStopSec=10
LimitNOFILE=32768
Restart=always
RestartSec=10
EnvironmentFile=-/home/ada/.adaenv

[Install]
WantedBy= multi-user.target
```

Create the systemd unit file and startup script so systemd can manage cardano-submit-api.

```bash
nano ${HOME}/.local/bin/cardano-submit-service
```

```bash
#!/bin/bash
. /home/ada/.adaenv

cardano-submit-api \
  --socket-path ${CARDANO_NODE_SOCKET_PATH} \
  --port 8090 \
  --config /home/ada/pi-pool/files/tx-submit-mainnet-config.yaml \
  --listen-address 0.0.0.0 \
  --mainnet
```

Allow execution of our new cardano-submit-api service script.

```bash
chmod +x ${HOME}/.local/bin/cardano-submit-service
```

Create /etc/systemd/system/cardano-submit.service.

```bash
sudo nano /etc/systemd/system/cardano-submit.service
```

Paste the following, You will need to edit the username here if you chose to not use ada. save & exit.

```bash
# The Cardano Submit Service (part of systemd)
# file: /etc/systemd/system/cardano-submit.service

[Unit]
Description     = Cardano submit service
Wants           = network-online.target
After           = network-online.target

[Service]
User            = ada
Type            = simple
WorkingDirectory= /home/ada/pi-pool
ExecStart       = /bin/bash -c "PATH=/home/ada/.local/bin:$PATH exec /home/ada/.local/bin/cardano-submit-service"
KillSignal=SIGINT
RestartKillSignal=SIGINT
TimeoutStopSec=10
LimitNOFILE=32768
Restart=always
RestartSec=10
EnvironmentFile=-/home/ada/.adaenv

[Install]
WantedBy= multi-user.target
```

Reload systemd so it picks up our new service files.

```bash
sudo systemctl daemon-reload
```

Let's add a couple functions to the bottom of our .adaenv file to make life a little easier.

```bash
nano ${HOME}/.adaenv
```

```bash
cardano-service() {
    #do things with parameters like $1 such as
    sudo systemctl "$1" cardano-node.service
}

cardano-submit() {
    #do things with parameters like $1 such as
    sudo systemctl "$1" cardano-submit.service
}
```

Tallenna & poistu.

```bash
source ${HOME}/.adaenv
```

What we just did there was add a couple functions to control our cardano-service and cardano-submit without having to type out

> > sudo systemctl enable cardano-node.service sudo systemctl start cardano-node.service sudo systemctl stop cardano-node.service sudo systemctl status cardano-node.service

Nyt meidän täytyy vain:

* cardano-service enable (enables cardano-node.service auto start at boot)
* cardano-service start (starts cardano-node.service)
* cardano-service stop (stops cardano-node.service)
* cardano-service status (shows the status of cardano-node.service)

Or

* cardano-submit enable (enables cardano-submit.service auto start at boot)
* cardano-submit start (starts cardano-submit.service)
* cardano-submit stop (stops cardano-submit.service)
* cardano-submit status (shows the status of cardano-submit.service)

The submit service listens on port 8090. You can connect your Nami wallet like below to submit tx's yourself in Nami's settings.

```bash
http://<node lan ip>:8090/api/submit/tx
```

### ⛓ Ketjun synkronointi ⛓

Olet nyt valmis käynnistämään cardano-noden. Käynnistäminen aloittaa oman nodesi synkronoinnin Cardano lohkoketjun kanssa. This is going to take about 48 hours and the db folder is about 13GB in size right now. Aiemmin ensimmäinen node tuli synkronoida kokonaan, alusta loppuun jonka jälkeen tietokanta voitiin kopioida toiseen nodeen. However...

#### Lataa tilannekuva

Olen alkanut ottaa tilannekuvia oman vara noden tietokanta kansiosta ja se on saatavilla web-hakemistosta. With this service it takes around 20 minutes to pull the latest snapshot and maybe another hour to sync up to the tip of the chain. Palvelu tarjotaan sellaisenaan. Valinta on sinun. Jos haluat synkronoida ketjun omin avuin, yksinkertaisesti:

```bash
cardano-service enable
cardano-service start
cardano-service status
```

Otherwise, be sure your node is **not** running & delete the db folder if it exists and download db/.

```bash
cardano-service stop
cd $NODE_HOME
rm -r db/
```

**Download Database**

```bash
wget -r -np -nH -R "index.html*" -e robots=off https://$NODE_CONFIG.adamantium.online/db/
```

Kun wget valmistuu, ota käyttöön cardano-node & käynnistä se.

```bash
cardano-service enable
cardano-service start
cardano-service status
```

### gLiveView.sh

Guild operators scripts has a couple useful tools for operating a pool. We do not want the project as a whole, though there are a couple scripts we are going to use.

{% embed url="https://github.com/cardano-community/guild-operators/tree/master/scripts/cnode-helper-scripts" %}

```bash
cd $NODE_HOME/scripts
wget https://raw.githubusercontent.com/cardano-community/guild-operators/master/scripts/cnode-helper-scripts/env
wget https://raw.githubusercontent.com/cardano-community/guild-operators/master/scripts/cnode-helper-scripts/gLiveView.sh
```

{% hint style="info" %}
You can change the port cardano-node runs on in the .adaenv file in your home directory. Open the file edit the port number. Load the change into your shell & restart the cardano-node service.

```bash
nano /home/ada/.adaenv
source /home/ada/.adaenv
cardano-service restart
```
{% endhint %}

Add a line sourcing our .adaenv file to the top of the env file and adjust some paths.

```bash
sed -i env \
    -e "/#CNODEBIN/i. ${HOME}/.adaenv" \
    -e "s/\#CNODE_HOME=\"\/opt\/cardano\/cnode\"/CNODE_HOME=\"\${HOME}\/pi-pool\"/g" \
    -e "s/\#CNODE_PORT=6000"/CNODE_PORT=\"'${NODE_PORT}'\""/g" \
    -e "s/\#CONFIG=\"\${CNODE_HOME}\/files\/config.json\"/CONFIG=\"\${NODE_FILES}\/"'${NODE_CONFIG}'"-config.json\"/g" \
    -e "s/\#TOPOLOGY=\"\${CNODE_HOME}\/files\/topology.json\"/TOPOLOGY=\"\${NODE_FILES}\/"'${NODE_CONFIG}'"-topology.json\"/g" \
    -e "s/\#LOG_DIR=\"\${CNODE_HOME}\/logs\"/LOG_DIR=\"\${CNODE_HOME}\/logs\"/g"
```

Salli gLiveView.sh:n suorittaminen.

```bash
chmod +x gLiveView.sh
```

### topologyUpdater.sh

Kunnes vertaisverkko on otettu käyttöön verkko-operaattorit tarvitsevat tavan saada listan releistä/vertaisverkoista, joihin muodostaa yhteyden. Topologian päivityspalvelu toimii taustalla cron kanssa. Joka tunti skripti toimii ja kertoo palvelulle, että olet relay ja haluat olla osa verkkoa. It will add your relay to it's directory after four hours you should see in connections in gLiveView.

{% hint style="info" %}
The list generated will show you the distance & a clue as to where the relay is located.
{% endhint %}

Download the topologyUpdater script and have a look at it. Lower the number of peers to 10 and add any custom peers you wish. These are outgoing connections. You will not see any incoming transactions untill other nodes start connecting to you.

```bash
wget https://raw.githubusercontent.com/cardano-community/guild-operators/master/scripts/cnode-helper-scripts/topologyUpdater.sh
```

Lower the number of MX\_PEERS to 10.

```bash
nano topologyUpdater.sh
```

Tallenna, sulje ja tee se suoritettavaksi.

```bash
chmod +x topologyUpdater.sh
```

{% hint style="Huomaa" %}
Et pysty suorittamaan ./topologyUpdater.sh onnistuneesti ennen kuin nodesi on täysin synkronoitu ketjun kärkeen.
{% endhint %}

Luo cron työ, joka suorittaa skriptin tunnin välein.

```bash
crontab -e
```

{% hint style="info" %}
Valitse nano pyydettäessä editoria.
{% endhint %}

Lisää seuraava tiedoston loppuun omalle riville, tallenna & sulje nano.

{% hint style="info" %}
Pi-node-imagessassa tämä cron merkintä on oletuksena pois päältä. You can enable it by removing the #.
{% endhint %}

```bash
SHELL=/bin/bash
PATH=/home/ada/.local/bin:/usr/bin:/bin:/usr/sbin:/sbin:/usr/local/bin
33 * * * * . $HOME/.adaenv; $HOME/pi-pool/scripts/topologyUpdater.sh
```

After four hours you can open ${NODE\_CONFIG}-topology.json and inspect the list of out peers the service suggested for you. Remove anything more than 7k distance(or less). IOHK recently suggested 8 out peers. The more out peers the more system resources it uses. You can also add any peers you wish to connect to manualy inside the script. This is where you would add your block producer or any friends nodes.

```bash
nano $NODE_FILES/${NODE_CONFIG}-topology.json
```

{% hint style="info" %}
You can use gLiveView.sh to view ping times in relation to the peers in your {NODE\_CONFIG}-topology file. Käytä Ping:ää palvelimien nimien selvittämiseen IP-osoitteissa.
{% endhint %}

Muutokset tässä tiedostossa tulevat käyttöön vasta kun cardano-service käynnistetään uudelleen.

{% hint style="Huomaa" %}
Älä unohda poistaa viimeistä pilkkua topologiatiedostosta!
{% endhint %}

Tilan tulisi näyttää enabled & running.

Once your node syncs past epoch 208(shelley era) you can use gLiveView.sh to monitor your sync progress.

{% hint style="danger" %}
It can take over an hour for cardano-node to sync to the tip of the chain. Käytä ./gliveView.sh, htop ja log tietoja tarkastellaksesi prosessia. Olipa kärsivällinen, kärki saavutetaan kyllä.
{% endhint %}

```bash
cd $NODE_HOME/scripts
./gLiveView.sh
```

![](../../.gitbook/assets/pi-node-glive (7).png)

### Prometheus, Node Exporter & Grafana

Prometheus yhdistää cardano-noden backendiin ja lähettää metriikkaa http:n kautta. Grafana puolestaan voi käyttää näitä tietoja kaavioiden näyttämiseen ja hälytysten luomiseen. Meidän Grafana kojelautamme koostuu Ubuntu järjestelmän & cardano-noden datasta. Grafana can display data from other sources as well, like [adapools.org](https://adapools.org).

{% hint style="info" %}
You can connect a [Telegram bot](https://docs.armada-alliance.com/learn/intermediate-guide/grafana-alerts-with-telegram) to Grafana which can alert you of problems with the server. Tämä on paljon helpompaa kuin yrittää määritellä sähköpostihälytyksiä.
{% endhint %}

{% embed url="https://github.com/prometheus" %}

![](../../.gitbook/assets/pi-pool-grafana (2) (2) (2) (2) (1) (7).png)

#### Asenna Prometheus & Node Exporter.

{% hint style="info" %}
Prometheus pystyy onkimaan myös muiden node exporteria käyttävien palvelimien http päätetapahtumat. Tämä tarkoittaa, että Grafanaa ja Prometheusta ei tarvitse asentaa ydin tai relay nodeesi. Only the package prometheus-node-exporter is required if you would like to build a central Grafana dashboard for the pool, freeing up resources and having a single dashboard to monitor everything.
{% endhint %}

```bash
sudo apt install prometheus prometheus-node-exporter -y
```

Poista ne systemd:n käytöstä toistaiseksi.

```bash
sudo systemctl disable prometheus.service
sudo systemctl disable prometheus-node-exporter.service
```

#### Määritä Prometheus

Avaa prometheus.yml.

```bash
sudo nano /etc/prometheus/prometheus.yml
```

Korvaa tiedoston sisältö alla olevan kanssa.

{% hint style="Huomaa" %}
Sisennyksen on oltava oikea YAML muoto tai Prometheus ei käynnisty.
{% endhint %}

```yaml
global:
  scrape_interval: 15s # By default, scrape targets every 15 seconds.

  # Attach these labels to any time series or alerts when communicating with
  # external systems (federation, remote storage, Alertmanager).
  external_labels:
    monitor: "codelab-monitor"

# A scrape configuration containing exactly one endpoint to scrape:
# Here it's Prometheus itself.
scrape_configs:
  # The job name is added as a label job=<job_name> to any timeseries scraped from this config.
  - job_name: "Prometheus" # To scrape data from Prometheus Node Exporter
    scrape_interval: 5s
    static_configs:
      #      - targets: ['<CORE PRIVATE IP>:12798']
      #        labels:
      #          alias: 'C1'
      #          type:  'cardano-node'
      #      - targets: ['<RELAY PRIVATE IP>:12798']
      #        labels:
      #          alias: 'R1'
      #          type:  'cardano-node'
      - targets: ["localhost:12798"]
        labels:
          alias: "N1"
          type: "cardano-node"

      #      - targets: ['<CORE PRIVATE IP>:9100']
      #        labels:
      #          alias: 'C1'
      #          type:  'node'
      #      - targets: ['<RELAY PRIVATE IP>:9100']
      #        labels:
      #          alias: 'R1'
      #          type:  'node'
      - targets: ["localhost:9100"]
        labels:
          alias: "N1"
          type: "node"
```

Tallenna & poistu.

Start Prometheus.

```bash
sudo systemctl start prometheus.service
```

#### Asenna Grafana

{% embed url="https://github.com/grafana/grafana" %}

Lisää Grafanan gpg avain Ubuntuun.

```bash
wget -q -O - https://packages.grafana.com/gpg.key | sudo apt-key add -
```

Lisää uusin vakaa repo apt lähteisiin.

```bash
echo "deb https://packages.grafana.com/oss/deb stable main" | sudo tee -a /etc/apt/sources.list.d/grafana.list
```

Päivitä pakettilistat & asenna Grafana.

```bash
sudo apt update; sudo apt install grafana
```

Muuta portti jota Grafana kuuntelee, jotta se ei ole ristiriidassa cardano-noden kanssa.

```bash
sudo sed -i /etc/grafana/grafana.ini \
         -e "s#;http_port#http_port#" \
         -e "s#3000#5000#"
```

Start Grafana

```bash
sudo systemctl start grafana-server.service
```

#### cardano-monitor bash-toiminto

Open .adaenv.

```bash
cd ${HOME}; nano .adaenv
```

Lisää tiedoston loppuun:

```bash
cardano-monitor() {
    #do things with parameters like $1 such as
    sudo systemctl "$1" prometheus.service
    sudo systemctl "$1" prometheus-node-exporter.service
    sudo systemctl "$1" grafana-server.service
}
```

Tallenna, poistu & source.

```bash
source .adaenv
```

Täällä yhdistimme kaikki kolme palvelua yhteen tehtävään. Ota Prometheus.service, prometheus-node-exporter.service & grafana-server.service käyttöön käynnistyksen yhteydessä ja käynnistä palvelut.

```bash
cardano-monitor enable
cardano-monitor start
```

{% hint style="Huomaa" %}
Tässä vaiheessa saatat haluta käynnistää cardano-servicen ja synkronoida nodesi lohkoketjun kanssa ennen kuin jatkamme Grafanan konfigurointia. Go to the syncing the chain section. Choose whether you want to wait 30 hours or download the latest chain snapshot. Palaa tähän kun gLiveView.sh näyttää, että olet ketjun kärjessä.
{% endhint %}

### Grafana, Nginx proxy\_pass & snakeoil

Let's put Grafana behind Nginx with self signed(snakeoil) certificate. Sertifikaatti luotiin, kun asensimme ssl-cert paketin.

Voit saada varoituksen selaimestasi. This is because ca-certificates cannot follow a trust chain to a trusted (centralized) source. Yhteys on kuitenkin salattu, ja se suojaa salasanojasi, jotka liitelevät bittiavaruudessa pelkkänä tekstinä.

```bash
sudo nano /etc/nginx/sites-available/default
```

Korvaa tiedoston sisältö alla olevan kanssa.

```bash
# Default server configuration
#
server {
        listen 80 default_server;
        return 301 https://$host$request_uri;
}

server {
        # SSL configuration
        #
        listen 443 ssl default_server;
        #listen [::]:443 ssl default_server;
        #
        # Note: You should disable gzip for SSL traffic.
        # Katso: https://bugs.debian.org/773332
        #
        # Lukea ssl_ciphers varmistaaksesi turvallisen konfiguraation.
        # Katso: https://bugs.debian.org/765782
        #
        # Itse allekirjoitetut sertit luotu ssl-cert paketti
        # Älä käytä niitä tuotantopalvelimella!
        #
        include snippets/snakeoil.conf;

        add_header X-Proxy-Cache $upstream_cache_status;
        location / {
          proxy_pass http://127.0.0.1:5000;
          proxy_redirect      off;
          include proxy_params;
        }
}
```

Tarkista, että Nginx on tyytyväinen muutoksiimme ja käynnistä se uudelleen.

```bash
sudo nginx -t
## if ok, do
sudo service nginx restart
```

You can now visit your pi-nodes ip address without any port specification, the connection will be upgraded to SSL/TLS and you will get a scary message(not really scary at all). Jatka kohti kojelautaasi.

![](../../.gitbook/assets/snakeoil.png)

#### Määritä Grafana

On your local machine open your browser and enter your nodes private ip address.

Kirjaudu sisään ja aseta uusi salasana. Oletus käyttäjätunnus ja salasana on **admin:admin**.

**Määritä tietolähde**

In the left hand vertical menu go to **Configure** > **Datasources** and click to **Add data source**. Valitse Prometheus. Enter [http://localhost:9090](http://localhost:9090) where it is grayed out, everything else can be left default. Alareunassa save & test. Sinun pitäisi saada vihreä "Data source is working", jos kardano-monitor on päällä. Jos jostain syystä nämä palvelut eivät käynnistyneet, käytä komentoa **cardano-service restart**.

**Tuo kojelaudat**

Tallenna kojelaudan json tiedostot paikalliseen koneeseen.

{% embed url="https://github.com/armada-alliance/dashboards" %}

In the left hand vertical menu go to **Dashboards** > **Manage** and click on **Import**. Valitse tiedosto, jonka juuri latasit tai loit ja tallenna. Head back to **Dashboards** > **Manage** and click on your new dashboard.

![](../../.gitbook/assets/pi-pool-grafana (2) (2) (2) (2) (1) (5).png)

#### Määritä poolDataLive

Here you can use the poolData api to bring extra pool data into Grafana like stake & price.

{% embed url="https://api.pooldata.live/dashboard" %}

Noudata ohjeita asentaaksesi Grafana plugin, määritä datasource ja tuo dashboard.

```bash
sudo grafana-cli plugins install simpod-json-datasource
cardano-monitor restart
```

### Useful Commands

{% hint style="info" %}
Katso, kuinka paljon zram swap:ia cardano-node käyttää.

```bash
CNZRAM=$(pidof cardano-node)
grep --color VmSwap /proc/$CNZRAM/status
```

Seuraa lokin ulostuloa päiväkirjaan.

```bash
sudo journalctl --unit=cardano-node --follow
```

Seuraa lokin ulostuloa stdoutiin.

```bash
sudo tail -f /var/log/syslog
```

View network connections with netstat.

```bash
sudo netstat -puntw
```
{% endhint %}

From here you have a Pi-Node with tools to build an active relay or a stake pool from the following pages. Best of luck and please join the [armada-alliance](https://armada-alliance.com), together we are stronger! :muscle:
