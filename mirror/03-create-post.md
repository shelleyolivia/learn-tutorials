# Step 3: Writing a Post

Traditional blogs use databases to persist post data. An aurthor might write some content and publish it, which triggers a create action on the server that writes the data to the database. The database would have schema that define the data structure. For instance, we might have a posts table with columns for post id, content, and author id.

The blockchain however is a bit different because it's decentralized. Rather than writing to a specific database, we'll be leveraging a distributed system that can't be controlled by a central authority.
  
## Arweave - Sustainable Onchain Storage

Arweave is a decentralized network that allows users with extra storage space to provide it as a storage node to the network in exchange for economic incentives. Users looking to store data in a decentralized, secure, censorship-resistant way, can leverage the network to do so. Effectively, Arweave is the storage part of the stack for dApps.

Arwaeve uses a technology called "blockweaves" to achieve this. Similar to a blockchain, which connects transactions in a main chain across blocks, blockweaves are connected blocks that contain data. One of the main differences is that rather than connect blocks as a singly linked list, blockweaves are more like graphs.

This system requires a robust and sustainable economic structure to ensure storage providers are incentivized to replicate information forever at reasonable costs to the user. Basically, the user pays a small fee upfront to permanently store data. That fee is treated as principal that generates interest over time for storage providers.

The architecture to achieve all this is beyond the scope of this tutorial, but if you're interested in going deeper, we recommend checking out the [Arweave yellow paper](https://www.arweave.org/yellow-paper.pdf).

# Implementation 🧩

## Steps
* Setup Arweave wallet
* Update API route in `/api/arweave/post`in order to create, sign and post transaction. Make sure that relevant tags are added
* Update handleSubmit in CreatePostForm with:
  * Request to above API endpoint. (Submit JSON metadata composed of post title and body to Arweave)

##### _Listing 3.1: Code for creating a post_
>>>>>>> Insert code here

# Challenge 🏋️

Navigate to `[ ]` in your editor and follow the steps included as comments to finish writing the `[ ]` function. We include a description along with a link to the documentation you need to review in order to implement each line. The relevant code block is also included in [Listing 3.2](#listing-32-instructions-for-creating-a-post) below.

##### _Listing 3.2: Instructions for creating a post_
