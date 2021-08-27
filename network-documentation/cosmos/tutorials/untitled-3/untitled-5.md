---
description: Learn how to build a nameservice application (continued)
---

# Entry points

## Entry points <a id="entry-points"></a>

In Golang the convention is to place files that compile to a binary in the `./cmd` folder of a project. For your application there are 2 binaries that you want to create:

* `nsd`: This binary is similar to `bitcoind` or other cryptocurrency daemons in that it maintains p2p connections, propagates transactions, handles local storage and provides an RPC interface to interact with the network. In this case, Tendermint is used for networking and transaction ordering.
* `nscli`: This binary provides commands that allow users to interact with your application.

To get started create two files in your project directory that will instantiate these binaries:

* `./cmd/nsd/main.go`
* `./cmd/nscli/main.go`

### `nsd` <a id="nsd"></a>

Start by adding the following code to `cmd/nsd/main.go`:

> _NOTE_: Your application needs to import the code you just wrote. Here the import path is set to this repository \(`github.com/cosmos/sdk-tutorials/nameservice`\). If you are following along in your own repo you will need to change the import path to reflect that \(`github.com/{ .Username }/{ .Project.Repo }`\).

```javascript
package main

import (
    "encoding/json"
    "io"

    "github.com/cosmos/cosmos-sdk/server"
    "github.com/cosmos/cosmos-sdk/x/staking"

    "github.com/spf13/cobra"
    "github.com/spf13/viper"
    "github.com/tendermint/tendermint/libs/cli"
    "github.com/tendermint/tendermint/libs/log"

    "github.com/cosmos/cosmos-sdk/baseapp"
    "github.com/cosmos/cosmos-sdk/client/debug"
    "github.com/cosmos/cosmos-sdk/client/flags"
    sdk "github.com/cosmos/cosmos-sdk/types"
    "github.com/cosmos/cosmos-sdk/x/auth"
    genutilcli "github.com/cosmos/cosmos-sdk/x/genutil/client/cli"
    app "github.com/cosmos/sdk-tutorials/nameservice"
    abci "github.com/tendermint/tendermint/abci/types"
    tmtypes "github.com/tendermint/tendermint/types"
    dbm "github.com/tendermint/tm-db"
)

func main() {
    cobra.EnableCommandSorting = false

    cdc := app.MakeCodec()

    config := sdk.GetConfig()
    config.SetBech32PrefixForAccount(sdk.Bech32PrefixAccAddr, sdk.Bech32PrefixAccPub)
    config.SetBech32PrefixForValidator(sdk.Bech32PrefixValAddr, sdk.Bech32PrefixValPub)
    config.SetBech32PrefixForConsensusNode(sdk.Bech32PrefixConsAddr, sdk.Bech32PrefixConsPub)
    config.Seal()

    ctx := server.NewDefaultContext()

    rootCmd := &cobra.Command{
        Use:               "nsd",
        Short:             "nameservice App Daemon (server)",
        PersistentPreRunE: server.PersistentPreRunEFn(ctx),
    }
    // CLI commands to initialize the chain
    rootCmd.AddCommand(genutilcli.InitCmd(ctx, cdc, app.ModuleBasics, app.DefaultNodeHome))
    rootCmd.AddCommand(genutilcli.CollectGenTxsCmd(ctx, cdc, auth.GenesisAccountIterator{}, app.DefaultNodeHome))
    rootCmd.AddCommand(genutilcli.MigrateGenesisCmd(ctx, cdc))
    rootCmd.AddCommand(
        genutilcli.GenTxCmd(
            ctx, cdc, app.ModuleBasics, staking.AppModuleBasic{},
            auth.GenesisAccountIterator{}, app.DefaultNodeHome, app.DefaultCLIHome,
        ),
    )
    rootCmd.AddCommand(genutilcli.ValidateGenesisCmd(ctx, cdc, app.ModuleBasics))
    // AddGenesisAccountCmd allows users to add accounts to the genesis file
    rootCmd.AddCommand(AddGenesisAccountCmd(ctx, cdc, app.DefaultNodeHome, app.DefaultCLIHome))
    rootCmd.AddCommand(flags.NewCompletionCmd(rootCmd, true))
    rootCmd.AddCommand(debug.Cmd(cdc))

    server.AddCommands(ctx, cdc, rootCmd, newApp, exportAppStateAndTMValidators)

    // prepare and add flags
    executor := cli.PrepareBaseCmd(rootCmd, "NS", app.DefaultNodeHome)
    err := executor.Execute()
    if err != nil {
        panic(err)
    }
}

func newApp(logger log.Logger, db dbm.DB, traceStore io.Writer) abci.Application {
    return app.NewNameServiceApp(logger, db, baseapp.SetMinGasPrices(viper.GetString(server.FlagMinGasPrices)))
}

func exportAppStateAndTMValidators(
    logger log.Logger, db dbm.DB, traceStore io.Writer, height int64, forZeroHeight bool, jailWhiteList []string,
) (json.RawMessage, []tmtypes.GenesisValidator, error) {

    if height != -1 {
        nsApp := app.NewNameServiceApp(logger, db)
        err := nsApp.LoadHeight(height)
        if err != nil {
            return nil, nil, err
        }
        return nsApp.ExportAppStateAndValidators(forZeroHeight, jailWhiteList)
    }

    nsApp := app.NewNameServiceApp(logger, db)

    return nsApp.ExportAppStateAndValidators(forZeroHeight, jailWhiteList)
}
```

