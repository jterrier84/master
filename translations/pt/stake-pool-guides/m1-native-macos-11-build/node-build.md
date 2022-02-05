---
description: Command line prerequisites, compiling the node and node setup.
---

# Node Build

### Going Headless

Fire up putty or your favorite terminal app from another machine on your network and connect to your M1's IP address. If you're on the macOS terminal skip the ssh part here.

```
ssh <user>@<ip address>
```

{% hint style="info" %}
If you turned Remote Login on it should connect and ask for a password. At some point you should setup a SSH key and share it with your M1 so you don't need a password, but I won't cover that here.
{% endhint %}

Once on the command prompt, lets brew install a few things.

```bash
# Install Homebrew for package management
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"

# Stop snooping on me!
brew analytics off

# Installs the modern bash shell into /opt/homebrew/bin/bash
brew install bash
# Change the shell for your user to the newer shell
chsh -s /opt/homebrew/bin/bash

# If you want to export node metrics to your grafana instance
brew install prometheus
brew install node_exporter

# A more powerful time keeper
brew install chrony

# Replacement for top
brew install htop

# Web content retriever
brew install wget

# Replacement for the unavailable sha256sum used in the guild operators env file
brew install sha3sum

# Who doesn't love wireguard?
brew install wireguard-tools
```

### Build the Node

I followed the guide below word for word. You've already installed XCode and Homebrew so skip that. Once finished you should have the necessary packages installed and a working 1.33 (or newer) version of cardano-node and cardano-cli.

{% hint style="warning" %}
Take note of the callouts for the M1 and use the references for $HOME/.bashrc not $HOME/.zshrc.
{% endhint %}

{% embed url="https://developers.cardano.org/docs/get-started/installing-cardano-node#macos" %}

### Setup User Environment

Following the build, let's setup a few things in the terminal. You'll notice here I am using a $HOME/pi-pool directory. My old block producer was created from the Armada Alliance image which uses this path instead of the usual /opt/cardano/cnode you'll find in the Cardano docs. Adjust accordingly.

```bash
##############################################################
# Add a .bash_profile if it doesn't already exist
nano ~/.bash_profile

# Add these lines to the file:
eval "$(/opt/homebrew/bin/brew shellenv)"
test -f ~/.bashrc && source ~/.bashrc

# Save and exit nano
##############################################################

# Create the directory for the blockchain db files
mkdir -p $HOME/pi-pool/db

# Create the directory for the node configuration files
mkdir $HOME/pi-pool/files

# Create the directory for the core key files
mkdir $HOME/pi-pool/keys

# Create the directory for the node logs
mkdir $HOME/pi-pool/logs

# Create the directory for the node related scripts
mkdir $HOME/pi-pool/scripts

##############################################################
# Make some changes to the .bashrc file
nano ~/.bashrc

# Add the following to the bottom of the file:

export CARDANO_NODE_SOCKET_PATH="$HOME/pi-pool/db/socket"
export GENESIS_BLOCK_HASH="8e4d2a343f3dcf9330ad9035b3e8d168e6728904262f2c434a4f8f934ec7b676"

cardano-service() {
    sudo launchctl "$1" cardano.service
}

# Save and exit nano
##############################################################

# Source the new changes into your shell
source ~/.bashrc
```

Allow the pooladmin group that we created earlier access to start/stop the node without sudo.

```bash
##############################################################
# Create a pooladmin sudoers file
sudo visudo /etc/sudoers.d/pooladmin

# Add the following to this file:

%pooladmin ALL=NOPASSWD: /bin/launchctl stop cardano.service
%pooladmin ALL=NOPASSWD: /bin/launchctl start cardano.service

# Save and exit by typing :wq
##############################################################
```

### Create System Services

Create the cardano-service file in \~/.local/bin and then we'll add it as a system service to the macOS launchctl utility.

