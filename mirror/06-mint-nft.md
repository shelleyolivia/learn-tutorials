# Step 6: Minting a Post NFT

Non-fungible tokens (NFTs) are a fascinating technology enabled by the ledger capabilities of blockchain protocols. They allow us to prove ownership over digital assets by providing an ownership record that is decentralized and easy to verify.

In our case we'll be leveraging a standard referred to as [ERC-721](https://ethereum.org/en/developers/docs/standards/tokens/erc-721/), which defines the basic functionality of an NFT. More importantly, there are standard smart contract implementations that we can leverage to minimize our work and ensure security.

{% sidenote title="Box 6.1: A note on smart contracts" %}
Smart contracts are computer programs that run on certain blockchain protocols and can be executed to perform actions on the blockchain. They are deterministic and have specific, limited functionality designed to react to an input with certain predictable state changes. A common physical analogy are vending machines. You insert a predefined amount of money, and the machine drops a snack into a slot for you to pick up. Note that, as with many labels in crypto, the term smart contract can be a bit confusing since these are not smart in the intelligence sense of the word and they're certainly not legal contracts.

## Key Development Tools - OpenZeppelin and Hardhat

We'll be leveraging [OpenZeppelin](https://openzeppelin.com/), a library of standard smart contracts that have been audited for security and are fully composable. As with any library or package you may have used before, this allows us to leverage off-the-shelf functionality and extend it if we need anything custom.

[Hardhat](https://hardhat.org/) is another important tool that we'll be leveraging as a development environment. It will allow us to compile Solidity, which is the programming language we'll use to write our smart contract. Moreover, it will provide us with a testing framework for unit tests, and will facilitate the deployment to Polygon when we're ready.

# Implementation 🧩

## Steps
* Write simple smart contract for NFT using solidity. Smart contract exposes createToken method which takes in Arweave transactionId
* Write unit test for createToken functionality
* Add createToken smart contract call to handleSubmit in CreatePostForm right after successfull creation of entry on Arweave.
* Create a NFTDetails component in which you retrieve tokenId for given transactionId and it's owner. You can display tokenId to the user to indicate that entry has a token associated with it.

##### _Listing 6.1: Code for minting a post NFT_
>>>>>>> Insert code here

# Challenge 🏋️

Navigate to `[ ]` in your editor and follow the steps included as comments to finish writing the `[ ]` function. We include a description along with a link to the documentation you need to review in order to implement each line. The relevant code block is also included in [Listing 6.2](#listing-62-instructions-for-minting-a-post-NFT) below.

##### _Listing 6.2: Instructions for minting a post NFT_
