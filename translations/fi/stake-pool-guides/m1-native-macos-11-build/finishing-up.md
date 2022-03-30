---
description: Setup chrony, start prometheus and fix up gLiveView.
---

# Finishing Up

### Chrony

At the time of writing this the brew install of chrony does not create the necessary system service .plist file so we need to create one.

```bash
##############################################################
# Create the .plist service definition
sudo nano /opt/homebrew/opt/chrony/homebrew.mxcl.chrony.plist

# Add the following to the file:

<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
        <key>Label</key>
        <string>homebrew.mxcl.chrony</string>
        <key>ProgramArguments</key>
        <array>
                <string>/opt/homebrew/sbin/chronyd</string>
        </array>
        <key>RunAtLoad</key>
        <true/>
        <key>StandardErrorPath</key>
        <string>/var/log/chronyd.err.log</string>
        <key>StandardOutPath</key>
        <string>/var/log/chronyd.log</string>
</dict>
</plist>

# Save and exit nano
##############################################################

# Create the run folder for Chrony
sudo mkdir /var/run/chrony
```

Now we need to create the /etc/chrony.conf file which the service will use. I just copied the one from my other block producer.

```bash
##############################################################
sudo nano /etc/chrony.conf

# Add the following to the file:

pool time.google.com        iburst minpoll 2 maxpoll 2 maxsources 3 maxdelay 0.3
pool time.euro.apple.com    iburst minpoll 2 maxpoll 2 maxsources 3 maxdelay 0.3
pool time.apple.com         iburst minpoll 2 maxpoll 2 maxsources 3 maxdelay 0.3
pool ntp.ubuntu.com         iburst minpoll 2 maxpoll 2 maxsources 3 maxdelay 0.3

# This directive specify the location of the file containing ID/key pairs for
# NTP authentication.
keyfile /etc/chrony.keys

# This directive specify the file into which chronyd will store the rate
# information.
driftfile /var/lib/chrony/chrony.drift

# Uncomment the following line to turn logging on.
#log tracking measurements statistics

# Log files location.
logdir /var/log/chrony

# Stop bad estimates upsetting machine clock.
maxupdateskew 5.0

# This directive enables kernel synchronisation (every 11 minutes) of the
# real-time clock. Note that it can’t be used along with the 'rtcfile' directive.
rtcsync

# Step the system clock instead of slewing it if the adjustment is larger than
# one second, but only in the first three clock updates.
makestep 0.1 -1

leapsectz right/UTC

local stratum 10

# Save and exit nano
##############################################################

# Fire up chrony using Brew
sudo brew services start chrony

# Verify chrony started successfully - should see one line:
ps aux | grep "[c]hronyd"

# If something is hosed, check the logs:
less /var/log/system.log
less /var/log/chronyd.err.log
less /var/log/chronyd.log
```



### Prometheus/Node Exporter

For the mini M1 the thermal readings aren't available per node_exporter's default thermal collector. So we need to shut off that collector so logs don't fill up. Let's also add a textfile exporter directory so we can collect custom stats.

```bash
##############################################################
nano /opt/homebrew/etc/node_exporter.args

# Add these two lines:
--collector.textfile.directory=/opt/homebrew/opt/node_exporter/stats
--no-collector.thermal

# Save and exit nano
##############################################################

# Create the textfile collector directory
mkdir /opt/homebrew/opt/node_exporter/stats
```

Register the services with launchctl and start 'em up

```bash
# Start prometheus
sudo brew services start prometheus

# Start the node-exporter
sudo brew services start node_exporter

# Verify they are registered and started
sudo brew services list

# Should see this:
Name          Status  User File
node_exporter started root /Library/LaunchDaemons/homebrew.mxcl.node_exporter.plist
prometheus    started root /Library/LaunchDaemons/homebrew.mxcl.prometheus.plist
```

At this point if you have a Grafana instance on your network you should be able to add the M1 into the mix. I won't cover how to do that here.

{% hint style="info" %}
I am noticing that some of the M1 metrics are named slightly differently than the Ubuntu metrics and some are not available at all - like thermal zones. If you are using Grafana you'll need to play with the metrics to get them correct.
{% endhint %}

{% hint style="Huomaa" %}
If you turned on the M1 firewall you'll need to ensure port 9090 is available if you're going to add the M1 to Grafana if the Grafana server is sitting elsewhere on your network.
{% endhint %}

### gLiveView

The normal guild operators env and gLiveView.sh scripts will complain out of the box on the macOS. So, we need to tweak them a little bit.

Unfortunately this requires us to change stuff in the "Do NOT modify code below" section of gLiveView.sh. Which means if you don't specify the **-u** option it'll see the script has changed and ask you to download the new one.

```bash
##############################################################
nano ~/pi-pool/scripts/gLiveView.sh

# Change the shebang line to this so we use the new shell:
#!/usr/bin/env bash

# This is the only change that will cause gLiveView.sh to ask for script updates so you'll need to use the -u option. MacOS BSD doesn't have a workaround for ps.
# Change this line:
read -ra proc_data <<<"$(ps -q ${CNODE_PID} -o pcpu= -o rss=)"
# to this:
read -ra proc_data <<<"$(ps -p ${CNODE_PID} -o pcpu= -o rss=)"

# Save and exit nano
##############################################################

##############################################################
nano ~/pi-pool/scripts/env

# Change the shebang line to this so we use the new shell:
#!/usr/bin/env bash

# Add the following 5 lines right above the # Do NOT modify code below # line:
# special mapping of coreutils for MacOS
if [[ $(uname) == Darwin ]]; then
   sed () { gsed "$@"; }
   head () { ghead "$@"; }
fi

# Save and exit nano
##############################################################
```

You should now be able to run gLiveView.sh with the -u option and it should work:

`~/pi-pool/scripts/gLiveView.sh -u`
