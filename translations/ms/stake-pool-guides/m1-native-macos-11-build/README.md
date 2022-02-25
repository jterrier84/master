---
description: >-
  This guide is a series of steps I took to spin up a block producer node on the M1 apple silicon using the native macOS 11 Big Sur. No VMs, just pure Apple. Brought to you by @eastpiada
---

# M1: Native macOS 11 Build

Disclaimer: I'm not a Mac expert. This was the first real exposure to the macOS I've ever had. Working from notes, so if something is slightly off the solution should be nearby. This was a proof of concept to see if I could shift from a Raspberry Pi 4 mainnet block producer to an M1. Go big or go home, right? Let's do this.

#### Assumptions

* You have an M1 setup with a user account that has access to the Mac Apple Store.
* You have admin rights to the M1, in other words you can sudo in the terminal.
* You have an existing mainnet relay or block producer built from the Armada Alliance image on your network already. If you have one that was not built from the AA image you'll notice where you need to adjust accordingly.
  * I used my existing block producer to rsync config files and the db over to the M1.

### Prep Work

Fire up the M1 to the desktop. We need to do a few tasks while we're on the desktop to support the work later on. This can probably be done on the command line, but like I said, I'm not a Mac expert and honestly I don't know how and didn't take the time to figure it out. :smile:

* Open System Preferences > Users & Groups
  * Click the padlock to unlock the settings
  * Make sure your user is highlighted and right-click the icon to select Advanced Options
    * Change your login shell to /bin/bash - note this is an older version of the bash shell which we'll change later
  * Click the plus icon (+) under Login Options
    * Change New Account type to Group
    * Add group name:  pooladmin
  * Click Login Options
    * Turn automatic login off
* System Preferences > Network
  * Configure a static IP on your network
  * Shut off bluetooth, wifi, thunderbolt - I won't use them, save some watts
* System Preferences > Sharing
  * Turn Remote Login on
* System Preferences > Desktop & Screen Saver
  * Turn off the screen saver
  * Turn off desktop change picture
  * These are heavy users of resources
* System Preferences > Energy Saver
  * Prevent computer from sleeping is checked
  * Put disks to sleep is not checked
  * Start up automatically is checked
* Close System Preferences
* I didn't mess around with the firewall at first, left it off (default I think). If you turn it on or it's on just make sure you have Remote Login access for now.
  * Once I finished this guide I turned the firewall on. MacOS should auto-negotiate the ports for you and just ensure they are set to open (green).
* Install Xcode from the Mac App Store
  * After it finishes, install Xcode command line tools
  * Open the macOS terminal and type:  `xcode-select --install`
* You can now log off the M1 macOS.
  * I chose to go headless and connect over ssh from another machine on my network.
  * If you want to use the macOS terminal or your favorite Mac terminal program beyond this point, I think you'll be okay.

