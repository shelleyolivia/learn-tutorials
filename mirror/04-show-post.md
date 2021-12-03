# Step 4: Fetching a Post

The previous step left us hanging since we still having seen our post. Let's keep progressing by building an endpoint to retrieve a post.

# Implementation 🧩

We first need to write another endpoint that will fetch a certain post for our application. We've structured the endpoint at `api/arweave/[transactionHash].ts` but we still need to write the core functionality.

Thinking about this endpoint, we should be able to pass in an identifier for a transaction and get the transaction data in response. We can do that by leveraging the `getData` method for Arweave transactions and we can parse the JSON it returns:

```javascript
const txDataResp = (await arweave.transactions.getData(
  transactionHash as string,
  {
    decode: true,
    string: true,
  },
)) as string;
const txData = JSON.parse(txDataResp);
```

We should then check whether the transaction has been confirmed before sending a response to the client.

{% sidenote title="Box 4.1: What does it mean for Arweave transactions to confirm?" %}
As with any distributed system, there is a trade off between decentralized and speed. In order for the data to be stored, the Arweave protocol has to achieve consensus. You can read more about Arweave's consensus mechanism [here](https://arweave.medium.com/what-is-arweave-explain-like-im-five-425362144eb5) but suffice it to say that the lag most databases experience when writing data is exacerbated in distributed storage. That lag is amplified by the fact that data is not only being written, but agreed upon across the protocol. This takes anywhere from 5-10 minutes at the time of writing.

It will be clear why in Step 6 when we associate NFTs with each post but as a preview, we wouldn't want users claiming NFTs before a post is actually created on Arweave.

We should also note that you can choose how many confirmations a post should receive before considering it confirmed. For this template, we have set them in `constants.ts` through the `MIN_NUMBER_OF_CONFIRMATIONS` environment variable.

```javascript
const txStatusResp = await arweave.transactions.getStatus(
  transactionHash as string,
);

const txStatus =
  txStatusResp.status === 200 &&
  txStatusResp.confirmed &&
  txStatusResp.confirmed.number_of_confirmations >=
    MIN_NUMBER_OF_CONFIRMATIONS
    ? TransactionStatusE.CONFIRMED
    : TransactionStatusE.NOT_CONFIRMED;
```

Once we've confirmed that our data has been written to Arweave, we can get the transaction by using its identifier from the query param `transactionHash`. 

```javascript
const tx = await arweave.transactions.get(transactionHash as string);
```

We should also get the tags, so we have access to the user's address:

```javascript
const tags = {} as PostTagsT;
(tx.get('tags') as any).forEach((tag) => {
  const key = tag.get('name', {decode: true, string: true});
  tags[key] = tag.get('value', {decode: true, string: true});
});
```

And we should get the block, so we have access to the post's timestamp:

```javascript
const block = txStatusResp.confirmed
  ? await arweave.blocks.get(txStatusResp.confirmed.block_indep_hash)
  : null;
```

With that we have all the building blocks the client will need to display a post - post data, author, and timestamp. We can build the response and return it:

```javascript
res.status(200).json({
  id: transactionHash as string,
  data: txData,
  status: txStatus,
  timestamp: block?.timestamp,
  tags,
});
```

"But we still haven't displayed a post!", you yell. Yes, yes - we absolutely should do that. We'll get to that in the Step 5.

![Figure 6: We're getting very close to some magic.](https://raw.githubusercontent.com/figment-networks/learn-tutorials/mirror-tutorial/mirror/assets/map.jpeg?raw=true)

{% label %}
Figure 6: We're getting very close to some magic.

##### _Listing 4.1: Code for fetching a post_
```javascript
const {transactionHash} = req.query;
const txDataResp = (await arweave.transactions.getData(
  transactionHash as string,
  {
    decode: true,
    string: true,
  },
)) as string;
const txData = JSON.parse(txDataResp);

const txStatusResp = await arweave.transactions.getStatus(
  transactionHash as string,
);
const txStatus =
  txStatusResp.status === 200 &&
  txStatusResp.confirmed &&
  txStatusResp.confirmed.number_of_confirmations >=
    MIN_NUMBER_OF_CONFIRMATIONS
    ? TransactionStatusE.CONFIRMED
    : TransactionStatusE.NOT_CONFIRMED;

if (txStatus === TransactionStatusE.CONFIRMED) {
  const tx = await arweave.transactions.get(transactionHash as string);

  const tags = {} as PostTagsT;
  (tx.get('tags') as any).forEach((tag) => {
    const key = tag.get('name', {decode: true, string: true});
    tags[key] = tag.get('value', {decode: true, string: true});
  });

  const block = txStatusResp.confirmed
    ? await arweave.blocks.get(txStatusResp.confirmed.block_indep_hash)
    : null;

  res.status(200).json({
    id: transactionHash as string,
    data: txData,
    status: txStatus,
    timestamp: block?.timestamp,
    tags,
  });
} else {
  throw new Error('Transaction not confirmed');
}
```

# Challenge 🏋️

Navigate to `api/arweave/[transactionHash].ts` in your editor and follow the steps included as comments to finish writing the endpoint to retrieve posts. We include a description along with a link to the documentation you need to review in order to implement each line. The relevant code block is also included in [Listing 4.2](#listing-42-instructions-for-fetching-a-post) below.

##### _Listing 4.2: Instructions for fetching a post_
```javascript
const {transactionHash} = req.query;
  // Get Arweave transaction data. Documentation can be found here: https://github.com/ArweaveTeam/arweave-js

  // Get Arweave transaction status. Documentation can be found here: https://github.com/ArweaveTeam/arweave-js
  const txStatus = undefined;
if (txStatus === TransactionStatusE.CONFIRMED) {
  // Get Arweave transaction. Documentation can be found here: https://github.com/ArweaveTeam/arweave-js

  // Get Arweave transaction tags. Documentation can be found here: https://github.com/ArweaveTeam/arweave-js
  
  // Get Arweave transaction block in order to retrieve timestamp. Documentation can be found here: https://github.com/ArweaveTeam/arweave-js

  // Return JSON response in form:
  // res.status(200).json({
  //   id: transactionHash as string,
  //   data: txData,
  //   status: txStatus,
  //   timestamp: block?.timestamp,
  //   tags,
  // });
} else {
  throw new Error('Transaction not confirmed');
}
```
