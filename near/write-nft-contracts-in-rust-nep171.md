# Introduction

[Non-Fungible Tokens](https://en.wikipedia.org/wiki/Non-fungible_token) are unique records of ownership on the blockchain. Usually an NFT is tied to something interesting and rare, such as an artwork, a concert ticket, a collectable cat, a domain name, or a real physical object. NFTs can be bought, sold, given away, and can even be minted or destroyed, depending on the rules of the contract. [Cryptokitties](https://www.cryptokitties.co/), [SuperRare](https://superrare.co/) and [Bored Ape Yacht Club](https://boredapeyachtclub.com/) are just a few popular examples of Ethereum-based NFT projects. We can implement NFTs just as easily on NEAR.

In this tutorial we will create a simple collectable NFT artwork called a Flarn, consisting of a name, a description and a JPEG image. Our smart contract will allow Flarns to be created, collected and traded on the NEAR blockchain.  

# Prerequisites

* [The NEAR 101 pathway](https://learn.figment.io/protocols/near) 
* [How to write and deploy a smart contract in Rust ](https://learn.figment.io/tutorials/write-and-deploy-a-smart-contract-on-near)

# Requirements

* [Install Node.js and npm](https://docs.npmjs.com/downloading-and-installing-node-js-and-npm)
* [Use npm to install the NEAR CLI](https://www.npmjs.com/package/near-cli)
* [Create a wallet on the NEAR Testnet](https://nearhelp.zendesk.com/hc/en-us/articles/1500002248242-Creating-a-NEAR-Wallet-account)
* [Install Rust and Rustup](https://www.rust-lang.org/tools/install)

__NOTE FOR M1 MAC USERS__: The NFT examples repo used by this tutorial does not currently compile on Apple products with an M1 chip. If you don't have access to an Intel-based Mac, you can instead do this tutoral on __GitPod__.  [Open this link in a separate tab](https://gitpod.io/#https://github.com/near-examples/NFT), then skip ahead to the next section: __Getting To Know The NFT Contract__.

# About NFT standards

There are many standards for NFTs! However the most widely used by far is Ethereum's [ERC721](https://eips.ethereum.org/EIPS/eip-721) standard, which defines how NFTs can be created and transfered among other things. This standard works well, but like all the ERC standards it is defined only for the Ethereum blockchain. Recent high gas prices on Ethereum have driven a lot of interest in more affordable alternatives, such as NEAR.

To support NFT development on NEAR, the core developers and marketplace stakeholders have proposed the new NEP-171 standard.  NEP-171 attempts to handle all of the same NFT use-cases as ERC721, as well as optional support for a number of advanced features for NFT marketplaces.  For instance, NEP-171 NFTs can be offered for sale in multiple marketplaces at once, and complex distributions of royalties are baked in.  Also, NFTs can be attached directly to smart contract calls, as a form of payment or as part of more complex trading projects. 

# Setting up the project

**Install the WebAssembly target using rustup**:

NEAR uses WebAssembly as its virtual machine. Many languages can compile to WebAssembly, but NEAR core developers recommend that all financial smart contracts be written in the Rust language. Before we can start working on the Rust contract, we need to install the cross-compilation tools that compile a Rust smart contract into WebAssembly, using Rustup.  Run this command:

```text
rustup target add wasm32-unknown-unknown
```

Example output:

```text
**info:** downloading component 'rust-std' for 'wasm32-unknown-unknown'
**info:** installing component 'rust-std' for 'wasm32-unknown-unknown'
15.6 MiB /  15.6 MiB (100 %) 8.7 MiB/s in  1s ETA:  0s
```

# Clone the NEAR NFT repo

In this tutorial we'll modify NEAR's NFT-171 example code from the NEAR repository on Github. On Linux, Unix, or macOS run these commands in the `bash` (or `zsh`) shell to clone that repo and install its requirements:

```text
git clone https://github.com/near-examples/NFT
```

Example output:

```text
Cloning into 'NFT'...
remote: Enumerating objects: 1334, done.
remote: Counting objects: 100% (456/456), done.
remote: Compressing objects: 100% (90/90), done.
remote: Total 1334 (delta 397), reused 398 (delta 363), pack-reused 878
Receiving objects: 100% (1334/1334), 1.41 MiB | 2.40 MiB/s, done.
Resolving deltas: 100% (642/642), done.
```

To build the example, enter this command in the shell:

```text
cd NFT
./build.sh
```

Because this is our first build, Cargo and Rust will download and build all the dependencies. If might take some time. When it's finished, you should see something like this at the end of the output:

```text
Compiling near-sdk v3.1.0
Compiling near-contract-standards v3.2.0
Compiling approval-receiver v0.0.1 (/Users/mykle/Documents/near/flarns2.0/NFT/test-approval-receiver)
Compiling token-receiver v0.0.1 (/Users/mykle/Documents/near/flarns2.0/NFT/test-token-receiver)
Compiling non-fungible-token v1.1.0 (/Users/mykle/Documents/near/flarns2.0/NFT/nft)
Finished release [optimized] target(s) in 2m 07s
```

We can also run all of the included unit tests with this command:

```text
cargo test -- --nocapture
```

The unit test output is messy, but at the end you should see a summary of results.

```text
test result: ok. 10 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out
```

If you see `test result: ok`, all is well.

**NOTE: At the time of this writing, these tests fail on M1-based Macs, with an error about `get_fault_info`. We recommend M1 users use Gitpod to complete this tutorial.**

# Getting to know the NFT Contract

The Rust smart contract that we will modify is at `nft/src/lib.rs`. Open that file in your editor. We'll make a few simple changes.

If you browse this contract, you'll find it only implements three methods: `new` and `new_default_meta` for initializing a new contract, and `nft_mint` for minting a new NFT.  But the contract also imports symbols, methods and other features from a group of related packages that together provide all the core and optional features of the NEP-171 standard.  Also notice that a broad set of unit tests make up more than half of the contract file.  There are also lots of tests in the imported modules.  We will run all of the tests after we modify the contract, to be sure we haven't broken anything.

## Customize the contract

NEP-171 requires our contract to have a human-readable name and an abbreviated symbol for use by marketplaces and wallets.  These are defined in the `new_default_meta` function that starts on line 52.

Let's modify that function to use our own name and symbol.  Change the name to "CryptoFlarns" and the symbol to "FLARN".  (Also change the "base_uri" field as shown; we'll explain why shortly.)  When you're done, the method should look like this:

```rust
	/// Initializes the contract owned by `owner_id` with
	/// default metadata (for example purposes only).
	#[init]
	pub fn new_default_meta(owner_id: ValidAccountId) ->  Self {
		Self::new(
			owner_id,
			NFTContractMetadata {
				spec: NFT_METADATA_SPEC.to_string(),
				name: "CryptoFlarns".to_string(),
				symbol: "FLARN".to_string(),
				icon: Some(DATA_IMAGE_SVG_NEAR_ICON.to_string()),
				base_uri: Some("https://nft.storage/".to_string()),
				reference: None,
				reference_hash: None,
			},
		)
	}
```

NEP-171 contracts can also have an optional icon, which is shown in the NEAR wallet and used by marketplaces and other NFT apps.  Your contract doesn't need to have an icon, but since this example contract defines one, let's change it to something more colorful.  

NEP-171 advises that the `icon` field should be encoded in [data-URI format](https://en.wikipedia.org/wiki/Data_URI_scheme), for maximum compatibility with web browsers.  The icon in the example contract is a simple SVG file, data-URI encoded according to [this guide by Jenny Knuth](https://bl.ocks.org/jennyknuth/222825e315d45a738ed9d6e04c7a88d0).  Other image file formats such as GIF and PNG can also be data-URI encoded, but SVGs are nice because they can be quite small.  

Here is our very simple example icon:
![CryptoFlarns icon](https://raw.githubusercontent.com/figment-networks/learn-tutorials/master/assets/nep171/cryptoflarns_icon.svg)

The icon is defined on line 39 of the example contract, as a long string of text.  In your editor, change that very long line to this even longer line:

```rust
const DATA_IMAGE_SVG_NEAR_ICON: &str  =  "data:image/svg+xml,%3Csvg%20xmlns%3D%27http%3A%2F%2Fwww.w3.org%2F2000%2Fsvg%27%20width%3D%2764%27%20height%3D%2764%27%20xml%3Aspace%3D%27preserve%27%3E%3Crect%20width%3D%27100%25%27%20height%3D%27100%25%27%20fill%3D%27transparent%27%2F%3E%3Crect%20style%3D%27stroke%3Anone%3Bstroke-width%3A1%3Bstroke-dasharray%3Anone%3Bstroke-linecap%3Abutt%3Bstroke-dashoffset%3A0%3Bstroke-linejoin%3Amiter%3Bstroke-miterlimit%3A4%3Bfill%3A%23ffec20%3Bfill-rule%3Anonzero%3Bopacity%3A1%27%20vector-effect%3D%27non-scaling-stroke%27%20x%3D%27-32%27%20y%3D%27-32%27%20rx%3D%270%27%20ry%3D%270%27%20width%3D%2764%27%20height%3D%2764%27%20transform%3D%27translate%2832%2032%29%27%2F%3E%3Ccircle%20style%3D%27stroke%3A%23000%3Bstroke-width%3A0%3Bstroke-dasharray%3Anone%3Bstroke-linecap%3Abutt%3Bstroke-dashoffset%3A0%3Bstroke-linejoin%3Amiter%3Bstroke-miterlimit%3A4%3Bfill%3A%2370ff10%3Bfill-rule%3Anonzero%3Bopacity%3A1%27%20vector-effect%3D%27non-scaling-stroke%27%20r%3D%2735%27%20transform%3D%27matrix%28.37%200%200%20.37%2032%2032%29%27%2F%3E%3C%2Fsvg%3E";
```

Now build and test the contract again, to confirm there are no typos:

```text
./build.sh
cargo test -- --nocapture
```

Look for `Finished` after the build, and `test result: ok` after the tests.

# Designing our first NFT
 
For this example, let's create a new kind of collectable digital NFT artwork called a Flarn. Every NEP-171 Flarn will have a unique Token ID, an Owner ID to record ownership, and three pieces of metadata: a name, a description and a JPEG image.

## Define NFT Metadata

NEP-171 doesn't actually require NFTs to have any metadata at all; you can still mint and trade completely blank records of ownership, if that's what you're into.  But without metadata, NFTs would be pretty boring!  Here are the types of NFT metadata supported by the standard:

-   `title`: The name of this specific token.
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
-   `reference_hash`: Base64-encoded sha256 hash of JSON from reference field. Required if `reference` is included.

For our example we'll use `title`, `description` and `media`, and ignore the rest. Because we use `media` we are required to also use `media_hash`. We can leave `copies` blank to say our flarns are unique; `copies` defaults to `1` if not specified.  

NEP-171's built-in metadata fields are intended to cover the major needs of most NFT projects.  If some future NFT project requires other metadata fields not on this list, NEP-171 gives you two options.  You can either encode it as a JSON string in the `extra` field, or place it off-chain and store a link to it in the `reference` field.  (But if you use `reference`, you also need to store a hash of the data in the `reference_hash` field, to prevent tampering.)

## Create your masterpiece

You can either invent your own metadata at this point, or you can use this example data:

- `title`: Alice
- `description`: Alice is uniquely adorable & loves long walks on the beach.
- `media`: [![A Flarn called Alice](https://raw.githubusercontent.com/figment-networks/learn-tutorials/master/assets/nep171/flarn.jpg "Alice")](https://raw.githubusercontent.com/figment-networks/learn-tutorials/master/assets/nep171/flarn.jpg)

## Include off-chain data

Due to the high cost of Ethereum storage, most Ethereum NFTs of digital media do not store the media on the blockchain.  On-chain storage is much cheaper on NEAR, but since media files are large, it will still be thrifty of us to store our Flarn image elsewhere.  The `media` field of our on-chain record will hold a URL pointing to our off-chain image.  We could host that image anywhere on the Web we choose, but it would be nice to put it somewhere that's always online, decentralized, and free. 

## Upload the image to NFT Storage

For this example, we'll use the free  [NFT.Storage](https://nft.storage/#getting-started) service built specifically for storing off-chain NFT data. NFT.Storage offers free decentralized storage and bandwidth for NFTs on  [IPFS](https://ipfs.io/)  and  [Filecoin](https://filecoin.io/). 

**Steps**:

1. [Register an account](https://nft.storage/login/) and log in to [nft.storage](https://nft.storage/login/).
    
2. Go to the [Files](https://nft.storage/files/) section, and click on the [Upload](https://nft.storage/new-file/) button to upload your Flarn image.
    
![nft.storage](https://raw.githubusercontent.com/figment-networks/learn-tutorials/master/assets/nep171/uploading_to_nft_storage_1.png)
    
3. Once you have uploaded your file, select `Actions -> View URL` to see your upload in a new browser tab. Then copy the URL from that tab. That is the URL for your content, based on an IPFS Content-ID string. It should look something like this:
 
```text
https://ipfs.io/ipfs/bafkreic2y4z2hvfkzalogw3yeh5hntbvr4op4a5ccjo5zfhouss4mozlnm
```

![nft.storage](https://raw.githubusercontent.com/figment-networks/learn-tutorials/master/assets/nep171/uploading_to_nft_storage_2.png)
   
4. NEP-171 requires the `base_uri` field of the contract to point to a site that will have reliable access to the URLs in the `media` and `reference` fields. That's why we set `base_uri` to `https://nft.storage/` when we customized the contract.  Since our media is stored with NFT.Storage, we can use their URL for this field. 

## Generate the media_hash

The checksum of our media file stored in the `media_hash` field, must be a Base64-encoded SHA256 hash of the original data, to prove the file hasn't been tampered with.  NEP-171 requires Base64 encoding of the hash for maximum portability between platforms.  However, most online and command-line SHA256 tools generate hashes as Hexadecimal+Unicode, so we also will need to convert from that to Base64. 

There are a few different ways to do this.  On a UNIX-based system, use this command:

```text
cat flarn.jpg | shasum -a 256 | xxd -r -p | base64
```

On the web, you can generate the media hash in two steps.  
1: Upload your file to [this online SHA256 calculator](https://emn178.github.io/online-tools/sha256_checksum.html) to generate a SHA256 hash in hexadecimal format.
![Online SHA256 calculator](/https://raw.githubusercontent.com/figment-networks/learn-tutorials/master/assets/nep171/sha256.png)
2: Copy the hexadecimal string from that tool, and paste it into [this online Hex-Base64 converter](https://base64.guru/converter/encode/hex):
![Online Base64 converter](https://raw.githubusercontent.com/figment-networks/learn-tutorials/master/assets/nep171/hex_to_base64.png)

__NOTE__: do not upload sensitive or private data to these web services! Use the UNIX commands above if privacy is important.

If you used [our example Flarn image](https://raw.githubusercontent.com/figment-networks/learn-tutorials/master/assets/nep171/flarn.jpg), the strings should match the screenshots. If you're using your own image, the strings will be different.

# Assemble the pieces

Now that we've chosen our metadata, have stored our media stored off-chain, and have generated the media URL and media hash, we can construct our NFT Token record.  Use your editor to create a file in the current directory called `token.json`, containing a single object of JSON metadata. You can start by pasting in this text:

```json
{
	"title": "Alice",
	"description": "Alice is uniquely adorable & loves long walks on the beach.",
	"media": "https://bafyreiabag3ztnhe5pg7js4bj6sxuvkz3sdf76cjvcuqjoidvnfjz7vwrq.ipfs.dweb.link/",
	"media_hash": "e8MzVN0Pq1ztofypQoperJC8gWo2xfs8f9few+SwHes="
}
```

Now edit the following values:
- Use the URL you got from NFT.storage as the value for "media".
- Use the base64 sha256 hash you generated for "media_hash", if you're using a different image file than our example.
- Use your own NEAR testnet id for "receiver_id", so that you will own the newly minted NFT.
- Change `title` and `description` to whatever you like, or leave them as-is.

Save the file, then verify it at [JSONLint](https://jsonlint.com/).

![JSONLint](https://raw.githubusercontent.com/figment-networks/learn-tutorials/master/assets/nep171/jsonlint.png)

# Deploying and using the contract

We can use the NEAR CLI to deploy this contract, and to test that it's working. If you configured your environment in Tutorials 1 and 2, the CLI will connect to DataHub's high-availability testnet. If you don't have DataHub access you can still run the CLI with its defaults, but the default testnet node may be slower to respond. 

## Log in

Log in to your testnet account with `near-cli` by running the following command in your terminal.

```text
near login
```

This will open your testnet NEAR wallet in a web browser, so you can authorize the NEAR CLI with your testnet account. Once that's done, you should see something like this output in the terminal:

```text
Logged in as [ **accountname.testnet** ] with public key [ **ed25519:BDGh7Q...** ] successfully
```

To make this tutorial easier to copy/paste, we're going to set an environment variable with your testnet account ID. Run the command below, replacing  `accountname.testnet`  with the Account ID field output by `near login`

```text
export ID=accountname.testnet
```

Test that the environment variable is set correctly by running:

```text
echo $ID
```

Verify that the correct account ID is printed in the terminal. If everything looks correct you can now deploy your contract.

# Deploy the contract 

Run this command to deploy the contract in your testnet account:

```text
near deploy --wasmFile res/non_fungible_token.wasm --accountId $ID
```

If you have previously deployed a contract on this account, you will be asked if you want to replace it.  Answer `y` here:

```text
This account already has a deployed contract [G6TEwD4VXXYaUjgMmTw7y41R4x2DyDjcbXdX4xkESnXX]. Do you want to proceed? (y/n)
```

Example output:

```text
Starting deployment. Account id: accountname.testnet, node: https://rpc.testnet.near.org, helper: https://helper.testnet.near.org, file: res/non_fungible_token.wasm
Transaction Id F9p9s7DKNkekZYeBcSiYt2ZMzwpuVsugDDVGcDizxBNN
To see the transaction in the transaction explorer, please open this url in your browser
https://explorer.testnet.near.org/transactions/F9p9s7DKNkekZYeBcSiYt2ZMzwpuVsugDDVGcDizxBNN
Done deploying to accountname.testnet
```

The provided link will give you complete details about the deployment in the NEAR Explorer.

![NEAR explorer](https://raw.githubusercontent.com/figment-networks/learn-tutorials/master/assets/nep171/contract_deployed.png)

# Initialize the contract

A smart contract can define an initialization method that can be used to set the contract's initial state. This NFT example contract must be initialized before use. Run this command to initialize it with the default metadata and set the owner to your dev account:

```text
near call $ID new_default_meta '{"owner_id": "'$ID'"}' --accountId $ID
```

You can then view the contract metadata by running the following  `view`  call:

```text
near view $ID nft_metadata
```

Example response:

```text
View call: accountname.testnet.nft_metadata()
{
	spec: 'nft-1.0.0',
	name: 'CryptoFlarns',
	symbol: 'FLARN',
	icon: "data:image/svg+xml,%3Csvg xmlns='http://www.w3.org/2000/svg' viewBox='0 0 288 288'%3E%3Cg id='l' data-name='l'%3E%3Cpath d='M187.58,79.81l-30.1,44.69a3.2,3.2,0,0,0,4.75,4.2L191.86,103a1.2,1.2,0,0,1,2,.91v80.46a1.2,1.2,0,0,1-2.12.77L102.18,77.93A15.35,15.35,0,0,0,90.47,72.5H87.34A15.34,15.34,0,0,0,72,87.84V201.16A15.34,15.34,0,0,0,87.34,216.5h0a15.35,15.35,0,0,0,13.08-7.31l30.1-44.69a3.2,3.2,0,0,0-4.75-4.2L96.14,186a1.2,1.2,0,0,1-2-.91V104.61a1.2,1.2,0,0,1,2.12-.77l89.55,107.23a15.35,15.35,0,0,0,11.71,5.43h3.13A15.34,15.34,0,0,0,216,201.16V87.84A15.34,15.34,0,0,0,200.66,72.5h0A15.35,15.35,0,0,0,187.58,79.81Z'/%3E%3C/g%3E%3C/svg%3E",
	base_uri: null,
	reference: null,
	reference_hash: null
}
```

# Mint an NFT!

Now let's mint our first token! Run the following command to mint one copy of your NFT, using the metadata in `token.json`:

```text
near call $ID nft_mint '{"token_id": "0", "receiver_id": "'$ID'", "token_metadata": '"`cat token.json`}" --accountId $ID --deposit 0.1
```

Example response:

```text
Scheduling a call: accountname.testnet.nft_mint({"token_id": "2", "receiver_id": "accountname.testnet", "token_metadata": {
			"title": "Alice",
			"description": "Alice is uniquely adorable & loves long walks on the beach.",
			"media": "https://bafyreiabag3ztnhe5pg7js4bj6sxuvkz3sdf76cjvcuqjoidvnfjz7vwrq.ipfs.dweb.link/",
			"media_hash": "e8MzVN0Pq1ztofypQoperJC8gWo2xfs8f9few+SwHes="
}}) with attached 0.1 NEAR
Doing account.functionCall()
Transaction Id HQtMD9M4WnjJ4bJ8xz2A82pBMDNgMthkaegbNb4GsP7f
To see the transaction in the transaction explorer, please open this url in your browser
https://explorer.testnet.near.org/transactions/HQtMD9M4WnjJ4bJ8xz2A82pBMDNgMthkaegbNb4GsP7f
{
	token_id: '2',
	owner_id: 'accountname.testnet',
	metadata: {
		title: 'Alice',
		description: 'Alice is uniquely adorable & loves long walks on the beach.',
		media: 'https://bafyreiabag3ztnhe5pg7js4bj6sxuvkz3sdf76cjvcuqjoidvnfjz7vwrq.ipfs.dweb.link/',
		media_hash: 'e8MzVN0Pq1ztofypQoperJC8gWo2xfs8f9few+SwHes=',
		copies: null,
		issued_at: null,
		expires_at: null,
		starts_at: null,
		updated_at: null,
		extra: null,
		reference: null,
		reference_hash: null
	},
	approved_account_ids: {}
}
```

The contract has minted our NFT.  The return value of the contract call is the NFT record itself.  It's pretty much the same data we sent, shown as it's stored on the blockchain. 

- Notice that `receiver_id` became `owner_id`. 
- All the optional metadata fields we didn't specify are set to `null`.  
- The `approved_account_ids` field would list any other NEAR users who are authorized to manipulate this NFT, but we didn't authorize anybody, so that list is empty.

To view all the flarns you own, you can call the NFT contract with the following `near-cli` command:

```text
near view $ID nft_tokens_for_owner '{"account_id": "'$ID'"}'
```

Since this is your only flarn so far, the result will be a JSON array containing just the one NFT you've already seen: 

```text
[
  {
    token_id: '0',
	owner_id: 'accountname.testnet',
	metadata: {
		title: 'Alice',
		description: 'Alice is uniquely adorable & loves long walks on the beach.',
		media: 'https://bafyreiabag3ztnhe5pg7js4bj6sxuvkz3sdf76cjvcuqjoidvnfjz7vwrq.ipfs.dweb.link/',
		media_hash: 'e8MzVN0Pq1ztofypQoperJC8gWo2xfs8f9few+SwHes=',
		copies: null,
		issued_at: null,
		expires_at: null,
		starts_at: null,
		updated_at: null,
		extra: null,
		reference: null,
		reference_hash: null
	},
	approved_account_ids: {}
  }
]
```

Also, the NEAR Wallet will automatically check the entire network for NFTs you own, so you can see your new Flarn in your Wallet, under the Collectables tab:
[Flarn NFT in wallet](https://raw.githubusercontent.com/figment-networks/learn-tutorials/master/assets/nep171/flarn_in_wallet.png)

# Conclusion

You now have deployed an NFT-171 smart contract on the NEAR testnet, and have minted one CryptoFlarn NFT. From here you could use the CLI or the NEAR Javascript SDK to transfer ownership of that token, or to mint more tokens. If your next NFT project needs more complex metadata, you've seen how that can be added.  

When you're ready to go public with your new NFT contract, you can easily [point NEAR CLI at mainnet](https://docs.near.org/docs/tools/near-cli#network-selection), and deploy using the same steps.

The complete code for this tutorial can be found on [Github](https://github.com/figment-networks/tutorials/tree/main/near/6_NFT).

# Next Steps

From here you can implement a web interface to show off your NFTs, using [near-api-js](https://docs.near.org/docs/api/javascript-library) to fetch the metadata of the NFTs in your contract.  Then you can explore other optional substandards of NEP-171, such as the [Approval Management](https://nomicon.io/Standards/NonFungibleToken/ApprovalManagement) standard (NEP-178) that lets you authorize marketplaces to trade your NFT on your behalf, or the [Royalties and Payouts](https://nomicon.io/Standards/NonFungibleToken/Payout) standard (NEP-199) that lets you specify how the proceeds of an NFT sale should be distributed.

# About the Author

This tutorial was created by [Mykle Hansen](https://github.com/myklemykle), a contributor to the [Plantary](https://github.com/myklemykle/plantary) project which allows users to grow and harvest plant NFTs on NEAR.
