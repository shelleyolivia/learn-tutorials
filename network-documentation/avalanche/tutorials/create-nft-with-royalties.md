---
description: 'Learn how to create an NFT with royalties on Avalanche.'
---

# Create an NFT with royalties on Avalanche

## Introduction

NFT is an ERC721 token which is a form of “art” that can be a picture, tweet, audio, etc in a broader perspective. A non-fungible token \(NFT\) is a unit of data on a digital ledger called blockchain, where each NFT represents something unique item, that can’t be interchanged. This enables many use cases that would be impossible with interchangeable tokens, like utility, proof of ownership, and a unique asset transaction history. Refer to the below link to learn more about [ERC-721 token](https://ethereum.org/en/developers/docs/standards/tokens/erc-721/).

## Prerequisites

In preparation for the tutorial, we will need to have some basic understanding of Remix IDE and smart contracts. Please refer to the documentation for more information on the Avalanche wallet, [here](https://docs.avax.network/build/tutorials/smart-contracts/deploy-a-smart-contract-on-avalanche-using-remix-and-metamask).

## Write a smart contract using Remix IDE

In the Remix IDE, compile and deploy an ERC721 smart contract imported from OpenZeppelin.

```
// contracts/GameItem.sol
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

import "@openzeppelin/contracts/token/ERC721/ERC721.sol";
import "@openzeppelin/contracts/token/ERC721/extensions/ERC721URIStorage.sol";
import "@openzeppelin/contracts/utils/Counters.sol";

contract GameItem is ERC721URIStorage {
    using Counters for Counters.Counter;
    Counters.Counter private _tokenIds;

    constructor() ERC721("GameItem", "ITM") {}

    function awardItem(address player, string memory tokenURI)
        public
        returns (uint256)
    {
        _tokenIds.increment();

        uint256 newItemId = _tokenIds.current();
        _mint(player, newItemId);
        _setTokenURI(newItemId, tokenURI);

        return newItemId;
    }
}
```

Open file ERC721.sol to declare royalties in contract.

```
contract ERC721 is Context, ERC165, IERC721, IERC721Metadata {
    using Address for address;
    using Strings for uint256;

    // Token name
    string private _name;

    // Token symbol
    string private _symbol;
    
    // Royalties
    uint8 private _royalties;
```

After declaration of royalties in contract ERC721 initialize uint8 royalties in constructor.

```
    constructor(string memory name_, string memory symbol_, uint8 royalties_) {
        _name = name_;
        _symbol = symbol_;
        _royalties = royalties_;
    }
```


Also, to get the royalties from smart contract create function royalties().

```
    function royalties() public view virtual returns (uint8) {
        return _royalties;
    }
```

Then compile the ERC721 smart contract and check for warnings, make sure the declared solidity version and compiler version are similar.

Now, download and install the metamask wallet on your browser and connect to the Avalanche Fuji C-Chain. Make sure the parameters like Network Name, RPC URL, Chain ID etc. are exactly the same as per the Avalanche Docs.

Request AVAX from the test faucet for the deployment, [here](https://faucet.avax-test.network/).

Pass the required params i.e. Token Name, Symbol and Royalties. Eg. "Peter", "PTR", 15 and then press the deploy button.

![](/.gitbook/assets/ERC721-Deploy.png)

Now, call the respective functions i.e. *name*, *royalties* and *symbol* from the deployed ERC721 smart contract.

![](/.gitbook/assets/Deploy-And-Run-Transections.png)

Now copy the address of the deployed contract and import that in the avalanche wallet. In the collectibles tab in the portfolio section inside your [Avalanche Wallet](https://wallet.avax.network/).

![](/.gitbook/assets/Add-Collectibles.png)

Congrats! You have successfully added the NFT as collectible in the avalanche wallet.

## Summary

Now, you should know how to create NFT with royalties using a smart contract. Have fun with it! If you would like to know the technical background of how NFTs work on the Avalanche network or would like to build products using NFTs, please check out this [Avalanche NFT tutorial](https://learn.figment.io/network-documentation/avalanche/tutorials/create-mint-transfer-nft)!.

If you had any difficulties following this tutorial or simply want to discuss Avalanche tech with us you can [**join our community today**](https://community.figment.io/) or [**Join our discord channel**](https://discord.gg/fszyM7K)!

## About the author

[Devendra Yadav](https://community.figment.io/u/dev.koold)

## References

This tutorial is based on the official [Avalanche Documentation](https://docs.avax.network/build/tutorials/smart-contracts/deploy-a-smart-contract-on-avalanche-using-remix-and-metamask).
