---
description: Teaching how to Transfer ERC-20 tokens from C-chain to ETH address
---

# Transfer ERC-20 tokens from the C-chain of your AVAX wallet to an ETH address

## Introduction

Continuing the theme of transfers of tokens, in this tutorial, we are going to learn how to programmatically transfer ERC-20 tokens from the C-chain to an ETH wallet.

## Getting things set up

Simply put, ERC-20 tokens are the tokens that meet the technical standards of the Ethereum blockchain. So, they natively reside on the Ethereum blockchain. Thanks to the C-chain's full Ethereum compatibility, one can transfer ERC-20 token using the Avalanche-Ethereum bridge, which can be found [here](https://aeb.xyz/#/transfer).

However, for the tutorial, in order not to use tokens of real monetary value, we are going to do everything on the Fuji testnet. 

So, before we proceed, we have to configure your Metamask wallet to work with the Fuji testnet of Avalanche. Also, we have to get on the [AVAX Fuji Testnet Faucet](https://faucet.avax-test.network/) and send some AVAX tokens to your Metamask wallet and the C-chain address of your private Avalanche wallet (the Fuji AVAX tokens are needed to pay transaction fees later on). 

To configure your Metamask wallet, click to metamask icon on the browser and select the network drop-down menu. Here we should connect to C-Chain. Click to “Custom RPC”, as shown in the image below.

![metamask_settings](https://miro.medium.com/max/408/0*0HGM4O_J5iF3943S) 

Then, you will see several boxes (image below), which we need to fill with correct values for the Fuji testnet. 

![boxes_empty](https://miro.medium.com/max/989/1*Y7O1bBeTWnuQBAqTnwmqUQ.png)

Now, we need to set these boxes with correct values.

For `Network Name`, type in `Avalanche FUJI C-Chain`. For `New RPC URL`, type in `https://api.avax-test.network/ext/bc/C/rpc`. For `ChainID`, type in `43113`. For `Symbol`, type in `AVAX`. For `Explorer`, type in `https://cchain.explorer.avax-test.network`. All this information could also be found from [here](https://docs.avax.network/build/tutorials/smart-contracts/deploy-a-smart-contract-on-avalanche-using-remix-and-metamask#fuji-testnet-settings).  

Using the faucet to fund your wallets is simple. Refer to the image below in case you are confused. 

![funding](https://i.imgur.com/Uj6zZr8.png)

At this point, you should have AVAX both on the C-chain of your Avalanche private wallet and in your metamask wallet.

For the purpose of this tutorial, I have created an ERC-20 token named AVAXDATAHUB and created an AVAXDATAHUB - AVAX liquidity pool on [Pangolin](https://app.pangolin.exchange/#/swap) on the Fuji testnet. Pangolin is a decentralized exchange built on Avalanche and it is compatible with the Fuji testnet as well, for testing purposes. 

We will go on Pangolin and swap AVAX for some AVAXDATAHUB tokens. The token contract address is `0x6089f3b5f97eCDc8d31f317C7b442580E4258ef7`. 

Go on [Pangolin](https://app.pangolin.exchange/#/swap) and swap AVAX on your Metamask for some AVAXDATAHUB, as outlined in the image below.

![AVAXDATAHUB](https://i.imgur.com/CCUNU5z.png) 

After the swap, go back to Metamask, click on `Add Token` and type in the contract address as shown in the image below. Without manually adding the AVAXDATAHUB token, it will not show up on your Metamask.  

![AVAXDATAHUB_IN_METAMASK](https://i.imgur.com/K8YgMrY.png)

The AVAXDATAHUB tokens you got from Pangolin should now show up on your Metamask wallet, as shown below (8.979 AVAXDATAHUB tokens).

![FROM_PANGOLIN](https://i.imgur.com/o9JE1kJ.png)

For the sake of the tutorial, let's send some of those AVAXDATAHUB tokens from Metamask to your Avalanche private wallet. Refer to the image below if you are confused about how to do this.

![send_to_avax_wallet](https://i.imgur.com/NC7HDqh.png)

In the image above, it is shown that 4 AVAXDATAHUB tokens have been sent to the C-chain of the Avalanche wallet. 

However, when logging into your Avalanche wallet, you will not be able to see the tokens yet. 

![not_shown](https://i.imgur.com/LwPWIlo.png)

In order to view those tokens, you have to add the token contract address.

Click on `Add Token` below the coin list of the wallet and type in the AVAXDATAHUB contract address, then the rest of the information should automatically be loaded, as shown in the image below.

![contract_address_typed](https://i.imgur.com/r8aglQ9.png)

Then, you should be able to see the tokens show up on your list, as shown in the image below.

![avaxdatahub_added](https://i.imgur.com/VlDuDn8.png)

What we have done up to this point has taught you that any ERC-20 token can be stored on the C-chain of your Avalanche wallet. 

## Transfer of ERC-20 tokens from C-chain to ETH address

Now, we are finally ready to start the actual tutorial on ERC-2O token transfer from the C-chain of an AVAX wallet to an ETH wallet.

Similar to the previous C-chain to ETH address transfer of AVAX token transfer tutorial, we will start by installing an Ethereum library called `ethers`.

```bash
$ npm install ethers
```

If `ethers` is already installed from the previous tutorial, you can skip the installation. 

After installing the library, we are going to create a new file called `ERC20_fromC_to_ETH_address.js` in the root directory of your project. Once you create a .js file under the specified name, we will type in the following blocks of code in. 

First, we need to import the library installed earlier to interact with the Avalanche C chain

```bash
const { ethers } = require('ethers');
```

The mnemonic key from your AVAX wallet needs to go between the quotation marks below. This is later used to extract the C chain wallet address.

```bash
const mnemonic = "";
```

The code below is pointing to AVAX mainnet.

```bash
const provider = new ethers.providers.JsonRpcProvider('https://api.avax-test.network/ext/bc/C/rpc');
```

With the mnemonic phrase provided earlier, we can extract the corresponding ETH wallet private key. With this, we can unlock the Avalanche C-chain wallet and sign transactions, which is accomplished with the code below.

```bash
const walletMnemonic = new ethers.Wallet.fromMnemonic(mnemonic);
const pvtKey  = walletMnemonic.privateKey;
const wallet = new ethers.Wallet(pvtKey, provider);
```
Transferring tokens from one wallet to another is a transaction. In order to perform a transaction, certain information needs to be provided. The token address \(also known as contract address\) of the ERC-20 token \(AVAXDATAHUB in this case\) needs to be provided. So, we are going to store the contract address and the ticker below. The token ticker is not needed, but we are adding it for our own use. A lot of the token contract addresses for Avalanche can be found [here](https://github.com/pangolindex/tokenlists/blob/main/aeb.tokenlist.json).

```bash
const tokenAddress = "0x6089f3b5f97eCDc8d31f317C7b442580E4258ef7";  
const token_name = "AVAXDATAHUB";
```
Logically, we also need provide the destination address. Put the destination address between the quotation marks.

```bash
const toAddr = "";
```

Another piece needed for issuing a transaction is what's called the ABI. The Contract Application Binary Interface \(ABI\) is the standard way to interact with contracts in the Ethereum ecosystem. The format of the ABI is provided [here](https://docs.ethers.io/v5/getting-started/#getting-started--contracts).

```bash
const tknAbi = [
  // Some details about the token
  "function name() view returns (string)",
  "function symbol() view returns (string)",

  // Get the account balance
  "function balanceOf(address) view returns (uint)",

  // Send some of your tokens to someone else
  "function transfer(address to, uint amount)",

  // An event triggered whenever anyone transfers to someone else
  "event Transfer(address indexed from, address indexed to, uint amount)"
  ];
```
Although it is not necessary to print the AVAXDATAHUB balance of the destination wallet address to perform a transfer, we will show here how to read the balance of a certain token. 

```bash
const getBalance = async () => {
  // Create Contract object connected to provider
  const tknContract = new ethers.Contract(tknAddr, tknAbi, provider);

  // Get balance as BigNumber and convert to Number
  const balanceBigNum = await tknContract.balanceOf(toAddr);
  const balanceNum = Number(ethers.utils.formatEther(balanceBigNum));

  // Set precision and convert to string
  const precision = 4;
  const balanceStr = balanceNum.toFixed(precision).toString();

  return balanceStr;
}
```

Now, we are going to define `sendToken` function, which will execute a transfer of 1 AVAXDATAHUB token from the C-chain of the Avalanche wallet to the ETH destination address.

```bash
const sendToken = async () => {
  if (provider === null || wallet === null) {
    console.error("Encountered null object, unable to send token.");
    return;
  } 

  // Create Contract object connected to wallet
  const tknContract = new ethers.Contract(tknAddr, tknAbi, wallet);

  // Specify amount to send (e.g. 1 ERC20 token)
  const amt = ethers.utils.parseEther("1.0");

  // Send amount to destination
  const tx = tknContract.transfer(toAddr, amt);

  return tx;
}
```
Now that we have the functions to read the balance and execute a transfer, we are ready to implement them. The code below reads the balance before and after the transfer of 1 AVAXDATAHUB token. 

```bash
getBalance()
  .then(initBalance => {
    console.log("Initial destination balance: ", initBalance);

    // Send ERC20 token from AVAX wallet C-Chain to ETH address
    sendToken()
      .then(_tx => {
        console.log("Transfer successful!");

        // Log final destination address balance
        getBalance()
          .then(finalBalance => {
            console.log("Final destination balance: ", finalBalance);
          })
          .catch(console.error);
      })
      .catch(console.error);
  })
  .catch(console.error);
  ```
At this point, you have gone through the entire script.

The finished script should look as follows:
```bash
const { ethers } = require('ethers');
const mnemonic = "";
const provider = new ethers.providers.JsonRpcProvider('https://api.avax-test.network/ext/bc/C/rpc');

const walletMnemonic = new ethers.Wallet.fromMnemonic(mnemonic);
const pvtKey  = walletMnemonic.privateKey;
const wallet = new ethers.Wallet(pvtKey, provider);

const tokenAddress = "0x6089f3b5f97eCDc8d31f317C7b442580E4258ef7";  
const token_name = "AVAXDATAHUB";
const toAddr = "";

const tknAbi = [
  // Some details about the token
  "function name() view returns (string)",
  "function symbol() view returns (string)",

  // Get the account balance
  "function balanceOf(address) view returns (uint)",

  // Send some of your tokens to someone else
  "function transfer(address to, uint amount)",

  // An event triggered whenever anyone transfers to someone else
  "event Transfer(address indexed from, address indexed to, uint amount)"
  ];

const getBalance = async () => {
  // Create Contract object connected to provider
  const tknContract = new ethers.Contract(tknAddr, tknAbi, provider);

  // Get balance as BigNumber and convert to Number
  const balanceBigNum = await tknContract.balanceOf(toAddr);
  const balanceNum = Number(ethers.utils.formatEther(balanceBigNum));

  // Set precision and convert to string
  const precision = 4;
  const balanceStr = balanceNum.toFixed(precision).toString();

  return balanceStr;
}

const sendToken = async () => {
  if (provider === null || wallet === null) {
    console.error("Encountered null object, unable to send token.");
    return;
  } 

  // Create Contract object connected to wallet
  const tknContract = new ethers.Contract(tknAddr, tknAbi, wallet);

  // Specify amount to send (e.g. 1 ERC20 token)
  const amt = ethers.utils.parseEther("1.0");

  // Send amount to destination
  const tx = tknContract.transfer(toAddr, amt);

  return tx;
}

getBalance()
  .then(initBalance => {
    console.log("Initial destination balance: ", initBalance);

    // Send ERC20 token from AVAX wallet C-Chain to ETH address
    sendToken()
      .then(_tx => {
        console.log("Transfer successful!");

        // Log final destination address balance
        getBalance()
          .then(finalBalance => {
            console.log("Final destination balance: ", finalBalance);
          })
          .catch(console.error);
      })
      .catch(console.error);
  })
  .catch(console.error);
```

To run the script `ERC20_fromC_to_ETH_address.js`, type `node ERC20_fromC_to_ETH_address.js` and run it in your terminal (`node` before the file name is for NodeJs runtime.environment).

The output of the script in the terminal should look similar to what is shown below.

![example_output](https://i.imgur.com/hVX9aJA.png)

You can confirm the result by looking at the change of the balance on Metamask as well.

As you can see in the image below, my metamask balance did change to 5.97.

![metamask_balance](https://i.imgur.com/CTIA5eu.png)

## Wrapping Up

That’s it! This tutorial has taught you how to transfer ERC-20 tokens from the C-chain to an ETH wallet. Remember that the C-chain uses the Ethereum Virtual Machine and is compatible with all of the key Ethereum tools.
<<<<<<< HEAD
=======

## About the author

This tutorial was created by [Seongwoo Oh](https://github.com/blackwidoq). He is a student and an Avalanche novice.

>>>>>>> upstream/master
