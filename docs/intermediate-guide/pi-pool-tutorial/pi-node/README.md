---
description: >-
  Rakenna synkronoitu node noin tunnissa (ei enää tuntia 1.29)!
---

# Pi-Node (pikaopas)

{% hint style="info" %}
It will take about 30 minutes to download the chain and another couple hours or so to sync to the tip. Et voi tehdä paljoakaan ennen kuin node on synkronoitu lohkoketjun kärkeen asti.

It can take anywhere from 2 to 30 minutes to sync after a reboot depending how the node was shut down or restarted. Tarkista htopilla, onko prosessi käynnissä. Jos se on, käytä gLiveView.sh -skriptiä monitorointiin tai mene kävelylle. Node synkronoituu ja socket luodaan.

On parasta vain jättää se käyntiin. 🏃♀
{% endhint %}

## Pikaohje

<<<<<<< HEAD
### **1. Download and flash the** [**Pi-Node.img.gz**](https://mainnet.adamantium.online/Pi-Node.img.gz)**.**
=======
### **1. Lataa ja asenna** [**Pi-Node.img.gz**](https://mainnet.adamantium.online/Pi-Node.img.gz)**.**
>>>>>>> master

### 2. Ota ssh-yhteys palvelimeen.

```bash
ssh ada@<pi-node private IPv4>
```

Oletustiedot = **ada:lovelace**

{% hint style="Huomaa" %}
Tarkista, mikä cardano-noden versio imagessa on. Noudata staattisen rakentamisen päivityksen ohjeita päivittääksesi. [static-build.md](../../updating-a-cardano-node/static-build.md "mention")

```bash
cardano-node version
```
{% endhint %}

### 3. Mene pi-poolin kansioon.

```bash
cd /home/ada/pi-pool
```

### 4. Lataa tietokannan tilannekuva.

```bash
wget -r -np -nH -R "index.html*" -e robots=off https://mainnet.adamantium.online/db/
```

### 5. Ota käyttöön & aloita cardano-palvelu.

{% hint style="Huomaa" %}
Odota, että wget saa ketjun lataamisen loppuun ennen cardano-servicen aloittamista. Odottaessasi, voit päivittää Ubuntun avaamalla palvelimeen toisen pääteikkunan.

```bash
sudo apt update
sudo apt upgrade
```
{% endhint %}

```bash
cardano-service enable
cardano-service start
```

### 6. Ota käyttöön & aloita cardano-monitor.

```bash
cardano-monitor enable
cardano-monitor start
```

### 7. Vahvista että palvelut ovat käynnissä.

```bash
cardano-service status
cardano-monitor status
```

Seuraa päiväkirja tai syslogia

```
sudo journalctl --unit=cardano-node --follow
sudo tail -f /var/log/syslog
```

### 8. gLiveView.sh

```bash
cd $NODE_HOME/scripts
./gLiveView.sh
```

### 9. Grafana.

Syötä Node:n IPv4 -osoite selaimesi osoitekenttään.

Oletus käyttäjätunnus ja salasana = **admin:admin**

#### Kojelaudat löytyvät täältä.

{% embed url="https://github.com/armada-alliance/dashboards" %}

{% embed url="https://api.pooldata.live/" %}

{% hint style="info" %}
Seuraava opas rakentaa imagen, käytä sitä viitteenä ja voit vapaasti pyytää selvennystä Telegram kanavassamme. [https://t.me/armada\_alli](https://t.me/armada\_alli)
{% endhint %}
