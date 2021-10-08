## Mapping event to feed entity

Remember in the `Hacking the manifest` step we have defined a handler next to an event. This handler will the maestro of our subgraph. This piece of code will orchestrate event with entity with the aim to give what we are asking for? A well sounded concerto or data!

## Define entities

Now, we're going to create the `handleAssign` function.

- Open `src/mapping.ts`
- Erase the content

First we need a bunch of import and the prototype of the function

```typescript
import { BigInt } from "@graphprotocol/graph-ts";

import { PunkBought as PunkBoughtEvent } from "../generated/punks/punks";
import { Account, Punk } from "../generated/schema";

export function handlePunkBought(event: PunkBoughtEvent): void {
  // fill here
}
```

`Account` and `Punk` imported objects are the one we've just defined before, and `AssignEvent` is referencing the definition of event we gave on `subgraph.yaml`.

```typescript
let buyerAccount = Account.load(event.params.toAddress.toHexString());
```

To create the Account entity, we first need to test if the entity is already existing.

```typescript
if (buyerAccount == null) {
  buyerAccount = new Account(event.params.toAddress.toHexString());
  buyerAccount.id = event.params.toAddress.toHexString();
  buyerAccount.numberOfPunkBought = BigInt.fromI32(1);
  buyerAccount.numberOfPunkSell = BigInt.fromI32(0);
  buyerAccount.LastSell = BigInt.fromI32(0);
}
```

if not we create a new one filling all the field

otherwise, we only need to increment the numberOfPunkBought

```typescript
buyerAccount.numberOfPunkBought = buyerAccount.numberOfPunkBought.plus(
  BigInt.fromI32(1)
);
```

At last and for both case we update the last field and save

```typescript
const timestamp = event.block.timestamp;
sellerAccount.LastSell = timestamp;
sellerAccount.save();
```

The creation of Punk entity follow the same logic.

At the end your `src/mapping.ts` should look like this.

```typescript
import { BigInt } from "@graphprotocol/graph-ts";

import { PunkBought as PunkBoughtEvent } from "../generated/punks/punks";
import { Account, Punk } from "../generated/schema";

export function handlePunkBought(event: PunkBoughtEvent): void {
  let buyerAccount = Account.load(event.params.toAddress.toHexString());
  if (buyerAccount == null) {
    buyerAccount = new Account(event.params.toAddress.toHexString());
    buyerAccount.id = event.params.toAddress.toHexString();
    buyerAccount.numberOfPunkBought = BigInt.fromI32(1);
    buyerAccount.numberOfPunkSell = BigInt.fromI32(0);
    buyerAccount.LastSell = BigInt.fromI32(0);
  } else {
    buyerAccount.numberOfPunkBought = buyerAccount.numberOfPunkBought.plus(
      BigInt.fromI32(1)
    );
  }
  buyerAccount.LastBought = event.block.timestamp;
  buyerAccount.save();

  let sellerAccount = Account.load(event.params.fromAddress.toHexString());
  if (sellerAccount == null) {
    sellerAccount = new Account(event.params.fromAddress.toHexString());
    sellerAccount.id = event.params.fromAddress.toHexString();
    sellerAccount.numberOfPunkBought = BigInt.fromI32(0);
    sellerAccount.numberOfPunkSell = BigInt.fromI32(1);
    sellerAccount.LastSell = BigInt.fromI32(0);
  } else {
    sellerAccount.numberOfPunkSell = sellerAccount.numberOfPunkSell.plus(
      BigInt.fromI32(1)
    );
  }
  const timestamp = event.block.timestamp;
  sellerAccount.LastSell = timestamp;
  sellerAccount.save();

  let punk = Punk.load(event.params.punkIndex.toHexString());
  if (punk == null) {
    punk = new Punk(event.params.punkIndex.toHexString());
    punk.id = event.params.punkIndex.toHexString();
    punk.tokenId = event.params.punkIndex;
  }
  punk.currentOwner = event.params.toAddress.toHexString();
  punk.previousOwner = event.params.fromAddress.toHexString();
  punk.lastValue = event.params.value;
  punk.tradeDate = timestamp;

  punk.save();
}
```

## Make sure it works

Last but not least, run the following command to create the subgraph and deploy it to our local graph node.

```bash
yarn create-local
yarn deploy-local
```

![](../../../.gitbook/assets/pathways/the_graph/create-deploy-local.gif)


Now, it's time for you to verify if you have followed the instructions carefully, click on the button **Check subgraph deployment** to check that your deployment is well done.
