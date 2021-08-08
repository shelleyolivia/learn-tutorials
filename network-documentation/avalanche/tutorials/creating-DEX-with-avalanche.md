# Create a Decentralized Exchange(DEX) on Avalanche using Trufflesuite and ReactJS

## Introduction
Decentralized exchange is a peer-to-peer network, where anyone can exchange cryptocurrency tokens with a blockchain and a wallet with another person by executing a trade between two respective tokens, say AVAX and LINK tokens.

## Prerequisites
You must have gone through this tutorial Run an Avalanche Node and have performed a cross-chain swap via the Transfer AVAX Between X-Chain and C-Chain tutorial to get AVAX test tokens to your C-Chain address.

## Requirement  
* [NodeJS](https://nodejs.org/en)
* Truffle, which you can install with `npm install -g truffle`
* Metamask extension added to the browser, which you can add from [here](https://metamask.io/download.html)

**Create AvaSwap directory and install dependencies**

Open a new terminal tab so we can create a directory and install some further dependencies.
First, navigate to the directory within which you intend to create your working directory:
```
cd /path/to/directory
```
Create and enter a new directory named AvaSwap:
```
mkdir AvaSwap; cd AvaSwap
```
Use npm to install web3, which is a library through which we can talk to the EVM:
```
npm install web3 -s
```
We'll use web3 to set an HTTP Provider which is how web3 will speak to the EVM. Lastly, create a boilerplate truffle project:
```
truffle init
```

**Update truffle-config.js**

One of the files created when you ran truffle init is truffle-config.js. Add the following to `truffle-config.js`.
```
const Web3 = require('web3');
const protocol = "http";
const ip = "localhost";
const port = 9650;
module.exports = {
  networks: {
   development: {
     provider: function() {
      return new Web3.providers.HttpProvider(`${protocol}://${ip}:${port}/ext/bc/C/rpc`)
     },
     network_id: "*",
     gas: 3000000,
     gasPrice: 225000000000
   }
  }
};
```
Note that you can change the `protocol`, `ip` and `port` if you want to direct API calls to a different AvalancheGo node. Also, note that we're setting the `gasPrice` and `gas` to the appropriate values for the Avalanche C-Chain.
## Add AvaSwap.sol
In the contracts directory add a new file called `AvaSwap.sol` and add the following block of code:
```
pragma solidity 0.6.7;
import "@chainlink/contracts/src/v0.6/interfaces/AggregatorV3Interface.sol";
import './DevToken.sol';
contract AvaSwap {
  string public name = "AvaSwap Network Exchange";
  DevToken public Token;
  uint public rate;
  AggregatorV3Interface internal priceFeed;
  event TokenPurchase(
    address account,
    address token,
    uint amount,
    uint rate
  );
  event TokenSold(
    address account,
    address token,
    uint amount,
    uint rate
  );
  constructor(DevToken _Token) public {
    Token = _Token;
    priceFeed = AggregatorV3Interface(0x22B58f1EbEDfCA50feF632bD73368b2FdA96D541);
    rate = uint256(getLatestPrice());
  }


      /**
     * Returns the latest price
     */
    function getLatestPrice() public view returns (int) {
        (
            uint80 roundID,
            int price,
            uint startedAt,
            uint timeStamp,
            uint80 answeredInRound
        ) = priceFeed.latestRoundData();
        // If the round is not complete yet, timestamp is 0
        // require(timeStamp > 0, "Round not complete");
        return 1e18/price;
    }

  function buyTokens() public payable {
    /* Calculate no. of tokens to buy
     Ether Amount * Redemption rate */
    uint tokenAmount = msg.value * rate;
    Token.transfer(msg.sender, tokenAmount);

    // Emit an event
    emit TokenPurchase(msg.sender, address(Token), tokenAmount, rate);
  }

  function sellToken(uint _amount) public {
    // User can't sell more tokens than they have
    require(Token.balanceOf(msg.sender) >= _amount);

    //Calculate the amount of the ether to redeem
    uint etherAmount =  _amount / rate;

    // Require that AvaSwap has enough ether
    require(address(this).balance >= etherAmount);

    // Perform Sale
    Token.transferFrom(msg.sender, address(this), _amount);
    msg.sender.transfer(etherAmount);

    //Emit an event
    emit TokenSold(msg.sender, address(Token), _amount, rate);
  }
}
```

## Add new migration
Create a new file in the ```migrations``` directory named ```2_deploy_contracts.js```, and add the following block of code. This handles deploying the ```Storage``` smart contract to the blockchain.

```
const AvaSwap = artifacts.require("AvaSwap");
const DevToken = artifacts.require("DevToken");

