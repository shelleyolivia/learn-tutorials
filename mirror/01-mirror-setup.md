# Step 1: Template and Configuration

You'll be building the blog application on top of a [Next.js](https://nextjs.org/) template that we have pre-built. This way you can quickly scaffold your way into developing the Web 3 dApp with minimal configuration.

## Setting Up

Make sure you have [git](https://git-scm.com/book/en/v2/Getting-Started-Installing-Git), [Node](https://nodejs.org/en/) and [yarn](https://yarnpkg.com/getting-started/install) installed. Make sure you've setup [SSH or token-based authentication](https://github.blog/2020-12-15-token-authentication-requirements-for-git-operations/) for Github. Then clone the repo and run the yarn command to install the app dependencies:

```bash
$ git clone git@github.com:figment-networks/web3-evm-template.git
$ cd web3-evm-template
$ yarn
```

If you into any Node issues, make sure you're running version 14.17.0 or greater. You can use the [Node Version Manager](https://github.com/nvm-sh/nvm) if you need to manage multiple Node versions locally.

Now that you have the template, you need to navigate to this tutorial's branch:

```bash
$ git checkout mirror-clone
```

Next, you'll need to copy the example environment file `.env.local.example` to set the environment variables we'll need to write and deploy the smart contract. 

```bash
$ cp .env.local.example .env.local
```

Navigate to `.env.local` and set the four environment variables based on the instructions included below.

```text
ETHERSCAN_API_KEY=Polygonscan API Key. Go to https://polygonscan.com/login and login or create an account. Then click on API-KEYs on the left navigation panel to create an api key. Name the app MirrorClone. Copy-paste the API key here.
MAINNET_NODE_URL=https://matic-mainnet.chainstacklabs.com
TESTNET_NODE_URL=https://matic-mumbai.chainstacklabs.com
PRIVATE_KEY=Your private key exported from MetaMask. Go to https://metamask.zendesk.com/hc/en-us/articles/360015289632-How-to-Export-an-Account-Private-Key to learn how to export your private key.
```

{% sidenote title="Box 1.1: What is MetaMask?" %}
[MetaMask](https://metamask.io/) is a crypto wallet built as a browser extension that allows you to interact with Ethereum and Ethereum-compatible blockchains as well as dApps built on top of them.

{% hint style="warning" %}
The **private key** environment variable is required to deploy the smart contract in Step 6. Despite the fact that the `.env.local` file has been placed in `.gitignore` already so you don't expose your private key, we still recommend you create a wallet whose sole purpose is its use in tutorials and local development. That way, you can mitigate the risk of exposing your wallet's private key.
{% endhint %}

Finally, we want to add certain libraries that will set us up for smart contract development in Step 6 but without which we'll get a few compiling errors.

```bash
$ yarn web3:compile
```

With that in place, our dApp template is ready to be built into a functional blog. You can run the local server to get started:

```bash
$ yarn dev
```

## Reviewing the Template

Before we dive into the code, we should understand what the template comes with. Along with the standard folder structure and dependencies found in most NextJS applications, we have added two Web 3-specific libraries that we'll leverage throughout the tutorial - [ethers.js](https://docs.ethers.io/) and [Hardhat](https://hardhat.org/).

Ethers is a JavaScript library designed to interact with Ethereum and Ethereum-compatible blockchains. We'll first encounter ethers in Step 2 to interface with MetaMask and connect our wallet.

Hardhat is a development environment for Ethereum and Ethereum-compatible blockchains. It allows you to write and compile Solidity locally, run tests, and deploy smart contracts, all while fully supporting TypeScript. We'll leverage Hardhat in Step 6 to manage development of the smart contract used to mint and transfer NFTs related to blog posts.

Feel free to explore the rest of the template. It includes standard hooks and components that we'll leverage to add functionality to our blog. But that's enough set up for now.

Let's build!

![Figure 3: “I can feel the anticipation.”](https://raw.githubusercontent.com/figment-networks/learn-tutorials/mirror-tutorial/mirror/assets/lego.jpeg?raw=true)

{% label %}
Figure 3: “I can feel the anticipation.”
