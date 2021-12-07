Non-fungible tokens (NFTs) are a fascinating technology enabled by the ledger capabilities of blockchain protocols. They allow us to prove ownership over digital assets by providing an ownership record that is decentralized and easy to verify.

In our case we'll be leveraging a standard referred to as [ERC-721](https://ethereum.org/en/developers/docs/standards/tokens/erc-721/), which defines the basic functionality of an NFT. More importantly, there are standard smart contract implementations that we can leverage to minimize our work and ensure security.

{% sidenote title="Box 6.1: A note on smart contracts" %}
Smart contracts are computer programs that run on certain blockchain protocols and can be executed to perform actions on the blockchain. They are deterministic and have specific, limited functionality designed to react to an input with certain predictable state changes. A common physical analogy are vending machines. You insert a predefined amount of money, and the machine drops a snack into a slot for you to pick up. Note that, as with many labels in crypto, the term smart contract can be a bit confusing since these are not smart in the intelligence sense of the word and they're certainly not legal contracts.

# Development Tools 🛠

We'll be leveraging [OpenZeppelin](https://openzeppelin.com/), a library of standard smart contracts that have been audited for security and are fully composable. As with any library or package you may have used before, this allows us to leverage off-the-shelf functionality and extend it if we need anything custom.

[Hardhat](https://hardhat.org/) is another important tool that we'll be leveraging as a development environment. It will allow us to compile [Solidity](https://docs.soliditylang.org/), which is the programming language we'll use to write our smart contract. Moreover, it will provide us with a testing framework for unit tests, and will facilitate the deployment to Polygon when we're ready.

# Implementation 🧩

If you navigate to `web3/contracts/MirrorClone.sol` you'll notice we have scaffolded a smart contract for you already. Before diving into the `createToken` function we need to write, let's dive into the general structure of the code.

# Smart Contract Review 🧐

Solidity programs always start with the Solidity version at the top using the keyword `pragma`. This tells the compiler what version of Solidity to use.

In this case, we've also imported a few pre-built contracts from the OpenZeppelin library. Our contract will be inheriting the basic functionality of an [ERC-721 token](https://ethereum.org/en/developers/docs/standards/tokens/erc-721/). That will allow us to focus on the functionality we want to add instead of having to write it from scratch.

You can think of the smart contract as a class with properties, a constructor, and methods. That maps pretty closely to the subsequent lines, which include defining the properties for our smart contract (e.g. a mapping called `tokenURIToTokenId`), setting up a constructor that takes in a name and symbol, and adding functions to extend the inherited functionality.

# Creating Tokens 🏭

Focusing on the `createToken` function, the first instruction tells us we need to make sure we are passing a `tokenURI` before creating a token. In this context, the token URI will be the transaction id, also known as the transaction hash, from Arweave.

We'll need to leverage Solidity's `require` function. If the requirement fails, we should return a simple, descriptive error message.

```solidity
require(bytes(_tokenURI).length > 0, "Empty tokenURI");
```

Next we should increment the counter for `tokenIds` so we can assign a unique token ID for each entry.

```solidity
_tokenIds.increment();
uint256 newItemId = _tokenIds.current();
```

We can then leverage the `_safeMint` function we inherited from the **ERC721.sol contract** to create a token for the entry's author. The intent will be for the dApp to immediately mint an entry's NFT when the author clicks publish.

Once minted we should set the token's URI to link it to the entry using the Arweave transaction hash. We should also add this relationship to our mapping tracker, `tokenURIToTokenId`, so we can later query NFTs by their token URI.

Finally, we should emit an event that includes the author's address, the token ID and the token URI before the function returns the token ID.

```solidity
_safeMint(msg.sender, newItemId);
_setTokenURI(newItemId, _tokenURI);
tokenURIToTokenId[_tokenURI] = newItemId;

emit TokenMinted(msg.sender, newItemId, _tokenURI);

return newItemId;
```

# Testing the Smart Contract 🧪

Smart contracts are no different than any other programs we write, so we should write tests to help us build robust contracts and provide a safety net for changes throughout development. We have already written 3 tests for you, and you can confirm they're passing by running:

```text
$ yarn web3:compile
$ yarn web3:test
```

All three of these should be passing at this point because we implemented the code above. But to sanity check, we can comment out the code for the `createToken` function we just wrote. If we run the tests again, notice that they will fail:

```text
$ yarn web3:test
yarn run v1.22.11
$ npx hardhat test
.
.
.
MirrorClone
    methods
      createToken
        1) reverts when empty tokenURI passed
        2) mints new token
        3) emits TokenMinted event
0 passing
3 failing
```

If you uncomment the code again, everything will be nice and green.

![Nice and green, just the way we like it.](https://raw.githubusercontent.com/figment-networks/learn-tutorials/mirror-tutorial/mirror/assets/green.jpeg?raw=true)

{% label %}
Nice and green, just the way we like it.

Let's write one more test for the smart contract. In Solidity, mappings return 0 when a key doesn't have a value assigned to it. In other words, all keys exist with a default value of 0. We can write a test to make sure that any `tokenURI` that doesn't exist returns 0.

Add the following test to `web3/test/index.test.ts` :

```typescript
describe('tokenURIToTokenId', () => {
  it('returns 0 if tokenURI does not exist', async () => {
    expect(await contract.tokenURIToTokenId('ar://does-not-exist')).to.eq(
      0,
    );
  });
});
```

Running the tests again should confirm all four tests are passing. 

# Deploying the Smart Contract 🚀

Now that we have a functional smart contract that works as intended, we're ready to deploy. We're going to leverage the Polygon protocol to avoid the high gas fees on Ethereum. For our purposes Polygon works identically and is fully compatible with the Ethereum Virtual Machine.

Moreover, we're going to be deploying to Polygon's Mumbai testnet so you don't have to use valuable tokens to complete this dApp. In fact, if you were developing this for production, you'd deploy it to testnet first in order to test the functionality before deploying it to mainnet. Most production projects do that to make sure that mainnet deployments have been battle tested.

{% sidenote title="Box 6.1: What's a testnet?" %}
If you're unfamiliar with the concept of various networks, you can think of it as different environments for an app in Web 2 (e.g. development, test, production, etc). Most protocols have a mainnet blockchain for production deployments with real economic value, and a testnet for experimentation. The testnet matches the mainnet in functionality but doesn't manage economic value.

In order to deploy the contract, we can use the `web3/scripts/deploy.ts` file where the template provides us with a place to leverage **ethers** and **hardhat**. We can reference the `getContractFactory` to load the **MirrorClone** smart contract we just wrote. We can then use the `deploy` method passing in the name and the symbol as required by the `constructor`.

We can also include a console log that prints the smart contract public address, which we'll need to add as an environment variable shortly.

```typescript
const MirrorClone = await ethers.getContractFactory('MirrorClone');
const mirrorClone = await MirrorClone.deploy('Mirror Clone', 'MRM');

await mirrorClone.deployed();

console.log('MirrorClone deployed to:', mirrorClone.address);
```

Now we can go to the command line and deploy the contract. Note that this can fail from connectivity issues sometimes. If it does, just try again.

```text
$ yarn web3:deploy:testnet
```

The contract public address was logged to the console by our deployment script. Let's copy that and replace the default zeros in `.env.development` for the `NEXT_PUBLIC_CONTRACT_ADDRESS` environment variable. The dApp will leverage this when publishing entries to assign them NFTs.

We'll need to restart the local server for the changed variable to be available. Stop the running server by pressing CTRL+C in the terminal where it is running. Start it again with the command:

```text
$ ARWEAVE_WALLET=$(cat arweave-wallet.json) yarn dev
```

We'll also want to verify the contract, which allows chain explorers like [Polygonscan](https://polygonscan.com/) and [Etherscan](https://etherscan.io/) to confirm the deployed contract matches the source code and to display the source code for developers and users to review.

Note that you should replace the `INSERT_CONTRACT_ADDRESS_HERE` in the following command with your smart contract public address. It is also very important to pass the same constructor arguments that were used during deployment of the contract - make sure the 'Mirror Clone' and 'MRM' arguments are using identical case and spacing as the arguments passed to `MirrorClone.deploy()`.

```text
$ yarn web3:verify:testnet INSERT_CONTRACT_ADDRESS_HERE 'Mirror Clone' 'MRM'
```

If the verification fails, double check your `PRIVATE_KEY` and `ETHERSCAN_API_KEY`. Also make sure that the name you passed in as the smart contract name in the verify command above matches the name you gave your API key on Polygonscan.

If you visit [Polygonscan's testnet explorer](https://mumbai.polygonscan.com/), you can copy-paste the contract address from the command line into the search box and confirm the contract is on the blockchain!

![If a contract deployment isn't reason for fireworks, I don't know what is. ](https://raw.githubusercontent.com/figment-networks/learn-tutorials/mirror-tutorial/mirror/assets/fireworks.jpeg?raw=true)

{% label %}
If a contract deployment isn't reason for fireworks, I don't know what is.

# Minting NFTs for Entries 🍬

Recall from Step 3 that we wrote a function that created entries in Arweave. We now need to integrate the NFT minting functionality into that workflow. If we return to `CreateEntryForm.tsx`, we notice instructions at the bottom of the try block in `handleSubmit` for minting an NFT.

We'll need to instantiate a signer and connect the contract to the signer. We'll then need to leverage the `createToken` function we just wrote to mint the NFT.

```typescript
const signer = provider.getSigner();
const contractWithSigner = contract.connect(signer);

const resp = await contractWithSigner.createToken(transactionId);
const rec = await resp.wait();
```

If you try to create an entry now, your MetaMask wallet will ask you to sign a transaction to mint the NFT. After Arweave confirms the entry, you'll be able to navigate to the entry and see the NFT!

But wouldn't it be nice if you could transfer that NFT to someone else? Perhaps someone wants to purchase the entry from you or you want to gift ownership of the entry to a friend. We'll tackle that next in Step 7.

##### _Listing 6.1: Code for smart contract_
```javascript
function createToken(string memory _tokenURI) public returns (uint) {
    // require statement to check if _tokenURI is not empty
    require(bytes(_tokenURI).length > 0, "Empty tokenURI");

    // Increment counter so it starts with 0
    _tokenIds.increment();
    uint256 newItemId = _tokenIds.current();

    // Mint token
    _safeMint(msg.sender, newItemId);

    // Set token URI
    _setTokenURI(newItemId, _tokenURI);

    // Set tokenURIToTokenId
    tokenURIToTokenId[_tokenURI] = newItemId;

    // Emit TokemMinted event
    emit TokenMinted(msg.sender, newItemId, _tokenURI);

    // Return new tokenId
    return newItemId;
}
```

##### _Listing 6.2: Code for tokenURIToTokenId test_
```javascript
describe('tokenURIToTokenId', () => {
  it('returns 0 if tokenURI does not exist', async () => {
    expect(await contract.tokenURIToTokenId('ar://does-not-exist')).to.eq(
      0,
    );
  });
});
```

##### _Listing 6.3: Code for deployment_
```typescript
async function main() {
  const MirrorClone = await ethers.getContractFactory('MirrorClone');
  const mirrorClone = await MirrorClone.deploy('Mirror Clone', 'MRM');

  await mirrorClone.deployed();

  console.log('MirrorClone deployed to:', mirrorClone.address);
}
```

##### _Listing 6.4: Code for minting the NFT when publishing post_
```typescript
const handleSubmit = useCallback(
.
.
.
    try {
      .
      .
      .
      if (provider && contract) {
        .
        .
        .
        // Mint NFT
        const signer = provider.getSigner();
        const contractWithSigner = contract.connect(signer);

        const resp = await contractWithSigner.createToken(transactionId);
        const rec = await resp.wait();

        alert('Entry created successfully');
      }
    .
    .
    .
```

# Challenge 🏋️

Open `web3/contracts/MirrorClone.sol`, `index.test.ts`, `deploy.ts`, and `CreatePostForm.tsx` in your editor and follow the steps included as comments to finish writing the smart contract, its tests, the deploy script and the NFT minting functionality. We include a description along with a link to the documentation you need to review in order to implement each line. The relevant code blocks are also included in the listings below.

##### _Listing 6.5: Instructions for smart contract_
```javascript
function createToken(string memory _tokenURI) public returns (uint) {
    // require statement to check if _tokenURI is not empty

    // Increment counter so it starts with 0

    // Mint token

    // Set token URI

    // Set tokenURIToTokenId

    // Emit TokenMinted event

    // Return new tokenId
}
```

##### _Listing 6.2: Code for tokenURIToTokenId test_
```javascript
// Write a describe block that tests for a 0 return when a token URI doesn't exist in the tokenURIToTokenId mapping
```

##### _Listing 6.3: Code for deployment_
```javascript
async function main() {
  // Deploy MirrorClone smart contract
  // More information can be found here: https://hardhat.org/guides/deploying.html

  console.log('MirrorClone deployed to:', '<CONTRACT ADDRESS>');
}
```

##### _Listing 6.4: Code for minting the NFT when publishing post_
```javascript
const handleSubmit = useCallback(
.
.
.
    try {
      .
      .
      .
      if (provider && contract) {
        .
        .
        .
        // Mint NFT
        // Get signer and connect it to smart contract
        // More information can be found here: https://docs.ethers.io/v5/getting-started/#getting-started--writing

        // Call `createToken` method passing in transactionId
      }
    .
    .
    .
```

Once you have completed the code, creating an entry will mint it as an NFT automatically:

![Screenshot of NFT](https://raw.githubusercontent.com/figment-networks/learn-tutorials/mirror-tutorial/mirror/assets/nft.jpg?raw=true)