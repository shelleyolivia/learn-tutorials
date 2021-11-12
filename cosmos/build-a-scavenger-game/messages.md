---
description: Learn how to build a scavenger game (continued)
---

# Messages

## Messages <a id="messages"></a>

Messages are a great place to start when building a module because they define the actions that your application can make. Think of all the scenarios where a user would be able to update the state of the application in any way. These should be boiled down into basic interactions, similar to **CRUD** \(Create, Read, Update, Delete\).

Let's start with **Create**

### MsgCreateScavenge

Messages are `types` which live inside the `./x/scavenge/types/` directory. There is already a `msg.go` file but we will make a new file for each Message type. We can use `msg.go` as a starting point by renaming it to `MsgCreateScavenge.go` like:

```javascript
# Assuming your current working directory is the root of your application
mv ./x/scavenge/types/msg.go  ./x/scavenge/types/MsgCreateScavenge.go
```

Inside this new file we will uncomment and follow the instructions of renaming variables until it looks as follows:

```javascript
package types

import (
    sdk "github.com/cosmos/cosmos-sdk/types"
    sdkerrors "github.com/cosmos/cosmos-sdk/types/errors"
)

// MsgCreateScavenge
// ------------------------------------------------------------------------------
var _ sdk.Msg = &MsgCreateScavenge{}

// MsgCreateScavenge - struct for unjailing jailed validator
type MsgCreateScavenge struct {
    Creator      sdk.AccAddress `json:"creator" yaml:"creator"`           // address of the scavenger creator
    Description  string         `json:"description" yaml:"description"`   // description of the scavenge
    SolutionHash string         `json:"solutionHash" yaml:"solutionHash"` // solution hash of the scavenge
    Reward       sdk.Coins      `json:"reward" yaml:"reward"`             // reward of the scavenger
}

// NewMsgCreateScavenge creates a new MsgCreateScavenge instance
func NewMsgCreateScavenge(creator sdk.AccAddress, description, solutionHash string, reward sdk.Coins) MsgCreateScavenge {
    return MsgCreateScavenge{
        Creator:      creator,
        Description:  description,
        SolutionHash: solutionHash,
        Reward:       reward,
    }
}

// CreateScavengeConst is CreateScavenge Constant
const CreateScavengeConst = "CreateScavenge"

// nolint
func (msg MsgCreateScavenge) Route() string { return RouterKey }
func (msg MsgCreateScavenge) Type() string  { return CreateScavengeConst }
func (msg MsgCreateScavenge) GetSigners() []sdk.AccAddress {
    return []sdk.AccAddress{sdk.AccAddress(msg.Creator)}
}

// GetSignBytes gets the bytes for the message signer to sign on
func (msg MsgCreateScavenge) GetSignBytes() []byte {
    bz := ModuleCdc.MustMarshalJSON(msg)
    return sdk.MustSortJSON(bz)
}

// ValidateBasic validity check for the AnteHandler
func (msg MsgCreateScavenge) ValidateBasic() error {
    if msg.Creator.Empty() {
        return sdkerrors.Wrap(sdkerrors.ErrInvalidAddress, "creator can't be empty")
    }
    if msg.SolutionHash == "" {
        return sdkerrors.Wrap(sdkerrors.ErrInvalidRequest, "solutionScavengerHash can't be empty")
    }
    return nil
}
```

Notice that all Messages in the app need to follow the `sdk.Msg` interface. The Message `struct` contains all the necessary information when creating a new scavenge:

* `Creator` - Who created it. This uses the `sdk.AccAddress` type which represents an account in the app controlled by public key cryptograhy.
* `Description` - What is the question to be solved or description of the challenge.
* `SolutionHash` - The scrambled solution.
* `Reward` - This is the bounty that is awarded to whoever submits the answer first.

The `Msg` interface requires some other methods be set, like validating the content of the `struct`, and confirming the msg was signed and submitted by the Creator.

Now that one can create a scavenge the only other essential action is to be able to solve it. This should be broken into two separate actions as described before: `MsgCommitSolution` and `MsgRevealSolution`.

### MsgCommitSolution <a id="msgcommitsolution"></a>

This message type should live in `./x/scavenge/types/MsgCommitSolution.go` and look like:

