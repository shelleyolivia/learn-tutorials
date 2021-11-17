---
description: Learn how to build a nameservice application (continued)
---

# Errors

## Errors <a id="errors"></a>

Start by navagating to the `errors.go` file within the types folder. Within your `errors.go` file, define errors that are custom to your module along with their codes.

```javascript
package types

import (
    sdkerrors "github.com/cosmos/cosmos-sdk/types/errors"
)

var (
    ErrNameDoesNotExist = sdkerrors.Register(ModuleName, 1, "name does not exist")
)
```

You must also add the corresponding method that'll be called at the time of error handling. For instance, let's say we try to delete a name that is not present in the store. In this case, an error should be thrown as the name does not exist.

We'll see later on in the tutorial where this method is called.

Now we move on to writing the Keeper for the module.

If you had any difficulties following this tutorial or simply want to discuss Cosmos tech with us you can [**join our community today**](https://discord.gg/fszyM7K)!

