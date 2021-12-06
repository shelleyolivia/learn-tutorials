# Wrapping up 🎁

Congratulations! You've built a Web 3-native blog that resembles the functionality of [Mirror](https://mirror.xyz/).

By leveraging a handful of protocols and tools like Polygon, Arweave, Hardhat, ethers.js, and ArDB, you were able to quickly build a full stack distributed application (dApp). In **Step 1** we leveraged a Next.js template to scaffold a simple application & we immediately got to work on turning it into a dApp. In **Step 2** we built functionality to connect to a user's MetaMask wallet.

We then gave our users the ability to publish entries by sending data to Arweave in **Step 3**. Not only that but since Arweave stores data forever (or pretty much forever), we gave our users the ability to publish censorship-resistant, immutable entries over which they have full control.

Expanding from write to read functionality, in **Step 4** we implemented the application's fetch function that retrieves data from Arweave by leveraging a transaction id and the address tag on our Arweave application deployment. This allowed us to display individual entries using the transaction id in the URI params.

In **Step 5**, we implemented similar functions to fetch the last 10 entries and display them on the dApp's home page. By clicking on these, users call the functionality implemented in Step 4 to display individual entries.

Not satisfied with a simple blog, and keen to explore the possibilities provided by leveraging the economics layer native to crypto rails, we established a smart contract to mint and manage NFTs for each entry in **Step 6**. The contract automatically mints an NFT owned by the author when the entry is published.

Finally, **Step 7** completed the NFT feature by giving users the ability to transfer their NFTs. An author can choose to transfer their entry's NFT by clicking a button and providing the recipient's address. Thereafter, the NFT owner can do the same with any other user.

In the process of building the blog, we learned how to connect a Web 3 wallet to effectively authenticate. We also expanded our understanding of Web 3 primitives like providers, signers and NFTs. 

Moreover, we learned how the frontend of the stack is similar to the Web 2 stack you're used to. We expanded our backend toolbox by interacting with Arweave for storage and Polygon for tracking NFT ownership. And we became familiar with ethers.js, the JavaScript library for connecting a user's wallet to the dApp and interacting with blockchain protocols.

Web 3 offers developers a new world in which to build the future of internet and re-imagine the possibilities. It expands the building blocks and in so doing enhances the ability to create new, innovative applications to solve worthwhile problems. 

The opportunities are endless and we wish you well on your journey!

_If you want to connect with an amazing community of developers, join us on [Discord](https://figment.io/devchat)._

![With ten miles behind me and ten thousand more to go…](https://raw.githubusercontent.com/figment-networks/learn-tutorials/mirror-tutorial/mirror/assets/hike.jpeg)
##### _Figure 11: With ten miles behind me and ten thousand more to go…_

# Additional Resources 📚

- [Arweave Overview](https://www.arweave.org/technology)
- [Arweave Lightpaper](https://www.arweave.org/files/arweave-lightpaper.pdf)
- [OpenZeppelin](https://openzeppelin.com/)
- [Hardhat](https://hardhat.org/)
- [Ethereum Virtual Machine](https://ethereum.org/en/developers/docs/evm/)
