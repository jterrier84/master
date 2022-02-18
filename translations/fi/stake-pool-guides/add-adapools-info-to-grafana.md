---
description: Miten lisätä adapools.org summary.json tiedot Grafana tapahtumaksi.
---

# Lisää adapoolien mittareita Grafanaan 📊

## Oletukset

Olet rakentanut Cardano noden käyttäen yhtä tutoriaaleistamme [täällä](pi-pool-tutorial/). Jos näin on, sinulla pitäisi olla tarvittavat riippuvuudet asennettuna, joita alla olevat ohjeet käyttävät. If not, see the apt install [Environment Setup](../cardano-developer-guides/raspi-node/environment-setup.md#install-packages) section of the Pi-Pool Tutorial.

## Luo uusi hakemisto

Aloittaaksesi, valitse sijainti koneessa, jossa on Grafana. Täällä voit luoda uuden hakemiston node exporterin käyttöön. Solmun viejä sijaitsee todennäköisesti /opt/cardano/monitoring/**node\_exporter** pi-poolin oletussijainnin vuoksi. __Jos tämä ei pidä paikkansa, koita löydätkö sen käyttämällä komentoa "which node\_exporter". Jos tämä ei löydä sitä, hakemisto, jossa se sijaitsee, ei ole sinun $PATH ja sinun täytyy kaivaa syvemmälle. [Check this git](https://github.com/prometheus/node\_exporter) for more information.

Muuta uuden hakemiston sijaintia, tässä olen valinnut paikallisen bin käyttäjälleni.

```
> cd $HOME/.local/bin
```

Nyt tee uusi hakemisto, täällä voimme tallentaa mukautetun tekstitiedoston tilastot joita node\_exporter jäsentää. Kutsun hakemistoa **customStats**, mutta voit nimetä sen haluamallasi tavalla.

```
> mkdir customStats
```

## Hae adapoolien Yhteenvetotiedosto

adapools.org sivusto tarjoaa **summary.json** tiedoston jokaiselle rekisteröidylle poolille. Käytämme tätä tiedostoa jäsentääksemme haluamamme tiedot ja tallentaaksemme sen juuri luomaamme hakemistoon. Voimme luoda bash skriptin, joka käsittelee tämän meille. Olen $HOME/.local/bin hakemistossa:

```
> nano getAdaPoolsSummary.sh
```

Lisää tämä sisältö alla, korvaa **POOLIDI** oman poolisi ID-tunnuksella, tallenna ja poistu. Essentially this pulls a copy of the **summary.json** file for your pool, removes some things that the node exporter cannot parse (string values) and saves a copy in our new directory.

```
curl https://js.adapools.org/pools/<YOUR POOL ID>/summary.json 2>/dev/null \
| jq '.data | del(.direct, .hist_bpe, .handles, .hist_roa, .db_ticker, .db_name, .db_description, .db_url, .ticker_orig, .pool_id, .pool_id_bech32, .group_basic)' \
| tr -d \"{},: \
| awk NF \
| sed -e 's/^[ \t]*/adapools_/' > $HOME/.local/bin/customStats/adapools.prom
```

Nyt kun **getAdaPoolsSummary.sh** on suoritettu, se päivittää tiedoston nimeltä **adapools.prom** uudessa hakemistossamme. Tämä tiedosto sisältää mittareita, jotka alkavat termillä **adapools** ja näkyvät Grafana kyselyn rakentajan mittariosiossa sellaisenaan.

{% hint style="Huomaa" %}
On tärkeää, että tiedoston tulokset eivät sisällä merkkijonon arvoja. Node exporter ilmoittaa virheestä etkä näe adapoolsin metriikkaa.
{% endhint %}

Jos huomaat merkkijonon arvoja, voit poistaa ne lisäämällä uuden avaimen "del" osioon skriptin yllä. For example, to remove the **adapools\_db\_description** metric (has a string value), you'd add **.db\_description** to the **del( )** section.

## Luo crontab Sääntö

Riippuen siitä, kuinka usein haluat päivittää kopion näistä tilastoista, voit luoda paikallisen crontab merkinnän ja vetää tuoreen kopion adapools.prom tiedostosta.

```
> crontab -e
```

Seuraava rivi **ajaa luomamme skriptin 5 minuutin välein**. Lisää rivi, tallenna ja poistu. Koska nämä tiedot eivät muutu kovin usein, sinun ei pitäisi myöskään vetää päivitystä kovin usein. Älä suututa adapools.org:n väkeä vetämällä tätä tietoa 5 sekunnin välein - se ei ole tarpeen. Muita esimerkkejä crontab ajoajoista, [katso tämä ihana linkki](https://crontab.tech/examples).

```
*/5 * * * * $HOME/.local/bin/getAdaPoolsSummary.sh
```

## Suorita node exporter käsky

Nyt kun olemme luomassa **adapools.prom** tiedostoa, meidän täytyy kertoa node exporterille mistä se löytää mukautetun tekstitiedostomme. Riippuen siitä, miten käytät node exporteria, sinun täytyy lisätä seuraavat komentoriviparametrit. Tämä saattaa löytyä **startMonitor** skriptistä, joka sisältyy pi-poolin oletusversioon.

```
> node_exporter --collector.textfile.directory=$HOME/.local/bin/customStats --collector.textfile
```

Jos kaikki menee suunnitelmien mukaisesti, sinun pitäisi pystyä nostamaan tämä URL selaimessasi ja nähdä uusia **adapools** mittareita. Jos tämä toimi, uusien mittareidesi pitäisi näkyä Grafana kyselyn rakentajassa.

```
http://<YOUR GRAFANA NODE IP>:9100/metrics
```

{% hint style="info" %}
On olemassa muitakin menetelmiä, joita voit käyttää tämän lähestymistavan toteuttamiseen. Periaatteessa, jos luot tekstitiedoston avaimen/arvon pareilla ja laitat sen tähän uuteen kansioon, node exporterin pitäisi vetää tiedot Grafanaan. Se avaa laajan valikoiman mahdollisuuksia. Just ensure you prefix the label names with a unique value (the **adapools\_** \_\_part in the adapools.prom file above) per file.
{% endhint %}

Oliko tämä tieto hyödyllistä? Ansaitse palkintoja kanssamme! [Consider delegating some ADA](../delegate.md).
