---
description: Learn how to build a nameservice application (continued)
---

# Expected Keepers

## Expected Keepers <a id="expected-keepers"></a>

Next create a file within the `/types` directory called `expected_keepers.go`. In this file we will be defining what we expect other modules to have.

For example in the nameservice module we will be using the bank module to facilitate transfers between two parties. To make this happen we will rely on two functions from the bank module.

* `SubtractCoins(ctx sdk.Context, addr sdk.AccAddress, amt sdk.Coins) (sdk.Coins, error)`
* `SendCoins(ctx sdk.Context, fromAddr sdk.AccAddress, toAddr sdk.AccAddress, amt sdk.Coins) error`

you can see below how the file is structured below:

```javascript
package types

import (
    sdk "github.com/cosmos/cosmos-sdk/types"
)

// When a module wishes to interact with an otehr module it is good practice to define what it will use
// as an interface so the module can not use things that are not permitted.
type BankKeeper interface {
    SubtractCoins(ctx sdk.Context, addr sdk.AccAddress, amt sdk.Coins) (sdk.Coins, error)
    SendCoins(ctx sdk.Context, fromAddr sdk.AccAddress, toAddr sdk.AccAddress, amt sdk.Coins) error
}
```

If you had any difficulties following this tutorial or simply want to discuss Cosmos tech with us you can [**join our community today**](https://discord.gg/fszyM7K)!

