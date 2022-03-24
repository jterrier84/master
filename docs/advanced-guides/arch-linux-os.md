# Asahi Arch Linux, Apple Silicon

Install Asahi Arch, minimal or Desktop

log in to both alarm and root. Change the passwords.

Update as root

```
pacman -Syu
```

Start and enable sshd. pw auth is disabled for root, login with alarm user.

```
systemctl start sshd.service
systemctl enable sshd.service
```
Install the sudo package and open the sudoers file with visudo and enable the wheel group

```
pacman -S sudo git curl wget htop rsync
sudo EDITOR=nano visudo
```

Add a new user to the wheel group, give it a password.

```
useradd -m -G wheel -s /bin/bash ada
passwd ada
```

Log out and back in as your new user with SSH. Test sudo by upgrading the system again.

```
pacman -Syu
```

{% hint style="info" %}
The Arch Bash shell is boring. Optionally install [Bash-it](https://bash-it.readthedocs.io/en/latest/installation/) for a fancy shell.
{% endhint %}

{% hint style="warning" %}
Remember to copy your ssh key and disable password aurthentication in sshd_config.
{% endhint %}

## Bash completion
Add 'complete -cf sudo' to the bottom of .bash_profile and source.

```
echo complete -cf sudo >> ${HOME}/.bash_profile; . $HOME/.bash_profile
```

## Locales

Generate the [locales](https://wiki.archlinux.org/title/locale) you need by uncommenting what you want(en_US.UTF-8 UTF-8 for example) and generating.

```
sudo nano /etc/locale.gen
sudo locale-gen
sudo localectl set-locale LANG=en_US.UTF-8
```

## Time

Set your timezone

```
sudo timedatectl set-timezone America/New_York
```

No more daylight savings, possible to set RTC to local? testing, might not want to do this.

```
sudo timedatectl set-local-rtc 1
# set to 0 for UTC
```

## Chrony

While we are messing with time.. Install and open chrony.conf and replace contents with below (use ctrl+k to cut whole lines).


```
sudo pacman -S chrony
sudo nano /etc/chrony.conf
```

{% hint style="warning" %}
Note: systemd-timesyncd.service is in conflict with chronyd, so you need to disable it first if you want to enable chronyd properly.
{% endhint %}


```
sudo systemctl stop systemd-timesyncd.service
sudo systemctl disable systemd-timesyncd.service
# enable and start chrony
sudo systemctl start chronyd.service
sudo systemctl enable chronyd.service
```

## Packages

Add the following packages to build and run cardano-node.

```
sudo pacman -S --needed base-devel
sudo pacman -S openssl libtool unzip jq bc xz numactl
```

## zram swap

Install and create a conf file with following.

```
sudo pacman -S zram-generator
sudo nano /usr/lib/systemd/zram-generator.conf
```

Add and save.

```
[zram0]
zram-size =  min(ram / 1)
```
Reboot and check htop to confirm.

## Prometheus

Install Prometheus and Prometheus-node-exporter
```
sudo pacman -S prometheus prometheus-node-exporter
```

## Build Grafana from AUR

No arm64 candidate in repos.

```
mkdir ~/git
cd ~/git

git clone https://aur.archlinux.org/grafana-bin.git
cd grafana-bin
makepkg -si
sudo systemctl start grafana.service
sudo systemctl enable grafana.service

```

## Wireguard
```
sudo pacman -Ss wireguard-tools

```

## Static ip

[netctl](https://wiki.archlinux.org/title/netctl)

Copy the ethernet-static template into place with your interface's name and edit.

```
sudo cp /etc/netctl/examples/ethernet-static /etc/netctl/enp3s0
sudo nano /etc/netctl/enp3s0
```

Edit the interface name to match and add one static ip like below for Mac Mini

```
Description='A basic static ethernet connection'
Interface=enp3s0
Connection=ethernet
IP=static
Address=('192.168.1.152/24')
Gateway='192.168.1.1'
DNS=('192.168.1.1')
```

Enable/start the profile and disable/stop dhcp. It will complain when you try and start it. Don't worry, be happy.

```
sudo netctl enable enp3s0
sudo netctl start enp3s0
sudo systemctl stop dhcpcd
sudo systemctl disable dhcpcd
sudo reboot
```

## Build Libsodium

This is IOHK's fork of Libsodium. It is needed for the dynamic build binary of cardano-node.

```bash
cd; cd git/
git clone https://github.com/input-output-hk/libsodium
cd libsodium
git checkout 66f017f1
./autogen.sh
./configure
make
sudo make install
```

Add library path to ldconfig.

```
sudo touch /etc/ld.so.conf.d/local.conf 
echo "/usr/local/lib" | sudo tee -a /etc/ld.so.conf.d/local.conf 
```
Echo library paths into .bashrc file and source it.

```bash
echo "export LD_LIBRARY_PATH="/usr/local/lib:$LD_LIBRARY_PATH"" >> ~/.bashrc
echo "export PKG_CONFIG_PATH="/usr/local/lib/pkgconfig:$PKG_CONFIG_PATH"" >> ~/.bashrc
. ~/.bashrc
```

Update link cache for shared libraries and confirm.

```bash
sudo ldconfig; ldconfig -p | grep libsodium
```

## LLVM 9.0.1

```
cd ~/git
wget https://github.com/llvm/llvm-project/releases/download/llvmorg-9.0.1/clang+llvm-9.0.1-aarch64-linux-gnu.tar.xz
tar -xf clang+llvm-9.0.1-aarch64-linux-gnu.tar.xz
export PATH=~/git/clang+llvm-9.0.1-aarch64-linux-gnu/bin/:$PATH
cd ~/git
git clone https://aur.archlinux.org/ncurses5-compat-libs.git
cd ncurses5-compat-libs/
gpg --recv-key CC2AF4472167BE03
## If this fails it is probly due to your DNS service(Google). 
## Use https://www.quad9.net/
nano PKGBUILD
```
Change target archetecture to aarch64.

```
arch=(aarch64)
```
and build it.

```
makepkg -si
```

Confirm.

```
clang --version
```

## GHCUP, GHC & Cabal

Install ghcup use defaults when asked.

```
curl --proto '=https' --tlsv1.2 -sSf https://get-ghcup.haskell.org | sh
```

```
. ~/.bashrc
ghcup upgrade
ghcup install cabal 3.4.0.0
ghcup set cabal 3.4.0.0

ghcup install ghc 8.10.4
ghcup set ghc 8.10.4
```

build cardano-node

```
cd $HOME/git
git clone https://github.com/input-output-hk/cardano-node.git
cd cardano-node
git fetch --all --recurse-submodules --tags
git checkout $(curl -s https://api.github.com/repos/input-output-hk/cardano-node/releases/latest | jq -r .tag_name)
```

Configure with 8.10.4 set libsodium flag and clean build folder

```
cabal configure -O0 -w ghc-8.10.4

echo -e "package cardano-crypto-praos\n flags: -external-libsodium-vrf" > cabal.project.local
sed -i $HOME/.cabal/config -e "s/overwrite-policy:/overwrite-policy: always/g"
rm -rf dist-newstyle/build/aarch64-linux/ghc-8.10.4

```

Build them.

```
cabal build cardano-cli cardano-node cardano-submit-api
```

Add them to your PATH.

```
cp ~/git/cardano-node/dist-newstyle/build/aarch64-linux/ghc-8.10.4/cardano-cli-1.34.1/x/cardano-cli/build/cardano-cli/cardano-cli $HOME/.local/bin/

cp ~/git/cardano-node/dist-newstyle/build/aarch64-linux/ghc-8.10.4/cardano-node-1.34.1/x/cardano-node/build/cardano-node/cardano-node $HOME/.local/bin/

cp ~/git/cardano-node/dist-newstyle/build/aarch64-linux/ghc-8.10.4/cardano-submit-api-3.1.2/x/cardano-submit-api/build/cardano-submit-api/cardano-submit-api $HOME/.local/bin/

```

Check

```
cardano-node version
cardano-cli version
```







