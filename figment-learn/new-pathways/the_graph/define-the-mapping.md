## Mapping event to feed entity

Remember in the `Hacking the manifest` step we have defined a handler next to an event. This handler will the maestro of our subgraph. This piece of code will orchestrate event with entity with the aim to give what we are asking for? A well sounded concerto or data!

## Define entities

Now, we're going to create the `handleAssign` function.

- Open `src/mapping.ts`
- Erase the content

First we need a bunch of import and the prototype of the function

```typescript
import { BigInt } from "@graphprotocol/graph-ts";

import { Assign as AssignEvent } from "../generated/punks/punks";
import { Account, Punk } from "../generated/schema";

// Handler for event Assign(address indexed to, uint256 punkIndex);
export function handleAssign(event: AssignEvent): void {
  // fill here
}
```

`Account` and `Punk` imported objects are the one we've just defined before, and `AssignEvent` is referencing the definition of event we gave on `subgraph.yaml`.

```typescript
let account = Account.load(event.params.to.toHexString());
```

To create the Account entity, we first need to test if the entity is already existing.

```typescript
if (account == null) {
  account = new Account(event.params.to.toHexString());
  account.id = event.params.to.toHexString();
  account.numberOfPunksOwned = BigInt.fromI32(1);
}
```

if not we create a new one filling all the field

otherwise, we only need to increment the number of owned punk

```typescript
else {
  account.numberOfPunksOwned =
    account.numberOfPunksOwned.plus(BigInt.fromI32(1));
}
```

At last and for both case we update the timestamp

```typescript
account.LastMvtAt = event.block.timestamp;
account.save();
```

The creation of Punk entity follow the same logic.

At the end your `src/mapping.ts` should look like this.

```typescript
import { BigInt } from "@graphprotocol/graph-ts";

import { Assign as AssignEvent } from "../generated/punks/punks";
import { Account, Punk } from "../generated/schema";

export function handleAssign(event: AssignEvent): void {
  let account = Account.load(event.params.to.toHexString());
  if (account == null) {
    account = new Account(event.params.to.toHexString());
    account.id = event.params.to.toHexString();
    account.numberOfPunksOwned = BigInt.fromI32(1);
  } else {
    account.numberOfPunksOwned =
      account.numberOfPunksOwned.plus(BigInt.fromI32(1));
  }
  account.LastMvtAt = event.block.timestamp;
  account.save();

  let punk = Punk.load(event.params.punkIndex.toHexString());
  if (punk == null) {
    punk = new Punk(event.params.punkIndex.toHexString());
    punk.id = event.params.punkIndex.toHexString();
    punk.owner = event.params.to.toHexString();
  } else {
    punk.owner = event.params.to.toHexString();
  }
  punk.LastAssignAt = event.block.timestamp;
  punk.save();
}
```

## Make sure it works

Last but not least, run the following command to create the subgraph and deploy it to our local graph node.

```bash
yarn create-local
yarn deploy-local
```

Now, it's time for you to verify if you have followed the instructions carefully, click on the button **Check subgraph deployment** to check that your deployment is well done.
