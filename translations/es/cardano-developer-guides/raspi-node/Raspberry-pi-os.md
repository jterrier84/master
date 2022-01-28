---
description: Flash image
---

You must use the Raspberry Pi 4 with 8gb of ram!

# Descargar y Flashear

## Install Raspi-Imager

Descarga, instala y abre [ Raspberry pi Imager ](https://github.com/raspberrypi/rpi-imager/releases/latest). Conecte su unidad USB de destino.

#### 64 bit Raspberry Pi OS desktop

There is now a 64bit image you can install, it is not available in raspi-imager selection, IDK why. Check out the images in the link below grab the latest version. It is a zip file so we have to unzip it before flashing.

{% embed url="https://downloads.raspberrypi.org/raspios_arm64" %}

Unzip the img file and flash it with Raspi-imager. Plug it into your Raspberry Pi 4 and go through the initial setup. Default username=pi and the password=raspberrypi.

You can find documentation here [https://www.raspberrypi.com/documentation/](https://www.raspberrypi.com/documentation/)

Inserta la SSD en uno de los puertos azules de usb3. Luego inserta el HDMI, Keyboard, Mouse, Ethernet y fuente de energía.

{% hint style="danger" %}
Los primeros Pi4's a enviar no arrancan desde USB3 por defecto, hoy en día ya lo hacen. Si tu imagen no arranca los dos problemas más comunes son que el firmware es antiguo en tu Raspberry Pi o un adaptador USB3 incompatible.
{% endhint %}

![](../../.gitbook/assets/pi4.jpeg)

{% hint style="info" %}
Todo lo que realmente necesitamos hacer aquí es desactivar el inicio de sesión automático & crear el usuario ada con privilegios sudo. Después de volver a iniciar sesión, eliminaremos el usuario predeterminado de Pi y configuraremos el servidor & entorno para el nodo cardano & cardano-cli.
{% endhint %}

![Abra la utilidad de configuración de Raspberry Pi.](../../.gitbook/assets/raspberrypi-configuration.png)

![Establecer inicio automático deshabilitado](../../.gitbook/assets/disable-auto-login.png)

## Create the ada user

This guide strives to be user agnostic so you can choose a different username and you should be ok. When creating the systemd services whoever you will have to edit the user. Pay attention!

Open terminal and create a new user and add it to the sudo group.

```bash
sudo adduser ada; sudo adduser ada sudo
```
Update Raspbery Pi OS and reboot the server to make sure you are on the latest kernel. Reboot and login as your new user.

```bash
sudo apt update; sudo apt upgrade
```

### Change password

You can change the users password at anytime with.

```bash
passwd
```

{% hint style="warning" %}
Careful where you use sudo. For example issuing 'sudo passwd' would change the root password. This seems to be a place where new users get confused.
{% endhint %}

### Delete the pi user

The pi user is set to auto login and does not require a password for sudo commands. Best to just trash it to avoid any potential security issues.

```bash
sudo deluser --remove-home pi
```

## Server setup

## Configure Hardware

Let's save some power, raise the governor on the CPU a bit, and set GPU ram as low as we can.

{% hint style="warning" %}
Here are some links for overclocking and testing your drive speeds. If you have heat sinks you can safely go to 2000. Just pay attention to over volt recommendations to go with your chosen clock speed.

- [https://www.raspberrypi.org/documentation/configuration/config-txt/overclocking.md](https://www.raspberrypi.org/documentation/configuration/config-txt/overclocking.md)
- [https://www.seeedstudio.com/blog/2020/02/12/how-to-safely-overclock-your-raspberry-pi-4-to-2-147ghz/](https://www.seeedstudio.com/blog/2020/02/12/how-to-safely-overclock-your-raspberry-pi-4-to-2-147ghz/)
- [https://www.tomshardware.com/how-to/raspberry-pi-4-23-ghz-overclock](https://www.tomshardware.com/how-to/raspberry-pi-4-23-ghz-overclock)
- [https://dopedesi.com/2020/11/24/upgrade-your-raspberry-pi-4-with-a-nvme-boot-drive-by-alex-ellis-nov-2020/](https://dopedesi.com/2020/11/24/upgrade-your-raspberry-pi-4-with-a-nvme-boot-drive-by-alex-ellis-nov-2020/)
- [Legendary Technology: New Raspberry Pi 4 Bootloader USB](https://jamesachambers.com/new-raspberry-pi-4-bootloader-usb-network-boot-guide/)

#### Overclock, memoria & radios

Edit /boot/config.txt. Simplemente pegue las adiciones de Pi Pool en la parte inferior. CTRL+x to save and y to confirm and exit.

```bash
sudo nano /boot/config.txt
```

```bash
## Pi Pool ##
over_voltage=6
arm_freq=2000
```

Save and reboot.

```text
sudo reboot
```

## Configure Ubuntu

### Deshabilitar el usuario root

```
sudo passwd -l root
```

### Securizar la memoria compartida

Mount shared memory as read only. Abrir /etc/fstab.

```
sudo nano /etc/fstab
```

Add this line at the bottom, save & exit.

```
tmpfs    /run/shm    tmpfs    ro,noexec,nosuid    0 0
```

### Increase open file limit for $USER

Add a couple lines to the bottom of /etc/security/limits.conf

```bash
sudo bash -c "echo -e '${USER} soft nofile 800000\n${USER} hard nofile 1048576\n' >> /etc/security/limits.conf"
```

Confirm it was added to the bottom.

```bash
cat /etc/security/limits.conf
```

### Optimizar rendimiento y seguridad

{% hint style="info" %}
[https://gist.github.com/lokhman/cc716d2e2d373dd696b2d9264c0287a3](https://gist.github.com/lokhman/cc716d2e2d373dd696b2d9264c0287a3)
{% endhint %}

{% hint style="warning" %}
If you would like to disable ipv6 or turn on forwarding you can below.
{% endhint %}

Agregue lo siguiente en la parte inferior de /etc/sysctl.conf . Save and exit.

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

#### Cargará nuestros cambios después del inicio

Crea un nuevo Archivo Pega, guarda y cierra.

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

### Chrony

Necesitamos que nuestra sincronización de la hora sea lo más precisa posible. Abre/chrony/chrony.conf

```
sudo apt install chrony
```

```bash
sudo nano /etc/chrony/chrony.conf
```

Reemplaza el contenido del archivo con el de abajo, Guardar y salir.

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
Guardar y salir

```bash
sudo service chrony restart
```
### Zram swap

{% hint style="info" %}
Hemos descubierto que el nodo de cardano puede usar de forma segura esta swap comprimida de Ram dándonos, básicamente, alrededor de 20gb de Ram. Ya establecimos los parámetros del kernel para zram en /etc/sysctl.conf
{% endhint %}

Swapear a disco es lento, swapear a espacio comprimido de ram es más rápido y nos da algo de tiempo antes de llenar la memoria \(oom\).

{% embed url="https://haydenjames.io/raspberry-pi-performance-add-zram-kernel-parameters/" caption="" %}

{% embed url="https://lists.ubuntu.com/archives/lubuntu-users/2013-October/005831.html" caption="" %}

Deshabilitar el archivo de intercambio (swapfile) de Raspbian

```text
sudo systemctl disable dphys-swapfile.service
```

```text
sudo apt install zram-tools
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

Save and reboot.

```bash
sudo reboot
```

## Instalar paquetes

Instala los paquetes que necesitaremos.

```bash
sudo apt install build-essential libssl-dev tcptraceroute python3-pip \
         make automake unzip net-tools nginx ssl-cert pkg-config jq \
         libffi-dev libgmp-dev libssl-dev libtinfo-dev libsystemd-dev \
         zlib1g-dev g++ libncursesw5 libtool autoconf unattended-upgrades -y
```

```
sudo reboot
```
## Automatic security updates

Enable automatic security updates.

```bash
sudo dpkg-reconfigure -plow unattended-upgrades
```

# Configuración del entorno

---
description: Configure the environment for Cardano Node
---

## Choose testnet or mainnet.

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

### Create bash variables & add \~/.local/bin to our $PATH 🏃

{% hint style="info" %}
[Variables de entorno en Linux/Unix](https://askubuntu.com/questions/247738/why-is-etc-profile-not-invoked-for-non-login-shells/247769#247769).
{% endhint %}

{% hint style="warning" %}
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

## Build Libsodium

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

### Recuperar archivos del nodo

```bash
cd $NODE_FILES
wget -N https://hydra.iohk.io/build/${NODE_BUILD_NUM}/download/1/${NODE_CONFIG}-config.json
wget -N https://hydra.iohk.io/build/${NODE_BUILD_NUM}/download/1/${NODE_CONFIG}-byron-genesis.json
wget -N https://hydra.iohk.io/build/${NODE_BUILD_NUM}/download/1/${NODE_CONFIG}-shelley-genesis.json
wget -N https://hydra.iohk.io/build/${NODE_BUILD_NUM}/download/1/${NODE_CONFIG}-alonzo-genesis.json
wget -N https://hydra.iohk.io/build/${NODE_BUILD_NUM}/download/1/${NODE_CONFIG}-topology.json
wget -N https://raw.githubusercontent.com/input-output-hk/cardano-node/master/cardano-submit-api/config/tx-submit-mainnet-config.yaml
```

Run the following to modify ${NODE_CONFIG}-config.json and update TraceBlockFetchDecisions to "true" & listen on all interfaces with Prometheus Node Exporter.

```bash
sed -i ${NODE_CONFIG}-config.json \
    -e "s/TraceBlockFetchDecisions\": false/TraceBlockFetchDecisions\": true/g" \
    -e "s/127.0.0.1/0.0.0.0/g"
```

{% hint style="info" %}
**Tip for relay nodes**: It's possible to reduce memory and cpu usage by setting "TraceMemPool" to "false" in **{NODE_CONFIG}-config.json.** This will turn off mempool data in Grafana and gLiveView.sh.
{% endhint %}

### Retrieve aarch64 1.33.1 and cardano-submit-api binaries

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

{% hint style="warning" %}
If binaries already exist (if updating) you will have to confirm overwriting the old ones.
{% endhint %}

Confirm binaries are in $USER's $PATH.

```bash
cardano-node version
cardano-cli version
which cardano-submit-api
```

### Systemd unit startup scripts

Create the systemd unit file and startup script so systemd can manage cardano-node.

```bash
nano ${HOME}/.local/bin/cardano-service
```

Pega lo siguiente, guardar & salir.

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

Abre /etc/systemd/system/cardano-node.service

```bash
sudo nano /etc/systemd/system/cardano-node.service
```

Paste the following, You will need to edit the username here if you chose to not use ada. Guardar y salir

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

Guardar y salir

```bash
source ${HOME}/.adaenv
```

What we just did there was add a couple functions to control our cardano-service and cardano-submit without having to type out

> > sudo systemctl enable cardano-node.service sudo systemctl start cardano-node.service sudo systemctl stop cardano-node.service sudo systemctl status cardano-node.service

Ahora sólo tenemos que hacer:

- cardano-service enable (enables cardano-node.service auto start at boot)
- cardano-service start (starts cardano-node.service)
- cardano-service stop (stops cardano-node.service)
- cardano-service status (shows the status of cardano-node.service)

Or

- cardano-submit enable (enables cardano-submit.service auto start at boot)
- cardano-submit start (starts cardano-submit.service)
- cardano-submit stop (stops cardano-submit.service)
- cardano-submit status (shows the status of cardano-submit.service)

The submit service listens on port 8090. You can connect your Nami wallet like below to submit tx's yourself in Nami's settings.

```bash
http://<node lan ip>:8090/api/submit/tx
```

## ⛓ Sincronización de la cadena ⛓

Ahora estás listo para empezar a usar cardano-node. Hacer esto iniciará el proceso de "sincronización de la cadena". This is going to take about 48 hours and the db folder is about 13GB in size right now. Si tenemos un nodo y sincronizado podemos copiar de ese nodo la carpeta db a nuestros nuevo nodo para ahorrar tiempo. However...

### Descargar la instantánea

He empezado a tomar instantáneas de mi carpeta db de backup y la he alojado en un directorio web. With this service it takes around 20 minutes to pull the latest snapshot and maybe another hour to sync up to the tip of the chain. El servicio se presta según lo establecido. Depende de ti. Si quieres sincronizar la cadena por tu cuenta simplemente:

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

#### Download Database

```bash
wget -r -np -nH -R "index.html*" -e robots=off https://$NODE_CONFIG.adamantium.online/db/
```

Una vez que wget se haya completado habilitar & iniciar cardano-nodo.

```bash
cardano-service enable
cardano-service start
cardano-service status
```

## gLiveView.sh

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

Permitir la ejecución de gLiveView.sh.

```bash
chmod +x gLiveView.sh
```

## topologyUpdater.sh

Hasta que el peer to peer no esté habilitado en los operadores de red se necesita una forma de obtener una lista de relays/peers a los que conectarse. El servicio de actualizador de topología (topology updater) se ejecuta en segundo plano con cron. Cada hora el script se ejecutará y le dirá al servicio que eres un relay y quieres ser parte de la red. It will add your relay to it's directory after four hours you should see in connections in gLiveView. &#x20;

{% hint style="info" %}
The list generated will show you the distance & a clue as to where the relay is located.
{% endhint %}

Download the topologyUpdater script and have a look at it. Lower the number of peers to 10 and add any custom peers you wish. These are outgoing connections. You will not see any incoming transactions untill other nodes start connecting to you.

```bash
wget https://raw.githubusercontent.com/cardano-community/guild-operators/master/scripts/cnode-helper-scripts/topologyUpdater.sh
```

Lower the number of MX_PEERS to 10.

```bash
nano topologyUpdater.sh
```

Guardar, salir y hacerlo ejecutable.

```bash
chmod +x topologyUpdater.sh
```

{% hint style="warning" %}
No podrá ejecutar con éxito ./topologyUpdater.sh hasta que esté completamente sincronizado hasta el final de la cadena.
{% endhint %}

Crea una tarea de cron (cron job) que ejecutará el script cada hora.

```bash
crontab -e
```

{% hint style="info" %}
Elegir nano cuando se te pida que editor quieres usar.
{% endhint %}

Añade lo siguiente en la parte inferior, guardar & salir.

{% hint style="info" %}
La imagen de Pi-Node tiene esta línea de cron deshabilitada por defecto. You can enable it by removing the #.
{% endhint %}

```bash
SHELL=/bin/bash
PATH=/home/ada/.local/bin:/usr/bin:/bin:/usr/sbin:/sbin:/usr/local/bin
33 * * * * . $HOME/.adaenv; $HOME/pi-pool/scripts/topologyUpdater.sh
```

After four hours you can open ${NODE_CONFIG}-topology.json and inspect the list of out peers the service suggested for you. Remove anything more than 7k distance(or less). IOHK recently suggested 8 out peers. The more out peers the more system resources it uses. You can also add any peers you wish to connect to manualy inside the script. This is where you would add your block producer or any friends nodes.

```bash
nano $NODE_FILES/${NODE_CONFIG}-topology.json
```

{% hint style="info" %}
You can use gLiveView.sh to view ping times in relation to the peers in your {NODE_CONFIG}-topology file. Usa Ping para resolver el hostname de la IP.
{% endhint %}

Los cambios en este archivo se verán afectados al reiniciar cardano-service.

{% hint style="warning" %}
¡No olvides eliminar la última coma en tu archivo topology!
{% endhint %}

El estado debe mostrarse como habilitado y en ejecución.

Once your node syncs past epoch 208(shelley era) you can use gLiveView.sh to monitor your sync progress.

{% hint style="danger" %}
It can take over an hour for cardano-node to sync to the tip of the chain. Use el script ./gliveView.sh, el comando htop y la salida del log para ver cómo evoluciona el proceso. Ten paciencia finalizará.
{% endhint %}

```bash
cd $NODE_HOME/scripts
./gLiveView.sh
```

![](../../../../.gitbook/assets/pi-node-glive (7).png)

## Prometheus, Node Exporter & Grafana

Prometheus se conecta al backend de los cardano-nodes y muestra sus métricas mediante http. Grafana a su vez puede usar esos datos para mostrar gráficos y crear alertas. Nuestro panel de control de Grafana estará compuesto de datos de nuestro sistema Ubuntu & nodo de tarjeta. Grafana can display data from other sources as well, like [adapools.org](https://adapools.org).

{% hint style="info" %}
You can connect a [Telegram bot](https://docs.armada-alliance.com/learn/intermediate-guide/grafana-alerts-with-telegram) to Grafana which can alert you of problems with the server. Mucho más fácil que intentar configurar las alertas por correo electrónico.
{% endhint %}

{% embed url="https://github.com/prometheus" %}

![](../../../../.gitbook/assets/pi-pool-grafana (2) (2) (2) (2) (1) (7).png)

### Instalar Prometheus & Node Exporter.

{% hint style="info" %}
Prometheus puede tomar los datos http de otros servidores ejecutando el node-exporter. El mantenimiento de Grafana y Prometheus no tiene que ser instalado en su core ni en sus relays. Only the package prometheus-node-exporter is required if you would like to build a central Grafana dashboard for the pool, freeing up resources and having a single dashboard to monitor everything.
{% endhint %}

```bash
sudo apt install prometheus prometheus-node-exporter -y
```

Desactívalos en el systemd por ahora.

```bash
sudo systemctl disable prometheus.service
sudo systemctl disable prometheus-node-exporter.service
```

### Configurar Prometheus

Abre prometheus.yml.

```bash
sudo nano /etc/prometheus/prometheus.yml
```

Reemplaza el contenido del archivo.

{% hint style="warning" %}
La indentación debe ser correcta en formato YAML o Prometheus no podrá iniciarse.
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

Guardar y salir

Start Prometheus.

```bash
sudo systemctl start prometheus.service
```

### Instalar Grafana

{% embed url="https://github.com/grafana/grafana" %}

Añade la clave gpg de Grafana a Ubuntu.

```bash
wget -q -O - https://packages.grafana.com/gpg.key | sudo apt-key add -
```

Añade el último repositorio estable a las fuentes apt.

```bash
echo "deb https://packages.grafana.com/oss/deb stable main" | sudo tee -a /etc/apt/sources.list.d/grafana.list
```

Actualiza tus listas de paquetes e instala Grafana.

```bash
sudo apt update; sudo apt install grafana
```

Cambia el puerto que escucha Grafana para que no choque con el nodo cardano.

```bash
sudo sed -i /etc/grafana/grafana.ini \
         -e "s#;http_port#http_port#" \
         -e "s#3000#5000#"
```

Start Grafana

```bash
sudo systemctl start grafana-server.service
```

### Función de cardano-monitor bash

Open .adaenv.

```bash
cd ${HOME}; nano .adaenv
```

Abajo en la parte inferior.

```bash
cardano-monitor() {
    #do things with parameters like $1 such as
    sudo systemctl "$1" prometheus.service
    sudo systemctl "$1" prometheus-node-exporter.service
    sudo systemctl "$1" grafana-server.service
}
```

Save, exit & source.

```bash
source .adaenv
```

Aquí vinculamos los tres servicios bajo una sola función. Habilita a Prometheus.service, prometheub node-exporter.service & grafana-server.service para ejecutarse en el arranque e iniciar los servicios.

```bash
cardano-monitor enable
cardano-monitor start
```

{% hint style="warning" %}
En este punto es posible que quieras iniciar el cardano-service y que se sincronice antes de continuar con la configuración de Grafana. Go to the syncing the chain section. Choose whether you want to wait 30 hours or download the latest chain snapshot. Vuelve aquí una vez que gLiveView.sh muestra que estás al final de la cadena.
{% endhint %}

## Grafana, Nginx proxy_pass & snakeoil

Let's put Grafana behind Nginx with self signed(snakeoil) certificate. El certificado se generó cuando instalamos el paquete ssl-cert.

Recibirás una advertencia de tu navegador. This is because ca-certificates cannot follow a trust chain to a trusted (centralized) source. Sin embargo, la conexión está cifrada y protegerá sus contraseñas volando en texto plano.

```bash
sudo nano /etc/nginx/sites-available/default
```

Reemplaza el contenido del archivo.

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
        # See: https://bugs.debian.org/773332
        #
        # Read up on ssl_ciphers to ensure a secure configuration.
        # See: https://bugs.debian.org/765782
        #
        # Self signed certs generated by the ssl-cert package
        # Don't use them in a production server!
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

Compruebe que Nginx está contento con nuestros cambios y reinicie.

```bash
sudo nginx -t
## if ok, do
sudo service nginx restart
```

You can now visit your pi-nodes ip address without any port specification, the connection will be upgraded to SSL/TLS and you will get a scary message(not really scary at all). Continúe hasta su panel de control.

![](../../../../.gitbook/assets/snakeoil.png)

### Configurar Grafana

On your local machine open your browser and enter your nodes private ip address.

Inicie sesión y establezca una nueva contraseña. El nombre de usuario y contraseña por defecto son **admin:admin**.

#### Configurar la fuente de datos

In the left hand vertical menu go to **Configure** > **Datasources** and click to **Add data source**. Elige Prometheus. Enter [http://localhost:9090](http://localhost:9090) where it is grayed out, everything else can be left default. En la parte inferior pinchar en save & test. Deberías obtener el verde "Data source is working" si el cardano-monitor está iniciado. Si por alguna razón estos servicios no pudieron iniciar, reinicie con **cardano-service restart**.

#### Importar Panel de Control (Dashboards)

Guarda los archivos json del dashboard en tu máquina local.

{% embed url="https://github.com/armada-alliance/dashboards" %}

In the left hand vertical menu go to **Dashboards** > **Manage** and click on **Import**. Selecciona el archivo que acabas de descargar/crear y guardar. Head back to **Dashboards** > **Manage** and click on your new dashboard.

![](../../../../.gitbook/assets/pi-pool-grafana (2) (2) (2) (2) (1) (5).png)

### Configurar poolDataLive

Here you can use the poolData api to bring extra pool data into Grafana like stake & price.

{% embed url="https://api.pooldata.live/dashboard" %}

Siga las instrucciones para instalar el plugin Grafana, configurar su fuente de datos e importar el Panel de control.

```bash
sudo grafana-cli plugins install simpod-json-datasource
cardano-monitor restart
```

## Useful Commands

{% hint style="info" %}

Revisa cuánto utiliza el nodo de zram swap.

```bash

CNZRAM=$(pidof cardano-node)
grep --color VmSwap /proc/$CNZRAM/status
```

Seguir salida de log al journal.

```bash
sudo journalctl --unit=cardano-node --follow
```

Seguir salida de log al stdout (log general).

```bash
sudo tail -f /var/log/syslog
```

View network connections with netstat.

```bash
sudo netstat -puntw
```

{% endhint %}

From here you have a Pi-Node with tools to build an active relay or a stake pool from the following pages. Best of luck and please join the [armada-alliance](https://armada-alliance.com), together we are stronger! :muscle:
