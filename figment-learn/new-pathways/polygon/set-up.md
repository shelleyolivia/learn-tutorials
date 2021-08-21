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
