# Create an NFT with royalties on Avalanche

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

Open file ERC721.sol to add royalties in contract.

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

After adding royalties in contract ERC721 add uint8 royalties in constructor.

```
    constructor(string memory name_, string memory symbol_, uint8 royalties_) {
        _name = name_;
        _symbol = symbol_;
        _royalties = royalties_;
    }
```


Also, to get the royalties from smart contract add function royalties().

```
    function royalties() public view virtual returns (uint8) {
        return _royalties;
    }
```

Then compile the ERC721 smart contract and check for warnings, make sure the declared solidity version and compiler version are similar.

Now, Download and install the metamask wallet on your browser and connect to the Avalanche Fuji C-Chain. Make sure the parameters like Network Name, RPC URL, Chain ID etc. are exactly the same as shown in the image below.

Request AVAX from the test faucet for the gas fees for deployment.

Pass the required params i.e. Token Name, Symbol and Royalties. Eg. "Peter", "PTR", 15 and then press the deploy button.

![](/.gitbook/assets/ERC721-Deploy.png)

Now, call the respective functions i.e. *name*, *royalties* and *symbol* from the deployed ERC721 smart contract.

![](/.gitbook/assets/Deploy-And-Run-Transections.png)

Now copy the address of the deployed contract and import that in the avalanche wallet. In the collectibles tab in the portfolio section.

![](/.gitbook/assets/Add-Collectibles.png)

Congrats! You have successfully added the NFT as collectible in the avalanche wallet.
