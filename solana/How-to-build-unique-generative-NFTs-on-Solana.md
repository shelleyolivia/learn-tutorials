# Introduction

In this tutorial, you will learn to create a collection of generative NFTs and a minting machine on the Solana blockchain.

First, you'll generate your NFT collection using an algorithm and then you'll upload those to arweave using metaplex and finally, you'll create a marketplace for users to mint your NFTs. You can refer to [pixelape.art](https://pixelape.art/#/) as an example of the final result.

# Prerequisites

Basic familiarity with JavaScript and JSON.

# Requirements

GitHub account

A code editor, download VS Code from [here](https://code.visualstudio.com/Download)

Git, install from [here](https://git-scm.com/downloads)

Solana CLI, install from [here](https://docs.solana.com/cli/install-solana-cli-tools#use-solanas-install-tool)

Node.js (v14.18.1+), install it from [here](https://nodejs.org/en/download/)

ts-node, install it from [here](https://www.npmjs.com/package/ts-node#installation)

Yarn, install it from [here](https://classic.yarnpkg.com/en/docs/install)

phantom, install it from [here](https://phantom.app/)

# Preparing tools

To successfully launch your Collection on Solana you'll need to go through three steps:
1- generating your collection of unique NFTs using hashlips art engine
2- uploading the collection to Arweave using metaplex
3- creating a store for our collection using candy machine

You will find more extensive explanations about tools later in the tutorial where you will be learning to use them. First, we create an empty folder on our desktop. Then we open GitHub repositories of these tools and clone those in our folder. (links are provided in the references section)
For example for hashlips art engine, first, we open the link in our browser:

![hashlips github.jpg](https://github.com/figment-networks/learn-tutorials/raw/master/assets/hashlips_github.jpg)

Then first we click on the green Code button and from the new menu we click on Download ZIP:

![download zip in github.jpg](https://github.com/figment-networks/learn-tutorials/raw/master/assets/download_zip_in_github.jpg)

After the download is completed we simply extract the folder inside the zip file to the folder we created. We simply repeat this process for "metaplex" and "candy machine" as well and the final result should be like this:

![folders in man folder.jpg](https://github.com/figment-networks/learn-tutorials/raw/master/assets/folders_in_man_folder.jpg)

# Setting up Solana-Cli

Make sure Solana-CLI is installed by running the `Solana --version` in a command line. If it shows the version, then CLI is installed, otherwise, you may need to follow the installation process provided up in the requirements section.

```
$ solana --version
solana-cli <version installed in your machine>
```

If you want to see the current config being used in CLI, you can always run command `solana config get` in a terminal and the result would be like:

```
$ solana config get
Config File: /home/<your username>/.config/solana/cli/config.yml
RPC URL: https://api.mainnet-beta.solana.com
WebSocket URL: wss://api.mainnet-beta.solana.com/ (computed)
Keypair Path: /home/<your username>/.config/solana/id.json
Commitment: confirmed
```

<aside>
💡 The word **<your username>** would be your username on your local Linux machine. In all following outputs and commands, you should be seeing or using your own username.

If you don't know your username run command `whoami` to see it

</aside>

To change the default network to **devnet** first run command `solana config set --url devnet` :

```
$ solana config set --url devnet
Config File: /home/<your username>/.config/solana/cli/config.yml
RPC URL: https://api.devnet.solana.com
WebSocket URL: wss://api.devnet.solana.com/ (computed)
Keypair Path: /home/<your username>/.config/solana/id.json
Commitment: confirmed
```

Then create a wallet by running the command below:

 `solana-keygen new --outfile ~/.config/solana/devnet.json`

Terminal prompts message bellow:

```
$ solana-keygen new --outfile ~/.config/solana/devnet.json
Generating a new keypair

For added security, enter a BIP39 passphrase

NOTE! This passphrase improves security of the recovery seed phrase NOT the
keypair file itself, which is stored as insecure plain text

BIP39 Passphrase (empty for none):
```

You can enter a password but make sure you save your entered passphrase in a safe place because if you forget it you wouldn't be able to restore your wallet.

```

Wrote new keypair to /home/mmostafavi/.config/solana/temp.json
===================================================================
pubkey: dgFgNJo6ietb5Vm7XKbjuPj3LtxRMCuHgUM39zL6xDt
===================================================================
Save this seed phrase and your BIP39 passphrase to recover your new keypair:
<here will be your 12 word seed phrase you can recover your wallet with>
===================================================================
```

<aside>
🚨 you shouldn't share your seed phrase under any circumstances.

</aside>

Then set this new wallet as the default wallet in the Solana CLI:

`solana config set --keypair /home/mmostafavi/.config/solana/devnet.json`

```
$ solana config set --keypair /home/mmostafavi/.config/solana/devnet.json
Config File: /home/<your username>/.config/solana/cli/config.yml
RPC URL: https://api.devnet.solana.com
WebSocket URL: wss://api.devnet.solana.com/ (computed)
Keypair Path: /home/<your username>/.config/solana/devnet.json
Commitment: confirmed
```

And finally, airdrop some SOL to your new wallet: `solana airdrop 5`

```
$ solana airdrop 5
Requesting airdrop of 5 SOL

Signature: <transaction signature>

5 SOL
```

And to check whether it was successful: `solana balance`

```
$ solana balance
5 SOL
```

## Adding the new wallet to Phantom

Later in the tutorial, you will connect phantom to your NFT minting dApp to mint some NFTs. so as the last step let's add your wallet to phantom. for that first we fetch the path to our wallet by running:

```
$ solana config get
Config File: /home/<your username>/.config/solana/cli/config.yml
RPC URL: https://api.devnet.solana.com
WebSocket URL: wss://api.devnet.solana.com/ (computed)
Keypair Path: /home/<your username>/.config/solana/devnet.json
Commitment: confirmed
```

Then copy the path in Keypair Path: "/home/<your username>/.config/solana/devnet.json"

next run command cat /home/<your username>/.config/solana/devnet.json to print content of the devnet.json file in terminal:

```
$ cat /home/<your username>/.config/solana/devnet.json
<it will print your private key in array format>
```

Now you need to head over to the phantom extension in the browser. open it. from hamburger menu at left click at "Add / Connect Wallet":

![import private key in phantom screen shot.jpg](https://github.com/figment-networks/learn-tutorials/raw/master/assets/import_private_key_in_phantom_screen_shot.jpg)

Then choose a name and copy-paste the private key:

![linking NFTs wallet in phantom screenshot.jpg](https://github.com/figment-networks/learn-tutorials/raw/master/assets/linking_NFTs_wallet_in_phantom_screenshot.jpg)

Since you will be working with **devnet**, you need to set the network to **devnet** in phantom's settings. For that click on ⚙️ then "Change Network" then "Devnet":

![changing the network from mainnet to devnet screenshot.jpg](https://github.com/figment-networks/learn-tutorials/raw/master/assets/changing_the_network_from_mainnet_to_devnet_screenshot.jpg)

# Generating NFTs

Layers play an important role in the generation of unique NFTs. Those are like traits and characteristics and a set of these layers together creates a unique NFT. The Hash lips art engine makes sure none of the two NFTs in the collection have the same layers and this way those are guaranteed to be unique.

Hash lips art engine is an open-source engine for creating unique NFT collections using layers as their building blocks. If we provide unique classified layers for our collection, NFTs are guaranteed to be unique by the engine.

In this tutorial, we will create a very simple set of layers to give you an idea about them but this is the part that makes each Collection distinct from others. So you may want to spend more time on this step and provide much more details in your layers.

## Creating the layer

You can use any tool like photoshop or something similar for creating your layers but make sure the layers which are going to sit on top of other layers have a transparent background. Here are some simple layers I created using the paint app!

backgrounds:

![background1.png](https://github.com/figment-networks/learn-tutorials/raw/master/assets/background1.png)

![background2.png](https://github.com/figment-networks/learn-tutorials/raw/master/assets/background2.png)

circles:

![circle1.png](https://github.com/figment-networks/learn-tutorials/raw/master/assets/circle1.png)

![circle2.png](https://github.com/figment-networks/learn-tutorials/raw/master/assets/circle2.png)

![circle3.png](https://github.com/figment-networks/learn-tutorials/raw/master/assets/circle3.png)

and triangles:

![triangle1.png](https://github.com/figment-networks/learn-tutorials/raw/master/assets/triangle1.png)

![triangle2.png](https://github.com/figment-networks/learn-tutorials/raw/master/assets/triangle2.png)

![triangle3.png](https://github.com/figment-networks/learn-tutorials/raw/master/assets/triangle3.png)

So now I have 3 Layers and overall 8 traits. you can download these layers from the link in references.

## Putting layers in layers folder

In hashlips engine, we can specify the rarity of a trait within a group of traits. To do that we add a `#` and a percentage following that to the end of the assets' file name. for example if we wanted **background2.png** to appear 80% of the time we would rename it to **background2#80.png** and the other background would appear in the other 20% of the time by renaming it **background1#20.png**.

So for each Layer (aka background, triangle, or circle), we specify the rarity of traits. the sum of rarity numbers should be 100. here is an example:

- Background
    
    background1#50.png
    
    background2#50.png
    
- Triangle
    
    triangle1#30.png
    
    triangle2#40.png
    
    triangle3#30.png
    
- Circle
    
    circle1#30.png
    
    circle2#40.png
    
    circle3#30.png
    

The next step is putting our layers in the **layers** folder. First, delete all initial folders and layers inside the layers folder. Then create three folders and name them after our layers: Background, Circle, and Triangle (first letters are uppercase). Then copy and paste traits to their relative Folder and specify the rarity for them.

## Setting up the Engine

We need to set up metadata related to our collection like the owner of the collection or the name of the collection. To do that we need to open the `config.js` file inside the **src** folder. Let's go through the code inside this file step by step and configure our NFT collection.

This part of the code imports some internal variables and modules and you don't need to touch it.

```jsx
const basePath = process.cwd();
const { MODE } = require(`${basePath}/constants/blend_mode.js`);
const { NETWORK } = require(`${basePath}/constants/network.js`);
```

**network**: You can change the network you are going to deploy your collection to, by changing **`eth`** to your target network. which in our case it is **`sol`**(sol here stands for Solana)

```jsx
const network = NETWORK.sol;
```

Here you can specify the name and a description for your collection. ignore the comment, it's not specifically for Ethereum. And leave the baseUri as it is.

```jsx
// General metadata for Ethereum
const namePrefix = "Triangle and Circle";
const description = "Unique combinatons of a Circle and a Triangle with two backgrounds";
const baseUri = "ipfs://NewUriToReplace";
```

Here we set up metadata specific to Solana NFTs. 

**symbol**: In symbol, you can set an abbreviation for the name of your collection. I chose the first letters of the words in the name.

**seller_fee_basis_points**:  this parameter specifies the share which creators want to earn from future sales of their NFTs. Each point is 0.01% so if a creator wants to earn 1% from future trades of their collection they should set it to100 points. This parameter is known as royalty as well.

**external_url**: if the collection has a website or a YouTube channel or a link related, you can put it here

**creators**: In this field, you can specify shares of profit between creators. If there is just a single creator then you can put the public key of them in the **address** field and set the **share** field to 100 (standing for 100%).

If there is more than one creator for the collection you can add a similar object in the **creators** array for all other creators and set their shares and their Solana addresses

for this collection, we have one creator so we set shares to 100 and put our Solana address for address.

<aside>
💡 as a reminder, you can see your Solana address by running the command `solana address` in terminal

</aside>

```jsx
const solanaMetadata = {
  symbol: "TAC",
  seller_fee_basis_points: 100, // Define how much % you want from secondary market sales 1000 = 10%
  external_url: "",
  creators: [
    {
      address: "<your solana address>",
      share: 100,
    },
		/* you can add new creators like this
		{
      address: "creator's solana address",
      share: creator's share out of 100,
    },
		*/
  ],
};
```

**growEditionSizeTo**: In this part of the code we specify the number of NFTs we want to generate. An important factor to consider here is the maximum unique NFTs we can expect to yield from the layers we have provided. we have 3 circles and 3 triangles and 2 backgrounds then:

maximum number of unique NFTs =  3*3*2 = 18.

You can put any number equal to or less than 18.

**layersOrder**: In this field, we specify the order we want the layers to sit on each other. Since the background is the bottom-most layer we put it first and the other two layers after that.

```jsx
// If you have selected Solana then the collection starts from 0 automatically
const layerConfigurations = [
  {
    growEditionSizeTo: 5,
    layersOrder: [
      { name: "Background" },
      { name: "Circle" },
      { name: "Triangle" },
    ],
  },
];
```

we are done with configuring our collection but for the advanced configuration, you can follow the next chapter.

## Advanced Configuration (Optional)

**shuffleLayersConfigurations**: this field is for when we have more than one layer Configuration. here is an example:

```jsx
const layerConfigurations = [
	// we want 5 images with this configuration
  {
    growEditionSizeTo: 5,
    layersOrder: [
      { name: "Background" },
      { name: "Circle" },
      { name: "Triangle" },
    ],
  },

// and we want 10 images with this configuration
{
    growEditionSizeTo: 10,
    layersOrder: [
      { name: "Circle" },
      { name: "Triangle" },
			{ name: "Background" },
    ],
  },
];
```

In the example above we have two configurations. if we set the field to the **true** engine wouldn’t follow the order of our configurations. for example image number 1 can be created from any of these configurations. It is false by default and will save all images in numerical order. (first 5 images using the first config and then 10 images using the second config).

**debugLogs**:  If you want to have logs to debug and see what is happening when you generate images you can set the variable `debugLogs` ****to true. It is false by default, so you will only see general logs.

```jsx
const shuffleLayerConfigurations = false;

const debugLogs = false;
```

In this part you can specify output image width and height sizes in pixel

```jsx
const format = {
  width: 512,
  height: 512,
  smoothing: false,
};
```

If you want gif outputs you can set `export` to true. repeat is the number of cycles in each gif. 

`repeat: -1` will produce a one-time render and `reapeat: 0` will loop forever.

You can specify the quality you want for your gifs. A higher-quality means larger files in size. You can specify the delay in **ms** before the animation starts in the gif.

```jsx
const gif = {
  export: false,
  repeat: 0,
  quality: 100,
  delay: 500,
};
```

If you want to change the ratio of the pixilation then you can update the `ratio` property. The lower the number on the left, the more pixelated the image will be.

```jsx
const pixelFormat = {
  ratio: 2 / 128,
};
```

If you want to add any extra metadata to your collection you can put it inside `extraMetadata` object. For example:

```jsx
const extraMetadata = {
	creator: "<Put creator's name>"
};
```

## Generating images

First, you need to install dependencies for this engine. Open a terminal and make sure you’re at **hashlips_art_engine**  directory and run `yarn install`.

To generate images run `yarn run generate`:

```jsx
hashlips_art_engine-main$ yarn run generate
yarn run v1.22.17
warning ../../../../../../package.json: No license field
$ node index.js
Created edition: 0, with DNA: 327303194f43b8111321be04af9aefa991021daf
Created edition: 1, with DNA: 830a7d0d732f8b676ff6349659409b5efc171513
Created edition: 2, with DNA: 2aed7015b3023e509c46dd1fb870cf40d3c8ea59
Created edition: 3, with DNA: 63c561781f810ddbecce9ca086b1c7b2dcc1a589
Created edition: 4, with DNA: 35742657abda4d22190a0761df5d0426da72ece5
```

Awesome! now you have your images in the build>images folder. and their metadata inside the build>json folder. check them out. here are my 5 generated images altogether. 

![preview.png](https://github.com/figment-networks/learn-tutorials/raw/master/assets/preview.png)

In the next section, we will upload our assets to Arweave and we will create a candy machine for minting our NFTs.

# Uploading to the Arweave and creating a candy machine

We will use candy-machine-cli to upload our images to the Arwaeve and create a minting machine for our NFTs.

## What is Candy machine CLI?

Candy Machine is a tool built by Metaplex and Solana for uploading NFT assets to arweave and building a minting machine program on Solana For NFTs.

In this tutorial, we are using it to upload the assets and create a minting machine for our NFTs.

## What is Arweave?

If you open Arweave’s website you find this definition: 

> Arweave is a new type of storage that backs data with sustainable and perpetual endowments, allowing users and developers to truly store data forever – for the very first time.

As a collectively owned hard drive that never forgets, Arweave allows us to remember and preserve valuable information, apps, and history indefinitely. By preserving history, it prevents others from rewriting it.
> 

In simple words, Arweave enables you to store documents and applications permanently. Candy machine CLI uploads our assets to the arweave by default.

## Preparing Assets for upload

First, we need to create an “**assets**” folder inside **metaplex-master** repository we downloaded before. Copy all the images from the build>images folder inside hashlips_art_engine folder and paste them to the assets folder. do the same thing with build>json folder inside hashlips_art_engine except for for**`_metadata.json`** file. now in your assets folder, you should have the structure below:

- assets
    - 0.json
    - 0.png
    - 1.json
    - 1.png
    - 2.json
    - 2.png
    - 3.json
    - 3.png
    - 4.json
    - 4.png

## Installing candy-machine-cli

 Before using candy-machine-cli we need to install its dependencies and build an executable version of that. For that first open a terminal in `metaplex-master/js` directory. To install dependencies run `yarn install`. It may take a while.

to build an executable version of candy-machine-cli run `yarn run build` and wait for that to finish as well. and finally, run `yarn run bootstrap`. You are done with installing candy-machine-cli and can return to the main directory by running `cd ..`

## Uploading to Arweave

In the terminal make sure you’re in metaplex-master directory. To upload the assets to arweave we need to run the command below:

`ts-node js/packages/cli/src/candy-machine-cli.ts upload ./assets --env devnet --keypair /home/<your username>/.config/solana/devnet.json`

- `ts-node js/packages/cli/src/candy-machine-cli.ts` compiles the executable and runs it
- `upload` is the function we want to use from executable
- `./assets` is the directory our images and their metadata exist
- `--env devnet` specifies which network we are targeting. If you wanted to deploy your collection on mainnet-beta then replace it with `--env mainnet-beta`
- `--keypair /home/<your username>/.config/solana/devnet.json` specifies the path to keypair which will pay for the fees of uploading. here we are using the keypair we generated at beginning of the tutorial.

```
metaplex-master$ ts-node js/packages/cli/src/candy-machine-cli.ts upload ./assets --env devnet --keypair /home/<your username>/.config/solana/devnet.json
Beginning the upload for 5 (png+json) pairs
started at: 1639748729410
wallet public key: <your wallet public key will be printed here>
Processing file: 0, 0.png
initializing config
initialized config for a candy machine with publickey: <another publick key will be allocated for your candy machine here>
Processing file: 1, 1.png
Processing file: 2, 2.png
Processing file: 3, 3.png
Processing file: 4, 4.png
Writing indices 0-4
Done. Successful = true.
ended at: 2021-12-17T13:46:37.408Z. time taken: 00:01:07
```

## Creating a candy machine

The next step is creating a candy machine. for that you need to run the command below:

`ts-node js/packages/cli/src/candy-machine-cli.ts create_candy_machine --env devnet --keypair /home/<your username>/.config/solana/devnet.json -p <price for minting one of your NFTs>`

- `ts-node js/packages/cli/src/candy-machine-cli.ts` compiles the executable and runs it
- `create_candy_machine` is the function we need to call for creating the candy machine
- `--env devnet` specifies which network we are targeting. If you wanted to deploy your collection on mainnet-beta then replace it with `--env mainnet-beta`
- `--keypair /home/<your username>/.config/solana/devnet.json` specifies the path to keypair which will pay for the fees of uploading. here we are using the keypair we generated at beginning of the tutorial.
- `-p <price for minting one of your NFTs>` specifies the price in $SOL for minting each NFT

```
metaplex-master$ ts-node js/packages/cli/src/candy-machine-cli.ts create_candy_machine --env devnet --keypair /home/<your username>/.config/solana/devnet.json -p 0.1
wallet public key: <your public key will be printed here>
create_candy_machine finished. candy machine pubkey: <public key of your candy machine will be printed here>
```

## Updating the candy machine

Your candy machine is created but before users are able to mint your NFTs you need to tell the candy machine the date for launching your collection. Only after that date, users can mint your NFTs. You need to use `update_candy_machine` function to update the date field. you can update the price of your NFTs in this command too.

`ts-node js/packages/cli/src/candy-machine-cli.ts update_candy_machine --env devnet --keypair /home/<your username>/.config/solana/devnet.json -p <set your price> --date <set the launch date>`

- `ts-node js/packages/cli/src/candy-machine-cli.ts` compiles the executable and runs it
- `update_candy_machine` is the function we need for updating our candy machine
- `--env devnet` specifies which network we are targeting. If you wanted to deploy your collection on mainnet-beta then replace it with `--env mainnet-beta`
- `--keypair /home/<your username>/.config/solana/devnet.json` specifies the path to keypair which will pay for the fees of uploading. here we are using the keypair we generated at beginning of the tutorial.
- `-p <set your price>` here you can update price of your NFTs if you want or put the previous price
- `--date <set the launch date>` here you can set your launch date. for example: 
`--date "1 Dec 2021 00:00:00 GMT"`

```
metaplex-master$ ts-node js/packages/cli/src/candy-machine-cli.ts update_candy_machine --env devnet --keypair /home/<your username>/.config/solana/devnet.json -p 0.1 --date "1 Dec 2021 00:00:00 GMT"wallet public key: 3PB5heBX81zkb5aFzDh4R3Z5d7oF3FjMjUXtbP3w6VWa
 - updated startDate timestamp: 1638316800 (1 Dec 2021 00:00:00 GMT)
 - updated price: 100000000 lamports (0.1 SOL)
update_candy_machine finished <transaction hash will be printend here>
```

Now, your candy machine is created and you can find all of the information and details about it in a file created in the **.cache** folder. Depending on the network you have chosen for creating your candy machine name of the file has the structure below:

 **<name of the network you’ve deployed your collection to>-temp**

for instance, if you have deployed on devnet, it would be **devnet-temp**. 

```
{
  "program": {
    "uuid": <your uuid>,
    "config": <public key for configuring the candy machine dApp>,
  },
  "items": {
    /*
			here you find information about each NFT item inside your collection
			with this structure:

			// for NFT number "i" in the NFT collection
			"i" : {
				"link": <a link to the asset on the arweave>,
	      "imageLink": <a link to image of the NFT on arweave>,
	      "name": < name of the NFT >,
	      "onChain": < true or false >
			}, 
		
		*/
  },
  "env": <network the collection is on>,
  "cacheName": <suffix has been used in filename>,
  "authority": <owner of the candy machine, which will be your public key>,
  "candyMachineAddress": <the public key of your minting machine>,
	"startDate": <the timestamp for the launch date>
}
```

You will need some of this information later in configuring the candy machine dApp.

Awesome! Now we have our candy machine all set up and updated. let's build a Frontend connected to our candy machine.

# Building a Frontend for our candy machine

To create the frontend first head back again to three folders you cloned and this time open the third folder "candy-machine-mint-main". This is a react project created for the candy machine and gives us a basic UI and minting functionality and we can customize the UI.

First, you need to install all the dependencies of this project. run `yarn install`. before you start the frontend, you need to configure the frontend. We have a .env.example file with the content below:

```
REACT_APP_CANDY_MACHINE_CONFIG=__PLACEHOLDER__
REACT_APP_CANDY_MACHINE_ID=__PLACEHOLDER__
REACT_APP_TREASURY_ADDRESS=__PLACEHOLDER__
REACT_APP_CANDY_START_DATE=__PLACEHOLDER__

REACT_APP_SOLANA_NETWORK=devnet
REACT_APP_SOLANA_RPC_HOST=https://explorer-api.devnet.solana.com
```

Let’s configure it step by step. To configure this file we need to use **devnet-temp** file.

```
// this takes config field
REACT_APP_CANDY_MACHINE_CONFIG= <put the config public key here>

// this takes candyMachineAddress
REACT_APP_CANDY_MACHINE_ID= <put the candyMachineAddress here>

// this takes authority
REACT_APP_TREASURY_ADDRESS= <put the authority here>

// this takes startDate
REACT_APP_CANDY_START_DATE= <put the startDate here>

// we are using devent so we don't need to change this variable
// but if you wanted to connect to a candy machine on mainnet-beta
// you should change devnet to mainnet-beta
REACT_APP_SOLANA_NETWORK=devnet

// we are using devent so we don't need to change this variable
// but if you wanted to connect to a candy machine on mainnet-beta
// you should change devnet to mainnet-beta
REACT_APP_SOLANA_RPC_HOST=https://explorer-api.devnet.solana.com
```

To let your react app use these environment variables you should rename the **.env.example** file to **.env** and then run `yarn start` to run our dApp locally on localhost:3000

![initial state of dapp candy machine screenshot.jpg](https://github.com/figment-networks/learn-tutorials/raw/master/assets/initial_state_of_dapp_candy_machine_screenshot.jpg)

It’s a simple ugly looking UI with just a “Connect Wallet” button. Ok let’s connect our wallet: 

![cansy store up and running wallet connected.jpg](https://github.com/figment-networks/learn-tutorials/raw/master/assets/cansy_store_up_and_running_wallet_connected.jpg)

You may ask why we launched it on devnet and not on mainnet-beta?! for two reasons:

1. You can estimate the costs of launching on mainnet-beta
2. You can make sure there isn't any spooky bug stopping you in the middle of the process

From the UI aspect our app isn’t that good. So let’s add some styling to our app in the next section.

# Styling the frontend

Replace the code below with code in **index.css**. It defines some CSS classes which we will utilize in Home.tsx component.

```css
body {
  margin: 0;
  font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', 'Roboto', 'Oxygen',
    'Ubuntu', 'Cantarell', 'Fira Sans', 'Droid Sans', 'Helvetica Neue',
    sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  background-color: #303030;
  color: #FFFFFF;
  background: rgb(2,0,36);
  background: linear-gradient(90deg, rgba(2,0,36,1) 0%, rgba(9,9,121,1) 35%, rgba(0,212,255,1) 100%);
}

code {
  font-family: source-code-pro, Menlo, Monaco, Consolas, 'Courier New',
    monospace;
}

.Welcome-header {
  text-align: center;
  font-size: 80px;
  font-weight: bolder;
  
}

.Welcome-description {
  text-align: center;
  align-self: stretch;
  font-size: 40px;
  font-weight: bold;
}

.Connected-wallet {
  /* display: block; */
  position: absolute;
  top: 1rem;
  right: 1rem;
  padding: 1rem;
  background-color: salmon;
  color: rgb(0, 0, 0);
  width: 15rem;
  border-radius: 25px;
}

.Collection-description {
  padding: 1rem;
  margin: auto;
  margin-top: 15rem;
  font-size: xx-large;
  width: 50%;
}

.Redeemed-by-available {
  text-align: center;
  padding: 1rem;
  font-size: x-large;
}
```

Head over to src/Home.tsx and find the variable **MintContainer** and add styles below :

```jsx
const MintContainer = styled.div`
  margin: auto;
  width: 25rem;
  display: flex;
  flex-direction: column;
  align-items: center;
`
```

Replace return part of Home.tsx component with code below. New code removes redundant information like remaining NFTs and Balance. it creates a nice-looking component for a connected wallet at the top-right of the screen. It adds some explanations and details about the NFT collection as well.

```jsx
return (
    <main>
      {wallet && (
        <>
          <div className="Connected-wallet">
            Connected Wallet: {shortenAddress(wallet.publicKey.toBase58() || "")}
          </div>

          <div className="Collection-description">
            This collection is consisted of 5 unique combinations of a circle and a triangle
            with a background either red or white.
          </div>

          <div className="Redeemed-by-available">
          {itemsRedeemed} / {itemsAvailable}
          </div>
        </>
        
      )}

      <MintContainer>
        {!wallet ? (
          <>
            <p className="Welcome-header">Welcome!</p>
            <p className="Welcome-description">
              Our NFTs would be out of stock soon!
              Hurry up and mint yours!
            </p>
            <ConnectButton>Connect Wallet</ConnectButton>
          </>
        ) : (
          <MintButton
            disabled={isSoldOut || isMinting || !isActive}
            onClick={onMint}
            variant="contained"
          >
            {isSoldOut ? (
              "SOLD OUT"
            ) : isActive ? (
              isMinting ? (
                <CircularProgress />
              ) : (
                "MINT"
              )
            ) : (
              <Countdown
                date={startDate}
                onMount={({ completed }) => completed && setIsActive(true)}
                onComplete={() => setIsActive(true)}
                renderer={renderCounter}
              />
            )}
          </MintButton>
        )}
      </MintContainer>

      <Snackbar
        open={alertState.open}
        autoHideDuration={6000}
        onClose={() => setAlertState({ ...alertState, open: false })}
      >
        <Alert
          onClose={() => setAlertState({ ...alertState, open: false })}
          severity={alertState.severity}
        >
          {alertState.message}
        </Alert>
      </Snackbar>
    </main>
  );
```

Let’s mint an NFT in our nice-looking app!

![NFT minting gif.gif](https://github.com/figment-networks/learn-tutorials/raw/master/assets/NFT_minting_gif.gif)

YaY, we did it! Let’s check out our newly minted NFT in our wallet:

![Checking NFT in wallet gif.gif](https://github.com/figment-networks/learn-tutorials/raw/master/assets/Checking_NFT_in_wallet_gif.gif)

# Launching to Mainnet-beta

You successfully launched your collection on devnet and now you are ready to deploy to mainnet-beta. There isn't any significant difference between deploying to devnet and mainnet-beta. you need to switch back to mainnet-beta by running `solana config set --url mainnet-beta`. this time we need to top our wallet up with real SOL. You can easily purchase some from an exchange like [FTX](https://ftx.com/).

generating the assets wouldn’t be any different from devnet approach since it’s agnostic about the Solana network you are using.

In the tutorial where you are creating a candy machine and uploading the assets, you need to change `--env devnet` to  `--env mainnet-beta` in all three commands.

so new commands would be:

1- `ts-node js/packages/cli/src/candy-machine-cli.ts upload ./assets --env mainnet-beta --keypair /home/<your username>/.config/solana/devnet.json`

2- `ts-node js/packages/cli/src/candy-machine-cli.ts create_candy_machine --env mainnet-beta --keypair /home/<your username>/.config/solana/devnet.json -p 0.1`

3- `ts-node js/packages/cli/src/candy-machine-cli.ts update_candy_machine --env mainnet-beta --keypair /home/<your username/.config/solana/devnet.json -p 0.1 --date "17 Dec 2021 00:00:00 GMT"`

Easy peasy! 

In the tutorial where you are configuring your dApp you will need to do some adjustments. This time inside the **`.cache`** folder we would have **`mainnet-beta-temp`** instead of **`devnet-temp`** but the structure of the two files is the same. Because this time we are launching on **mainnet-beta**, we need to set the last two .env variables like below:

```
// same as devnet approach you should put
// appropriate values from mainnet-beta-temp
// in variables below

REACT_APP_CANDY_MACHINE_CONFIG=__PLACEHOLDER__
REACT_APP_CANDY_MACHINE_ID=__PLACEHOLDER__
REACT_APP_TREASURY_ADDRESS=__PLACEHOLDER__
REACT_APP_CANDY_START_DATE=__PLACEHOLDER__

// change "devnet" in two variables bellow to "mainnet-beta"
REACT_APP_SOLANA_NETWORK=mainnet-beta
REACT_APP_SOLANA_RPC_HOST=https://explorer-api.mainnet-beta.solana.com
```

That's All of the adjustments you need to make to launch your minting dApp on **mainnet-beta**.

# Conclusion

In this tutorial, we learned how to generate unique NFTs and then create a minting dApp for them.

# About the Author

I'm Mahdi Mostafavi a web developer and I'm learning about Solana. my GitHub: [@mmostafavi](https://github.com/mmostafavi)

If you found any misinformation in the article or had any suggestions to improve the tutorial you can message me on Twitter: [@mahdi_ftp](http://twitter.com/mahdi_ftp).

# References

hashlips art engine: [https://github.com/HashLips/hashlips_art_engine](https://github.com/HashLips/hashlips_art_engine)

metaplex: [https://github.com/metaplex-foundation/metaplex](https://github.com/metaplex-foundation/metaplex)

candy machine: [https://github.com/exiled-apes/candy-machine-mint](https://github.com/exiled-apes/candy-machine-mint)‣

NFT collection assets: [https://bit.ly/3EYmc5X](https://bit.ly/3EYmc5X)
