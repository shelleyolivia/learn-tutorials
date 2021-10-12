## 🧐 Querying the subgraph

After deploying the subgraph, we need to wait a little in order to make it sync with the Ethereum mainnet.

We can follow the progression of the sync looking at the logged output by the running docker instance of our local graph node.

{% hint style="info" %}
Before, running the query you'll need to check you have a [metamask](https://metamask.io/) extension installed with your browser and you're connected to the mainnet of Ethereum. The reason for this requirement it's, we'll decorate the date return by the **GraphQL** query with data coming from the [CryptoPunks Data](https://etherscan.io/address/0x16F5A35647D6F03D5D3da7b35409D65ba03aF3B2#readContract)
{% endhint %}

## 👨‍💻 Using the GraphQL endpoint

Having a fresh deployed subgraph make no sense if we don't query him. A graph node come up with a **GraphQL** endpoint, available at `http://localhost:8000/subgraphs/name/punks`.

We're going to display the ten most expensive trades done on the CryptoPunk contract. As a helper please consider this example query. The file to modify to complete the query is `components/protocols/the_graph/graphql/index.ts`

```graphql
query {
  punks(first: 1) {
    id
    index
    owner {
      id
    }
  }
}
```

Some hints to help you:

- Seem like `value` and `date` are missing.
- We need the `first` 10, `orderBy` value and the `orderDirection` desc.
- To exercise, use the **GraphQL** playground available [here](http://localhost:8000/subgraphs/name/punks)

# 👉 The solution

In `components/protocols/the_graph/graphql/index.ts` replace the **GraphQL** query with:

```graphql
// solution
query {
  punks(first: 10, orderBy: value, orderDirection: desc) {
    id
    index
    owner {
      id
    }
    value
    date
  }
}
```

## 🥳 Enjoy the result of your work

Now, it's time to enjoy the result of your work! Click on the **Query the subgraph** button on the right, and say hello to the Punks.

## 🏁 Conclusion

Nice, you made it! You were able to complete the **THE GRAPH** pathway. If you want to learn more check ours [tutorials](https://www.learn.figment.io).
