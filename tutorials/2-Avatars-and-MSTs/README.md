In this tutorial you will learn how to

* Register an Avatar
* Verify an Avatar against an address
* Customize and Issue an MST from your Avatar
* Check your MST Balances
* Transfer MST's programatically
* Integrate Avatar and MST functionality into your app


## Introduction

Avatars are digital representations of personal Identities that exist on the Metaverse Blockchain
Avatars have an alphanumeric symbol.
Avatars get attached to a Metaverse Address. There can only be one Avatar per address.
ETP and smart tokens can be sent to avatars instead of addresses.
MST's and MIT's can only be issued by Avatars

For more info: https://medium.com/metaverse-blockchain/metaverse-explained-avatars-57be355d42d4

MST's are Metaverse Smart Tokens. MST is Metaverse's fungible token standard. MST's can be seen as subcurrencies on the Metaverse Blockchain.
It costs 10 ETP to create an MST. MST's must be issued by an avatar. MST SYMBOLS MUST BE ALL CAPS!

## Hands on Tutorial

First lets create an html front end

```html
<!DOCTYPE html>
<html lang="en" dir="ltr">
  <head>
    <meta charset="utf-8">
    <title></title>
  </head>
  <body>

    <h3> Register Avatar </h3>

    <input placeholder="Avatar Name"></input>
    <label>Avatar Address</label> <select></select>
    <button>Register</button>

<button></button>
    <h3> Issue MST </h3>

    <input placeholder="symbol"></input> <br>

    <input placeholder="max_supply"></input>

    <button>Issue</button>

    <h3> Balances </h3>

    <table>
      <th>MST</th>
      <th>Issuer</th>
      <th>Amount</th>

    </table>

    <h3> Send MST </h3>

    <label></label><select></select>
    <input placeholder ="sendTo"></input><br>
    <input placeholder = "amount"></input><br>

  </body>
</html>

```

nodejs

Create a function to register your avatar.

```javascript
async function registerAvatar(avatar_name,avatar_address) {

    let change_address = avatar_address
    let height = await blockchain.height()
    let txs = await blockchain.addresses.txs([avatar_address])
    let utxos = await Metaverse.output.calculateUtxo(txs.transactions, [avatar_address]) //Get all utxo for the avatar address
    let result = await Metaverse.output.findUtxo(utxos, {}, height, 100000000) //Collect utxo to pay for the fee of 1 ETP
    let tx = await Metaverse.transaction_builder.issueDid(result.utxo, avatar_address, avatar_name, change_address, result.change, 80000000, 'testnet')
    tx= await wallet.sign(tx)
    tx = await tx.encode()
    tx = await tx.toString('hex')
    tx = await blockchain.transaction.broadcast(tx)
}

```

Create a function to look up an avatar address

```
let avatarInfo = await blockchain.avatar.get(avatar)
return avatarInfo.address
```


Create a function to issue an MST.


```javascript

async function issueMST(issuer,symbol,max_supply,decimalPrecision,description){

  let issuingAddress = await getAvatar(issuer)
  let recipient_address = issuingAddress;
  let change_address = issuingAddress;

  let height = await blockchain.height()
  console.log(issuingAddress)

  let txs = await blockchain.addresses.txs(wallet.getAddresses())
  let utxos = await Metaverse.output.calculateUtxo(txs.transactions, wallet.getAddresses()) //Get all utxo
  let result = await Metaverse.output.findUtxo(utxos, {}, height, 1000000000) //Collect utxo to pay for the fee of 10 ETP
  let tx = await Metaverse.transaction_builder.issueAsset(result.utxo, recipient_address, symbol, max_supply, precision, issuer, description, 0,false, change_address, result.change,true,0,'testnet',null,undefined)
  tx = await wallet.sign(tx)
  tx = await tx.encode()
  tx = await tx.toString('hex')
  tx = await blockchain.transaction.broadcast(tx)

  console.log(tx);

}

```

Create a function to get MST balance

```javascript

async function getBalances(){
  let wallet = await Metaverse.wallet.fromMnemonic(mnemonic, 'testnet')
  let height = await blockchain.height()
  let addresses = wallet.getAddresses()
  let txs = await blockchain.addresses.txs(addresses)

  let utxo = await Metaverse.output.calculateUtxo(txs.transactions, wallet.getAddresses())
  let balances = await blockchain.balance.all(utxo, wallet.getAddresses(), height)
  console.log(balances.MST)

}
```


Create a function to transfer MST's


```javascript

async function transferMST(amount,recipient_address,MSTSymbol) {

  amount = parseInt(amount)
  recipient_address = await getAvatar(recipient_address)


  let target = {};
  target[MSTSymbol] = amount
  console.log(target)

  change_address = wallet.getAddress(0)
  let height = await blockchain.height()
  let txs = await blockchain.addresses.txs(wallet.getAddresses())
  let utxos = await Metaverse.output.calculateUtxo(txs.transactions, wallet.getAddresses()) //Get all utxo
  let result = await Metaverse.output.findUtxo(utxos, target, height) //Collect utxo for given target
  let tx = await Metaverse.transaction_builder.send(result.utxo, recipient_address, undefined, target, change_address, result.change)
  tx = await wallet.sign(tx)
  tx = await tx.encode()
  tx = await blockchain.transaction.broadcast(tx.toString('hex'))
  console.log(tx)

}

```

### Connect to your dApp

To interact with metaversejs in your webapp, you need to reference metaversejs in your HTML.

```html

<script type="text/javascript" src="/dist/metaverse.min.js"></script>

```

Also reference your tut2.js file.

```html

<script type="text/javascript" src="tut2.js"></script>

```

Verify that you have connected metaverse to the webapp by opening the browser console and typing "Metaverse". You should see the Metaverse object come up and look something like



Next connect elements to the js functions and youre done!
