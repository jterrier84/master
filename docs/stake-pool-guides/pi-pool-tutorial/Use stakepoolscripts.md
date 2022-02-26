---
description: How to use the stakepoolscripts to start a pool, rotate KES and update pool data.
---

# Create a stake pool

Now that everything is set up let's start creating the pool. Please read the [documentation](https://github.com/gitmachtl/scripts) Martin provides to get a better understanding of the scripts. 
His tutorial is much more detailed and covers a lot of options. 
This tutorial is only for the basic workflow. It contains the necessary steps to get a stake pool running and registered on chain.

{% embed url="https://github.com/gitmachtl/scripts" %}


{% hint style="warning" %} Basically everything is created offline. Make sure that you never expose your secret keys to an online environment and back them up, multiple times best case. The only keys you need on your core are: kes-xxx.skey, vfr.skey and node-xxx.opcert. {% endhint %}


{% hint style="info" %} The transfer with the USB device is fully automated. It just needs to be mounted at the current working environment, which should also work automated.
If not mount it with ```sudo mount ~/usb_transfer``` {% endhint %}

## Setup 

Let's begin with a directory for your keys. 
Also make sure the offline machine's time is correct. You'll have to do it everytime you use it. 

{% tab title="offline" %}
```bash
cd
mkdir pool_keys
cd pool_keys
```
{% endtab %}

{% tab title="offline" %}
```bash
timedatectl
```
{% endtab %}

## Creating and funding a wallet

First of all you'll need a wallet and with it a staking key. Create the keys and name the wallet accordingly.

{% tab title="offline" %}
```bash
03a_genStakingPaymentAddr.sh wallet_name cli
```
{% endtab %}

Now copy the addresses to your core to fund the new wallet. You'll need your fresh USB drive for that.

{% tab title="offline" %}
```bash
01_workOffline.sh attach wallet_name.payment.addr
01_workOffline.sh attach wallet_name.staking.addr
```
{% endtab %}

Switch the USB drive from offline to online machine.

{% tab title="online" %}
```bash
cd $HOME/pi-pool
01_workOffline.sh extract
```
{% endtab %}

Retrieve the address and send some funds to your new wallet. You'll need at least 502 ADA + tx fees + your pledge.

{% tab title="online" %}
```bash
cat wallet_name.payment.addr
```
{% endtab %}

Query the balance and wait until the new UTXO shows up.

{% tab title="online" %}
```bash
01_queryAddress.sh wallet_name.payment
```
{% endtab %}

When the funds arrived copy the UTXO data to your offline machine. 

{% tab title="online" %}
```bash
01_workOffline.sh add wallet_name.payment
```
{% endtab %}



Generate the stakeaddress registration transaction.

{% tab title="offline" %}
```bash 
03b_regStakingAddrCert.sh wallet_name.staking 
```
{% endtab %}

Generate the keys for your core node

{% tab title="offline" %}
```bash
04a_genNodeKeys.sh pool_name cli
04b_genVRFKeys.sh pool_name
04c_genKESKeys.sh pool_name
04d_genNodeOpCert.sh pool_name
```
{% endtab %}




Generate your stakepool certificate and metadata.json.

{% tab title="offline" %}
```bash
05a_genStakepoolCert.sh pool_name
```
{% endtab %}

This creates a pool_name.pool.json file, which you can edit according to your needs and wishes. 
In this case we get a pool with 100k ADA pledge, 340 ADA fixed cost (minimum) and 1% margin.
For OwnerName and poolRewards enter the name of your wallet.
Add as many relays as you want. Either ip or dns based.
Pool description can contain up to 255 characters.
metaUrl only 64. Ticker 3-5.
You may also add an url to your extended.metadata.json. Example for extended metadata below.


{% tab title="metadata.json" %}
```bash
{
   "poolName": "pool_name",  
   "poolOwner": [
      {
      "ownerName": "wallet_name",
      "ownerWitness": "local"
      }
   ],
   "poolRewards": "wallet_name",
   "poolPledge": "100000000000",    
   "poolCost": "340000000",
   "poolMargin": "0.01"
   "poolRelays": [
      {
      "relayType": "dns",
      "relayEntry": "relay.mypool.com",
      "relayPort": "3001"
      }
      {
      "relayType": "ip",
      "relayEntry": "x.x.x.x (ipv4 of relay)",
      "relayPort": "3002"
      }
   ],
   "poolMetaName": "This is my Pool",
   "poolMetaDescription": "This is the description of my Pool!",
   "poolMetaTicker": "POOL",
   "poolMetaHomepage": "https://mypool.com",
   "poolMetaUrl": "https://mypool.com/mypool.metadata.json",
   "poolExtendedMetaUrl": "",
   "---": "--- DO NOT EDIT BELOW THIS LINE ---"
}
```
{% endtab %}

{% tab title="extended.metadata.json" %}
```bash
{
    "info": {
        "url_png_icon_64x64": "",
        "url_png_logo": "",
        "location": "",
        "social": {
            "twitter_handle": "",
            "telegram_handle": "",
            "facebook_handle": "",
            "youtube_handle": "",
            "twitch_handle": "",
            "discord_handle": "",
            "github_handle": ""
        },
        "company": {
            "name": "",
            "city": "",
            "country": ""
                    },
        "about": {
            "me": "",
            "server": "",
	    "company": ""
        },
    "my-pool-ids": {
        "0": ""
    },
    "when-satured-then-recommend": {
        "0": ""
    }
    }
   
}
```
{% endtab %}

Run 05a_genStakepoolCert.sh pool_name again. This will generate the pool_name.pool.cert file and the pool_name.metadata.json.
Now upload the metadata.json to the URL you specified in the previous step. Do not edit it anymore or the hash will not fit!


Delegate to your own pool as owner -> pledge 

{% tab title="offline" %}
```bash
05b_genDelegationCert.sh pool_name wallet_name
```
{% endtab %}


Generate the stakepool registration transaction. The script also attaches the new pool_name.metadata.json to the offlinetransfer file

{% tab title="offline" %}
```bash
05c_regStakepoolCert.sh pool_name wallet_name.payment
``` 
{% endtab %}
