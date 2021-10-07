# What's The Graph?

You might have already read about The Graph and just want to roll up your sleeves and get started with coding. In which case feel free to skip to the "Setup the project" section below

If you want to read about The Graph we've curated a bunch of resources to help you get started:

- We wrote [this Twitter thread](https://twitter.com/sprngtheory/status/1425137466789486592) to explain The Graph without technical jargon. It's a great place to start
- Projects' own documentations are not always the best place to start but The Graph is an exception. They have very well written and approachable docs. [Give them a read](https://thegraph.com/docs/about/introduction)
- More of a visual learner? The rock start of blockchain Youtube - Finematics - have the perfect video for you. [Bing it here](https://www.youtube.com/watch?v=7gC7xJ_98r8)
- You're still here?? We have curated more links on [Figment Learn](https://learn.figment.io/protocols/thegraph). Knock yourself out!

# Setup the project

## 🐳 Docker

In this pathway we'll use Docker to easily run a local Graph node on your machine. [You can install Docker here](https://www.docker.com). Make sure it's installed and running before going forward.

## 🔑 Get an Alchemy API key

We'll need to connect to the Ethereum mainnet to be able to listen to new events happening on the network. Alchemy is a blockchain development platform that provides access to Ethereum through their hosted nodes. 

To get an Alchemy API key, create an account [here](https://www.alchemy.com/) and create a new application. From that application dashboard, click on **View Details** and then on **View Key**. Copy the HTTP endpoint URL.

<img width="600" alt="Screen Shot 2021-10-06 at 8 36 11 PM" src="https://user-images.githubusercontent.com/206753/136302457-a18a5f9c-82dc-40b1-838a-4bd655290433.png">


## 👣 Next Steps

Ok, enough of setup! Let's start getting our hands dirty with the first basic step: setting up a local Graph node to listen to the Ethereum network.
