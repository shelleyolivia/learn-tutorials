
# Introduction

[Non-Fungible Tokens](https://en.wikipedia.org/wiki/Non-fungible_token) are unique records of ownership on the blockchain. Usually an NFT is tied to something interesting and rare, such as an artwork, a concert ticket, a collectable cat, a domain name, or a real physical object. NFTs can be bought, sold, given away, and can even be minted or destroyed, depending on the rules of the contract. [Cryptokitties](https://www.cryptokitties.co/) and [SuperRare](https://superrare.co/) are two popular examples of Ethereum-based NFTs. We can implement NFTs just as easily on NEAR.
TODO: BETTER EXAMPLES IN 2022?

In this tutorial we will create a simple collectable NFT artwork called a Flarn, consisting of a name, a description and a JPEG image. Our smart contract will allow Flarns to be created, collected and traded on the NEAR blockchain.  

## About NFT standards

TODO: REPLACE MOST OF THIS WITH ONE SENTENCE ABOUT NEP-171 & ITS PURPOSE.
There are many standards for NFTs! However the most widely used by far is Ethereum's [ERC721](https://eips.ethereum.org/EIPS/eip-721) standard, which defines how NFTs can be created and transfered among other things. This standard works well, but like all the ERC standards it is defined only for the Ethereum blockchain. Recent high gas prices on Ethereum have driven a lot of interest in alternatives, and NEAR has a lot to offer here. 

To support NFT development on NEAR, the core developers and marketplace stakeholders have proposed the new NEP-171 standard.  NEP-171 attempts to handle all of the same NFT use-cases as ERC721, as well as to support a number of advanced features for NFT marketplaces.  For instance, NEP-171 NFTs can be offered for sale in multiple marketplaces at once, and complex distributions of royalties are baked in.  Also, NFTs can be attached directly to smart contract calls, as a form of payment or as part of more complex trading projects. 

# Prerequisites

 If you have completed [the NEAR pathway](https://learn.figment.io/pathways/near-pathway) and  [the Figment NEAR + Rust tutorial](https://learn.figment.io/tutorials/write-and-deploy-a-smart-contract-on-near), you should have already taken care of all prerequisites. For this tutorial you must:

* Install Node.js and npm, and set up your DataHub environment
* [Use npm to install the NEAR CLI](https://www.npmjs.com/package/near-cli)
* [Create a wallet on the NEAR Testnet](https://nearhelp.zendesk.com/hc/en-us/articles/1500002248242-Creating-a-NEAR-Wallet-account)
* [Install Rust and Rustup](https://www.rust-lang.org/tools/install)


# Setting up the project
## Install the WebAssembly target using rustup

NEAR uses WebAssembly as its virtual machine.  Many languages can compile to WebAssembly, but NEAR core developers recommend that all financial smart contracts be written in the Rust language. Before we can start working on the Rust contract, we need to install the cross-compilation tools that compile a Rust smart contract into WebAssembly, using Rustup.  Run this command:

```text
rustup target add wasm32-unknown-unknown
```

TODO: expected output?

## Create a working directory
Pick a location for all the project files, and initialize the npm catalog in that location.

```text
mkdir flarns2.0 
cd flarns2.0
npm init
```

## Clone the NEAR NFT repo

In this tutorial we'll modify NEAR's NFT-171 example code from the NEAR repository on Github. On Unix, run these commands in the `bash` shell to clone that repo and install its requirements:

```text
git clone https://github.com/near-examples/NFT
```

The file README.md explains the purpose of the various files and directories in the repo ... TODO FINISH

To build the example, enter this command in the shell:

```text
cd NFT
./build.sh
```

Because this is our first build, Cargo and Rust will download and build all the dependencies. If might take some time. When it's finished, you should see something like this at the end of the output:

TODO UPDATE
```text
   Compiling non-fungible-token v1.1.0 (/Volumes/External/mykle/Documents/near/figment/flarn2.0/NFT/nft)
    Finished release [optimized] target(s) in 1m 34s
```

We can also run all of the included unit tests with this command:

```text
cargo test -- --nocapture
```
TODO: FIX ERROR: M1 RELATED?  UPDATE XCODE, TRY ON INTEL ...

The unit test output is messy, but at the end you should see a summary of results.

```text
test result: ok. 10 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out
```

If you see `test result: ok`, all is well.


# Getting to know a Rust Contract
TODO: REVIEW THIS AFTER I GET THROUGH PREVIOUS STEPS
TODO: THE NEW VERSION IS SPREAD THROUGH SEVERAL FILES ...

The Rust smart contract that we will modify is at `nft/src/lib.rs`. Open that file in your editor. We'll visit all the sections, and make a few important additions.

## Preamble

The first section is some boilerplate that imports various NEAR and NEP-171 standard code:

```rust
use  near_contract_standards::non_fungible_token::metadata::{
	NFTContractMetadata, NonFungibleTokenMetadataProvider, TokenMetadata, NFT_METADATA_SPEC,
};
use  near_contract_standards::non_fungible_token::{Token, TokenId};
use  near_contract_standards::non_fungible_token::NonFungibleToken;
```

All the NFT-specific methods and data structures in this file come from these imports from the near_contract_standards package. This example uses the NFT core standards and the metadata standards.  A contract that wanted to use other NEAR NFT features would include other standards here, but this will be fine for our example.

After that comes a section of boilerplate code that gives us all the standard methods and structures needed by any NEAR smart contract.  There are only two parts we need to customize.  One is this section, which encodes an icon for our 

```rust
use  near_sdk::borsh::{self, BorshDeserialize, BorshSerialize};
use  near_sdk::collections::LazyOption;
use  near_sdk::json_types::ValidAccountId;
use  near_sdk::{
	env, near_bindgen, AccountId, BorshStorageKey, PanicOnDefault, Promise, PromiseOrValue,
};

near_sdk::setup_alloc!();
```

## Customize the contract
NEP-171 requires our contract to have a human-readable name and an abbreviated symbol for marketplace use.  

TODO: MAYBE DON'T TOUCH THIS PART? There is also an option for an icon to represent the contract visually.  

Let's modify the example code to give it our own name and symbol ... TODO FINISH

### build and test

# Design our first NFT
 
For this example, let's create a new kind of collectable digital NFT artwork called a Flarn. Every Flarn will include at least name and a JPEG image, plus a unique Token ID within the contract and an owner ID to record ownership.

## NFT Metadata
NEP-171 doesn't actually require NFTs to have any metadata at all -- you can still mint and trade completely blank records of ownership, if that's what you're into.  But without metadata, NFTs would be pretty boring.  Here are the types of NFT metadata supported by the standard:
-   `name`: The name of this specific token.
-   `description`: A longer description of the token.
-   `media`: URL to associated media. 
-   `media_hash`: the base64-encoded sha256 hash of content referenced by the  `media`  field. This is to guard against off-chain tampering.
-   `copies`: How many copies of this specific token exist
-   `issued_at`: Timestamp when token was issued or minted
-   `expires_at`: Timestamp when token expires
-   `starts_at`: Timestamp when token starts being valid
-   `updated_at`: Timestamp when token was last updated
-   `extra`: anything extra the NFT wants to store on-chain. Can be stringified JSON.
-   `reference`: URL to an off-chain JSON file with more info.
-   `reference_hash`: Base64-encoded sha256 hash of JSON from reference field. Required if  `reference`  is included.

For our example we'll ignore a lot of these, but we will use `name`, `description` and `media`.  Because we use `media` we are required to also use `media_hash`. We can leave `copies` blank to say our flarns are unique -- effectively `copies` defaults to `1`.

(If some future NFT project requires other metadata fields not on this list, NEP-171 gives you two options.  You can either encode it as a JSON string in the `extra` field, or place it off-chain and store a link to it in the `reference` field -- in which case you also need to store a hash of the data in the `reference_hash` field, to prevent tampering.)

## Create your masterpiece
You can either come up with your metadata at this point, or you can use this example data:

* `name`: Bruce
* `description`: Bruce is uniquely adorable & loves long walks on the beach.
* `media`: TODO: JPEG WITH DOWNLOAD LINK

## Linking to off-chain data
Due to the cost of on-chain storage, most Ethereum NFTs of digital media do not store the media on the blockchain.  On-chain storage is much cheaper on NEAR, but since media files are large, it will still be thrifty of us to store our Flarn image elsewhere.  The `media` field of our on-chain record will hold a URL pointing to our image.  We could put that on any webserver we choose, but it would be nice to put it somewhere that's always online, decentralized, and free. 

### Uploading the image to NFT Storage

For this example, we'll use the free  [NFT Storage](https://nft.storage/#getting-started) service built specifically for storing off-chain NFT data. NFT Storage offers free decentralized storage and bandwidth for NFTs on  [IPFS](https://ipfs.io/)  and  [Filecoin](https://filecoin.io/).

#### Steps

1.  [Register an account](https://nft.storage/login/)  and log in to  [nft.storage](https://nft.storage/login/).
    
2.  Go to the  [Files](https://nft.storage/files/)  section, and click on the  [Upload](https://nft.storage/new-file/)  button to upload your Flarn image.
    
    TODO: REPLACE WITH OUR OWN IMAGE NOT NEAR'S![nft.storage](https://docs.near.org/assets/images/nft-storage-4a42a724bcd3e29196d2f6320aa8d6a3.png)
    
3.  Once you have uploaded your file, you'll get a unique URL for your content, based on an IPFS Content-ID string:
    
    ```
    https://bafyreiabag3ztnhe5pg7js4bj6sxuvkz3sdf76cjvcuqjoidvnfjz7vwrq.ipfs.dweb.link/
    ```
Open that URL in a new browser tab to confirm it exists:
TODO: IMAGE
    
> **Tip:**  check the  [NFT.Storage Docs](https://nft.storage/api-docs/)  for information on uploading multiple files and available API endpoints.

### Generate the `media_hash`
The checksum of our media file stored in the `media_hash` field, must be a Base64-encoded SHA256 hash of the original data, to prove the file hasn't been tampered with.  NEP-171 uses Base64 encoding of the hash for maximum portability between platforms.  However, most online and command-line SHA256 tools generate Unicode hashes, so we also will need to convert from Unicode to Base64. There are a few different ways to do this.  On a UNIX-based system, use this command:

```
cat flarn.jpg | sha256sum | xxd -r -p | base64
TODO output
```

On the web, you can generate the media hash in two steps.  
1: Upload your file to [this online SHA256 calculator](https://emn178.github.io/online-tools/sha256_checksum.html) to generate a SHA246 hash in hexadecimal format.
TODO: screenshot
2: Copy the hexadecimal string from that tool, and paste it into [this online Hex-Base64 converter](https://base64.guru/converter/encode/hex):
TODO: screenshot
NOTE: do not upload sensitive or private data to these web services -- or to any web service, ever! Use the UNIX commands above if privacy is important.

If you used [our example image](TODO LINK), the strings should match the screenshots. If you're using your own image, the strings will be different.

## Build a complete metadata record
Now that we know our metadata, have our media stored off-chain, and have generated the media hash, we can construct our NFT Token record.  Use your editor to create a file called `token.json`, containing a single object of JSON data which will be passed to the contract's `nft_mint()` method.  You can start by pasting in this text:
```json
{
	"token_id": "0", 
	"receiver_id": "my_testnet_id", 
	"token_metadata": { 
		"name": "Bruce",
		"description": "Bruce is uniquely adorable & loves long walks on the beach.",
		"media": "https://bafyreiabag3ztnhe5pg7js4bj6sxuvkz3sdf76cjvcuqjoidvnfjz7vwrq.ipfs.dweb.link/",
		"media_hash": "e8MzVN0Pq1ztofypQoperJC8gWo2xfs8f9few+SwHes="
	}
}
```
Now edit the following values:
* Use the URL you got from NFT.storage as the value for "media".
* Use the base64 sha256 hash you generated for "media_hash", if you're using a different image file than our example.
* Use your own NEAR testnet id for "receiver_id", so that you will own the newly minted NFT.
* Change the other fields to whatever you like, or leave them as-is.

Save that file, then verify it at [JSONLint](https://jsonlint.com/)
TODO: screencap


# Deploying and using the contract


We can use the NEAR CLI to deploy this contract, and to test that it's working. If you configured your environment in Tutorials 1 and 2, the CLI will connect to DataHub's high-availability testnet. If you don't have DataHub access you can still run the CLI with its defaults, though the default testnet node may be slower to respond. 

TODO: CREATE A SEPERATE ACCT JUST FOR THIS TUT?

This smart contract will be deployed to your NEAR account. Because NEAR allows the ability to upgrade contracts on the same account, initialization functions must be cleared TODO WTF DOES THAT MEAN?

> **Note:**  If you'd like to run this example on a NEAR account that has had prior contracts deployed, please use the  `near-cli`  command  `near delete`  and then recreate it in Wallet. To create (or recreate) an account, please follow the directions in  [Test Wallet](https://wallet.testnet.near.org/)  or ([NEAR Wallet](https://wallet.near.org/)  if we're using  `mainnet`).
> TODO EXPLAIN THIS SHIT MORE CLEARLY, FIGURE OUT A TEST FOR IT ...

Log in to your newly created account with  `near-cli`  by running the following command in your terminal.

```
near login
```

To make this tutorial easier to copy/paste, we're going to set an environment variable for your account ID. In the command below, replace  `YOUR_ACCOUNT_NAME`  with the account name you just logged in with including the  `.testnet`  (or  `.near`  for  `mainnet`):  

```
export ID=YOUR_ACCOUNT_NAME
```

Test that the environment variable is set correctly by running:

```
echo $ID
```

Verify that the correct account ID is printed in the terminal. If everything looks correct you can now deploy your contract. In the root of your NFT project run the following command to deploy your smart contract.

```
near deploy --wasmFile res/non_fungible_token.wasm --accountId $ID
```

The output will show details of the deployment transaction, and the ID of the test NEAR account that the CLI auto-generated for you. It should look something like this:

```
Starting deployment. Account id: ex-1.testnet, node: https://rpc.testnet.near.org, helper: https://helper.testnet.near.org, file: res/non_fungible_token.wasmTransaction Id E1AoeTjvuNbDDdNS9SqKfoWiZT95keFrRUmsB65fVZ52To see the transaction in the transaction explorer, please open this url in your browserhttps://explorer.testnet.near.org/transactions/E1AoeTjvuNbDDdNS9SqKfoWiZT95keFrRUmsB65fVZ52Done deploying to ex-1.testnet
```


> **Note:**  For  `mainnet`  you will need to prepend your command with  `NEAR_ENV=mainnet`.  [See here](https://docs.near.org/docs/tools/near-cli#network-selection)  for more information.



The provided link will give you complete details about the deployment in the NEAR Explorer.

TODO: SCREENSHOT
TODO: supporting mainnet in this tut or not???



A smart contract can define an initialization method that can be used to set the contract's initial state. This NFT example contract must be initialized before use. For now, you'll initialize it with the default metadata, and set the owner to ourself.

> **Note:**  each account has a data area called  `storage`, which is persistent between function calls and transactions. For example, when you initialize a contract, the initial state is saved in the persistent storage.

```
near call $ID new_default_meta '{"owner_id": "'$ID'"}' --accountId $ID
```

> **Tip:**  you can find more info about the NFT metadata at  [nomicon.io](https://nomicon.io/Standards/NonFungibleToken/Metadata.html).

You can then view the contract metadata by running the following  `view`  call:

```
near view $ID nft_metadata
```

Example response:
TODO: 2.0?
```
{
  "spec": "nft-1.0.0",  
  "name": "Example NEAR non-fungible token",
  "symbol": "EXAMPLE",
  "icon": "data:image/svg+xml,%3Csvg xmlns='http://www.w3.org/2000/svg' viewBox='0 0 288 288'%3E%3Cg id='l' data-name='l'%3E%3Cpath d='M187.58,79.81l-30.1,44.69a3.2,3.2,0,0,0,4.75,4.2L191.86,103a1.2,1.2,0,0,1,2,.91v80.46a1.2,1.2,0,0,1-2.12.77L102.18,77.93A15.35,15.35,0,0,0,90.47,72.5H87.34A15.34,15.34,0,0,0,72,87.84V201.16A15.34,15.34,0,0,0,87.34,216.5h0a15.35,15.35,0,0,0,13.08-7.31l30.1-44.69a3.2,3.2,0,0,0-4.75-4.2L96.14,186a1.2,1.2,0,0,1-2-.91V104.61a1.2,1.2,0,0,1,2.12-.77l89.55,107.23a15.35,15.35,0,0,0,11.71,5.43h3.13A15.34,15.34,0,0,0,216,201.16V87.84A15.34,15.34,0,0,0,200.66,72.5h0A15.35,15.35,0,0,0,187.58,79.81Z'/%3E%3C/g%3E%3C/svg%3E",
  "base_uri": null,
  "reference": null,
  "reference_hash": null
}
```


# Mint an NFT!

Now let's mint our first token! The following command will mint one copy of your NFT:

```
near call $ID nft_mint "`cat token.json`" --accountId $ID --deposit 0.1
```

Example response:
```
{
  "token_id": "0",
  "owner_id": "dev-xxxxxx-xxxxxxx",
  "metadata": {
    "title": "Some Art",
    "description": "My NFT media",
    "media": "https://bafkreiabag3ztnhe5pg7js4bj6sxuvkz3sdf76cjvcuqjoidvnfjz7vwrq.ipfs.dweb.link/",
    "media_hash": null,
    "copies": 1,
    "issued_at": null,
    "expires_at": null,
    "starts_at": null,
    "updated_at": null,
    "extra": null,
    "reference": null,
    "reference_hash": null
  },
  "approved_account_ids": {}
}
```
This is pretty much the same data we sent, but shown as it's stored on the blockchain. Notice that `receiver_id` became `owner_id`, and that all the optional metadata fields we didn't specify are set to `null`.  The `approved_account_ids` field would list any other NEAR users who are authorized to manipulate this NFT.  But we didn't authorize anybody, so that list is empty.


To view all the flarns you own, call the NFT contract with the following  `near-cli`  command:

```
near view $ID nft_tokens_for_owner '{"account_id": "'$ID'"}'
```

Since this is your only flarn so far, the result will be a JSON array containing just one record, the one you've already seen: 
```
[
  {
    "token_id": "0",
    "owner_id": "dev-xxxxxx-xxxxxxx",
    "metadata": {
      "title": "Some Art",
      "description": "My NFT media",
      "media": "https://bafkreiabag3ztnhe5pg7js4bj6sxuvkz3sdf76cjvcuqjoidvnfjz7vwrq.ipfs.dweb.link/",
      "media_hash": null,
      "copies": 1,
      "issued_at": null,
      "expires_at": null,
      "starts_at": null,
      "updated_at": null,
      "extra": null,
      "reference": null,
      "reference_hash": null
    },
    "approved_account_ids": {}
  }
]
```

TODO: can we see this in wallet now?

# Conclusion

You now have deployed an NFT smart contract on the NEAR testnet, and have minted one CryptoFlarn NFT. From here you could use the CLI or the NEAR Javascript SDK to transfer ownership of that token, or to make more tokens. NFT marketplaces are already under development on NEAR; when they support NEP-4, you'll be able to trade these tokens there. If your next NFT project needs more complex metadata, you've seen how that can be added.

The complete code for this tutorial can be found on [Github](https://github.com/figment-networks/tutorials/tree/main/near/6_NFT).
# About the Author

This tutorial was created by [Mykle Hansen](https://github.com/myklemykle), a contributor to the [Plantary](https://github.com/myklemykle/plantary) project which allows users to grow and harvest plant NFTs on NEAR.