```javascript
package types

import (
    sdk "github.com/cosmos/cosmos-sdk/types"
    sdkerrors "github.com/cosmos/cosmos-sdk/types/errors"
)

// MsgCommitSolution
// ------------------------------------------------------------------------------
var _ sdk.Msg = &MsgCommitSolution{}

// MsgCommitSolution - struct for unjailing jailed validator
type MsgCommitSolution struct {
    Scavenger             sdk.AccAddress `json:"scavenger" yaml:"scavenger"`                         // address of the scavenger
    SolutionHash          string         `json:"solutionhash" yaml:"solutionhash"`                   // solutionhash of the scavenge
    SolutionScavengerHash string         `json:"solutionScavengerHash" yaml:"solutionScavengerHash"` // solution hash of the scavenge
}

// NewMsgCommitSolution creates a new MsgCommitSolution instance
func NewMsgCommitSolution(scavenger sdk.AccAddress, solutionHash string, solutionScavengerHash string) MsgCommitSolution {
    return MsgCommitSolution{
        Scavenger:             scavenger,
        SolutionHash:          solutionHash,
        SolutionScavengerHash: solutionScavengerHash,
    }
}

// CommitSolutionConst is CommitSolution Constant
const CommitSolutionConst = "CommitSolution"

// nolint
func (msg MsgCommitSolution) Route() string { return RouterKey }
func (msg MsgCommitSolution) Type() string  { return CommitSolutionConst }
func (msg MsgCommitSolution) GetSigners() []sdk.AccAddress {
    return []sdk.AccAddress{sdk.AccAddress(msg.Scavenger)}
}

// GetSignBytes gets the bytes for the message signer to sign on
func (msg MsgCommitSolution) GetSignBytes() []byte {
    bz := ModuleCdc.MustMarshalJSON(msg)
    return sdk.MustSortJSON(bz)
}

// ValidateBasic validity check for the AnteHandler
func (msg MsgCommitSolution) ValidateBasic() error {
    if msg.Scavenger.Empty() {
        return sdkerrors.Wrap(sdkerrors.ErrInvalidAddress, "creator can't be empty")
    }
    if msg.SolutionHash == "" {
        return sdkerrors.Wrap(sdkerrors.ErrInvalidRequest, "solutionHash can't be empty")
    }
    if msg.SolutionScavengerHash == "" {
        return sdkerrors.Wrap(sdkerrors.ErrInvalidRequest, "solutionScavengerHash can't be empty")
    }
    return nil
}
```

The Message `struct` contains all the necessary information when revealing a solution:

* `Scavenger` - Who is revealing the solution.
* `SolutionHash` - The scrambled solution \(hash\).
* `SolutionScavengerHash` - This is the hash of the combination of the solution and the person who solved it.

This message also fulfils the `sdk.Msg` interface.

### MsgRevealSolution <a id="msgrevealsolution"></a>

This message type should live in `./x/scavenge/types/MsgRevealSolution.go` and look like:

```javascript
package types

import (
    "crypto/sha256"
    "encoding/hex"
    "fmt"

    sdk "github.com/cosmos/cosmos-sdk/types"
    sdkerrors "github.com/cosmos/cosmos-sdk/types/errors"
)

// MsgRevealSolution
// ------------------------------------------------------------------------------
var _ sdk.Msg = &MsgRevealSolution{}

// MsgRevealSolution - struct for unjailing jailed validator
type MsgRevealSolution struct {
    Scavenger    sdk.AccAddress `json:"scavenger" yaml:"scavenger"`       // address of the scavenger scavenger
    SolutionHash string         `json:"solutionHash" yaml:"solutionHash"` // SolutionHash of the scavenge
    Solution     string         `json:"solution" yaml:"solution"`         // solution of the scavenge
}

// NewMsgRevealSolution creates a new MsgRevealSolution instance
func NewMsgRevealSolution(scavenger sdk.AccAddress, solution string) MsgRevealSolution {

    var solutionHash = sha256.Sum256([]byte(solution))
    var solutionHashString = hex.EncodeToString(solutionHash[:])

    return MsgRevealSolution{
        Scavenger:    scavenger,
        SolutionHash: solutionHashString,
        Solution:     solution,
    }
}

// RevealSolutionConst is RevealSolution Constant
const RevealSolutionConst = "RevealSolution"

// nolint
func (msg MsgRevealSolution) Route() string { return RouterKey }
func (msg MsgRevealSolution) Type() string  { return RevealSolutionConst }
func (msg MsgRevealSolution) GetSigners() []sdk.AccAddress {
    return []sdk.AccAddress{sdk.AccAddress(msg.Scavenger)}
}

// GetSignBytes gets the bytes for the message signer to sign on
func (msg MsgRevealSolution) GetSignBytes() []byte {
    bz := ModuleCdc.MustMarshalJSON(msg)
    return sdk.MustSortJSON(bz)
}

// ValidateBasic validity check for the AnteHandler
func (msg MsgRevealSolution) ValidateBasic() error {
    if msg.Scavenger.Empty() {
        return sdkerrors.Wrap(sdkerrors.ErrInvalidAddress, "creator can't be empty")
    }
    if msg.SolutionHash == "" {
        return sdkerrors.Wrap(sdkerrors.ErrInvalidRequest, "solutionScavengerHash can't be empty")
    }
    if msg.Solution == "" {
        return sdkerrors.Wrap(sdkerrors.ErrInvalidRequest, "solutionHash can't be empty")
    }

    var solutionHash = sha256.Sum256([]byte(msg.Solution))
    var solutionHashString = hex.EncodeToString(solutionHash[:])

    if msg.SolutionHash != solutionHashString {
        return sdkerrors.Wrap(sdkerrors.ErrInvalidRequest, fmt.Sprintf("Hash of solution (%s) doesn't equal solutionHash (%s)", msg.SolutionHash, solutionHashString))
    }
    return nil
}
```