module.exports = function (deployer) {
    // Deploy DevToken
  await deployer.deploy(DevToken, '5777');
  const devToken = await DevToken.deployed();
  //Deploy AvaSwap with DevToken
  await deployer.deploy(AvaSwap, devToken.address);
  avaSwap =  await AvaSwap.deployed();
  // Mint 0.001 DevToken to AvaSwap
  await devToken.transfer(avaSwap.address, '1000000000000000000000000');

};
```
## Compile Contracts with Truffle
Any time you make a change to `Storage.sol` you need to run `truffle` compile.
```
truffle compile
```
You should see:
```
Compiling your contracts...
===========================
> Compiling ./contracts/Migrations.sol
> Compiling ./contracts/Storage.sol
> Artifacts written to /path/to/build/contracts
> Compiled successfully using:
   - solc: 0.5.16+commit.9c3226ce.Emscripten.clang
```
## Create, fund and unlock an account on the C-Chain
When deploying smart contracts to the C-Chain, the truffle will default to the first available account provided by your C-Chain client as the from the address used during migrations.
Create an account
Truffle has a very useful console that we can use to interact with the blockchain and our contract. Open the console:
```
truffle console --network development
```
Then, in the console, create the account:
```
truffle(development)> let account = await web3.eth.personal.newAccount()
```
This returns:
```
undefined
```
Print the account:
```
truffle(development)> account
```
This prints the account:
```
'0x090172CD36e9f4906Af17B2C36D662E69f162282'
```
Unlock your account:
```
truffle(development)> await web3.eth.personal.unlockAccount(account)
```
This returns:
```
true
```

## Run Migrations
Now everything is in place to run migrations and deploy the Storage contract:
```
truffle(development)> migrate --network development
```
You should see:
```
Compiling your contracts...
===========================
> Everything is up to date, there is nothing to compile.

Migrations dry-run (simulation)
===============================
> Network name:    'development-fork'
> Network id:      1
> Block gas limit: 99804786 (0x5f2e672)


1_initial_migration.js
======================

   Deploying 'Migrations'
   ----------------------
   > block number:        4
   > block timestamp:     1607734632
   > account:             0x34Cb796d4D6A3e7F41c4465C65b9056Fe2D3B8fD
   > balance:             1000.91683679
   > gas used:            176943 (0x2b32f)
   > gas price:           225 gwei
   > value sent:          0 ETH
   > total cost:          0.08316321 ETH

   -------------------------------------
   > Total cost:          0.08316321 ETH

2_deploy_contracts.js
=====================

   Deploying 'Storage'
   -------------------
   > block number:        6
   > block timestamp:     1607734633
   > account:             0x34Cb796d4D6A3e7F41c4465C65b9056Fe2D3B8fD
   > balance:             1000.8587791
   > gas used:            96189 (0x177bd)
   > gas price:           225 gwei
   > value sent:          0 ETH
   > total cost:          0.04520883 ETH

   -------------------------------------
   > Total cost:          0.04520883 ETH

Summary
=======
> Total deployments:   2
> Final cost:          0.13542204 ETH
```
If you didn't create an account on the C-Chain you'll see this error:
```
Error: Expected parameter 'from' not passed to function.
```
If you didn't fund the account, you'll see this error:
```
Error:  *** Deployment Failed ***

"Migrations" could not deploy due to insufficient funds
   * Account:  0x090172CD36e9f4906Af17B2C36D662E69f162282
   * Balance:  0 wei
   * Message:  sender doesn't have enough funds to send tx. The upfront cost is: 1410000000000000000 and the sender's account only has: 0
   * Try:
      + Using an adequately funded account

If you didn't unlock the account, you'll see this error:
Error:  *** Deployment Failed ***

"Migrations" -- Returned error: authentication needed: password or unlock.

```
## Interacting with your contract

Buy Tokens
```
result = await avaSwap.buyTokens({ from : investor, value: web3.utils.toWei('1', 'ether')})
```
Sell Tokens
Investor must approve the token before transaction :
```
result = await Token.approve(avaSwap.address, tokens('100'), { from: investor})
```
Investor sells tokens :
```
result = await avaSwap.sellToken(tokens('100'), { from: investor })
```
Check Balance
Check Eth balance :
```
let EthBalance = await web3.eth.getBalance(avaSwap.address)
```
Check Token balance :
```
let TokenBalance = await Token.balanceOf(avaSwap.address)
```

## Summary
Now you have the knowledge of creating a Decentralized Exchange (DEX) with the Avalanche network. Creating a truffle project, as well as create, compile, deploy and interact with Solidity contracts.

If you had any difficulties following this tutorial or simply want to discuss Avalanche tech with us you can [**join our community today**](https://community.figment.io/) or [**Join our discord channel**](https://discord.gg/fszyM7K)!

## About the author

[Devendra Yadav](https://community.figment.io/u/dev.koold)

## References

- https://learn.figment.io/network-documentation/avalanche/tutorials/getting-started-with-smart-contracts-development/using-truffle-with-the-avalanche-c-chain
- https://github.com/OpenZeppelin/openzeppelin-contracts
- https://github.com/devilla/Avaswap
