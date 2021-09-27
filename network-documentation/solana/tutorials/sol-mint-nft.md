# How to create an NFT Marketplace on Solana

## Minting a non-fungible token on the Solana blockchain

## Introduction

In this tutorial we will be creating a [non-fungible token](https://www.blockchain-council.org/blockchain/a-quick-guide-to-fungible-vs-non-fungible-tokens/#:~:text=Non%2DFungible%20are%20Non%2DInterchangeable,token%20of%20the%20same%20type.) marketplace, otherwise known as an NFT, using the **Solana** blockchain. Since NFT's are 1 of 1, they can represent real-world objects like music, art, in-game items and videos.

We are going to create our NFT on our own marketplace. Refer to this [existing marketplace](https://jacobmakarsky.github.io/metaplex#/art/create) I created for example, built with [Metaplex](https://github.com/metaplex-foundation/metaplex).

Once we create the NFT, we will be able to list it for sale on larger marketplaces such as [DigitalEyes](https://digitaleyes.market/) or [Solanart](https://solanart.io/).

## Prerequisites

* Around $2 USD of [SOL](https://coinmarketcap.com/currencies/solana/) accessible, which we will send to a browser wallet. You can get some SOL through an exchange, such as [FTX](https://ftx.com/#a=13426316).
* Something to upload in the NFT, such as an image, video, audio file or an AR/3D file (.glb).
* [Github account](https://github.com/)
* [Git](https://git-scm.com/) installed

### **1) Create an SOL wallet**

First, we are going to create a wallet on Solana using [Phantom wallet](https://phantom.app/).

Follow the steps on Phantom's website on how to create a wallet: [How to create a new wallet](https://help.phantom.app/hc/en-us/articles/4406388623251-How-to-create-a-new-wallet)

### **2) Fund the wallet**

Since minting an NFT on the blockchain causes a change in the data on the blockchain, we will have to pay for the transaction. Solana uses [SOL token](https://coinmarketcap.com/currencies/solana/) to pay for their transactions, so we will need to fund our new phantom wallet with SOL token. 

Follow the steps on Phantom's website here on how to deposit SOL to your wallet: [How to deposit SOL](https://help.phantom.app/hc/en-us/articles/4406393831187-How-to-deposit-SOL).

### ** 1) Fork the Metaplex repository to your Github

First, we are going to fork the metaplex repository to our Github so we can host it on Github pages. 

* Fork the [Metaplex repository](https://github.com/metaplex-foundation/metaplex)

![metaplexRepo](../../../.gitbook/assets/metaplexRepo.png)

### ** 2) Clone the repository to your desktop

Now, we need to copy all of the Metaplex files over to our computer.

* Create a folder on your desktop called "myNftMarketplace"

* Copy the location of the folder

* Change directory to your marketplace folder in the command line

```bash
cd /Users/yourusername/Desktop/myNftMarketplace 
```

![cdMarketplaceSol](../../../.gitbook/assets/cdMarketplaceSol.png)

* Clone the repository to your marketplace folder

```bash
git clone https://github.com/yourusername/metaplex.git
```

### ** 3) Change the default addresses

Whenever we deploy the application, we want it to save to the forked Metaplex repository that we created. This will allow us to easily get our own live marketplace online.

* Change the deployment address to your Github username

![deployAddressSol](../../../.gitbook/assets/deployAddressSol.png)

Notice where my name is. This should be where our Github username goes. Make sure to follow the path taken, on the left of the picture, to get to the `package.json` file.

* Change the store's wallet address

![storeWalletAddress](../../../.gitbook/assets/storeWalletAddress.png)

This is where we add the public address of our browser wallet, so the store knows who to give administrative access to. Make sure to follow the path taken, on the left of the picture, to get to the `.env` file.

### ** 4) Setup and deploy our store

We are in the final stages of making our store live! Now, we will download all of the necessary packages that will allow us to deploy our marketplace website. 

* Change directory to the `js` folder in the command line

```bash
cd metaplex-master/js
```

We should still have our terminal open from before, when we changed directory into our marketplace folder. Now, we changed directory to the `js` folder. 

* Run the `yarn` command to download all of the dependencies

![cdJsSol](../../../.gitbook/assets/cdJsSol.png)

This command will begin downloading all of the dependencies that are required to work with the code. When we cloned the Metaplex repository, it didn't download the packages for us, it only downloaded the code that says what packages/dependencies are required. 

The command will take awhile to finish downloading everything. Once it's done, we will see a `Done` line at the end. 

![yarnPackagesSol](../../../.gitbook/assets/cdJsSol.png)

* Run the `yarn bootstrap` command to check that all of the dependencies are working.

```bash
yarn bootstrap
```

We can now locally test our new NFT marketplace before we upload it to the internet. 

* Run the `yarn start` command to check out our marketplace locally

```bash
yarn start
```

It may take several minutes for everything to compile. Once the terminal says `compiled successfully`, we can visit `http://localhost:3000/#/` to view our brand new marketplace! But, this is only good for browsing around. Wait until we get it live online to connect a wallet and mint something.

* Run the `yarn build` command

![yarnBuildSol](../../../.gitbook/assets/yarnBuildSol.png)

After a bit of time, we should see a `Done` line, indicating the build has finished. We will now see a `build` folder inside the `js` folder. 

* Change directory to the `web` folder

```bash
cd packages/web
```

* Run the `yarn deploy` command

We should again see a `Done` line, indicating the deployment has finished. This command has deployed our marketplace to our Github repository, which means... our marketplace is live!

* Go check out our fresh new NFT marketplace on Solana

Congratulations! The URL for our marketplace will most likely be located at `https://yourusername.github.io/metaplex#/`, unless you changed the URL for your website in the Github repository's settings. For example, my marketplace is located at `https://jacobmakarsky.github.io/metaplex#/`. 

Now, we can move on to creating our own NFT on Solana by using our marketplace. 

### **5) Mint an NFT on a marketplace**

There are many marketplaces on Solana for listing NFT's, but only very recently was a tool created that allows the everyday user to mint their own NFT, called [Metaplex](https://www.metaplex.com/).

Metaplex allows us to create our own NFT marketplace, but does not have one general NFT marketplace for everyone to use. I made a marketplace for us to use in this tutorial [under my github here](https://jacobmakarsky.github.io/metaplex#/art/create). 

* Head over to the "My Items" tab

![myItemsPage](../../../.gitbook/assets/myItemsPage.png)

* Click "Create" at the top

![createNFTButton](../../../.gitbook/assets/createNFTButton.png)

Now we should be on the "Category" section. 

* Choose which kind of NFT we are creating. 

I created an AR file that people could interact with on [Vecteezy](https://www.vectary.com/3d-modeling-news/create-ar-content-ARkit-ios12/). If you take this route to make something quick, make sure you export to .obj file type, and then convert the 2 files given after conversion to .glb [here](https://products.aspose.app/3d/conversion/obj-to-glb). Making an AR file gives us the ability to interact with and spin our NFT around. 

* On the "Upload" section, follow the instructions and upload what's required depending on the category of the NFT you chose. ***Make sure your filename does not have spaces in it.*** 

* Press "Continue to Mint".

![uploadNFTImages](../../../.gitbook/assets/uploadNFTImages.png)

* Go ahead and give the NFT a cool title and a description about what it is. 

Feel free to stick in a link or any info that would help someone understand what the NFT is. Or, leave the description empty and mysterious. 

The maximum supply is the number of prints that would be created of the NFT, with each NFT being a numbered edition. The creator can set the “Maximum Supply” of the master edition just like a regular mint on Solana, with the main difference being that each print is a numbered edition created from it. - [Metaplex Developer Guide](https://docs.metaplex.com/)

The attributes, for example, would be "background: blue", "eyes: closed", "mouth: smoking" etc.

* Press "Continue to Royalties".

![nftInfoPage](../../../.gitbook/assets/nftInfoPage.png)

Metaplex allows us to modify the royalties and amount to split of the intiial sale. 

* Set a royalties percentage, so the creators will recieve part of every future sale that happens of the NFT.

Make sure to add creators if anyone else had taken part in creating the NFT.

![nftRoyaltiesPage](../../../.gitbook/assets/nftRoyaltiesPage.png)

It's time to launch our NFT!

* Make sure there is enough SOL in your browser wallet and click "Pay with SOL". 

It will take up to a few minutes for the NFT to upload, as the metadata (title, description, attributes etc.) is being saved on [Arweave](https://www.arweave.org/). This is a blockchain tool for storing data eternally. Storing data on the Solana chain itself would be too expensive, so our non-fungible token on Solana contains a link to where the NFT information is stored on Arweave. 

![nftLaunchPage](../../../.gitbook/assets/nftLaunchPage.png)

Wooo! Your new NFT should be finished minting. Congratulations, you now have an accessible NFT on the Solana network. What can we do with it now?

![nftMintFinished](../../../.gitbook/assets/nftMintFinished.png)

* Check the "My Items" tab for your NFT

![myItemsPage](../../../.gitbook/assets/myItemsPage.png)

You should be able to see your NFT on the [My Items](https://jacobmakarsky.github.io/metaplex#/artworks) page. 

If you've been using a Phantom browser wallet, go ahead and check your "Collectibles" tab. Your brand new NFT should appear in your wallet! It may take some time for the images and metadata to be synced around. 

![phantomCollectibles](../../../.gitbook/assets/phantomCollectibles.png)

### **6) BONUS: List the NFT for sale**

Awesome, we now have our own NFT that we can view inside our wallet. We certainly don't have to list our fresh NFT for sale if we don't want to, but for the sake of education, let's go ahead and try listing it. 

* Go to [DigitalEyes](https://digitaleyes.market/)

I wouldn't recommend listing the NFT on the marketplace I made for us (even though I do plan to make it cool in the future), so for now DigitalEyes is the best option. 

* Connect your wallet

![connectWalletToMarket](../../../.gitbook/assets/connectWalletToMarket.png)

* Go to the "Sell" tab 

![yourNFTs](../../../.gitbook/assets/yourNFTs.png)

* Click on the NFT you want to sell

* Choose the amount you want to sell it for and list the NFT

![listNFTButton](../../../.gitbook/assets/listNFTButton.png)

* Approve the transaction to list the NFT

The transaction cost me 0.1 SOL, so make sure you have enough SOL in your wallet to list the NFT. 

![approveNFTTransaction](../../../.gitbook/assets/approveNFTTransaction.png)

Now we should be able to see our NFT listed on the marketplace! Search for the "Unverifeyed" collection in the search bar, and scroll to your NFT. The marketplaces are still in development, so hopefully it'll be much easier to search for the NFT's soon. 

![listedNFTs](../../../.gitbook/assets/listedNFTs.png)

Go ahead and click on the NFT that's listed. You'll see the option to unlist it and some labels saying "Unverifeyed". This is just because DigitalEyes has not verified our NFT, so they aren't sure if it's a fake copy of an exisiting collection. For any questions about DigitalEyes, such as getting your NFT verified, [go here](https://digitaleyes.market/faq). 

# Conclusion

Woohoo! If you made it through all the steps, you have successfully created your own NFT on the Solana blockchain. If you completed the bonus, you now have an NFT on the worldwide marketplace 👍

# Next Steps

In this tutorial, we created our own NFT on the Solana blockchain using Metaplex for minting and DigitalEyes for listing. Feel free to continue experimenting by attempting to mint more NFT's, sending the token around to other wallets, creating a collection, etc.

# About the author

This tutorial was created by Jacob Makarsky. He can be found on [Github](https://github.com/jacobmakarsky) or the [Figment Forum](https://community.figment.io/u/jacobmakarsky/summary).

# References

* [James Bachini](https://www.youtube.com/watch?v=_36JcJRAHHI)
* [Solana's Blog](https://solana.blog/solana-metaplex-tutorial-deploy-your-own-store-mint-nfts-and-setup-auctions/)