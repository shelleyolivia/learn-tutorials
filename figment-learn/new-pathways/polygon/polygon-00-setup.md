# 

## Requirements

* [Node.js](https://nodejs.org) v14+ installed (we recommend using [nvm](https://github.com/nvm-sh/nvm))
* [yarn](https://yarnpkg.com/) 
* [git](https://git-scm.com/book/en/v2/Getting-Started-Installing-Git) installed
* [Metamask](https://metamask.io/) browser extension installed. 


## Safety disclaimers

{% hint style="info" %}
If you **ALREADY** have Metamask installed and are using it for a hot wallet, we _**strongly recommend**_ creating an entirely new wallet in Metamask for the purposes of these tutorials. Figment Learn wants nothing to do with your personal keys. We do not want any accidents involving anybody's cryptocurrency! Again, you _must not_ _continue_ until you take care of this.  
{% endhint %}

{% hint style="danger" %}
If you **DO NOT** already have Metamask installed, welcome to the wonderful world of web3!   
The first piece of advice we will give you is to make _absolutely sure that you write down the_ [**Secret Recovery Phrase**](https://community.metamask.io/t/what-is-a-secret-recovery-phrase-and-how-to-keep-your-crypto-wallet-secure/3440) that is generated during Metamask's initial setup. Do not store it in a digital format. Do not share it with anybody. Please do keep it accessible to yourself, because we will be using it during the pathway. 
{% endhint %}

-------------------------------------

## Project Setup

The **Polygon** Pathway is in the form of a Next.js app which can be run on your local machine. We will be interacting with it through the web browser, and making changes to the code to make it a basic dApp, using **Polygon**!

Run the following command in a terminal to clone the `learn-web3-dapp` code repository and install the dependencies with `yarn` :

```bash
git clone https://github.com/figment-networks/learn-web3-dapp.git
cd learn-web3-dapp
cp .env.example .env.local  
yarn
```

Once the installation is complete, then start the local development server with:

```bash
yarn dev
```

{% hint style="warning" %}
Also be sure to rename the file `.env.example` to `.env.local` before continuing with the Pathway. This file is where we define various endpoint URLs and related data like API keys.
{% endhint %}

-------------------------------------

## The Pathway UI

Now visit the URL [`http://localhost:3000`](http://localhost:3000) in your web browser to see the home page:

![](../../../.gitbook/assets/pathway-home.gif)

In these tutorials we will cover connecting to and interacting with **Polygon** using the Metamask wallet, as this is the most common use-case even for development. 

-------------------------------------


## Add the Mumbai testnet 

The first task is to connect to the Polygon Mumbai testnet by adding it to the list of RPC endpoints in Metamask. Click on the Fox head icon in your web browser to open the popup, and then follow this workflow to complete the process :

* Click on the current network at the top of the Metamask popup (by default is says "Ethereum Mainnet")
* Scroll down and click on "Custom RPC"
* Fill in the form:
  * Network Name: `Polygon Mumbai`
  * New RPC URL: `https://rpc-mumbai.maticvigil.com/`
  * Chain ID: `80001`
  * Currency Symbol: `MATIC`
  * Block Explorer URL : `https://mumbai.polygonscan.com`
* Double check the information, then click on the Save button.

![](../../../.gitbook/assets/add_mumbai.png)

We use the testnet for development before moving into production on the main network or "mainnet".

-------------------------------------

## Next Steps

Now that we have our environment set up we can connect to the network.
