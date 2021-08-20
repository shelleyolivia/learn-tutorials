---
description: Learn how to build a nameservice application (continued)
---

# Codec File

## Codec File <a id="codec-file"></a>

To [register your types with Amino](https://github.com/tendermint/go-amino#registering-types) so that they can be encoded/decoded, there is a bit of code that needs to be placed in `./x/nameservice/types/codec.go`. Any interface you create and any struct that implements an interface needs to be declared in the `RegisterCodec` function. In this module the three `Msg` implementations \(`SetName`, `BuyName` and `DeleteName`\) need to be registered, but your `Whois` query return type does not. In addition, we define a module specific codec for use later.

```javascript
package types

import (
    "github.com/cosmos/cosmos-sdk/codec"
)

// ModuleCdc is the codec for the module
var ModuleCdc = codec.New()

func init() {
    RegisterCodec(ModuleCdc)
}

// RegisterCodec registers concrete types on the Amino codec
func RegisterCodec(cdc *codec.Codec) {
    cdc.RegisterConcrete(MsgSetName{}, "nameservice/SetName", nil)
    cdc.RegisterConcrete(MsgBuyName{}, "nameservice/BuyName", nil)
    cdc.RegisterConcrete(MsgDeleteName{}, "nameservice/DeleteName", nil)
}
```

### Next you need to define CLI interactions with your module. <a id="next-you-need-to-define-cli-interactions-with-your-module"></a>

If you had any difficulties following this tutorial or simply want to discuss Cosmos tech with us you can [**join our community today**](https://discord.gg/fszyM7K)!