Notes on the above code:

* Most of the code above combines the CLI commands from Tendermint, Cosmos-SDK and your Nameservice module.

### `nscli` <a id="nscli"></a>

Finish up by building the `nscli` command:

> _NOTE_: Your application needs to import the code you just wrote. Here the import path is set to this repository \(`github.com/cosmos/sdk-tutorials/nameservice`\). If you are following along in your own repo you will need to change the import path to reflect that \(`github.com/{ .Username }/{ .Project.Repo }`\).

```javascript
package main

import (
    "os"
    "path"

    "github.com/cosmos/cosmos-sdk/client"
    "github.com/cosmos/cosmos-sdk/client/flags"
    "github.com/cosmos/cosmos-sdk/client/keys"
    "github.com/cosmos/cosmos-sdk/client/lcd"
    "github.com/cosmos/cosmos-sdk/client/rpc"
    sdk "github.com/cosmos/cosmos-sdk/types"
    "github.com/cosmos/cosmos-sdk/version"
    "github.com/cosmos/cosmos-sdk/x/auth"
    authcmd "github.com/cosmos/cosmos-sdk/x/auth/client/cli"
    authrest "github.com/cosmos/cosmos-sdk/x/auth/client/rest"
    "github.com/cosmos/cosmos-sdk/x/bank"
    bankcmd "github.com/cosmos/cosmos-sdk/x/bank/client/cli"
    app "github.com/cosmos/sdk-tutorials/nameservice"
    "github.com/spf13/cobra"
    "github.com/spf13/viper"
    amino "github.com/tendermint/go-amino"
    "github.com/tendermint/tendermint/libs/cli"
)

func main() {
    cobra.EnableCommandSorting = false

    cdc := app.MakeCodec()

    // Read in the configuration file for the sdk
    config := sdk.GetConfig()
    config.SetBech32PrefixForAccount(sdk.Bech32PrefixAccAddr, sdk.Bech32PrefixAccPub)
    config.SetBech32PrefixForValidator(sdk.Bech32PrefixValAddr, sdk.Bech32PrefixValPub)
    config.SetBech32PrefixForConsensusNode(sdk.Bech32PrefixConsAddr, sdk.Bech32PrefixConsPub)
    config.Seal()

    rootCmd := &cobra.Command{
        Use:   "nscli",
        Short: "nameservice Client",
    }

    // Add --chain-id to persistent flags and mark it required
    rootCmd.PersistentFlags().String(flags.FlagChainID, "", "Chain ID of tendermint node")
    rootCmd.PersistentPreRunE = func(_ *cobra.Command, _ []string) error {
        return initConfig(rootCmd)
    }

    // Construct Root Command
    rootCmd.AddCommand(
        rpc.StatusCommand(),
        client.ConfigCmd(app.DefaultCLIHome),
        queryCmd(cdc),
        txCmd(cdc),
        flags.LineBreak,
        lcd.ServeCommand(cdc, registerRoutes),
        flags.LineBreak,
        keys.Commands(),
        flags.LineBreak,
        version.Cmd,
        flags.NewCompletionCmd(rootCmd, true),
    )

    executor := cli.PrepareMainCmd(rootCmd, "NS", app.DefaultCLIHome)
    err := executor.Execute()

    if err != nil {
        panic(err)
    }
}

func registerRoutes(rs *lcd.RestServer) {
    client.RegisterRoutes(rs.CliCtx, rs.Mux)
    authrest.RegisterTxRoutes(rs.CliCtx, rs.Mux)
    app.ModuleBasics.RegisterRESTRoutes(rs.CliCtx, rs.Mux)
}

func queryCmd(cdc *amino.Codec) *cobra.Command {
    queryCmd := &cobra.Command{
        Use:     "query",
        Aliases: []string{"q"},
        Short:   "Querying subcommands",
    }

    queryCmd.AddCommand(
        authcmd.GetAccountCmd(cdc),
        flags.LineBreak,
        rpc.ValidatorCommand(cdc),
        rpc.BlockCommand(),
        authcmd.QueryTxsByEventsCmd(cdc),
        authcmd.QueryTxCmd(cdc),
        flags.LineBreak,
    )

    // add modules' query commands
    app.ModuleBasics.AddQueryCommands(queryCmd, cdc)

    return queryCmd
}

func txCmd(cdc *amino.Codec) *cobra.Command {
    txCmd := &cobra.Command{
        Use:   "tx",
        Short: "Transactions subcommands",
    }

    txCmd.AddCommand(
        bankcmd.SendTxCmd(cdc),
        flags.LineBreak,
        authcmd.GetSignCommand(cdc),
        authcmd.GetMultiSignCommand(cdc),
        flags.LineBreak,
        authcmd.GetBroadcastCommand(cdc),
        authcmd.GetEncodeCommand(cdc),
        authcmd.GetDecodeCommand(cdc),
        flags.LineBreak,
    )

    // add modules' tx commands
    app.ModuleBasics.AddTxCommands(txCmd, cdc)

    // remove auth and bank commands as they're mounted under the root tx command
    var cmdsToRemove []*cobra.Command

    for _, cmd := range txCmd.Commands() {
        if cmd.Use == auth.ModuleName || cmd.Use == bank.ModuleName {
            cmdsToRemove = append(cmdsToRemove, cmd)
        }
    }

    txCmd.RemoveCommand(cmdsToRemove...)

    return txCmd
}

func initConfig(cmd *cobra.Command) error {
    home, err := cmd.PersistentFlags().GetString(cli.HomeFlag)
    if err != nil {
        return err
    }

    cfgFile := path.Join(home, "config", "config.toml")
    if _, err := os.Stat(cfgFile); err == nil {
        viper.SetConfigFile(cfgFile)

        if err := viper.ReadInConfig(); err != nil {
            return err
        }
    }
    if err := viper.BindPFlag(flags.FlagChainID, cmd.PersistentFlags().Lookup(flags.FlagChainID)); err != nil {
        return err
    }
    if err := viper.BindPFlag(cli.EncodingFlag, cmd.PersistentFlags().Lookup(cli.EncodingFlag)); err != nil {
        return err
    }
    return viper.BindPFlag(cli.OutputFlag, cmd.PersistentFlags().Lookup(cli.OutputFlag))
}
```

Note:

* The code combines the CLI commands from Tendermint, Cosmos-SDK and your Nameservice module.
* The [`cobra` CLI documentation](http://github.com/spf13/cobra) will help with understanding the above code.
* You can see the `ModuleClient` defined earlier in action here.
* Note how the routes are included in the `registerRoutes` function.

#### Now that you have your binaries defined its time to deal with dependency management and build your app. <a id="now-that-you-have-your-binaries-defined-its-time-to-deal-with-dependency-management-and-build-your-app"></a>

If you had any difficulties following this tutorial or simply want to discuss Cosmos tech with us you can [**join our community today**](https://discord.gg/fszyM7K)!

