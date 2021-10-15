# Create a Lending Marketplace dapp on Polygon with Truffle Suite.

## Introduction

A Lending Marketplace provides a secure, flexible, open-source foundation for a decentralized loan marketplace on the Polygon blockchain. It provides the pieces necessary to create a decentralized lending exchange, including the requisite lending assets, repayments, and collateral infrastructure, enabling third parties to build applications for lending.

## Prerequisites

[MetaMask](https://metamask.io/) is a browser-based blockchain wallet that can be used to store any kind of digital assets and cryptocurrency.

[Polygon](https://docs.polygon.technology/) is a blockchain that is EVM compatible.

## Requirement

[Node.js](https://nodejs.org/en/) enables the development of fast web servers in JavaScript by bringing event-driven programming to web servers.

[Truffle Suite](https://www.trufflesuite.com/) is a development environment and testing framework for EVM-based blockchains.

[React.js](https://reactjs.org/) is an open-source JavaScript library that is used to create single-page applications' user interfaces.

## Create a truffle project

Install Truffle:
```
npm i -g truffle
```

Clone this [Git Repository](https://github.com/devilla/cryptolend.eth) and read the [Deploying and Debugging Smart Contracts on Polygon](https://learn.figment.io/tutorials/deploying-and-debugging-smart-contracts-on-polygon) tutorial to setup network config inside Truffle and learn the deployment on the Polygon network.

```
git clone https://github.com/Devilla/cryptolend.eth.git
```
Go to the repository:
```
cd cryptolend.eth
```

Install the required depencencies:
```
npm i
```

## Compile and migrate

Open `truffle console` to run a local blockchain in your terminal at `http://127.0.0.1:9545/`:
```
truffle develop
```

This will and display `Account addresses` along with their `Private Keys` and `Mnemonic` required for deploying the smart contracts.

In the `truffle console` compile the smart contracts:

```
truffle(develop)> compile

Compiling your contracts...
===========================
> Compiling .\contracts\LoanContract.sol
> Compiling .\contracts\LoanCreator.sol
> Compiling .\contracts\LoanProduct.sol
> Compiling .\contracts\Migrations.sol
> Compiling .\contracts\StandardToken.sol
> Compiling .\contracts\libs\DateTime\DateTime.sol
> Compiling .\contracts\libs\DateTime\api.sol
> Compiling .\contracts\libs\LoanMath.sol
> Compiling .\contracts\libs\LoanMethods.sol
> Compiling .\contracts\libs\String.sol
> Compiling openzeppelin-solidity\contracts\GSN\Context.sol
> Compiling openzeppelin-solidity\contracts\access\Roles.sol
> Compiling openzeppelin-solidity\contracts\access\roles\PauserRole.sol
> Compiling openzeppelin-solidity\contracts\lifecycle\Pausable.sol
> Compiling openzeppelin-solidity\contracts\math\SafeMath.sol
> Compiling openzeppelin-solidity\contracts\ownership\Ownable.sol
> Compiling openzeppelin-solidity\contracts\token\ERC20\IERC20.sol
> Compilation warnings encountered:

> Artifacts written to C:\Users\hp\cryptolend\build\contracts
> Compiled successfully using:
   - solc: 0.5.0+commit.1d4f565a.Emscripten.clang
```

Now, `migrate` the compiled smart contracts:

```
truffle(develop)> migrate

Starting migrations...
======================
> Network name:    'develop'
> Network id:      5777
> Block gas limit: 6721975 (0x6691b7)


2_deploy_contracts.js
=====================

   Deploying 'LoanCreator'
   -----------------------
   > transaction hash:    0x232be40e9171c62f74585c52e15492a8a8653b8a65eb9f97f6e57ccdcb0eec66
   > Blocks: 0            Seconds: 0
   > contract address:    0xc7Eb239cA1e53093B645A50d70B4a895AAD94cb0
   > block number:        4
   > block timestamp:     1629372448
   > account:             0x2F3CeD6f849630301feC1dD613869E8cc3857665
   > balance:             99.990053476
   > gas used:            2460473 (0x258b39)
   > gas price:           2 gwei
   > value sent:          0 ETH
   > total cost:          0.004920946 ETH


   Deploying 'StandardToken'
   -------------------------
   > transaction hash:    0xa74cf285dc8e80b73b91a9334304a408c973a67ecd9d8e700559c7c7d8e321d8
   > Blocks: 0            Seconds: 0
   > contract address:    0xBD613f04E9Fd211b95A608776620B6C49f11A421
   > block number:        5
   > block timestamp:     1629372449
   > account:             0x2F3CeD6f849630301feC1dD613869E8cc3857665
   > balance:             99.988584512
   > gas used:            734482 (0xb3512)
   > gas price:           2 gwei
   > value sent:          0 ETH
   > total cost:          0.001468964 ETH


   > Saving migration to chain.
   > Saving artifacts
   -------------------------------------
   > Total cost:         0.010922144 ETH


Summary
=======
> Total deployments:   2
> Final cost:          0.010922144 ETH
- Saving migration to chain.
```

# Building UI with ReactJS

Clone this [Git Repository](https://github.com/Devilla/cryptolend.ui)
```
git clone https://github.com/Devilla/cryptolend.ui.git
```
Browse the project directory:
```
cd cryptolend.ui
```

Install dependencies required for the project:
```
npm i
```

Run the app in the browser:
```
npm start
```

# Conclusion

Now you know about creating a Lending Marketplace with Truffle Suite and ReactJS on the Polygon network.

If you had any difficulties following this tutorial or simply want to discuss Polygon tech with us you can [**join our community today**](https://community.figment.io/) or [**Join our discord channel**](https://discord.gg/fszyM7K)!

# About the author

[Devendra Yadav](https://community.figment.io/u/dev.koold)

# References
- https://github.com/crypto-lend
- https://learn.figment.io/tutorials/deploying-and-debugging-smart-contracts-on-polygon
