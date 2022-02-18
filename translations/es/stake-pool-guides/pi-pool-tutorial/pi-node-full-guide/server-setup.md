---
description: optimizar hardware, securizar Ubuntu
---

# Configuración del servidor

## Configure Hardware

Let's save some power, raise the governor on the CPU a bit, and set GPU ram as low as we can.

{% hint style="warning" %}
Here are some links for overclocking and testing your drive speeds. If you have heat sinks you can safely go to 2000. Just pay attention to over volt recommendations to go with your chosen clock speed.

- [https://www.raspberrypi.org/documentation/configuration/config-txt/overclocking.md](https://www.raspberrypi.org/documentation/configuration/config-txt/overclocking.md)
- [https://www.seeedstudio.com/blog/2020/02/12/how-to-safely-overclock-your-raspberry-pi-4-to-2-147ghz/](https://www.seeedstudio.com/blog/2020/02/12/how-to-safely-overclock-your-raspberry-pi-4-to-2-147ghz/)
- [https://www.tomshardware.com/how-to/raspberry-pi-4-23-ghz-overclock](https://www.tomshardware.com/how-to/raspberry-pi-4-23-ghz-overclock)
- [https://dopedesi.com/2020/11/24/upgrade-your-raspberry-pi-4-with-a-nvme-boot-drive-by-alex-ellis-nov-2020/](https://dopedesi.com/2020/11/24/upgrade-your-raspberry-pi-4-with-a-nvme-boot-drive-by-alex-ellis-nov-2020/)
- [Legendary Technology: New Raspberry Pi 4 Bootloader USB](https://jamesachambers.com/new-raspberry-pi-4-bootloader-usb-network-boot-guide/)

Take note that Ubuntu stores config.txt in a different location than Raspbian.
{% endhint %}

### Overclock, memoria & radios

Edit /boot/firmware/config.txt. Just paste Pi Node additions in at the bottom.

```bash
sudo nano /boot/firmware/config.txt
```

```
## Pi Node ##
over_voltage=6
arm_freq=2000
gpu_mem=16
dtoverlay=disable-wifi
dtoverlay=disable-bt
```

Save and reboot.

```
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

# swap more to zram
vm.vfs_cache_pressure=500
vm.swappiness=100
vm.dirty_background_ratio=1
vm.dirty_ratio=50

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

```
#!/bin/bash

# Give CPU startup routines time to settle.
sleep 120

sysctl -p /etc/sysctl.conf

exit 0
```

### Deshabilitar IRQ balance

{% hint style="info" %}
[**http://bookofzeus.com/harden-ubuntu/server-setup/disable-irqbalance/**](http://bookofzeus.com/harden-ubuntu/server-setup/disable-irqbalance/)
{% endhint %}

Deberías deshabilitar el IRQ Balance para asegurarte de que no tiene interrupciones de hardware en los procesos. Si deshabilitar el IRQ Balance, se optimizará el equilibrio entre ahorro de energía y rendimiento mediante la distribución de interrupciones de hardware a través de sus múltiples procesadores.

Abre la ventana de inicio /default/irqbalance y añade en la parte inferior. Guarda, sal y reinicia.

```
sudo nano /etc/default/irqbalance
```

```
ENABLED="0"
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

```bash
sudo service chrony restart
```

### Zram swap

{% hint style="info" %}
Hemos descubierto que el nodo de cardano puede usar de forma segura esta swap comprimida de Ram dándonos, básicamente, alrededor de 20gb de Ram. Ya establecimos los parámetros del kernel para zram en /etc/sysctl.conf
{% endhint %}

Swapping to disk is slow, swapping to compressed ram space is faster and gives us some overhead before out of memory (oom).

{% embed url="https://haydenjames.io/raspberry-pi-performance-add-zram-kernel-parameters/" %}

{% embed url="https://lists.ubuntu.com/archives/lubuntu-users/2013-October/005831.html" %}

```
sudo apt install zram-config linux-modules-extra-raspi
```

```bash
sudo nano /usr/bin/init-zram-swapping
```

Multiplica la configuración predeterminada por 3. This will give you 11.5GB of virtual compressed swap in ram.

{% hint style="info" %}
mem=$((totalmem / 2 _ 1024 _ 3))
{% endhint %}

```bash
#!/bin/sh

modprobe zram

# Calculate memory to use for zram (1/2 of ram)
totalmem=`LC_ALL=C free | grep -e "^Mem:" | sed -e 's/^Mem: *//' -e 's/  *.*//'`
mem=$((totalmem / 2 * 1024 * 3))

# initialize the devices
echo $mem > /sys/block/zram0/disksize
mkswap /dev/zram0
swapon -p 5 /dev/zram0

```

### Raspberry Pi & entropy

Before we start generating keys with a headless server we should have a safe amount of entropy.

{% hint style="info" %}
[https://hackaday.com/2017/11/02/what-is-entropy-and-how-do-i-get-more-of-it/](https://hackaday.com/2017/11/02/what-is-entropy-and-how-do-i-get-more-of-it/)

[https://github.com/nhorman/rng-tools](https://github.com/nhorman/rng-tools)
{% endhint %}

> But consider the fate of a standalone, headless server (or a micro controller for that matter) with no human typing or mousing around, and no spinning iron drive providing mechanical irregularity. Where does _it_ get entropy after it starts up? What if an attacker, or bad luck, forces periodic reboots? This is a [real problem](http://www.theregister.co.uk/2015/12/02/raspberry_pi_weak_ssh_keys/).

```
sudo apt-get install rng-tools
```

## Automatic security updates

Enable automatic security updates.

```bash
sudo dpkg-reconfigure -plow unattended-upgrades
```

## Instalar paquetes

Instala los paquetes que necesitaremos.

```bash
sudo apt install build-essential libssl-dev tcptraceroute python3-pip \
         make automake unzip net-tools nginx ssl-cert pkg-config \
         libffi-dev libgmp-dev libssl-dev libtinfo-dev libsystemd-dev \
         zlib1g-dev g++ libncursesw5 libtool autoconf flex bison -y
```

```
sudo reboot
```

### Optionally test drive speed

#### Velocidad de escritura

```
sudo dd if=/dev/zero of=/tmp/output conv=fdatasync bs=384k count=1k; sudo rm -f /tmp/output
```

#### Velocidad de lectura

```
sudo hdparm -Tt /dev/sda
```
