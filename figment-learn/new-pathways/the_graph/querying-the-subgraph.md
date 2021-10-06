## Querying the subgraph

After deploying the subgraph, we need to wait a little bit in order to make it sync with the Ethereum mainnet.

We can follow the progression of the sync looking at the logged output by the running docker instance of our local graph node.

## Using the graphql endpoint

Having a fresh deployed subgraph make no sense if we don't query him. A graph node come up with a **graphql** endpoint, available at `http://localhost:8000/subgraphs/name/punks`.

## Time to verify your work

Now, it's time for you to verify if you have followed the instructions carefully, click on the button **Query the subgraph** to check the manifest file.

## Conclusion

Nice, you made it! You were able to add the correct information to the manifest to get the subgraph working from the [CryptoPunk](https://www.larvalabs.com/cryptopunks) smart contract.
