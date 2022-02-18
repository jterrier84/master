# Update Pools Registration Certificate

Query the chain for protocols, store them in a file named params.json.


{% tabs %}
{% tab title="Core" %}

```bash
cd ${NODE_HOME}
cardano-cli query protocol-parameters \
  --${CONFIG_NET} \
  --out-file $NODE_HOME/params.json
```

{% endtab %}
{% endtabs %}

If you plan to edit your poolMetaData.json file do so now, run the below command and move poolMetaDataHash.txt to your cold machine. If your poolMetaData.json is not being edited head over to your cold machine.

{warning}
{% tabs %}
{% tab title="Core" %}

```bash
cardano-cli stake-pool metadata-hash \
  --pool-metadata-file poolMetaData.json > poolMetaDataHash.txt
```

{% endtab %}
{% endtabs %}

## Create a new pool certificate

{% tabs %}
{% tab title="Cold Offline" %}

{% hint style="danger" %}
--metadata-url must be 64 characters or less.
{% endhint %}

 Open or create a file name registration-cert.txt. Use this file to edit the below command before you issue it. It's also handy to leave this file on the cold machine for any future edits. Below is 1,000 ada pledge, 340 cost and a 1% margin. Refer back to the core guide if you are using more than one relay.

 ```bash
 cd ${NODE_HOME}
 nano registration-cert.txt
 ```

```bash
cardano-cli stake-pool registration-certificate \
  --cold-verification-key-file ${HOME}/cold-keys/node.vkey \
  --vrf-verification-key-file vrf.vkey \
  --pool-pledge 10000000000 \
  --pool-cost 340000000 \
  --pool-margin 0.01 \
  --pool-reward-account-verification-key-file stake.vkey \
  --pool-owner-stake-verification-key-file stake.vkey \
  --${CONFIG_NET} \
  --single-host-pool-relay <r1.example.com> \
  --pool-relay-port 3001 \
  --metadata-url <https://example.com/poolMetaData.json> \
  --metadata-hash $(cat poolMetaDataHash.txt) \
  --out-file pool.cert
```

{% endtab %}
{% endtabs %}

Once you are satisfied with your edits copy the command, save the file and issue it in your terminal.

Issue a delegation certificate from **stake.skey** & **node.vkey**.

{% tabs %}
{% tab title="Cold Offline" %}

```bash
cardano-cli stake-address delegation-certificate \
  --stake-verification-key-file stake.vkey \
  --cold-verification-key-file ${HOME}/cold-keys/node.vkey \
  --out-file deleg.cert
```

{% endtab %}
{% endtabs %}

Move **pool.cert**, **deleg.cert** to your online core machine.

Query the current slot number or tip of the chain.

{% tabs %}
{% tab title="Core" %}

```bash
slotNo=$(cardano-cli query tip --${CONFIG_NET} | jq -r '.slot')
echo slotNo: ${slotNo}
```

{% endtab %}
{% endtabs %}

Query the wallets utxo history and build variables for a transaction.

{% tabs %}
{% tab title="Core" %}

```bash
cardano-cli query utxo \
  --address $(cat payment.addr) \
  --${CONFIG_NET} > fullUtxo.out

tail -n +3 fullUtxo.out | sort -k3 -nr > balance.out
cat balance.out
tx_in=""
total_balance=0

while read -r utxo; do
  in_addr=$(awk '{ print $1 }' <<< "${utxo}")
  idx=$(awk '{ print $2 }' <<< "${utxo}")
  utxo_balance=$(awk '{ print $3 }' <<< "${utxo}")
  total_balance=$((${total_balance}+${utxo_balance}))
  echo TxHash: ${in_addr}#${idx}
  echo Lovelace: ${utxo_balance}
  tx_in="${tx_in} --tx-in ${in_addr}#${idx}"
done < balance.out
txcnt=$(cat balance.out | wc -l)
echo Total ADA balance: $((${total_balance} / 1000000))
echo Number of UTXOs: ${txcnt}
```

{% endtab %}
{% endtabs %}

Build temporary **tx.tmp** to hold information while we build our raw transaction file.

{% tabs %}
{% tab title="Core" %}

```bash
cardano-cli transaction build-raw \
  ${tx_in} \
  --tx-out $(cat payment.addr)+$(( ${total_balance} - ${stakePoolDeposit}))  \
  --invalid-hereafter $(( ${slotNo} + 10000)) \
  --fee 0 \
  --certificate-file pool.cert \
  --certificate-file deleg.cert \
  --out-file tx.tmp
```

{% endtab %}
{% endtabs %}

Calculate the transaction fee.

{% tabs %}
{% tab title="Core" %}

```bash
fee=$(cardano-cli transaction calculate-min-fee \
  --tx-body-file tx.tmp \
  --tx-in-count ${txcnt} \
  --tx-out-count 1 \
  --${CONFIG_NET} \
  --witness-count 3 \
  --byron-witness-count 0 \
  --protocol-params-file params.json | awk '{ print $1 }')
  echo fee: ${fee}
```

{% endtab %}
{% endtabs %}

Calculate output that comes back to you (change).

{% tabs %}
{% tab title="Core" %}

```bash
txOut=$((${total_balance}-${stakePoolDeposit}-${fee}))
echo txOut: ${txOut}
```

{% endtab %}
{% endtabs %}

Build your **tx.raw** (unsigned) transaction file.

{% tabs %}
{% tab title="Core" %}

```bash
cardano-cli transaction build-raw \
  ${tx_in} \
  --tx-out $(cat payment.addr)+${txOut} \
  --invalid-hereafter $(( ${slotNo} + 10000)) \
  --fee ${fee} \
  --certificate-file pool.cert \
  --certificate-file deleg.cert \
  --out-file tx.raw
```

{% endtab %}
{% endtabs %}

Move **tx.raw** to your cold offline machine.

Sign the transaction with your **payment.skey**, **node.skey** & **stake.skey**.

{% tabs %}
{% tab title="Cold Offline" %}

```bash
cardano-cli transaction sign \
  --tx-body-file tx.raw \
  --signing-key-file payment.skey \
  --signing-key-file ${HOME}/cold-keys/node.skey \
  --signing-key-file stake.skey \
  --${CONFIG_NET} \
  --out-file tx.signed
```

{% endtab %}
{% endtabs %}

Move **tx.signed** back to your core node & submit the transaction to the blockchain.

{% tabs %}
{% tab title="Core" %}

```bash
cardano-cli transaction submit \
  --tx-file tx.signed \
  --${CONFIG_NET}
```

{% endtab %}
{% endtabs %}

If you lower your pledge you need to wait two epochs before you can remove the ada or your pledge will show as unmet and you will not forge blocks.
