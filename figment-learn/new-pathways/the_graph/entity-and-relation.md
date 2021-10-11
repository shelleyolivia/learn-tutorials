## 👤 Entity and relation

To capture and process information from a blockchain, we must:

- Define which information we're looking for
- Know what this information is made of
- Define the shape of the processed information when we are finished with it

The right place to do so is within the `schema.graphql`.

## 👥 Define entities

To define our entities, we must create the

```graphql
type Account @entity {
  id: ID!
  punksBought: [Punk!] @derivedFrom(field: "currentOwner")
  punksSell: [Punk!] @derivedFrom(field: "previousOwner")
  numberOfPunkBought: BigInt!
  numberOfPunkSell: BigInt!
  LastBought: BigInt!
  LastSell: BigInt!
}

type Punk @entity {
  id: ID!
  tokenId: BigInt!
  currentOwner: Account!
  previousOwner: Account
  lastValue: BigInt!
  tradeDate: BigInt!
}
```
# 👉 The solution

Replace the existing contents of `schema.graphql` with the following code snippet:

```graphql
// solution
type Account @entity {
  id: ID!
  punksBought: [Punk!] @derivedFrom(field: "currentOwner")
  punksSell: [Punk!] @derivedFrom(field: "previousOwner")
  numberOfPunkBought: BigInt!
  numberOfPunkSell: BigInt!
  LastBought: BigInt!
  LastSell: BigInt!
}

type Punk @entity {
  id: ID!
  tokenId: BigInt!
  currentOwner: Account!
  previousOwner: Account
  lastValue: BigInt!
  tradeDate: BigInt!
}
```

We have now matched the changes to the manifest, by adding the entities `Account` and `Punk`.

## Specify relation

In the above code snippet, there are two points worth mentioning:

- For the purpose of indexing, entities _must have_ an `ID` field to uniquely identify them.
- As an `Account` can be the **owner** of multiple `Punk` we must explicitly define the `1:n` relation on the `Account`'s **punksOwned** attribute using `[Punk!] @deriveFrom(field: "owner")` directive:

```text
                               |      ------
                               | --- | Punk |
                               |      ------
                               ...
     ---------                 |      ------
    | Account |  1 : ----- n : | --- | Punk |
     ---------                 |      ------
                               ...
                               |      ------
                               | --- | Punk |
                               |      ------
```

## ✅ Make sure it works

Last but not least, run the following command to generate boilerplate code from our **entities**:

```text
cd punks
yarn codegen
```

![](../../../.gitbook/assets/pathways/the_graph/yarn-codegen.gif)

Now it's time for you to verify that you have followed the instructions carefully. Click on the **Check for expected entites** button on the right to see that your entities are properly defined.
