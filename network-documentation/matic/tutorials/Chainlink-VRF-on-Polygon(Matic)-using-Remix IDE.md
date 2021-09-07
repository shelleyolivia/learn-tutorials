#  Chainlink VRF on Polygon (Matic) using Remix IDE 
Learn how to generate random numbers in your smart contracts on Polygon (Matic)

## Introduction
This tutorial is a brief introduction to generating random numbers in your smart contracts using Chainlink VRF (Verifiable Random Function). Chainlink VRF is used for verifiable source of randomness.  We will be using Polygon \(Matic\) Mumbai Testnet and Remix IDE.

## Why used Chainlink VRF?
Solidity contracts are deterministic. Anyone who figures out how your contract produces randomness can predictable its results. Chainlink VRF generates a random number offchain with cryptographic proof.

## Prerequisites
You should have some basic understanding of Solidity [smart contracts](https://solidity-by-example.org/), [Remix IDE](https://remix.ethereum.org/), and completed [Polygon (Matic) Pathway](https://learn.figment.io/network-documentation/matic/polygon-matic-pathway).

## Requirements
* [MetaMask](https://metamask.io)

## Using Chainlink VRF 

### 1. Getting testnet tokens
You will need testnet MATIC and LINK for this tutorial. MATIC token will be used to pay for contract deployment and making transaction.  LINK token will be used to pay chainlink oracle for randomness.  

Open your Metamask and change the network to Polygon \(Matic\) Mumbai Testnet and copy your address.  If you do not have Polygon \(Matic\) Mumbai Testnet on your Metamask, here is a link [Configure Polygon on Metamask](https://docs.matic.network/docs/develop/metamask/config-polygon-on-metamask) on how to add it and make sure you select `Mumbai Testnet` tab to see the configuration.

Go to the [Matic Faucet](https://faucet.matic.network/) to get some MATIC and LINK testnet tokens by pasting your address.

You can go to this URL [https://mumbai.polygonscan.com/](https://mumbai.polygonscan.com/) to check your balances by entering your address on the search bar.

To see `LINK` token on your Metamask:
1) Click on `Add Token` button
2) Enter `0x326C977E6efc84E512bB9C30f76E30c160eD06FB` on `Token Contract Address` text field.  `Token Symbol` and `Token Decimal` text fields will be prefilled when you enter the Token Contract Address.
3) Click on `Next` button
4) Click on `Confirm` button to add `LINK` token to your Metamask
![](../../../.gitbook/assets/matic-chainlinkvrf-tutorial-img1.png)

### 2. Using Remix IDE
We will use the Remix IDE - an online IDE to develop smart contracts.

Head over to [https://remix.ethereum.org](https://remix.ethereum.org/). Click on the `File Icon` to create a file and named it `RandomNumber.sol`.

![](../../../.gitbook/assets/matic-chainlinkvrf-tutorial-img2.png)

### 3. Coding smart contract
On RandomNumber.sol, copy the entire smart contract code and paste it in the editor.

```javascript
// SPDX-License-Identifier: MIT
pragma solidity 0.6.6;

import "@chainlink/contracts/src/v0.6/VRFConsumerBase.sol";

contract RandomNumber is VRFConsumerBase {
    bytes32 internal keyHash;
    uint256 internal fee;
    
    uint256 public randomResult;
    
    /**
     * Constructor inherits VRFConsumerBase
     * 
     * Network: Polygon (Matic) Mumbai Testnet
     * Chainlink VRF Coordinator address: 0x8C7382F9D8f56b33781fE506E897a4F1e2d17255
     * LINK token address:                0x326C977E6efc84E512bB9C30f76E30c160eD06FB
     * Key Hash: 0x6e75b569a01ef56d18cab6a8e71e6600d6ce853834d4a5748b720d06f878b3a4
     */
    constructor() 
        VRFConsumerBase(
            0x8C7382F9D8f56b33781fE506E897a4F1e2d17255, // VRF Coordinator
            0x326C977E6efc84E512bB9C30f76E30c160eD06FB  // LINK Token
        ) public
    {
        keyHash = 0x6e75b569a01ef56d18cab6a8e71e6600d6ce853834d4a5748b720d06f878b3a4;
        fee = 0.0001 * 10 ** 18; // 0.0001 LINK
    }
    
    /** 
     * Requests randomness 
     */
    function getRandomNumber() public returns (bytes32 requestId) {
        require(LINK.balanceOf(address(this)) > fee, "Not enough LINK - fill contract with faucet");
        return requestRandomness(keyHash, fee);
    }

    /**
     * Callback function used by VRF Coordinator
     */
    function fulfillRandomness(bytes32 requestId, uint256 randomness) internal override {
        randomResult = randomness;
    }
}
```

This will inherit from VRFConsumerBase contract for randomness.  VRFConsumerBase takes it two parameters (VRF Coordinator, LINK Token).  We will use `0x8C7382F9D8f56b33781fE506E897a4F1e2d17255` for VRF Coordinator and `0x326C977E6efc84E512bB9C30f76E30c160eD06FB` for LINK Token which will work on Polygon (Matic) Mumbai Testnet.

We will use `0x6e75b569a01ef56d18cab6a8e71e6600d6ce853834d4a5748b720d06f878b3a4` for specific chainlink oracle and set the fee to 0.0001 LINK to pay the that chainlink oracle for randomness.

```javascript
constructor() 
    VRFConsumerBase(
        0x8C7382F9D8f56b33781fE506E897a4F1e2d17255, // VRF Coordinator
        0x326C977E6efc84E512bB9C30f76E30c160eD06FB  // LINK Token
    ) public
{
    keyHash = 0x6e75b569a01ef56d18cab6a8e71e6600d6ce853834d4a5748b720d06f878b3a4;
    fee = 0.0001 * 10 ** 18; // 0.0001 LINK
}
```

This will get a random number from the chainlink oracle.  `requestRandomness()` comes from `VRFConsumerBase` contract that we imported.

```javascript
function getRandomNumber() public returns (bytes32 requestId) {
    require(LINK.balanceOf(address(this)) > fee, "Not enough LINK - fill contract with faucet");
    return requestRandomness(keyHash, fee);
}
```

This is a callback function used by VRF Coordinator to determine if the number is really random
```javascript
function fulfillRandomness(bytes32 requestId, uint256 randomness) internal override {
    randomResult = randomness;
}
```

### 4. Deploy the contract with Remix
On Remix, click on Solidity Compiler icon on the Sidebar

![](../../../.gitbook/assets/matic-chainlinkvrf-tutorial-img3.png)

Select `0.6.6+commit.6c089d02` as the compiler and click `Compile RandomNumber.sol` button to compile the smart contract. Once compiled, the smart contract is ready to be deployed onto the Polygon \(Matic\) Mumbai Testnet.

Click on `Solidity Compiler` icon on the Sidebar.

![](../../../.gitbook/assets/matic-chainlinkvrf-tutorial-img4.png)

Select `Injected Web3 as the environment` in the dropdown. It will connect to your Metamask and find the network ID and account address.  80001 is the network ID for Polygon \(Matic\) Mumbai Testnet.

Select `RandomNumberConsumer - RandomNumber.sol` as the contract in the dropdown and click Deploy button.

Metamask should popup and click Confirm button to confirm the transaction.  This should deploy the contract to the 
Polygon \(Matic\) Mumbai Testnet.

![](../../../.gitbook/assets/matic-chainlinkvrf-tutorial-img5.png)

You should see your deployed contract and transaction information.

![](../../../.gitbook/assets/matic-chainlinkvrf-tutorial-img6.png)

### 5. Fund your contract with Link
You will need to fund your contract with LINK token for random number to work.  Click on `Copy Icon` to copy your contract address.

![](../../../.gitbook/assets/matic-chainlinkvrf-tutorial-img7.png)

Go to your Metamask:
1) Click on the `Link` token
2) Click on `Send` button
3) Paste your contract address on the search bar
4) Set Amount to `1` Link
5) Click on `Next` button
6) Click on `Confirm` button to confirm the transaction

