---
description: Flash image
---

# Descargar y Flashear

## Flashea la imagen

Descarga, instala y abre [ Raspberry pi Imager ](https://github.com/raspberrypi/rpi-imager/releases/latest). Conecte su unidad USB de destino.

{% tabs %}
{% tab title="Local Machine (Ubuntu)" %}
```bash
# Ubuntu users can download and install with snapd
sudo apt update
sudo apt install snapd
sudo snap install rpi-imager
```
{% endtab %}
{% endtabs %}

{% hint style="danger" %}
Los modelos más antiguos del Pi4B 8GB necesitan tener su cargador de arranque (boot loader) actualizado para arrancar desde un USB. Si tu imagen no arranca, retire la unidad USB3 y usa rpi-imager para flashear el recovery de arranque Pi 4 EEPROM a una tarjeta SD.

Conecta el Pi a un monitor, inserta la tarjeta SD y enciéndela. Una vez que vea una pantalla verde sería buen momento para arrancar desde tu unidad USB3. Las versiones más recientes vienen con un cargador de arranque compatible con USB. **Feeling lucky?**

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
