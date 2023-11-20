# Hydra Poll dApp

Hydra-Poll was a quick attempt to have an independent frontend working with the
opened Hydra Head.

What we wanted is to have users build and sign Hydra transactions on the frontend.
There is a live Hydra Head between two participants and the frontend connects to it
using websocket protocol.

So how can any user sign and submit a Hydra transaction if they are not part of
the Head I hear you ask?

What we did is commit a script output into a Head. The script is very simple
and its validator always returns True so anybody can spend this utxo over and
over again. You don't actually need to sign the transaction but we wanted to
add some interactivity to the app. Don't worry the fees are zero so you don't
need to pay anything thanks to the altered protocol parameters in the Head.

One problem we had and this is where the flexibility of Hydra really shines is
that normally ledger rules will prevent you from providing script output as
collateral input. Since Hydra uses it's own local ledger which you can tweak to
your needs we were able to alter it and remove this rule so that we can
actually use the same script utxo we are spending in the Head also as the
collateral input!

We couldn't find an easy way to trick MeshJS into not requiring collateral
input since normally when you want to spend from a script collateral input must
be provided (we are working with their team to have a bit more flexible rules
when it comes to building transactions).

In the end the app is really simple but might be of use to someone who wants to
start to build on Hydra Head protocol.

By the way we opened the necessary ports on our server so anybody can connect
to this Hydra Head instance from anywhere and run this frontend or even better
go wild and build something a bit different.

What we currently do is put number 1,2 or 3 in the metadata. These numbers
represent three voting options. When clicking the vote button we send out a
new Hydra transaction and then observe `TxValid` Hydra message over websocket.
When we detect it, it means the transaction is valid and applied to Hydra's
local ledger and then the counter is incremented. You could alter the metadata
to include something you would like and build a completely different frontend!

### How to run the backend 