You can go to this URL [https://mumbai.polygonscan.com/](https://mumbai.polygonscan.com/) to check your contract balances by entering your contract address on the search bar.

![](../../../.gitbook/assets/matic-chainlinkvrf-tutorial-img8.png)

### 6. Trying contract
Click on this `Arrow Icon` to see your contract methods

![](../../../.gitbook/assets/matic-chainlinkvrf-tutorial-img9.png)

Click on `getRandomNumber` button.  You should see a metamask popup.  Click on `Confirm` button to confirm the transaction.

When the transaction is success, click on `randomResult` button and you should see a random number that is not zero.

**Note:** It may take around 1 min for random number to change.

![](../../../.gitbook/assets/matic-chainlinkvrf-tutorial-img10.png)

## Common Error
If you get this error, it mean that your contract does not have enough `LINK` token and you should send some `LINK` token to your contract.
![](../../../.gitbook/assets/matic-chainlinkvrf-tutorial-img11.png)


## Conclusion
Congratulates! This tutorial was aimed give introduction to Chainlink VRF.  Now you can generate random numbers in your smart contracts using Chainlink VRF.

To learn more about Chainlink VRF, visit [Chainklink Doc](https://docs.chain.link/docs/chainlink-vrf/).

## About The Author
This tutorial was created by You Song Hou who is a full stack developer.

## References
- Chainlink VRF Documentation: https://docs.chain.link/docs/chainlink-vrf/
- Polygon (Matic) Documentation: https://docs.matic.network/docs/develop/getting-started