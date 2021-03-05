# Wallet-provider-interface

Typescritp Interface definition for a wallet-provider

## Assumptions

* For use in a WebApp UI for blockchain smart contracts

In order to keep the providers as simple as possible, key management, contract ABI, Tx composition and other functionality are intentionally kept outside of the scope of a wallet provider, but can be added by other supporting libraries.

## Functionalities

### Sign Message [required] 

The provider receives a message string and returns a signed message. 

### View-Call (optional)

The provider queries general data from the blockchain, or calls a function in a smart contract that does not require the tx to be signed.

### Send Tx (Sign & Send) [optional]

The provider receives a transaction, signs the Tx, send it to the blockchain and returns: `Tx-hash` & `status` to the webApp. Providers can be "complete-tx-flow" vs "pending-tx-flow". Providers with "complete-tx-flow" capabilities always return status "completed" (when the blockchain finality is under 10 secs, e.g. NEAR). Providers with "pending-tx-flow" capabilities return a tx-hash and status "pending". In this case, the webApp must periodically use an `view-call` to check the tx status.

### Login/Logout [required]

The provider receives the smart contract account id, and perform login/logout actions depending on the walllet. For some wallet/connection mechanisms no actions might be needed.

## Capabilities

### Transport

* **Sign**: The provider can sign messages. (the webapp must create the tx-msg, send it to the chain after signature, and manage responses)
* **Build-Tx**: The provider can compose a message from a batch-transacion. (the webapp doesn't need to know how to encode the tx-msg)
* **Sign-and-Send**: The provider can compose a message from a batch-transacion, sign it and send it. (off-loads from the webapp the need to create the tx-msg and all chain communication). **Sign-and-Send** implies the provider handles all communication with the blockchain.

### Finality
* **complete-tx-flow**: The blockchain finality is under 10 secs. Sign-and-Send always return tx-hash + status:completed + err|data 
* **pending-tx-flow**: The blockchain finality is above 10 secs. Sign-and-Send always return tx-hash + status:pending + err|data

### Communication
* **redirect**: The wallet is a Web wallet. Some functions return Promises, some others can cause a navigation to the Wallet website. The webapp must have a callback-url to receive responses. (NEAR Web Wallet)
* **browser-extension**: The wallet uses a browser-extension, the provider uses in-browser channels to talk to the wallet and to relay responses to the webapp. All provider functions return Promises. (Narwallets)
* **qr-code**: The wallet is a mobile wallet. Actions cause a QR-code to appear. ??????? to relay responses to the webapp. Promises??? (e.g Webconnect)
 
## Transactions

* To provide a for the common single-call use case the provider can execute single items, but internally all transactions are BatchTransaction.
* Wallet-providers with **Sign-Tx** capabilities have functions to create transactions items
 * * `provider.viewItem(contract_id:string, method:string, args:Record<string,any> : TxItem`
 * * `provider.callItem(contract_id:string, method:string, args:Record<string,any>, gas:number, attachedNativeToken:numer) : TxItem`

Example, single call:
```
 const result = await provider.view('contract.near','get_owner_id',{})
 const result2 = await provider.call('contract.near','sell',{amount:"50000000000000000000000000"})
```

Example, Batch Transaction:
```
 let tx = provider.newBatchTx();
 tx.push(provider.callItem('contract.near','buy',{token:'stNEAR'},20,50));
 tx.push(provider.callItem('usdnear.stable.near','sell',{token:'USDNEAR'},50,0));
 const result = await provider.signAndSend(tx)
```

## References

https://gov.near.org/t/wallet-selection-sdk/215/6

https://walletconnect.org/

https://github.com/mathwallet/math-nearjs

https://github.com/Narwallets/narwallets-extension/blob/master/API-design.md