```bash
##############################################################
# Add the cardano-service file
nano ~/.local/bin/cardano-service

# Add the following to the file adjusting your port accordingly:

#!/usr/bin/env bash
NODE=${HOME}/pi-pool
PORT=3000
HOSTADDR=0.0.0.0
TOPOLOGY=${NODE}/files/mainnet-topology.json
DBPATH=${NODE}/db
SOCKETPATH=${NODE}/db/socket
CONFIG=${NODE}/files/mainnet-config.json

# If you are running this node as a block producer adjust these lines:
KES_SK="${NODE}/keys/<KES>.skey"
VRF_SK="${NODE}/keys/<VRF>.skey"
OPCERT="${NODE}/keys/<NODE>.opcert"

# TODO: Uncomment one of the run lines below. If you are not running as a block
# producer then remove the KES_SK, VRF_SK and OPCERT variables from the line.

# For an 8GB M1 use this:
#$HOME/.local/bin/cardano-node +RTS -N4 -I0.1 -Iw3600 -A32M -AL128M -n4m -F1.1 -H3500M -O3500M -RTS run --config $CONFIG --topology $TOPOLOGY --database-path $DBPATH --socket-path $SOCKETPATH --host-addr $HOSTADDR --port $PORT --shelley-kes-key $KES_SK --shelley-vrf-key $VRF_SK --shelley-operational-certificate $OPCERT

# For a 16GB M1 use this:
#$HOME/.local/bin/cardano-node +RTS -N4 -I0.1 -Iw39600 -A64M -AL256M -n16m -F0.3 -O12G -M23G -c99 -Mgrace=1G -C0 -T -RTS run --config $CONFIG --topology $TOPOLOGY --database-path $DBPATH --socket-path $SOCKETPATH --host-addr $HOSTADDR --port $PORT --shelley-kes-key $KES_SK --shelley-vrf-key $VRF_SK --shelley-operational-certificate $OPCERT

# Save and exit nano
##############################################################

# Make it executable
chmod +x ~/.local/bin/cardano-service
```

Create the launchctl system service .plist file. This will add the cardano-service file we just created to the system so that it starts automatically on a reboot.

```bash
##############################################################
# Create the .plist service definition
sudo nano /Library/LaunchDaemons/cardano.service.plist

# Add the following to the file. Note: change ???? under UserName to your
# user name and also in the ProgramArguments.

<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
        <key>Label</key>
        <string>cardano.service</string>
        <key>UserName</key>
        <string>????</string>
        <key>ProgramArguments</key>
        <array>
                <string>/Users/????/.local/bin/cardano-service</string>
        </array>
        <key>RunAtLoad</key>
        <false/>
        <key>ProcessType</key>
        <string>Interactive</string>
</dict>
</plist>

# Save and exit nano
##############################################################

# Load the new system service definition
sudo launchctl load /Library/LaunchDaemons/cardano.service.plist

# Verify the service definition was loaded successfully
sudo launchctl list cardano.service
```

### Configuration Files

At this point you should now have the system service loaded. Now we are ready to pull over the necessary config files and database snapshot onto the M1 from another relay on our network before we can start the node.

{% hint style="info" %}
If you have an existing block producer or relay on your network, you can rsync the \~/pi-pool files over which makes things easy. If you do not you'll need to get the Cardano config files. The Armada Alliance has them available [here](https://github.com/armada-alliance/cardano-node-binaries/tree/main/config-files/1.30.1-mainnet-config).
{% endhint %}

Assuming you have an existing relay on your network, ssh into that relay. I do and I have the directory structure \~/pi-pool available on this relay. From the relay, perform the following rsync commands:

```bash
# Replace ???? instances with your M1 user name

# rsync the pi-pool config files over from a different node on your network
rsync -a ~/pi-pool/files/ <user>@<M1 IP address>:/Users/????/pi-pool/files

# rsync the pi-pool script files over from a different node on your network
rsync -a ~/pi-pool/scripts/ <user>@<M1 IP address>:/Users/????/pi-pool/scripts

# If your M1 will be a block producer, you need to rsync the pi-pool keys over 
# from your existing block producer on your network.
rsync -a ~/pi-pool/keys/ <user>@<M1 IP address>:/Users/????/pi-pool/keys
```

At this point you need to adjust the \~/pi-pool/files/**mainnet-config.json** and the \~/pi-pool/files/**mainnet-topology.json** on your M1 to account for any changes in paths and topology depending on whether you're running as a relay or block producer.

### db Files

Now for the database snapshot we have a couple options. Easiest and quickest will be to use the already synced \~/pi-pool/db folder from the existing relay you just did the rsync commands above.

* Stop the cardano-node service on the existing relay
* Run the following rsync command on the relay to pull the /db folder over to your M1
  * `rsync -a ~/pi-pool/db/ <user>@<M1 IP address>:/Users/????/pi-pool/db`
  * Give it some time, there's a lot here
* Start the cardano-node service back up on the existing relay

### Start the Node!

Using the service shortcut functions we created in the \~/.bashrc file we can now do this:

```bash
# Fire up the node :)
cardano-service start

# Stop the node using this:
cardano-service stop

# Verify the node is running - should see one line:
ps aux | grep "[c]ardano-node"
# or
sudo htop

# If something is hosed, check the system log for information:
less /var/log/system.log
```

{% hint style="warning" %}
If you turned on the M1 firewall you'll need to ensure the port you used in the cardano-service file is available.
{% endhint %}
