# Introduction

A decentralized exchange is a network, where anyone can exchange cryptocurrency tokens over blockchain by executing a transaction between two respective tokens, say AVAX with LINK, DAI or DevToken.

# Prerequisites

You must have gone through this tutorial [Create a Local Test Network on Avalanche](https://learn.figment.io/network-documentation/avalanche/tutorials/platform-overview/create-a-local-test-network) and have performed a cross-chain swap via the [Transfer AVAX Between X-Chain and C-Chain](https://learn.figment.io/network-documentation/avalanche/tutorials/token-transfer-tutorials/transfer-avax-between-the-x-chain-and-c-chain) tutorial to get AVAX test tokens to your C-Chain address.

# Requirements

* [NodeJS](https://nodejs.org/en)
* [ReactJS](https://reactjs.org/)
* Truffle, which you can install with `npm install -g truffle`
* Install [Metamask extension](https://metamask.io/download.html) in your browser.

**Create AvaSwap directory and install dependencies**

Open a new terminal tab so we can create a directory and install some further dependencies.
First, navigate to the directory within which you intend to create your working directory:

```
cd /path/to/directory
```

Create and enter a new directory named `AvaSwap`:

```text
mkdir AvaSwap
cd AvaSwap
```

We'll be using the web3.js library to set up an HTTP provider so we can "talk" to the EVM. Install it as a dependency with the command:

```text
npm install web3 -s
```

Then, create a boilerplate truffle project:

```text
truffle init
```

**Update truffle-config.js**

`truffle-config.js` is the configuration file created when you run `truffle init`. Add the following to `truffle-config.js` to set up the web3.js connection to Avalanche:

```javascript
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

```javascript
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

  // Returns the latest price
  function getLatestPrice() public view returns (int) {
    (
      uint80 roundID,
      int price,
      uint startedAt,
      uint timeStamp,
      uint80 answeredInRound
    ) = priceFeed.latestRoundData();
    // If the round is not complete yet, timestamp is 0
    require(timeStamp > 0, "Round not complete");
    return 1e18/price;
  }

  function buyTokens() public payable {
    // Calculate number of tokens to buy:
    // Avax Amount * Redemption rate 
    uint tokenAmount = msg.value * rate;
    Token.transfer(msg.sender, tokenAmount);
    // Emit an event
    emit TokenPurchase(msg.sender, address(Token), tokenAmount, rate);
  }

  function sellToken(uint _amount) public {
    // User can't sell more tokens than they have
    require(Token.balanceOf(msg.sender) >= _amount);
    // Calculate the amount of the avax to redeem
    uint avaxAmount =  _amount / rate;
    // Require that AvaSwap has enough avax
    require(address(this).balance >= avaxAmount);
    // Perform Sale
    Token.transferFrom(msg.sender, address(this), _amount);
    msg.sender.transfer(avaxAmount);
    // Emit an event
    emit TokenSold(msg.sender, address(Token), _amount, rate);
  }
}
```

## Add new migration

Create a new file in the `migrations` directory named `2_deploy_contracts.js`, and add the following block of code. This handles deploying the `AvaSwap` smart contract to the blockchain.

```javascript
const AvaSwap = artifacts.require("AvaSwap");
const DevToken = artifacts.require("DevToken");
module.exports = function (deployer) {
  // Deploy DevToken
  await deployer.deploy(DevToken, '5777');
  const devToken = await DevToken.deployed();
  // Deploy AvaSwap with DevToken
  await deployer.deploy(AvaSwap, devToken.address);
  avaSwap =  await AvaSwap.deployed();
  // Mint 0.001 DevToken to AvaSwap
  await devToken.transfer(avaSwap.address, '1000000000000000000000000');
};
```

## Compile Contracts with Truffle

Any time you make a change to `AvaSwap.sol` you must compile the contracts again.

```
truffle compile
```

You should see:
```bash
Compiling your contracts...
===========================
> Compiling ./contracts/Migrations.sol
> Compiling ./contracts/AvaSwap.sol
> Artifacts written to /path/to/build/contracts
> Compiled successfully using:
   - solc: 0.5.16+commit.9c3226ce.Emscripten.clang
```

## Create and unlock an account on the C-Chain

When deploying smart contracts to the C-Chain, Truffle will default to the first available account provided by your C-Chain client as the `from` address used during migrations.

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

Now everything is in place to run the migrations and deploy the contract:

```
truffle(development)> migrate --network development
```

You should see:

```bash
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

   Deploying 'AvaSwap'
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

```bash
Error:  *** Deployment Failed ***

"Migrations" could not deploy due to insufficient funds
   * Account:  0x090172CD36e9f4906Af17B2C36D662E69f162282
   * Balance:  0 wei
   * Message:  sender doesn't have enough funds to send tx. The upfront cost is: 1410000000000000000 and the sender's account only has: 0
   * Try:
      + Using an adequately funded account
```

If you didn't unlock the account, you'll see this error:

```
Error:  *** Deployment Failed ***

"Migrations" -- Returned error: authentication needed: password or unlock.
```

## Interacting with your contract with React UI

To interact with the contract, we will be using `Main.js`, `BuyForm.js` & `SellForm.js` components. Here are the details of the components:

**Main.js**

In this file, we must import the `BuyForm` and `SellForm` components, then render them.

```javascript
import React, { Component } from "react";
import BuyForm from "./BuyForm";
import SellForm from "./SellForm";

class Main extends Component {
  constructor(props) {
    super(props);
    this.state = {
      currentForm: "buy",
    };
  }

  handleTokenChange = (token) => {
    this.props.handleTokenChange(token);
  };

  render() {
    let content;
    if (this.state.currentForm === "buy")
      content = (
        <BuyForm
          selectedToken={this.props.selectedToken}
          ethBalance={this.props.ethBalance}
          tokenBalance={this.props.tokenBalance}
          buyTokens={this.props.buyTokens}
          handleTokenChange={this.handleTokenChange}
        />
      );
    else
      content = (
        <SellForm
          selectedToken={this.props.selectedToken}
          ethBalance={this.props.ethBalance}
          tokenBalance={this.props.tokenBalance}
          sellTokens={this.props.sellTokens}
          handleTokenChange={this.handleTokenChange}
        />
      );
    return (
      <div id="content" className="mt-3">
        <div className="d-flex justify-content-between mb-3">
          <button
            className={
              this.state.currentForm === "buy"
                ? "btn btn-primary"
                : "btn btn-light"
            }
            onClick={(event) => {
              this.setState({ currentForm: "buy" });
            }}
          >
            Buy
          </button>
          <span>&lt; &nbsp; &gt;</span>
          <button
            className={
              this.state.currentForm === "sell"
                ? "btn btn-primary"
                : "btn btn-light"
            }
            onClick={(event) => {
              this.setState({ currentForm: "sell" });
            }}
          >
            Sell
          </button>
        </div>

        <div className="card mb-4">
          <div className="card-body">{content}</div>
        </div>
      </div>
    );
  }
}

export default Main;
```

**Buy Tokens:**

```
result = await avaSwap.buyTokens({ from : investor, value: web3.utils.toWei('1', 'Ether')})
```

In the BuyForm component, import the logos and ABIs of the AVAX, Dai & ChainLink tokens.

BuyForm.js

```javascript
import React, { Component } from "react";
import avaxLogo from "../avax-logo.png";
import tokenLogo from "../token-logo.png";
import daiLogo from "../dai-logo.png";
import chainLinkLogo from "../chainlink-link-logo.png";

class BuyForm extends Component {
  constructor(props) {
    super(props);
    this.state = {
      output: "0",
      rate: 100,
      selected: props.selectedToken.name,
    };
  }

  handleChange = (event) => {
    this.setState({ selected: event.target.value });
    this.props.handleTokenChange(event.target.value);
  };

  render() {
    let { selected, rate } = this.state;
    return (
      <form
        className="mb-5"
        onSubmit={(event) => {
          event.preventDefault();
          let avaxAmount;
          avaxAmount = this.input.value.toString();
          avaxAmount = window.web3.utils.toWei(avaxAmount, "Ether");
          this.props.buyTokens(avaxAmount);
        }}
      >
        <div>
          <label className="float-left">
            <b>Input</b>
          </label>
          <span className="float-right text-muted">
            Balance: {window.web3.utils.fromWei(this.props.ethBalance, "Ether")}
          </span>
        </div>
        <div className="input-group mb-4">
          <input
            type="text"
            onChange={(event) => {
              const avaxAmount = this.input.value.toString();
              this.setState({
                output: avaxAmount * rate,
              });
            }}
            ref={(input) => {
              this.input = input;
            }}
            placeholder="0"
            className="form-control form-control-lg"
            required
          />
          <div className="input-group-append">
            <div className="input-group-text">
              &nbsp;&nbsp;&nbsp;
              <img src={avaxLogo} height="32" alt="" />
              &nbsp;&nbsp;&nbsp; AVAX &nbsp;&nbsp;&nbsp;
            </div>
          </div>
        </div>
        <div>
          <label className="float-left">
            <b>Output</b>
          </label>
          <span className="float-right text-muted">
            Balance:{" "}
            {window.web3.utils.fromWei(this.props.tokenBalance, "Ether")}
          </span>
        </div>
        <div className="input-group mb-2">
          <input
            value={this.state.output}
            type="text"
            placeholder="0"
            className="form-control form-control-lg"
            disabled
          />
          <div className="input-group-append">
            <div className="input-group-text">
              <img
                src={
                  selected === "LINK"
                    ? chainLinkLogo
                    : selected === "DAI"
                    ? daiLogo
                    : tokenLogo
                }
                height="32"
                alt=""
              />
              &nbsp;
              <select onChange={this.handleChange}>
                <option defaultValue={selected}>LINK</option>
                <option defaultValue={selected}>DEV</option>
                <option defaultValue={selected}>DAI</option>
              </select>
            </div>
          </div>
        </div>
        <div className="mb-5">
          <span className="float-left text-muted">
            <b>Exchange Rate</b>
          </span>
          <span className="float-right text-muted">
            1 AVAX = {rate} {selected}
          </span>
        </div>
        <button type="submit" className="btn btn-primary btn-block btn-lg">
          SWAP!
        </button>
      </form>
    );
  }
}

export default BuyForm;
```

**Sell Tokens:**

The investor must approve the token before transaction :

```
result = await Token.approve(avaSwap.address, tokens('100'), { from: investor})
```

The investor sells tokens :

```
result = await avaSwap.sellToken(tokens('100'), { from: investor })
```

SellForm.js:

```javascript
import React, { Component } from "react";
import avaxLogo from "../avax-logo.png";
import tokenLogo from "../token-logo.png";
import daiLogo from "../dai-logo.png";
import chainLinkLogo from "../chainlink-link-logo.png";

class SellForm extends Component {
  constructor(props) {
    super(props);
    this.state = {
      output: "0",
      selected: props.selectedToken.name,
    };
  }

  handleChange = (event) => {
    this.setState({ selected: event.target.value });
    this.props.handleTokenChange(event.target.value);
  };

  render() {
    let { selected } = this.state;
    return (
      <form
        className="mb-5"
        onSubmit={(event) => {
          event.preventDefault();
          let tokenAmount;
          tokenAmount = this.input.value.toString();
          tokenAmount = window.web3.utils.toWei(tokenAmount, "Ether");
          this.props.sellTokens(tokenAmount);
        }}
      >
        <div>
          <label className="float-left">
            <b>Input</b>
          </label>
          <span className="float-right text-muted">
            Balance:{" "}
            {window.web3.utils.fromWei(this.props.tokenBalance, "Ether")}
          </span>
        </div>
        <div className="input-group mb-4">
          <input
            type="text"
            onChange={(event) => {
              const tokenAmount = this.input.value.toString();
              this.setState({
                output: tokenAmount / 100,
              });
            }}
            ref={(input) => {
              this.input = input;
            }}
            placeholder="0"
            className="form-control form-control-lg"
            required
          />
          <div className="input-group-append">
            <div className="input-group-text">
              <img
                src={
                  selected === "LINK"
                    ? chainLinkLogo
                    : selected === "DAI"
                    ? daiLogo
                    : tokenLogo
                }
                height="32"
                alt=""
              />
              &nbsp;
              <select onChange={this.handleChange}>
                <option selected={selected === "LINK"} defaultValue="LINK">
                  LINK
                </option>
                <option selected={selected === "DEV"} defaultValue="DEV">
                  DEV
                </option>
                <option selected={selected === "DAI"} defaultValue="DAI">
                  DAI
                </option>
              </select>
            </div>
          </div>
        </div>
        <div>
          <label className="float-left">
            <b>Output</b>
          </label>
          <span className="float-right text-muted">
            Balance: {window.web3.utils.fromWei(this.props.ethBalance, "Ether")}
          </span>
        </div>
        <div className="input-group mb-2">
          <input
            value={this.state.output}
            type="text"
            placeholder="0"
            className="form-control form-control-lg"
            disabled
          />
          <div className="input-group-append">
            <div className="input-group-text">
              &nbsp;&nbsp;&nbsp;
              <img src={avaxLogo} height="32" alt="" />
              &nbsp;&nbsp;&nbsp; AVAX &nbsp;&nbsp;&nbsp;
            </div>
          </div>
        </div>
        <div className="mb-5">
          <span className="float-left text-muted">
            <b>Exchange Rate</b>
          </span>
          <span className="float-right text-muted">
            100 {selected} = 1 AVAX
          </span>
        </div>
        <button type="submit" className="btn btn-primary btn-block btn-lg">
          SWAP!
        </button>
      </form>
    );
  }
}

export default SellForm;
```

Check AVAX balance:

```
let AvaxBalance = await web3.eth.getBalance(avaSwap.address)
```

Check Token balance:

```
let TokenBalance = await Token.balanceOf(avaSwap.address)
```

# Summary

Now you know about creating a Decentralized Exchange (DEX) with Trufflesuite and ReactJS on the Avalanche network.

If you had any difficulties following this tutorial or simply want to discuss Avalanche tech with us you can [**join our community today**](https://community.figment.io/) or [**Join our discord channel**](https://discord.gg/fszyM7K)!

# About the author

[Devendra Yadav](https://community.figment.io/u/dev.koold)

# References

- https://learn.figment.io/network-documentation/avalanche/tutorials/getting-started-with-smart-contracts-development/using-truffle-with-the-avalanche-c-chain
- https://github.com/OpenZeppelin/openzeppelin-contracts
- https://github.com/makerdao/dss
- https://github.com/smartcontractkit/LinkToken
- https://github.com/devilla/Avaswap
