---
description: Flash image
---

# Lataa & Polta

## Flash Image

Lataa, asenna & avaa [Raspberry Pi Imager](https://github.com/raspberrypi/rpi-imager/releases/latest). Kytke USB-asema laittteeseen.

{% tabs %}
{% välilehden otsikko="Paikallinen kone (Ubuntu)" %}
```bash
# Ubuntu users can download and install with snapd
sudo apt update
sudo apt install snapd
sudo snap install rpi-imager
```
{% endtab %}
{% endtabs %}

{% hint style="danger" %}
Vanhempien Pi4B 8GB mallien bootloader täytyy päivittää, jotta laite käynnistys USB portin kautta. Jos laitteesi ei käynnisty USB3 portista, käytä rpi-imager ohjelmaa ja tee Pi 4 EEPROM boot recovery sd kortti.

Kytke Pi monitoriin, aseta SD-kortti paikoilleen ja laita virta päälle. Kun näet vihreän näytön, laitteesi pitäisi olla valmis käynnistymään USB3 asemasta. Uudemmat RPi4B versiot toimitetaan bootloaderilla joka tukee valmiiksi USB-käynnistystä. **Feeling lucky?**

**Choose OS -> Misc utility images -> Raspberry Pi 4 EEPROM boot recovery** [https://www.raspberrypi.org/documentation/hardware/raspberrypi/booteeprom.md](https://www.raspberrypi.org/documentation/hardware/raspberrypi/booteeprom.md)
{% endhint %}

![](../../../../.gitbook/assets/otgpoltut (1) (1) (3) (3).png)

{% tabs %}
{% tab title="Pre configured Pi-Node.img.gz" %}
#### Obtain Pi-Node.img.gz file

| [Pi-Node](https://mainnet.adamantium.online/Pi-Node.img.gz)

#### Within Raspberry Pi Imager

**Choose OS -> Use custom**

Locate the .img.gz file you downloaded & wish to flash.

Locate your target drive & write it to disk.

![](../../../.gitbook/assets/custom\_os.png)
{% endtab %}

{% tab title="Fresh Ubuntu 22.04 LTS installation" %}
#### Within Raspberry Pi Imager

#### Download Ubuntu Server 22.04 (RPI 3/4/400)

[Raspberry Pi Generic (64-bit ARM) preinstalled server image](https://cdimage.ubuntu.com/ubuntu-server/daily-preinstalled/current/jammy-preinstalled-server-arm64+raspi.img.xz)

****Choose OS -> Use custom**** Locate the .img.gz file you downloaded & wish to flash. Locate your target drive & write it to disk.
{% endtab %}
{% endtabs %}
