# Alpine Linux OS

![](../../.gitbook/assets/image (1).png)

## Miksi käyttää AlpineOS Raspberry Pi:ssä? Tässä muutamia syitä:

* **Erittäin alhainen muistinkulutus (~50MB käytetään idle vs ~350MB Ubuntu 20.04).**
* **Alempi suorittimen kuormitus** **(27 tehtävää / 31 threadiä aktiivisena Alpinessa vs 57 tehtävää / 111 threadiä Ubuntussa, kun cardano-node on käynnissä).**
* **Viileämpi Pi 😎 (kirjaimellisesti, CPU toimii viileämpänä alemman suorittimen kuormituksen ansiosta).**
* **Ja lopuksi, miksi ei? Jos tulet käyttämään staattisia binäärejä, voit yhtä hyvin hyödyntää AlpineOS:ää 😜**

## Jos olet aiemmin käyttänyt tätä opasta ja aiot päivittää komentosarjoja, seuraa näitä ohjeita. Seuraa sitten muita tässä oppaassa hahmoteltuja vaiheita vastaavasti 🙂.

1) Päivitä paikallinen git repo.

```
cd ~/alpine-rpi-os
```

```
git fetch --recurse-submodules --tags --all
```

2) Tunnista viimeisin tagi.

```
git tag
```

3\) Replace `<tag>` in this step with the latest tag such as `v1.5.0`.

```
git checkout tags/<tag>
```

## Upgrading to Alpine v3.15 from Alpine v3.14:

1) Päivitä nykyinen AlpineOS-versio.

```
sudo apk update
```

```
sudo apk upgrade
```

2\) Edit the repository to reflect Alpine v3.15.

```
sudo sed -i 's@v3.14@v3.15@g' /etc/apk/repositories
```

3) Päivitä pakettiluettelo.

```
sudo apk update
```

4\) Upgrading packages to v3.15

```
sudo apk add --upgrade apk-tools
```

```
sudo apk upgrade --available
```

```
sudo sync
```

```
sudo reboot now
```

5\) Now you should have AlpineOS upgraded to v3.15 🍷.

```
cat /etc/alpine-release
```

6\) To troubleshoot the upgrade, refer to the link: [https://wiki.alpinelinux.org/wiki/Upgrading\_Alpine](https://wiki.alpinelinux.org/wiki/Upgrading\_Alpine)

## AlpineOS: ensiasennus Raspberry Pi 4B 8GB koneeseen:

1\) Download the AlpineOS for RPi 4 aarch64 [here](https://dl-cdn.alpinelinux.org/alpine/v3.15/releases/aarch64/alpine-rpi-3.15.0-aarch64.tar.gz).&#x20;

2) Pura .tar.gz tiedosto ja kopioi sen sisältö SSD/SD kortille

3) Kytke näppäimistö ja monitori.

4) Kirjaudu sisään käyttäjätunnuksella 'root'.

5) Suorita komento `setup-alpine` ja noudata ohjeita.

{% hint style="info" %}
Kun olet `setup-alpinessa`, sinua kehotetaan valitsemaan järjestelmälevy. Kun olet tässä vaiheessa, syötä, `y`, määrittääksesi levyn ja luodaksesi osion `sys`:lle.
{% endhint %}

6) Käynnistä kone uudelleen.

7) Lisää uusi käyttäjä nimeltä cardano käyttämällä komentoa `adduser cardano` ja sille salasana ohjeiden mukaisesti.

8) Suorita seuraavat komennot myöntääksesi uudelle käyttäjälle root-oikeudet

```
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

9) Joko poistu root roolista `exit` komennon avulla tai käynnistä uudelleen ja kirjaudu sisään käyttäjänä cardano

10) Asenna bash varmistaaksesi bash skriptien yhteensopivuus

```
    sudo apk add bash
```

11) Asenna myös git ja wget, tarvitsemme niitä myöhemmin.

```
    sudo apk add git wget
```

12) Oletuksena AlpineOS käyttää virransäästön hallintaa, joka asettaa suorittimen taajuuden alhaisimmille mahdolliselle. Käyttääksesi ondemand säätöä, joka skaalaa suorittimen taajuutta järjestelmän kuormituksen mukaan, `cpufreq.start` sisältyy tähän repositoryyn, joka tulee lisätä kansioon /etc/local.d/. Voit käyttää seuraavia komentoja tehdäksesi tämän.

```
    cd ~
