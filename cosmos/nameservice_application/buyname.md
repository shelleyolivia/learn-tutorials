---
description: Learn how to build a nameservice application (continued)
---

# BuyName

## MsgBuyName <a id="msgbuyname"></a>

Now it is time to define the `Msg` for buying names and add it to the `./x/nameservice/types/msgs.go` file. This code is very similar to `SetName`:

```javascript
// MsgBuyName defines the BuyName message
type MsgBuyName struct {
    Name  string         `json:"name"`
    Bid   sdk.Coins      `json:"bid"`
    Buyer sdk.AccAddress `json:"buyer"`
}

// NewMsgBuyName is the constructor function for MsgBuyName
func NewMsgBuyName(name string, bid sdk.Coins, buyer sdk.AccAddress) MsgBuyName {
    return MsgBuyName{
        Name:  name,
        Bid:   bid,
        Buyer: buyer,
    }
}

// Route should return the name of the module
func (msg MsgBuyName) Route() string { return RouterKey }

// Type should return the action
func (msg MsgBuyName) Type() string { return "buy_name" }

// ValidateBasic runs stateless checks on the message
func (msg MsgBuyName) ValidateBasic() error {
    if msg.Buyer.Empty() {
        return sdkerrors.Wrap(sdkerrors.ErrInvalidAddress, msg.Buyer.String())
    }
    if len(msg.Name) == 0 {
        return sdkerrors.Wrap(sdkerrors.ErrUnknownRequest, "Name cannot be empty")
    }
    if !msg.Bid.IsAllPositive() {
        return sdkerrors.ErrInsufficientFunds
    }
    return nil
}

// GetSignBytes encodes the message for signing
func (msg MsgBuyName) GetSignBytes() []byte {
    return sdk.MustSortJSON(ModuleCdc.MustMarshalJSON(msg))
}

// GetSigners defines whose signature is required
func (msg MsgBuyName) GetSigners() []sdk.AccAddress {
    return []sdk.AccAddress{msg.Buyer}
}
```

Next, in the `./x/nameservice/handler.go` file, add the `MsgBuyName` handler to the module router:

```javascript
// NewHandler returns a handler for "nameservice" type messages.
func NewHandler(keeper Keeper) sdk.Handler {
    return func(ctx sdk.Context, msg sdk.Msg) (*sdk.Result, error) {
        switch msg := msg.(type) {
        case MsgSetName:
            return handleMsgSetName(ctx, keeper, msg)
        case MsgBuyName:
            return handleMsgBuyName(ctx, keeper, msg)
        default:
            return nil, sdkerrors.Wrap(sdkerrors.ErrUnknownRequest, fmt.Sprintf("Unrecognized nameservice Msg type: %v", msg.Type()))
        }
    }
}
```

Finally, define the `BuyName` `handler` function which performs the state transitions triggered by the message. Keep in mind that at this point the message has had its `ValidateBasic` function run so there has been some input verification. However, `ValidateBasic` cannot query application state. Validation logic that is dependent on network state \(e.g. account balances\) should be performed in the `handler` function.

```javascript
// Handle a message to buy name
func handleMsgBuyName(ctx sdk.Context, keeper Keeper, msg MsgBuyName) (*sdk.Result, error) {
    // Checks if the the bid price is greater than the price paid by the current owner
    if keeper.GetPrice(ctx, msg.Name).IsAllGT(msg.Bid) {
        return nil, sdkerrors.Wrap(sdkerrors.ErrInsufficientFunds, "Bid not high enough") // If not, throw an error
    }
    if keeper.HasOwner(ctx, msg.Name) {
        err := keeper.CoinKeeper.SendCoins(ctx, msg.Buyer, keeper.GetOwner(ctx, msg.Name), msg.Bid)
        if err != nil {
            return nil, err
        }
    } else {
        _, err := keeper.CoinKeeper.SubtractCoins(ctx, msg.Buyer, msg.Bid) // If so, deduct the Bid amount from the sender
        if err != nil {
            return nil, err
        }
    }
    keeper.SetOwner(ctx, msg.Name, msg.Buyer)
    keeper.SetPrice(ctx, msg.Name, msg.Bid)
    return &sdk.Result{}, nil
}
```

First check to make sure that the bid is higher than the current price. Then, check to see whether the name already has an owner. If it does, the former owner will receive the money from the `Buyer`.

If there is no owner, your `nameservice` module "burns" \(i.e. sends to an unrecoverable address\) the coins from the `Buyer`.

If either `SubtractCoins` or `SendCoins` returns a non-nil error, the handler throws an error, reverting the state transition. Otherwise, using the getters and setters defined on the `Keeper` earlier, the handler sets the buyer to the new owner and sets the new price to be the current bid.

> _NOTE_: This handler uses functions from the `coinKeeper` to perform currency operations. If your application is performing currency operations you may want to take a look at the [godocs for this module](https://godoc.org/github.com/cosmos/cosmos-sdk/x/bank#BaseKeeper) to see what functions it exposes.

### Great, now owners can `BuyName`s! But what if they don't want the name any longer? Your module needs a way for users to delete names! Let us define the `DeleteName` message. <a id="great-now-owners-can-buynames-but-what-if-they-don-t-want-the-name-any-longer-your-module-needs-a-way-for-users-to-delete-names-let-us-define-define-the-deletename-message"></a>

If you had any difficulties following this tutorial or simply want to discuss Cosmos tech with us you can [**join our community today**](https://discord.gg/fszyM7K)!

