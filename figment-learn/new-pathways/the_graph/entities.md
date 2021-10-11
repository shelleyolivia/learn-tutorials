## 👤 Entity and relation

We just tweaked the manifest to declare what information we were looking for. We declared two entities called `Punk` (for the actual CryptoPunk NFT) and `Account` (for the owner of the NFT). Now we need to implement those entities: for each of them, define what attributes they have and what are those attribute types.

This is analogous to the process of defining the Models in an MVC framework.

Entities will be defined in the `schema.graphql` file.

![Entities](https://user-images.githubusercontent.com/206753/136861292-2c178573-5dc8-48c5-92c3-4482a8963887.png)

## 🧑🏼‍💻 Your turn! Define the Punk and Account entities

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
