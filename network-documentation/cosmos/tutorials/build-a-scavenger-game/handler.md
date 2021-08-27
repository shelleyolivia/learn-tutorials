---
description: Learn how to build a scavenger game (continued)
---

# Handler

## Handler <a id="handler"></a>

In order for a **Message** to reach a **Keeper**, it has to go through a **Handler**. This is where logic can be applied to either allow or deny a `Message` to succeed. It's also where logic as to exactly how the state should change within the Keeper should take place. If you're familiar with [Model View Controller](https://en.wikipedia.org/wiki/Model%E2%80%93view%E2%80%93controller) \(MVC\) architecture, the `Keeper` is a bit like the **Model** and the `Handler` is a bit like the **Controller**. If you're familiar with [React/Redux](https://en.wikipedia.org/wiki/React_%28web_framework%29) or [Vue/Vuex](https://en.wikipedia.org/wiki/Vue.js) architecture, the `Keeper` is a bit like the **Reducer/Store** and the `Handler` is a bit like **Actions**.

Our Handler will go in `./x/scavenge/handler.go` and will follow the suggestions outlined in the boilerplate. We will create handler functions for each of our three `Message` types, `MsgCreateScavenge`, `MsgCommitSolution` and `MsgRevealSolution` until the file looks as follows:

```javascript
package scavenge

import (
    "crypto/sha256"
    "encoding/hex"
    "fmt"

    sdk "github.com/cosmos/cosmos-sdk/types"
    sdkerrors "github.com/cosmos/cosmos-sdk/types/errors"
    "github.com/cosmos/sdk-tutorials/scavenge/x/scavenge/types"
    "github.com/tendermint/tendermint/crypto"
)

// NewHandler creates an sdk.Handler for all the scavenge type messages
func NewHandler(k Keeper) sdk.Handler {
    return func(ctx sdk.Context, msg sdk.Msg) (*sdk.Result, error) {
        ctx = ctx.WithEventManager(sdk.NewEventManager())
        switch msg := msg.(type) {
        case MsgCreateScavenge:
            return handleMsgCreateScavenge(ctx, k, msg)
        case MsgCommitSolution:
            return handleMsgCommitSolution(ctx, k, msg)
        case MsgRevealSolution:
            return handleMsgRevealSolution(ctx, k, msg)
        default:
            return nil, sdkerrors.Wrap(sdkerrors.ErrUnknownRequest,
                fmt.Sprintf("unrecognized %s message type: %T", types.ModuleName, msg))
        }
    }
}

// handleMsgCreateScavenge creates a new scavenge and moves the reward into escrow
func handleMsgCreateScavenge(ctx sdk.Context, k Keeper, msg MsgCreateScavenge) (*sdk.Result, error) {
    var scavenge = types.Scavenge{
        Creator:      msg.Creator,
        Description:  msg.Description,
        SolutionHash: msg.SolutionHash,
        Reward:       msg.Reward,
    }
    _, err := k.GetScavenge(ctx, scavenge.SolutionHash)
    if err == nil {
        return nil, sdkerrors.Wrap(sdkerrors.ErrInvalidRequest, "Scavenge with that solution hash already exists")
    }
    moduleAcct := sdk.AccAddress(crypto.AddressHash([]byte(types.ModuleName)))
    sdkError := k.CoinKeeper.SendCoins(ctx, scavenge.Creator, moduleAcct, scavenge.Reward)
    if sdkError != nil {
        return nil, sdkError
    }
    k.SetScavenge(ctx, scavenge)
    ctx.EventManager().EmitEvent(
        sdk.NewEvent(
            sdk.EventTypeMessage,
            sdk.NewAttribute(sdk.AttributeKeyModule, types.AttributeValueCategory),
            sdk.NewAttribute(sdk.AttributeKeyAction, types.EventTypeCreateScavenge),
            sdk.NewAttribute(sdk.AttributeKeySender, msg.Creator.String()),
            sdk.NewAttribute(types.AttributeDescription, msg.Description),
            sdk.NewAttribute(types.AttributeSolutionHash, msg.SolutionHash),
            sdk.NewAttribute(types.AttributeReward, msg.Reward.String()),
        ),
    )
    return &sdk.Result{Events: ctx.EventManager().Events()}, nil
}

func handleMsgCommitSolution(ctx sdk.Context, k Keeper, msg MsgCommitSolution) (*sdk.Result, error) {
    var commit = types.Commit{
        Scavenger:             msg.Scavenger,
        SolutionHash:          msg.SolutionHash,
        SolutionScavengerHash: msg.SolutionScavengerHash,
    }
    _, err := k.GetCommit(ctx, commit.SolutionScavengerHash)
    // should produce an error when commit is not found
    if err == nil {
        return nil, sdkerrors.Wrap(sdkerrors.ErrInvalidRequest, "Commit with that hash already exists")
    }
    k.SetCommit(ctx, commit)
    ctx.EventManager().EmitEvent(
        sdk.NewEvent(
            sdk.EventTypeMessage,
            sdk.NewAttribute(sdk.AttributeKeyModule, types.AttributeValueCategory),
            sdk.NewAttribute(sdk.AttributeKeyAction, types.EventTypeCommitSolution),
            sdk.NewAttribute(sdk.AttributeKeySender, msg.Scavenger.String()),
            sdk.NewAttribute(types.AttributeSolutionHash, msg.SolutionHash),
            sdk.NewAttribute(types.AttributeSolutionScavengerHash, msg.SolutionScavengerHash),
        ),
    )
    return &sdk.Result{Events: ctx.EventManager().Events()}, nil
}

func handleMsgRevealSolution(ctx sdk.Context, k Keeper, msg MsgRevealSolution) (*sdk.Result, error) {
    var solutionScavengerBytes = []byte(msg.Solution + msg.Scavenger.String())
    var solutionScavengerHash = sha256.Sum256(solutionScavengerBytes)
    var solutionScavengerHashString = hex.EncodeToString(solutionScavengerHash[:])
    _, err := k.GetCommit(ctx, solutionScavengerHashString)
    if err != nil {
        return nil, sdkerrors.Wrap(err, "Commit with that hash doesn't exists")
    }

    var solutionHash = sha256.Sum256([]byte(msg.Solution))
    var solutionHashString = hex.EncodeToString(solutionHash[:])
    var scavenge types.Scavenge
    scavenge, err = k.GetScavenge(ctx, solutionHashString)
    if err != nil {
        return nil, sdkerrors.Wrap(err, "Scavenge with that solution hash doesn't exists")
    }
    if scavenge.Scavenger != nil {
        return nil, sdkerrors.Wrap(sdkerrors.ErrInvalidRequest, "Scavenge has already been solved")
    }
    scavenge.Scavenger = msg.Scavenger
    scavenge.Solution = msg.Solution

    moduleAcct := sdk.AccAddress(crypto.AddressHash([]byte(types.ModuleName)))
    sdkError := k.CoinKeeper.SendCoins(ctx, moduleAcct, scavenge.Scavenger, scavenge.Reward)
    if sdkError != nil {
        return nil, sdkError
    }
    k.SetScavenge(ctx, scavenge)
    ctx.EventManager().EmitEvent(
        sdk.NewEvent(
            sdk.EventTypeMessage,
            sdk.NewAttribute(sdk.AttributeKeyModule, types.AttributeValueCategory),
            sdk.NewAttribute(sdk.AttributeKeyAction, types.EventTypeSolveScavenge),
            sdk.NewAttribute(sdk.AttributeKeySender, msg.Scavenger.String()),
            sdk.NewAttribute(types.AttributeSolutionHash, solutionHashString),
            sdk.NewAttribute(types.AttributeDescription, scavenge.Description),
            sdk.NewAttribute(types.AttributeSolution, msg.Solution),
            sdk.NewAttribute(types.AttributeScavenger, scavenge.Scavenger.String()),
            sdk.NewAttribute(types.AttributeReward, scavenge.Reward.String()),
        ),
    )
    return &sdk.Result{Events: ctx.EventManager().Events()}, nil
}
```

### moduleAcct <a id="moduleacct"></a>

You might notice the use of `moduleAcct` within the `handleMsgCreateScavenge` and `handleMsgRevealSolution` handler functions. This account is not controlled by a public key pair, but is a reference to an account that is owned by this actual module. It is used to hold the bounty reward that is attached to a scavenge until that scavenge has been solved, at which point the bounty is paid to the account who solved the scavenge.

### Events <a id="events"></a>

At the end of each handler is an `EventManager` which will create logs within the transaction that reveals information about what occurred during the handling of this message. This is useful for client side software that wants to know exactly what happened as a result of this state transition. These Events use a series of pre-defined types that can be found in `./x/scavenge/types/events.go` and look as follows:

```javascript
package types

// scavenge module event types
const (
    EventTypeCreateScavenge = "CreateScavenge"
    EventTypeCommitSolution = "CommitSolution"
    EventTypeSolveScavenge  = "SolveScavenge"

    AttributeDescription           = "description"
    AttributeSolution              = "solution"
    AttributeSolutionHash          = "solutionHash"
    AttributeReward                = "reward"
    AttributeScavenger             = "scavenger"
    AttributeSolutionScavengerHash = "solutionScavengerHash"

    AttributeValueCategory = ModuleName
)
```

Now that we have all the necessary pieces for updating state \(`Message`, `Handler`, `Keeper`\) we might want to consider ways in which we can _query_ state. This is typically done via a REST endpoint and/or a CLI. Both of those clients interact with part of the app which queries state, called the `Querier`.

If you had any difficulties following this tutorial or simply want to discuss Cosmos tech with us you can [**join our community today**](https://discord.gg/fszyM7K)!

