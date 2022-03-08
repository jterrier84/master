# Staattinen Versio

{% hint style="info" %}
Tämä opas noudattaa samoja asetuksia kuin meidän [Pi-Node opas ja image](../pi-pool-tutorial/) joten sinun täytyy ehkä säätää osioita noden ympäristön ja asetusten perusteella.
{% endhint %}

{% hint style="success" %}
#### Current Official Cardano Node Version: 1.34.1
{% endhint %}

### Overview 🗒

* [ ] Lataa Cardano Noden Staattinen versio & konfiguraatiotiedosto
* [ ] Pura tiedoston sisältö
* [ ] Tarkista, jos sinulla on jo Cardano Node -palvelu käynnissä
  * Sammuta turvallisesti Cardano node, jos se on käynnissä
* [ ] Korvaa vanhat binaarit uudella cardano-nodella ja cardano-cli:llä
* [ ] Tarkista, että cardano-node ja -cli versio on päivitetty nykyiseen versioon
* [ ] Korvaa vanhat asetustiedostot uusilla (jos tarpeen)
* [ ] Käynnistä Cardano node uudelleen
* [ ] Tarkista, että palvelin on käynnistynyt oikein

## Lataa cardano-node & cli

### Static binaries and Cardano node configuration files are provided by [\[ZW3RK\]](https://armada-alliance.com/identities/zw3rk) pool🙏 and can be found at our [Github repository](https://github.com/armada-alliance/cardano-node-binaries/tree/main/static-binaries).

```bash
wget -O 1_34_1.zip https://github.com/armada-alliance/cardano-node-binaries/blob/main/static-binaries/1_34_1.zip?raw=true
```

Pura zip tiedoston sisältö.

```bash
unzip 1_34_1.zip
```

### Tarkista, onko cardano-node jo käynnissä

{% hint style="warning" %}
**Nyt meidän on varmistettava, ettei meidän kardano-node ole jo käynnissä. Jos näin on, meidän on suljettava se ennen jatkamista.**
{% endhint %}

You can check if you have a cardano-node process already running a few ways like using `htop` or by checking your systemd service.

Jos olet seurannut [Pi-Node -opasta](../pi-pool-tutorial/) voit tarkistaa cardano-noden tilan ja lopettaa sen käyttämällä seuraavia komentoja.

```bash
cardano-service status
cardano-service stop
```

{% hint style="info" %}
Jos käytät Linuxin `htop` -komentoa, tarkista vain prosessi, joka alkaa `cardano-node run` ja käytä `SIGINT` lopettaaksesi prosessin.
{% endhint %}

## Korvaa vanhat binäärit ja asetustiedostot uusilla

Jos käytät [Pi-Node -opasta](../pi-pool-tutorial/) ja cardano-node & -cli ovat kansiossa `~/.local/bin`

```bash
mv cardano-node/* ~/.local/bin
```

### Tarkista cardano-noden versio

```bash
cardano-node --version
```

#### Tuloste:

```bash
cardano-node 1.34.1 - linux-aarch64 - ghc-8.10
git rev 0000000000000000000000000000000000000000
```

### Tarkista cardano-cli versio

```bash
cardano-cli --version
```

#### Tuloste:

```bash
cardano-cli 1.34.1 - linux-aarch64 - ghc-8.10
git rev 0000000000000000000000000000000000000000
```

### Download & Replace the Cardano node configuration files (Optional)

{% hint style="info" %}
This step is not needed every time you update your node, typically you only need to update/replace config files after hard fork events when moving into new eras of the [Cardano blockchain](https://roadmap.cardano.org/en/).
{% endhint %}

{% tabs %}
{% tab title="Mainnet" %}
```bash
cd $NODE_FILES
wget https://hydra.iohk.io/build/7370192/download/1/mainnet-config.json
wget https://hydra.iohk.io/build/7370192/download/1/mainnet-byron-genesis.json
wget https://hydra.iohk.io/build/7370192/download/1/mainnet-shelley-genesis.json
wget https://hydra.iohk.io/build/7370192/download/1/mainnet-alonzo-genesis.json
wget https://hydra.iohk.io/build/7370192/download/1/mainnet-topology.json
```
{% endtab %}

{% tab title="Testnet" %}
```bash
cd $NODE_FILES
wget https://hydra.iohk.io/build/7370192/download/1/testnet-config.json
wget https://hydra.iohk.io/build/7370192/download/1/testnet-byron-genesis.json
wget https://hydra.iohk.io/build/7370192/download/1/testnet-shelley-genesis.json
wget https://hydra.iohk.io/build/7370192/download/1/testnet-alonzo-genesis.json
wget https://hydra.iohk.io/build/7370192/download/1/testnet-topology.json
```
{% endtab %}
{% endtabs %}

## Download Database snapshot

{% hint style="info" %}
Thanks to [OTG pool](https://armada-alliance.com/stake-pools/c825168836c5bf850dec38567eb4771c2e03eea28658ff291df768ae) for providing an up to date  snapshot of the Cardano blockchain to help speed up sync times for a node dramatically.
{% endhint %}

```bash
cd $NODE_HOME && rm -rf db
wget -r -np -nH -R "index.html*" -e robots=off https://$NODE_CONFIG.adamantium.online/db/
```

## Käynnistä Cardano Node uudelleen

Nyt meidän täytyy vain käynnistää uudelleen cardano-node palvelu, jos käytät meidän [Pi-Node opasta](../pi-pool-tutorial/) käytä tätä komentoa

```bash
cardano-service start
```

Odota muutama sekunti tai tarkista sitten prosessin status

```bash
cardano-service status
```