```

```
    git clone https://github.com/armada-alliance/alpine-rpi-os
```

```
    git tag
```

Korvaa `<tag>` uusimmalla tunnisteella seuraavassa komennossa.

```
    git checkout tags/<tag>
```

```
    sudo cp alpine-rpi-os/alpine_cnode_scripts_and_services/etc/local.d/cpufreq.start /etc/local.d/
```

```
    sudo chmod +x /etc/local.d/cpufreq.start
```

```
    sudo rc-update add local default
```

12) **[CPU Governor - Optional]** AlpineOS käyttää oletuksena powersave säädintä, joka asettaa suorittimen taajuuden alhaisimmille. Käyttääksesi ondemand säätöä, joka skaalaa suorittimen taajuutta järjestelmän kuormituksen mukaan, `cpufreq.start` sisältyy tähän repositoryyn, joka tulee lisätä kansioon /etc/local.d/. Voit käyttää seuraavia komentoja tehdäksesi tämän.

```
    cd ~
```

```
    git clone https://github.com/armada-alliance/alpine-rpi-os
```

```
    cd alpine-rpi-os
```

```
    sudo cp alpine-rpi-os/alpine_cnode_scripts_and_services/etc/local.d/cpufreq.start /etc/local.d/
```

```
    sudo chmod +x /etc/local.d/cpufreq.start
```

```
    sudo rc-update add local default
```

13) **[ZRAM - Valinnainen]** RPin RAM-muistin rajallisuuden helpottamiseksi ZRAM:n käyttöönotto on suositeltavaa jotta saat RAM-kompression käyttöön. Käytä seuraavia vaiheita zram-initin ja skriptien asentamiseen. Annetut skriptit mahdollistavat 50 %: n lisäyksen käytettävissä olevaan RAM-kapasiteettiin. Tämä kohta olettaa, että olet seurannut vaihetta 12.

```
    sudo apk add zram-init
```

```
    sudo cp alpine-rpi-os/alpine_cnode_scripts_and_services/etc/local.d/zram.* /etc/local.d/
```

```
    sudo chmod +x /etc/local.d/zram.*
```

14) Käynnistä järjestelmä uudelleen. Raspberry Pi 4B 8GB koneella, voit odottaa noin 3.81 Gt swapia ZRAMin kautta kun tarkastelet sitä `htop` -käskyn kautta (`sudo apk add htop` jos htop ei ole käytettävissä).

## 'cardano-node' ja 'cardano-cli' staattisten binäärien Asentaminen/päivittäminen (AlpineOS käyttää lähes yksinomaan staattisia binäärejä, joten vältä ei-staattiset rakennelmia)

{% hint style="info" %}
**You may obtain the static binaries for version 1.33.0 via this** [**link** ](https://dl-cdn.alpinelinux.org/alpine/v3.15/releases/aarch64/alpine-rpi-3.15.0-aarch64.tar.gz)**thanks to Moritz Angermann, the SPO of ZW3RK pool 🙏**
{% endhint %}

**Suorita seuraavat komennot ladataksesi ja asentaaksesi binäärit oikeaan kansioon.**

* Lataa binäärit

```
    wget -O ~/aarch64-unknown-linux-musl-cardano-node-1.33.0.zip https://ci.zw3rk.com/build/427926/download/1/aarch64-unknown-linux-musl-cardano-node-1.33.0.zip
