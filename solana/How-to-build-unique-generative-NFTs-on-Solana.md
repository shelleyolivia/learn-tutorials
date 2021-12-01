# Introduction

In this tutorial, we will be creating a collection of generative NFTs and a minting machine on the Solana blockchain.

First, We generate our NFT collection using an algorithm and then upload those to blockchain and finally create a marketplace for users to mint our NFTs and own them. You can refer to [pixelape.art](https://pixelape.art/#/) as an example of what we will be building.

We will use three tools:
**HashLips art engine**: this tool is an open-source program to create unique generative NFTs.
**Metaplex**: this tool helps us to launch our collection on Solana.
**Candy machine**: this tool helps us to create a frontend for our candy mint machine

# Prerequisites

Basic familiarity with javascript and json.

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

To successfully launch our Collection on Solana we need to go through three steps:
1- generating our collection of unique NFTs using hashlips art engine
2- uploading the collection on Solana using metaplex
3- creating a store for our collection using a candy machine

First, we create an empty folder in our desktop named "solana_generative_nfts". Then we open GitHub repositories of these tools and clone those in our folder. (links are provided in the references section)
For example for hashlips art engine first, we open the link in our browser:

![hashlips github.jpg](https://github.com/figment-networks/learn-tutorials/raw/master/assets/hashlips_github.jpg)


Then first we click on the green Code button and from the new menu we click on Download ZIP:

![download zip in github.jpg](https://github.com/figment-networks/learn-tutorials/raw/master/assets/download_zip_in_github.jpg)

After the download is completed we simply extract the folder inside the zip file in our "solana_generative_nfts" folder. We simply repeat this process for "metaplex" and "candy machine" as well and the final result should be like this:

![folders in man folder.jpg](https://github.com/figment-networks/learn-tutorials/raw/master/assets/folders_in_man_folder.jpg)

# Setting up Solana-Cli

Make sure Solana-CLI is installed by running the Solana --version in a command line. If it shows the version, then CLI is installed, otherwise, you may need to follow the installation process provided up in the requirements section.

We can first deploy our collection on **devnet** and then in case of successful launch we may want to deploy the same collection this time on **mainnet-beta**.

If you want to see current config being used in CLI, you can always run command `solana config get` in a terminal and result would be like:

![solana config get screenshot.jpg](https://github.com/figment-networks/learn-tutorials/raw/master/assets/solana_config_get_screenshot.jpg)

To change the default network to devnet first we run command `solana config set --url devnet` :

![setting solana-cli to devnet screenshot.jpg](https://github.com/figment-networks/learn-tutorials/raw/master/assets/setting_solana-cli_to_devnet_screenshot.jpg)

Then we create a wallet by running command below:

 `solana-keygen new --outfile ~/.config/solana/devnet.json`

Terminal prompts message bellow:

![enter BIP39 pashh in terminal screenshot.jpg](https://github.com/figment-networks/learn-tutorials/raw/master/assets/enter_BIP39_pashh_in_terminal_screenshot.jpg)

You can enter a password but make sure you save your entered passphrase in a safe place because if you forget it you wouldn't be able to recover your wallet.

![confirmation for creating a devnet keypair.jpg](https://github.com/figment-networks/learn-tutorials/raw/master/assets/confirmation_for_creating_a_devnet_keypair.jpg)

I'm sharing my seed phrase with you because of educational purposes and the fact that I'm gonna remove this keypair after this tutorial but you shouldn't share yours under any circumstances.

We need to copy the private key of our wallet, so keep the path to your wallet in mind:  `/home/mmostafavi/.config/solana/devnet.json`

Then we set this path as default wallet in the Solana CLI:

`solana config set --keypair /home/mmostafavi/.config/solana/devnet.json`

And finally, we airdrop some SOL to our wallet:

`solana airdrop 5`

And to check whether it was successful:

`solana balance`

It should print "5 SOL"

Ok, let's add our newly created wallet to our phantom wallet on the browser. for that first, we need to go to the path containing our wallet Keypair. First, we open a new terminal and run `cd /home/mmostafavi/.config/solana/` and now we can copy our private key from devnet.json. then open phantom and from the side menu select **Add / Connect Wallet** then **import private key** and paste your private key.

# Generating NFTs

Layers play an important role in the generation of unique NFTs. Those are like traits and characteristics and a set of these layers together creates a unique NFT. The art engine makes sure none of the two NFTs in the collection have the same layers and this way those are guaranteed to be unique.

In this tutorial, I'll step you through the process of creating and launching the NFT collection but you may need to create your layers for your collection. I've added a YouTube link in the References section for "how to create layers for our NFTs", which may be a good starting point.

This engine comes with some initial Layers and we will be using them to generate our Collection. If you open the Layers folder you'd see some folders. Each folder is a layer and contains different forms of that Layer. For instance, in the Bottom id Layer, there are three layers. High#20.png, Low#40.png, and Middle#40.png. numbers in files' names are related to their chance of appearing in an NFT.

To put it in simple words; if we generate10,000 NFTs, roughly 20% of them would have High#20.png, 40% would have Low#40.png and 40% would have Middle#40.png as their Bottom id layer. So you can specify your customization for your layers.

## Configuring our collection's metadata

We open hashlips art engine folder in VS code (or code editor of your choice) and open a new terminal ( `Ctrl + ``  or hovering over Terminal Tab and selecting open a new Terminal). 
Make sure the current directory in the terminal is hashlips art engine folder and then run `yarn install` to install dependencies.

Open "**Config.js**" file in "**src**" folder. in this file, we specify important metadata like the **name of the collection**, **website related to collection**, **network which our collection would be deployed**, etc.

```jsx
const basePath = process.cwd();
const { MODE } = require(`${basePath}/constants/blend_mode.js`);
const { NETWORK } = require(`${basePath}/constants/network.js`);

const network = NETWORK.sol;

// General metadata for Ethereum
const namePrefix = "Eyes on you";
const description = "a collection of eyes";
const baseUri = "ipfs://NewUriToReplace";

const solanaMetadata = {
  symbol: "EOY",
  seller_fee_basis_points: 250, // Define how much % you want from secondary market sales 1000 = 10%
  external_url: "https://yourwebsite.com",
  creators: [
    {
      address: "A7pGKQxXTFX8DSFURgJtqM9ccCp7RMJvhyDKZDVWPAuB",
      share: 100,
    },
  ],
};

// If you have selected Solana then the collection starts from 0 automatically
const layerConfigurations = [
  {
    growEditionSizeTo: 10,
    layersOrder: [
      { name: "Background" },
      { name: "Eyeball" },
      { name: "Eye color" },
      { name: "Iris" },
      { name: "Shine" },
      { name: "Bottom lid" },
      { name: "Top lid" },
    ],
  },
];
```

The code above shows the parts you need to adjust. 

In line 5 we replaced "eth" with "sol". 

Then we set our Collection's name and description changing values of variables "namePrefix" and "description".

Then in solanaMetadata object, we set symbol, the fee we want to charge for secondary sales of our NFTs ( 1000pts = 10% then 250pts = 2.5% ), a URI related to our NFT collection and creators' address which is the pubKey we generated at the previous step in CLI setup. ( yours would be different )

And the last thing we need to specify is the number of NFTs to be generated. I set it to 10 but you may want to have more or less. Finally, we save our changes in config.js.

If you want to dig deeper and maybe add other metadata I recommend referring to hashlips' GitHub and reading their guides.

Now we're OK with configuration and we can generate our Collection. To do so, in a terminal make sure you're in hashlips art engine folder and then run `yarn run generate`:

![NFTs generated screenshot.jpg](https://github.com/figment-networks/learn-tutorials/raw/master/assets/NFTs_generated_screenshot.jpg)

Awesome! now we have our NFTs and their metadata inside the build folder.

# Creating a Candy Machine

We will use Metaplex to create a Candy Machine for our collection on Solana. Candy machine is a minimal marketplace where people can mint our NFTs for themselves. We first head back to three folders we had then we open metaplex folder in VS code. Inside the "metaplex-master" folder we create a new folder and name it "assets". Then from the build folder in hashlips folder, we copy all the .png files and .json files except _metadata.json. the final result should be like this:

![assets_in_metaplex_screenshot.jpg](https://github.com/figment-networks/learn-tutorials/raw/master/assets/assets_in_metaplex_screenshot.jpg)

In the new VS code window open a terminal and run `cd js` then run `yarn install` wait for it to install dependencies. After completion of that run `yarn run build` and wait for that to finish as well. then run `yarn run bootstrap`.

We are done with js folder. run `cd ..`

We have prepared everything to start uploading our collection and creating a candy mint machine for our collection.

We will use candy-machine-cli.ts to set up our candy machine. To upload your collection for your candy machine We run the command bellow:

`ts-node js/packages/cli/src/candy-machine-cli.ts upload ./assets --env devnet --keypair /home/mmostafavi/.config/solana/devnet.json`

after `--keypair` you need to put your wallet's address.

![upload sucessful candy machine.jpg](https://github.com/figment-networks/learn-tutorials/raw/master/assets/upload_sucessful_candy_machine.jpg)

Wait for it to finish. The next step is creating a candy machine. for that run command bellow:

`ts-node js/packages/cli/src/candy-machine-cli.ts create_candy_machine --env devnet --keypair /home/mmostafavi/.config/solana/devnet.json -p 1`

![candy machine created screen shot.jpg](https://github.com/figment-networks/learn-tutorials/raw/master/assets/candy_machine_created_screen_shot.jpg)

Now after this is finished our candy machine is ready. We can see the pubKey of our candy machine in the terminal but you can find all information related to our candy machine in the newly generated file in .cache folder.

The last thing we need to do in metaplex is to update our candy machine. We need to update the date users can mint NFTs after that date and the price of our NFTs. in command bellow we set the price of each NFT to 0.1 SOL and we set "17 Oct 2021 00:00:00 GMT" as the launch date so we make sure users can mint as soon as we put our candy machine online.

`ts-node js/packages/cli/src/candy-machine-cli.ts update_candy_machine --env devnet --keypair /home/mmostafavi/.config/solana/devnet.json -p 0.1 --date "17 Oct 2021 00:00:00 GMT"`

![updated the candy machine with date and price.jpg](https://github.com/figment-networks/learn-tutorials/raw/master/assets/updated_the_candy_machine_with_date_and_price.jpg)

Awesome! Now we have our candy machine all set up and updated. let's build a dApp connected to our candy machine.

# Creating a candy machine dApp

To create dApp first we head back again to three folders we had and this time we open the third folder "candy-machine-mint-main". 

First, we want to install all the dependencies of this project. so we need to run `yarn install`

We have a .env.example file with the content below:

```
REACT_APP_CANDY_MACHINE_CONFIG=__PLACEHOLDER__
REACT_APP_CANDY_MACHINE_ID=__PLACEHOLDER__
REACT_APP_TREASURY_ADDRESS=__PLACEHOLDER__
REACT_APP_CANDY_START_DATE=__PLACEHOLDER__

REACT_APP_SOLANA_NETWORK=devnet
REACT_APP_SOLANA_RPC_HOST=https://explorer-api.devnet.solana.com
```

Network and RPC host is set correctly since we are launching on devnet but we need to replace __PLACEHOLDER__ with appropriate data.

If we head back to the previous folder we were in, Metaplex-master, now there is a new folder "**.cache**" and inside that a new file named "**devnet-temp**" basically that's all the information related to the candy machine we created. from this file we want to pick some values to fill out our .env file with.

from **devnet-temp** find:

- program.config and copy its value to REACT_APP_CANDY_MACHINE_CONFIG
- authority and copy its value to REACT_APP_TREASURY_ADDRESS
- candyMachineAddress and copy its value to REACT_APP_CANDY_MACHINE_ID
- startDate and copy its value to REACT_APP_CANDY_START_DATE

Now we have set up everything up for our dApp. We can run `yarn start` to run our dApp locally on localhost;//3000.

![initial state of dapp candy machine screenshot.jpg](https://github.com/figment-networks/learn-tutorials/raw/master/assets/initial_state_of_dapp_candy_machine_screenshot.jpg)

Now let's connect our phantom wallet

![cansy store up and running wallet connected.jpg](https://github.com/figment-networks/learn-tutorials/raw/master/assets/cansy_store_up_and_running_wallet_connected.jpg)

Awesome! now we have an ugly-looking dApp! but we can change the UI of our website inside the "candy-machine-mint-main" folder, so it's not a big deal.

You may ask why we launched it on devnet and not on mainnet-beta?! for two reasons:

1. We can estimate the costs of our launch on mainnet-beta
2. We can make sure there isn't any spooky bug stopping us in the middle of the process

Awesome! now we have a react candy mint machine app we can deploy to any hosting service we may want. if you're curious how you can deploy your collection to mainnet-beta read the next section.

# Launching on Mainnet-beta

We successfully launched our collection on devnet and now we are ready to deploy on mainnet-beta. There isn't any significant difference between deploying on devnet and mainnet-beta. We will need to switch back to mainnet-beta by running `solana config set --url mainnet-beta` and then this time we need to charge our wallet with real SOL. you can easily purchase some from an exchange like [FTX](https://ftx.com/). Let's assume we've purchased some and transferred them to our local wallet we created at beginning of this tutorial.

In the second stage of the tutorial where we are creating a candy machine and uploading the assets, we need to change `--env devnet` to  `--env mainnet-beta` in all three commands.

so new commands would be:

1- `ts-node js/packages/cli/src/candy-machine-cli.ts upload ./assets --env mainnet-beta --keypair /home/mmostafavi/.config/solana/devnet.json`

2- `ts-node js/packages/cli/src/candy-machine-cli.ts create_candy_machine --env mainnet-beta --keypair /home/mmostafavi/.config/solana/devnet.json -p 1`

3- `ts-node js/packages/cli/src/candy-machine-cli.ts update_candy_machine --env mainnet-beta --keypair /home/mmostafavi/.config/solana/devnet.json -p 0.1 --date "17 Oct 2021 00:00:00 GMT"`

Easy peasy! 

In the third stage of the tutorial where we are configuring our dApp we will need to some adjustments. This time inside "**.cache**" folder we would have "**mainnet-beta-temp**" instead of "**devnet-temp**" but the structure of files is the same. But because this time we are launching on mainnet-beta, we need to set the last two env variables like below:

```
REACT_APP_CANDY_MACHINE_CONFIG=__PLACEHOLDER__
REACT_APP_CANDY_MACHINE_ID=__PLACEHOLDER__
REACT_APP_TREASURY_ADDRESS=__PLACEHOLDER__
REACT_APP_CANDY_START_DATE=__PLACEHOLDER__

REACT_APP_SOLANA_NETWORK=mainnet-beta
REACT_APP_SOLANA_RPC_HOST=https://explorer-api.mainnet-beta.solana.com
```

That's All of the adjustments you need to make to launch your minting dApp on mainnet-beta.

# Conclusion

In this tutorial, we learned how to generate unique NFTs and then create a minting dApp for them.

# About the Author

I'm Mahdi Mostafavi a web developer and I'm learning about Solana. my GitHub: [@mmostafavi](https://github.com/mmostafavi)

If you found any misinformation in the article or had any suggestions to improve the tutorial you can message me on Twitter: [@mahdi_ftp](http://twitter.com/mahdi_ftp).

# References

hashlips art engine: [https://github.com/HashLips/hashlips_art_engine](https://github.com/HashLips/hashlips_art_engine)

metaplex: [https://github.com/metaplex-foundation/metaplex](https://github.com/metaplex-foundation/metaplex)

candy machine: [https://github.com/exiled-apes/candy-machine-mint](https://github.com/exiled-apes/candy-machine-mint)‣

how to create layers for our NFTs: [https://www.youtube.com/watch?v=eYkAu4eZG80](https://www.youtube.com/watch?v=eYkAu4eZG80)
