---
description: Learn how to build a scavenger game (continued)
---

# Querier

## Querier <a id="querier"></a>

In order to query the data of our app we need to make it accessible using our `Querier`. This piece of the app works in tandem with the `Keeper` to access state and return it. The `Querier` is defined in `./x/scavenge/keeper/querier.go`. Our `scaffold` tool starts us out with some suggestions on how it should look, and similar to our `Handler` we want to handle different queried routes. You could make many different routes within the `Querier` for many different types of queries, but we will just make three:

* `listScavenges` will list all scavenges
* `getScavenge` will get a single scavenge by `solutionHash`
* `getCommit` will get a single commit by `solutionScavengerHash`

Combined into a switch statement and with each of the functions fleshed out it should look as follows:

```javascript
package keeper

import (
    abci "github.com/tendermint/tendermint/abci/types"

    "github.com/cosmos/cosmos-sdk/codec"
    sdk "github.com/cosmos/cosmos-sdk/types"
    sdkerrors "github.com/cosmos/cosmos-sdk/types/errors"
    "github.com/cosmos/sdk-tutorials/scavenge/x/scavenge/types"
)

// NewQuerier creates a new querier for scavenge clients.
func NewQuerier(k Keeper) sdk.Querier {
    return func(ctx sdk.Context, path []string, req abci.RequestQuery) ([]byte, error) {
        switch path[0] {
        case types.QueryListScavenges:
            return listScavenges(ctx, k)
        case types.QueryGetScavenge:
            return getScavenge(ctx, path[1:], k)
        case types.QueryCommit:
            return getCommit(ctx, path[1:], k)
        default:
            return nil, sdkerrors.Wrap(sdkerrors.ErrUnknownRequest, "unknown scavenge query endpoint")
        }
    }
}

// RemovePrefixFromHash removes the prefix from the key
func RemovePrefixFromHash(key []byte, prefix []byte) (hash []byte) {
    hash = key[len(prefix):]
    return hash
}

func listScavenges(ctx sdk.Context, k Keeper) ([]byte, error) {
    var scavengeList types.QueryResScavenges

    iterator := k.GetScavengesIterator(ctx)

    for ; iterator.Valid(); iterator.Next() {
        scavengeHash := RemovePrefixFromHash(iterator.Key(), []byte(types.ScavengePrefix))
        scavengeList = append(scavengeList, string(scavengeHash))
    }

    res, err := codec.MarshalJSONIndent(k.cdc, scavengeList)
    if err != nil {
        return res, sdkerrors.Wrap(sdkerrors.ErrJSONMarshal, err.Error())
    }

    return res, nil
}

func getScavenge(ctx sdk.Context, path []string, k Keeper) (res []byte, sdkError error) {
    solutionHash := path[0]
    scavenge, err := k.GetScavenge(ctx, solutionHash)
    if err != nil {
        return nil, err
    }

    res, err = codec.MarshalJSONIndent(k.cdc, scavenge)
    if err != nil {
        return nil, sdkerrors.Wrap(sdkerrors.ErrJSONMarshal, err.Error())
    }

    return res, nil
}

func getCommit(ctx sdk.Context, path []string, k Keeper) (res []byte, sdkError error) {
    solutionScavengerHash := path[0]
    commit, err := k.GetCommit(ctx, solutionScavengerHash)
    if err != nil {
        return nil, err
    }
    res, err = codec.MarshalJSONIndent(k.cdc, commit)
    if err != nil {
        return nil, sdkerrors.Wrap(sdkerrors.ErrJSONMarshal, err.Error())
    }
    return res, nil
}
```

### Types <a id="types"></a>

You may notice that we use three different imported types on our initial switch statement. These are defined within our `./x/scavenge/types/querier.go` file as simple strings. That file should look like the following:

```javascript
package types

import "strings"

// query endpoints supported by the nameservice Querier
const (
    QueryListScavenges = "list"
    QueryGetScavenge   = "get"
    QueryCommit        = "commit"
)

// // QueryResResolve Queries Result Payload for a resolve query
// type QueryResResolve struct {
//     Value string `json:"value"`
// }

// // implement fmt.Stringer
// func (r QueryResResolve) String() string {
//     return r.Value
// }

// QueryResScavenges Queries Result Payload for a names query
type QueryResScavenges []string

// implement fmt.Stringer
func (n QueryResScavenges) String() string {
    return strings.Join(n[:], "\n")
}
```

Our queries are rather simple since we've already outfitted our `Keeper` with all the necessary functions to access state. You can see the iterator being used here as well.

Now that we have all of the basic actions of our module created, we want to make them accessible. We can do this with a CLI client and a REST client. For this tutorial we will just be creating a CLI client. If you are interested in what goes into making a REST client, check out the [Nameservice Tutorial](https://tutorials.cosmos.network/nameservice/tutorial/00-intro.html).

Let's take a look at what goes into making a CLI.

If you had any difficulties following this tutorial or simply want to discuss Cosmos tech with us you can [**join our community today**](https://discord.gg/fszyM7K)!