```

* Pura ja asenna binäärit komentojen kautta

```
    unzip -d ~/ aarch64-unknown-linux-musl-cardano-node-1.33.0.zip

    sudo mv ~/cardano-node/* /usr/local/bin/
```

## Asenna Armada Alliancen Alpine Linux Cardano node -palvelu

{% hint style="success" %}
#### Jos olet päättänyt käyttää AlpineOS käyttöjärjestelmää Cardano stake poolissasi, saatat löytää tästä skripti ja palvelu kokoelmasta hyödyllisiä työkaluja.
{% endhint %}

{% hint style="info" %}
#### Asentaaksesi skriptit ja palvelut oikein, älä ohita vaiheita 🏴‍☠️😎
{% endhint %}

1) Kloonaa tämä repo saadaksesi tarvittavat kansiot ja skriptit cardano noden nopeaan käynnistämiseen. Voit ohittaa tämän vaiheen, jos olet jo kloonannut tämän repon vaiheesta 12 AlpineOS:n perustamisen yhteydessä.

```
    cd ~
```

```
    git clone https://github.com/armada-alliance/alpine-rpi-os
```

```
    git tag
```

Korvaa `<tag>` uusimmalla tunnisteella seuraavassa komennossa.

```
    git checkout tags/<tag>
```

2) Suorita seuraavat komennot ja asenna sitten **cnode** -kansio, skriptit ja palvelut oikeisiin kansioihin. **cnode** kansio sisältää kaiken mitä **Cardano node** tarvitsee käynnistyäkseen toiminnallisena relay nodena.

```
    cd ~
```

```
    cp -r alpine-rpi-os/alpine_cnode_scripts_and_services/home/cardano/* ~/
```

```
    sudo cp alpine-rpi-os/alpine_cnode_scripts_and_services/etc/init.d/* /etc/init.d/
```

```
    chmod +x ~/start_stop_cnode_service.sh ~/cnode/autorestart_cnode.sh
```

```
    sudo chmod +x /etc/init.d/cardano-node /etc/init.d/prometheus /etc/init.d/node-exporter
```

3) Nopeampaa synkronointia varten, harkitse tätä valinnaista komentoa uusimman db-kansion lataamiseen yhden Alliance-jäsenemme ylläpitämältä serveriltä.

```
    wget -r -np -nH -R "index.html*" -e robots=off https://mainnet.adamantium.online/db/ -P ~/cnode
```

4) Asennettuasi **cnode**, skriptit ja palvelut, noudata **$HOME** hakemiston **README.txt** tiedoston sisältämää opasta.

```
    more ~/README.txt
```

## Asenna prometheus ja node exporter

1) Lataa Prometheus ja node exporter kotihakemistoon

```
    wget -O ~/prometheus.tar.gz https://github.com/prometheus/prometheus/releases/download/v2.32.1/prometheus-2.32.1.linux-arm64.tar.gz
```

```
    wget -O ~/node_exporter.tar.gz https://github.com/prometheus/node_exporter/releases/download/v1.3.1/node_exporter-1.3.1.linux-arm64.tar.gz
```

2) Pura tarballit

```
tar -xzvf prometheus.tar.gz
```

```
tar -xzvf node_exporter.tar.gz
```

3\) If you are upgrading from previous versions of prometheus and node-exporter, make a backup. You may delete these folders if they do not contain anything useful.

```
mv prometheus prometheus-bak
```

```
mv node_exporter node_exporter-bak
```

4\) Rename the folders with the following commands

```
    mv prometheus-2.32.1.linux-arm64 prometheus
```

```
    mv node_exporter-1.3.1.linux-arm64 node_exporter
```

4\) Follow the guide written in **README.txt** contained in the $HOME directory after installing cnode, scripts and services to start the services accordingly.

```
    more ~/README.txt
```

## Yleinen Vianmääritys

1\) If you have trouble with port forwarding via SSH, run the following command

```
sudo nano /etc/ssh/sshd_config
```

Muokkaa riviä `AllowTcpForwarding no` vastaamaan `AllowTcpForwarding yes`

{% hint style="info" %}
Varmista, ettei tätä riviä ole kommentoitu pois `#` -merkillä
{% endhint %}

2\) If you run into a situation where you get the error:

> /lib/rc/sh/openrc-run.sh: source: line 10: can't open '/home//cnode\_env': No such file or directory

Do the following:

```
export USER=$(whoami)
sudo ash -c "echo 'export USER=$USER' >> /etc/profile"
```

Then reboot:

```
sudo reboot
```

{% hint style="success" %}
We would like to give a special shoutout to our [alliance member](https://armada-alliance.com), [Sayshar](https://armada-alliance.com/identities/sayshar-srn), operator of [\[SRN\] Pool](https://armada-alliance.com/stake-pools/cc1b1c03798884c636703443a23b8d9e827d6c0417921600394198a0), for providing this tutorial 🏴‍☠️ 🙏 😎
{% endhint %}
