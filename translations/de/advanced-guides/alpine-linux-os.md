# Alpine Linux OS \(COMING-SOON\)

## Warum AlpineOS auf der Raspberry Pi benutzen? Hier sind einige Gründe:

1. Sehr niedriger Speicherverbrauch \(~50MB im idle ~350MB für Ubuntu 20.04\).
2. Weniger CPU nutzung \(27 tasks/ 31 Threads aktiv für Alpine vs 57 tasks / 111 Threads für Ubuntu, wenn cardano-node läuft\).
3. Cooler Pi😎 \(Wortwörtlich, CPU ist kühler wegen der geringen last!\).
4. Und zu guter Letzt, warum nicht? Wenn Du statische Binärdateien verwenden willst, dann gleich mit den Vorteilen von AlpineOS😜

### Initiales Setup für AlpineOS mit Raspberry Pi 4B 8GB:

1. AlpineOS für RPi 4 aarch64 herunterladen: [https://dl-cdn.alpinelinux.org/alpine/v3.13/releases/aarch64/alpine-rpi-3.13.5-aarch64.tar.gz](https://dl-cdn.alpinelinux.org/alpine/v3.13/releases/aarch64/alpine-rpi-3.13.5-aarch64.tar.gz)
2. Dekomprimiere die .tar.gz Datei und kopiere ihren Inhalt in eine SSD/SD-Karte
3. Schließe eine Tastatur und einen Monitor an.
4. ~Login mit Benutzername 'root'. Es sollte Dich bei der ersten Anmeldung nach einem neuen Passwort fragen. ~~
5. Führen Sie den Befehl `setup-disk` aus und erstellen die Partition. Wenn das nicht klappt, probiere es erneut und lösche die ganze Festplatte.
6. Führe den Befehl `setup-alpine` aus und folge den Anweisungen.
7. Füge einen neuen Benutzer namens cardano über den Befehl `adduser cardano` und sein Passwort wie beschrieben hinzu.
8. Führe die folgenden Befehle aus, um die neuen Benutzer root rechte zu geben

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

1. Verlasse root über den Befehl `exit` oder starte den Pi neu und melde dich mit dem neuen Cardano user an
2. bash installieren, um die Kompatibilität des Bash-Scripts sicherzustellen
3. git auch gleich installieren, wird für später benötigt.

## Installation der statischen Binärdateien 'cardano-node' und 'cardano-cli' \(AlpineOS verwendet statische Binärdateien - vermeide nicht statische!\)

#### Die statischen Binärdateien für die Version 1.27.0 können über den Link \[[https://ci.zw3rk.com/build/1758](https://ci.zw3rk.com/build/1758)\] mit freundlicher Genehmigung von Moritz Angermann, dem SPO von ZW3RK, heruntergeladen werden. Führe folgende Befehle aus um die Binaries im korrekten Order zu installieren:

1. Die Binärdateien herunterladen

   ```text
   wget -O https://ci.zw3rk.com/build/1758/download/1/aarch64-unknown-linux-musl-cardano-node-1.27.0.zip
   ```

2. Entpacke installiere die Binärdateien über die Befehle

   ```text
   unzip -d ~/ aarch64-unknown-linux-musl-cardano-node-1.27.0.zip

   sudo mv ~/cardano-node/* /usr/local/bin/
   ```

## Wenn du AlpineOs für einen stake-pool verwendest, findest du im folgenden einige nützliche Tools.

### Skripte und Dienste korrekt installieren:

1. Klone dieses Repo mit den notwendigen Ordner und Skripte für die cardano-node. Befehl verwenden:

   ```text
   git clone https://github.com/armada-alliance/alpine-rpi-os
   ```

2. Führe die folgenden Befehle aus, um dann den cnode Ordner, Skripte und Dienste in die richtigen Ordner zu installieren. Der **cnode** Ordner enthält alles, was eine cardano-node braucht um als relay zu fungieren:

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

3. Folge nun dem README.txt guide welches im $HOME directory ist.

### Bei benutzung von prometheus und node exporter sollte folgendes gemacht werden:

1. Prometheus und node-exporter in das Home-Verzeichnis herunterladen

   ```text
   wget -O ~/prometheus.tar.gz https://github.com/prometheus/prometheus/releases/download/v2.27.1/prometheus-2.27.1.linux-arm64.tar.gz
   ```

   ```text
   wget -O ~/node_exporter.tar.gz https://github.com/prometheus/node_exporter/releases/download/v1.1.2/node_exporter-1.1.2.linux-arm64.tar.gz
   ```

2. Ordner mit folgenden Befehlen umbenennen

   ```text
   mv prometheus-2.27.1.linux-arm64 prometheus
   ```

   ```text
   mv node_exporter-1.1.2.linux-arm64 node_exporter
   ```

3. Folge nun dem README.txt guide welches im $HOME directory ist um die node und Services zu starten.

{% hint style="success" %}
Herzlichen Dank an unsere alliance members und Operators vom  [\[SRN\] Pool](https://www.adasrn.com/), die dieses Tutorial möglich gemacht haben! 🏴‍☠️ 🙏 😎
{% endhint %}



