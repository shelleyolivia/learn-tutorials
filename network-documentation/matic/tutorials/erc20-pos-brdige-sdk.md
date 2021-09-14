How to transfer ERC20 tokens to Polygon Network using PoS bridge SDK/maticjs
===
**Polygon PoS Bridge** is a mechanism and set of contracts on both child and root chains that will help us in moving assets between **Ethereum** and **Polygon**.

In this tutorial, step by step we go through transferring `ERC20` assets or an `ERC20` custom token to the polygon chain, using **PoS SDK**.  
We will use the **Ethereum Goerli** testnet and **Polygon Mumbai** testnet, and a custom `ERC20` token I have deployed and verified. [Here is the step by step guide](https://github.com/mlibre/blockchain/tree/master/Ethereum/ERC20)

## Table of content
* [High-level flow](#high-level-flow)
* [Glossary](#glossary)
* [Requirements](#requirements)
* [MetaMask](#metamask)
	* [Goerli](#goerli)
	* [Mumbai](#mumbai)
* [MLB ERC20 Contract](#mlb-erc20-contract)
* [Mapping](#mapping)
* [Transfer using SDK](#transfer-using-sdk)
	* [Providers](#providers)
	* [Installing SDK](#installing-sdk)
	* [Approve](#approve)
	* [Deposit](#deposit)
* [Transfer using Web UI](#transfer-using-web-ui)
* [Conclusion](#conclusion)
 

## High-level flow
1. In order to transfer assets between **root** and **child** contracts, they should be mapped first. If the token you intend to transfer is already in **Polygon**, means you don't have to `map` them again. If it is your own `ERC20` token, we need to `map`.
2. Now that contracts are mapped. it's time to transfer the assets. We can either use [web UI](https://wallet.polygon.technology/login/) or the **SDK**
	* We use the **SDK** for **our** `ERC20` token that is in **Goerli**
	* And **Web UI** for a mainnet real token

## Glossary
* **Root chain**: The chain you want to transfer your asset `from` (Ethereum main/test net)
* **Root token**: The token in the root chain (`ERC20` in Ethereum main/test net)
* **Child chain**: The chain you want to transfer your asset `to` (Polygon)
* **Child token**: The token in the child chain (`ERC20` in Polygon main/test net)
* **PoS Bridge**: Polygon Proof of Stake bridge
* **Goerli**: Ethereum testnet
* **Mumbai**: Polygon testnet
* **MLB**: Symbole name of the custom `ERC20` token written in **Solidity/Openzeppelin**. [more](https://github.com/mlibre/blockchain/tree/master/Ethereum/ERC20)


## Requirements

* [**Metamask**](https://metamask.io/)
* [**Nodejs**](https://nodejs.org/en/download/): `v14.17.4`
* **npm**: `7.6.3`
* [**Geth**](https://geth.ethereum.org/docs/install-and-build/installing-geth): `1.10.8`

## Metamask
Before we go technical, let's set up **Metamask** so we can check our **ETH/MLB** token amounts.

### Goerli
It is pre-configured on **Metamask**, in case it was not, sign up here [infura](https://infura.io) and get your own **Goerli** API keys.  
Then click on `Add Network` in metamsk and put the keys & info.  

* Network Info

	![metamask goerli](../../../.gitbook/assets/erc20-pos-metamask-goerli.png)

* Fund

	![metamask goerli fund](../../../.gitbook/assets/erc20-pos-metamask-goerli-fund.png)

You can fund your **Goerli** account [here](https://faucet.goerli.mudit.blog/)

### Mumbai
You can either open [mumbai.polygonscan.com](https://mumbai.polygonscan.com/) and click on `Add Mumbai Network` at the bottom of the page. **or** add it yourself:

* **Network Name:** Polygon Testnet
* **RPC URL:** https://rpc-mumbai.maticvigil.com/
* **Chain ID:** 80001
* **Currency Symbol:** MATIC
* **Block Explorer URL:** https://mumbai.polygonscan.com/

![mumbai](../../../.gitbook/assets/erc20-pos-mumbai.png)

You can fund your **Mumbai** account with MATIC [here](https://faucet.polygon.technology)


## MLB ERC20 Contract
The token we'll **map** and transfer. A standard **openzeppelin** `ERC20` token. I deployed before in **Goerli**.  
You can find the [step by step guide here](https://github.com/mlibre/blockchain/tree/master/Ethereum/ERC20)  
Token info:
```
Name: Mlibre
Symbole: MLB
Owner: 0xD8f24D419153E5D03d614C5155f900f4B5C8A65C
Contract Address: 0xd2d40892B3EebdA85e4A2742A97CA787559BF92f
Goerli etherscan: https://goerli.etherscan.io/address/0xd2d40892B3EebdA85e4A2742A97CA787559BF92f
```
Gather this information for the **token** you intent to **map**.

## Mapping
Now that everything is ready. Let's map our `MLB` token.
* Open [mapper.matic.today](https://mapper.matic.today/map/) and fill the form
* Make sure the token you want to map is [verfied](https://etherscan.io/verifyContract)
* Choose **Gorli Testnet - Mumbai testnet**

	![map image](../../../.gitbook/assets/erc20-pos-map.png)

* Right now the mapping process is not instantly, It takes up to 3 days to confirm.  
Then open [mapper.matic.today](https://mapper.matic.today/), and enter the contract address. see if it is added. 

	![mapped image](../../../.gitbook/assets/erc20-pos-mapped.png)

As you may notice the contract address in **Gorli** and **Mumbai** are not the same. so let's add it to **Metamask** as well

1. Open **Metamask**
2. Choose Polygon testnet
3. Add Token
4. put the contract address there (`0x0F6886ca4476D3aAb965F2D1b9185F2dd857E653`)

Now it should be something like:

![metamask after map](../../../.gitbook/assets/erc20-pos-metamask-after-map.png)

We don't have any `MLB` tokens there yet. now let's transfer some and check our **metamask** again.

## Transfer using SDK
Let's take a look at transferring cycle:

1. **Approve:** Owner of the token has to **approve** the `Ethereum Predicate Contract` which will **lock** the amount of token we want to transfer to Polygon.
2. **Deposit:** Then a function has to be called on the `RootChainManger` contract which will trigger the `ChildChainManager` contract on **Mumbai**. And the `ChildChainManager` contract will call the **deposit** function of the `Child token` contract.

### Providers
To Interact with **Goerli** and **Mumbai** we either should run a local node or use providers' **RPC APIs**.  
For **Goerli**, we will run a local `Geth` node. you can also use [infura](https://infura.io).  
For **Mumbai**, we will use [figment](https://auth.figment.io/)

#### Goerli
Install `Geth` if you have not it already. and run:
```bash
geth --goerli --http --syncmode=light --http.api="eth,net,web3,personal,txpool" --allow-insecure-unlock  --http.corsdomain "*"
```
The default endpoint is `127.0.0.1:8545`.  
You can get attached and see if everything is fine:
```bash
geth attach http://127.0.0.1:8545
eth.getBalance("0xD8f24D419153E5D03d614C5155f900f4B5C8A65C")
```

#### Mumbai
* Sign up [figment](https://auth.figment.io/)  
* Choose **Polygon** service
* Find `Matic Mumbai Testnet JSONRPC`. It is probably located [here](https://datahub.figment.io/services/Polygon/matic-mumbai--jsonrpc)
* Copy the **URL**. It is something like: 
```
https://matic-mumbai--jsonrpc.datahub.figment.io/apikey/a434343/
```

### Installing SDK
It's finally time to code :)  
Install **Polygon** SDK
```bash
npm install @maticnetwork/maticjs --save
npm install @truffle/hdwallet-provider --save
```

### Approve
To **approve** the **Ethereum Predicate Contract** we just simply need to call `approveERC20ForDeposit`  

```javascript
await maticPOSClient.approveERC20ForDeposit(rootToken, amount.toString(), {
	from,
	gasPrice: "10000000000"
});
```

### Deposit
Now we should call `depositERC20ForUser`
```javascript
await maticPOSClient.depositERC20ForUser(rootToken, from, amount.toString(), {
	from,
	gasPrice: "10000000000",
});
```

And it's done. The whole code is pretty simple:
```javascript
const HDWalletProvider = require('@truffle/hdwallet-provider')
const {MaticPOSClient} = require('@maticnetwork/maticjs')
const secrets = require('./secrets.json')

let from = "0xD8f24D419153E5D03d614C5155f900f4B5C8A65C";
let rootToken = "0xd2d40892B3EebdA85e4A2742A97CA787559BF92f";
let amount = 999 * (10 ** 18);

const parentProvider = new HDWalletProvider(secrets.seed, 'http://127.0.0.1:8545')
const maticProvider = new HDWalletProvider(secrets.seed, secrets.mumbai)

const maticPOSClient = new MaticPOSClient({
	network: "testnet",
	version: "mumbai",
	parentProvider,
	maticProvider
});


(async () => {
	try
	{
		let result = await maticPOSClient.approveERC20ForDeposit(rootToken, amount.toString(), {
			from,
			gasPrice: "10000000000"
		});
		let result_2 = await maticPOSClient.depositERC20ForUser(rootToken, from, amount.toString(), {
			from,
			gasPrice: "10000000000",
		 });
		console.log(result);
		console.log(result_2);
	}
	catch (error)
	{
		console.log(error);
	}
})()
```

Just a few things to mention:
* `secrets.json`: contains **Seed**, **privateKey** of the address (0xd8f2). And **Mumbai API URL**
* `@truffle/hdwallet-provider`: Handles signing transactions proccess
* `from`: The Goerli address we created token and want to send transactions with
* `rootToken`: The `ERC20` contract address in **Goerli**
* `amount`: the amount of **token** we want transfer. By default, **open zeppelin** V4 `ERC20` contract uses a value of **18** for **decimals**. That is why **999** is multiplied by **(10 ** 18)**

### Not able to run `main.js` 
* If you are facing an error message like
	```bash
	Error: execution reverted: ERC20: approve to the zero address
	```
	The contract probably has not mapped yet.

### Sync & Confirmation

It takes up to 5 min, so that **Polygon** read data from the **Goerli** chain and `sync` itself. Then we can check your balance on **Metamask**

![MLB Metamask](../../../.gitbook/assets/erc20-MLB-token.png)

## Transfer using Web UI
Transferring assets through **Web UI** is pretty simple. We will transfer some **DAI** tokens from **Ethereum** mainnet to **Polygon**.  

1. Open [wallet.polygon.technology](https://wallet.polygon.technology/login?next=%2Fbridge)
2. Make sure **Ethereum Maninet** is selected

	![Metamask Ethereum](../../../.gitbook/assets/erc20-pos-metamask-eth-mainnet.png)

3. Click on **Metamask**. first login option
4. You will be asked to sign a **Signature Request**
5. Now enter the amount of `DAI` you want to transfer

	![DAI](../../../.gitbook/assets/erc20-pos-web-ui-dai.png)

6. Click on **Transfer**
7. Then you confirm all the **information**, **fees** and other details

	![transfer](../../../.gitbook/assets/erc20-pos-web-ui-transfer.png)

8. And that is it! it takes up to 7 min to complete the transfer

## Conclusion
Congratulations! You learned how to use `Pos Bridge SDK`.  
We have configured `Metamask` and `Geth`, then we `mapped` our `ERC20` token.  
And finally, we called `PoS Bridge` contracts, and moved our assets to **Polygon**.  