The Message `struct` contains all the necessary information when revealing a solution:

* `Scavenger` - Who is revealing the solution.
* `SolutionHash` - The scrambled solution.
* `Solution` - This is the plain text version of the solution.

This message also fulfils the `sdk.Msg` interface.

### Codec <a id="codec"></a>

Once we have defined our messages, we need to describe to our encoder how they should be stored as bytes. To do this we edit the file located at `./x/scavenge/types/codec.go`. By describing our types as follows they will work with our encoding library:

```javascript
package types

import (
    "github.com/cosmos/cosmos-sdk/codec"
)

// RegisterCodec registers concrete types on codec
func RegisterCodec(cdc *codec.Codec) {
    cdc.RegisterConcrete(MsgCreateScavenge{}, "scavenge/CreateScavenge", nil)
    cdc.RegisterConcrete(MsgCommitSolution{}, "scavenge/CommitSolution", nil)
    cdc.RegisterConcrete(MsgRevealSolution{}, "scavenge/RevealSolution", nil)
}

// ModuleCdc defines the module codec
var ModuleCdc *codec.Codec

func init() {
    ModuleCdc = codec.New()
    RegisterCodec(ModuleCdc)
    codec.RegisterCrypto(ModuleCdc)
    ModuleCdc.Seal()
}
```

### Alias <a id="alias"></a>

Now that we have these new message types, we'd like to make sure other parts of the module can access them. To do so we use the `./x/scavenge/alias.go` file. This imports the types from the nested `types` directory and makes them accessible at the modules top level directory.

```javascript
package scavenge

import (
    "github.com/cosmos/sdk-tutorials/scavenge/x/scavenge/keeper"
    "github.com/cosmos/sdk-tutorials/scavenge/x/scavenge/types"
)

const (
    ModuleName        = types.ModuleName
    RouterKey         = types.RouterKey
    StoreKey          = types.StoreKey
    DefaultParamspace = types.DefaultParamspace
    // QueryParams       = types.QueryParams
    QuerierRoute = types.QuerierRoute
)

var (
    // functions aliases
    NewKeeper           = keeper.NewKeeper
    NewQuerier          = keeper.NewQuerier
    RegisterCodec       = types.RegisterCodec
    NewGenesisState     = types.NewGenesisState
    DefaultGenesisState = types.DefaultGenesisState
    ValidateGenesis     = types.ValidateGenesis

    // variable aliases
    ModuleCdc = types.ModuleCdc

    NewMsgCreateScavenge = types.NewMsgCreateScavenge
    NewMsgCommitSolution = types.NewMsgCommitSolution
    NewMsgRevealSolution = types.NewMsgRevealSolution
)

type (
    Keeper       = keeper.Keeper
    GenesisState = types.GenesisState
    Params       = types.Params

    MsgCreateScavenge = types.MsgCreateScavenge
    MsgCommitSolution = types.MsgCommitSolution
    MsgRevealSolution = types.MsgRevealSolution
)
```

It's great to have Messages, but we need somewhere to store the information they are sending. All persistent data related to this module should live in the module's `Keeper`.

Let's make a `Keeper` for our Scavenge Module next.

If you had any difficulties following this tutorial or simply want to discuss Cosmos tech with us you can [**join our community today**](https://discord.gg/fszyM7K)!

