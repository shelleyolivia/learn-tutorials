You'll be building the blog application on top of a [Next.js](https://nextjs.org/) template that we have pre-built. This way you can quickly scaffold your way into developing the Web 3 dApp with minimal configuration.

# Add the Mumbai testnet to MetaMask ⛓

You'll need to connect to the Polygon Mumbai testnet for these tutorials, so add it to the list of RPC endpoints in MetaMask. Click on the Fox head icon in your web browser to open the popup, and then follow this workflow to complete the process :

- Click on the current network at the top of the MetaMask popup (by default is says "Ethereum Mainnet")
- Scroll down and click on "Custom RPC"
- Fill in the form:
  - Network Name: `Polygon Mumbai`
  - New RPC URL: `https://rpc-mumbai.maticvigil.com/`
  - Chain ID: `80001`
  - Currency Symbol: `MATIC`
  - Block Explorer URL : `https://mumbai.polygonscan.com`
- Double check the information, then click on the **Save** button.

![](https://raw.githubusercontent.com/figment-networks/learn-web3-dapp/main/markdown/__images__/polygon/add_mumbai.png?raw=true)

We strongly recommend using the testnet for development before moving into production on the main network or "mainnet".

# Fund your account with MATIC 🤑

You will need some MATIC tokens to be able to fund the deployment of the smart contract in one of the following tutorials! Think of this as stopping at the service station to fill up your gastank before a long road trip.

Visit the Polygon Faucet at [https://faucet.polygon.technology](https://faucet.polygon.technology) and paste the address from your selected account in MetaMask into the textinput. It is OK to leave the default options for the **Mumbai** network and **MATIC** tokens selected. 
Click the **Submit** button, then click again on the popup to confirm the transaction. You'll be rewarded with 0.5 MATIC every time you do this, however there is a rate limit so don't get greedy. Half a MATIC is plenty to deploy several contracts and run test transactions.

# Getting Started 🎬

Make sure you have [git](https://git-scm.com/book/en/v2/Getting-Started-Installing-Git), [Node](https://nodejs.org/en/) and [yarn](https://yarnpkg.com/getting-started/install) installed. Make sure you've setup [SSH or token-based authentication](https://github.blog/2020-12-15-token-authentication-requirements-for-git-operations/) for Github. 

Use `git` to clone the project template code repository, then change into the root directory of the project. 

```text
$ git clone git@github.com:figment-networks/web3-evm-template.git
$ cd web3-evm-template
```

{% hint style="warning" %}
When you see a code block that has lines beginning with `$`, it indicates that the commands should be run in your terminal. The dollar sign is a _command prompt_ and should not be included in the command.
{% endhint %}

Now that you have the template, you need to navigate to the branch containing the code for this tutorial and then install the app dependencies with the yarn package manager:

```text
$ git checkout mirror-clone
$ yarn
```

If you run into any Node.js issues, make sure you're running version 14.17.0 or greater. You can use the [Node Version Manager](https://github.com/nvm-sh/nvm) if you need to manage multiple Node versions locally.

Next, you'll need to copy the example environment file `.env.local.example` to set the environment variables needed to deploy the smart contract with Hardhat.

```text
$ cp .env.local.example .env.local
```

Now open the file `.env.local` and set the four environment variables based on the instructions included below. If you are not familiar with them, read more about the [dotenv package and `.env` files](https://docs.figment.io/network-documentation/extra-guides/dotenv-and-.env).

```text
POLYGONSCAN_API_KEY=Polygonscan API Key. Go to https://polygonscan.com/login and login or create an account. Then click on API-KEYs on the left navigation panel to create an api key. Name the app MirrorClone. Copy-paste the API key here.
MAINNET_NODE_URL=https://matic-mainnet.chainstacklabs.com
TESTNET_NODE_URL=https://matic-mumbai.chainstacklabs.com
PRIVATE_KEY=The private key of an account funded with MATIC that you've exported from MetaMask. Go to https://metamask.zendesk.com/hc/en-us/articles/360015289632-How-to-Export-an-Account-Private-Key to learn how to export your private key.
```

If you don't already have MetaMask installed as a browser extension, you should do that before proceeding. You can go to its download page [here](https://metamask.io/download.html).

{% sidenote title="Box 1.1: What is MetaMask?" %}
[MetaMask](https://metamask.io/) is a crypto wallet built as a browser extension that allows you to interact with Ethereum and Ethereum-compatible blockchains as well as dApps built on top of them.

{% hint style="warning" %}
The **PRIVATE_KEY** environment variable is required to deploy the smart contract in Step 6. Despite the fact that the `.env.local` file has been placed in `.gitignore` already so that you don't expose your private key, we still recommend you create a wallet specifically for use in tutorials and local development. That way, you can mitigate the risk of exposing your wallet's private key in case it contains any mainnet tokens with monetary value.
{% endhint %}

Finally, we want to add certain libraries that will set us up for smart contract development in Step 6 but without which we'll get a few compiling errors. 

```text
$ yarn web3:compile
```

With that in place, our dApp template is ready to be built into a functional blog. You can run the local server to get started - once it is up and running, view in your web browser by going to [http://localhost:3000](http://localhost:3000).

```text
$ yarn dev
```

# Reviewing the Template 🧐

Before we dive into the code, we should understand what the template comes with. Along with the standard folder structure and dependencies found in most NextJS applications, we have added two Web 3-specific libraries that we'll leverage throughout the tutorial - [ethers.js](https://docs.ethers.io/) and [Hardhat](https://hardhat.org/).

Ethers is a JavaScript library designed to interact with Ethereum and Ethereum-compatible blockchains. We'll first encounter ethers in Step 2 to interface with MetaMask and connect our wallet.

Hardhat is a development environment for Ethereum and Ethereum-compatible blockchains. It allows you to write and compile Solidity locally, run tests and deploy smart contracts, all while fully supporting Typescript. We'll leverage Hardhat in Step 6 to manage development of the smart contract used to mint and transfer NFTs related to blog entries.

Feel free to explore the rest of the template on your own. It includes standard hooks and components that we'll use to add functionality to our blog. But that's enough set up for now.

Let's build!

![“I can feel the anticipation.”](https://raw.githubusercontent.com/figment-networks/learn-tutorials/mirror-tutorial/mirror/assets/lego.jpeg?raw=true)

{% label %}
“I can feel the anticipation.”
