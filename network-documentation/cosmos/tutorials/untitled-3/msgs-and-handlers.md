---
description: Learn how to build a nameservice application (continued)
---

# Msgs and Handlers

## Msgs and Handlers <a id="msgs-and-handlers"></a>

Now that you have the `Keeper` setup, it is time to build the `Msgs` and `Handlers` that actually allow users to buy names and set values for them.

### `Msgs` <a id="msgs"></a>

`Msgs` trigger state transitions. `Msgs` are wrapped in [`Txs`](https://github.com/cosmos/cosmos-sdk/blob/master/types/tx_msg.go#L34-L41) that clients submit to the network. The Cosmos SDK wraps and unwraps `Msgs` from `Txs`, which means, as an app developer, you only have to define `Msgs`. `Msgs` must satisfy the following interface \(we'll implement all of these in the next section\):

```javascript
// Transactions messages must fulfill the Msg
type Msg interface {
    // Return the message type.
    // Must be alphanumeric or empty.
    Type() string

    // Returns a human-readable string for the message, intended for utilization
    // within tags
    Route() string

    // ValidateBasic does a simple validation check that
    // doesn't require access to any other information.
    ValidateBasic() Error

    // Get the canonical byte representation of the Msg.
    GetSignBytes() []byte

    // Signers returns the addrs of signers that must sign.
    // CONTRACT: All signatures must be present to be valid.
    // CONTRACT: Returns addrs in some deterministic order.
    GetSigners() []AccAddress
}
```

Copy// Transactions messages must fulfill the Msg type Msg interface { // Return the message type. // Must be alphanumeric or empty. Type\(\) string // Returns a human-readable string for the message, intended for utilization // within tags Route\(\) string // ValidateBasic does a simple validation check that // doesn't require access to any other information. ValidateBasic\(\) Error // Get the canonical byte representation of the Msg. GetSignBytes\(\) \[\]byte // Signers returns the addrs of signers that must sign. // CONTRACT: All signatures must be present to be valid. // CONTRACT: Returns addrs in some deterministic order. GetSigners\(\) \[\]AccAddress }

### `Handlers` <a id="handlers"></a>

`Handlers` define the action that needs to be taken \(which stores need to get updated, how, and under what conditions\) when a given `Msg` is received.

In this module you have three types of `Msgs` that users can send to interact with the application state: [`SetName`](https://tutorials.cosmos.network/nameservice/tutorial/set-name.html), [`BuyName`](https://tutorials.cosmos.network/nameservice/tutorial/buy-name.html) and [`DeleteName`](https://tutorials.cosmos.network/nameservice/tutorial/delete-name.html). They will each have an associated `Handler`.

Now that you have a better understanding of `Msgs` and `Handlers`, you can start building your first message: `SetName`

If you had any difficulties following this tutorial or simply want to discuss Cosmos tech with us you can [**join our community today**](https://discord.gg/fszyM7K)!

