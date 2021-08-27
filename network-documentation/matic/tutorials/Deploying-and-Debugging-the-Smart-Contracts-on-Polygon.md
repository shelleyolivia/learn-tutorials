# Introduction
In this tutorial, we will cover how to deploy smart contracts to the Polygon (Matic) Mumbai test network.
We'll cover some of the possible errors which might occur during the deployment.
So, grab a cup of coffee ☕️ and follow the steps.

# Prerequisites
This tutorial assumes that you have some beginner-level experience in programming & blockchain understanding.

# Requirements

* [Truffle](https://www.trufflesuite.com/)
* MetaMask setup: 
To deploy the smart contracts on Matic you first have to add an RPC endpoint in your MetaMask wallet.
`Settings -> Networks -> Add network -> Save`

![](../../../.gitbook/assets/deploy-&-debug-on-polygon-1.png)

To get test Matic for deployment and testing,
* `go to Matic Faucet -> Select Mumbai -> Paste wallet address -> Submit`, Matic Faucet [link](https://faucet.matic.network).

Done! check your wallet, you'll see some Matic there.

![](../../../.gitbook/assets/deploy-&-debug-on-polygon-2.png)

# truffle-config
* `truffle-config.js` for Mac users
* `truffle.js` for Windows users

The truffle-config file is an important file to understand. In this file, we must configure the path to the DTube Solidity file (smart contract), the contract ABI, and define the available **networks**.

```javascript
const HDWalletProvider = require("@truffle/hdwallet-provider")
require('dotenv').config(); // Load .env file

module.exports = {
  networks: {
   // For Ganache, your personal blockchain
   development: {
      host: "127.0.0.1",     // Localhost (default: none)
      port: 8545,            // Standard Ethereum port 
      network_id: "*",       // Any network (default: none)
    },
  },
  contracts_directory: './src/contracts/', // path to Smart Contracts
  contracts_build_directory: './src/abis/', // Path to ABIs
  compilers: {
    solc: {
      optimizer: {
        enabled: true,
        runs: 200
      }
    }
  }
}
```

Ensure you create an `.env` file in the project root directory (`~/DTube/.env`) and paste into it the Secret Recovery Phrase (12 words) of your preferably newly generated and testnet-only MetaMask wallet with the variable name MNEMONIC. This will be loaded by truffle at runtime, and the environment variable can then be accessed with `process.env.MNEMONIC`.

```
MNEMONIC= 12 secret words here..
```

Now, let's add `matic` network in our truffle-config file which will contain our environment variable MNEMONIC and RPC URL.

```javascript
    matic: {
      provider: () => new HDWalletProvider(process.env.MNEMONIC, 
      `https://rpc-mumbai.matic.today`),
      network_id: 80001,
      confirmations: 2,
      timeoutBlocks: 200,
      skipDryRun: true,
      gas: 6000000,
      gasPrice: 10000000000,
    },
```

You can set the gas price and gas limits for faster transactions as shown in the above code block.

# Deploy Smart Contracts
* Command: `truffle migrate --network matic`

If you're deploying it for the second time then deploy with this command just to **reset** and avoid JSON errors.

* Command: `truffle migrate --network matic --reset`

If everything worked fine, you'll see something like this:

```bash
2_deploy_contracts.js
=====================

   Replacing 'MyContract'
   ------------------
   > transaction hash:    0x1c94d095a2f629521344885910e6a01076188fa815a310765679b05abc09a250
   > Blocks: 5            Seconds: 5
   > contract address:    0xbFa33D565Fcb81a9CE8e7a35B61b12B04220A8EB
   > block number:        2371252
   > block timestamp:     1578238698
   > account:             0x9fB29AAc15b9A4B7F17c3385939b007540f4d791
   > balance:             79.409358061899298312
   > gas used:            1896986
   > gas price:           0 gwei
   > value sent:          0 ETH
   > total cost:          0 ETH

   Pausing for 2 confirmations...
   ------------------------------
   > confirmation number: 5 (block: 2371262)
initialised!

   > Saving migration to chain.
   > Saving artifacts
   -------------------------------------
   > Total cost:                   0 ETH


Summary
=======
> Total deployments:   2
> Final cost:          0 ETH
```

*Code snippet from matic truffle docs.*

# Deployment errors and solutions
If you get any of these errors then follow these steps

**Error**

```
Error: PollingBlockTracker - encountered an error while attempting to update latest block:
```

**Fix_1**
Change the RPC endpoint URL in Metamask from 'https://rpc-mumbai.matic.today' to an [Infura RPC endpoint](https://infura.io/). This will require you to register for an Infura account and set up a Project, to get a Project ID. If you already have an Infura project ID, add it to the `.env` file => `PROJECT_ID=<your project ID>`

`infura -> Create new project -> Settings -> Endpoints -> Polygon Mumbai`

```javascript
    matic: {
      provider: () => new HDWalletProvider(process.env.MNEMONIC, 
      `https://polygon-mumbai.infura.io/v3/process.env.PROJECT_ID`),
      network_id: 80001,
      confirmations: 2,
      timeoutBlocks: 200,
      skipDryRun: true,
    },
  },
```

Paste your PROJECT_ID there from .env file.
* `truffle migrate --network matic --reset`

If the error still occurs, try another alternate RPC endpoint from [MaticVigil](https://maticvigil.com/).

**Fix_2**
Change `https://rpc-mumbai.matic.today` by using [Matic custom RPC](https://rpc.maticvigil.com/).

```javascript
    matic: {
      provider: () => new HDWalletProvider(process.env.MNEMONIC, 
      `https://rpc-mumbai.maticvigil.com/v1/process.env.PROJECT_ID`),
      network_id: 80001,
      confirmations: 2,
      timeoutBlocks: 200,
      skipDryRun: true,
    },
  },
```

Paste your PROJECT_ID there from .env file.
* `truffle migrate --network matic --reset`

**Error:**

```
*** Deployment Failed ***

"Migrations" -- only replay-protected (EIP-155) transactions allowed over RPC.
```

**Fix:**
* `npm install @truffle/hdwallet-provider@1.4.0`

Truffle hdwallet-provider version 1.4.0 will fix this error.

**Error:**

```
Error:  *** Deployment Failed ***

"Migrations" -- Transaction was not mined within 750 seconds, please make sure your transaction was properly sent. Be aware that it might still be mined!.
```

**Fix:**

```javascript
    matic: {
      provider: () => new HDWalletProvider(process.env.MNEMONIC, 
      `https://rpc-mumbai.maticvigil.com/v1/process.env.PROJECT_ID`),
      network_id: 80001,
      confirmations: 2,
      timeoutBlocks: 200,
      skipDryRun: true,
      networkCheckTimeout: 100000,
    },
  },
```

Just add `networkCheckTimeout: 100000`


*If you discover any new errors and If you know the solution for it, then feel free to make a PR, we'll add your Error-Fix here.*

# Conclusion
After this tutorial you will be able to:
* Deploy the smart contracts on polygon (Matic) Mumbai Test Network.
* Tackle the errors while deploying the smart contracts on polygon (Matic) Mumbai Test Network.

# About the Author
I'm Akhilesh Thite, an Indian tech enthusiast with a passion for Software Development, Open-Source & Decentralization. Feel free to connect with me on [GitHub](https://github.com/AkhileshThite) & [Twitter](https://twitter.com/AkhileshThite_).

# References
* *Truffle docs: https://www.trufflesuite.com/docs/truffle/overview*
* *Polygon (Matic) docs: https://docs.matic.network/docs/develop/getting-started*
* *GitHub repo: https://github.com/AkhileshThite/DTube*
