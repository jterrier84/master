---
description: >-
  Tämän oppaan avulla voit rakentaa Cardano Stake Pool, joka kuluttaa vain 4 wattia per Pi. Pi-Node.img.gz:n referenssiopas (reference guide for the Pi-Node.img.gz).
---

# Pi-Node Image & Ohjeet

The guide builds the image from a fresh install of Ubuntu. If you flashed the image you only need the guide as a reference. Head over to the quickstart to get synced up. The relay, core and cold guides will configure the the Pi-Node further.

The [Pi-Node.img.gz](https://mainnet.adamantium.online/Pi-Node.img.gz) is a pre configured Cardano node that can be used for minting NFT's, submitting transactions and can be further configured into a relay or core node for stake pool creation. It is configured with everything you need to quickly bootstrap a synced node with Nginx proxypass of Grafana for TLS encryption with a self signed certificate, working topology updater and gLiveView.

![](../../../.gitbook/assets/photo\_2021-03-09-13.40.29.jpeg)

{% hint style="danger" %}
It is strongly recommended to work through the [Stake Pool School](https://cardano-foundation.gitbook.io/stake-pool-course/) course presented by the Cardano Foundation.
{% endhint %}

{% hint style="Huomaa" %}
If you would like to create a .img file of your work that can be flashed for reuse on other Raspberry Pi's you should build on an 8GB sd card. It will take less time to make an image. See [image creation section](https://app.gitbook.com/@ada-pi/s/raspi-spo/intermediate-guide/pi-pool-tutorial/create-.img-file).
{% endhint %}

## Miksi tämä opas?

Consolidate and organize the various guides into a single document that can be followed or referenced _specifically_ for running a pool using two (or more) Raspberry Pi 4B (the 8GB version) and one offline Pi for cold key operations.

Provide documentation of every step taken while building the Pi-Node image available for bootstrapping pool creation. A reference & guide.

The most popular guides out there are aimed at x86 architecture and '_knowing what to throw away and knowing what to keep_' is not always clear. I aim to change that '_with a little help from my friends_'. 🎸

## Laitteisto

{% hint style="info" %}
[Here is a list of working adapters.](https://jamesachambers.com/raspberry-pi-4-usb-boot-config-guide-for-ssd-flash-drives/)
{% endhint %}

### Ostoslista

* 2 [Pi4B 8GB](https://thepihut.com/products/raspberry-pi-4-model-b?variant=31994565689406) versio.
* 2 Drives : SSD of some sort. Check your drive for compatability.
* M.2 or other scsi to USB3 adapter or whatever works with your drive.
* Kolmas 64bit kykenevä Pi offline-kone (Cold).
* Class 10 micro sd card 16GB or larger.
* Ylimääräisiä USB flash-asemia avainten ja konfiguraatioiden varmuuskopiointiin.
* Harkitse yhtä 50 watin virtalähdettä
* Harkitse 5 voltin gigabitin kytkintä
* Harkitse koteloa, jossa tuuletin

## Kiitokset ja Yhteisö

* [Alessandro konrad](https://github.com/alessandrokonrad) |[ Berry](https://adapools.org/pool/2a748e3885f6f73320ad16a8331247b81fe01b8d39f57eec9caa5091) (@berry_ales)
* Moritz Angermann | [zw3rk](https://adapools.org/pool/e2c17915148f698723cb234f3cd89e9325f40b89af9fd6e1f9d1701a) (@zw3rk)
* [CoinCashew: guide-how-to-build-a-haskell-stakepool-node](https://www.coincashew.com/coins/overview-ada/guide-how-to-build-a-haskell-stakepool-node)
* [Chris-Graffagnino](https://github.com/Chris-Graffagnino)/[Setup Cardano Shelley staking node](https://github.com/Chris-Graffagnino/Jormungandr-for-Newbs/blob/master/docs/jormungandr_node_setup_guide.md)
* [Arming Cardano](https://t.me/joinchat/wvTcCSpdjGllMmFk) Telegram Group
* [Berry Pool](https://t.me/berry_pool) Telegram group
* [Legendary Technology: New Raspberry Pi 4 Bootloader USB](https://jamesachambers.com/new-raspberry-pi-4-bootloader-usb-network-boot-guide/)
* [StakePool Operator Scripts (SPOS)](https://github.com/gitmachtl/scripts/)

## Lataukset

* [Pi-Node.img.gz](https://mainnet.adamantium.online/Pi-Node.img.gz)
* Viimeisimmät epäviralliset [staattiset arm binäärit](https://ci.zw3rk.com/build/1758)
  * [Moritz Angermann](https://t.me/joinchat/FeKTCBu-pn5OUZUz4joF2w)
* Raspberry Pi Imager ([rpi-imager](https://github.com/raspberrypi/rpi-imager))
  * päivitä eeprom
  * flash .img tiedostot/asenna Ubuntu
* [PiShrink](https://github.com/Drewsif/PiShrink)
* [cardanocli-js](https://docs.pipool.online)
* Viimeisimmän ketjun tilannekuva nopeampaa synkronointia varten
  * wget -r -np -nH -R "index.html\*" -e robots=off [https://mainnet.adamantium.online/db/](https://mainnet.adamantium.online/db/)

## Linkit

* [https://cryptsus.com/blog/how-to-secure-your-ssh-server-with-public-key-elliptic-curve-ed25519-crypto.html](https://cryptsus.com/blog/how-to-secure-your-ssh-server-with-public-key-elliptic-curve-ed25519-crypto.html)
* [https://www.raspberrypi.org/forums/viewtopic.php?t=245931](https://www.raspberrypi.org/forums/viewtopic.php?t=245931)
