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
  punksBought: // Specify relation using the next section
  numberOfPunkBought: BigInt!
}

type Punk @entity {
  // How can you index the Punk entity?
  index: BigInt!
  owner: Account!
  value: BigInt!
  date: BigInt!
}
```

## Specify relation

In the above code snippet, there are two points worth mentioning:

- For the purpose of indexing, entities _must have_ an `ID` field to uniquely identify them.
- As an `Account` can be the **owner** of multiple `Punk` we must explicitly define the `1:n` relation on the `Account`'s **punksBought** attribute using `[Punk!] @deriveFrom(field: "owner")` directive:

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

# 👉 The solution

Replace the existing contents of `schema.graphql` with the following code snippet:

```graphql
// solution
type Account @entity {
  id: ID!
  punksBought: [Punk!] @derivedFrom(field: "owner")
  numberOfPunkBought: BigInt!
}

type Punk @entity {
  id: ID!
  index: BigInt!
  owner: Account!
  value: BigInt!
  date: BigInt!
}
```

We have now matched the changes to the manifest, by adding the entities `Account` and `Punk`.

## ✅ Make sure it works

Last but not least, run the following command to generate boilerplate code from our **entities**:

```text
cd punks
yarn codegen
```

About `yarn codegen`

- This command create boilerplate code under `generated` folder
- This boilerplate code define as much **typescript** class there is defined `entities`. You can have a taste of it looking at `generated/schema.ts`
- We will re-use it in the next step to define the mapping between our `entities` and the smart-contract `event`

![](../../../.gitbook/assets/pathways/the_graph/yarn-codegen.gif)

Now it's time for you to verify that you have followed the instructions carefully. Click on the **Check for expected entities** button on the right to see that your entities are properly defined.
