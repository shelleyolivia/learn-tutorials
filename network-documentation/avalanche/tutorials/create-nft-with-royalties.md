---
description: 'Learn how to create an NFT with royalties on Avalanche.'
---

# Create an NFT with royalties on Avalanche

## Introduction

An ERC721 token is a type of Non-Fungible Token (NFT) which can contain or reference metadata in the form of art or digital content such as pictures, audio, social media posts, websites and even generative art projects in a broader perspective - even Virtual Reality and Augmented Reality!

An NFT is represented by a discrete unit of data on a digital ledger known as a blockchain, where each token represents some unique digital properties, that cannot be interchanged.
These tokens enable many use cases that would be impossible with interchangeable or "fungible" tokens, like specific utility, proof of ownership, and a unique asset transaction history.

Refer to the official Ethereum developer documentation to learn more about [ERC-721 tokens](https://ethereum.org/en/developers/docs/standards/tokens/erc-721/).

## Prerequisites

In preparation for the tutorial, you will need to have a basic understanding of [Remix IDE](https://remix.ethereum.org/) and Solidity [smart contracts](https://solidity-by-example.org/). Please refer to the Avalanche [smart contract tutorial](https://learn.figment.io/network-documentation/avalanche/tutorials/deploy-a-smart-contract-on-avalanche-using-remix-and-metamask) for more information on the Avalanche wallet.

## Write a smart contract using Remix IDE

In the Remix IDE, create an ERC721 smart contract imported from OpenZeppelin.

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

![](/.gitbook/assets/create-ERC721-contract.png)

Go to the second tab in the left sidebar to compile and install dependecies.

![](/.gitbook/assets/compile-and-install-deps.png)

Open the file ERC721.sol in the ```.deps/npm/openzeppelin/contracts/token/ERC721/ERC721.sol``` 
directory to declare a royalties variable in the contract.

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

![](/.gitbook/assets/add-royalties-ERC721.png)

After declaring the royalties variable in the ERC721 contract, we must also initialize it in the constructor.

```
    constructor(string memory name_, string memory symbol_, uint8 royalties_) {
        _name = name_;
        _symbol = symbol_;
        _royalties = royalties_;
    }
```

![](/.gitbook/assets/initialize-royalties-ERC721.png)

To get the current value of the royalties variable from the smart contract, create a function royalties().

```
    function royalties() public view virtual returns (uint8) {
        return _royalties;
    }
```

We can now compile the amended ERC721 smart contract and check for warnings, making sure the declared solidity version and compiler version are similar. The Solidity compiler uses [semantic versioning](http://semver.org/), so be aware that for example the version strings ```0.8.0``` and ```^0.8.0``` are not equal.

![](/.gitbook/assets/compile-ERC721.png)

Next, download and install the Metamask wallet for your browser. Create a new wallet in Metamask, then [follow our quick guide](https://learn.figment.io/network-documentation/avalanche/tutorials/deploy-a-smart-contract-on-avalanche-using-remix-and-metamask#step-1-setting-up-metamask) to connect to the Avalanche Fuji C-Chain

Request AVAX from the test faucet for the deployment, from https://faucet.avax-test.network/.

Pass the required params i.e. Token Name, Symbol and Royalties. Eg. "Peter", "PTR", 15 and then press the deploy button.

![](/.gitbook/assets/ERC721-Deploy.png)

Now, call the respective functions i.e. *name*, *royalties* and *symbol* from the deployed ERC721 smart contract.

![](/.gitbook/assets/Deploy-And-Run-Transections.png)

Copy the address of the deployed contract to import in the Avalanche wallet. It should now appear on the collectibles tab in the portfolio section inside your [Avalanche wallet](https://wallet.avax.network/).

![](/.gitbook/assets/Add-Collectibles.png)

Congratulations! You have successfully added the NFT as collectible in the Avalanche wallet.

## Summary

Thank you for completing this tutorial, you should now know how to create an NFT with royalties on Avalanche using a simple Solidity smart contract. Have fun with it! If you would like to know the technical background of how NFTs work on the Avalanche network or would like to build products using NFTs, please check out this [Avalanche NFT tutorial](https://learn.figment.io/network-documentation/avalanche/tutorials/create-mint-transfer-nft)!.

If you had any difficulties following this tutorial or simply want to discuss Avalanche tech with us you can [**join our community today**](https://community.figment.io/) or [**Join our discord channel**](https://discord.gg/fszyM7K)!

## About the author

[Devendra Yadav](https://community.figment.io/u/dev.koold)

## References

This tutorial is based on the official [Avalanche Documentation](https://docs.avax.network/build/tutorials/smart-contracts/deploy-a-smart-contract-on-avalanche-using-remix-and-metamask).
