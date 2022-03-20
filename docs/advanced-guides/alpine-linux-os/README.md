# armada-alliance/alpine-rpi-os

## Why use AlpineOS on the Raspberry Pi? Here are some reasons:

1. Very low memory consumption \(~50MB utilized during idle vs ~350MB for Ubuntu 20.04\).
2. Lower CPU overhead \(27 tasks/ 31 threads active for Alpine vs 57 tasks / 111 threads for Ubuntu when cardano-node is running\).
3. Cooler Pi😎 \(Literally, CPU runs cooler because of the lower CPU overhead\).
4. And finally, why not? If you're gonna use static binaries, might as well take advantage of AlpineOS😜

### Initial Setup for AlpineOS on Raspberry Pi 4B 8GB:

1. Download the AlpineOS for RPi 4 aarch64 here: [https://dl-cdn.alpinelinux.org/alpine/v3.13/releases/aarch64/alpine-rpi-3.13.5-aarch64.tar.gz](https://dl-cdn.alpinelinux.org/alpine/v3.13/releases/aarch64/alpine-rpi-3.13.5-aarch64.tar.gz)
2. Decompress the .tar.gz file and copy its contents into an SSD/SD card
3. Plug in a keyboard and monitor.
4. Login with username 'root'. It should prompt you for a new password on the first login.
5. Run the command `setup-disk` and create the partition. You may have to retry and erase the entire disk.
6. Run the command `setup-alpine` and follow the instructions.
7. Add a new user called cardano via the command `adduser cardano` and its password as instructed.
8. Run the following commands to grant the new user root privileges

```text
apk add sudo
echo '%wheel ALL=(ALL) ALL' > /etc/sudoers.d/wheel
addgroup cardano wheel
addgroup cardano sys
addgroup cardano adm
addgroup cardano root
addgroup cardano bin
addgroup cardano daemon
addgroup cardano disk
addgroup cardano floppy
addgroup cardano dialout
addgroup cardano tape
addgroup cardano video
```

1. Either exit root via the command `exit` or reboot and login to cardano
2. Install bash to ensure bash script compatibility
3. Also, install git, we will need it later.

## Installing the 'cardano-node' and 'cardano-cli' static binaries \(AlpineOS uses static binaries almost exclusively so you should avoid non-static builds\)

#### You can obtain the static binaries for version 1.27.0 via the link \[[https://ci.zw3rk.com/build/1758](https://ci.zw3rk.com/build/1758)\] courtesy of Moritz Angermann, the SPO of ZW3RK. You can follow the following commands to install the binaries into the correct folder:

1. Download the binaries

   ```text
   wget -O https://ci.zw3rk.com/build/1758/download/1/aarch64-unknown-linux-musl-cardano-node-1.27.0.zip
   ```

2. Unzip and install the binaries via the commands

   ```text
   unzip -d ~/ aarch64-unknown-linux-musl-cardano-node-1.27.0.zip

   sudo mv ~/cardano-node/* /usr/local/bin/
   ```

## If you have decided to use AlpineOS for your Cardano stake pool operations, you may find this collection of script and services useful.

### To install the scripts and services correctly:

1. Clone this repo to obtain the neccessary folder and scripts to quickly start your cardano node. Use the command:

   ```text
   git clone https://github.com/armada-alliance/alpine-rpi-os
   ```

2. Run the following commands to then install the cnode folder, scripts and services into the correct folders. The **cnode** folder contains everything a cardano-node needs to start as a functional relay node:

   ```text
   cd alpine-rpi-os

   sudo cp alpine_cnode_scripts_and_services/home/cardano/* ~/
   ```

   ```text
   sudo cp alpine_cnode_scripts_and_services/etc/init.d/* /etc/init.d/
   ```

   ```text
   chmod +x start_stop_cnode_service.sh cnode/autorestart_cnode.sh
   ```

   ```text
   sudo chmod +x /etc/init.d/cardano-node /etc/init.d/prometheus /etc/init.d/node-exporter
   ```

3. Follow the guide written in README.txt contained in the $HOME directory after installing cnode, scripts and services.

### If you plan on using prometheus and node exporter, do the following:

1. Download prometheus and node-exporter into the home directory

   ```text
   wget -O ~/prometheus.tar.gz https://github.com/prometheus/prometheus/releases/download/v2.27.1/prometheus-2.27.1.linux-arm64.tar.gz
   ```

   ```text
   wget -O ~/node_exporter.tar.gz https://github.com/prometheus/node_exporter/releases/download/v1.1.2/node_exporter-1.1.2.linux-arm64.tar.gz
   ```

2. Rename the folders with the following commands

   ```text
   mv prometheus-2.27.1.linux-arm64 prometheus
   ```

   ```text
   mv node_exporter-1.1.2.linux-arm64 node_exporter
   ```

3. Follow the guide written in README.txt contained in the $HOME directory after installing cnode, scripts and services to start the services accordingly.

{% hint style="success" %}
We would like to give a special shoutout to our [alliance member](https://armada-alliance.com) and operator of [\[SRN\] Pool](https://www.adasrn.com/) for providing this tutorial 🏴‍☠️ 🙏 😎
{% endhint %}