You need to clone `hydra-node` from
(here)[https://github.com/input-output-hk/hydra] 

and then switch to a `hydra-poll` branch since that
one contains altered ledger rules to allow transaction collateral coming from a
script utxo. This was mentioned in the text above already. 

Take a look at how to run the demo from the official
(documentation)[https://hydra.family/head-protocol/docs/getting-started/demo/].


You can choose the option to run the demo with or without docker depending on
your preference and you can also decide how many participants the Head will
have. To keep things minimal we chose to have only one participant in the Head
protocol - Alice.

This is how we run alice's hydra-node after starting cardano-node and funding the participant address:

```
source .env && hydra-node \
  --node-id 1 --port 5001 --api-port 4001 --monitoring-port 6001 \
  --hydra-signing-key alice.sk \
  --hydra-scripts-tx-id $HYDRA_SCRIPTS_TX_ID \
  --cardano-signing-key devnet/credentials/alice.sk \
  --ledger-protocol-parameters devnet/protocol-parameters.json \
  --testnet-magic 42 \
  --node-socket devnet/node.socket \
  --persistence-dir devnet/persistence/alice
```

After you initialize the Head (you can do that using the TUI) we need to commit
a script output to a head. It is important to have the simple script that
_always returns True_ since we want this output to be spendable by anyone.
There is a script like that already present in this repo in the `.env` file 

```
{
    "type": "PlutusScriptV2",
    "description": "",
    "cborHex": "59064559064201000032323232332222253353332221220023333573466e1cd55ce9baa0034800080208c98c8028cd5ce0038040041999ab9a3370e6aae74dd5001240004010464c6401466ae7001c020020c8cccd5cd19b8735573a0029000119191919191919910919800801801191999ab9a3370e6aae74005200023232323232323232323232323232323232323232323232323232323333332222221233333333333300100701200600500400e00d00c00300a0020083300d2323333573466e1cd55ce800a4000464646466442466002006004604c00460280026ae84d5d128011aba15001135573c004464c6406266ae700b80bc0bcdd5000806198068070051998083ae500f00933301075ca01e0106601aeb8010ccc041d710008011aba135744a0206ae85403cd5d0a8079aba1500f35742a01e6ae85403cd5d0a8079aba1500f35742a01e6ae85403cd5d0a8079aba1500f2322300237580026044446666aae7c00480848cd4080c010d5d080118019aba20020222323333573466e1cd55ce800a4000464646464646464646666444424666600200a008006004646666ae68cdc39aab9d001480008c8c8c8cc8848cc00400c008c090008cc02808c004d5d09aba2500235742a00226aae780088c98c80b4cd5ce0150158159baa00433300d75ca01800664646666ae68cdc3800a4008464244460040086ae84d55cf00191999ab9a3370e0049001119091118008021bae357426aae780108cccd5cd19b87003480008488800c8c98c80c0cd5ce0168170170168161aab9d00137540046600aeb8004d5d09aba2500535742a0086ae854010d5d0a8021119191999ab9a3370e002900011919091180100198030009aba135573c00646666ae68cdc3801240044244002464c6405866ae700a40a80a80a4d55ce8009baa001135744a00226ae8940044d55cf00111931901199ab9c0200210213754002266002eb9d69119118011bab00130202233335573e002403e46466a03e66442466002006004600c6aae754004c014d55cf280098021aba200313574200404026ae8940044d5d1280089aba25001135744a00226ae8940044d5d1280089aba25001135744a00226ae8940044d5d1280089aab9e0022326320133357380200220226ea8008c8c8cccd5cd19b87001480188c848888c010014c8c8cccd5cd19b870014803084888888800c8c8c8c8cccd5cd19b87005480288488888880108cccd5cd19b87006480208c8c8cc8848888888cc004024020dd70011bad001357426ae894010d5d0a80191999ab9a3370e00e900311919199109111111198010048041bae002375c0026ae84d5d128031aba1500523333573466e1c021200423232332212222222330060090083013002375c0026ae84d5d128041aba1500723333573466e1c0252002232321222222230070083013001357426aae7802c8cccd5cd19b8700a480008c8c848888888c014020c050004d5d09aab9e00c23263202033573803a03c03c03a03803603403226aae780144d55cf00209aab9e00301535573a0026ea8d5d09aab9e00323333573466e1c0092004232321222230020053009001357426aae780108cccd5cd19b87003480088c8c848888c004014c024004d5d09aab9e00523333573466e1c0112000232122223003005375c6ae84d55cf00311931900b99ab9c01401501501401301235573a0026ea8004d5d09aba2500535742a0084646666ae68cdc39aab9d001480008c8c8c8cc8848cc00400c008c8cccd5cd19b8735573a002900011bae357426aae780088c98c8058cd5ce00980a00a1baa002375a0026ae84d5d128011aba15001135573c004464c6402266ae7003803c03cdd5000919191999ab9a3370e0029001119191919191999110911998008028020019bad003375a0046eb4004d5d09aba2500335742a0046ae8540084d5d1280089aab9e00323333573466e1c00920002323212230020033007001357426aae780108c98c8048cd5ce0078080080079aab9d0013754002464646666ae68cdc3800a400446424460020066eb8d5d09aab9e00323333573466e1c00920002321223002003375c6ae84d55cf00211931900899ab9c00e00f00f00e35573a0026ea80044d55cf00111931900599ab9c0080090093754002200e264c6401266ae7124103505435000071220021221223300100400349010350543100120014988c8c00400488cc00cc0080080041"
}

```
Let's save this in a always-succeeds.plutus file in the demo folder.

We can use cardano-cli to _publish_ this smart contract to our local running devnet.

Let's try to find out the script address first:

```
cardano-cli address build \
  --payment-script-file always-succeeds.plutus \
  --testnet-magic 42 \
  --out-file always-succeeds.addr

```
This should yield `addr_test1wrrjcfmvzn3tjuwa92yagf079lfh7xfpawak47jxjmjm62snxuzpf`

NOTE: 
The script datum and redeemer is just the unit value `()`. Useful information is that the cbor encoding of a `()` is `d87980` and it's hex encoded cbor string is `923918e403bf43c34b4ef6b48eb2ee04babed17320d8d1b9ff9ad086e86f44ec`. We will need this later when drafting a commit transaction.


We will also need some wallet key not known to the running hydra-node to commit this script so let's use Alice's external key. Let's take a note of this address:

```

cardano-cli address build \
  --payment-verification-key-file devnet/credentials/alice-funds.vk \
  --testnet-magic 42 \
  --out-file alice.addr

```

Alice's address should be: `addr_test1vp5cxztpc6hep9ds7fjgmle3l225tk8ske3rmwr9adu0m6qchmx5z`

Let's query Alice's wallet UTxO's to have them available for the transaction inputs in the cardano-cli:

```
cardano-cli query utxo --address $(cat alice.addr) --socket-path devnet/node.socket --testnet-magic 42

```
The output will, of course, vary on each devnet run but currently it looks like this:

```
                           TxHash                                 TxIx        Amount
--------------------------------------------------------------------------------------
d8df0d83b5f417d84814dc13e88bbd82e8b1f3102cb1b399e56f6c28b738ab25     0        100000000 lovelace + TxOutDatumNone

```
In order to construct the correct inline datum the plutus script needs we will create a file called datum.json and 
put the json encoded datum inside:

```
{"constructor":0,"fields":[]}

```

Now let's use this output to build the transaction, please make sure to use the correct amount of lovelace:

```
cardano-cli transaction build \
  --tx-in d8df0d83b5f417d84814dc13e88bbd82e8b1f3102cb1b399e56f6c28b738ab25#0 \
  --tx-out $(cat always-succeeds.addr)+2000000 \
  --change-address $(cat alice.addr) \
  --tx-out-inline-datum-file datum.json  \
  --testnet-magic 42 \
  --socket-path devnet/node.socket \
  --out-file tx-script.build
```

We need to sign the transaction:


```

cardano-cli transaction sign \
   --tx-body-file tx-script.build \
   --signing-key-file devnet/credentials/alice-funds.sk \
   --testnet-magic 42 \
   --out-file tx-script.signed

```

And finally submit it to the network:

```
cardano-cli transaction submit --testnet-magic 42 --socket-path devnet/node.socket --tx-file tx-script.signed

```
If we query script address we should see a UTxO with a datum hash:

```

cardano-cli query utxo --address $(cat always-succeeds.addr) --socket-path devnet/node.socket --testnet-magic 42

                           TxHash                                 TxIx        Amount
--------------------------------------------------------------------------------------
c9ca66210a33173f7731b351dc341df328bb776307502f46c38dd4fa487a443a     0        2000000 lovelace + TxOutDatumInline ReferenceTxInsScriptsInlineDatumsInBabbageEra (HashableScriptData "\216y\128" (ScriptDataConstructor 0 []))

```

Now it is time to ask our hydra-node to draft a commit transaction using this UTxO. 
For this we can use curl or any other tool capable of sending network requests:

```
curl -i     -H "Accept: application/json" -H "Content-Type: application/json" -X POST -d '{"c9ca66210a33173f7731b351dc341df328bb776307502f46c38dd4fa487a443a#0":{"address":"addr_test1wrrjcfmvzn3tjuwa92yagf079lfh7xfpawak47jxjmjm62snxuzpf","datum":null,"inlineDatum":{"constructor":0,"fields":[]},"inlineDatumhash":"923918e403bf43c34b4ef6b48eb2ee04babed17320d8d1b9ff9ad086e86f44ec","referenceScript":null,"value":{"lovelace":2000000}, "witness": { "plutusV2Script": { "cborHex": "59064559064201000032323232332222253353332221220023333573466e1cd55ce9baa0034800080208c98c8028cd5ce0038040041999ab9a3370e6aae74dd5001240004010464c6401466ae7001c020020c8cccd5cd19b8735573a0029000119191919191919910919800801801191999ab9a3370e6aae74005200023232323232323232323232323232323232323232323232323232323333332222221233333333333300100701200600500400e00d00c00300a0020083300d2323333573466e1cd55ce800a4000464646466442466002006004604c00460280026ae84d5d128011aba15001135573c004464c6406266ae700b80bc0bcdd5000806198068070051998083ae500f00933301075ca01e0106601aeb8010ccc041d710008011aba135744a0206ae85403cd5d0a8079aba1500f35742a01e6ae85403cd5d0a8079aba1500f35742a01e6ae85403cd5d0a8079aba1500f2322300237580026044446666aae7c00480848cd4080c010d5d080118019aba20020222323333573466e1cd55ce800a4000464646464646464646666444424666600200a008006004646666ae68cdc39aab9d001480008c8c8c8cc8848cc00400c008c090008cc02808c004d5d09aba2500235742a00226aae780088c98c80b4cd5ce0150158159baa00433300d75ca01800664646666ae68cdc3800a4008464244460040086ae84d55cf00191999ab9a3370e0049001119091118008021bae357426aae780108cccd5cd19b87003480008488800c8c98c80c0cd5ce0168170170168161aab9d00137540046600aeb8004d5d09aba2500535742a0086ae854010d5d0a8021119191999ab9a3370e002900011919091180100198030009aba135573c00646666ae68cdc3801240044244002464c6405866ae700a40a80a80a4d55ce8009baa001135744a00226ae8940044d55cf00111931901199ab9c0200210213754002266002eb9d69119118011bab00130202233335573e002403e46466a03e66442466002006004600c6aae754004c014d55cf280098021aba200313574200404026ae8940044d5d1280089aba25001135744a00226ae8940044d5d1280089aba25001135744a00226ae8940044d5d1280089aab9e0022326320133357380200220226ea8008c8c8cccd5cd19b87001480188c848888c010014c8c8cccd5cd19b870014803084888888800c8c8c8c8cccd5cd19b87005480288488888880108cccd5cd19b87006480208c8c8cc8848888888cc004024020dd70011bad001357426ae894010d5d0a80191999ab9a3370e00e900311919199109111111198010048041bae002375c0026ae84d5d128031aba1500523333573466e1c021200423232332212222222330060090083013002375c0026ae84d5d128041aba1500723333573466e1c0252002232321222222230070083013001357426aae7802c8cccd5cd19b8700a480008c8c848888888c014020c050004d5d09aab9e00c23263202033573803a03c03c03a03803603403226aae780144d55cf00209aab9e00301535573a0026ea8d5d09aab9e00323333573466e1c0092004232321222230020053009001357426aae780108cccd5cd19b87003480088c8c848888c004014c024004d5d09aab9e00523333573466e1c0112000232122223003005375c6ae84d55cf00311931900b99ab9c01401501501401301235573a0026ea8004d5d09aba2500535742a0084646666ae68cdc39aab9d001480008c8c8c8cc8848cc00400c008c8cccd5cd19b8735573a002900011bae357426aae780088c98c8058cd5ce00980a00a1baa002375a0026ae84d5d128011aba15001135573c004464c6402266ae7003803c03cdd5000919191999ab9a3370e0029001119191919191999110911998008028020019bad003375a0046eb4004d5d09aba2500335742a0046ae8540084d5d1280089aab9e00323333573466e1c00920002323212230020033007001357426aae780108c98c8048cd5ce0078080080079aab9d0013754002464646666ae68cdc3800a400446424460020066eb8d5d09aab9e00323333573466e1c00920002321223002003375c6ae84d55cf00211931900899ab9c00e00f00f00e35573a0026ea80044d55cf00111931900599ab9c0080090093754002200e264c6401266ae7124103505435000071220021221223300100400349010350543100120014988c8c00400488cc00cc0080080041", "description": "", "type": "PlutusScriptV2" }, "redeemer": "d87980" }}}'     http://localhost:4001/commit

```

Hydra node successfully returnes this transaction to us:

```

{"cborHex":"84a70083825820604cbd5e2609f2de81c7e9934f0467122a33c2bdbe944653103d517c0b973d0b01825820604cbd5e2609f2de81c7e9934f0467122a33c2bdbe944653103d517c0b973d0b02825820c9ca66210a33173f7731b351dc341df328bb776307502f46c38dd4fa487a443a000d81825820604cbd5e2609f2de81c7e9934f0467122a33c2bdbe944653103d517c0b973d0b021281825820469ac87d0cfcbdebdeb4d46efea3dfaa02d197c9f70c5f0dd1728f51fe7056f3000182a300581d70171a1e6bdbc8aa96d957a65b3f505517386af06ba265e3f784741f6701821a003d0900a1581c5328ed7bd729eb0e2e6a118423e897ce4c77bc851bf15e39e3acecb7a1581cf8a68cd18e59a6ace848155a0e967af64f4d00cf8acee8adc95a6b0d01028201d81858b7d8799f5820b37aabd81024c043f53a069c91e51a5b52e4ea399ae17ee1fe3cb9c44db707eb9fd8799fd8799fd8799f5820c9ca66210a33173f7731b351dc341df328bb776307502f46c38dd4fa487a443aff00ff5840d8799fd8799fd87a9f581cc72c276c14e2b971dd2a89d425fe2fd37f1921ebbb6afa4696e5bd2affd87a80ffa140a1401a001e8480d87b9fd87980ffd87a80ffffff581c5328ed7bd729eb0e2e6a118423e897ce4c77bc851bf15e39e3acecb7ffa200581d60f8a68cd18e59a6ace848155a0e967af64f4d00cf8acee8adc95a6b0d011a012109c0021a0035d8600e81581cf8a68cd18e59a6ace848155a0e967af64f4d00cf8acee8adc95a6b0d0b582097247407d626ea711ae18fbc51eee7ee0e53ddd7dfaa80264b3377fa87a38ab2a30081825820eb94e8236e2099357fa499bfbc415968691573f25ec77435b7949f5fdfaa5da05840e31a2ad066ac2f20e7f1509d8b229a3f91c1bb054963d9e6ec94fe8cb85b95e74a23814e66d607e06fadce343b1930b7c64c8349507a816021d03f33528d4d0b068159064559064201000032323232332222253353332221220023333573466e1cd55ce9baa0034800080208c98c8028cd5ce0038040041999ab9a3370e6aae74dd5001240004010464c6401466ae7001c020020c8cccd5cd19b8735573a0029000119191919191919910919800801801191999ab9a3370e6aae74005200023232323232323232323232323232323232323232323232323232323333332222221233333333333300100701200600500400e00d00c00300a0020083300d2323333573466e1cd55ce800a4000464646466442466002006004604c00460280026ae84d5d128011aba15001135573c004464c6406266ae700b80bc0bcdd5000806198068070051998083ae500f00933301075ca01e0106601aeb8010ccc041d710008011aba135744a0206ae85403cd5d0a8079aba1500f35742a01e6ae85403cd5d0a8079aba1500f35742a01e6ae85403cd5d0a8079aba1500f2322300237580026044446666aae7c00480848cd4080c010d5d080118019aba20020222323333573466e1cd55ce800a4000464646464646464646666444424666600200a008006004646666ae68cdc39aab9d001480008c8c8c8cc8848cc00400c008c090008cc02808c004d5d09aba2500235742a00226aae780088c98c80b4cd5ce0150158159baa00433300d75ca01800664646666ae68cdc3800a4008464244460040086ae84d55cf00191999ab9a3370e0049001119091118008021bae357426aae780108cccd5cd19b87003480008488800c8c98c80c0cd5ce0168170170168161aab9d00137540046600aeb8004d5d09aba2500535742a0086ae854010d5d0a8021119191999ab9a3370e002900011919091180100198030009aba135573c00646666ae68cdc3801240044244002464c6405866ae700a40a80a80a4d55ce8009baa001135744a00226ae8940044d55cf00111931901199ab9c0200210213754002266002eb9d69119118011bab00130202233335573e002403e46466a03e66442466002006004600c6aae754004c014d55cf280098021aba200313574200404026ae8940044d5d1280089aba25001135744a00226ae8940044d5d1280089aba25001135744a00226ae8940044d5d1280089aab9e0022326320133357380200220226ea8008c8c8cccd5cd19b87001480188c848888c010014c8c8cccd5cd19b870014803084888888800c8c8c8c8cccd5cd19b87005480288488888880108cccd5cd19b87006480208c8c8cc8848888888cc004024020dd70011bad001357426ae894010d5d0a80191999ab9a3370e00e900311919199109111111198010048041bae002375c0026ae84d5d128031aba1500523333573466e1c021200423232332212222222330060090083013002375c0026ae84d5d128041aba1500723333573466e1c0252002232321222222230070083013001357426aae7802c8cccd5cd19b8700a480008c8c848888888c014020c050004d5d09aab9e00c23263202033573803a03c03c03a03803603403226aae780144d55cf00209aab9e00301535573a0026ea8d5d09aab9e00323333573466e1c0092004232321222230020053009001357426aae780108cccd5cd19b87003480088c8c848888c004014c024004d5d09aab9e00523333573466e1c0112000232122223003005375c6ae84d55cf00311931900b99ab9c01401501501401301235573a0026ea8004d5d09aba2500535742a0084646666ae68cdc39aab9d001480008c8c8c8cc8848cc00400c008c8cccd5cd19b8735573a002900011bae357426aae780088c98c8058cd5ce00980a00a1baa002375a0026ae84d5d128011aba15001135573c004464c6402266ae7003803c03cdd5000919191999ab9a3370e0029001119191919191999110911998008028020019bad003375a0046eb4004d5d09aba2500335742a0046ae8540084d5d1280089aab9e00323333573466e1c00920002323212230020033007001357426aae780108c98c8048cd5ce0078080080079aab9d0013754002464646666ae68cdc3800a400446424460020066eb8d5d09aab9e00323333573466e1c00920002321223002003375c6ae84d55cf00211931900899ab9c00e00f00f00e35573a0026ea80044d55cf00111931900599ab9c0080090093754002200e264c6401266ae7124103505435000071220021221223300100400349010350543100120014988c8c00400488cc00cc00800800410582840000d87a9f9fd8799fd8799f5820c9ca66210a33173f7731b351dc341df328bb776307502f46c38dd4fa487a443aff00ffffff821a00ac03371b00000001d5cf18dd840002d87980821a00299c481a7e3ccb22f5f6","description":"Hydra commit transaction","type":"Tx BabbageEra"}

```

NOTE: Currently cardano-cli version we are using `8.1.2` is not inline with the hydra-node when it comes to `cardano-api` package. This is unfortunate since the transaction returned by the hydra-node cannot be plugged in the cardano-cli as is.
Discrepancy is in the `type` of a transaction and we need to manually alter the type when saving to file to be "Witnessed Tx BabbageEra" instead of just "Tx BabbageEra".

Let's save this transaction as a `tx.raw` file and make sure to change the type to "Witnessed Tx BabbageEra".
Now we need to sign and submit this transaction using Alice's secret key, similar to what we did before when starting our smart contract:

NOTE: cardano-cli for some reason overrides the transaction witnesses! Normally, hydra-node would sign the commit transaction using it's internal key so we would 
just need to add Alice's signature but perhaps there is a bug in cardano-cli and we need to sign again using the same key?

```

cardano-cli transaction sign \
   --tx-body-file tx.raw \
   --signing-key-file devnet/credentials/alice.sk \
   --signing-key-file devnet/credentials/alice-funds.sk \
   --testnet-magic 42 \
   --out-file tx.signed

```

And submit it to the network:

```

cardano-cli transaction submit --testnet-magic 42 --socket-path devnet/node.socket --tx-file tx.signed
```

### How to run the frontend

This repo contains the frontend part of the `Hydra Poll`. It is built using MeshJS
and the first thing to do to run the app is to install dependencies.
This can be done using `npm` or `yarn`:

```
$ npm i
```

or:

```
$ yarn
```

After the dependencies are installed we need to build the project:

```
$ npm run build

```

If the frontend is successfully built you can invoke:

```
$ npm run start

```

for the development version you can run:

```
$ npm run dev
```

and the frontend will start on default port 3000.

You need to make sure to actually have the backend to connect to and that is covered in the above section.
