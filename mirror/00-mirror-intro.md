# Blog Platform Tutorial

Welcome to the Blog Platform Tutorial. The goal of this tutorial is to introduce the Web 3 development process in the context of a full stack decentralized application. Specifically, we'll be bridging you from Web 2 to Web 3 by building a blog similar to [Mirror.xyz](https://mirror.xyz/) using a Web 3 stack. As you build, we'll introduce key concepts to help you scaffold your mental model and start adding Web 3 skills into your toolbox.

## Prerequisites

If you know how to program and have [JavaScript](https://www.javascript.com/) experience, you'll be able to complete the tutorial. Having said that, you'll be more comfortable if you have some experience with [TypeScript](https://www.typescriptlang.org/), [React](https://reactjs.org/), and [Next.js](https://nextjs.org/). Moreover, while you may not have built full stack Web 3 applications before, hopefully you have used a crypto wallet like [MetaMask](https://metamask.io/) and you're familiar with basic blockchain concepts like public address, private keys, and signing transactions.

If any of that sounds unfamiliar, or the idea of engaging with those concepts sounds daunting at this point in your learning, we recommend starting with the [Solana Wallet](https://learn.figment.io/tutorials/solana-wallet-intro) tutorial before completing this one.

As always, we think of ourselves as your guide - for a brief amount of time - on your journey to build a better internet. We hope you walk away with a better understanding of what it takes to develop full stack Web 3 decentralized applications (dApps) and perhaps even a bit more confidence to tackle significant projects.

We've tried to show you the door (or maybe part of it). Your job is to discover your path and walk it courageously.

![Figure 1: “I do not believe it to be a matter of hope, it is simply a matter of time.”](https://raw.githubusercontent.com/figment-networks/learn-tutorials/mirror-tutorial/mirror/assets/matrix.jpeg?raw=true)

{% label %}
Figure 1: “I do not believe it to be a matter of hope, it is simply a matter of time.”

## Tutorial Structure

The tutorial is structured as a set of steps that break down the development process into specific work blocks on each part of the stack. Each step discusses key concepts and mental models that create the necessary context for you to better understand what we're building and why. Think of this as the **warm up**.

After the warm up, we provide a step-by-step implementation of the functionality including a code snippet of the full solution. While the warm up served to build context, this part serves to guide you through the logic of the **implementation** and helps scaffold your mental model for the feature.

Finally, we **challenge** you to implement the feature on your own by providing just enough instructions. This allows you to exercise [active recall](https://en.wikipedia.org/wiki/Active_recall) and increases your ability to make connections.

## Blog Preview

By the time you're done, you'll have a blog similar to [Mirror.xyz](https://mirror.xyz/). Users will be able to write posts, publish them, mint NFTs for each post, and transfer those NFTs to other public addresses.

The posts will be saved on [Arweave](https://www.arweave.org/), a decentralized storage solution for immutable data storage, and the NFTs will be [ERC-721](https://eips.ethereum.org/EIPS/eip-721) tokens on [Polygon](https://polygon.technology/), a protocol for scalable Ethereum dApps.

Step 1 will walk you through some basic set up and configuration. This will include cloning a template repo so you don't have to write code you're already familiar with (i.e. NextJS). It will also include setting up a few environment variables you'll need in later steps.

In Step 2, we'll introduce the [ethers.js](https://docs.ethers.io/) library and learn how to a user's MetaMask wallet to the dApp.

> TBD insert screenshots
![Screenshot of connecting wallet]()

Then we'll introduce [Arweave](https://www.arweave.org/) - a decentralized, censorship resistant storage solution - in Step 3. We'll write an endpoint to create blog posts and finish a form to submit them. In Step 4 we'll learn how to fetch them and in Step 5 we'll write code to display a list of the latest posts.

![Screenshot displaying a list of posts]()

Step 6 will be a brief introduction to smart contracts including inheriting from standard libraries, writing functions, testing them and deploying the smart contract to the Polygon testnet. We'll be minting an NFT for each post so authors can own their posts.

![Screenshot of NFT]()

Finally, Step 7 will implement functionality that allows author's to transfer NFTs. This sets up the application for expansion into a marketplace for blog post ownership.

![Screenshot of transfer]()

There are countless ways a dApp like this could be expanded. You could implement the NFT marketplace natively, or you might choose to implement a tipping button for readers who want to reward the writer.

You could even leverage the same smart contracts and Arweave data to restructure the dApp altogether and create a completely new client-side application. Part of what makes Web 3 so powerful is the composability of its stack. There are endless opportunities to get creative and build when data gardens aren't walled and users own their data.

At the end of the tutorial, we'll include a few [additional resources]() if you want to dive deeper into these topics or take the next step in your learning journey.

Let's get started!

![Figure 2: All big things come from small beginnings](https://raw.githubusercontent.com/figment-networks/learn-tutorials/mirror-tutorial/mirror/assets/ladder.jpeg?raw=true)

{% label %}
Figure 2: All big things come from small beginnings
