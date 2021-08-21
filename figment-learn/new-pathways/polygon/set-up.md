# Set up your Project

## Requirements

* [Node.js](https://nodejs.org) v14+ installed (we recommend using [nvm](https://github.com/nvm-sh/nvm) or [fnm](https://github.com/Schniz/fnm))
* [yarn](https://yarnpkg.com/) installed
* [git](https://git-scm.com/book/en/v2/Getting-Started-Installing-Git) installed
* [Metamask](https://metamask.io/) browser extension installed. Chrome users may want to pin the extension to the Extensions toolbar for easy access. Firefox does this by default.

![Pin Metamask to the toolbar in Chrome](../../../.gitbook/assets/pin_metamask.png)

{% hint style="info" %}
If you **ALREADY** have Metamask installed and are using it for a hot wallet, we _**strongly recommend**_ creating an entirely new wallet in Metamask for the purposes of these tutorials. Figment Learn wants nothing to do with your personal keys. We do not want any accidents involving anybody's cryptocurrency! Again, you _must not_ _continue_ until you take care of this.  
{% endhint %}

{% hint style="danger" %}
If you **DO NOT** already have Metamask installed, welcome to the wonderful world of web3!   
The first piece of advice we will give you is to make _absolutely sure that you write down the_ [_Secret Recovery Phrase_](https://community.metamask.io/t/what-is-a-secret-recovery-phrase-and-how-to-keep-your-crypto-wallet-secure/3440) that is generated during Metamask's initial setup. Do not store it in a digital format. Do not share it with anybody. Please do keep it accessible to yourself, because we will be using it during the pathway. 
{% endhint %}

## Project Setup

The Polygon Pathway is in the form of a Next.js app which can be run on your local machine. We will be interacting with it through the web browser, and making changes to the code to make it a basic dApp, using Polygon!

Run the following command in a terminal to clone the `learn-web3-dapp` code repository and install the dependencies with `yarn` :

{% hint style="info" %}
If you find yourself confused by any of the terminology used in the tutorials, or are new to software development in general, please refer to this Beginners Guide to Learn.
{% endhint %}

```bash
git clone https://github.com/figment-networks/learn-web3-dapp.git
cd learn-web3-dapp
yarn
```

Once the installation is complete, then start the local development server with:

```bash
yarn dev
```

{% hint style="warning" %}
Also be sure to rename the file `.env.example` to `.env.local` before continuing with the Pathway. This file is where we define various endpoint URLs and related data like API keys.
{% endhint %}

## The Pathway UI

Now visit the URL [`http://localhost:3000`](http://localhost:3000) in your web browser to see the All Pathways page:

![Click on the Polygon icon to begin your journey on the Polygon Pathway!](../../../.gitbook/assets/all_pathways.png)

In these tutorials we will cover connecting to and interacting with Polygon using the Metamask wallet, as this is the most common use-case even for development. Moving on, we will look at querying information from the blockchain and displaying it on the UI using the ethers library. Having tokens to test with is an important consideration, and we cover how to get test tokens from a faucet website. We will also discuss Solidity, the language used to write Ethereum and Polygon smart contracts. The basic workflow of writing, testing and deploying smart contracts will be the homestretch. We can then interact with the deployed contract to demonstrate its functionality!  

By the end of the Pathway tutorials, you will be confidently interacting with Polygon, sending transactions which cost fractions of a cent and deploying smart contracts.

## Next Steps

Now that we gave you a quick overview of Polygon ecosystem, we still need to set up our environment.
