#

## Install Rust and configure the Solana CLI

For simplicity, perform both of these installations inside the project root:

* [Install the latest Rust stable](https://rustup.rs) : 

```bash
    curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
```

## Install Solana CLI

* [Install Solana CLI](https://docs.solana.com/cli/install-solana-cli-tools) v1.6.6 or later :

```bash
    sh -c "$(curl -sSfL https://release.solana.com/stable/install)"
```

## Set up the 

Set the CLI config URL to the devnet cluster by running this command in your Terminal:

```bash
    solana config set --url https://api.devnet.solana.com
```

## Set up a keypair

If this is your first time using the Solana CLI, you will need to generate a new keypair. 
Run the following command in your Terminal :

```bash
solana-keygen new
```

It will be written to `~/.config/solana/id.json` and will be used every time you use the CLI.

