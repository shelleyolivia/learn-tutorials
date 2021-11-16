---
description: Learn how to build a nameservice application (continued)
---

# Alias

## Alias <a id="alias"></a>

Start by navigating to the `./x/nameservice/alias.go` file. The main reason for having this file is to prevent import cycles. You can read more about import cycles in go here: [Golang import cycles](https://stackoverflow.com/questions/28256923/import-cycle-not-allowed)

First start by importing the "types" folder you have created.

### There are three kinds of types we will create in the alias.go file. <a id="there-are-three-kinds-of-types-we-will-create-in-the-alias-go-file"></a>

* A constant, this is where you will define immutable variables.
* A variable, which you will define to contain information such as your messages.
* A type, here you will define the types you have created in the types folder.

```javascript
package nameservice

import (
    "github.com/cosmos/sdk-tutorials/nameservice/x/nameservice/keeper"
    "github.com/cosmos/sdk-tutorials/nameservice/x/nameservice/types"
)

const (
    ModuleName   = types.ModuleName
    RouterKey    = types.RouterKey
    StoreKey     = types.StoreKey
    QuerierRoute = types.QuerierRoute
)

var (
    NewKeeper        = keeper.NewKeeper
    NewQuerier       = keeper.NewQuerier
    NewMsgBuyName    = types.NewMsgBuyName
    NewMsgSetName    = types.NewMsgSetName
    NewMsgDeleteName = types.NewMsgDeleteName
    NewWhois         = types.NewWhois
    ModuleCdc        = types.ModuleCdc
    RegisterCodec    = types.RegisterCodec
)

type (
    Keeper          = keeper.Keeper
    MsgSetName      = types.MsgSetName
    MsgBuyName      = types.MsgBuyName
    MsgDeleteName   = types.MsgDeleteName
    QueryResResolve = types.QueryResResolve
    QueryResNames   = types.QueryResNames
    Whois           = types.Whois
)
```

Now you have aliased your needed constants, variables, and types. We can move forward with the creation of the module.

Register your types in the Amino encoding format next.

If you had any difficulties following this tutorial or simply want to discuss Cosmos tech with us you can [**join our community today**](https://discord.gg/fszyM7K)!

