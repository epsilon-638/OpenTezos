---
id: taquito
disable_pagination: true
title: Taquito
authors: Benjamin Pilia
---

Interactions with a Tezos blockchain can be done thanks to the Tezos CLI. 
However, it is not suitable to build Dapps, since it needs to be integrated into applications, mostly web interfaces.

Fortunately, the Tezos ecosystem offers libraries in several languages, that enable developers to build efficient dapps.
_Taquito_ is one of these libraries: it is a Typescript library developed and maintained by _ECAD Labs_.
This library offers developers all the expected interactions with the blockchain: retrieving information about a Tezos network, sending a transaction, contract origination and interactions (calling an entrypoint, fetching the storage...), delegation, metadata...

For example, all these wallets ([AirGap](https://airgap.it/), [Galleon](https://cryptonomic.tech/galleon.html), [Kukai](https://wallet.kukai.app/), [Spire](https://spirewallet.com/), [Temple](https://templewallet.com/download/) ) make use of this library to function.

A full reference is available [here](https://tezostaquito.io/docs/quick_start).

This chapter is an introduction to _Taquito_: using interactions with a deployed Raffle smart contract, the basic instructions of Taquito will be explained.

# Installation

The _Taquito_ library is broken down into several modules:
- [@taquito/taquito](https://www.npmjs.com/package/@taquito/taquito): High-level functionalities that build upon the other packages in the Tezos Typescript Library Suite.
- [@taquito/ledger-signer](https://www.npmjs.com/package/@taquito/ledger-signer): Ledger signer provider.
- [@taquito/rpc](https://www.npmjs.com/package/@taquito/rpc): Provides low-level methods, and types to invoke RPC calls from a Nomadic Tezos RPC node.
- [@taquito/utils](https://www.npmjs.com/package/@taquito/utils): Converts michelson data and types into convenient JS/TS objects.
- [@taquito/michelson-encoder](https://www.npmjs.com/package/@taquito/michelson-encoder): Converts michelson data and types into convenient JS/TS objects.
- [@taquito/michel-codec](https://www.npmjs.com/package/@taquito/michel-codec): Michelson parser/validator/formatter.
- [@taquito/local-forging](https://www.npmjs.com/package/@taquito/local-forging): Provide local forging functionality.
- [@taquito/signer](https://www.npmjs.com/package/@taquito/signer): Provide signing functionality.

The main module is `@taquito/taquito`: it will be used for most actions. The other modules are used by the `@taquito/taquito` methods, as complementary features.

Let's initialize a Typescript project and install taquito:

``` shell
$ mkdir taquito-poc
$ mkdir taquito-poc/src
$ touch taquito-poc/src/app.ts taquito-poc/main.ts
$ cd taquito-poc
$ yarn add @taquito/taquito
```

The `maint.ts` file will import an `App` class from `src/app.ts` and run it:
``` typescript
import { App } from './src/app';

new App().main();

```

Let's create the `App` class with a `main` method. We will import the `TezosToolkit` class to check if `@taquito/taquito` is indeed installed:

``` typescript
import { TezosToolkit } from '@taquito/taquito';
export class App {

    public async main() { }

}
```

Let's run it with:

``` shell
$ npx ts-node main.ts
```

It should not raise any exception. Let's start _Taquito_!

# Taquito configuration

First, we need to configure _Taquito_ with an RPC URL. To do that we will use the `TezosToolkit`: it is the "facade class that surfaces all of the libraries capability and allow its configuration". When created, it accepts an RPC url. Here, we will use the _SmartPy_ RPC url: [https://florencenet.smartpy.io/](https://florencenet.smartpy.io/)


``` typescript
// src/app.ts
import { TezosToolkit } from '@taquito/taquito';
export class App {

    private tezos: TezosToolkit;

    constructor(rpcUrl: string) {
        this.tezos = new TezosToolkit(rpcUrl);
    }

    public async main() { }
```

``` typescript
// main.ts
import { App } from './src/app';

const RPC_URL = "https://florencenet.smartpy.io/"

new App(RPC_URL).main();
```

# Interactions without an account

_Taquito_ is already ready for some actions: it can retrieve all the information about the Tezos network, the accounts, the smart contracts...

For instance, let's retrieve the balance of an account, with the `getBalance` method:
``` typescript
// src/app.ts
import { TezosToolkit } from '@taquito/taquito';
export class App {

    private tezos: TezosToolkit;

    constructor(rpcUrl: string) {
        this.tezos = new TezosToolkit(rpcUrl);
    }

    public getBalance(address: string) : void {
    this.tezos.rpc
        .getBalance(address)
        .then(balance => console.log(balance))
        .catch(e => console.log('Address not found'));
    }

    public async main() { }
```

Every interaction with the Tezos network through _Taquito_  will be handled using a `Promise`.

Let's call this method for the address: `tz1YWK1gDPQx9N1Jh4JnmVre7xN6xhGGM4uC`

``` typescript
// main.ts
import { App } from './src/app';

const RPC_URL = "https://florencenet.smartpy.io/"
const ACCOUNT_TO_CHECK = "tz1YWK1gDPQx9N1Jh4JnmVre7xN6xhGGM4uC"

new App(RPC_URL).getBalance(ACCOUNT_TO_CHECK);

```

Let's run it:
``` shell
$ npx ts-node main.ts 
BigNumber { s: 1, e: 10, c: [ 53152138122 ] }
```

## Contract data
We can also retrieve the metadata and storage of a contract.

``` typescript
// src/app.ts
import { TezosToolkit } from '@taquito/taquito';
export class App {

    private tezos: TezosToolkit;

    constructor(rpcUrl: string) {
        this.tezos = new TezosToolkit(rpcUrl);
    }

    public getBalance(address: string) : void {
    this.tezos.rpc
        .getBalance(address)
        .then(balance => console.log(balance))
        .catch(e => console.log('Address not found'));
    }


    public getContractEntrypoints(address: string) {
        this.tezos.contract
            .at(address)
            .then((c) => {
                let methods = c.parameterSchema.ExtractSignatures();
                console.log(JSON.stringify(methods, null, 2));
            })
            .catch((error) => console.log(`Error: ${error}`));
    }

    public async main() { }
```

We will use a simple `Counter` contract on Florencenet.
``` typescript
import { App } from './src/app';

const RPC_URL = "https://florencenet.smartpy.io/"
const ACCOUNT_TO_CHECK = "tz1YWK1gDPQx9N1Jh4JnmVre7xN6xhGGM4uC"
const COUNTER_CONTRACT = "KT1BEMULAGQ58C5NNdWQM3WYLjUtwgJ8X8aN"

new App(RPC_URL).getContractEntrypoints(COUNTER_CONTRACT);

```

``` shell
$ npx ts-node main.ts 
[
  [
    "decrement",
    "int"
  ],
  [
    "increment",
    "int"
  ]
]
```

# Interactions with an account

_Taquito_ can of course sign and send transactions, but it needs a private key to do that. Let's retrieve a faucet from [https://faucet.tzalpha.net/](https://faucet.tzalpha.net/) and put it in the project directory.

## Activating the account from _Taquito_

Every implicit address must be activated on a public network. Let's activate ours on Florencenet.

First, we need to install the `@taquito/signer` module, used to sign the transactions.

``` shell
$ yarn add @taquito/signer
```

We will use the `InMemorySigner`: it loads a private key in memory and uses it to sign transactions.

> Storing private keys in memory is suitable for development workflows but risky for production use-cases! Use the `InMemorySigner` appropriately given your risk profile

You can find a complete reference [here](https://tezostaquito.io/docs/inmemory_signer), and find more _signers_ [here](https://tezostaquito.io/docs/tezbridge_signer) and [here](https://tezostaquito.io/docs/ledger_signer).

First, we need to set the signer of our TezosToolkit using `setSignerProvider`. The signer will load a private key from our faucet with the `fromFundraiser` method.

``` typescript
// src/app.ts
import { TezosToolkit } from '@taquito/taquito';
import { InMemorySigner } from '@taquito/signer';
const faucet = require('../faucet.json');
export class App {

    private tezos: TezosToolkit;

    constructor(rpcUrl: string) {
        this.rpcUrl = rpcUrl
        this.tezos = new TezosToolkit(rpcUrl);
        this.tezos.setSignerProvider(InMemorySigner.fromFundraiser(faucet.email, faucet.password, faucet.mnemonic.join(' ')))
    }

    public async main() { }
```

We can now add an `activateAccount` method, with an `activate` method taking the address and the secret key as inputs.
``` typescript
// src/app.ts
import { TezosToolkit } from '@taquito/taquito';
import { InMemorySigner } from '@taquito/signer';
const faucet = require('../faucet.json');
export class App {

    private tezos: TezosToolkit;

    constructor(rpcUrl: string) {
        this.rpcUrl = rpcUrl
        this.tezos = new TezosToolkit(rpcUrl);
        this.tezos.setSignerProvider(InMemorySigner.fromFundraiser(faucet.email, faucet.password, faucet.mnemonic.join(' ')))
    }

    public async activateAccount() {
        const {pkh, secret} = faucet;

        try {
            const operation = await this.tezos.tz.activate(pkh, secret);
            await operation.confirmation();
        } catch (e) {
            console.log(e)
        }

    }

    public async main() { }
```

And call it from our `main.ts` file:
```typescript
import { App } from './src/app';

const RPC_URL = "https://florencenet.smartpy.io/"
const ACCOUNT_TO_CHECK = "tz1YWK1gDPQx9N1Jh4JnmVre7xN6xhGGM4uC"
const COUNTER_CONTRACT = "KT1BEMULAGQ58C5NNdWQM3WYLjUtwgJ8X8aN"

new App(RPC_URL).activateAccount();
```

If you take a look on an explorer ([https://florence.tzstats.com/](https://florence.tzstats.com/) for instance), you will see your activated account on it.

## Sending a transaction

Now that _Taquito_ is configured with an activated account, we can send transactions. Let's send some to another address.

Transactions can be sent with `this.tezos.contract.transfer`. It returns a `Promise<TransactionOperation>`. A `TransactionOperation` contains the information about this transaction. It also has a `confirmation` method. This method will wait for several confirmations (that can be passed as input). But, we will be notified when a transaction is confirmed.

Let's create a `sendTz` method that will send an `amount` of tz to the recipient `address`.

``` typescript
// src/app.ts
import { TezosToolkit } from '@taquito/taquito';
import { InMemorySigner } from '@taquito/signer';
const faucet = require('../faucet.json');
export class App {

    private tezos: TezosToolkit;

    constructor(rpcUrl: string) {
        this.rpcUrl = rpcUrl
        this.tezos = new TezosToolkit(rpcUrl);
        this.tezos.setSignerProvider(InMemorySigner.fromFundraiser(faucet.email, faucet.password, faucet.mnemonic.join(' ')))
    }


    public sendTz(address: string, amount: number) {

        console.log(`Transfering ${amount} ꜩ to ${address}...`);
        this.tezos.contract.transfer({ to: address, amount: amount })
            .then(op => {
                console.log(`Waiting for ${op.hash} to be confirmed...`);
                return op.confirmation(1).then(() => op.hash);
            })
            .then(hash => console.log(`${hash}`))
            .catch(error => console.log(`Error: ${error} ${JSON.stringify(error, null, 2)}`));
    }
}
```



``` typescript
import { App } from './src/app';

const RPC_URL = "https://florencenet.smartpy.io/"
const ACCOUNT_TO_CHECK = "tz1YWK1gDPQx9N1Jh4JnmVre7xN6xhGGM4uC"
const COUNTER_CONTRACT = "KT1BEMULAGQ58C5NNdWQM3WYLjUtwgJ8X8aN"
const RECIPIENT = "tz1WD73bxtb3oeBBTU471Yz5gcy9NMzepfMJ"
const AMOUNT = 10
new App(RPC_URL).sendTz(RECIPIENT,AMOUNT);

```

Let's run it
``` shell
$ npx ts-node main.ts 
Transfering 10 ꜩ to tz1WD73bxtb3oeBBTU471Yz5gcy9NMzepfMJ...
Waiting for ooYGXazAECCMTehpfsPWo6JxavJs2a5KYP6a1i6eU5ofATWnHbH to be confirmed...
ooYGXazAECCMTehpfsPWo6JxavJs2a5KYP6a1i6eU5ofATWnHbH

```

We can then check the transaction on an explorer: [https://florence.tzstats.com/ooYGXazAECCMTehpfsPWo6JxavJs2a5KYP6a1i6eU5ofATWnHbH](https://florence.tzstats.com/ooYGXazAECCMTehpfsPWo6JxavJs2a5KYP6a1i6eU5ofATWnHbH)


## Making a contract call

_Taquito_ can make contract calls. We will use a simple Counter contract. First, we need to know what are the available entrypoints. We can use the `getContractEntrypoints` defined in the [Contract data subsection](##contract-data).

Let's call the `increment` entrypoint. It takes an int as an input.

To do so, we need:
1. to get the contract: `this.tezos.contract.at(contract)`. It returns a `Promise<ContractAbstraction<ContractProvider>>`
2. get the entrypoints: The `ContractAbstraction<ContractProvider>` has a `methods` property: it contrains the entrypoints `increment` and `decrement`
3. get the increment entrypoint: `methods.increment(2)`. It increments the counter by `2`
4. send the contract call and inspect the transaction: `contract.methods.increment(i).send()` / `contract.methods.increment(i).toTransferParams()`
5. If the transaction is sent, wait for a number of confirmations (just like in a transfer transaction)


``` typescript
import { TezosToolkit } from '@taquito/taquito';
import { InMemorySigner } from '@taquito/signer';
const faucet = require('../faucet.json');
export class App {

    private tezos: TezosToolkit;

    constructor(rpcUrl: string) {
        this.rpcUrl = rpcUrl
        this.tezos = new TezosToolkit(rpcUrl);
        this.tezos.setSignerProvider(InMemorySigner.fromFundraiser(faucet.email, faucet.password, faucet.mnemonic.join(' ')))
    }


    public increment(increment: number, contract: string) {
        this.tezos.contract
            .at(contract) // step 1
            .then((contract) => {
                console.log(`Incrementing storage value by ${increment}...`);
                return contract.methods.increment(increment).send(); // steps 2, 3 and 4
            })
            .then((op) => {
                console.log(`Awaiting for ${op.hash} to be confirmed...`);
                return op.confirmation(3).then(() => op.hash); // step 5
            })
            .then((hash) => console.log(`Operation injected: https://florence.tzstats.com/${hash}`))
            .catch((error) => console.log(`Error: ${JSON.stringify(error, null, 2)}`));
    }
}
```

``` typescript
import { App } from './src/app';

const RPC_URL = "https://florencenet.smartpy.io/"
const ACCOUNT_TO_CHECK = "tz1YWK1gDPQx9N1Jh4JnmVre7xN6xhGGM4uC"
const COUNTER_CONTRACT = "KT1BEMULAGQ58C5NNdWQM3WYLjUtwgJ8X8aN"
const RECIPIENT = "tz1WD73bxtb3oeBBTU471Yz5gcy9NMzepfMJ"
const AMOUNT = 10
const INCREMENT = 5
new App(RPC_URL).increment(INCREMENT, COUNTER_CONTRACT);

```

The `send()` function can take  an object with fields as an input, such as `amount` (which defines an amount to be sent with the contract call), `storageLimit`...


## Sending several transactions

Let's consider this app:

``` typescript
// src/app.ts
import { TezosToolkit } from '@taquito/taquito';
import { InMemorySigner } from '@taquito/signer';
const faucet = require('../faucet.json');
export class App {

    private tezos: TezosToolkit;

    constructor(rpcUrl: string) {
        this.rpcUrl = rpcUrl
        this.tezos = new TezosToolkit(rpcUrl);
        this.tezos.setSignerProvider(InMemorySigner.fromFundraiser(faucet.email, faucet.password, faucet.mnemonic.join(' ')))
    }


    public increment(increment: number, contract: string) {
        this.tezos.contract
            .at(contract) // step 1
            .then((contract) => {
                console.log(`Incrementing storage value by ${increment}...`);
                return contract.methods.increment(increment).send(); // steps 2, 3 and 4
            })
            .then((op) => {
                console.log(`Awaiting for ${op.hash} to be confirmed...`);
                return op.confirmation(3).then(() => op.hash); // step 5
            })
            .then((hash) => console.log(`Operation injected: https://florence.tzstats.com/${hash}`))
            .catch((error) => console.log(`Error: ${JSON.stringify(error, null, 2)}`));
    }

    public sendTz(address: string, amount: number) {

        console.log(`Transfering ${amount} ꜩ to ${address}...`);
        this.tezos.contract.transfer({ to: address, amount: amount })
            .then(op => {
                console.log(`Waiting for ${op.hash} to be confirmed...`);
                return op.confirmation(1).then(() => op.hash);
            })
            .then(hash => console.log(`${hash}`))
            .catch(error => console.log(`Error: ${error} ${JSON.stringify(error, null, 2)}`));
    }
}
```

In a dapp, we might be facing a use-case where we need to send several transactions at the same time (contract calls, originations or transfer transactions). One could be tempted to make those calls one after the other:

``` typescript
import { App } from './src/app';

const RPC_URL = "https://florencenet.smartpy.io/"
const ACCOUNT_TO_CHECK = "tz1YWK1gDPQx9N1Jh4JnmVre7xN6xhGGM4uC"
const COUNTER_CONTRACT = "KT1BEMULAGQ58C5NNdWQM3WYLjUtwgJ8X8aN"
const RECIPIENT = "tz1WD73bxtb3oeBBTU471Yz5gcy9NMzepfMJ"
const AMOUNT = 10
const INCREMENT = 5

const app : App = new App(RPC_URL);
app.increment(INCREMENT, COUNTER_CONTRACT);
app.sendTz(RECIPIENT, AMOUNT);
```

Here, we make a contract call then we send some funds to an address. Let's try it:

``` shell
$ npx ts-node main.ts 
Transfering 10 ꜩ to tz1WD73bxtb3oeBBTU471Yz5gcy9NMzepfMJ...
Incrementing storage value by 5...
Waiting for opYNFzprpcnTCS2dWP9STdJJ8HUpcMGeJNcczmKnBK1SNpXQeoC to be confirmed...
Error: {
  "message": "Http error response: (500) [{\"kind\":\"temporary\",\"id\":\"failure\",\"msg\":\"Error while applying operation ongme9f4evozEpAAtP3MUeiU79emuc8KGyoaFGYxvPUUFR3TDUA:\\nbranch refused (Error:\\n                  Counter 334156 already used for contract tz1YWK1gDPQx9N1Jh4JnmVre7xN6xhGGM4uC (expected 334157)\\n)\"}]\n",
  "status": 500,
  "statusText": "Internal Server Error",
  "body": "[{\"kind\":\"temporary\",\"id\":\"failure\",\"msg\":\"Error while applying operation ongme9f4evozEpAAtP3MUeiU79emuc8KGyoaFGYxvPUUFR3TDUA:\\nbranch refused (Error:\\n                  Counter 334156 already used for contract tz1YWK1gDPQx9N1Jh4JnmVre7xN6xhGGM4uC (expected 334157)\\n)\"}]\n",
  "url": "https://florencenet.smartpy.io/injection/operation",
  "name": "HttpResponse"
}
opYNFzprpcnTCS2dWP9STdJJ8HUpcMGeJNcczmKnBK1SNpXQeoC
```

The meaningful part is `Counter 334156 already used for contract tz1YWK1gDPQx9N1Jh4JnmVre7xN6xhGGM4uC`.
Each transaction in our app is performed asynchronously: the application made the contract call to the `increment` entrypoint, did not wait for the confirmation and then made a transfer transaction. The contract call transaction was s  till in the mempool when the transfer transaction was sent. Thus, it failed. 

However, _Taquito_ offers a `batch` method, which enables the dapp to send several transactions at the same time.

To do so, we will need to:
1. retrieve the contract that we want to call
2. call the batch method
3. Add the calls with `withTransfer` and `withContractCall`
4. send the transactions batch
5. wait for their confirmation

``` typescript
    public async sendInBatch(contractAddress: string, recipientAddress : string) {
        const contract = await this.tezos.contract.at(contractAddress) //step 1

        const batch = this.tezos.contract.batch() // step 2
            .withTransfer({ to: recipientAddress, amount: 10 }) // step 3
            .withTransfer({ to: recipientAddress, amount: 100 }) // step 3
            .withTransfer({ to: recipientAddress, amount: 1000 }) // step 3
            .withContractCall(contract.methods.increment(10)) // step 3

        const batchOp = await batch.send(); // step 4

        await batchOp.confirmation(); // step 5
    }

```

The calling of this method will give this output on _Tzstats_:
[https://florence.tzstats.com/opNz4g3XTd9oAAyPe4jMiEqXLQ67EfPPTZkXhhvXje8DoMg5D5u/2402084](https://florence.tzstats.com/opNz4g3XTd9oAAyPe4jMiEqXLQ67EfPPTZkXhhvXje8DoMg5D5u/2402084)

Our three transfer transactions and our contract call are now indeed batched together in an operation.


# Conclusion

_Taquito_ facilitates the interactions of developers with the Tezos network: it can read all the data from a blockchain, send transactions, originate a contract...and Dapps can be built with this tool.

However, dapps require the ability to manage keys. In our example, there was only a single key to manage. In real-life dapps, each user will want to safely use a key that they owns. That is where _wallets_ come in play: those tools are built upon _Taquito_ and make dapps more user-friendly and accessible. _Taquito_ can also be used along with those wallets.

